# A Data-Driven Approach to Supporting Autism Screening in Adults

**Author:** Jasmine Baldwin
**Course:** Statistics 6302
**Date:** May 2025

## Overview

Autism spectrum disorder (ASD) diagnoses in adults are frequently delayed by limited screening resources and lengthy, multi-session assessment protocols. This project explores whether classical statistical models can support — not replace — the clinical screening pathway by flagging adults who may benefit from a full diagnostic evaluation.

Using the **Autistic Spectrum Disorder Screening Data for Adults** dataset (704 observations, 21 variables), this project builds and compares several logistic regression models to predict the likelihood of an autism diagnosis based on AQ-10 survey responses and demographic factors.

**Research question:** *Can a logistic regression model accurately predict the likelihood of an autism diagnosis in adults based on demographic factors and survey responses?*

## Dataset

The dataset includes:
- 10 binary item-level responses to the **AQ-10** (Autism Spectrum Quotient, 10-item version) screening survey
- A total AQ-10 score (sum of the 10 items, max score = 10, clinical referral threshold = 6)
- Demographics: age, gender, ethnicity, country of residence
- Medical/contextual history: jaundice at birth, prior app usage, relation of survey respondent to subject
- The app's own AQ-10-based classification of the subject
- Outcome variable: clinician-relevant autism diagnosis (Yes/No)

See `Appendix A` and `Appendix B` in the full report for complete variable definitions and AQ-10 item wording/scoring rules.

## Methods

### Exploratory Data Analysis
- Examined distributions of all 10 survey items, total score, age, gender, ethnicity, country of residence, jaundice history, respondent relation, and app usage history
- Identified and corrected a data entry error in `age` (a value of 383 corrected to 38)
- Assessed missingness: `age` (2 missing, mean-imputed), `ethnicity` and `relation` (95 missing each, mode-imputed)
- Removed `age_desc` (zero variance), `used_app_before` (not predictive), and the standalone `result`/total-score variable (perfectly collinear with the 10 individual item scores)
- Quantified class imbalance (no-information rate ≈ 87.1%) and addressed it via up-sampling in later models
- Explored correlations among numeric variables and associations between categorical predictors and diagnosis
- Systematically tested all pairwise interaction terms via likelihood ratio tests, then visually inspected significant pairs to prioritize candidates for modeling

### Models

| Model | Description |
|---|---|
| **Base Model** | Logistic regression using all available predictors (survey items, demographics, jaundice history, respondent relation, app classification), dummy-encoded. Evaluated with repeated 10-fold cross-validation (5 repeats), stratified by outcome. |
| **Elastic Net Model** | Extends the base model with elastic net regularization and up-sampling to address class imbalance. |
| **Custom Model 1** | Extends the base model with two theory-driven interaction terms (ethnicity × screening result; gender × jaundice) plus up-sampling. |
| **Custom Model 2** | Penalized logistic regression with elastic net regularization, where both the penalty strength and the mixture (lasso/ridge balance) were tuned via a space-filling design. Only predictors with non-zero coefficients were retained in the final model. |

All models were evaluated using accuracy, ROC AUC, sensitivity, specificity, Cohen's Kappa, and Brier score.

## Key Results

| Model | Accuracy | AUC | Sensitivity | Specificity | Kappa | Brier |
|---|---|---|---|---|---|---|
| Base Model | 84.6% | 0.62 | 13.9% | 95.1% | 0.12 | 0.124 |
| Elastic Net | 68.1% | 0.71 | 64.0% | 68.6% | — | — |
| Custom Model 1 | — | 0.608 | 52.8% | 72.1% | — | — |
| Custom Model 2 | 68.3% | 0.71 | 62.5% | 69.2% | 0.18 | 0.20 |

- The **base model** had very high specificity but very poor sensitivity, reflecting how strong class imbalance can mask weak true-positive detection even at high overall accuracy.
- The **elastic net model** offered the most balanced sensitivity/specificity trade-off and the highest discriminative ability (AUC ≈ 0.71) among the models tested.
- **Custom Model 2** achieved comparable balanced performance while also surfacing interpretable, clinically/literature-supported predictors (e.g., AQ-10 item responses, neonatal jaundice history) among its top positive predictors, after regularization-based feature selection.
- Across models, country-of-residence and ethnicity coefficients were often the largest in magnitude, but these likely reflect small subgroup sample sizes and demographic imbalance in the dataset rather than robust effects.

## Conclusions and Limitations

The elastic net model emerged as the most balanced and statistically reliable of the four, but **Custom Model 2** is recommended as the more practically useful screening aid, given its more interpretable, clinically grounded top predictors and its handling of multicollinearity through regularization. That said, this recommendation comes with important caveats:

- The dataset is self-reported, demographically imbalanced (particularly by country and ethnicity), and may not generalize well to broader populations.
- Up-sampling to address class imbalance introduces some risk of overfitting due to repeated rows.
- Several of the strongest model coefficients are likely artifacts of small subgroup sizes rather than genuine effects.
- No model in this project should be used as a diagnostic tool. At best, these models could support a **preliminary screening aid** to help flag individuals for further clinical evaluation — never as a substitute for a formal assessment by a licensed clinician.

Future work should focus on acquiring more representative and diverse data, validating results on independent samples, and carefully assessing the ethical implications of using demographic variables (e.g., ethnicity, country of residence) as predictors in any screening context.

## Tools & Methods Used

- **R** with `tidymodels` for model specification, resampling, and tuning
- Logistic regression, elastic net regularization, up-sampling for class imbalance
- Repeated 10-fold cross-validation (5 repeats), stratified by outcome
- Likelihood ratio tests for interaction term selection
- Space-filling design for hyperparameter tuning (penalty and mixture)

## References

- Ally Pediatric Therapy. "How Long Is an Autism Evaluation at a Center?" *Ally Pediatric Therapy*, 19 July 2024.
- Amin SB, Smith T, Wang H. "Is neonatal jaundice associated with Autism Spectrum Disorders: a systematic review." *J Autism Dev Disord*, 2011 Nov;41(11):1455-63.
- Armstrong, Bonnie. "What Is Autism Spectrum Disorder (ASD)?" *The Key School*, 26 Apr. 2024.
- "The AQ-10." *Embrace Autism*, 23 Apr. 2025.

## Disclaimer

This project is for academic and research purposes only. None of the models presented here constitute a diagnostic tool, and results should not be used to make clinical decisions. Any individual with concerns about autism should consult a licensed clinician for a formal evaluation.
