# Results: event camera vs RGB + AI for small-drone detection (FRED, October 2026)

**Setup in one paragraph.** FRED provides synchronised events (Prophesee IMX636, 1280×720) and 30 Hz RGB video of five UAV models. The frozen split is 45 train, 5 val and 25 test sequences.
- Events: PropPulse (PP; final version with per-event resonator front end and learned decision layer), and YOLO11n on 33 ms event histograms.
- RGB: background subtraction, 3-frame U-Net (123 GFLOPs) and tiled YOLO11n.
- Each model was trained once.
- The metric is P_D over (0.2 s bin, target) pairs at each method's own threshold for a 1% or 5% false-alarm-bin rate.
- Range is emulated by k×k binning (k = 1, 2, 4).
- CIs are bootstrapped over test sequences, and paired differences are bootstrapped per sequence.

## 1. Results

### 1.1 Main comparison (FRED test, 25 sequences, k pooled)

| Method | P_D @1% [95% CI] | P_D @5% [95% CI] | k=1 / 2 / 4 @5% | <100 px/s @5% | ≥300 px/s @5% | AP (centre-hit) | best F1 |
|---|---|---|---|---|---|---|---|
| PropPulse | **0.32 [0.26–0.38]** | **0.70 [0.64–0.76]** | 0.71 / 0.69 / 0.71 | **0.57** | **0.82** | **0.863** | **0.850** |
| PP, no classifier | 0.02 [0.01–0.04] | 0.16 [0.09–0.25] | 0.18 / 0.17 / 0.13 | 0.19 | 0.08 | 0.578 | 0.691 |
| Event YOLO11n | 0.22 [0.15–0.30] | 0.49 [0.40–0.58] | 0.65 / 0.54 / 0.28 | 0.39 | 0.57 | 0.802 | 0.770 |
| RGB U-Net | 0.31 [0.23–0.40] | 0.50 [0.41–0.60] | 0.34 / 0.60 / 0.55 | 0.53 | 0.38 | 0.815 | 0.758 |
| RGB YOLO11n | 0.29 [0.21–0.37] | 0.44 [0.35–0.54] | 0.48 / 0.44 / 0.40 | 0.50 | 0.33 | 0.593 | 0.624 |
| Background subtraction | 0.01 [0.00–0.03] | 0.06 [0.02–0.10] | 0.06 / 0.05 / 0.05 | 0.03 | 0.06 | 0.552 | 0.661 |

### 1.2 Paired per-sequence differences (event − RGB; 2,000 bootstrap replicates)

| Pair | @1% | @5% |
|---|---|---|
| PropPulse − U-Net | +0.01 [−0.08, 0.09] | **+0.20 [0.12, 0.28], p < 0.001** |
| PropPulse − RGB YOLO | +0.03 [−0.06, 0.12] | **+0.26 [0.15, 0.37], p < 0.001** |
| Event YOLO − U-Net | **−0.09 [−0.17, −0.01]** (RGB better) | −0.01 [−0.10, 0.07] |
| Event YOLO − RGB YOLO | −0.06 [−0.15, 0.03] | +0.05 [−0.07, 0.17] |

The final PropPulse improves on the earlier version (0.28 / 0.64 on the same clips) by +0.04 [0.02, 0.07] @1% and +0.06 [0.04, 0.08] @5% (paired, p < 0.001). Its decision settings were chosen by cross-validation on training recordings only; the test set was evaluated once.

### 1.3 Target size (apparent span ℓ, native px, @5%)

| ℓ bin | PropPulse | Event YOLO | U-Net | RGB YOLO | bins |
|---|---|---|---|---|---|
| < 8 px | **0.54** | 0.02 | 0.32 | 0.26 | 433 |
| 8–12 px | **0.68** | 0.09 | 0.50 | 0.36 | 1,797 |
| 12–16 px | **0.68** | 0.26 | 0.59 | 0.42 | 1,560 |
| 16–24 px | **0.70** | 0.48 | 0.63 | 0.44 | 2,548 |
| 24–32 px | **0.64** | 0.57 | 0.53 | 0.43 | 1,763 |
| 32–48 px | **0.75** | 0.68 | 0.42 | 0.45 | 2,586 |
| 48–72 px | **0.73** | 0.72 | 0.42 | 0.59 | 1,696 |
| ≥ 72 px | 0.79 | **0.81** | 0.35 | 0.46 | 809 |

PropPulse leads at every size below 72 px. Event YOLO is competitive only above about 24 px. The U-Net peaks at 16–24 px and declines for large targets, consistent with its training-crop scale. The full curves are in `results/paper/fig3_range.png`.

### 1.4 Background class around the target (P_D @5% / @1%)

| Background | PropPulse | Event YOLO | U-Net | RGB YOLO | Sequences |
|---|---|---|---|---|---|
| Structure | **0.66 / 0.33** | 0.52 / 0.27 | 0.50 / 0.32 | 0.38 / 0.28 | 22 |
| Sky / plain | **0.75** / 0.34 | 0.53 / 0.24 | 0.65 / 0.43 | **0.75 / 0.51** | 21 |
| Mixed (vegetation, ground, glare) | **0.71 / 0.33** | 0.40 / 0.17 | 0.38 / 0.21 | 0.21 / 0.11 | 21 |
| Night | **0.65 / 0.24** | 0.56 / 0.21 | 0.38 / 0.20 | 0.28 / 0.13 | **2** (34, 111) |

