# Method report: CYP3A4 protein–ligand structure prediction

**Participant:** harinu123

**Track:** Structure Prediction

**Primary metric:** LDDT-PLI

**Proprietary data:** No

**Open-code claim:** No

## 1. Summary

The submitted archive predicts 20 CYP3A4–ligand complexes. Eighteen structures
are preserved from a frozen physically guided, multistate, multi-generator
hybrid. OCNT-2313038 uses a stereochemistry-repaired pose with direct type-II
heme coordination supported by independent validation on public CYP structures.
OCNT-2395645 uses a bounded-conformation Boltz pose selected by a public-only
geometry audit and four-seed ranker consensus.

The method prioritizes physical validity and generator diversity. All candidate
poses are normalized to a common protein/HEM/ligand representation, screened
with organizer-compatible PoseBusters checks, and audited against authoritative
ligand stereochemistry before selection.

## 2. Candidate generation

The frozen production pool contained 900 poses across the 20 targets:

- 700 Boltz-2 poses using physical guidance, seven receptor/template arms,
  and five diffusion samples per target/arm;
- 100 RoseTTAFold 3 poses, five samples per target; and
- 100 OpenBind-0 poses, five samples per target.

Boltz-2 used physical potentials (`physical_guidance_update=True`) without FK
steering particles. Multiple CYP3A4 receptor/template states were included to
sample binding-site flexibility.

## 3. Frozen hybrid selection

After physical and stereochemical filtering, the base selector:

1. computed symmetry-aware ligand RMSDs in a fixed public 9BVB HEM frame;
2. identified 2.0 Å average-linkage pose modes with equal generator weighting;
3. selected the medoid of a mode supported by at least two generators;
4. allowed a RoseTTAFold 3 override only when at least 3/5 samples agreed and
   the pose lay within 3.0 Å of a PoseBusters-valid Boltz pose; and
5. otherwise fell back to the valid Boltz confidence baseline.

The selector was frozen on five leave-cognate-template-out public systems. Its
mean strict LDDT-PLI was 0.4266 with a 100% PoseBusters pass rate, compared with
0.3988 for the Boltz multitemplate confidence baseline, 0.3012 for the
RoseTTAFold 3 confidence baseline, and 0.2507 for the OpenBind confidence
baseline.

## 4. Mechanism-guided correction for OCNT-2313038

OCNT-2313038 contains an accessible pyridine nitrogen capable of type-II heme
coordination. The frozen hybrid placed this nitrogen 2.829 Å from the heme iron.
An independently ranked ESMFold2/Mol-1 family consistently produced direct
coordination near 2.1–2.2 Å.

The external selector was developed on a separate public family and tested on
later public CYP structures. For 18 public structures with observed
heme-coordinating ligands, the selected full-complex ESM-family structure beat
the production baseline in 17/18 systems, with mean LDDT-PLI change +0.1641.

The best raw challenge pose had the intended binding mode but inverted both
declared stereocenters. The submitted pose repaired the stereochemistry while
moving the source ligand by only 0.136 Å heavy-atom RMSD. Its checks were:

- N–Fe distance: 2.058 Å;
- exact match at both expected stereocenters;
- PoseBusters: 22/22 candidate checks passed;
- post-tether MMFF energy: 51.379, lower than pre-tether;
- 0.359 Å agreement with rescued Mol-1 after protein alignment;
- 0.489 Å agreement with an independent ESM seed; and
- 2.550 Å difference from the frozen pose, confirming a genuine mode change.

No blinded target structure was available or used. Public leaderboard outcomes
were used only to preserve the strong 19-target base and reject broad
reranking, not to fit coordinates for OCNT-2313038.

## 5. Bounded-conformation correction for OCNT-2395645

A public method report from an independently developed high-scoring entry
described a bounded-conformation rule: preserve ligand placement within a
0.8 Å centroid shift and 15° rigid-body rotation, then select the largest
remaining internal-shape change. We evaluated that disclosed rule on our own
poses and did not use or copy another participant's coordinates.

Across 29 public CYP complexes, the supported Boltz-only implementation changed
mean LDDT-PLI by +0.0063 relative to the top-confidence Boltz pose, with 17
wins, 11 losses, and one tie. Extending the rule indiscriminately to all three
generators did not improve transfer, so the final policy remained Boltz-only.

OCNT-2395645 was the only disclosed-rule target that also passed a stringent
independent model gate: four separately seeded GNINA-augmented pairwise rankers
all voted positive, with target-level predicted changes of approximately
+0.132. The installed pose remained close in placement while changing internal
shape:

- ligand centroid shift from the Boltz reference: 0.640 Å;
- rigid-body rotation from the reference: 4.66°;
- internal-shape RMSD after rigid alignment: 1.063 Å;
- PoseBusters: 22/22 candidate checks passed; and
- authoritative stereochemistry audit: accepted.

This is a one-target, label-free selection. The public audit is supportive but
not conclusive: the closest public-chemistry similarity was low, and a strict
out-of-fold model gate selected only two public systems, one improved and one
worsened, with a small positive mean. The correction is therefore an
evidence-weighted extrapolation rather than a guaranteed gain.

## 6. Validation and packaging

The submitted archive passed all release gates:

- exactly 20 unique target systems and 20 flat PDB members;
- official organizer structure validator passed;
- 20/20 selected poses passed strict PoseBusters;
- 20/20 poses passed again after ZIP extraction;
- all four systems with declared stereochemistry reported `match`;
- the corrected organizer-defined center for OCNT-2312554 matched; and
- no target labels were used in generation, filtering, or coordinate selection.

## 7. Submitted artifact

```text
filename: structures.zip
SHA-256:  c79a380a8ac22c292090ad7d37437212b61399764511d6a618674e84c9482c41
systems:  20
preserved from frozen hybrid: 18
controlled corrections: OCNT-2313038, OCNT-2395645
```

The archive immediately preceding this final one achieved live LDDT-PLI 0.4696,
BiSyRMSD 3.4826, LDDT-LP 0.8048, and coverage 1.0. The final archive preserves
19/20 of those structures and changes only OCNT-2395645.

## 8. Limitations

- The frozen selector's held-out structure set contained only five ligands.
- Public CYP structures do not perfectly reproduce the challenge distribution.
- Consensus can reinforce correlated generator errors, so generator weighting
  and physics/stereochemistry gates are necessary but not sufficient.
- The one-target mechanistic correction is specialized to an accessible
  coordinating heteroatom and should not be generalized to every ligand.
- The bounded-conformation correction has concordant external and model support,
  but its strict public out-of-fold sample is too small for certainty.
- Public live scoring may differ from the final evaluation.

## 9. Reproducibility record

All source poses, normalized complexes, selection tables, official-validator
outputs, PoseBusters results, stereochemistry audits, and archive checksums are
retained by the participant. The submitted archive is content-addressed by the
SHA-256 above and can be supplied to the organizers if requested.

## 10. References

- [Challenge Space](https://huggingface.co/spaces/openadmet/cyp-challenge)
- [Official challenge tutorial](https://github.com/OpenADMET/CYP-Challenge-Tutorial)
- [Independent public structure-method report](https://gist.github.com/zkirby/68e31839e4a71173dae800870fee1173)
- [PoseBusters](https://pubs.rsc.org/en/content/articlelanding/2024/sc/d3sc04185a)
