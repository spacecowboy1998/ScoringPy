# ScoringPy

**Overview**  
ScoringPy is an open-source Python library designed to streamline the development and deployment of classical credit scorecards. It simplifies the entire process from data preprocessing to scorecard scaling by providing robust tools and methods that ensure data integrity and model performance. By incorporating multiple layers of data anomaly detection, ScoringPy minimizes errors before model training, enhancing efficiency and reliability.

The library is divided into three main components:
1. **Data Preprocessing Pipeline**: Automate and save data manipulation steps using pipelines, ensuring consistency and efficiency when validating models or testing new data.
2. **Feature and Binning Selection**: Perform both automatic and manual feature selection and binning, with comprehensive reports and visualizations based on Weight of Evidence (WoE) analysis.
3. **Scorecard Deployment and Scaling**: Deploy and scale scorecards with customizable parameters, allowing for precise control over model coefficients and constants.

By using ScoringPy, you can build robust credit scoring models with ease, reduce error rates, and enhance efficiency throughout the credit scoring process.

## Features
- **Data Preprocessing with Pipeline**: Automate and save every data manipulation step using a pipeline, which can be easily reapplied to new data. This ensures consistent preprocessing and reduces the likelihood of errors.
- **Feature Selection with WoE Analysis**: Generate detailed reports and visualizations for each feature based on WoE and Information Value (IV). This includes statistical summaries that help in understanding the predictive power of each feature.
- **Binning (Manual and Automatic)**: Bin continuous features for classical scoring models. Choose between manual binning or automatic suggestions provided by the library. Binning validation is included in the feature statistics report, checking if any data falls outside bin ranges.
- **Final Data Transformation**: Apply a second layer of protection against outlier data. The library alerts you if any data points fall outside the defined bin ranges during transformation.
- **Scorecard Deployment and Scaling**: Scale scores based on the model's coefficients and constants. The scaling is fully customizable, with default values optimized for most scaling scenarios.
- **Performance Testing**: Easily test the scorecard's performance on different data populations using the preprocessing pipelines.
- **Monitoring**: Track scorecard and population performance over time, leveraging the consistent preprocessing steps provided by the pipelines.

## Installation

You can install ScoringPy using either `pip` or `conda`.

### Using pip

```bash
pip install ScoringPy
```

### Using conda

```bash
conda install -c conda-forge ScoringPy
```

## Usage

ScoringPy provides several modules, each designed for a specific part of the credit scoring process:

- **Processing**: For data preprocessing.
- **WoeAnalysis**: For feature selection and binning using WoE analysis.
- **WoeBinning**: For transforming data based on the selected features and bins.
- **CreditScoring**: For scaling scores and probabilities based on the model and scaling constants.

Below are detailed explanations and examples for each module.

### Processing
The **Processing** module automates data preprocessing steps using pipelines. Every transformation is saved and can be easily reapplied to new data, which is crucial for model validation and testing.

#### Pipeline Initialization
To create a processing pipeline, initialize it using the `Processing` class. You can enable or disable automatic data flow between steps using the `flow` parameter.

- **flow** (optional, default `True`): If `True`, the output from each function (step) will be passed as input to the next function automatically. If `False`, you must manage data flow manually.

#### Type 1: Sequential Data Transformation with Automatic Flow
In this example, we'll create a pipeline with automatic data flow between steps:

```python
from ScoringPy import Processing
import pandas as pd
import dill

# Initialize the pipeline with flow control enabled
pipeline = Processing(flow=True)

# Define preprocessing functions
def fill_missing_age(data):
    """Fill missing values in the 'Age' column with the mean."""
    data['Age'] = data['Age'].fillna(data['Age'].mean())
    return data

def double_age(data):
    """Double the values in the 'Age' column."""
    data['Age'] = data['Age'] * 2
    return data

def scale_age(data):
    """Scale the 'Age' column by dividing by 5."""
    data['Age'] = data['Age'] / 5
    return data

# Add steps to the pipeline
pipeline.add_step(fill_missing_age)
pipeline.add_step(double_age)
pipeline.add_step(scale_age)

# Save the pipeline using dill
with open('pipeline.pkl', 'wb') as file:
    dill.dump(pipeline, file)

# Load your dataset
df = pd.read_csv('data.csv')

# Run the pipeline on the dataset
df_processed = pipeline.run(initial_data=df)

# Clear the pipeline if needed
pipeline.clear()
```


