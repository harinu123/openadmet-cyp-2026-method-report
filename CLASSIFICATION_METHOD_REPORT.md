# Final method report: CYP time-dependent-inhibition classification

**Participant:** harinu123  
**Final model:** v204  
**Track:** Classification Prediction  
**Endpoints:** CYP2D6 and CYP3A4 time-dependent inhibition (TDI)  
**Primary metric:** macro-averaged Matthews correlation coefficient (MA-MCC)  
**Proprietary data:** No  
**Open-code claim:** No

## 1. Summary

The final classifier is a deliberately conservative endpoint-specific rank
ensemble. CYP3A4 is frozen from the earlier validated v119 lineage. CYP2D6 uses
a fixed 50-call operating point and combines the v119 molecular/multi-assay
rank with a parent-series-aware outcome meta-head:

```text
CYP2D6 score = 0.60 × rank(v119)
             + 0.40 × rank(parent-series outcome meta-head)
prediction   = positive for the top 50 of 750 compounds
```

The meta-head averages six LightGBM models, six ExtraTrees models, and one
regularized logistic regression over 58 deployable assay, molecular, analogue,
and parent-series features. The final CYP2D6 vector contains 50 positive calls;
the frozen CYP3A4 vector contains 224.

The final policy, blend weight, and call count were frozen without blinded
labels and without using leaderboard values as a selection objective. The live
leaderboard was used only for reporting after submission. An earlier model in
the lineage did use leaderboard feedback to diagnose an overly aggressive
operating point; this is disclosed in Section 7.

## 2. Task and labels

The track predicts whether a compound exhibits significant time-dependent
inhibition for CYP2D6 or CYP3A4. The organizer-defined outcome is based on a
greater-than-two-fold IC50 shift between direct and preincubation/TDI assay
conditions, including the organizers' rules for confidently inferred outcomes
where a shift is not directly measurable.

The final output is binary for each of 750 blinded molecules and each of the
two isoforms. The operating points were treated as part of the model and were
validated with complete analogue-family grouping rather than chosen from the
public leaderboard.

## 3. Data and representations

### 3.1 Supervision

The model used the public challenge releases: official TDI labels, direct and
TDI-condition pIC50 measurements, pIC50 shifts, Emax arms,
single-concentration readouts, and assay-advancement/scoreability indicators.
No proprietary assay collection was used.

The final CYP2D6 meta-head fit contained:

- 4,816 proxy-population rows;
- 1,122 rows with official CYP2D6 TDI outcomes;
- 239 positive and 883 negative labelled rows; and
- 58 deployable features.

### 3.2 Molecular and assay representations

The lineage used public pretrained ChemProp-medium and CheMeleon molecular
representations, Morgan/fingerprint and nearest-neighbour analogue summaries,
and a published LightGBM ranking component. ChemProp was also trained as a
masked multi-assay predictor over 28 related assay targets. The 58-feature v204
meta-head included:

- frozen v105, v119, and development v128 ranks;
- embedding, fingerprint, and k-nearest-neighbour outcome/scoreability ranks;
- parent-compound TDI, direct-potency, TDI-condition, and shift information;
- parent and query–parent similarity/count features;
- cross-isoform selection and assay features;
- within-series mean, maximum, standard deviation, and rank features; and
- separately cross-fitted scoreability and conditional-outcome predictions.

No target-test labels, private wet-lab outcomes, or structure-track ground truth
were used.

## 4. Model lineage

### 4.1 CYP3A4 lineage

The CYP3A4 vector originates from v98 and is unchanged in v105, v119, and v204.
The v98 rank was:

```text
0.75 × rank(ChemProp-medium → PCA64 → three-seed TabICL mean-minus-SD)
+ 0.25 × rank(public LightGBM)
```

The top 224 compounds were called positive. The 224-call operating point came
from a 29.855% pooled evaluated-call fraction selected by six-fold rank-based
validation, avoiding an absolute probability threshold that had previously
proved calibration-sensitive.

### 4.2 CYP2D6 v105 and v119 anchors

v105 combined 90.5% of the prior v98 rank with 9.5% of a lower-confidence-bound
rank from ChemProp-medium and CheMeleon assay-funnel models, then reduced the
operating point from 65 to 50 calls using chemistry-group validation.

v119 added a 5% rank increment from a ChemProp multi-assay mechanistic head. It
jointly predicted the continuous measurements underlying the TDI rule and used
the margin

```text
min((predicted pIC50 shift - log10(2)) / 0.25,
    (predicted TDI-condition pIC50 - 4.30) / 0.35)
```

The deployed ChemProp score was a lower-confidence bound across three
frozen-encoder seeds, one last-message-block fine-tune, and one low-learning-rate
full-encoder fine-tune. Each specification was an ensemble across six
chemistry-group folds. CYP2D6 remained fixed at 50 calls.

### 4.3 Final v204 meta-head

v204 re-ranked v119 with an outcome head trained to recognize likely negative
calls near the decision boundary. For each of six random seeds, LightGBM and
ExtraTrees models were fitted; a class-balanced logistic model provided a
different inductive bias. The three model-family ranks were averaged as a
negative-outcome consensus and inverted into a positive-outcome rank.

