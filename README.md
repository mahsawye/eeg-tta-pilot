# EEG Test-Time Adaptation Pilot

A small-scale pilot on cross-subject motor-imagery EEG decoding. It tests
whether combining Euclidean Alignment (EA) with lightweight test-time
adaptation (Tent) is more stable than either alone. This is preparation for
a proposed MSc project on stable test-time adaptation for EEG foundation models.

## Setup
- Dataset: BCI Competition IV-2a (9 subjects, 4-class motor imagery), loaded via MOABB.
- Model: EEGNet (PyTorch), trained from scratch.
- Protocol: leave-one-subject-out (train on 8 subjects, test on the 9th).
- Methods: no adaptation, EA, BN statistics only, Tent, EA + Tent,
  EA + Tent with entropy-based sample filtering.
- 3 random seeds (model initialization and training order).

## Results (mean accuracy over 9 subjects; mean and std over 3 seeds)
| Method | Accuracy (%) | Subjects worse than no adaptation (per seed) |
|---|---|---|
| No adaptation | 48.4 +- 0.9 | - |
| EA | 57.2 +- 0.5 | [from script] |
| BN statistics only | 51.9 +- 0.4 | [from script] |
| Tent | 51.5 +- 0.5 | [from script] |
| EA + Tent | 59.2 +- 0.2 | [from script] |
| EA + Tent + filter | 59.4 +- 0.2 | [from script] |

Per-subject, per-seed accuracies are in results/results.csv.

## Findings
- EA gave the largest gain (+8.8 points) over no adaptation.
- Tent alone gave no gain over updating only BN statistics (51.5 vs 51.9).
- EA + Tent improved over EA alone by about 2 points, consistently across seeds.
- The entropy filter made no measurable difference (+0.2 points, within noise).
- Accuracy varied strongly across subjects (about 30% to 67% without adaptation),
  illustrating the distribution-shift problem.

## Limitations
- Only 9 subjects. Seeds vary model training only, so the std does not capture
  subject-level uncertainty; no significance tests were run.
- Small EEGNet trained from scratch, not an EEG foundation model.
- EA is computed offline on all unlabeled test trials of the target subject.
- Hyperparameters (learning rate, batch size, filter threshold) were fixed, not tuned.

## Next steps
- Repeat with a pretrained EEG foundation model.
- Test streaming (online) alignment and additional datasets.
- Design an adaptation method that is both accurate and stable.

## Run
pip install -r requirements.txt, then run notebooks/pilot.ipynb.

## References
- Lawhern et al., EEGNet, J. Neural Eng., 2018.
- He and Wu, Transfer learning for BCIs: a Euclidean space data alignment approach, IEEE TBME, 2020.
- Wang et al., Tent: Fully test-time adaptation by entropy minimization, ICLR 2021.
- Jayaram and Barachant, MOABB, J. Neural Eng., 2018.
- Brunner et al., BCI Competition 2008, Graz dataset 2a.
- Lee, Pradeepkumar, Sun, Test-Time Adaptation for EEG Foundation Models, arXiv:2604.16926.
