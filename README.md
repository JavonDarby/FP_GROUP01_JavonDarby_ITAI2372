# FP_GROUP01_Javon_Darby_ITAI2372
## AI-Powered Astronaut Health Monitoring System

**Course:** ITAI 2372  
**Track:** Conceptual Design Track  
**Submission Date:** April 26, 2025  

---

## Student Information
- **Name:** Javon Darby
- **Group:** Group 01

---

## Project Description

This project proposes an AI-powered health monitoring system designed for astronauts on long-duration space missions. The system uses an ensemble of machine learning models to continuously analyze biometric sensor data — including heart rate, blood oxygen (SpO2), heart rate variability, skin temperature, and cognitive performance scores — and detects physiological anomalies in real time before they become medical emergencies.

The core challenge this project addresses is that during missions to the Moon or Mars, Earth-based physicians face communication delays of up to 20 minutes one-way. An autonomous AI system can act as a first-responder layer, flagging critical health events without waiting for ground control input.

---

## Project Track: Conceptual Design

This submission follows the **Conceptual Design Track**, which includes:

1. A detailed project proposal
2. A full AI solution design plan
3. A comprehensive testing plan
4. A final presentation (PDF slides)

No executable code is required for this track. All documentation is written to a level of detail sufficient for a development team to implement the system.

---

## How to Read This Documentation

Navigate the repository folders in this order:

| Folder | File | Description |
|--------|------|-------------|
| `/proposal` | `project_proposal.md` | Problem statement, objectives, and AI justification |
| `/design` | `solution_plan.md` | Full AI architecture, data pipeline, and model design |
| `/testing` | `testing_plan.md` | Testing methodology, metrics, and risk analysis |
| `/presentation` | `slides.pdf` | Final presentation slides (PDF) |

Start with the **proposal**, then move to the **design**, then **testing**, then review the **slides** for a visual summary.

---

## Project Structure

```
FP_GROUP01_Javon_Darby_ITAI2372/
├── README.md                  ← You are here
├── proposal/
│   └── project_proposal.md   ← Project Proposal (20 pts)
├── design/
│   └── solution_plan.md      ← Main Design Document (40 pts)
├── testing/
│   └── testing_plan.md       ← Testing Plan (25 pts)
└── presentation/
    └── slides.pdf            ← Final Slides (15 pts)
```

---

## Key Technologies Referenced

- **Isolation Forest** — unsupervised anomaly detection
- **LSTM Autoencoder** — temporal sequence learning
- **Random Forest Classifier** — risk-level classification
- **NASA Life Sciences Data Archive (LSDA)** — reference dataset
- **Python / scikit-learn / TensorFlow** — proposed implementation stack

---

## References

- NASA Life Sciences Data Archive: https://lsda.jsc.nasa.gov
- NASA Human Research Program: https://humanresearchroadmap.nasa.gov
- ESA Space Medicine: https://www.esa.int/Science_Exploration/Human_and_Robotic_Exploration/Research
