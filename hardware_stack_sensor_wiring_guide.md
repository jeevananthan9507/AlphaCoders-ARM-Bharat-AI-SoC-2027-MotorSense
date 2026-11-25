# Hardware Stack & Sensor Wiring Guide

This guide outlines the target edge hardware architecture represented in the **MotorSense** 3D digital twin, including sensor mounting locations, signal interfaces to the Raspberry Pi 4B/5 cabinet, and safety precautions.

---

## Target Hardware Stack

| Subsystem | Component | Interface / Mounting | Role in MotorSense |
| :--- | :--- | :--- | :--- |
| **Edge Compute** | **Raspberry Pi 4B or Raspberry Pi 5** (with aluminum heatsink) | DIN-rail / brass standoff mount inside open-front control cabinet | Runs real-time digital filtering, 1024-pt FFT, kurtosis calculation, adaptive baseline model, and local logging. |
| **Data Acquisition** | **Industrial SPI / I2C ADC HAT** with pluggable screw terminals | 40-pin GPIO ribbon/header connection to Raspberry Pi | Converts analog current/temperature signals and interfaces high-speed SPI MEMS accelerometer data. |
| **Vibration Sensor** | **ADXL1002 MEMS Accelerometer** (single/tri-axial industrial housing) | Magnetic or threaded stud mount on motor drive-end bell | Captures high-bandwidth radial casing vibration for RMS, kurtosis, and FFT fault detection. |
| **Load Sensor** | **Split-Core Hall-Effect Current Sensor** ($5\text{–}25\text{ A}$ range) | Clamped around one insulated phase conductor outside terminal box | Provides real-time electrical load context without breaking the motor power circuit. |
| **Speed Sensor** | **Optical or Hall-Effect RPM Tachometer** | Rigid bracket aimed at reflective/magnetic shaft index strip | Measures shaft rotational speed ($\text{RPM}$) and establishes the $1\times$ frequency phase reference. |
| **Thermal Sensor** | **PT100 RTD Temperature Probe** | Threaded or bonded onto gearbox / drive-end bearing housing | Tracks bearing thermal trends and detects lubrication or cooling anomalies. |
| **Local HMI & Alerts** | **Cabinet OLED Status Display + 3-Color LED Signal Tower** | I2C display bus + GPIO relay/driver outputs | Displays live RMS, baseline, RPM, temperature, and status (`NORMAL` / `WARNING` / `CRITICAL`) offline. |

---

## Sensor Mounting & Wiring Notes

### 1. Accelerometer Mounting (`ADXL1002`)
- Mount as close as practical to the **drive-end bearing housing** on a clean, flat metallic surface to preserve high-frequency transmission (`> 200 Hz`) needed for bearing fault detection.
- Avoid mounting on thin sheet-metal fan cowls or flexible brackets that introduce structural resonances.
- Use shielded twisted-pair cable and strain-relief clamps along the motor skid before entering the control cabinet.

### 2. Split-Core Hall Current Sensor
- Clamp around a **single insulated phase conductor** (not the entire 3-phase jacket, where magnetic fields cancel out).
- Verify arrow orientation on the sensor core matches supply-to-motor current flow and terminate secondary leads into the ADC HAT screw block.

### 3. Shaft RPM Tachometer
- Apply a high-contrast reflective tape strip (for optical sensors) or a balanced ferrous target collar (for Hall/inductive proximity sensors) on an accessible section of the shaft between the coupling shroud and gearbox.
- Secure the sensor bracket rigidly to the skid baseplate so vibration does not cause missed pulses.

### 4. PT100 Temperature Probe
- Install in an existing bearing housing thermowell or secure with thermally conductive epoxy/clamp near the gearbox input bearing.
- Use a 3-wire PT100 configuration on the ADC HAT to compensate for lead-wire resistance over cable runs.

---

## Mechanical & Electrical Safety Precautions

> **WARNING — Rotating Machinery & High Voltage**
> Always follow site Lockout/Tagout (LOTO) procedures before installing sensors, adjusting brackets, or opening electrical enclosures.

1. **Fixed Coupling & Shaft Guarding**: The jaw coupling and eccentric test mass must remain fully enclosed inside a rigid stationary coupling shroud during operation. Never operate the motor with exposed rotating keys, set screws, or unguarded couplings.
2. **De-Energize Before Wiring**: Disconnect and verify zero voltage on the 3-phase motor supply before clamping current transformers or opening the motor terminal box.
3. **Galvanic Isolation & Cable Separation**: Route low-voltage sensor cables (`3.3 V / 5 V` SPI, I2C, and analog signals) physically separate from 3-phase AC power conduits to prevent electromagnetic interference (EMI) and ground loops.
4. **Cabinet Ventilation**: Ensure adequate airflow around the Raspberry Pi heatsink and power supply inside the control cabinet when operating in high-ambient industrial environments.