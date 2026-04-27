# Testing Plan
## AI-Powered Astronaut Health Monitoring System

**Course:** ITAI 2372  
**Track:** Conceptual Design Track  
**Student:** Javon Darby  
**Group:** Group 01  
**Date:** April 26, 2025  

---

## 1. Overview

This document describes the testing methodology for the Astronaut Health Monitoring AI (AHMAI). Because this is a conceptual design project, all testing plans are described at a specification level — defining what tests would be run, using what data, measured against which success criteria.

Testing is organized into four phases: data validation, model performance evaluation, system integration testing, and ethical/bias auditing.

---

## 2. Phase 1 — Data Validation Testing

### 2.1 Purpose
Verify that the data preprocessing pipeline correctly handles real-world sensor data, including noise, gaps, and edge cases.

### 2.2 Test Cases

| Test ID | Test Description | Input | Expected Output | Pass Criterion |
|---------|-----------------|-------|----------------|----------------|
| DV-01 | Normal sensor stream | 24 hours of clean HR data | Normalized output, no flags | Output values within ±0.01 of expected |
| DV-02 | Short data gap | 45-second gap in SpO2 stream | Gap filled via interpolation | Interpolated values plausible; gap flagged in log |
| DV-03 | Long data gap | 5-minute sensor disconnection | Gap flagged as missing; no interpolation attempted | Missing data flag present in output |
| DV-04 | Artifact spike | Single HR reading of 250 bpm | Spike removed and flagged | Spike absent in cleaned output; artifact count incremented |
| DV-05 | All sensors nominal | Full multi-sensor stream, 7 days | Feature vectors computed correctly | All 12 features present; no NaN values |
| DV-06 | Cold start (no baseline) | First day of mission, no personal baseline yet | System falls back to population norms | Alert generated indicating baseline not yet established |

### 2.3 Data Source
NASA Life Sciences Data Archive (LSDA) — publicly available ISS mission biometric logs used as input for all data validation tests.

---

## 3. Phase 2 — Model Performance Testing

### 3.1 Anomaly Detection Evaluation (Isolation Forest + LSTM Autoencoder)

**Methodology:** Synthetic anomaly injection. A known normal dataset (30 days of healthy astronaut data from LSDA) is used as baseline. Synthetic anomalies are injected at known timestamps, and the system's ability to detect them is measured.

**Anomaly types injected:**

| Anomaly Type | How Injected | Clinical Analog |
|-------------|-------------|----------------|
| SpO2 gradual decline | Reduce SpO2 by 0.5% per hour over 8 hours | Early hypoxia |
| HR irregular pattern | Replace 10 minutes of HR with atrial fibrillation waveform | Cardiac arrhythmia |
| HRV suppression | Reduce HRV by 40% over 48 hours | Chronic stress / overtraining |
| Sleep disruption cascade | Reduce sleep efficiency to <60% for 5 consecutive nights | Sleep deprivation syndrome |
| CO2 cognitive impact | Inject CO2 spike to 3000 ppm; verify cognitive score decline detected | Environmental health event |

**Evaluation Metrics:**

| Metric | Formula | Target |
|--------|---------|--------|
| Precision | TP / (TP + FP) | ≥ 0.85 |
| Recall (Sensitivity) | TP / (TP + FN) | ≥ 0.90 |
| F1 Score | 2 × (Precision × Recall) / (Precision + Recall) | ≥ 0.87 |
| ROC-AUC | Area under receiver operating characteristic curve | ≥ 0.92 |
| Mean Time to Detect | Average time from anomaly onset to alert generation | ≤ 15 minutes |

**Definition of terms:**
- True Positive (TP): System correctly alerts on an injected anomaly
- False Positive (FP): System alerts when no anomaly was injected (false alarm)
- False Negative (FN): System fails to alert on an injected anomaly (missed detection)

### 3.2 False Positive Rate Testing

False alarms are a critical failure mode in health monitoring. A system that alerts too frequently will cause alert fatigue, and crew will begin ignoring warnings.

**Test scenarios specifically designed to generate false positives:**

| Scenario | Expected System Behavior |
|----------|------------------------|
| Vigorous exercise session (90 min) | No HIGH or CRITICAL alert; activity flag suppresses HR anomaly detection |
| EVA (spacewalk) — elevated HR and SpO2 variation | Alerts suppressed during logged EVA window; post-EVA recovery monitored |
| Sleep — low HR and reduced movement | No alert; sleep state correctly classified |
| Meal consumption — temporary HR increase | No alert |
| High-intensity training day | No alert; training log consulted |

**Target:** False positive rate during exercise/sleep/EVA windows < 5%

### 3.3 Risk Classification Evaluation (Random Forest)

**Methodology:** Using the annotated MIMIC-III clinical dataset, classify 1,000 held-out patient episodes (500 normal, 500 abnormal with labeled severity) and compare classifier output to ground truth labels.

