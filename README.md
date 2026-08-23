# Supercapacitor-Assisted Hybrid Energy Management System for DC Microgrids

A **20 W battery-less DC microgrid** designed for residential DC applications using **solar PV generation, supercapacitor energy storage, and automatic AC-grid backup**.

## 📌 Project Overview

This project presents the design and implementation of a **supercapacitor-assisted DC microgrid** for residential applications. The system is designed to supply a regulated ** DC bus** to DC loads while reducing the dependence on conventional batteries.

Solar energy is processed through a **synchronous buck converter with MPPT control**, while a supercapacitor bank provides short-term energy storage and buffering. When the available solar and stored energy are insufficient to maintain the required DC-bus voltage, the system automatically switches to **AC-grid backup**.

The system eliminates battery-related issues such as degradation, limited cycle life, and replacement costs, making it suitable for applications requiring frequent charge/discharge cycles.

---

## ⚡ System Architecture

```text
                    Solar PV
                       │
                       ▼
            ┌─────────────────────┐
            │ Synchronous Buck    │
            │ Converter + MPPT    │
            │ ESP32 Controller    │
            └──────────┬──────────┘
                       │
                       ▼
              ┌────────────────┐
              │ Supercapacitor │
              │     Bank       │
              │ 6 × 2.7 V      │
              └───────┬────────┘
                      │
                      ▼
              ┌────────────────┐
              │ XL4016 Buck    │
              │   Converter    │
              └───────┬────────┘
                      │
                      ▼
                 ┌─────────┐
                 │ 12 V DC │
                 │   BUS   │
                 └────┬────┘
                      │
          ┌───────────┼───────────┐
          │           │           │
          ▼           ▼           ▼
       DC Fan      DC Light    USB/PD
                                  │
                                Laptop/
                                Phone

        AC Grid ───────► Grid Backup
                         │
                         ▼
                      12 V DC Bus
```

---

## 🔋 Key Features

* Battery-less energy storage using supercapacitors
* Solar PV energy harvesting
* Incremental Conductance MPPT algorithm
* Synchronous buck converter
* ESP32-based digital control
* Regulated 12 V DC distribution bus
* Automatic AC-grid fallback
* Voltage and current monitoring
* Over-current protection
* Under-voltage protection
* Supercapacitor over-voltage protection
* Duty-cycle matched converter restart
* Suitable for frequent charge/discharge operation

---

## 🛠️ Hardware Components

| Component           | Function                           |
| ------------------- | ---------------------------------- |
| ESP32               | Main controller and MPPT control   |
| Solar PV Panel      | Renewable energy source            |
| IR2104              | High/low-side MOSFET gate driver   |
| IRLZ44N MOSFETs     | Synchronous buck switching devices |
| Supercapacitor Bank | Energy storage and buffering       |
| XL4016              | DC-DC buck conversion              |
| INA219              | Voltage/current monitoring         |
| DC Loads            | Demonstration residential loads    |
| AC Grid Supply      | Backup energy source               |

### Supercapacitor Bank

The energy storage stage uses:

**6 × 2.7 V supercapacitors connected in series**

This provides a nominal maximum voltage of approximately:

```text
6 × 2.7 V = 16.2 V
```

The supercapacitor bank is therefore operated within a controlled voltage range to protect the storage elements and maintain stable DC-bus operation.

---

## 🧠 Control System

The ESP32 continuously monitors:

* PV input voltage
* PV input current
* Supercapacitor voltage
* DC-bus voltage
* Output current

The controller uses this information to determine the appropriate operating state of the system.

### MPPT Algorithm

An **Incremental Conductance (INC)** MPPT algorithm is implemented to extract maximum available power from the solar PV panel.

The algorithm determines the operating point of the PV panel by comparing:

```text
dI/dV
```

with:

```text
-I/V
```

The PWM duty cycle of the synchronous buck converter is then adjusted accordingly.

---

## 🔄 Energy Management

The energy management system operates according to the supercapacitor and DC-bus conditions.

### Normal Operation

When sufficient solar energy is available:

