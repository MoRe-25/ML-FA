# Beyond Response Prediction: Contact-History-Aware Customer Targeting

## Project overview

This project studies whether campaign-contact history improves bank marketing subscription prediction and whether that information changes which customers a bank would prioritize for outreach.

Using the Bank Marketing dataset, we compare a customer-profile model with a model that also includes campaign-history features. We then examine how the additional information changes customer rankings and explore an illustrative restriction on repeated contact.

This is an empirical student research extension of existing bank-marketing prediction research. It does **not** claim that contact-history modeling or these evaluation methods are new.

## Research questions

1. **Does contact history improve prediction?**  
   Compare a model using customer-profile features with one using profile and campaign-history features.

2. **Does contact history change customer prioritization?**  
   Compare ranked customer lists and the number of actual subscribers captured at different targeting budgets.

3. **What is the trade-off when repeated contact is restricted?**  
   Compare unrestricted ranking with an illustrative strategy that excludes records whose recorded campaign count is greater than two.

## Dataset

- **Records:** 45,211
- **Columns:** 17
- **Positive outcomes (`y = yes`):** 5,289
- **Negative outcomes (`y = no`):** 39,922
- **Overall positive rate:** 11.70%

The target variable is `y`, indicating whether the customer subscribed to the term deposit.

The analysis distinguishes between:
- **Customer-profile features:** customer and account attributes.
- **Campaign-history features:** recorded contact and previous-campaign information, including `campaign`, `pdays`, `previous`, and `poutcome`.

The `duration` feature was excluded from the predictive feature sets. Call duration is generally only known after a call has taken place, so including it may not fit a targeting decision that must be made before contacting a customer.

## Methodology

### 1. Model comparison

Four model families were considered in the initial benchmark:
- Logistic Regression
- Decision Tree
- Linear Support Vector Machine (SVM)
- Neural Network

Logistic Regression was used for the main feature-group experiments so that the comparison focused on the change from adding campaign-history features, rather than changing both the features and model at once.

### 2. Feature-group experiment

Two models were compared:

- **Model A — Customer profile:** profile features only.
- **Model B — Profile + campaign history:** profile features plus recorded campaign-history features.

The reported metrics are ROC-AUC and PR-AUC. Paired bootstrap resampling was used to estimate confidence intervals for the performance differences on the evaluation test set.

### 3. Customer-ranking experiment

Customers were ranked by each model's predicted subscription probability. We compared the top 10%, 20%, and 50% of each ranking, including:
- Number of actual subscribers captured
- Overlap between selected customer lists
- Rank correlation and the size of rank changes

### 4. Contact-frequency restriction experiment

We compared:
- **Strategy A — Unrestricted:** rank all customers.
- **Strategy B — Restricted:** exclude customers whose recorded `campaign` count is greater than two, then select from the remaining eligible customers.

This restriction is an illustrative retrospective rule, not a recommended operational policy. The dataset's `campaign` value represents the number of contacts during the current campaign, including the contact associated with the record; it is not a verified measure of every contact the customer has ever received.

### 5. Segment sensitivity analysis

The evaluation records were divided into four groups based on their original row positions, and ROC-AUC was calculated for both feature sets in each group.

**Important limitation:** Original row position is not a verified timestamp. These groups should not be interpreted as calendar periods or as a substitute for proper chronological validation.

## Results

### Experiment 1 — Prediction performance

| Metric | Model A: Profile | Model B: Profile + history | Difference |
|---|---:|---:|---:|
| ROC-AUC | 0.6661 | 0.7717 | +0.1056 |
| PR-AUC | 0.2135 | 0.4111 | +0.1976 |

Paired bootstrap confidence intervals (2,000 valid resamples):

| Difference | Estimate | 95% CI |
|---|---:|---:|
| ROC-AUC difference | +0.1056 | 0.0905–0.1203 |
| PR-AUC difference | +0.1976 | 0.1692–0.2258 |

