
# Car Price Prediction

This project is a web application that predicts the selling price of a car based on its mileage and age using a machine learning model. The application is built using Flask for the web interface and scikit-learn for the machine learning model.

#### Supervised by 
[Prof. Agughasi Victor Ikechukwu](https://github.com/Victor-Ikechukwu), 
(Assistant Professor) 
Department of CSE, MIT Mysore

#### Collaborators
- 4MH21CS006 [Appu C]
- 4MH21CS007 [Appuraj]
- 4MH22CS408 [Sumanth G N]("https://github.com/sumanthjangam")

## Project Structure

- `model.py`: Script to train the RandomForestRegressor model and save it along with the scaler in an HDF5 file.
- `app.py`: Flask application script to serve the web interface and make predictions.
- `templates/index.html`: HTML template for the web interface.
- `model/train_test_split.h5`: HDF5 file containing the trained model and scaler (generated by `model.py`).


```
Car Price Prediction/
│
├── models/
│   └── train_test_split.h5
│
├── templates/
│   └── index.html
│
├── app.py
└── model.py
```
## Project Files

**model.py**
- This script trains a RandomForestRegressor model on the car prices dataset and saves the trained model and scaler in an HDF5 file.

**app.py**
- This is the Flask application script. It loads the trained model and scaler from the HDF5 file and serves the web interface for making predictions.

**templates/index.html**
- This HTML file contains the web interface for the car price prediction form. It includes styling and animations for a better user experience.

**train_test_split.h5**
- This HDF5 file contains the trained RandomForestRegressor model and the scaler for feature scaling. It is generated by running the train_model.py script.


## Setup Instructions

## Prerequisites

- Python 3.12
- Required Python packages:
  - pandas
  - scikit-learn
  - h5py
  - flask
  - pickle
  - numpy

## Setup and Usage

### Installation

1. **Clone the repository**:
    ```bash
    git clone https://github.com/sumanthjangam/Car-Price-Prediction.git
    cd Car-Price-Prediction
    ```

2. **Create a virtual environment**:
    ```bash
    # On Windows use
    python -m venv venv
    venv\Scripts\activate
    ```

3. **Install the required packages**:
    ```bash
    pip install pandas scikit-learn h5py flask numpy
    ```

4. **Ensure the model file exists**:
    Make sure `train_test_split.h5` is present in the `models` directory.

## Usage

1. **Run the Flask application**:
    ```bash
    python app.py
    ```

2. **Access the application**:
    Open your web browser and go to `http://127.0.0.1:5000/`.

3. **Enter the required details**:
    Fill in the form with the necessary details and click on the "Predict" button to get the prediction.

## Code Explanation

### `app.py`

This is the main Flask application file.
```python
import pandas as pd
from sklearn.preprocessing import StandardScaler
from sklearn.ensemble import RandomForestRegressor
from flask import Flask, request, render_template
import h5py
import pickle
import numpy as np

app = Flask(__name__)

# Load the model and scaler once when the application starts
with h5py.File('models/train_test_split.h5', 'r') as h5f:
    model_data = h5f['model'][()]
    rf = pickle.loads(model_data)
    X_train_data = h5f['X_train'][:]
    scaler = StandardScaler()
    scaler.fit(X_train_data)

@app.route('/', methods=['GET', 'POST'])
def predict():
    result = None
    if request.method == 'POST':
        try:
            # Retrieve values from form
            Mileage = int(request.form['Mileage'])
            Age_yrs = int(request.form['Age(yrs)'])

            # Create a dataframe for features with the correct column names
            features = pd.DataFrame([[Mileage, Age_yrs]], columns=['Mileage', 'Age(yrs)'])

            # Scale features
            features_scaled = scaler.transform(features)

            # Make prediction
            prediction = rf.predict(features_scaled)

            # Interpret result
            result = f"Predicted Sell Price: ${prediction[0]:,.2f}"
        except ValueError:
            result = "Please enter valid values in all fields"

    return render_template('index.html', result=result)

if __name__ == '__main__':
    app.run(debug=True)
```

### `templates/index.html`

This is the HTML template for the web form.

```html
<!DOCTYPE html>
<html>
<head>
    <title>Car Price Prediction</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4f4;
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-image: linear-gradient(to right, rgba(255, 255, 255, 0.2) 1px, transparent 1px), linear-gradient(to bottom, rgba(255, 255, 255, 0.2) 1px, transparent 1px);
            background-size: 40px 40px;
            overflow: hidden;
        }

        @keyframes fadeIn {
            from {
                opacity: 0;
                transform: translateY(-20px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        .container {
            background-color: #fff;
            padding: 20px 40px;
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
            max-width: 400px;
            width: 100%;
            text-align: center;
            animation: fadeIn 1s ease-in-out;
        }

        h1 {
            color: #333;
            margin-bottom: 20px;
            animation: fadeIn 1.2s ease-in-out;
        }

        form {
            display: flex;
            flex-direction: column;
            animation: fadeIn 1.4s ease-in-out;
        }

        label {
            margin: 10px 0 5px;
            text-align: left;
        }

        input[type="text"] {
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
            margin-bottom: 20px;
            transition: border-color 0.3s;
        }

        input[type="text"]:focus {
            border-color: #007bff;
        }

        input[type="submit"] {
            background-color: #28a745;
            color: #fff;
            padding: 10px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            transition: background-color 0.3s, transform 0.3s;
        }

        input[type="submit"]:hover {
            background-color: #218838;
            transform: scale(1.05);
        }

        .result {
            margin-top: 20px;
            font-size: 18px;
            color: #333;
            animation: fadeIn 1.6s ease-in-out;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Car Price Prediction</h1>
        <form method="post">
            <label for="Mileage">Mileage:</label>
            <input type="text" id="Mileage" name="Mileage"><br><br>
            <label for="Age(yrs)">Age (yrs):</label>
            <input type="text" id="Age(yrs)" name="Age(yrs)"><br><br>
            <input type="submit" value="Predict">
        </form>
        {% if result %}
        <div class="result">
            <h2>Result: {{ result }}</h2>
        </div>
        {% endif %}
    </div>
</body>
</html>
```

## Model Training (`model.py`)

This script is used to train the Random Forest model and save it to an HDF5 file.

```python
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.ensemble import RandomForestRegressor
from sklearn.metrics import mean_squared_error
import h5py
import pickle
import numpy as np

# Load the dataset
df = pd.read_csv('data\carprices.csv')
print(df)

# Split data into training and test sets
X = df[['Mileage', 'Age(yrs)']]
y = df['Sell Price($)']
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=10)

# Scale the features
scaler = StandardScaler()
X_train = scaler.fit_transform(X_train)
X_test = scaler.transform(X_test)

# Train a Random Forest Regressor
rf = RandomForestRegressor(n_estimators=100, random_state=42)
rf.fit(X_train, y_train)

# Predict and display the mean squared error
y_pred = rf.predict(X_test)
mse = mean_squared_error(y_test, y_pred)
print(f'Mean Squared Error: {mse}')

# Save the model and data splits into an HDF5 file
hdf5_path = 'models/train_test_split.h5'
with h5py.File(hdf5_path, 'w') as h5f:
    h5f.create_dataset('X_train', data=X_train)
    h5f.create_dataset('X_test', data=X_test)
    h5f.create_dataset('y_train', data=y_train)
    h5f.create_dataset('y_test', data=y_test)
    model_data = pickle.dumps(rf)
    h5f.create_dataset('model', data=np.void(model_data))

print(f"Data and model saved to {hdf5_path}")
```
## Screenshots
![image](https://github.com/user-attachments/assets/eb503d96-09fc-4c5b-8b90-0a2bc5daf9e0)


## Conclusion

This project demonstrates how to build a car price prediction web application using Flask and scikit-learn. The model predicts car prices in rupees based on mileage and age. By following the setup instructions, you can train the model and deploy the web application locally. This project serves as a practical example of integrating machine learning with web development.


## License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