```text
Solar PV → MPPT Buck → Supercapacitor/DC Bus → Loads
```

### Energy Deficit

When solar generation is insufficient:

```text
Supercapacitor → DC Bus → Loads
```

### Low DC-Bus Voltage

If the DC-bus voltage falls below the defined threshold:

```text
AC Grid → Backup Converter → 12 V DC Bus
```

This provides uninterrupted operation of the connected DC loads.

---

## 🛡️ Protection System

Several protection mechanisms are implemented in the ESP32 firmware.

### Over-Current Protection

The system monitors PV and output currents and disables the converter when the current exceeds the defined safety limit.

### Supercapacitor Over-Voltage Protection

The supercapacitor bank is prevented from exceeding its maximum allowable operating voltage.

### Under-Voltage Protection

The converter is disabled when the supercapacitor voltage falls below the safe operating region.

### Solar Input Protection

The controller checks the PV input voltage before enabling the converter.

---

## 💻 Firmware

The control firmware is developed using the **Arduino framework for ESP32**.

### Main Firmware Functions

```text
PV Voltage Measurement
        ↓
PV Current Measurement
        ↓
Power Calculation
        ↓
Incremental Conductance MPPT
        ↓
PWM Duty-Cycle Control
        ↓
Protection Monitoring
        ↓
Energy Management State Machine
```

### Main Control States

```text
ST_INIT
   │
   ▼
ST_NORMAL
   │
   ├── Supercapacitor Over-Voltage
   │          ↓
   │      ST_OV_HOLD
   │
   ├── Supercapacitor Under-Voltage
   │          ↓
   │      ST_UV_PROT
   │
   └── Abnormal Condition
              ↓
          ST_FAULT
```

---

## 📊 Main System Specifications

| Parameter                     | Specification             |
| ----------------------------- | ------------------------- |
| System Type                   | Battery-less DC Microgrid |
| Rated Power                   | 20 W                      |
| DC Bus Voltage                | 12 V                      |
| Energy Storage                | Supercapacitor            |
| Supercapacitor Configuration  | 6 × 2.7 V                 |
| MPPT Method                   | Incremental Conductance   |
| Controller                    | ESP32                     |
| PWM Frequency                 | 60 kHz                    |
| MPPT Voltage Range            | 13.5–16 V                 |
| Maximum SC Protection Voltage | 16.2 V                    |
| Normal SC Operating Range     | Approximately 9.5–13.2 V  |
| DC-DC Converter               | Synchronous Buck + XL4016 |
| Monitoring                    | INA219                    |
| Backup Source                 | AC Grid                   |

---


## 🔬 Experimental Validation

The prototype was developed and tested to evaluate:

* Solar PV power extraction
* MPPT performance
* Supercapacitor charging and discharging
* DC-bus voltage regulation
* Load operation
* Automatic grid backup
* Converter protection
* System response under changing solar conditions

The experimental results demonstrate the feasibility of using **supercapacitors as an alternative short-term energy-storage technology for low-power DC microgrids**.

---

## 🌱 Applications

The proposed system can be adapted for:

* Residential DC microgrids
* Solar-powered DC homes
* DC lighting systems
* USB charging systems
* Low-power DC appliances
* IoT and embedded systems
* Renewable-energy-based backup systems

---

## 🚀 Future Improvements

Future development can focus on:

* Bidirectional supercapacitor converter
* Active cell balancing for the supercapacitor bank
* Improved MPPT efficiency
* Higher-power prototype
* IoT-based remote monitoring
* Real-time energy-management dashboard
* Advanced power-flow optimization
* PCB-based integrated power stage
* Improved grid/PV transition control

---

## 👨‍💻 Project

**Project Title:**
*Design and Implementation of Supercapacitor-Assisted Hybrid Energy Management System for DC Microgrids in Residential Applications*

**Developed using:** ESP32 • Power Electronics • MPPT • Supercapacitor Energy Storage • DC Microgrid • Embedded Control

---

## 📜 License

This project is intended for **academic, educational, and research purposes**.
