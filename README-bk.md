# Build an ML Pipeline for Short-term Rental Prices in NYC

## Dev Environment

- Macbook x86_64 (late 2020), Monterey 12.4
- python=3.9.13
- Follow README to create a compatible Python/Conda environment

### Pipeline Components

#### 1) get_data

- Input: Download sample data (csv file) from github.
- Output: Upload data artifact to wandb.

```
mlflow run . -P steps=download
```

#### 1b) EDA (optional)
```
mlflow run src/eda
```

Remember to `close and halt` the notebook then `quit` jupyter in order for mlflow to finish cleanly.

#### 2) data_cleaning

- Input: Download artifact from wandb.
- Output: Upload cleaned data artifact to wandb.

```
mlflow run . -P steps=basic_cleaning
```

#### 3) Data Checks

- Input: Sample reference artifact from wandb.
- Input: Sample data (snapshot) artifact from wandb.
- Output: Logs `pytest` results to standard output.

Go into the wandb UI and tag the latest `clean_sample.csv` artifact using a `reference` tag.
This is needed or the data checks will fail.

```
mlflow run . -P steps="data_check"
```

#### 4) Initial Training

- Split data into test and training sets.
  - Input: Sample data artifact from wandb.
  - Output: Training artifact to wandb.
  - Output: Test artifact to wandb.

```
mlflow run . -P steps="data_split"
```

- Train Model
 - Output: Save `random_forest_export` artifact to wandb
```
mlflow run . -P steps="train_random_forest"
```
- Hyper Parameter Tuning
```
mlflow run . -P steps="train_random_forest" -P hydra_options="hydra/launcher=joblib modeling.max_tfidf_features=10,15,30 modeling.random_forest.max_features=0.1,0.33,0.5,0.75,1.0 --multirun"
```

How to create a new experiment (wandb group)
```
mlflow run . -P steps="train_random_forest" -P hydra_options="hydra/launcher=joblib modeling.max_tfidf_features=10,15,30 modeling.random_forest.max_features=0.1,0.33,0.5,0.75,1.0 main.experiment_name=test_experiment --multirun"
```

Tag the model with the best MAE as `:prod` using the wandb UI.

- Validation Testing
```
mlflow run . -P steps=test_regression_model
```

#### 4) Pipeline Release and Updates

#### Run Entire Pipeline
```
mlflow run . -P steps="all"
```

### Misc Commands
```
wandb artifact ls nyc_airbnb

wandb artifact get nyc_airbnb/clean_sample.csv:v0
```
