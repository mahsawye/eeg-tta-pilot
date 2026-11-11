# EEG Test-Time Adaptation Pilot

A small-scale pilot on cross-subject motor-imagery EEG decoding. It tests
whether combining Euclidean Alignment (EA) with lightweight test-time
adaptation (Tent) is more stable than either alone. This is preparation for
a proposed MSc project on stable test-time adaptation for EEG foundation models.

## Setup
- Dataset: BCI Competition IV dataset 2a (Graz data set A): 9 subjects,
  4-class motor imagery, loaded via MOABB.
- Model: EEGNet-style CNN (simplified re-implementation in PyTorch), trained from scratch.
- Protocol: leave-one-subject-out (train on 8 subjects, test on the 9th).
- Methods: no adaptation; EA; BN statistics only; Tent; EA + Tent;
  EA + Tent with entropy-based sample filtering.
- 3 random seeds (model initialization and training order).

## Implementation details
- EA: each subject's trials are whitened by the inverse square root of that
  subject's mean trial covariance. For the test subject this uses all of its
  unlabeled trials (offline variant).
- Tent: updates only BatchNorm affine parameters (Adam, lr 1e-3, batch size 32),
  one online pass over the test trials; predictions are made before each update.
- BN-only control: uses test-batch BatchNorm statistics without any gradient step.
- Filter: only trials with prediction entropy below 0.4 * ln(4) contribute
  to the Tent update (threshold chosen a priori, not tuned).

## Results (mean accuracy over 9 subjects; mean and std over 3 seeds)
| Method | Accuracy (%) | Subjects worse than no adaptation (seed 0 / 1 / 2) |
|---|---|---|
| No adaptation | 48.4 +- 0.9 | - |
| EA | 57.2 +- 0.5 | 0 / 1 / 0 |
| BN statistics only | 51.9 +- 0.4 | 1 / 1 / 1 |
| Tent | 51.5 +- 0.5 | 2 / 1 / 2 |
| EA + Tent | 59.2 +- 0.2 | 0 / 0 / 0 |
| EA + Tent + filter | 59.4 +- 0.2 | 0 / 0 / 0 |

Per-subject, per-seed accuracies are in results/results.csv.

## Findings
- EA gave the largest gain (+8.8 points) over no adaptation.
- Tent alone gave no gain over updating only BN statistics (51.5 vs 51.9)
  and made some subjects worse in every seed.
- EA + Tent improved over EA alone by about 2 points, consistently across seeds.
- The entropy filter made no measurable difference (+0.2 points, within noise).
- Without adaptation, per-subject accuracy ranged from about 30% to 67% (seed 0),
  illustrating the distribution-shift problem.

## Limitations
- Only 9 subjects. The std reflects training randomness only, not subject-level
  uncertainty; no significance tests were run.
- Small EEGNet-style model trained from scratch, not an EEG foundation model.
- EA is computed offline on all unlabeled test trials of the target subject;
  the original EA paper also describes an online variant, which was not tested here.
- Hyperparameters (learning rate, batch size, filter threshold) were fixed, not tuned.

## Next steps
- Repeat with a pretrained EEG foundation model.
- Test streaming (online) alignment and additional datasets.
- Design an adaptation method that is both accurate and stable.

## Run
pip install -r requirements.txt, then run notebooks/pilot.ipynb.

## References
- Lawhern et al., EEGNet: a compact convolutional neural network for EEG-based
  brain-computer interfaces. J. Neural Eng. 15(5):056013, 2018.
- He and Wu, Transfer Learning for Brain-Computer Interfaces: A Euclidean Space
  Data Alignment Approach. IEEE Trans. Biomed. Eng. 67(2):399-410, 2020.
- Wang, Shelhamer, Liu, Olshausen, Darrell, Tent: Fully Test-time Adaptation by
  Entropy Minimization. ICLR 2021.
- Jayaram and Barachant, MOABB: trustworthy algorithm benchmarking for BCIs.
  J. Neural Eng. 15(6):066011, 2018.
- Brunner, Leeb, Muller-Putz, Schlogl, Pfurtscheller, BCI Competition 2008,
  Graz data set A (BCI Competition IV). https://www.bbci.de/competition/iv
- Lee, Pradeepkumar, Sun, Test-Time Adaptation for EEG Foundation Models:
  A Systematic Study under Real-World Distribution Shifts. arXiv:2604.16926.
