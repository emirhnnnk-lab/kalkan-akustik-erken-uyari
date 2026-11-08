# Kalkan (Shield): detecting FPV drones early by sound and with insect vision

*Türkçe: [README.md](README.md)*

Small FPV drones are cheap, fast and hard to see on radar. I wanted to know: can we detect them in advance with low-cost sensors, using **their sound** and **simple circuits inspired by insect eyes**? This repo is the record of that R&D: a chain that starts from raw audio and goes through signal processing, a small learning model, voting between sensors, and C code meant to run on a microcontroller.

> **Defensive and warning-only.** The system's only job is early warning; no version of it has any function that could harm a living being. If any living being is nearby, whoever it is, the system stops. See [ETIK_ILKE.md](ETIK_ILKE.md).

**Live demo:** [emirhnnnk-lab.github.io/kalkan-akustik-erken-uyari](https://emirhnnnk-lab.github.io/kalkan-akustik-erken-uyari/). The scenario simulation, viewfinder, sensor network, spectrogram "ear", insect brain and Monte Carlo tabs run in the browser.

## What I did
1. **First I tested it on real data.** The simple "harmonic comb" detector I had designed for synthetic sound was about as good as a coin flip on real recordings (AUC 0.54). For me this was the most important moment of the project: my synthetic assumption didn't work in reality.
2. **I fixed the signal processing.** After balancing out the background noise (spectral whitening) and scoring frame by frame instead of all at once, it reached **AUC 0.89** on recordings it had never seen.
3. I built a **small learning model** (log-mel features + logistic regression) and exported it **to C so it can run on a microcontroller**.
4. **Sensor network:** voting between 4 sensors (k of n), losing a sensor or a packet, placement geometry, and filtering out sudden bang sounds.
5. **Acoustic "approach" estimate:** estimating the time to impact from how fast the sound grows.

## Results (honest table)
| Experiment | Result |
|---|---|
| Simple harmonic comb, 160 real recordings | AUC **0.54** |
| Whitening + frame-wise comb, separate test set | AUC **0.89**, detection 88%, false alarms 22% |
| A different dataset (open field), same threshold | AUC 0.77; the threshold doesn't carry over, it needs tuning in the field |
| Field tuning with a 30-second recording of the surroundings | False alarms cut in half, detection unchanged |
| Filtering out sudden bangs | False alarms 22% → 15% |
| 2 of 4 sensors voting (independent noise) | Detection 98.7%, false alarms 4.3% |
| Augmenting data, adding different drone sounds | Didn't help generalization (I kept this negative result too) |

I kept the negative results on purpose. To me the most valuable lesson of the project is this: test the assumptions you built on synthetic data against real data **as early as possible**.

## Insect brain: borrowing nature's circuits for vision
The systems that are best at noticing something small and fast flying against a busy background, using very little energy, are insect brains. I modeled four insect circuits:

| Insect | Circuit | Its job in Kalkan |
|---|---|---|
| Dragonfly | STMD (small target motion detector) | Picking out a small moving dot against a background of trees |
| Fruit fly | T4/T5 (Hassenstein-Reichardt motion circuit) | Separating the camera's own shaking from movement in the world |
| Locust | LGMD (looming object neuron) | Estimating time to impact from a growing image: τ = 1 / d(ln θ)/dt |
| Ormia fly | Tiny mechanically linked ears | Precise sound direction with two microphones very close together (not simulated yet) |

- **In the demo:** the "Insect brain" tab runs in the browser: an 80×60 pixel synthetic forest scene, an event camera model, STMD, T4/T5 motion flow, the approach estimate and the firing pattern of 8 direction neurons.
- **Toward hardware:** I wrote the LGMD, STMD and T4/T5 circuits in Verilog for FPGA and tried them on a test bench. Every pixel can be processed in parallel.
  - T4/T5 works cleanly: rightward motion +, leftward −, still 0.
  - **Honest limit:** the simple dragonfly circuit can't fully separate the camera's own motion from a small target (small target 202, own motion 167 points). A clean separation needs its more advanced version (ESTMD).
- I built a physics twin in Gazebo: a scene with a drone flying toward the camera. I haven't yet written the bridge that feeds the camera image into the insect circuits.

## A bug I found after publishing (Nov 8)
The demo's "Measure live on real recordings" button worked on my computer but failed online. The reason: the button read 360 real audio recordings, and those recordings were not in the repo. The dataset has no open license, so I can't distribute them. The fix: I calculated each recording's score on my own computer with the same code and published only the numbers (`sonuclar/ikiz_skorlar.json`). The threshold and AUC are now calculated live in the browser from those numbers, and the result matches my published figures exactly: AUC 0.888, detection 88%, false alarms 22%.

## Structure
```
index.html   Interactive demo that runs in the browser
sonuclar/    JSON outputs of the experiments
ETIK_ILKE.md The project's fixed ethical principle
```
> Experiment code, model weights and firmware are kept **private**; they can be shown in technical interviews.
The audio comes from open datasets: DroneAudioDataset (Al-Emadi et al. 2019, requires citation, no open license) and DroneDetectionThesis (CC0). They are not in the repo because of license and size.

## Tech
Python · numpy · scipy · scikit-learn · STFT / log-mel · C · Verilog (FPGA) · Canvas JS · Gazebo / PX4 (simulation)

---
Built by **Emirhan Karaca**