### Explanation:

1. **Initialization**: We initialize the `Processing` pipeline with `flow=True`, enabling automatic data flow between steps.

2. **Function Definitions**: We define three functions (`fill_missing_age`, `double_age`, `scale_age`) that perform specific data transformations.

3. **Adding Steps**: We add these functions to the pipeline using `pipeline.add_step()`.

4. **Saving the Pipeline**: We use the `dill` library to serialize and save the pipeline for future reuse.

5. **Running the Pipeline**: We run the pipeline on the dataset using `pipeline.run(initial_data=df)`.

6. **Clearing the Pipeline**: We clear the pipeline using `pipeline.clear()` if we need to reset it.

### Reusing the Pipeline
You can load the saved pipeline and apply it to new data without redefining the steps:

```python
import dill
import pandas as pd

# Load the saved pipeline
with open('pipeline.pkl', 'rb') as file:
    pipeline = dill.load(file)

# Load new data
df_new = pd.read_csv('new_data.csv')

# Run the pipeline on the new data
df_processed_new = pipeline.run(initial_data=df_new)

# Clear the pipeline if needed
pipeline.clear()
```

### Type 2: Non-Sequential Data Processing with Manual Flow
If you need more control over the data flow between steps, you can set `flow=False` when initializing the pipeline.

```python
from ScoringPy import Processing
import pandas as pd
import dill

# Initialize the pipeline without automatic flow
pipeline = Processing(flow=False)

# Define functions for each step
def load_data_step1(path=None):
    """Load data from an Excel file."""
    data = pd.read_excel(path)
    return data

def load_data_step2():
    """Load additional data from another Excel file."""
    data = pd.read_excel('Data/step2.xlsx')
    return data

def concatenate_data():
    """Concatenate data from step 1 and step 2."""
    step1_data = pipeline.context.get('load_data_step1')
    step2_data = pipeline.context.get('load_data_step2')
    data = pd.concat([step1_data, step2_data], ignore_index=True)
    data['Age'] = data['Age'] * 2
    return data

def finalize_data(data):
    """Finalize the data by scaling the 'Age' column."""
    data['Age'] = data['Age'] / 5
    return data

# Add steps to the pipeline
pipeline.add_step(load_data_step1, path='Data/step1.xlsx')
pipeline.add_step(load_data_step2)
pipeline.add_step(concatenate_data, flow=True)
pipeline.add_step(finalize_data, flow=True)

# Save the pipeline
with open('pipeline.pkl', 'wb') as file:
    dill.dump(pipeline, file)

# Run the pipeline
df_processed = pipeline.run()

# Clear the pipeline if needed
pipeline.clear()
```

### Explanation:

1. **Initialization**: We initialize the `Processing` pipeline with `flow=False`, disabling automatic data flow.

2. **Function Definitions**: We define functions for loading data and concatenating datasets.

3. **Using `pipeline.context`**: We use `pipeline.context.get()` to retrieve data from previous steps.

4. **Flow Control**: We set `flow=True` for steps where we want the output to be passed to the next step.

### Reusing the Pipeline

```python
import dill

# Load the pipeline
with open('pipeline.pkl', 'rb') as file:
    pipeline = dill.load(file)

# Run the pipeline
df_processed = pipeline.run()

# Clear the pipeline if needed
pipeline.clear()

```

### Processing Optional Arguments

- **flow** (`bool`, default `True`): Controls automatic data flow between steps. If set to `False`, you must manage the data flow manually.

## WoeAnalysis

The **WoeAnalysis** module is designed for feature selection and binning using WoE (Weight of Evidence) analysis. It provides small reports for each feature, including statistical summaries based on WoE analysis.

### Methods

- **discrete**: Analyze discrete (categorical) variables.
- **continuous**: Analyze continuous variables.

Each method supports:

- **plot**: Visualizes WoE and IV analysis.
- **report**: Displays and optionally saves the report.

### Analyzing Discrete Variables

```python
from ScoringPy import WoeAnalysis

# Initialize WoeAnalysis
woe_analysis = WoeAnalysis(save=False, path="Data/", type=2)

# Analyze a discrete variable with safety checks
woe_analysis.discrete(column="MaritalStatus", df=X_train, target=y_train, safety=True, threshold=300).report()
```

