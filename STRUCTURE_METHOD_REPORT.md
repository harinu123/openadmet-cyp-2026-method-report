# Method report: CYP3A4 protein–ligand structure prediction

- **Participant:** harinu123
- **Latest accepted model:** v15 type-II coordination dual-switch
- **Track:** Structure Prediction
- **Primary metric:** LDDT-PLI
- **Proprietary data:** No
- **Open-code claim:** No

## 1. Summary

The v15 archive predicts 20 CYP3A4–ligand complexes. Eighteen structures are
preserved from a frozen physically guided, multistate, multi-generator hybrid.
Two targets, OCNT-2313038 and OCNT-2313397, use ESMFold2-family poses with
direct type-II heme coordination supported by validation on public CYP3A4
structures.

V15 is a controlled extension of v14. V14 changed only OCNT-2313038 and
improved the live mean LDDT-PLI from 0.4606 to 0.4696. V15 preserves that
successful correction and changes only OCNT-2313397 relative to v14.

## 2. Candidate generation

The frozen production pool contained 900 poses across the 20 targets:

- 700 Boltz-2 poses using physical guidance, seven receptor/template arms,
  and five diffusion samples per target/arm;
- 100 RoseTTAFold 3 poses, five samples per target; and
- 100 OpenBind-0 poses, five samples per target.

Boltz-2 used physical potentials (`physical_guidance_update=True`) without FK
steering particles. Multiple CYP3A4 receptor/template states were included to
sample binding-site flexibility. Additional ESMFold2-family samples were used
only for the mechanistically gated type-II coordination analysis.

## 3. Frozen hybrid selection

After physical and stereochemical filtering, the base selector:

1. computed symmetry-aware ligand RMSDs in a fixed public 9BVB HEM frame;
2. identified 2.0 Å average-linkage pose modes with equal generator weighting;
3. selected the medoid of a mode supported by at least two generators;
4. allowed an RF3 override only when at least 3/5 RF3 samples agreed and the
   pose lay within 3.0 Å of a PoseBusters-valid Boltz pose; and
5. otherwise fell back to the valid Boltz confidence baseline.

The selector was frozen on five leave-cognate-template-out public systems. Its
mean strict LDDT-PLI was 0.4266 with 100% PoseBusters pass rate, compared with
0.3988 for the Boltz multitemplate confidence baseline, 0.3012 for the RF3
confidence baseline, and 0.2507 for the OpenBind confidence baseline.

## 4. Mechanistic type-II corrections

### 4.1 OCNT-2313038: validated v14 correction

OCNT-2313038 contains an accessible pyridine nitrogen capable of type-II heme
coordination. The frozen hybrid placed this nitrogen 2.829 Å from the heme
iron. Independently generated ESMFold2-family poses consistently produced
direct coordination near 2.1–2.2 Å.

The selected source pose had the intended binding mode but inverted two
declared stereocenters. The v14 pose repaired those centers while moving the
source ligand by only 0.136 Å heavy-atom RMSD. Its checks included:

- N–Fe distance 2.058 Å;
- exact agreement at both expected stereocenters;
- 22/22 candidate PoseBusters checks passed;
- post-tether MMFF energy below the pre-tether value; and
- agreement with independently generated ESMFold2/Mol-1 and external poses.

This one-target change increased live mean LDDT-PLI by approximately 0.0090.

### 4.2 OCNT-2313397: public-only v15 extension

OCNT-2313397 exhibits the same accessible type-II coordination mechanism and
the v14 pose retained a non-coordinating approximately 2.83 Å Fe–N mode. The
v15 candidate was selected without challenge structure labels by a frozen
two-stage public selector: shortlist the two highest geometry-ranked ESMFold2
poses, then choose between them with a separately frozen residue/contact
ranker.

The exact policy selects the archived v15 pose hash
`b2f46cbc4c04c88e772af2b08cad77e8decec79392a21d7eb6d53091c959804e`.
Against exact public incumbent poses scored with organizer-compatible OST
LDDT-PLI, it produced:

