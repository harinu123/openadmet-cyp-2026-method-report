# CYP3A4 Protein–Ligand Structure Prediction Using Physically Constrained Multi-Model Consensus

**Participant:** harinu123  
**Challenge:** OpenADMET CYP Inhibition Blind Challenge  
**Track:** Structure Prediction  
**Primary evaluation metric:** LDDT-PLI  
**Proprietary experimental data used:** No  
**Public code release:** No

## Abstract

We developed a multi-generator structure-prediction workflow for 20 CYP3A4–ligand complexes in the OpenADMET structure-prediction track. The initial candidate pool comprised 900 complexes generated using Boltz-2, RoseTTAFold 3, and MoleculeMind-FMOM. Candidate structures were registered to a common heme reference frame and filtered for chemical validity, stereochemistry, and molecular completeness before selection. A deterministic selector combined symmetry-aware ligand-pose clustering with generator-balanced consensus, a reproducibility criterion for RoseTTAFold 3, and a conservative Boltz-2 fallback.

On five public CYP3A4 systems evaluated with cognate structural templates withheld, the selector achieved a mean strict LDDT-PLI of 0.4266, compared with 0.3988 for the best single-generator confidence baseline. Eighteen submitted complexes were retained from this fixed selection procedure. Two others underwent target-specific revisions: an ESMFold2-derived, stereochemically repaired pose supported by a heme-coordination hypothesis, and a Boltz-2 pose selected through a bounded conformational search.

The final archive contained seven Boltz-2, nine RoseTTAFold 3, three MoleculeMind-FMOM, and one ESMFold2-derived complex. All 20 submitted structures passed the local PoseBusters checks and the official submission-format validator. These checks establish chemical and packaging validity; the five-system public evaluation, rather than the validation checks, provides the available evidence of pose-prediction accuracy.

## 1. Prediction Task and Rationale

The structure-prediction track required full three-dimensional models of 20 CYP3A4–ligand complexes. The challenge's primary structural metric was the local distance difference test for protein–ligand interactions (LDDT-PLI). Predictions were also subjected to chemical and geometric validity checks, including PoseBusters, which could invalidate an otherwise geometrically accurate prediction [1, 2]. Experimental structures for the challenge targets were not available to the participant during model selection or preparation of the submitted archive.

CYP3A4 poses a particular challenge because its binding cavity accommodates chemically diverse ligands and exhibits substantial conformational flexibility. We therefore separated **candidate generation** from **candidate selection**. Multiple structure-prediction models and receptor/template states were used to explore possible binding modes. A subsequent selector prioritized chemically valid modes with support across distinct generators, rather than relying exclusively on the confidence score of any single model.

This design increases the chance of sampling a plausible binding mode, but cannot recover a correct pose if none of the generators proposes one.

## 2. Candidate Generation

The initial production pool contained 45 candidate complexes for each of the 20 targets, for a total of 900 structures.

| Structure generator | Candidates per target | Total candidates |
|---|---:|---:|
| Boltz-2 | 35 | 700 |
| RoseTTAFold 3 | 5 | 100 |
| MoleculeMind-FMOM | 5 | 100 |
| **Total** | **45** | **900** |

### 2.1. Boltz-2 sampling across receptor states

Boltz-2 [3] was run with physics-based guidance (`physical_guidance_update=True`) and without FK particle steering. Seven receptor/template sampling arms were used for each ligand. Five arms were initialized from the public CYP3A4 structure **PDB 9BVB**, using seeds 11, 37, 73, 109, and 151. Two additional arms used the public CYP3A4 structural states **PDB 38FU** and **PDB 38FY** [7]. Each arm generated five diffusion samples per target, giving 35 Boltz-2 candidates per ligand.

The use of distinct receptor states was intended to account for binding-site flexibility and to reduce dependence on a single template conformation. Physics-based potentials were used to discourage implausible geometries during sampling; FK particle steering was omitted because of its computational cost.

### 2.2. Additional structural generators