### Explanation:

1. **Initialization**: We initialize `WoeAnalysis` with optional parameters like `save`, `path`, and `type`.

2. **Safety Parameters**:
    - **safety** (`bool`, default `True`): Controls whether to perform safety checks on the feature before processing. If `True`, the method will refuse to process a feature if it contains more unique values than the specified threshold, preventing potential memory shortages or hardware crashes.
    - **threshold** (`int`, default `300`): Specifies the maximum number of unique values allowed in a discrete feature when `safety` is `True`. If the feature exceeds this threshold, it will not be processed until the user changes the threshold or turns off safety.

3. **Analyzing the Variable**: We call the `discrete` method, passing the column name, DataFrame `X_train`, target variable `y_train`, and safety parameters.

4. **Generating the Report**: We call the `report` method to display the analysis.

### Plotting and Saving the Report:

```python
# Generate a plot and display the report
woe_analysis.discrete(column="MaritalStatus", df=X_train, target=y_train, safety=True, threshold=300).plot(rotation=0).report()

# Save the report
woe_analysis.discrete(column="MaritalStatus", df=X_train, target=y_train, safety=True, threshold=300).report(save=True, type=1)

```

- **rotation**: Adjusts the rotation of x-axis labels in the plot.
- **save**: If `True`, saves the report.
- **type**: Specifies the format type when saving.

### Analyzing Continuous Variables
For continuous variables, you need to define bins.

```python
import numpy as np
import pandas as pd
from ScoringPy import WoeAnalysis

# Define bins using pandas IntervalIndex
bins = pd.IntervalIndex.from_tuples([
(-1, 0),(0, 0.2),(0.2, 0.35),(0.35, 0.45),(0.45, 0.55),(0.55, 0.65),(0.65, np.inf)])

# Analyze a continuous variable
woe_analysis.continuous(column="RefinanceRate", bins=bins, df=X_train, target=y_train).report()

# Plot and display the report
woe_analysis.continuous(column="RefinanceRate", bins=bins, df=X_train, target=y_train).plot(rotation=90).report()

# Save the report
woe_analysis.continuous(column="RefinanceRate", bins=bins, df=X_train, target=y_train).report(save=True)

```

### Additional Attributes

You can extract various attributes from the `woe_analysis` object for future use:

```python
WoE_dict = woe_analysis.WoE_dict           # Dictionary of WoE values
Variable_types = woe_analysis.Variable_types  # Types of variables analyzed
Variable_Ranges = woe_analysis.Variable_Ranges  # Ranges or bins used
IV_excel = woe_analysis.IV_excel           # IV values formatted for Excel
IV_dict = woe_analysis.IV_dict             # Dictionary of IV values
```

## WoeBinning

The `WoeBinning` module transforms your dataset based on the WoE analysis conducted earlier. It replaces the original feature values with their corresponding WoE values.

```python
from ScoringPy import WoeBinning

# Assume WoE_dict is obtained from WoeAnalysis
WoE_dict = woe_analysis.WoE_dict

# Initialize WoeBinning
woe_transform = WoeBinning(WoE_dict=WoE_dict, production=False)

# Transform the data
X_transformed = woe_transform.transform(X, dummy=False)

```

## Parameters:

- **WoE_dict**: The dictionary containing WoE values.
- **production** (`bool`, default `False`): Controls error handling for outliers.
    - If `False`, the transformer raises an error if it encounters outlier data not covered by the bins, allowing you to address data issues during development.
    - If `True`, it handles outliers by removing specific rows containing outliers and continues the transformation, which is suitable for production environments where interruptions are undesirable.
- **dummy** (`bool`, default `False`): Controls the structure of the output DataFrame.
    - If `True`, returns data with new columns derived from the WoE dictionary.
    - If `False`, transforms existing columns without changing the DataFrame's structure.

### Explanation:

1. **Transformation**: The transformed data will include only the columns specified in `WoE_dict`.

2. **Selective Transformation**: If you want to transform only specific features, remove unwanted features from `WoE_dict` before transformation.

## CreditScoring

