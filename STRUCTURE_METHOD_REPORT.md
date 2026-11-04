# Method report: CYP3A4 protein–ligand structure prediction

**Participant:** harinu123

**Track:** Structure Prediction

**Proprietary data:** No

**Open-code claim:** No

## 1. Summary

The submitted archive predicts 20 CYP3A4–ligand complexes. Eighteen structures
are preserved from a frozen physically guided, multistate, multi-generator
hybrid. 

The method prioritizes physical validity and generator diversity.

## 2. Candidate generation

The frozen production pool contained 900 poses across the 20 targets:

- 700 Boltz-2 poses using physical guidance, seven receptor/template arms,
  and five diffusion samples per target/arm;
- 100 RoseTTAFold 3 poses, five samples per target; and
- 100 Moleculemind-FMOM poses, five samples per target.

Boltz-2 used physical potentials (`physical_guidance_update=True`) without FK
steering particles. Multiple CYP3A4 receptor/template states were included to
sample binding-site flexibility.


The motivation here is to make use of various answers from these different models and assume that the correct pose exists within this set, 
then we put the burden of selection on a separate selector.

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

OCNT-2313038 in particular contains an accessible pyridine nitrogen capable of type-II heme
coordination. The frozen hybrid placed this nitrogen 2.829 Å from the heme iron.



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

No blinded target structure was available or used.

## 6. Validation and packaging

The submitted archive passed all release gates:

- exactly 20 unique target systems and 20 flat PDB members;
- official organizer structure validator passed;
- 20/20 selected poses passed strict PoseBusters;
- 20/20 poses passed again after ZIP extraction;
- all four systems with declared stereochemistry reported `match`;
- the corrected organizer-defined center for OCNT-2312554 matched; and
- no target labels were used in generation, filtering, or coordinate selection.



## 8. Limitations

- The frozen selector's held-out structure set contained only five ligands.
- Public CYP structures do not perfectly reproduce the challenge distribution.
- Consensus can reinforce correlated generator errors, so generator weighting
  and physics/stereochemistry gates are necessary but not sufficient.
- The one-target mechanistic correction is specialized to an accessible
  coordinating heteroatom and should not be generalized to every ligand.
- The bounded-conformation correction matches a disclosed external rule and has
  cross-generator support, but its strict public audit is still too small for
  certainty.

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