Five RoseTTAFold 3 complexes [4] and five MoleculeMind-FMOM complexes were generated per target. These generators supplied alternative structural hypotheses with different modelling assumptions. Their inclusion was intended to improve pose-space coverage and provide cross-model corroboration, not to assume that agreement between models guarantees correctness.

## 3. Structural Normalization and Validity Filtering

Candidate complexes were standardized to a common representation containing the CYP3A4 protein, the heme cofactor (HEM), and the query ligand. Atom and residue identities and ligand atom mappings were normalized before comparison. Because receptor/template structures originated from different coordinate systems, heme coordinates were aligned to the HEM group of PDB 9BVB. Ligand geometries could then be compared within a shared reference frame.

Candidate structures were required to pass the following criteria before entering pose clustering:

1. Strict, organizer-compatible PoseBusters docking checks [5].
2. Coordinate-derived stereochemical agreement with the authoritative ligand SMILES.
3. Consistent ligand atom identity and mapping.
4. Complete protein, HEM, and ligand components.

Invalid candidates were excluded rather than penalized in a continuous score. Thus, a frequently sampled binding mode could not be selected solely on the strength of its cluster size if it violated the chemical or stereochemical requirements.

## 4. Geometry-Based Pose Selection

### 4.1. Symmetry-aware pose clustering

For each target, pairwise ligand-pose distances were calculated after registration to the common heme frame. Distances used symmetry-corrected heavy-atom RMSD so that equivalent ligand atom mappings did not artificially increase the distance between chemically identical poses.

Valid candidates were grouped using average-linkage hierarchical clustering with a **2.0 Å RMSD cutoff**. Average linkage was preferred to single linkage to reduce the tendency of intermediate poses to connect otherwise distinct binding modes.

### 4.2. Generator-balanced mode ranking

Each resulting pose cluster was treated as a candidate binding mode. Modes were ranked lexicographically according to:

1. The number of distinct generators represented.
2. Generator-balanced support.
3. The number of distinct Boltz-2 receptor/template arms represented.
4. Within-mode compactness, with lower mean RMSD preferred.

For a mode `M`, let `M_g` be the poses in that mode contributed by generator `g`, and let `A_g` be all valid poses contributed by the same generator for that target. The generator-balanced support score was defined as:

`B(M) = sum over supporting generators g of [ |M_g| / |A_g| ]`.

Normalizing by the number of valid poses per generator was important because the initial pool contained 35 Boltz-2 candidates but only five candidates from each of the other two generators. Without this normalization, raw cluster size would favor Boltz-2 because it was sampled more extensively.

When the highest-ranked mode contained contributions from **at least two generators**, its representative was selected by a generator-balanced medoid rule. For each candidate pose, its mean RMSD to members of the mode was first calculated separately within each represented generator; these generator-specific means were then averaged with equal weight. The candidate with the smallest resulting distance was selected.

Model confidence was not used to choose among candidates within such a cross-generator mode.

### 4.3. Stable RoseTTAFold 3 selection and Boltz-2 fallback

A separate within-generator analysis clustered the five RoseTTAFold 3 poses at the same 2.0 Å cutoff. A RoseTTAFold 3 mode was considered reproducible if **at least three of five samples** belonged to it. The largest cluster was preferred; ties were resolved by within-cluster compactness, and its medoid was the representative pose.

A stable RoseTTAFold 3 pose was eligible to supersede the cross-generator medoid only if it lay within **3.0 Å** of at least one PoseBusters-valid Boltz-2 pose. This requirement coupled within-model reproducibility to support from an alternative generator.

The fixed selector applied the following decision order:

1. Select the medoid of a stable RoseTTAFold 3 mode if the three-of-five criterion and the 3.0 Å Boltz-2 corroboration criterion are both satisfied.
2. Otherwise, select the generator-balanced medoid of the highest-ranked mode supported by at least two generators.
3. If neither condition is met, select the highest-confidence valid Boltz-2 pose.

