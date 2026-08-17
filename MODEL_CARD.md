# Model Card: Fraud Detection

## Model summary

This project uses a **Gradient Boosting classifier** to identify fraudulent transactions in the BankSim synthetic payment dataset. Logistic Regression, Random Forest, and Gradient Boosting were compared on validation data. Gradient Boosting achieved the strongest validation F1 score and was selected with a probability threshold of **0.35**.

This model card documents the model's purpose, inputs, evaluation, and limitations. The complete reproducible workflow is available in `fraud-detection-analysis.ipynb`.

## Intended use

The model is appropriate for:

- learning and demonstrating imbalanced-class machine learning;
- comparing classification algorithms and decision thresholds;
- exploring fraud-related patterns in synthetic transaction data;
- serving as a portfolio example of leakage-aware model evaluation.

It is **not** intended to approve, decline, freeze, or investigate real financial accounts. A production fraud system would require real-world validation, probability calibration, cost analysis, monitoring, security review, and human oversight.

## Training data

| Property | Value |
|---|---:|
| Dataset | BankSim synthetic transactions |
| Transactions | 594,643 |
| Fraudulent transactions | 7,200 |
| Fraud rate | 1.21% |
| Simulation steps | 180 |
| Train / validation / test | 64% / 16% / 20% |

The dataset contains simulated rather than real customer activity. See `DATASET_LICENSE.md` for attribution and licensing information.

## Model inputs

The final pipeline uses:

- `step` — simulation time step;
- `amount_log` — log-transformed transaction amount;
- `merchant_txn_count` — merchant frequency learned from development data;
- `amount_ge_95p` and `amount_ge_99p` — high-amount indicators based on development-data percentiles;
- `age`, `gender`, and `category` — categorical transaction attributes.

Raw customer IDs, merchant IDs, ZIP-code fields, and raw transaction amount are not passed directly to the classifier. Percentile thresholds and merchant frequencies are learned only from the relevant training data to avoid validation or test leakage.

## Preprocessing

- Numeric features use median imputation and standard scaling.
- Categorical features use most-frequent imputation and one-hot encoding.
- Unknown categories are ignored safely during transformation.
- Stratified splits preserve the rare fraud-class ratio.

## Evaluation

The model and threshold were selected using validation F1. After selection, the model was refit on the combined training and validation data and evaluated once on the untouched test set.

| Test metric | Score |
|---|---:|
| Precision | 0.8265 |
| Recall | 0.7875 |
| F1 | 0.8065 |
| ROC-AUC | 0.9973 |
| PR-AUC | 0.8943 |

At the selected threshold, the test confusion matrix contains:

- 1,134 true positives;
- 238 false positives;
- 306 false negatives;
- 117,251 true negatives.

Because fraud is rare, precision, recall, F1, and PR-AUC are more informative than accuracy alone.

## Decision threshold

The threshold of **0.35** maximized validation F1 for the selected model. It is not a universal business threshold. Lower thresholds generally detect more fraud but create more false alerts; higher thresholds reduce false alerts but miss more fraudulent transactions. A real deployment should choose the threshold using the financial cost of each error type.

## Limitations

- BankSim is synthetic and cannot represent every real fraud pattern.
- The evaluation uses a stratified random split rather than a forward-looking temporal test.
- The project does not evaluate probability calibration or financial loss.
- Categorical patterns may not transfer across markets, products, or time periods.
- Fraud behavior changes, so production performance would require drift monitoring and scheduled reassessment.

## Responsible use

Predictions should support investigation rather than act as the sole basis for decisions affecting customers. Human review, appeal paths, privacy controls, and subgroup performance checks would be necessary before any real-world use.

