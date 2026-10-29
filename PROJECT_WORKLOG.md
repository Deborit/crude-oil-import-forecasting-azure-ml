# Project Work Log

This file records the important workflow, decisions, and caveats for the MBA project. It is a concise project record, not a verbatim export of the ChatGPT conversation.

## 1. Assignment

**Assignment:** Building Machine Learning Solutions using Azure ML Studio

**Required deliverables:**
- A presentation explaining a unique business problem, internet-based dataset, and statistical algorithm.
- A detailed algorithm overview using 5W1H, including how it works and real-world applications.
- A demonstration in Azure ML Studio, with interpretation of results and recommended business decisions.

## 2. Business problem

**Title:** Predicting U.S. Crude Oil Import Volumes and Identifying Country-Level Supply Risks Using Machine Learning.

The project asks whether historical monthly imports by source country can help forecast next-month U.S. crude oil imports and support procurement planning, inventory decisions, and supplier monitoring.

The current model is a forecasting decision-support prototype. It does not independently measure geopolitical supply risk or establish causal relationships.

## 3. Data source and scope

- Source: U.S. Energy Information Administration (EIA) petroleum import data.
- Raw file used locally: `PET_IMPORTS_full.csv` (approximately 350 MB); intentionally excluded from the repository due to size.
- Raw dataset profile: 972,117 observations, 46,140 unique series, 70 source geographies, and 49 U.S. destination geographies.
- Main modelling grain: eligible source country × month → Total U.S.
- Product: all grades of crude oil.
- Frequency: monthly series, identified through the source series identifier.
- Countries were retained using a minimum historical coverage threshold of 60 observations.

Do not mix Total U.S. aggregates with state, refinery, or port-level series in the same summed view; doing so can double-count volumes.

## 4. Prepared datasets

- `data/crude_import_ml_dataset.csv`: 4,249 rows and 15 columns before one-hot encoding; 33 eligible source countries; no missing values reported in the prepared dataset.
- `data/crude_import_tableau_dataset.csv`: 5,107 rows prepared for Tableau at country × month and Total U.S. destination grain.
- `outputs/prediction_results.csv`: 844 test-set predictions with actual values, predictions, signed errors, and absolute errors.

The Tableau preparation adds business-friendly country, destination, crude-grade, and date fields. The original EIA `frequency` field was entirely null, so a separate `Frequency = Monthly` field was created only after filtering monthly series; the original missing field was not imputed.

## 5. Feature engineering and target

The modelling target is `target_next_month`: the following month's import volume for each country.

Features include:
- Lagged import volumes: lag 1, 3, 6, and 12 months.
- Rolling averages over previous 3 and 6 months.
- Calendar fields: month, quarter, and year.
- One-hot encoded country indicators.

The final model matrix contains 42 features.

**Validation caveat:** Before production use or making strong accuracy claims, re-check the temporal alignment of the target, lag/rolling features, and chronological train/test split. Confirm that training labels do not extend across the forecast/test boundary and that every feature would have been available at the intended forecast origin.

## 6. Model comparison

Reported results on the chronological test set (844 rows, January 2023–June 2026):

| Model | MAE | RMSE | R² |
|---|---:|---:|---:|
| Linear Regression | 1,323.79 | 2,270.14 | 0.9922 |
| Random Forest | 1,447.81 | 2,654.31 | 0.9894 |
| HistGradientBoosting | 1,464.24 | 2,671.42 | 0.9892 |
| Gradient Boosting | 1,481.34 | 2,679.30 | 0.9892 |
| XGBoost | 1,558.43 | 2,934.93 | 0.9870 |
| Naive Forecast | 1,485.62 | 2,642.67 | 0.9895 |

Volumes and errors are in thousand barrels. Lower MAE and RMSE are better. R² = 0.9922 means the model explains about 99.22% of variation in the test targets; it does **not** mean 99.22% prediction accuracy.

Linear Regression was selected because it had the best reported out-of-sample MAE, RMSE, and R² among the tested approaches and outperformed the reported naive baseline on MAE and RMSE.

## 7. Azure Machine Learning implementation

Recorded environment:
- Workspace: `aml-petroleum-forecasting`
- Resource group: `rg-petroleum-forecasting`
- Region: East US 2
- Compute: `mlcompute-small` (Standard_DS11_v2)
- Data asset: `crude-import-ml-dataset`
- Notebook used in Azure: `07_AzureML_Linear_Regression.ipynb`

The model was trained using scikit-learn Linear Regression in an Azure ML notebook. The reported Azure evaluation matched the local model validation metrics.

**Repository note:** The Azure notebook was used in the Azure environment and may need to be downloaded into the local `notebooks/` folder before it can be included here. Never upload access tokens, Azure credentials, secrets, or private configuration.

## 8. Results interpretation and business use

The model can be used as an additional input for:
- Comparing expected country-level import volumes with procurement plans.
- Identifying forecasted changes that merit supplier monitoring.
- Reviewing large prediction deviations and country-level error patterns.
- Supporting inventory and supply-planning discussions.

A forecast decline, a large prediction error, and a real supply risk are different concepts. Decisions should combine the forecast with commercial judgment and, in future, price, freight, geopolitical, inventory, and refinery-demand information.

Some unconstrained Linear Regression predictions may be negative, which is physically impossible for import volume. A production version should apply appropriate constraints and evaluate their effect on performance.

## 9. Tableau preparation and dashboard plan

The Tableau dataset is intended for a main dashboard at source-country × month × Total U.S. destination grain. Recommended visuals include:
- Historical import trends.
- Top source countries.
- Country-level trends.
- Seasonality.
- Actual versus predicted values.
- Prediction-error distribution.
- Country-level error comparisons, when calculated from the correct prediction table.
- A decision-support view comparing forecasts with historical and procurement context.

Do not present a conceptual dashboard as implemented unless a Tableau workbook or screenshot is added.

## 10. Next documentation tasks

- Add the official EIA download/source-page link after confirming the exact URL used.
- Add the final presentation to a `presentation/` directory after checking group-member personal details are suitable for public release.
- Download and add the Azure ML notebook if permitted and free of credentials.
- Add a Tableau workbook or actual dashboard screenshots after creating the dashboard.
- Add a tested `requirements.txt` and execution instructions before claiming full reproducibility.
- Keep the large raw EIA file, virtual environments, credentials, and secrets out of Git.

## 11. Public repository checklist

This repository is public. Review all notebooks and files for credentials, personal information, private paths, and sensitive information before sharing. The raw EIA source file is intentionally not committed.
