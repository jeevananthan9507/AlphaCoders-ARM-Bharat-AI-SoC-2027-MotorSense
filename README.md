# AlphaCoders-ARM-Bharat-AI-SoC-2027-MotorSense


Context-Aware Predictive Maintenance Digital Twin for Rotating Machinery

Built for ARM Bharat AI SoC 2027 by AlphaCoders, MotorSense monitors rotating industrial machinery—such as 3-phase induction motors, shaft couplings, and gearboxes—using context-aware baselines so that legitimate increases in mechanical load or rotational speed (RPM) do not trigger false maintenance alarms.

---

# PROBLEM STATEMENT & CONTEXT-AWARE APPROACH

In real industrial duty cycles, motor vibration and bearing/casing temperature naturally rise when a machine runs faster or handles heavier loads. Conventional condition-monitoring setups that rely on static, single-value thresholds often struggle in two ways:

* Setting thresholds too low causes frequent false alarms during peak production shifts or high-load operation.
* Setting thresholds too high delays detection of real mechanical degradation during lighter or variable duty cycles.

MotorSense addresses this by evaluating live sensor readings against a context-aware healthy baseline that accounts for the active operating regime (RPM and motor load current in amps). It tracks deviation from that learned baseline, impulse sharpness (kurtosis), frequency-domain spectral behavior (FFT), and thermal trends to separate normal heavy work from genuine mechanical faults.

---

# REPOSITORY CONTENTS

* Bharat AI SoC 2027 Motorsense_Code.txt
Self-contained interactive 3D digital twin and predictive maintenance web simulation built with HTML, CSS, JavaScript, and Three.js. Includes Tab 01 (3D Digital Twin with interactive camera tour, load/RPM sliders, fault injection, live FFT vs. learned baseline spectrum, multi-sensor sparklines, and Raspberry Pi edge log) and Tab 02 (persistent browser localStorage telemetry store, custom operating-condition simulator, health score, Remaining Useful Life [RUL] and next scheduled repair window estimation, degradation curve, and CSV export).
* system_architecture_anomaly_model.md
Detailed documentation of the end-to-end sensing-to-edge pipeline, how RPM and load current context prevents false alarms, classification rules across NORMAL, WARNING, and CRITICAL states, RUL and repair scheduling logic, and limitations of the current web-based simulation model.
* hardware_stack_sensor_wiring_guide.md
Hardware reference covering the Raspberry Pi 4B/5 edge single-board computer (SBC) and SPI/I2C ADC HAT cabinet setup, local OLED status display and signal beacon tower, sensor mounting and wiring notes (MEMS accelerometer, Hall-effect current sensor, optical/Hall RPM sensor, and PT100 temperature probe), and mechanical/electrical safety precautions.
* LICENSE
MIT License governing reuse, modification, and distribution of the repository.
* README.md
Project overview, problem statement, file guide, usage instructions, and documentation links.

---

# KEY FEATURES

1. Context-Aware Baseline Learning
Compares measured vibration RMS and bearing/casing temperature against expected healthy values for the current RPM and current draw, and allows recalibrating/learning a new healthy baseline for the active operating point.
2. Multi-Sensor & Spectral Fault Correlation
Monitors vibration RMS, 1x shaft rotation frequency, 2x harmonics, high-frequency bearing bands, vibration kurtosis (impulsive spalling/impact indicator), Hall current, shaft RPM, and bearing/casing temperature across modes such as Healthy Baseline, Rotor Imbalance, Bearing Wear, and Shaft Misalignment.
3. Offline-Capable Edge Architecture
Architected around a Raspberry Pi 4B/5 edge node and ADC HAT with local OLED and signal-tower indicators so local monitoring, baseline comparison, and status alerting can operate even without continuous cloud connectivity.
4. Prognostics, Data Store & Repair Scheduling
Provides a persistent log in Tab 02 where users can capture live 3D twin states or enter custom operating parameters, inspect health percentages and RUL estimates, view projected degradation toward a critical threshold, schedule upcoming repair windows, and export data as CSV.

---

# HOW TO RUN & USE THE SIMULATION

1. Download or clone the repository onto your computer.
2. Locate Bharat AI SoC 2027 Motorsense_Code.txt. Save or copy it with a .html extension (for example, motorsense.html), or open the file in a modern desktop web browser (such as Chrome, Edge, or Firefox).
3. Explore Tab 01 (01 · 3D Digital Twin):
* Drag to orbit, scroll to zoom, or use the tour controls to walk through hardware chapters (Overview, Sensing & FFT, Raspberry Pi Edge, and Fault Diagnosis).
* Adjust the Load and RPM controls to observe how normal operating baselines shift.
* Switch Fault modes to inspect changes in the FFT spectrum, kurtosis, health score, edge log, and 3D cabinet indicators.
* Click Learn Baseline to calibrate a new healthy baseline at the current operating point, or save a snapshot to Tab 02.


4. Explore Tab 02 (02 · Data Store & Repair Schedule):
* Capture live states from the 3D twin or use presets / custom inputs (RPM, current, vibration RMS, kurtosis, temperature, daily duty hours, and inspection notes).
* Review the health status, maintenance recommendations, estimated RUL, next scheduled repair date, degradation trajectory chart, and stored table.
* Export stored records as a CSV file or reset sample data when needed.



---

# DOCUMENTATION & NOTES

* System Architecture & Anomaly Model: system_architecture_anomaly_model.md
* Hardware Stack & Sensor Wiring Guide: hardware_stack_sensor_wiring_guide.md

Note: The web simulation uses engineered mathematical/stochastic models for interactive visualization, fault demonstration, and maintenance planning, and stores telemetry records locally in browser localStorage unless exported to CSV.