This rule was deterministic given the candidate pool, atom mappings, RMSD calculations, and model confidence values. It did not use a learned pose-quality regressor or require a ground-truth structure for the target being selected.

## 5. Public-Structure Validation

The selection procedure was evaluated on **five public CYP3A4 ligand systems** with the corresponding cognate experimental template excluded during prediction for each system. Structural agreement was measured using strict LDDT-PLI, and all reported comparisons were restricted to PoseBusters-valid predictions.

| Selection procedure | Mean strict LDDT-PLI | PoseBusters pass rate |
|---|---:|---:|
| **Multi-generator hybrid selector** | **0.4266** | **100%** |
| Stable RoseTTAFold 3 rule alone | 0.4129 | 100% |
| Cross-generator consensus alone | 0.4125 | 100% |
| Boltz-2 multistate confidence baseline | 0.3988 | 100% |
| RoseTTAFold 3 confidence baseline | 0.3012 | 100% |
| MoleculeMind-FMOM confidence baseline | 0.2507 | 100% |

The hybrid procedure improved mean LDDT-PLI by **0.0278** relative to the Boltz-2 multistate confidence baseline on these five systems. The 100% PoseBusters pass rates reflect the imposed validity gate rather than an independently estimated success probability.

Its advantage over the two component selection rules was smaller, indicating that both cross-generator consensus and the stable RoseTTAFold 3 criterion contributed to the final selection procedure. Because this evaluation comprised only five systems, the results are descriptive rather than evidence of a statistically established improvement across CYP3A4 ligands more broadly.

The selector was fixed before the final challenge structures were chosen. No experimentally determined coordinates from the blinded challenge targets were used to compute or optimize the submitted poses.

## 6. Target-Specific Structural Refinements

The fixed selector initially selected seven Boltz-2, ten RoseTTAFold 3, and three MoleculeMind-FMOM structures. Eighteen of these choices were retained unchanged. Two targets received separate, explicitly motivated revisions based on structural hypotheses and public-data analyses rather than on challenge ground-truth coordinates.

### 6.1. OCNT-2313038: heme-coordination-aware selection

OCNT-2313038 contains an accessible pyridine nitrogen that could plausibly coordinate directly to the CYP3A4 heme iron, consistent with a **type-II coordination hypothesis**. The original hybrid-selected Boltz-2 pose placed this nitrogen **2.829 Å** from the iron. An alternative ESMFold2-derived structural family yielded poses with substantially shorter, approximately **2.1–2.2 Å**, nitrogen–iron separations.

To evaluate whether this mechanistic distinction justified a target-specific exception, an ESM-family selection procedure was assessed on a separate collection of **18 public CYP structures with experimentally observed heme-coordinating ligands**. The selected ESM-family structures exceeded the production-baseline LDDT-PLI in **17 of 18** systems, with a mean LDDT-PLI difference of **+0.1641**. This retrospective evidence motivated the correction; it does not establish the experimental binding geometry of the blinded target.

The preferred raw ESM-derived pose had the intended short nitrogen–iron contact but disagreed with both specified ligand stereocenters. A local stereochemical repair was therefore carried out while preserving the proposed coordination mode. The repaired submitted structure showed:

| Structural check | Result |
|---|---:|
| Coordinating nitrogen–iron distance | 2.058 Å |
| Ligand heavy-atom RMSD from the raw ESM-derived pose | 0.136 Å |
| Declared stereocenters | Both matched |
| PoseBusters checks | 22/22 passed |
| RMSD to an independently rescued alternative pose | 0.359 Å |
| RMSD to an independent ESM-derived sample | 0.489 Å |
| RMSD to the original hybrid-selected Boltz-2 pose | 2.550 Å |

The constrained MMFF repair also reduced the reported force-field energy, with a post-repair value of 51.379 under the original calculation settings. That energy was treated as a local geometry diagnostic, not as a cross-model binding free-energy estimate.

