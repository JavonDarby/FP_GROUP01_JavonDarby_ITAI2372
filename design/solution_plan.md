# AI Solution Plan
## AI-Powered Astronaut Health Monitoring System — Detailed Design

**Course:** ITAI 2372  
**Track:** Conceptual Design Track  
**Student:** Javon Darby  
**Group:** Group 01  
**Date:** April 26, 2025  

---

## 1. System Overview

The Astronaut Health Monitoring AI (AHMAI) is a multi-layered machine learning system that runs continuously aboard a spacecraft. It ingests real-time biometric and environmental sensor data, processes it through an ensemble of AI models, and produces tiered health alerts and daily summary reports for crew members and mission control.

The system is designed to operate autonomously during communication blackouts and high-latency mission phases, including Mars transit where Earth communication delay can exceed 20 minutes one-way.

---

## 2. Data Pipeline

### 2.1 Input Data Streams

The system monitors the following data sources simultaneously:

| Data Type | Sensor | Sampling Rate | Description |
|-----------|--------|--------------|-------------|
| Heart Rate (HR) | Wearable ECG patch | 1 Hz | Beats per minute |
| Heart Rate Variability (HRV) | Wearable ECG patch | Derived | RMSSD metric — autonomic nervous system indicator |
| Blood Oxygen (SpO2) | Pulse oximeter | 0.2 Hz | Peripheral oxygen saturation |
| Skin Temperature | Wearable thermistor | 0.1 Hz | Peripheral circulation proxy |
| Activity Level | Accelerometer | 50 Hz | Movement classification (rest/exercise/EVA) |
| Sleep Quality | Actigraphy + EEG headband | Nightly | Sleep stage duration and efficiency score |
| Cognitive Performance | Daily digital test battery | Once daily | Reaction time, working memory, attention tasks |
| Cabin CO2 Level | Environmental sensor | 0.017 Hz | Parts per million — affects cognition above 1000 ppm |
| Radiation Dose | Dosimeter | 1 Hz | Daily accumulated dose in millisieverts |

### 2.2 Data Preprocessing

Raw sensor data is preprocessed before entering any AI model:

**Step 1 — Artifact removal**  
Signal artifacts caused by movement, sensor disconnection, or electromagnetic interference are detected using z-score thresholding (>3.5 SD flagged) and replaced via linear interpolation for gaps under 60 seconds. Longer gaps are flagged as missing data.

**Step 2 — Normalization**  
Each astronaut's data is normalized against their own personal baseline (first 14 days of mission), not a population average. This personalized normalization is the core mechanism that enables individual-specific anomaly detection.

**Step 3 — Window aggregation**  
Data is aggregated into three time windows for different model inputs:
- **5-minute windows** — used for real-time acute alert detection
- **1-hour windows** — used for trend modeling
- **24-hour windows** — used for daily health summary and long-term drift detection

**Step 4 — Feature engineering**  
The following derived features are computed from raw signals:
- HR/HRV ratio (autonomic balance index)
- SpO2 rolling mean and standard deviation (5-min window)
- Sleep debt accumulation (rolling 7-day)
- Cognitive performance trend slope (7-day linear regression)
- Mission phase flag (launch, transit, surface operations, return)

---

## 3. AI Model Architecture

### 3.1 Layer 1 — Rule-Based Hard Thresholds (Always Active)

Before any ML model runs, a rule-based layer checks for immediately life-critical values. These rules fire independently of all other models and generate a CRITICAL alert with no delay.

| Parameter | Critical Threshold | Action |
|-----------|-------------------|--------|
| SpO2 | < 90% | Immediate CRITICAL alert |
| Heart Rate | > 160 bpm at rest | Immediate CRITICAL alert |
| Heart Rate | < 35 bpm | Immediate CRITICAL alert |
| Cabin CO2 | > 5000 ppm | Environmental CRITICAL alert |
| Radiation Dose | > 1 mSv in 24 hours | Radiation CRITICAL alert |

Rationale: ML models require processing time and can occasionally produce incorrect outputs. For values that represent immediate danger to life, deterministic rules provide a guaranteed safety net.

### 3.2 Layer 2 — Isolation Forest (Unsupervised Anomaly Detection)

**Algorithm:** Isolation Forest (Liu et al., 2008)  
**Input:** 5-minute aggregated multivariate feature vector (12 features)  
**Output:** Anomaly score (0.0 to 1.0); scores above 0.72 trigger investigation

**How it works:**  
Isolation Forest builds an ensemble of random decision trees. Normal data points require many splits to isolate; anomalous points are isolated with very few splits. The algorithm assigns an anomaly score based on average path length across all trees.

**Why this algorithm:**  
Isolation Forest does not require labeled training data (no historical anomaly examples are needed). It is computationally efficient enough to run in near-real-time on embedded hardware. It handles high-dimensional data well, which is necessary for multi-sensor fusion.

**Configuration:**
- Number of trees: 200
- Subsample size: 256
- Contamination parameter: 0.05 (assumes 5% of training windows may contain mild anomalies)
- Retrained monthly using the most recent 30 days of each astronaut's data

### 3.3 Layer 3 — LSTM Autoencoder (Temporal Anomaly Detection)

**Algorithm:** Long Short-Term Memory (LSTM) Autoencoder  
**Input:** 1-hour time-series sequences (12 features × 720 time steps at 0.2 Hz)  
**Output:** Reconstruction error (MSE); errors above the 99th percentile of training distribution trigger an alert

**How it works:**  
The LSTM Autoencoder learns to compress and reconstruct normal physiological time-series patterns. During training, it sees only normal (baseline) data. When an anomalous pattern occurs — such as an irregular cardiac rhythm or unusual SpO2 trajectory — the model cannot reconstruct it accurately, producing high mean squared error. High reconstruction error signals an anomaly.

