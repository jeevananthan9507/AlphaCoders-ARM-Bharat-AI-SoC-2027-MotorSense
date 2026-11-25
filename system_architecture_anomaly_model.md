# System Architecture & Context-Aware Anomaly Model

This document describes the end-to-end signal processing pipeline, context-aware baseline formulation, fault classification rules, and Remaining Useful Life (RUL) estimation logic used in **MotorSense**.

---

## End-to-End Data Pipeline

```text
[15 kW 3-Phase Induction Motor + Helical Gearbox]
   ├── ADXL1002 MEMS Accelerometer ──► High-Speed SPI / ADC ──┐
   ├── Split-Core Hall Current Sensor ─► ADC / I2C            ──┼──► Raspberry Pi 4B/5 Edge SBC
   ├── Optical / Hall RPM Tachometer ──► GPIO / Timer         ──┤      ├── 1. Digital Filtering & Windowing
   └── PT100 Bearing Temp Probe ───────► Conditioned ADC      ──┘      ├── 2. Feature Extraction (RMS, Kurtosis, 1024-pt FFT)
                                                                        ├── 3. Context Lookup (Expected Healthy Envelope)
                                                                        ├── 4. Multi-Sensor Fault Classification & Z-Score
                                                                        └── 5. Local OLED / Signal Tower + Telemetry Store
```

### Stage 1: Synchronous Multi-Sensor Acquisition
- **Vibration (`ADXL1002`)**: Captures broad-band radial casing acceleration on the motor drive-end housing.
- **Load Context (`Hall Current`)**: Measures 3-phase supply current ($\text{A}$) to determine instantaneous mechanical torque/load demand.
- **Speed & Phase Reference (`Optical/Hall Tachometer`)**: Tracks shaft rotational speed ($\text{RPM}$) and locks onto the fundamental rotational frequency $f_{1\times} = \text{RPM} / 60\text{ Hz}$.
- **Thermal Context (`PT100`)**: Monitors gearbox/bearing housing temperature ($^\circ\text{C}$).

### Stage 2: Edge Feature Extraction
On the Raspberry Pi 4B/5, sliding time windows are processed into compact diagnostic features:
- **Vibration RMS ($\text{mm/s}$)**: Overall broadband vibrational energy.
- **Kurtosis**: Fourth standardized moment of the vibration waveform ($\approx 3.0$ for Gaussian healthy noise; rises sharply above $4.2$ when localized bearing spalls generate repetitive micro-impacts).
- **1024-Point FFT Spectrum**: Resolves harmonic energy at $1\times$ shaft speed (unbalance), $2\times$ shaft speed (misalignment), and high-frequency bearing resonance bands ($240\text{–}420\text{ Hz}$).

---

## How Load & RPM Context Prevents False Alarms

Instead of comparing measured vibration $v_{\text{actual}}$ against a single static limit, MotorSense computes an expected healthy baseline $v_{\text{base}}(\text{RPM}, I)$ and thermal baseline $T_{\text{base}}(\text{RPM}, I)$:

$$v_{\text{base}}(\text{RPM}, I) = 0.95 + 1.15\,\hat{n} + 0.85\,\hat{i}^{1.15} + \Delta v_{\text{learned}}$$

$$T_{\text{base}}(\text{RPM}, I) = 36.0 + 11.0\,\hat{n} + 16.5\,\hat{i} + \Delta T_{\text{learned}}$$

where $\hat{n}$ and $\hat{i}$ are normalized speed and current indices, and $\Delta v_{\text{learned}}, \Delta T_{\text{learned}}$ are site-specific calibration offsets captured via **Learn Baseline**.

Anomalies are scored using the **context-adjusted deviation** ($\Delta v = v_{\text{actual}} - v_{\text{base}}$) and **baseline Z-score** ($z = \max(0, \Delta v / \sigma)$), ensuring that high-load production shifts do not trigger false alarms while subtle anomalies at low loads are still detected.

---

## Fault States & Classification Rules

MotorSense computes a composite severity index ($0\text{–}100$) combining positive vibration deviation $\Delta v$, excess kurtosis ($\text{kurtosis} > 3.35$), and excess temperature ($\Delta T > 5.0^\circ\text{C}$), yielding a **Health Score** ($\text{Health \%} = 100 - \text{Severity}$):

| Status | Trigger Criteria | Primary Diagnostic Focus | Edge Output |
| :--- | :--- | :--- | :--- |
| **NORMAL** | $\text{Health} \ge 82\%$, $\Delta v < 1.35\text{ mm/s}$, $\text{Kurtosis} < 4.2$, $T < 74^\circ\text{C}$ | Operating within learned load/RPM envelope. Routine lubrication schedule maintained. | Green beacon active; passive drift logging. |
| **WARNING** | $55\% \le \text{Health} < 82\%$, or $\Delta v \ge 1.35\text{ mm/s}$, or $\text{Kurtosis} \ge 4.2$, or $T \ge 74^\circ\text{C}$ | Early-to-moderate rotor imbalance ($1\times$), shaft misalignment ($2\times$), bearing wear, or cooling restriction. | Amber beacon active; preventive parts & inspection scheduled. |
| **CRITICAL** | $\text{Health} < 55\%$, or $\Delta v \ge 3.4\text{ mm/s}$, or $\text{Kurtosis} \ge 6.2$, or $T \ge 88^\circ\text{C}$ | Severe bearing spalling, mechanical looseness, or thermal overload risking unplanned failure. | Flashing red beacon; urgent maintenance window flagged. |

---

## Remaining Useful Life (RUL) & Repair Window Logic

1. **Operating Hours Estimation**:
   - **NORMAL**: $\text{RUL}_{\text{hrs}} \approx 1450 \times (\text{Health} / 100)$
   - **WARNING**: $\text{RUL}_{\text{hrs}} \approx 180 + (\text{Health} - 55) \times 14$
   - **CRITICAL**: $\text{RUL}_{\text{hrs}} \approx \max(12,\; \text{Health} \times 2.4)$
2. **Calendar Conversion**: Operating hours are divided by the machine's configured **Daily Duty Cycle** ($\text{hrs/day}$, default `16 hrs/day`) to project the **Next Scheduled Repair Date** and plot the exponential degradation curve toward the $+4.5\text{ mm/s}$ critical deviation threshold.

---

## Current Simulation Limitations

- **Synthetic Physical Model**: Sensor signals, FFT spectral bins, and RUL curves in `simulation/motorsense-twin.html` are generated by real-time analytical and stochastic equations designed for engineering demonstration and UI validation—not live ADC hardware streams.
- **Browser Persistence**: Telemetry logs in Tab 02 are stored in browser `localStorage` (up to 60 recent records) and should be exported via CSV for external archival.
- **Single-Asset Scope**: The current prototype models one motor-coupling-gearbox drivetrain per session.