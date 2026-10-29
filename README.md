# U.S. Crude Oil Import Forecasting with Azure Machine Learning

## Project overview
This MBA analytics project explores whether historical U.S. crude oil imports by source country can help forecast next-month import volumes and support procurement and supply-planning decisions.

**Data source:** U.S. Energy Information Administration (EIA) petroleum import data.  
**Statistical algorithm selected:** Linear Regression, after comparing out-of-sample results with alternative models.  
**Platform:** Microsoft Azure Machine Learning Studio.  
**Visualization:** Tableau preparation dataset included; add the workbook or screenshots after the dashboard is built.

## Business question
Which source countries may increase or decrease crude oil imports to the United States in the next month, and how could those estimates inform procurement planning and supplier monitoring?

## Repository contents
- `data/crude_import_ml_dataset.csv` — prepared country-level monthly ML dataset.
- `data/crude_import_tableau_dataset.csv` — cleaned dataset prepared for Tableau.
- `notebooks/` — data preparation and analysis notebooks.
- `models/` — serialized model and model feature list.
- `outputs/prediction_results.csv` — test-set predictions and errors.
- `presentation/` — project presentation, when added.

## Dataset scope and preparation
The modelling dataset uses monthly series for all grades of crude oil from eligible source countries to Total U.S. It contains 4,249 rows and 15 source columns before one-hot encoding. Feature engineering includes historical lag variables, rolling averages, and calendar variables. The final modelling matrix contains 42 features.

The source EIA file was approximately 350 MB and is intentionally not committed to this repository. Obtain the original data from the EIA and follow the documented preparation steps. Add the official source-page link here after confirming the exact download page used by the project.

## Model evaluation
Reported results from the chronological test set (844 rows, January 2023–June 2026):

| Model | MAE | RMSE | R² |
|---|---:|---:|---:|
| Linear Regression | 1,323.79 | 2,270.14 | 0.9922 |
| Random Forest | 1,447.81 | 2,654.31 | 0.9894 |
| HistGradientBoosting | 1,464.24 | 2,671.42 | 0.9892 |
| Gradient Boosting | 1,481.34 | 2,679.30 | 0.9892 |
| XGBoost | 1,558.43 | 2,934.93 | 0.9870 |
| Naive Forecast | 1,485.62 | 2,642.67 | 0.9895 |

Volume metrics are in thousand barrels. MAE and RMSE are lower-is-better; R² describes the fraction of test-target variation explained by the model and is not the same as prediction accuracy.

## Important caveats
- Verify temporal alignment between forecast target, input features, and train/test cutoff before treating results as production-ready.
- Linear Regression can predict negative values; actual import volumes cannot be negative.
- Historical import patterns do not establish causal effects or by themselves measure geopolitical supply risk.
- Country-level performance can differ from the pooled evaluation metrics.
- Price, freight, inventory, refinery demand, and geopolitical indicators are outside the current feature set.

## Reproducibility
The notebooks document the analysis sequence. Update this section with the tested Python version, dependency versions, and precise execution order before claiming full reproducibility. Azure resource names are provided in the presentation/notebook; do not publish credentials, secrets, or private configuration.

## Academic context
Prepared for the MBA assignment **Building Machine Learning Solutions using Azure ML Studio**.
