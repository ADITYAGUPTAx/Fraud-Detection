# Fraud-Detection
Random Forrest Classifier trained on 6.5 million row dataset for predicting whether a transaction is fraud or not.

Data Source: The dataset can be found here https://drive.usercontent.google.com/download?id=1VNpyNkGxHdskfdTNRSjjyNa5qC9u0JyV&export=download&authuser=0


# Final Summary

## Step 1: Data Loading & Initial Exploration
#### Loaded the dataset with ~6.3 million transactions and 10 columns.
#### Observed that the target column was 'isFraud', and another column 'isFlaggedFraud' was just a rule-based flag.
#### Dropped 'isFlaggedFraud' early on because it was not useful for model training (it's more of a hard-coded rule).

##  Step 2: Class Imbalance & Filtering
#### Noticed that only about 0.13% of transactions were actually labeled as fraud — highly imbalanced dataset.
#### Found that fraud only occurred in two types of transactions: 'TRANSFER' and 'CASH_OUT'.
#### So, I removed all rows where 'type' was anything other than those two — this helped reduce noise and improve focus.

## Step 3: Feature Cleanup and Engineering
#### Converted the 'type' column to numeric using mapping: 'CASH_OUT' → 0 and 'TRANSFER' → 1.
#### Dropped 'nameOrig' and 'nameDest' since they were just IDs and not useful for learning patterns.
#### But before dropping 'nameDest', I created a new binary column 'isMerchant' by checking if the destination name started with 'M'.
#### This preserved merchant info (which isn’t otherwise reflected in balance columns).
#### Then, created two engineered features:
####   - errorBalanceOrig = oldbalanceOrg - amount - newbalanceOrig
####   - errorBalanceDest = newbalanceDest - oldbalanceDest - amount
#### These helped capture inconsistencies in balance movements — which are strong fraud indicators.

## Step 4: Removing Duplicates
#### Checked and removed 16 completely duplicate rows to avoid bias and unnecessary redundancy in the data.

## Step 5: Train-Test Split
#### Split the dataset into 80% for training and 20% for testing.
#### Used stratified split to preserve the class imbalance ratio (fraud vs non-fraud).

## Step 6: Model Training
#### Chose RandomForestClassifier because it handles imbalanced data well and is great for tabular problems.
#### Used class_weight='balanced' to ensure the model paid more attention to the rare fraud cases.
#### Also set max_depth and limited n_estimators during initial runs to reduce training time without losing much performance.

## Step 7: Evaluation
#### Achieved outstanding performance:
####   - Precision, recall, and F1-score for class 1 (fraud) were nearly perfect.
####   - Confusion matrix showed almost no false positives or false negatives.
####   - ROC AUC score also confirmed strong discrimination between fraud and non-fraud cases.
#### The model wasn't just memorizing — it generalizes well and performs reliably on unseen data.

## Final Thoughts:
#### The key to this performance was clean preprocessing, focusing only on useful transaction types,
#### engineering informative features like balance errors, and balancing the model's focus using class_weight.
#### The model can now be used for proactive fraud detection in financial transactions with high confidence.

# Model Performance Evaluation

### After training the Random Forest model, I used multiple evaluation tools to check its performance — 
### especially because this is a highly imbalanced classification problem.

## 1. Confusion Matrix
### I used sklearn’s confusion_matrix and seaborn heatmap to visualize it clearly.
### The matrix showed almost perfect classification:
### - True Positives (frauds correctly caught) were very high
### - False Negatives (missed frauds) were almost zero
### - False Positives (false alarms) were also near zero
### This confirmed that the model was not just guessing, and was learning useful patterns.

## 2. Classification Report
### Used classification_report to get precision, recall, and F1-score for both classes.
### For class 1 (fraud), the recall and precision were both near 1.0 — which is exactly what we want.
### High recall means the model is catching almost all frauds.
### High precision means it’s not raising too many false alarms.

## 3. ROC AUC Score
### Also calculated the ROC AUC score using roc_auc_score and the model’s predicted probabilities.
### This metric is important for imbalanced problems because it shows how well the model separates frauds from normal transactions.
### My ROC AUC was close to 1.0, which confirms strong performance.

## 4. Feature Importance
### Used the RandomForest’s feature_importances_ to see which features were driving predictions.
### The engineered features (errorBalanceOrig, errorBalanceDest) were among the top — which matched the logic of fraud behavior.

### Overall, the model was evaluated using the right tools for both imbalanced data and explainability,
### and the results showed it’s not only accurate but also reliable for fraud detection in real scenarios.

# Prevention Strategies While Updating Infrastructure

## 1. Real-time fraud detection
### The model I built should be deployed as part of a real-time monitoring system.
### This means every transaction should be passed through the model before it's processed,
### especially for high-risk types like TRANSFER and CASH_OUT.

## 2. Threshold-based transaction limits
### Introduce dynamic limits based on user behavior. For example, if a user usually transfers ₹5K per week,
### suddenly transferring ₹2L should trigger an alert or require extra verification (OTP, biometric, etc.).

## 3. Flag inconsistent balance behavior
### Use features like errorBalanceOrig and errorBalanceDest in production to flag any balance mismatch in real time.
### Transactions with negative or suspicious balance shifts can be auto-paused for review.

## 4. Track merchant transactions separately
### Since merchant accounts don’t show balance info, apply stricter rules or flag large merchant transactions
### for manual verification — especially if they don’t match the customer’s usual pattern.

## 5. Audit logs and version control
### While updating infrastructure, keep all model versions, prediction logs, and transaction flows well-logged and auditable.
### This helps track any errors during deployment and makes it easier to revert if needed.

## 6. Role-based access & API rate limits
### Make sure internal systems and APIs are secure — no single user or system should be able to trigger unlimited transfers.
### Add role-based access, two-factor authentication, and request throttling.

### These steps combined can significantly reduce fraud risk while scaling up or upgrading systems.

# How to Measure If Fraud Prevention Actions Are Working

## 1. Compare Fraud Rate (Before vs After)
### Track the percentage of fraudulent transactions over time.
### If fraud rate drops after implementing the new model and prevention steps, that’s a clear sign it's working.

## 2. A/B Testing (if possible)
### Run the new system for a portion of users or transactions and compare results with the old one.
### Helps isolate the actual impact of the prevention measures.