The **2.550 Å** displacement from the original selection indicates a change in the proposed ligand binding mode, rather than only a minor local adjustment. The independent structural comparisons, stereochemical agreement, and chemical-validity checks supported inclusion of this revised pose. The chemical interpretation remains a coordination hypothesis pending experimental comparison.

### 6.2. OCNT-2395645: bounded conformational refinement

The fixed selector initially chose a RoseTTAFold 3 complex for OCNT-2395645. For this target, we applied a separate publicly described bounded-conformation search heuristic [8]. Relative to a reference pose, candidate ligands were required to maintain a **centroid displacement no greater than 0.8 Å** and a **rigid-body rotation no greater than 15°**. Among candidates satisfying both constraints and the validity checks, the procedure preferred greater internal ligand conformational change.

Two additional **template-free Boltz-2** pools were generated for this target: 25 physically guided samples and 50 more broadly sampled poses. Both used three recycles, 200 sampling steps, a step scale of 1.5, and no FK steering. The physically guided pool did not supply an eligible alternative to the reference pose. In the 50-sample pool, two candidates satisfied the geometric constraints and PoseBusters checks; the candidate with the larger internal conformational change was selected.

The chosen pose had the following properties:

| Structural or confidence measure | Result |
|---|---:|
| Ligand centroid displacement | 0.345 Å |
| Rigid-body rotation | 11.67° |
| Internal-shape RMSD | 0.483 Å |
| Full ligand-pose RMSD from reference | 0.763 Å |
| Boltz-2 confidence | 0.9517 |
| Ligand ipTM | 0.9845 |
| PoseBusters checks | 22/22 passed |
| Declared stereochemistry | Matched |

The selected pose was also **0.952 Å** from the nearest independently generated Boltz-2 structure from the 38FY template arm and **1.171 Å** from the nearest RoseTTAFold 3 structure. These comparisons supplied corroboration from the original production pool, although they cannot establish accuracy against the withheld experimental structure. This correction replaced one RoseTTAFold 3 selection with a Boltz-2 structure.

Including this target-specific search, **75 additional structures** were generated beyond the initial 900-candidate production pool. The final submitted structures were selected from these pools together with the separate ESMFold2-derived candidate family described above.

## 7. Final Submission and Validation

The completed submission contained **20 CYP3A4–ligand complexes**. The model-family composition was:

| Source of submitted pose | Number of complexes |
|---|---:|
| Boltz-2 | 7 |
| RoseTTAFold 3 | 9 |
| MoleculeMind-FMOM | 3 |
| ESMFold2-derived, stereochemically repaired | 1 |
| **Total** | **20** |

The original fixed selector was responsible for **18 of 20** final selections; two underwent target-specific revisions. Although Boltz-2 contributed 700 of the original 900 candidates (77.8%), seven of the 20 final complexes (35%) were Boltz-2-derived. This difference demonstrates that the selection procedure did not simply follow raw sampling frequency; it is not, by itself, a measure of structural accuracy.

Before submission, the archive satisfied the following recorded checks:

- Exactly **20 distinct target systems** and **20 flat PDB entries**.
- Successful execution of the official organizer structure validator.
- **20/20** selected poses passed the strict local PoseBusters checks before packaging.
- **20/20** poses passed the same checks after extraction from the ZIP archive.
- All **four** ligands with specified stereocenters matched the authoritative stereochemical assignments.
- The organizer-corrected stereochemical definition for **OCNT-2312554** was satisfied.
- Every complex included the required protein, HEM, and ligand components.

**Submitted archive:** `structures.zip`  
**Archive SHA-256:** `67a0f23d148fa7badd4c25cb344a262d239dd0764a7adc1b4d59358f2fb2816c`

Passing these checks establishes file-format compliance and local chemical validity, but it does not provide an experimental LDDT-PLI score for the 20 challenge targets. No blind-target experimental coordinates were used in candidate generation, filtering, selection, or the two refinements prior to submission.

## 8. Discussion and Limitations

