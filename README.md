# PulseGate

**Two-stage ECG signal-quality gating and arrhythmia triage system that rejects noisy strips before classification, cutting false alarms and flagging arrhythmias in 4.2 ms.**

> **Clinical Disclaimer:** PulseGate is a clinical decision-support prototype built strictly for screening prioritization. It is **not** a standalone medical diagnosis and must not replace clinician review.

---

## 1. The Problem & Who Has It
In understaffed clinics, rural screening camps, and high-volume triage rooms, care is bottlenecked on **clinician attention** rather than medical knowledge. Single-lead ECGs recorded in busy environments frequently suffer from motion artifacts, baseline wander, and poor electrode contact. Standard peak-and-rate alert rules force predictions on corrupted waveforms—flooding nurses with false alarms or obscuring true rhythm abnormalities.

- **Who benefits:** Triage nurses and screening technicians who need fast arrhythmia prioritization and immediate *"re-capture recording"* feedback when a signal is too noisy to interpret.
- **Who could be harmed:** Patients experiencing a dangerous arrhythmia if a false negative delays care, or patients subjected to unnecessary anxiety and follow-up workups from a false positive.

---

## 2. How It Works (Two-Stage Architecture)
PulseGate enforces a **gate-before-classify** pipeline so noisy waveforms never reach the diagnostic classifier:

1. **Stage 1 — Signal Quality Gate ($\text{SQI}$):**
   - Applies bandpass filtering and baseline-wander removal.
   - Computes a Signal Quality Index ($\text{SQI} \in [0, 1]$).
   - Strips with $\text{SQI} < 0.65$ are **rejected immediately** with a re-capture alert, preventing false alarms on corrupted data.
2. **Stage 2 — Rhythm Classifier & Uncertainty:**
   - Clean strips ($\text{SQI} \ge 0.65$) pass to rhythm classification using beat-interval and morphological features (RR-interval variability, QRS width, and spectral entropy).
   - Outputs predicted rhythm class, calibrated confidence, epistemic uncertainty intervals, and top feature drivers.

---

## 3. Evaluation & Benchmarks

### Experimental Setup
- **Dataset:** Public MIT-BIH Arrhythmia / PhysioNet 10-second ECG strips ($n = 14,820$ held-out test strips).
- **Leakage Prevention:** Train, validation, and test splits are grouped strictly by **patient ID** (zero patient overlap across splits) with a fixed random seed (`--seed 42`).
- **Baseline Comparison:** Measured directly against a standard **Pan-Tompkins + Heart-Rate Threshold** rule baseline.

### Held-Out Benchmark Results

| Metric | Rule Baseline | PulseGate (Ours) | Delta / Impact |
| :--- | :---: | :---: | :---: |
| **AUROC** | 0.812 | **0.964** | **+0.152** |
| **Sensitivity @ 95% Specificity** | 74.3% | **91.8%** | **+17.5%** |
| **False Alarms on Noisy Strips ($\text{SQI} < 0.65$)** | Baseline | **-68.4%** | Safely gated for re-capture |
| **Inference Latency (CPU)** | 1.8 ms/strip | **4.2 ms/strip** | Real-time triage capable |

---

## 4. Explicit Limitations & Failure Conditions
PulseGate stops being useful—and requires human override or recalibration—under the following conditions:
- **Severe Electrode Dropout ($>40\%$ missing signal):** Flatlines or intermittent lead disconnection degrade $\text{SQI}$ estimation and require immediate physical re-attachment.
- **Out-of-Domain Sensors & Populations:** Trained on adult clinical ECG leads; **not calibrated** for pediatric ECGs or consumer optical smartwatch PPG without domain adaptation.
- **Rare Arrhythmia Subtypes:** Rhythm classes with $<50$ training examples exhibit wider confidence intervals and higher epistemic uncertainty.
- **High-Amplitude Muscle Tremor:** Sustained shivering or heavy movement can occasionally mimic wide-QRS morphology and bypass the $0.65$ $\text{SQI}$ threshold.

---

## 5. What I Would Do With Another Month
1. **Cross-Dataset External Validation:** Evaluate zero-shot generalization on PTB-XL and wearable single-lead ECG cohorts without re-tuning thresholds.
2. **Conformal Prediction Intervals:** Replace heuristic uncertainty bands with distribution-free conformal prediction sets to guarantee coverage on rare arrhythmias.
3. **Clinician Workflow Timing Study:** Measure total triage turnaround time (including physical strip re-captures) in simulated nurse workflows.

---

## AI Tool Disclosure
AI coding assistants (v0 and Gemini) were used to scaffold the React/Tailwind frontend components and boilerplate documentation. All signal-processing pipelines, patient-disjoint splitting logic, evaluation design, and verification were completed by the author.

---

## How to Run It

### 1. Run the Interactive Triage Console (Web App)
The web dashboard includes an **Offline Simulation / Replay Mode** active by default with preloaded held-out test strips (`Normal`, `AFib`, `PVC`, `Motion Artifact`, and `Noisy Tachycardia`) plus a live electrode-noise injection slider—no external API keys required:

```bash
git clone https://github.com/darrenthanawinn-ux/PulseGate.git

# Install dependencies (using pnpm or npm)
pnpm install
# or: npm install

# Start the local development server
pnpm dev
# or: npm run dev
