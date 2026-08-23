<div align="center">

# Supercapacitor-Assisted DC Microgrid for DC Homes

*A battery-less residential DC microgrid using supercapacitor energy storage and autonomous grid-assisted backup*

Final Year Research Project · B.Sc. (Hons) Electronics and Computer Science
Department of Physics and Electronics, Faculty of Science, University of Kelaniya, Sri Lanka

</div>

---

## Contents

- [About this project](#about-this-project)
- [Why supercapacitors, not batteries](#why-supercapacitors-not-batteries)
- [How the system works](#how-the-system-works)
- [Hardware](#hardware)
- [Operating modes](#operating-modes)
- [What we found](#what-we-found)
- [Repository layout](#repository-layout)
- [Running the firmware](#running-the-firmware)
- [Authors and supervision](#authors-and-supervision)
- [Citing this work](#citing-this-work)
- [License](#license)
- [Thanks](#thanks)

---

## About this project

Most small-scale residential DC microgrids that use solar power still lean on a chemical battery for local energy storage, because that's the conventional, well-understood way to do it. But batteries come with real costs attached — a limited number of charge cycles before they degrade, sensitivity to heat, the need for periodic replacement, and an environmental footprint at end of life that nobody particularly wants to think about.

This project asks a more specific question: does a residential DC microgrid actually *need* a battery, or can a supercapacitor bank do that job on its own, provided there's an intelligent backup path for the times when solar and stored energy both fall short? We built a working prototype to find out — a photovoltaic source, a synchronous buck converter running Incremental Conductance MPPT, a supercapacitor bank as the only local storage, and a relay-controlled grid-assist supply, all coordinated in real time by an ESP32. It's not a simulation. The hardware exists, we instrumented it, and we measured how it actually performs.

The short version of what we found: it works, and it works efficiently. The complete system reached a peak end-to-end efficiency of 81.42%, with the supercapacitor bank consistently improving performance over the unbuffered configuration.

## Why supercapacitors, not batteries

Supercapacitors and batteries aren't really substitutes for each other in the general case — they're good at different things. A supercapacitor can charge and discharge in milliseconds and survive hundreds of thousands of cycles without meaningful degradation, but it stores far less energy per unit of weight or volume than a lithium-ion cell. That tradeoff is exactly why most of the existing literature treats supercapacitors as a *supplement* to a battery — something that absorbs transient spikes so the battery doesn't have to — rather than a replacement for one.

What we wanted to test was whether that assumption holds up at the scale of a single residential DC node. If the supercapacitor bank is sized appropriately and the system has a grid connection to fall back on during extended shortfalls — a cloudy week, or several nights in a row — does the battery actually buy you anything a supercapacitor-plus-grid combination can't? Our results suggest, at least for this scale and this load profile, that it doesn't.

## How the system works

<div align="center">
<img src="block_diagram.jpg" alt="System block diagram" width="520">

<sub>Block diagram of the supercapacitor-assisted DC microgrid.</sub>
</div>

Solar power comes in through a synchronous buck converter, which does two things at once: it regulates the variable panel voltage into a stable DC bus, and it charges the supercapacitor bank. An ESP32 sits at the center of the whole system, reading photovoltaic voltage and current, supercapacitor voltage, bus voltage, and load current through a pair of INA219 sensors, and using those readings to decide — moment to moment, with no human involved — which of three operating modes the system should be in.

<div align="center">
<img src="media/pcb_labeled.jpg" alt="Labeled PCB" width="600">

<sub>The assembled prototype board, with the main functional blocks labeled.</sub>
</div>

## Hardware

| Subsystem | What we used |
|---|---|
| Microcontroller | ESP32 |
| Power stage | Synchronous buck converter — IR2104 gate driver, IRLZ44N MOSFETs, 220 µH inductor |
| Energy storage | Six 2.7 V / 400 F supercapacitor cells in series |
| Grid backup | HW-803 optoisolated relay, 24 V / 2 A backup supply |
| Telemetry | Two INA219 current/voltage sensors over I²C |
| Display | 20×4 character LCD, showing live mode, duty cycle, and rail voltages/currents |

## Operating modes

The firmware moves between three states automatically, based on supercapacitor voltage and how much load the system is carrying:

| Mode | When it's active | What happens |
|---|---|---|
| **PV Priority** | Supercapacitor voltage is in its normal range and there's enough sun | Solar runs the load directly and tops up the supercapacitor bank; the grid path stays disconnected |
| **Supercapacitor Standalone** | The bank is fully charged | The supercapacitor alone carries the load until it drains to a lower threshold |
| **Grid Assist** | Supercapacitor voltage drops to a critical low point | The relay brings in the 24 V grid-backed supply, which powers the load and recharges the bank until things normalize |

None of these transitions require anyone to flip a switch — the ESP32 handles it based on continuous voltage and load monitoring.

## What we found

We tested the system in three stages, each one closer to the full real-world configuration:

| Configuration | Peak efficiency | Test condition |
|---|---|---|
| Buck converter alone | 80.34% | Fixed 12 V output, input swept 16–24 V |
| System with PV, no supercapacitor | 78.92% | Direct solar input, MPPT active |
| **Full system — PV + supercapacitor** | **81.42%** | Complete proposed architecture |

<div align="center">
<img src="media/efficiency_buck.jpg" alt="Buck converter efficiency" width="380">&nbsp;&nbsp;
<img src="media/efficiency_system.jpg" alt="System efficiency" width="380">
</div>

The supercapacitor consistently added 2–3 percentage points of efficiency over the unbuffered setup across the whole voltage range we tested — most likely because the bank absorbs a good deal of the switching-frequency ripple current that would otherwise show up as loss further downstream. Beyond the raw efficiency numbers, the thing we were watching closest was whether the mode transitions actually held up under real conditions, and they did: stable bus regulation, MPPT that tracked correctly, and no dropouts moving between modes — all without a battery anywhere in the system.



## Running the firmware

You'll need an ESP32 board, the Arduino IDE (or PlatformIO) with the ESP32 board package installed, and libraries for I²C, the INA219 sensors, and the LCD.

```bash
git clone https://github.com/<your-username>/supercapacitor-dc-microgrid.git
cd supercapacitor-dc-microgrid/firmware
# open in Arduino IDE or PlatformIO, select your ESP32 board, upload
```

The full converter design math, the MPPT algorithm derivation, and the complete experimental writeup are in `docs/thesis.pdf` if you want the details behind the numbers above.

## Authors and supervision

This work was carried out by **J. S. Jenif** (EC/2021/050) and **Y. Thuvarakan** (EC/2021/065), final-year Electronics and Computer Science students at the University of Kelaniya, under the supervision of:

- **Dr. P. L. A. K. Piyumal**, PhD (Kelaniya) — Principal Supervisor
- **Senior Prof. A. L. A. K. Ranaweera**, PhD (Kyung Hee) — Co-Supervisor
- **Senior Prof. Sudath R. D. Kalingamudali**, PhD (Sheffield) — Co-Supervisor

## Citing this work

```bibtex
@misc{jenif_thuvarakan_2026_dcmicrogrid,
  author       = {Jenif, J. S. and Thuvarakan, Y.},
  title        = {Supercapacitor-Assisted DC Microgrid for DC Homes},
  year         = {2026},
  note         = {Final Year Dissertation, Department of Physics and Electronics,
                   Faculty of Science, University of Kelaniya, Sri Lanka},
  howpublished = {\url{https://github.com/<your-username>/supercapacitor-dc-microgrid}}
}
```

## License

Released under the [MIT License](LICENSE), unless noted otherwise.

## Thanks

We're grateful to Dr. P. L. A. K. Piyumal for his guidance throughout this project, and to Senior Professor A. L. A. K. Ranaweera and Senior Professor Sudath R. D. Kalingamudali for their feedback along the way. Thanks also to the academic staff, lab instructors, and technical officers in the Department of Physics and Electronics who helped with the hardware side of things.