The principal methodological contribution is a **pose-selection strategy that separates broad candidate generation from physically constrained, generator-balanced decision making**. The initial sampling explores multiple model families and receptor states; the selection step then integrates chemical validity, cross-generator geometrical agreement, and repeated within-generator support. Two target-specific adjustments illustrate how mechanistic chemistry and local conformational exploration can supplement a general selection rule when their use is independently motivated.

Several limitations should qualify interpretation. First, the main selector was assessed on only five public leave-cognate-template-out systems; the reported improvement cannot be assumed to generalize to the 20 challenge structures. Second, distinct generative models may share training data or structural biases, so consensus is not equivalent to independent experimental evidence. Third, comparison after heme-frame registration emphasizes ligand-pose geometry and does not fully characterize receptor flexibility, interaction energetics, or alternative heme states. Fourth, consensus can overlook an accurate but rare pose supported by only one model. Fifth, the ESMFold2-derived coordination revision is applicable to a specific chemical hypothesis and should not be treated as a general preference for short nitrogen–iron distances. Finally, the bounded-conformation revision is a target-specific extrapolation, not a generally validated improvement to the primary selector.

The local PoseBusters pass rate and official archive validation should therefore be kept distinct from structural accuracy. The available accuracy evidence consists of the five-system public benchmark and the separate 18-system retrospective analysis of heme-coordinating ligands; neither directly measures the final blinded submission's experimental agreement.

## 9. Reproducibility and Data Availability

The participant retains the original candidate poses, normalized complexes, atom-mapping records, pairwise RMSD matrices, cluster assignments, selection tables, public validation results, stereochemistry audits, PoseBusters outputs, official-validator logs, and archive checksums. These records allow the final choices and the two target-specific revisions to be audited.

The source code has not been publicly released. No proprietary experimental assay or structural ground-truth data were used to construct the submission.

### Final pose provenance

| Generator | Target identifiers |
|---|---|
| Boltz-2 | OCNT-2312966, OCNT-2313397, OCNT-2315379, OCNT-2395328, OCNT-2395342, OCNT-2395645, OCNT-2535422 |
| RoseTTAFold 3 | OCNT-2313300, OCNT-2395415, OCNT-2395436, OCNT-2395457, OCNT-2395460, OCNT-2395464, OCNT-2395503, OCNT-2395712, OCNT-2395789 |
| MoleculeMind-FMOM | OCNT-2312554, OCNT-2313429, OCNT-2535519 |
| ESMFold2-derived | OCNT-2313038 |

## References

1. OpenADMET. [Halfway Through the OpenADMET CYP Challenge: Structure Track and Evaluation Protocol](https://openadmet.ghost.io/halfway-through-the-openadmet-cyp-challenge/).
2. OpenADMET. [CYP Challenge Tutorial and Submission Tools](https://github.com/OpenADMET/CYP-Challenge-Tutorial).
3. Passaro, S., et al. [Boltz-2: Towards Accurate and Efficient Binding Affinity Prediction](https://doi.org/10.1101/2025.06.14.659707). *bioRxiv* (2025).
4. [Accelerating Biomolecular Modeling with AtomWorks and RF3](https://doi.org/10.1101/2025.08.14.670328). *bioRxiv* (2025).
5. Buttenschoen, M., Morris, G. M., and Deane, C. M. [PoseBusters: AI-based docking methods fail to generate physically valid poses or generalise to novel sequences](https://doi.org/10.1039/D3SC04185A). *Chemical Science* **15**, 3130–3139 (2024).
6. Biohub. [ESMFold2 Model and Documentation](https://biohub.ai/models/esmfold2).
7. RCSB Protein Data Bank. [9BVB](https://www.rcsb.org/structure/9BVB), [38FU](https://www.rcsb.org/structure/38FU), and [38FY](https://www.rcsb.org/structure/38FY), public CYP3A4 structural references.
8. [Public Structure-Method Note Describing Bounded-Conformation Selection](https://gist.github.com/zkirby/68e31839e4a71173dae800870fee1173).
