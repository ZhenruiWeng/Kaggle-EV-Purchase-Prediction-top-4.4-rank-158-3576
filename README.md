# EV Grand Prix — 83-Engine Blend for EV Purchase Prediction

**Kaggle Playground Series S6E9 · Predicting Electric Vehicle Purchases — final rank
158 / 3,576 (top 4.4%), public LB 0.94659.**

A **CPU-only, training-free ensemble pipeline** that blends 83 public out-of-fold
prediction "engines" from 21 sources into one submission — and audits every step
statistically: probit rank normalization, correlation-fingerprint input checks,
nested Caruana hill climbing with fold gating, paired DeLong confidence intervals,
generator-rule masks, and a label-calibrated Fisher blending model that predicts
public-LB gains to within 0.00001.

> The notebook runs in ~4 minutes on CPU, trains nothing, and is written as a
> 12-lap race with ASCII telemetry. Every lap ends with a "pit-stop lesson."

## Result

| | |
|---|---|
| Final rank | **158 / 3,576 (top 4.4%)** |
| Public LB | **0.94659** (v6 challenger) |
| Blend OOF AUC (with rules) | 0.946487 |
| Best single engine (pole) | 0.946423 — a 61-model Fusion IV |
| Runtime | ~250 s, CPU only, no training |

## The method, lap by lap

| Lap | What happens | Key device |
|---:|---|---|
| 1 | Load & verify every input | strict path hints, ID-aligned merge, original-order assertions |
| 2 | Load 83 (OOF, test) engine pairs | 8 OOF datasets + 12 notebook outputs, **correlation fingerprint check** (max drift +0.016, none dropped) |
| 3 | Put every engine on one track | rank → probit ("Gaussianize") — scale-free, AUC-invariant |
| 4 | Single-engine AUC Grand Prix | OOF AUC ladder from 0.9455 to pole 0.946423 |
| 5 | Slipstream map | Spearman 1−ρ hierarchical clustering: 83 engines → **69 independent voices** |
| 6 | **Nested Caruana hill climb** | selection *inside* folds, accept only on 5/5 folds → OOF 0.946476 (vs pole +0.00005) |
| 7 | Control cars | equal mean / medoid mean / logistic stack / team car (hill + ¼ stack) → 0.946482, wins 5/5 |
| 8 | Does the gain beat noise? | **paired DeLong interval**: Δ +0.000058, 95% CI [+0.000039, +0.000078], z = 5.96 |
| 9 | Rules of the road | 4 pure generator cells, validated per fold (+0.0000056/fold) |
| 10 | **Pit-stop model** | k = (1−ρ_eff)/(1−ρ_top-half) calibrated on 66 labeled pairs (k = 0.285); spread collapses 34% → 4% |
| 11 | Timing tower | the calibrated Fisher model over every hash-pinned file with an exact public score |
| 12 | Cross the line | rules re-applied; challenger + OOF-only files written |

### The four generator rules (free AUC)

| Rule | Train rows | Buyer rate |
|---|---:|---:|
| `income ≥ 170,537` → always buys | 393 | 100% |
| `income 31,004–41,970` → never buys | 1,257 | 0% |
| `commute ≥ 83 km` → never buys | 186 | 0% |
| `30k income, no subsidy, concern ≤ 1 or anxious` → never buys | 7,157 | 0% |

### Version lineage (each version shipped one idea)

| Version | Idea added | Public LB |
|---|---|---:|
| v1 | 48 OOF engines, nested hill climb, uncalibrated Fisher merge | 0.94655 |
| v2 | 67 engines, 4 generator rules, Fisher fitted to public scores | 0.94657 |
| v3 | 81 engines, team car, **pit-stop model calibrated on labels** | 0.94657 |
| v4 | **timing tower** over files with exact public scores | 0.94659 |
| v5 | design, input hashing, 16 tower files | 0.94659 |
| **v6** | hash-pinned external blend in the tower, 10% floor, ASCII telemetry | **0.94659** |

## Honesty protocol (the part worth reading twice)

- Laps 1–10 use **only labeled OOF predictions and unlabeled test predictions**.
- Public scores are used to *check* the calibrated model's predictions, never to fit it
  (the pit-stop constant k is learned from 66 labeled pairs).
- Every external file is **SHA-256 hash-pinned**; a new 0.94656 "teacher–student"
  submission was caught byte-identical to a known public-split probe and dropped.
- Two final files: the `challenger` (uses public scores) and an `OOF-only` hill blend
  (labels only) — know which leaderboard you are optimizing.

## Data

Train 668,665 rows · test 286,571 rows · positive rate 17.46% · target `Will_Buy_EV`.

## Repository structure

```
├── notebooks/ev-grand-prix-v6-final-lap-s6e9.ipynb   # the complete pipeline, with all ASCII outputs
├── README.md · requirements.txt · LICENSE (Apache-2.0)
```

The notebook is the single self-contained pipeline (14 linear cells, ~750 lines).
No separate `src/` copy is provided because every cell reads from the Kaggle-attached
OOF input libraries listed in Lap 1 — on Kaggle, attach the inputs and run top-to-bottom.

## Credits

This blend stands on public OOF libraries and notebook outputs shared by the S6E9
community — @najiama, @megayak, @dariushafshar, @medvax, @blamerx, @legtarrr,
@tamerlanomralinov, @heuljax, @evgendvorkin, @kirill0212, @mizushimatoshihiko,
@marcmaldonado, @aryankaisth, @yekenot, @mikhailnaumov, @crystalbaby — plus the
public anchor catalog by @jazivxt, the Fisher-from-scores idea by @thisray, the
DeLong/medoid framing by @denpugovkin, generator rules mined by @megayak and
@defiaudit, the generator recipe insight by @cdeotte, and the leaderboard instrument
by @georgymamarin. Full per-lap credits are in the notebook's podium section.

## License

[Apache-2.0](LICENSE)
