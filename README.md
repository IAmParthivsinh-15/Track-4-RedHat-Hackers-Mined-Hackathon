# Track-4-RedHat-Hackers-Mined-Hackathon

## Overview
This project aims to build a machine learning model using XGBoost to classify data based on selected features. The project involves data preprocessing, feature selection, model training, evaluation, and prediction.

## Dataset Creation
The company informed us that they used the Cuckoo Sandbox for creating the dataset. In this sandbox, the main environment was separated from the main system, and various malware, worms, and viruses were applied to the files using API functions, portable executables (PE), and dynamic link libraries (DLLs).

## Table of Contents
- [Data Preprocessing](#data-preprocessing)
- [Feature Selection](#feature-selection)
- [Model Training](#model-training)
- [Model Evaluation](#model-evaluation)
- [Model Saving and Loading](#model-saving-and-loading)
- [Prediction](#prediction)
- [Files](#files)

## Data Preprocessing
The data preprocessing steps include reading CSV files containing selected features, merging them, and handling missing values.

```python
s_dll_df = pd.read_csv("/kaggle/input/selected-features/dll_selected_features.csv")
s_api_df = pd.read_csv("/kaggle/input/selected-features/api_selected_features.csv")
s_pe_df = pd.read_csv("/kaggle/input/selected-features/pe_selected_features.csv")

df = pd.concat([s_api_df, s_dll_df, s_pe_df], axis=0, ignore_index=True)
df.fillna(0, inplace=True)
df_merged = df.groupby("SHA256", as_index=False).max()
df_merged.to_csv("merged_selected_features.csv")
```

### Intuition
Combining features from different sources (DLL, API, PE) provides a comprehensive view of the data, which can improve the model's performance. Handling missing values ensures that the model does not encounter errors during training.

## Feature Selection
Selected features are loaded from a `.npy` file and used for training the model.

```python
sel_feature_list = np.load("/kaggle/input/sel-featurelist/selected_features.npy")
```

### Intuition
Feature selection helps in reducing the dimensionality of the data, which can lead to faster training times and potentially better model performance by removing irrelevant or redundant features.

## Model Training
The model is trained using the XGBoost algorithm. The `ModelWrapper` class encapsulates the entire pipeline, including scaling and model training.

```python
class ModelWrapper:
    def __init__(self, selected_features, n_components=0.95, random_state=42):
        self.random_state = random_state
        self.scaler = StandardScaler()
        self.model = xgb.XGBClassifier(
            n_estimators=100,
            max_depth=7,
            learning_rate=0.2,
            subsample=0.8,
            colsample_bytree=0.8
        )
        self.selected_features = selected_features

    def fit(self, X_train, y_train):
        X_train_scaled = self.scaler.fit_transform(X_train[self.selected_features])
        self.model.fit(X_train_scaled, y_train)

    def transform(self, X):
        X_scaled = self.scaler.transform(X[self.selected_features])
        return X_scaled

    def predict(self, X):
        return self.model.predict(self.transform(X))

    def evaluate(self, X_test, y_test):
        rf_preds = self.predict(X_test)
        rf_acc = accuracy_score(y_test, rf_preds)
        print(f"Model Accuracy: {rf_acc:.4f}")
        return rf_acc

    def save(self, filename="model.pkl"):
        with open(filename, "wb") as f:
            pickle.dump(self, f)
        print(f"Model saved as '{filename}'")

    @staticmethod
    def load(filename="model.pkl"):
        with open(filename, "rb") as f:
            model = pickle.load(f)
        print(f"Model loaded from '{filename}'")
        return model
```

### Intuition
XGBoost is a powerful and efficient implementation of gradient boosting that can handle large datasets and complex relationships. Scaling the data ensures that all features contribute equally to the model, and encapsulating the entire pipeline in a class makes the code modular and reusable.

## Model Evaluation
The model's accuracy is evaluated on the test set.

```python
model = ModelWrapper(sel_feature_list)
model.fit(X_train, y_train)
model.evaluate(X_test, y_test)
```

### Intuition
Evaluating the model on a separate test set provides an unbiased estimate of its performance on unseen data, which is crucial for assessing its generalization ability.

## Model Saving and Loading
The trained model is saved to a file and can be loaded later for predictions.

```python
model.save(filename="xgb_model.pkl")
model = ModelWrapper.load("xgb_model.pkl")
```

### Intuition
Saving the model allows for easy reuse without retraining, which is useful for deploying the model in production or for further analysis.

## Prediction
The model is used to make predictions on new data.

```python
test_df = pd.read_csv("/kaggle/input/test-dataset/test.csv")
X = test_df.drop(columns=["SHA256"])
test_pred = model.predict(X)
test_pred = pd.concat([test_df['SHA256'], pd.DataFrame(test_pred)], axis=1, ignore_index=True)
test_pred.columns = ['SHA256', 'pred']
test_pred.to_csv("test_result.csv")
```

### Intuition
Making predictions on new data is the ultimate goal of the model, and saving the results to a CSV file allows for easy analysis and sharing.

## Files
- `Model_XGBoost.ipynb`: Jupyter notebook containing the code for data preprocessing, model training, evaluation, and prediction.
- `Crest_RedHat-Hackers_Code.ipynb`: Additional notebook (details not provided).
- `README.md`: This file, providing an overview and detailed explanation of the project.
