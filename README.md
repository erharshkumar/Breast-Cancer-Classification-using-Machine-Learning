## Project Summary :-
- Breast cancer diagnosis is one of those problems where the cost of being wrong is not measured in points of accuracy, it’s measured in lives. That’s why I was drawn to this project. I wanted to build something that was not only a textbook classification task, but a model that acted responsibly in a domain where a false negative (calling a malignant tumor benign) is far more dangerous than a false positive.

- - **Problem:** Classify breast tumors as malignant or benign from diagnostic imaging measurements
- **Approach:** Compared four ML algorithms through proper cross-validation rather than committing to one model upfront
- **Outcome:** 98.25% test accuracy, 0.997 ROC-AUC, with 95% malignant recall to minimize the most dangerous error type
- **Why it matters:** Demonstrates a complete, leakage-free ML workflow built around the right evaluation priorities for a sensitive domain

### Objective
The goal I set for myself going in was easy to state but harder to get right: build a binary classifier for tumor malignancy and build it the way I would want a model that would be used in a clinical-adjacent setting to be built.

 **What I promised**
- **Create an end-to-end pipeline:** From loading data to evaluation with a saved, reusable model - not just a notebook ending in a single accuracy score
- **Compare, don't assume**. Try a few algorithms under the same conditions, don't just take the model I tried first
- **No data leakage:** All preprocessing must be strictly inside cross-validation folds and the final pipeline
- **Optimize for the right metric:** Optimize for catching malignant cases, not just overall accuracy, as there is an asymmetric cost of errors in this domain

  ### Dataset Overview
  I used the Breast Cancer Wisconsin (Diagnostic) dataset from scikit-learn's `load_breast_cancer()`. This dataset is a well known benchmark based on real diagnostic imaging data.

**Dataset Information**
scikit-learn built-in dataset (Breast Cancer Wisconsin (Diagnostic))
- **Samples:** 569 patient records 
- **Features:** 30 numeric features (mean, standard error and "worst" or largest mean value over 10 cell nuclei features: radius, texture, perimeter, smoothness, concavity and more)
- **Target classes:** Benign vs Malignant
- **Class balance:** Approximately 63% benign / 37% malignant
- **Data quality:** verified no missing values in all 569 rows of data so no imputation was needed
- **Label encoding callout:** `0` = Malignant, `1` = Benign, opposite of what you'd expect, so I kept this explicit all the way through to avoid any mix-ups downstream

### Project Workflow
I didn’t just jump into model fitting but approached it as a sequence of deliberate phases:

1. **Data Loading and Inspection** - Loaded the dataset into a pandas DataFrame and confirmed that there were zero missing values so I could skip imputation and focus on modeling decisions.
2. **Exploratory analysis** - Visualized class balance, created a correlation heat map across the “mean” feature group, and identified the individual features that correlated most strongly with diagnosis (mean concave points and mean radius jumped out early).
3. **Train-test split** - Split the data 80/20 with stratified sampling on the target so that the ratio of malignant to benign samples is similar in both sets.
4. **Baseline model** - Started with a simple Logistic Regression model for a baseline: 96.9% training accuracy, 93% test accuracy.
5. **Cross-Validation for model comparison** - Encapsulated four potential models (Logistic Regression, Random Forest, SVM, Gradient Boosting) using pipelines with `StandardScaler` included within each fold, but not outside of it in order to prevent data leakage. All four were evaluated on five-fold stratified CV in terms of accuracy, precision, recall, F1 and ROC-AUC metrics.
6. **Selection of final model and fitting** - Model with the highest ROC-AUC score in five-fold stratified CV was the SVM model with an RBF kernel, so I fit that one as the final pipeline.
7. **Evaluation** - Created classification report, confusion matrix, and ROC curve in order to evaluate the model performance on test set.
8. **Interpretable** - Permutation feature importance was used instead of coefficient values, because it is model agnostic and would still be relevant even when a different model is chosen at the end.
9. **Deployment** - Combined all components into a single `predict_tumor()` function and saved the whole pipeline (both scaler and the model) with joblib, meaning that the deployed artifact does its work right out of the box.

### Key Findings

**Model Performance**
- SVM (RBF) outperformed Logistic Regression, Random Forest, and Gradient Boosting on cross-validated ROC-AUC (0.9957)
- Final test performance: 98.25% accuracy, 0.997 ROC-AUC

**Clinical Safety**
- 95% recall on malignant cases - the model rarely misses a true malignant tumor
- 97% precision on benign cases - high confidence when predicting benign
- Confirms the model errs on the side of caution rather than missing dangerous cases

**Feature Importance**
- Permutation importance identified cell concavity, perimeter, and radius measurements as most predictive of malignancy
- Aligns with clinical intuition: more irregular, larger cell structures associate with malignant growth

**Methodology Validation**
- Scaling features inside the cross-validation pipeline (rather than before the split) was essential to getting a trustworthy generalization estimate
- An easy mistake to make that would have quietly inflated reported numbers if not handled carefully

### Skills Demonstrated

**Data Analysis**
- Exploratory Data Analysis (EDA)
- Statistical Summarization (descriptive stats, class distribution)
- Correlation Analysis (feature-to-feature and feature-to-target)

**Data Preprocessing**
- Feature Scaling (StandardScaler)
- Stratified Train-Test Splitting
- Data Leakage Prevention (scaling fit inside each CV fold via Pipeline)

**Machine Learning**
- Classification Algorithms (Logistic Regression, Random Forest, SVM, Gradient Boosting)
- Model Selection (comparative evaluation across 4 algorithms)
- Cross-Validation (5-fold Stratified K-Fold)
- Pipeline Architecture (scikit-learn Pipeline for reproducible preprocessing + modeling)

**Model Evaluation**
- Accuracy Analysis
- Precision & Recall Trade-off Optimization
- ROC-AUC Evaluation
- Confusion Matrix Interpretation
- Model Interpretability (Permutation Importance)

**Model Deployment**
- Model Serialization (joblib)
- Predictive Function Design (single reusable inference wrapper)

**Python Libraries**
- Pandas
- NumPy
- Matplotlib
- Seaborn
- Scikit-Learn
- Joblib


## Conclusion
The most important lesson from this exercise is that solid machine learning in a delicate field has less to do with getting the best accuracy result and more to do with being deliberate throughout the process so that the result actually has meaning. Through rigorous evaluation of the models and avoiding data leakage, but most importantly through error analysis based on error cost and not on frequency, I arrived at a model I would trust to explain, not just implement.
