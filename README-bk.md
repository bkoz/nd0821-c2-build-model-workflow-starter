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

Use `test_dataset` vs. `test_artifact` per the course project assignment page.

```
mlflow run . -P steps=test_regression_model
```

#### 4) Pipeline Release and Updates

```
mlflow run https://github.com/bkoz/nd0821-c2-build-model-workflow-starter.git -v 1.0.0 -P hydra_options="etl.sample='sample2.csv'"
```
Output
```
test_data.py::test_column_names PASSED                                                                                                                   [ 16%]
test_data.py::test_neighborhood_names PASSED                                                                                                             [ 33%]
test_data.py::test_proper_boundaries FAILED                                                                                                              [ 50%]
test_data.py::test_similar_neigh_distrib PASSED                                                                                                          [ 66%]
test_data.py::test_row_count PASSED                                                                                                                      [ 83%]
test_data.py::test_price_range PASSED                                                                                                                    [100%]

=========================================================================== FAILURES ===========================================================================
____________________________________________________________________ test_proper_boundaries ____________________________________________________________________

data =              id                                               name   host_id  ... reviews_per_month calculated_host_li...l's Kitchen  68119814  ...               NaN                              1               23

[46428 rows x 16 columns]

    def test_proper_boundaries(data: pd.DataFrame):
        """
        Test proper longitude and latitude boundaries for properties in and around NYC
        """
        idx = data['longitude'].between(-74.25, -73.50) & data['latitude'].between(40.5, 41.2)
    
>       assert np.sum(~idx) == 0
E       assert 1 == 0
E         +1
E         -0

test_data.py:49: AssertionError
=================================================================== short test summary info ====================================================================
FAILED test_data.py::test_proper_boundaries - assert 1 == 0
================================================================= 1 failed, 5 passed in 17.56s 
```



#### Run Entire Pipeline
```
mlflow run . -P steps="all"
```

### Misc Commands
```
wandb artifact ls nyc_airbnb

wandb artifact get nyc_airbnb/clean_sample.csv:v0
```
