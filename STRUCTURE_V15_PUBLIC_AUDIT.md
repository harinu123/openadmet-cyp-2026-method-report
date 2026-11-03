# Public-only audit for the v15 OCNT-2313397 structure switch

## Question

Does replacing the scored v14 OCNT-2313397 Boltz pose with the frozen v15
ESMFold2 pose have evidence independent of challenge structure labels,
per-target challenge scores, and external leaderboard models?

## Controlling comparison

- **Baseline:** the exact public `selected_current` pose for every matched
  system, scored with organizer-compatible OST LDDT-PLI.
- **Candidate policy:** shortlist the top two ESMFold2 poses using a frozen
  pre-cutoff geometry ranker, then choose the pose with the higher score from a
  separately frozen pre-cutoff residue/contact ranker.
- **Leakage control:** rankers used scaffold GroupKFold during development and
  were evaluated on later public structures. No challenge structure labels
  were read.
- **Exact reproduction:** the policy selects the frozen v15 OCNT-2313397 pose
  hash `b2f46cbc4c04c88e772af2b08cad77e8decec79392a21d7eb6d53091c959804e`.

The ranking tables contained 5,185 public-truth pose rows across 112 systems,
with no duplicate keys. Candidate and baseline OST scores agreed exactly where
the source tables overlapped. Twenty public systems had both an exact incumbent
baseline and eligible ESMFold2 candidates.

## Paired results

| Public stratum | n | Wins/losses | Mean change | System-bootstrap 95% interval | Cluster-bootstrap 95% interval |
|---|---:|---:|---:|---:|---:|
| All matched systems | 20 | 18/2 | +0.1644 | +0.0982 to +0.2257 | +0.0790 to +0.2227 |
| Type-II heme coordination | 18 | 17/1 | +0.1854 | +0.1200 to +0.2442 | +0.1355 to +0.2632 |
| Mechanism matched | 7 | 7/0 | +0.1323 | +0.0933 to +0.1786 | +0.1187 to +0.1554 |
| Strict non-congeneric match | 2 | 2/0 | +0.1554 | +0.1463 to +0.1644 | descriptive only |

The mechanism-matched stratum requires a type-II ligand, a Boltz incumbent,
and the same approximately 2.83 Å non-coordinating Fe–N mode observed for the
v14 OCNT-2313397 pose. Its leave-one-cluster-out minimum mean change was
+0.1270. All seven systems improved, but they represent only four independent
chemical clusters; the two-sided cluster sign-flip p-value was therefore 0.125.

## Selector sensitivity

The result was not specific to choosing a top-two shortlist:

| Geometry shortlist | Wins/losses | Mean mechanism-matched change |
|---:|---:|---:|
| 1 | 7/0 | +0.1410 |
| 2 | 7/0 | +0.1323 |
| 3 | 7/0 | +0.1414 |
| 5 | 7/0 | +0.1366 |

## What the audit supports

The evidence supports the complete frozen ESMFold2 pose-selection policy, not
merely editing the Fe–N distance. Although the ESMFold2 selections reduced mean
Fe–N error, only 8/18 type-II systems improved that error and Fe–N-error
reduction was not positively correlated with LDDT-PLI gain.

As a scale illustration, applying the mechanism-matched mean change to one of
20 challenge systems would move a 0.46956 mean to approximately 0.47618. Using
the strict two-system descriptive mean gives approximately 0.47733. These are
scenarios, not forecasts.

## Limitations and decision

- The strictest target-matched set contains only two public systems.
- The 18-system type-II set contains two large congeneric families and only
  five independent clusters.
- OCNT-2313397 is chemically out of domain: maximum observed Morgan-radius-2
  Tanimoto similarity to the public type-II ligands was approximately 0.157.
- A blinded target can still behave differently; only retaining v14 could
  guarantee no coordinate-induced decrease.

The audit therefore materially favors v15 as a controlled expected-value
submission, but it does not establish a guaranteed improvement. External
leaderboard poses were not used in this conclusion.