The **CreditScoring** module scales scores and probabilities based on your logistic regression model and specific scaling constants. It allows you to generate a scorecard and apply it to your dataset.

### Steps

1. **Train a Logistic Regression Model**: Use the transformed data to train your model.
2. **Initialize CreditScoring**: Provide the data, model, WoE dictionary, and production mode.
3. **Apply Scoring**: Generate the scorecard and apply it to your data.

### Example

```python
from sklearn.linear_model import LogisticRegression
from ScoringPy import CreditScoring

# Assume X_transformed is your WoE-transformed data
# Assume y is your target variable

# Train the logistic regression model
model = LogisticRegression(max_iter=1000, class_weight='balanced', C=0.1)

# Initialize CreditScoring
scoring = CreditScoring(data=X_train, model=model, WoE_dict=WoE_dict, production=True)

# Apply scoring to the data
result = scoring.apply(X_train)

# Access the scored data and scorecard
df_scored = result.data
scorecard = result.scorecard

```

### Parameters:

- **data**: The dataset to score.
- **model**: The trained logistic regression model.
- **WoE_dict**: The WoE dictionary used for transformations.
- **production** (`bool`, default `True`): Controls error handling for outliers during scoring.
    - If `False`, the process will raise an error if it encounters data issues, suitable for development and debugging.
    - If `True`, it will handle outliers gracefully, making it suitable for production environments.

### Explanation:

1. **Scorecard Generation**: The `apply_scoring` method generates a scorecard based on the model's coefficients and constants.

2. **Scored Data**: The resulting `df_scored` DataFrame includes the calculated scores for each record.


## Performance Testing and Monitoring

By reusing the preprocessing pipeline and WoE transformations, you can ensure consistency in data preparation. This allows for accurate performance comparisons across different data populations, facilitating performance testing and monitoring over time.

## Best Practices and Detailed Explanations

### Data Preprocessing Pipeline

- **Consistency**: Saving and reusing pipelines ensures that the same data transformations are applied consistently across training and new data.
- **Flow Control**: Decide between automatic and manual flow based on the complexity of your data transformations.
- **Serialization**: Use `dill` for serializing the pipeline, which can handle complex objects like custom functions and classes.

### WoE Analysis and Binning

- **Safety Checks**: Use parameters like `safety` and `threshold` to prevent creating features with too many unique values or inappropriate data types.
    - **safety** (`bool`, default `True`): If `True`, the method performs a safety check on the feature before processing, designed to prevent hardware crashes due to memory shortages when dealing with high-cardinality features.
    - **threshold** (`int`, default `300`): Specifies the maximum number of unique values allowed in a discrete feature when `safety` is `True`. If the feature exceeds this threshold, it will not be processed unless you either increase the threshold or set `safety=False`.
- **Handling High Cardinality**: High-cardinality features can cause performance issues. The `safety` parameter helps prevent such issues by limiting the number of unique values.
- **Manual vs. Automatic Binning**: Choose manual binning for more control, or use automatic suggestions provided by the library.
- **Outlier Handling**: Use binning validation reports to adjust bins as necessary, ensuring that data falls within defined ranges.

### Data Transformation with WoeBinning

- **Selective Transformation**: Modify `WoE_dict` to include only the features you want to transform.
- **Production Mode**:
    - **Development Environment**: Set `production=False` to raise errors when outliers are encountered, allowing you to identify and fix data issues.
    - **Production Environment**: Set `production=True` to handle outliers gracefully by removing affected rows, ensuring uninterrupted processing.

### Credit Score Scaling

- **Customization**: Adjust scaling constants and parameters to fit your specific use case or regulatory requirements.
- **Scorecard Generation**: Use the generated scorecard to understand how scores are computed and for transparency in decision-making.
- **Monitoring**: Regularly test and monitor the scorecard's performance on new data to ensure it remains predictive.

## Conclusion

ScoringPy provides a comprehensive toolkit for building classical credit scorecards, from data preprocessing to score scaling. By automating and standardizing key steps in the credit scoring process, it helps reduce errors, improve efficiency, and ensure consistency across different datasets. With features like safety checks and production modes, it is designed to handle both development and production environments effectively.

## Contribution and Support

As an open-source project, contributions are welcome. If you encounter any issues or have suggestions for improvements, feel free to submit issues or pull requests on the GitHub repository.
