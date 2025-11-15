# OPTIMUS V2 — Ghana Robotics Competition 2025 (Smart City Builders Challenge)

**MicroPython-powered robot designed for the Ghana Robotics Competition (Engineers League, Smart City Builders Challenge).**
Built using the **Xplore Bot kit** and a **Raspberry Pi Pico**, OPTIMUS V2 combines autonomous bridge repair, Bluetooth manual operation, and 3D-printed attachments to complete city-building and cleanup tasks.

---

## Table of Contents

* [Overview](#overview)

  * [Challenge Context](#challenge-context)
* [Repository Structure](#repository-structure)
* [Robot Summary](#robot-summary)
* [Hardware Overview](#hardware-overview)
* [Software Architecture](#software-architecture)
* [Quick Start (Flash & Run)](#quick-start-flash--run)

  * [Bluetooth Commands](#bluetooth-commands)
* [Behavior Summary](#behavior-summary)

  * [Autonomous Mode](#autonomous-mode-mandatory-30-seconds)
  * [Manual Mode](#manual-mode)
* [Mechanical Design](#mechanical-design)
* [Calibration](#calibration)
* [Rebuilding OPTIMUS V2](#rebuilding-optimus-v2)
* [Documentation & Media](#documentation--media)
* [Credits](#credits)

---

## Overview

### Challenge Context

The **Smart City Builders Challenge** tasks participants to design robots addressing three real-world problems in sustainable cities:

1. **Fixing a Broken Bridge** – repairing road infrastructure before autonomous cars arrive.
2. **Building Essential Services** – constructing schools, hospitals, and workplaces using color-coded blocks.
3. **Cleaning the City** – collecting rubbish balls and disposing them in bins.

Each match lasts **3 minutes**, beginning with a **30-second autonomous mode** (worth double points), followed by a **manual mode** (Bluetooth-controlled). Teams must score as many points as possible without external assistance.

---

## Repository Structure

```
├── docs/
│   ├── Engineering_Notebook.pdf
│   └── Game_Rules.pdf
├── models/
│   └── attachments/
│       ├── BAR.STL
│       ├── clipper_left.STL
│       ├── clipper_right.STL
│       ├── GEAR 9.STL
│       ├── holder.STL
│       ├── Long rack.STL
│       ├── Motor_Holder.STL
│       ├── RACK.STL
│       ├── ROUND.STL
│       ├── slider_left.STL
│       └── slider_right.STL
├── schemes/
│   └── wiring_diagram.png
├── src/
│   ├── main.py
│   └── utils.py
├── videos/
│   └── GRC_Optimus_demo.mp4
├── LICENSE
├── README.md
├── requirements.txt

```

---

## Robot Summary

| Component             | Description                                      |
| --------------------- | ------------------------------------------------ |
| **Name**              | OPTIMUS V2                                         |
| **Controller**        | Raspberry Pi Pico (MicroPython)                  |
| **Drive System**      | 4-wheel tank drive (DC motors via L298N drivers) |
| **Power Supply**      | 7.4V Li-ion battery pack                         |
| **Communication**     | HC-05 Bluetooth module (UART0, 9600 baud)        |
| **Autonomous Inputs** | 2 start buttons (Red = GPIO 2, Blue = GPIO 6)    |
| **Outputs**           | 4 DC motors, 3 servos                            |
| **Attachments**       | Rear bridge pusher + front dual-servo gripper    |

---

## Hardware Overview

### Core Components

| Component         | Purpose                | Notes                                          |
| ----------------- | ---------------------- | ---------------------------------------------- |
| Raspberry Pi Pico | Main brain             | Runs MicroPython code (v1.22+)                 |
| 4x DC Motors      | Movement               | Controlled by L298N drivers (PWM)              |
| 2x L298N Modules  | Motor control          | Each drives 2 motors                           |
| 3x Servos         | Gripper lift and claws | Temporarily disables one motor pin when in use |
| HC-05 Bluetooth   | Manual control         | UART0 pins (GP0 = TX, GP1 = RX)                |
| 2x Start Buttons  | Side selection         | Red = GPIO 2, Blue = GPIO 6                    |
| Power Source      | 7.4V battery           | Common ground with Pico                        |

*See* [`schemes/wiring_diagram.png`](schemes/wiring_diagram.png) *for detailed pin mapping.*

---

## Software Architecture

```bash
src/
├── main.py      # Main program (autonomous + manual control)
└── utils.py     # Movement, servo, and behavior utilities
```

* **`main.py`** handles startup, side selection, and switching between autonomous and manual modes.
* **`utils.py`** contains helper functions for movement, servo control, and timing calibration.

---

## Quick Start (Flash & Run)

1. **Flash MicroPython** to the Raspberry Pi Pico (via Thonny or `esptool`).
2. **Copy Files**: Upload `src/main.py` and `src/utils.py` to the Pico’s root directory.
3. **Wire Components**: Follow [`schemes/wiring_diagram.png`](schemes/wiring_diagram.png) to connect motors, servos, Bluetooth, and buttons.
4. **Power the Robot** and press:

   * **Red button** → Run *Red-side autonomous routine*.
   * **Blue button** → Run *Blue-side autonomous routine*.
5. After the autonomous phase or when Bluetooth input is received, the robot automatically switches to **manual control mode**.

### Bluetooth Commands

| Command | Action        |
| ------- | ------------- |
| `F`     | Move forward  |
| `B`     | Move backward |
| `L`     | Turn left     |
| `R`     | Turn right    |
| `S`     | Stop          |
| `1`     | Lower arms    |
| `2`     | Raise arms    |
| `3`     | Open gripper  |
| `4`     | Close gripper |

---

## Behavior Summary

### Autonomous Mode (Mandatory, 30 seconds)

* Starts upon pressing Red/Blue button.
* Executes bridge repair task based on field side.
* Each side routine uses pre-timed motion (via `calc_time(distance_cm)` calibration).
* Bluetooth input cancels autonomous mode immediately.

#### Autonomous Routines

| Side     | Behavior Summary                                                                   |
| -------- | ---------------------------------------------------------------------------------- |
| **Red**  | Moves forward ≈ 88 cm, turns toward bridge, aligns, and pushes pallets into place. |
| **Blue** | Mirror version of Red with ≈ 70 cm forward motion.                                        |

#### Scoring Reference (from Game Manual)

| Task                               | Autonomous  | Manual | Notes                |
| ---------------------------------- | ----------- | ------ | -------------------- |
| Pallet placed correctly            | 40 pts      | 20 pts | Double in autonomous |
| Fixed bridge before carbots arrive | +10 bonus   | —      | —                    |
| Carbot deviation                   | −30 penalty | —      | —                    |

---

### Manual Mode

* Activates automatically after autonomous mode or via Bluetooth input.
* Allows fine control for:

  * Collecting and stacking building blocks (school, hospital, workplace)
  * Cleaning the city (rubbish balls)

#### Building Rules

* Blocks must be stacked **Copper → Violet → Grey**.
* Each correctly placed block: **+5 points**.
* Wrong color order: **−10 penalty**.
* Complete structure in correct zone: **+50 bonus**.

#### Cleanup Rules

| Task                         | Autonomous | Manual | Notes                |
| ---------------------------- | ---------- | ------ | -------------------- |
| Rubbish deposited            | 10 pts     | 5 pts  | Must fully enter bin |
| Mishandled/dropped container | —          | −5 pts | —                    |

---

## Mechanical Design

| Module                | Description                                                   |
| --------------------- | ------------------------------------------------------------- |
| **Drive System**      | 4-wheel tank configuration for stability and turning control. |
| **Front Gripper**     | Dual 3D-printed rectangular arms (servo-controlled).          |
| **Gripper Functions** | Lift (servo 1), Open/Close (servos 2 & 3).                    |
| **Rear Attachment**   | Fixed pusher plate for bridge repair.                         |

3D models for attachments available in [`models/attachments`](models/attachments)
---

## Calibration

Motion timing is based on travel distance (40.5 cm/s baseline). The following formula is used for consistent movement:

```python
# utils.py
# Convert distance (cm) to time (s)
time = 1.8 * (distance_cm / 40.5)
```

Adjust the multiplier based on battery level and motor friction.

---

## Rebuilding OPTIMUS V2

To replicate the full robot:

1. Assemble the Xplore Bot chassis.
2. Attach rear bridge pusher and dual-servo front gripper.
3. Wire components per `schemes/wiring_diagram.png`.
4. Flash MicroPython to the Pico.
5. Copy `main.py` and `utils.py` into root.
6. Test motion timing with small distances before full run.
7. Calibrate servo angles in `utils.py`.
8. Verify Bluetooth communication using serial monitor or RC Controller app.

---

## Documentation & Media

* [`docs/Engineering_Notebook.pdf`](docs/Engineering_Notebook.pdf) — Build log, team notes, design iterations.
* [`docs/Game_Rules.pdf`](docs/Game_Rules.pdf) — Full official Smart City Builders Challenge manual.
* [`schemes/wiring_diagram.png`](schemes/wiring_diagram.png) — Electrical wiring reference.
* [`models/attachments`](models/attachments) — 3D models for attachments.
* [`videos/GRC_Optimus_demo.mp4`](videos/GRC_Optimus_demo.mp4) — Demo run footage.

---

## Credits

* Team **Optimus** — University of Ghana, October 2025
* **Members:**

  * Dominic Fatonade: Programmer — [mrdominicfatonade@gmail.com](mailto:mrdominicfatonade@gmail.com) || [dom1of1](https://github.com/dom1of1)
  * Bess-Marie Wuddah-Martey: Designer — [bessmariewuddahmartey@gmail.com](mailto:bessmariewuddahmartey@gmail.com) || [bess](https://github.com/--)
  * Anastasia Andoh: Builder — [andohanastasia3@gmail.com](mailto:andohanastasia3@gmail.com) || [Anadhilah](https://github.com/Anadhilah)
* **Event Organizer:** Fireflyio Robotics — Ghana Robotics Competition 2025
