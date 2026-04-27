# Project Proposal
## AI-Powered Astronaut Health Monitoring System

**Course:** ITAI 2372  
**Track:** Conceptual Design Track  
**Student:** Javon Darby  
**Group:** Group 01  
**Date:** November 2024  

---

## 1. Problem Statement

Long-duration spaceflight poses serious, well-documented risks to human health. Astronauts aboard the International Space Station (ISS) and future missions to the Moon and Mars face:

- **Cardiovascular deconditioning** — the heart adapts to microgravity and weakens over time
- **Immune system suppression** — stress and radiation reduce the body's ability to fight infection
- **Cognitive degradation** — sleep disruption, isolation, and elevated CO2 impair decision-making
- **Bone density loss** — skeletal deterioration at a rate of roughly 1–2% per month in microgravity
- **Radiation exposure** — increases cancer risk and can cause acute radiation sickness

Currently, astronaut health is monitored through periodic check-ins with Earth-based flight surgeons and manual review of biometric logs. This approach has two critical weaknesses:

1. **Communication delay:** A one-way signal to Mars takes between 3 and 22 minutes depending on orbital position. During a medical emergency, this delay can be fatal.
2. **Data volume:** Modern wearable sensors generate thousands of data points per hour. Manual review cannot detect subtle, gradual deterioration patterns that develop over days or weeks.

There is a clear need for an **autonomous, on-board AI system** that continuously monitors astronaut health data, detects anomalies early, and alerts crew members without requiring Earth-based intervention.

---

## 2. Project Objectives

This project aims to design — at a conceptual level — an AI health monitoring system with the following objectives:

1. **Real-time anomaly detection** — identify physiological readings that deviate significantly from an individual astronaut's personal baseline
2. **Multi-modal data fusion** — combine data from multiple sensor types (biometric, environmental, cognitive) into a unified health picture
3. **Risk stratification** — classify detected anomalies into risk tiers (low / medium / high / critical) to prioritize crew response
4. **Low false-positive rate** — distinguish true anomalies from normal physiological variation caused by exercise, sleep, or EVA (spacewalk) activity
5. **Explainability** — provide crew and mission control with a clear, human-readable explanation of why an alert was triggered

---

## 3. Proposed AI Approach

This system will use a **multi-model ensemble architecture** combining:

- **Unsupervised anomaly detection** (Isolation Forest) for detecting novel, unexpected patterns without labeled training data
- **Deep learning time-series modeling** (LSTM Autoencoder) for learning normal physiological rhythms and flagging high reconstruction error
- **Supervised classification** (Random Forest) for categorizing detected anomalies by type and severity
- **Rule-based threshold layer** for hard medical limits (e.g., SpO2 below 90%) that require immediate alerting regardless of model output

The rationale for this ensemble approach is that no single algorithm performs well across all health monitoring scenarios. Unsupervised models detect unknown anomalies; supervised models excel when labeled historical data is available; rule-based systems provide guaranteed response to life-critical thresholds.

---

## 4. Data Sources

This project will reference the following publicly available datasets for design validation:

| Dataset | Source | Description |
|---------|--------|-------------|
| NASA Life Sciences Data Archive (LSDA) | NASA JSC | Biometric data from ISS astronauts |
| NASA GeneLab | NASA | Omics and physiological data from spaceflight studies |
| PhysioNet MIMIC-III | MIT | Clinical ICU biometric time-series (Earth analog) |
| NASA Human Research Roadmap | NASA HRP | Risk framework and countermeasure data |

No proprietary or restricted data is required for this conceptual design.

---

## 5. Why AI Is the Right Solution

Traditional rule-based health monitoring systems use fixed thresholds (e.g., "alert if heart rate exceeds 100 bpm"). These systems fail in spaceflight for two reasons:

1. **Individual variation** — a trained astronaut's resting heart rate may naturally be 45 bpm; a fixed threshold calibrated for the general population will generate constant false alarms or miss real problems.
2. **Pattern complexity** — early-stage cardiovascular deconditioning manifests as subtle, multi-variable trends across heart rate variability, blood pressure, and activity level simultaneously. No single threshold captures this.

Machine learning models can learn each astronaut's personal physiological baseline and detect multivariate deviations that no rule-based system could anticipate. This is precisely the type of problem AI is best suited for.

---

## 6. Scope and Limitations

**In scope:**
- System architecture design
- Algorithm selection and justification
- Data pipeline design
- Alert and output design
- Testing and validation methodology
- Ethical considerations

**Out of scope (for this conceptual design):**
- Hardware sensor selection and integration
- Flight certification and NASA safety review process
- Real-time embedded systems programming
- Clinical trials with human subjects

---

## 7. Expected Outcomes

Upon completion, this project will deliver:

1. A complete conceptual design document for the AI health monitoring system
2. A testing plan with defined success metrics
3. A risk analysis covering technical failures, ethical concerns, and bias risks
4. A presentation suitable for a technical audience

---

## 8. References

- Williams, D., et al. (2009). Acclimation during space flight: effects on human physiology. *CMAJ*, 180(13).
- NASA Human Research Program. (2023). *Evidence Report: Risk of Adverse Health Outcomes and Decrements in Performance Due to In-Flight Medical Conditions.*
- Liu, F. T., Ting, K. M., & Zhou, Z. H. (2008). Isolation forest. *ICDM 2008.*
- Hochreiter, S., & Schmidhuber, J. (1997). Long short-term memory. *Neural Computation*, 9(8).