Both confidence intervals are above zero, supporting a positive performance difference between these two models on this evaluation test set. This does not guarantee the same improvement on future campaigns or other banks.

### Experiment 2 — Customer prioritization

| Targeting budget | Model A subscribers captured | Model B subscribers captured | List overlap |
|---|---:|---:|---:|
| Top 10% (905 customers) | 246 | 447 | 38.6% |
| Top 20% (1,809 customers) | 407 | 589 | 56.8% |
| Top 50% (4,522 customers) | 740 | 848 | 75.1% |

Additional ranking results:
- Spearman correlation between rankings: **0.6454**
- Median absolute rank change: **1,372 positions**

Adding campaign-history features changed the ranking of customers substantially in this experiment. At the top-10% targeting budget, Model B captured 201 more actual subscribers than Model A in the evaluation data.

### Experiment 3 — Contact-frequency restriction

| Targeting budget | Strategy A: Unrestricted | Strategy B: Restricted |
|---|---:|---:|
| Top 10% | 447 | 403 |
| Top 20% | 589 | 534 |
| Top 50% | 848 | 743 |

Selection-list overlap between the two strategies was 82.4% at the top 10%, 82.5% at the top 20%, and 74.5% at the top 50%.

Under this illustrative rule, the restricted strategy captured fewer historical subscribers at each reported budget. However, the experiment did not measure customer fatigue, complaints, opt-outs, customer satisfaction, or long-term relationship outcomes. Therefore, it cannot establish which strategy is better overall.

### Supporting segment sensitivity analysis

| Record-position segment | Test records | Positive rate | Profile ROC-AUC | Profile + history ROC-AUC |
|---|---:|---:|---:|---:|
| Rows 0–11,299 | 2,274 | 3.39% | 0.5180 | 0.5454 |
| Rows 11,300–22,599 | 2,270 | 5.24% | 0.5306 | 0.5573 |
| Rows 22,600–33,899 | 2,259 | 9.56% | 0.6289 | 0.7063 |
| Rows 33,900–end | 2,240 | 28.84% | 0.6679 | 0.7436 |

The history-aware model had higher ROC-AUC in all four groups. Because the groups are based on record positions rather than verified dates, this is exploratory segment sensitivity analysis—not chronological validation.

## Main conclusions

1. Adding campaign-history features improved ROC-AUC and PR-AUC in the evaluated test set.
2. The extra information changed customer rankings and increased the number of actual subscribers captured in the selected top-ranked groups.
3. Restricting customers based on recorded campaign count reduced the number of historical subscribers captured at equal reported targeting budgets.
4. Bootstrap intervals and segment sensitivity analysis support the observed comparison, but they do not establish causality or guarantee future performance.

## Limitations

- The experiments use one dataset and may not generalize to other banks, markets, or future campaigns.
- This is retrospective observational analysis; it does not establish that contacting a particular customer causes a subscription.
- The contact-frequency restriction is an illustrative rule. The analysis does not evaluate customer well-being or long-term business outcomes.
- Original record positions are not verified timestamps, so segment results must not be presented as chronological validation.
- Predictive ranking alone is not a complete contact-allocation policy. A real deployment would also need operational costs, contact limits, customer preferences, compliance requirements, and prospective evaluation.

## Reproducibility

Run the analysis notebook(s) in the repository in the order provided. Ensure the Bank Marketing dataset is available at the file path expected by the notebook, and install the Python packages imported by the notebook before running it.

The exact notebook filenames, dependency versions, and dataset path should be documented here once the repository structure is finalized. Do not treat the reported values above as independently reproducible until the corresponding notebook cells and data split have been checked.

## Reference

Moro, S., Cortez, P., & Rita, P. (2014). *A Data-Driven Approach to Predict the Success of Bank Telemarketing*. Decision Support Systems, 62, 22–31. https://doi.org/10.1016/j.dss.2014.03.001

## Disclaimer

This repository is intended for academic and educational purposes. Results are specific to the dataset and experimental setup described above and should not be used as a standalone basis for real customer-contact decisions.
