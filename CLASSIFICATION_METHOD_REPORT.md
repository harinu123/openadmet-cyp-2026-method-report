Prediction of CYP2D6 and CYP3A4 Time-Dependent Inhibition Using Molecular and Assay-Informed Ensembles
Participant: harinu123  
Challenge: OpenADMET CYP Inhibition Blind Challenge  
Track: Time-dependent inhibition classification  
Primary metric: Macro-averaged Matthews correlation coefficient (MA-MCC)  
Proprietary data used: No  
Public code release: No
Abstract
We developed endpoint-specific ensemble models to predict time-dependent inhibition (TDI) of CYP2D6 and CYP3A4 using publicly available challenge data and pretrained molecular representations. The CYP3A4 classifier combined a molecular-embedding-based tabular model with a LightGBM ranking model. The CYP2D6 classifier combined a molecular and multi-assay predictor with a second-stage ensemble incorporating parent-compound measurements, analogue-series information, and estimates of assay evaluability and TDI outcome. Model selection used validation splits that separated complete parent/analogue families, with an additional confirmation cohort reserved until the ensemble weight and classification operating point had been fixed. Adding the second-stage CYP2D6 model increased MCC by a mean of 0.1102 across seven development cohorts and by 0.0706 in the confirmation cohort. On the public leaderboard, the submitted predictions achieved an MA-MCC of 0.4474, with endpoint MCCs of 0.3794 for CYP2D6 and 0.5154 for CYP3A4. The submission contained predictions for 750 compounds, including 50 CYP2D6-positive and 224 CYP3A4-positive calls.
1. Prediction Task
The objective was to predict the organizer-defined binary TDI outcomes for CYP2D6 and CYP3A4. The challenge uses a greater-than-two-fold IC50 shift between direct-inhibition and preincubation conditions, together with additional rules for inferred outcomes when a shift is not directly measurable. Predictions are required for all 750 test compounds, while scoring uses outcomes that the organizers can assign with sufficient confidence.
Separate classifiers were constructed for the two isoforms. The models produced continuous ranking scores, which were converted to binary predictions using endpoint-specific numbers of positive calls. The primary evaluation metric was the mean of the two endpoint-specific Matthews correlation coefficients.
The test compounds were obtained through chemical analogue expansion around selected parent compounds. We therefore incorporated information about parent compounds and their analogue series, and used complete parent/analogue families as the principal unit of separation during validation. Here, a parent compound refers to the compound around which an analogue series was constructed.
2. Data and Molecular Representations
2.1 Training Information
Training used the public challenge releases, including official TDI labels, direct-inhibition and TDI-condition pIC50 measurements, pIC50 shifts, maximal-response measurements (Emax), single-concentration screening readouts, and indicators of assay advancement and evaluability. The pIC50 scale is defined as the negative base-10 logarithm of IC50 expressed in molar units. Assay evaluability denotes whether an observation supports an outcome eligible for scoring.
The CYP2D6 development population contained 4,816 proxy observations, including 1,122 observations with official TDI labels: 239 positive and 883 negative. The second-stage classifier used 58 features available for prediction. The larger proxy-population count should therefore be distinguished from the number of officially labelled observations.
No proprietary assay data, blinded test labels, private experimental outcomes, or structure-track ground truth were used.
2.2 Molecular and Assay Features
The molecular representations included pretrained Chemprop and CheMeleon embeddings, Morgan fingerprints, and nearest-neighbour summaries. These representations supported predictions of TDI outcome, assay advancement, and assay evaluability. A Chemprop multitask model additionally predicted 28 related assay targets, with missing target values masked during training.
The 58-feature CYP2D6 second-stage model combined the following information:
Feature group	Information represented
Base-model predictions	Precomputed ranks from molecular and assay predictors
Molecular and analogue evidence	Embedding-, fingerprint-, and nearest-neighbour-based summaries of TDI outcome and assay evaluability
Parent-compound measurements	Publicly available parent TDI outcomes, direct-inhibition potency, TDI-condition potency, and potency shifts
Parent and analogue relationships	Query-parent similarity and parent/analogue count features
Cross-isoform information	Assay measurements and selection indicators across CYP isoforms
Within-series summaries	Means, maxima, standard deviations, and ranks within analogue series
Auxiliary predictions	Separately cross-fitted predictions of assay evaluability and conditional TDI outcome
Parent and series features were derived from information available at prediction time. They did not use blinded experimental outcomes for the test compounds.
3. Predictive Models
3.1 Rank Aggregation and Decision Rules
The ensembles combined model rankings to reduce sensitivity to differences in numerical score calibration. Let $R(s)$ denote the rank transform of a score $s$ within the relevant prediction population, oriented so that larger values indicate stronger evidence of a positive TDI outcome. Weighted combinations of these ranks defined the endpoint-specific prediction scores.
Binary predictions were obtained by selecting the highest-scoring compounds at fixed operating points: 50 positive calls for CYP2D6 and 224 for CYP3A4 among the 750 test compounds. These call counts were treated as model parameters and evaluated through grouped validation. The ranking scores were not interpreted as calibrated probabilities.
3.2 CYP3A4 Classifier
For CYP3A4, Chemprop molecular embeddings were reduced to 64 dimensions using principal component analysis and supplied to TabICL, a tabular foundation model for in-context learning. Predictions from three random seeds were combined using the mean minus one standard deviation, penalizing disagreement across seeds. This score was rank-aggregated with the output of a LightGBM ranking model.
The CYP3A4 ensemble score was $S_{\mathrm{3A4}}(x) = 0.75,R(\mu_{\mathrm{TabICL}}(x)-\sigma_{\mathrm{TabICL}}(x)) + 0.25,R(s_{\mathrm{LightGBM}}(x))$.
Here, $\mu_{\mathrm{TabICL}}$ and $\sigma_{\mathrm{TabICL}}$ denote the mean and standard deviation across the three TabICL predictions. The function $R$ transforms each model's score into a rank prior to combination.
Six-fold rank-based validation selected a pooled evaluated-call fraction of 29.855%, yielding 224 positive calls in the submission. The CYP3A4 prediction vector was held fixed during subsequent CYP2D6 model selection.
3.3 CYP2D6 Molecular and Multi-Assay Predictor
The CYP2D6 reference predictor combined molecular prediction ranks with auxiliary models of the assay-selection process and a multitask predictor of the continuous measurements underlying the TDI label. A base molecular rank and an assay-selection rank were combined with weights of 0.905 and 0.095, respectively. The assay-selection models used Chemprop and CheMeleon representations and contributed a conservative score based on lower-confidence-bound ranking. A further 5% rank contribution was introduced from the multitask predictor.
The multitask component jointly predicted assay quantities relevant to TDI and constructed a score from the smaller of two standardized margins:
$m(x) = \min\left(\frac{\widehat{\Delta\mathrm{pIC50}}(x)-\log_{10}(2)}{0.25},\ \frac{\widehat{\mathrm{pIC50}}_{\mathrm{TDI}}(x)-4.30}{0.35}\right)$.
The first term measures the predicted potency shift relative to the two-fold threshold. The second measures predicted potency in the TDI condition relative to a pIC50 of 4.30. Taking the minimum requires support from both quantities for the margin to be positive. The constants 0.25 and 0.35 scale the two contributions; this margin is a model score, not a replacement for the organizers' complete labelling procedure.
The Chemprop score used three random seeds with a frozen encoder, one model with the final message-passing block fine-tuned, and one model with the full encoder fine-tuned at a low learning rate. Each training configuration was ensembled across six folds defined by chemical groups. A lower-confidence-bound summary of these predictions contributed to the reference rank. The CYP2D6 reference predictor used the same 50-positive-call operating point as the final ensemble.
3.4 CYP2D6 Parent-Series Classifier
A second-stage classifier used the 58 features described in Section 2.2 to identify likely negative outcomes among highly ranked CYP2D6 candidates. It comprised six LightGBM models, six ExtraTrees models, and one regularized, class-balanced logistic-regression model. The tree models used six random seeds. Predictions were summarized at the model-family level, and the three family-level rankings were averaged to form a negative-outcome consensus. This consensus was inverted to obtain a positive-outcome score.
Training weights emphasized observations near the classification boundary. Labelled observations in the reference predictor's top 50 received weight 4, observations ranked 51-100 received weight 2, and the remaining labelled observations received weight 1. This weighting increased emphasis on candidates capable of changing the selected positive set while retaining information from the wider labelled population.
The final CYP2D6 ensemble score was $S_{\mathrm{2D6}}(x) = 0.60,R(s_{\mathrm{mol}}(x)) + 0.40,R(s_{\mathrm{series}}(x))$.
Here, $s_{\mathrm{mol}}$ is the molecular and multi-assay reference predictor described in Section 3.3, and $s_{\mathrm{series}}$ is the positive-outcome score from the parent-series classifier. The 50 highest-scoring compounds were classified as positive.
4. Validation and Model Selection
4.1 Separation of Related Compounds
Validation was designed to limit information sharing between related observations. Each outer training partition excluded the held-out cohort and every observation sharing a parent molecule with that cohort. Preprocessing and model fitting were repeated within the corresponding training partition.
Seven cohorts were used for model selection. A separate confirmation cohort was excluded from model selection and evaluated once after the ensemble weight and positive-call count had been fixed. Auxiliary predictions of assay evaluability and conditional outcome were generated by separate cross-fitting procedures.
4.2 Selection and Robustness Analysis
The CYP2D6 parent-series classifier was evaluated against the molecular and multi-assay reference predictor at the same 50-positive-call operating point. This comparison assessed the effect of re-ranking while holding the number of positive predictions constant.
After the model specification was fixed, a separate audit repeated selection using nested cohort exclusions. The audit did not read submitted prediction files, blinded labels, or leaderboard scores. Uncertainty in the observed improvement was also assessed using 20,000 bootstrap replicates that jointly resampled cohorts and parent families.
5. Results
5.1 Internal Validation
Adding the parent-series classifier improved CYP2D6 MCC in six of the seven development cohorts and did not reduce MCC in any cohort.
Comparison with the molecular and multi-assay reference predictor	Result
Mean MCC change across development cohorts	+0.1102
Lowest cohort-level MCC change	0.0000
Cohorts with improved MCC	6/7
Cohorts with no reduction in MCC	7/7
MCC change in the reserved confirmation cohort	+0.0706
The robustness audit selected the 40% parent-series contribution in all seven leave-one-cohort analyses and all 21 leave-two-cohort analyses. Across scored prediction changes in development evaluations, 50 errors were corrected and 25 previously correct predictions became incorrect. The corresponding counts in the confirmation cohort were four and one.
The joint cohort/parent-family bootstrap estimated a mean MCC improvement of 0.1087, with a descriptive 95% interval of 0.0033-0.2239. Because the broader development process evaluated multiple hypotheses, this interval summarizes observed robustness and should not be interpreted as a prespecified confirmatory inference.
5.2 Prediction Stability
Relative to the molecular and multi-assay reference predictor, the final CYP2D6 ensemble changed eight test-set classifications: four compounds entered the positive set and four left it. Six independent final-model seed runs produced the same set of 50 positive predictions. CYP3A4 predictions remained unchanged.
5.3 Public-Leaderboard Performance
The submission passed the official format validator and contained 750 unique molecule identifiers. The public-leaderboard results recorded after submission on 31 October 2026 were:
Endpoint	Positive predictions in the full 750-compound submission	Public-leaderboard MCC
CYP2D6	50	0.3794
CYP3A4	224	0.5154
Macro-average	-	0.4474
The live leaderboard evaluates one half of the test set, with parent/analogue series kept together within that split. These scores therefore describe performance on the public subset, rather than the final full-test evaluation.
6. Use of Leaderboard Feedback
During initial development, public feedback was used to diagnose an excessive CYP3A4 positive-call rate under an absolute-threshold rule and to restore a previously evaluated CYP2D6 prediction set. Subsequent operating points were expressed as rank-based call counts, and proposed changes were assessed through grouped validation.
For the final CYP2D6 parent-series ensemble, leaderboard metrics were not used to select features, model families, the 40% blending weight, or the 50-positive-call operating point. The reported public score was observed after the specification had been fixed and did not trigger changes to the submitted predictions. Blinded test labels were never available during model development.
7. Discussion and Limitations
The internal validation results support the use of parent-compound and analogue-series information to refine CYP2D6 rankings in this challenge setting. The improvement was observed at a fixed number of positive calls, indicating that the second-stage classifier changed which compounds were selected, rather than increasing the number selected. The confirmation-cohort result and repeated cohort-exclusion analyses provide additional evidence that the benefit was not confined to a single development split.
Several limitations affect interpretation. First, TDI labels depend on assay conditions, censoring, and uncertainty near the two-fold shift threshold. Second, the validation cohorts were retrospective approximations to the challenge's analogue-expansion design and do not constitute an independent prospective experimental evaluation. Third, fixed-count classification assumes that an operating point selected during development remains appropriate for the evaluation population. A change in class prevalence or compound-selection strategy could require recalibration.
Parent-series information may be particularly useful when related compounds have existing assay measurements, but its contribution may diminish for chemically unrelated compounds or series without measured parents. The observed seed stability demonstrates consistency of the submitted positive set under the tested random seeds; it does not establish calibrated uncertainty for individual predictions. Finally, public-leaderboard performance and descriptive bootstrap intervals should be interpreted in the context of partial test-set evaluation and an adaptive development process.
8. Data, Code, and Reproducibility
The implementation used Python, pandas, NumPy, SciPy, scikit-learn, LightGBM, RDKit-derived molecular features, Chemprop-family representations, and TabICL. The final tree ensembles used random seeds 20201-20206, with deterministic stable ranking. Prediction artifacts were checksummed before submission.
No proprietary dataset was used. The implementation has not been publicly released. Model binaries and complete feature tables are retained by the participant and can be supplied to the organizers on request.
9. Conclusion
The submitted approach combined endpoint-specific rank aggregation, multi-assay molecular prediction, and parent-series modelling for CYP TDI classification. Parent-series re-ranking improved CYP2D6 performance across the available development cohorts and the reserved confirmation cohort at a fixed positive-call count. The submitted predictions achieved a public-leaderboard MA-MCC of 0.4474. Evaluation on the complete blinded test set is needed to determine how these results extend beyond the public subset.
References
OpenADMET. Announcing OpenADMET's CYP Inhibition Blind Challenge.
OpenADMET. CYP Challenge Tutorial.
OpenADMET. Halfway Through the OpenADMET CYP Challenge.
OpenADMET. CYP Blind Challenge Platform.
Heid, E., et al. Chemprop: A Machine Learning Package for Chemical Property Prediction. Journal of Chemical Information and Modeling 64(1), 9-17 (2024).
Burns, J. W., et al. Deep Learning Foundation Models from Classical Molecular Descriptors. arXiv, 2506.15792 (2025).
Qu, J., Holzmüller, D., Varoquaux, G., and Le Morvan, M. TabICL: A Tabular Foundation Model for In-Context Learning on Large Data. Proceedings of Machine Learning Research 267, 50817-50847 (2025).