**Confusion Matrix Target:**

|  | Predicted LOW | Predicted MEDIUM | Predicted HIGH | Predicted CRITICAL |
|--|--------------|-----------------|---------------|-------------------|
| **Actual LOW** | ≥ 95% | ≤ 5% | 0% | 0% |
| **Actual MEDIUM** | ≤ 10% | ≥ 85% | ≤ 5% | 0% |
| **Actual HIGH** | 0% | ≤ 10% | ≥ 88% | ≤ 2% |
| **Actual CRITICAL** | 0% | 0% | ≤ 5% | ≥ 95% |

**Critical requirement:** No CRITICAL event may be classified as LOW or MEDIUM. Downgrading a life-threatening event is an unacceptable failure.

---

## 4. Phase 3 — System Integration Testing

### 4.1 Alert Latency Testing

| Alert Type | Maximum Acceptable Latency | Test Method |
|-----------|--------------------------|-------------|
| CRITICAL (rule-based) | 5 seconds | Inject threshold-crossing value; measure time to audio alarm |
| HIGH (ML-detected) | 60 seconds | Inject anomaly at known timestamp; measure time to screen notification |
| MEDIUM | 5 minutes | Verify next scheduled check cycle catches anomaly |

### 4.2 Graceful Degradation Testing

Test that the system continues operating safely when components fail:

| Failure Scenario | Expected Behavior |
|-----------------|------------------|
| LSTM model crashes | System continues with Isolation Forest + rules; crew notified of degraded mode |
| Single sensor failure (one SpO2 sensor) | Alert generated; remaining sensors continue; missing data flagged |
| All ML models offline | Rule-based threshold layer continues operating; crew notified |
| Full system restart | Alert logs preserved; models reload from last checkpoint; no data loss |

### 4.3 Communication Blackout Testing

Simulate a 25-minute Earth communication blackout (Mars transit scenario):
- System continues alerting crew autonomously
- All alerts are logged with timestamps for transmission when communication resumes
- No dependence on Earth-side systems during blackout

---

## 5. Phase 4 — Bias and Ethical Auditing

### 5.1 Demographic Subgroup Analysis

All model performance metrics from Phase 2 are recalculated separately for:
- Biological sex (male / female)
- Age group (25–35 / 36–45 / 46–55)
- Fitness level at mission start (low / moderate / high VO2 max)

**Pass criterion:** No subgroup shows precision or recall more than 8 percentage points below the overall population metric.

### 5.2 Explainability Audit

For a random sample of 100 HIGH and CRITICAL alerts:
- Verify that every alert includes a plain-language explanation
- Verify that the top contributing features cited are physiologically plausible
- Verify that no explanation contains technical jargon inaccessible to a non-specialist crew member

### 5.3 Privacy Compliance Check

- Confirm biometric data is stored only in encrypted local storage
- Confirm no data is transmitted to Earth outside scheduled downlink windows
- Confirm crew members can review and annotate their own health data on request

---

## 6. Acceptance Criteria Summary

The system passes testing and is ready for implementation review if ALL of the following are met:

| Criterion | Target | Priority |
|-----------|--------|----------|
| Anomaly detection precision | ≥ 0.85 | Must pass |
| Anomaly detection recall | ≥ 0.90 | Must pass |
| F1 Score | ≥ 0.87 | Must pass |
| ROC-AUC | ≥ 0.92 | Must pass |
| CRITICAL event downgrade rate | 0% | Must pass |
| False positive rate (exercise/sleep) | < 5% | Must pass |
| CRITICAL alert latency | < 5 seconds | Must pass |
| Demographic subgroup performance gap | < 8 percentage points | Must pass |
| System uptime (graceful degradation) | ≥ 99.9% | Must pass |

---

## 7. Risk Register

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| Model fails on novel astronaut physiology not seen in training | Medium | High | Monthly retraining on mission data; human review of all HIGH/CRITICAL alerts |
| Sensor hardware failure | Medium | Medium | Redundant sensors for critical parameters (SpO2, HR) |
| Alert fatigue from false positives | Medium | High | Strict false positive testing; adjustable sensitivity per astronaut preference |
| Data drift over long mission | High | Medium | Monthly model retraining; drift detection monitoring |
| Adversarial mission stress degrades model accuracy | Low | High | Stress-period performance testing during validation |

---

## 8. References

- Johnson Space Center Medical Operations. (2022). *Crew Health and Performance during Long-Duration Spaceflight.*
- Pedregosa, F., et al. (2011). Scikit-learn: Machine learning in Python. *JMLR*, 12, 2825-2830.
- Johnson, A. E. W., et al. (2016). MIMIC-III, a freely accessible critical care database. *Scientific Data*, 3.