Training observations inside the current v119 top 50 received weight 4,
positions 51–100 received weight 2, and remaining labelled rows received weight
1. This concentrates model capacity on decisions capable of changing MCC while
retaining population context.

The frozen final blend changed only eight CYP2D6 decisions relative to v119:
four promotions and four demotions. CYP3A4 was byte-for-byte unchanged. Six
independent final-model seeds produced exactly the same blinded 50-call set.

## 5. Leakage controls and validation

The principal unit of separation was the complete parent/analogue family.
Every outer validation fit excluded:

1. the complete held cohort;
2. every row sharing a parent molecule with that cohort; and
3. the confirmation cohort during policy selection.

All preprocessing and model fits were repeated inside the corresponding
training partition. The blend weight and call count were fixed before the
single-read confirmation cohort was inspected.

### 5.1 Parent-purged development result

Across seven selection cohorts, relative to v119:

| Result | Value |
|---|---:|
| Mean MCC change | +0.1102 |
| Worst cohort MCC change | 0.0000 |
| Non-losing cohorts | 7/7 |
| Improved cohorts | 6/7 |
| Untouched confirmation MCC change | +0.0706 |

### 5.2 Post-freeze robustness audit

After freezing v204, a separate audit reconstructed the policy without reading
submission files, blinded labels, or leaderboard metrics:

- nested leave-one-cohort selection chose the 40% meta-head in 7/7 folds;
- nested leave-two-cohort selection chose it in 21/21 combinations;
- the policy helped 50 and harmed 25 scored call changes across development;
- the untouched confirmation cohort recorded four helped and one harmed change;
- a 20,000-replicate joint cohort/parent-family bootstrap gave a mean MCC
  change of +0.1087 and a descriptive 95% interval of +0.0033 to +0.2239.

These uncertainty calculations are descriptive because the broader research
program tested multiple hypotheses; they are not presented as a pristine
confirmatory p-value.

## 6. Final artifact and observed live result

The official submission validator passed with 750 unique molecule identifiers,
50 CYP2D6 calls, and 224 CYP3A4 calls.

```text
filename: classification_v204_d6_parent_purged_rankblend40_top50_a4_frozen.csv
SHA-256:  2fbfbbe9cef28d19755d43260afaa50e2dff7ac1f3f49bec2e65c24456d31cab
```

The first v204 upload was accepted on 2026-10-31 at 05:17 UTC. Its public-half
score was MA-MCC 0.4474, with CYP2D6 MCC 0.3794 and CYP3A4 MCC 0.5154. These
leaderboard results were observed only after the model was frozen and were not
used to modify v204.

## 7. Leaderboard-use disclosure and anti-overfitting policy

An early predecessor, v96, used an unstable absolute operating threshold and
performed poorly. Public feedback was used to diagnose excess CYP3A4 call rate
and to restore the previously stronger CYP2D6 vector when building v98. From
that point, operating points were converted to rank/count policies and changes
were required to pass grouped validation.

For v204 specifically:

- blinded labels were never available;
- leaderboard metrics did not select the features, learners, 0.40 weight, or
  50-call operating point;
- the public live score was reporting-only after submission; and
- later leaderboard-guided swaps were explicitly prohibited.

Post-v204 experiments involving uncertainty-aware labels, LambdaRank,
CatBoost, a larger feature panel, scoreability/outcome hurdle models,
blind-parent deployment purging, boundary-only precision rescue, mechanistic
residuals, joint assay modelling, metabolite features, external reversible
potency, and MoLFormer embeddings did not clear the parent-purged promotion
gates. None replaced v204.

## 8. Limitations

- TDI is a difficult, assay-dependent endpoint with substantial censoring and
  uncertainty near the two-fold shift boundary.
- Validation cohorts approximate the challenge's analogue-series construction
  but are retrospective proxies, not a second prospective wet-lab test.
- Fixed top-k calling improves calibration stability but assumes that class
  prevalence is reasonably stable between development and final evaluation.
- Parent-series features can be powerful in this challenge design but may
  transfer poorly to unrelated chemical distributions.
- Public leaderboard scores cover only one parent-series half of the test set;
  the final full-test rank may differ.

## 9. Reproducibility record

Core implementation used Python, pandas, NumPy, SciPy, scikit-learn, LightGBM,
RDKit-derived molecular features, ChemProp-family pretrained representations,
and TabICL. Final tree seeds were 20201–20206 with deterministic stable ranking.
All frozen artifacts were checksummed before upload. Model binaries and full
feature tables are retained by the participant and can be supplied to the
organizers if requested.

## 10. References

- [Challenge Space](https://huggingface.co/spaces/openadmet/cyp-challenge)
- [Official challenge tutorial](https://github.com/OpenADMET/CYP-Challenge-Tutorial)
- [Challenge announcement](https://www.openadmet.org/blog/announcing-openadmets-cyp-inhibition-blind-challenge/)
- [Midpoint and evaluation update](https://openadmet.ghost.io/halfway-through-the-openadmet-cyp-challenge/)