**Architecture:**
```
Encoder:
  LSTM Layer 1: 128 units, return_sequences=True
  LSTM Layer 2: 64 units, return_sequences=False
  Dense bottleneck: 32 units

Decoder:
  RepeatVector: 720 steps
  LSTM Layer 3: 64 units, return_sequences=True
  LSTM Layer 4: 128 units, return_sequences=True
  TimeDistributed Dense: 12 outputs (one per feature)
```

**Training:**  
- Trained on first 14 days of mission data (baseline establishment period)
- Fine-tuned monthly using rolling window of recent normal data
- Training time estimate: ~45 minutes on embedded GPU

**Why this algorithm:**  
LSTM networks excel at learning temporal dependencies in sequential data. Physiological signals are inherently time-dependent — a heart rate of 90 bpm is normal during exercise but abnormal at rest. The LSTM Autoencoder captures these contextual patterns that static models miss.

### 3.4 Layer 4 — Random Forest Classifier (Risk Classification)

**Algorithm:** Random Forest Classifier  
**Input:** Anomaly scores from Layers 2 and 3 + feature vector + mission context flags  
**Output:** Risk class label (LOW / MEDIUM / HIGH / CRITICAL) + top contributing features

**How it works:**  
When Layers 2 or 3 flag an anomaly, the Random Forest Classifier determines the severity and likely type of the anomaly. It is trained on labeled historical data from Earth-based clinical datasets (PhysioNet MIMIC-III) and augmented with annotated ISS mission data from the NASA LSDA.

**Output classes:**

| Class | Meaning | Crew Action |
|-------|---------|-------------|
| LOW | Minor deviation, within expected variation | Log and monitor |
| MEDIUM | Moderate anomaly, warrants crew check-in | Self-assessment + report to mission control |
| HIGH | Significant anomaly, possible medical issue | Medical countermeasure protocol + immediate Earth contact |
| CRITICAL | Life-threatening values detected | Emergency protocol activation |

**Configuration:**
- Number of trees: 500
- Max depth: 12
- Class weighting: Balanced (oversampled minority classes)
- Feature importance tracked for explainability output

### 3.5 Ensemble Decision Logic

The four layers combine using the following logic:

```
IF any Layer 1 rule fires:
    → Output: CRITICAL alert immediately

ELSE IF (Isolation Forest score > 0.72) OR (LSTM reconstruction error > 99th pctl):
    → Pass to Random Forest classifier
    → Output: risk class from classifier

ELSE:
    → No alert; update daily health summary
```

---

## 4. Output and Alert System

### 4.1 Alert Types

| Alert Level | Delivery Method | Who Sees It |
|-------------|----------------|-------------|
| CRITICAL | Audio alarm + wearable vibration + screen | Crew immediately |
| HIGH | Screen notification + mission control telemetry | Crew + Earth |
| MEDIUM | Screen notification (next check-in) | Crew |
| LOW | Logged to daily health report | Crew (daily review) |

### 4.2 Explainability Output

Every alert includes a plain-language explanation. Example:

> "SpO2 has been declining gradually over the past 6 hours (current: 94%, baseline: 98%). Heart rate variability also decreased 22% below your 7-day average. This pattern is consistent with early-stage respiratory fatigue. Recommended action: perform lung capacity self-test from the medical kit."

This is generated by a template-based system populated with the top 3 contributing features identified by the Random Forest's feature importance scores.

### 4.3 Daily Health Summary

Every 24 hours, the system generates a report containing:
- Health status dashboard (all parameters vs. personal baseline)
- Anomaly log for the past 24 hours
- 7-day trend graphs for key metrics
- Sleep quality score and recommendations
- Cumulative radiation dose and remaining mission dose budget

---

## 5. Ethical Considerations

### 5.1 Privacy
Biometric data is stored only on the spacecraft's local system and transmitted to Earth only during scheduled downlink windows. Data is not shared beyond the mission medical team.

### 5.2 Algorithmic Bias
Population-level health data used to train the supervised classifier may underrepresent certain demographic groups. Mitigation: personalized baseline normalization reduces dependence on population averages. All models are validated separately across demographic subgroups.

### 5.3 Over-reliance
Crew members must be trained to understand that AI alerts are probabilistic recommendations, not diagnoses. The system is designed to support — not replace — crew medical judgment and Earth-based physician oversight.

### 5.4 Failure Modes
The system must fail safely. If any model layer becomes unavailable (hardware failure, software crash), the system reverts to the rule-based threshold layer only and notifies the crew of degraded monitoring status.

---

## 6. System Requirements

| Requirement | Specification |
|-------------|--------------|
| Processing hardware | Embedded GPU (NVIDIA Jetson-class or equivalent) |
| Memory | 16 GB RAM minimum |
| Storage | 500 GB solid-state (2 years of sensor data) |
| Power consumption | < 50W continuous |
| Alert latency (CRITICAL) | < 5 seconds from threshold breach |
| Alert latency (HIGH/MEDIUM) | < 60 seconds from anomaly detection |
| Uptime requirement | 99.9% (< 8.7 hours downtime per year) |

---

## 7. References

- Liu, F. T., Ting, K. M., & Zhou, Z. H. (2008). Isolation forest. *2008 Eighth IEEE International Conference on Data Mining.*
- Hochreiter, S., & Schmidhuber, J. (1997). Long short-term memory. *Neural Computation*, 9(8), 1735-1780.
- Breiman, L. (2001). Random forests. *Machine Learning*, 45(1), 5-32.
- NASA Life Sciences Data Archive: https://lsda.jsc.nasa.gov
- PhysioNet MIMIC-III: https://physionet.org/content/mimiciii/