| Public held-out stratum | Systems | Wins/losses | Mean LDDT-PLI change |
|---|---:|---:|---:|
| All matched systems | 20 | 18/2 | +0.1644 |
| Type-II heme-coordinating systems | 18 | 17/1 | +0.1854 |
| Same non-coordinating Boltz Fe–N mode | 7 | 7/0 | +0.1323 |
| Strict non-congeneric mechanism match | 2 | 2/0 | +0.1554 |

The seven-system mechanism-matched result remained positive when the shortlist
was varied from top-1 through top-5. Cluster bootstrap and leave-one-family-out
checks were also positive. Full audit design, uncertainty, and limitations are
reported in [STRUCTURE_V15_PUBLIC_AUDIT.md](STRUCTURE_V15_PUBLIC_AUDIT.md).

## 5. Validation and packaging

The v15 archive passed every release gate:

- exactly 20 unique target systems and 20 flat PDB members;
- official organizer structure validator passed;
- 20/20 selected poses passed strict PoseBusters;
- 20/20 poses passed again after ZIP extraction;
- all four systems with declared stereochemistry reported `match`;
- the organizer-defined center for OCNT-2312554 matched;
- the two installed pose hashes matched the frozen manifest; and
- no challenge structure labels were used in generation, filtering, or
  coordinate selection.

An exactly-once wrapper rechecked the archive hash, file manifest, official
validator, PoseBusters, stereochemistry, live API schema, and 12-hour cooldown
before uploading.

## 6. Accepted artifact and live status

```text
filename: structures.zip
SHA-256:  f1812593926e6867bb7c7e177ab5aba064c1888bcd929554b95211b57de81eb3
systems:  20
preserved from frozen hybrid: 18
switched systems: OCNT-2313038, OCNT-2313397
accepted: 2026-11-03 21:03:37 UTC
submission event: 6ca4bac6b8e048919f9c0810581970dc
```

At report update time, v15 had been accepted and was still processing. The
previous v14 upload produced live LDDT-PLI 0.4696, BiSyRMSD 3.4826, LDDT-LP
0.8048, and coverage 1.0. No v15 leaderboard score is claimed here.

## 7. Leakage and leaderboard safeguards

- Public structure labels were used only for public validation and frozen
  selector development.
- No blinded challenge structure or per-target challenge score was available.
- Broad selectors that failed controlled submissions were rejected rather than
  repeatedly tuned to the public leaderboard.
- The scored v14 improvement established the mechanism for OCNT-2313038; the
  additional v15 target was chosen from a separate public-only audit.
- V15 changes one target relative to v14 and preserves all other coordinates.

## 8. Limitations

- The strictest OCNT-2313397-like public subset contains only two systems.
- The larger type-II evaluation includes congeneric groups and only five
  independent clusters, although cluster-aware analyses remain positive.
- OCNT-2313397 is chemically out of domain relative to the public type-II set;
  maximum observed Morgan-radius-2 Tanimoto similarity was approximately
  0.157. Transfer is based on mechanism and selector behavior, not close
  chemical analogy.
- Improved Fe–N distance alone did not explain public LDDT-PLI gains; the
  evidence supports the complete selected pose rather than a distance edit.
- A public-only positive audit cannot guarantee improvement on a blinded
  target. V15 is a controlled expected-value decision, not a certainty.

## 9. Reproducibility record

Source poses, normalized complexes, selection tables, official-validator
outputs, PoseBusters results, stereochemistry audits, public paired-comparison
tables, bootstrap summaries, and archive checksums are retained by the
participant. The v15 archive is content-addressed by the SHA-256 above and can
be supplied to the organizers if requested.

## 10. References

- [Challenge Space](https://huggingface.co/spaces/openadmet/cyp-challenge)
- [Official challenge tutorial](https://github.com/OpenADMET/CYP-Challenge-Tutorial)
- [PoseBusters](https://pubs.rsc.org/en/content/articlelanding/2024/sc/d3sc04185a)