### 1.5 UAV model (P_D @5%)

| Model | PropPulse | Event YOLO | U-Net | RGB YOLO |
|---|---|---|---|---|
| BetaFPV Air75 (small FPV) | **0.75** | 0.41 | 0.36 | 0.24 |
| DarwinFPV Cineape20 | **0.72** | 0.34 | 0.28 | 0.15 |
| DarwinFPV Cineape20 ger | 0.54 | 0.35 | 0.45 | **0.63** |
| DJI Mini 2 | **0.80** | 0.69 | 0.73 | 0.57 |
| DJI Mini 3 | **0.64** | 0.49 | 0.50 | 0.53 |
| DJI Tello EDU | 0.67 | 0.62 | **0.71** | **0.71** |

### 1.6 Single-modality urban generalisation (P_D @1% / @5% [CI], AP)

| Test set | Method | @1% | @5% | AP |
|---|---|---|---|---|
| EV-UAV (events, 346×260, ~7 px targets) | PropPulse, classifier trained on FRED only | 0.00 | 0.13 [0.06–0.21] | 0.374 |
| | PropPulse, classifier also trained on EV-UAV | 0.42 [0.32–0.54] | 0.53 [0.42–0.63] | 0.528 |
| | Event YOLO (EV-UAV in training) | **0.66 [0.58–0.74]** | **0.81 [0.73–0.87]** | **0.906** |
| DUT Anti-UAV (RGB stills) | U-Net (static stack) | 0.32 | 0.41 | 0.728 |
| | RGB YOLO | **0.77** | **0.87** | **0.958** |

### 1.7 Latency, throughput, data rate (E9a; k = 1, 238 tracks)

| Method | Median time to first detection | p90 | Detected ≤0.25 s |
|---|---|---|---|
| PropPulse | 120 ms | **1,308 ms** | 62% |
| Event YOLO | **33 ms** | 1,333 ms | **67%** |
| U-Net | 267 ms | 2,713 ms | 34% |
| RGB YOLO | **33 ms** | 1,553 ms | 50% |

**Compute per second of sensor data** (processor-seconds needed per second of data; Apple M3, idle machine):

| Method | Median recording | 90th percentile | Busiest recording |
|---|---|---|---|
| PropPulse, 1 CPU core (24 test recordings) | 0.10 | 1.4 | 7.9 |
| PropPulse, 4 CPU cores | 0.08 | 0.93 | 5.5 |
| YOLO11n, full 1280×720 frame, 30 fps (GPU, fp16) | 0.44 GPU + 0.19 CPU | — | — |

PropPulse's cost grows with the event rate (about 1.3 µs per event); YOLO's is fixed per frame. The tiled YOLO used for accuracy costs more than the full-frame figure.

- Event rate: median 89 k ev/s, which is 0.7 MB/s at 8 B/event; p90 is 12.0 MB/s.
- Raw 720p video: 27.6 MB/s in grey, 82.9 MB/s in colour.
- The busiest recording is a scene in which most of the image sways with the camera mount; it stays about 5× slower than real time.

### 1.8 Spiking resonator bank vs FFT verifier (E9b; 71,053 windows, 50 clips)

| Verifier | AUC | Decision time | Ops / window (median) | Energy estimate (45 nm) |
|---|---|---|---|---|
| FFT harmonic comb | 0.688 | window end, median 210 ms | 4.6e5 | 2.1 µJ |
| Resonate-and-fire bank (48 neurons) | 0.682 | first spike on drones, median 37 ms | 1.9e4 (19× fewer) | 0.02 µJ (≤ 0.35 µJ at 4 MAC/update) |
| Combined (rank average) | **0.761** | — | — | — |

40% of off-drone windows also spike (median 92 ms), against 83% of drone windows. For very dense windows the bank costs more than the FFT.

### 1.9 Blade-pass frequency (median f̂, score > 15, k = 1)

| Model | Median f̂ | IQR |
|---|---|---|
| DJI Mini 3 | 223 Hz | 199–242 Hz |
| DJI Mini 2 | 316 Hz | 301–332 Hz |
| DJI Tello EDU | 539 Hz | 477–574 Hz |
| BetaFPV Air75 | 738 Hz | 570–867 Hz |
| DarwinFPV Cineape20 | 547 Hz | 207–844 Hz (broad: throttle changes and notch exclusion) |

Periodicity is not observable on EV-UAV (346×260, ~7 px targets).

## Limitations

- One paired dataset (one city, five drone models, static camera).
- Each method's threshold was set from its own false-alarm rate on the test clips. Thresholds set on the small validation set gave about 5× the intended false-alarm rate on test.
- Range was emulated by k×k binning, which does not reproduce photon statistics at true distance and favours the event tracker.
- False alarms were measured only within clips that contain drones.
- The night class rests on two recordings, and background class is correlated with drone model.
- Energy figures are operation-count estimates, not measurements.

Licensed under CC BY-NC-ND 4.0.
