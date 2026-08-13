# Field Mapper Robot for Low-Field MRI

## Overview

This project presents the design and implementation of a modular, low-cost robotic field-mapping system for characterizing the magnetic field inside a low-field MRI Halbach array.

The system combines:

- A three-axis linear actuator
- Three stepper motors
- Three TB6600 (M422) motor drivers
- An Arduino UNO R4 Minima
- A Hall probe
- An AlphaLab GM2 gaussmeter
- Python-based motion control and data acquisition

The robot automatically moves the Hall probe through predefined coordinates inside the MRI bore, records the magnetic field at each position, and stores the measurements in a CSV file for further analysis.

---

## Project Objectives

- Measure the magnetic field distribution inside a low-field MRI system
- Automate magnetic field measurements
- Generate a three-dimensional magnetic field map
- Evaluate field homogeneity
- Identify the most homogeneous imaging region

---

## Hardware Requirements

| Component | Quantity |
| --- | --- |
| Arduino UNO R4 Minima | 1 |
| TB6600 (M422) stepper driver | 3 |
| Linear actuator (X axis) | 1 |
| Linear actuator (Y axis) | 1 |
| Linear actuator (Z axis) | 1 |
| Stepper motors | 3 |
| 24 V DC power supply | 1 |
| AlphaLab GM2 gaussmeter | 1 |
| Hall probe | 1 |
| Jumper wires | As required |
| USB-C cable | 1 |
| Screwdrivers | As required |

---

## Software Requirements

- Python 3.13
- Arduino IDE
- PyCharm

Required Python libraries:

```bash
pip install pyserial numpy matplotlib
```

---

## Wiring

### Arduino Pin Mapping

| Axis | Step Pin | Direction Pin |
| --- | --- | --- |
| X | 5 | 4 |
| Y | 3 | 2 |
| Z | 7 | 6 |

---

### TB6600 Driver Wiring

#### X Driver

```text
PUL+ → Arduino pin 5
PUL- → Arduino GND

DIR+ → Arduino pin 4
DIR- → Arduino GND

ENA+ → Not connected
ENA- → Not connected
```

#### Y Driver

```text
PUL+ → Arduino pin 3
PUL- → Arduino GND

DIR+ → Arduino pin 2
DIR- → Arduino GND

ENA+ → Not connected
ENA- → Not connected
```

#### Z Driver

```text
PUL+ → Arduino pin 7
PUL- → Arduino GND

DIR+ → Arduino pin 6
DIR- → Arduino GND

ENA+ → Not connected
ENA- → Not connected
```

---

### Stepper Motor Wiring

```text
A+ → Black wire
A- → Green wire

B+ → Red wire
B- → Blue wire
```

Motor wire colors may differ depending on the manufacturer.

Always verify the coil pairs using a multimeter.

Expected resistance between wires belonging to the same coil:

```text
≈1.9 Ω
```

---

### Power Supply Wiring

```text
Driver VCC → 24 V DC positive terminal

Driver GND → 24 V DC negative terminal
```

Verify all connections with a multimeter before powering the system.

---

## TB6600 DIP Switch Configuration

Recommended starting configuration:

```text
S1 = OFF
S2 = OFF
S3 = OFF

S4 = OFF
S5 = OFF
S6 = OFF
```

Adjust the current and microstepping settings according to the specifications of your stepper motors.

---

## Installation

Clone the repository:

```bash
git clone https://github.com/SipanHovsep/Field_mapper_robot
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Upload `Arduino_script.ino` to the Arduino UNO R4 Minima.

---

## Workflow

The software workflow follows four steps:

```text
1. Upload Arduino_script.ino

↓

2. Run Arduino_finder.py

↓

3. Run Path_maker.py

↓

4. Run Arduino_runner.py
```

---

## Script Overview

### Arduino_script.ino

Responsibilities:

- Configure Arduino pins
- Listen for serial commands
- Move the robot
- Convert movement commands into step pulses

Supported commands:

```text
X+
X-

Y+
Y-

Z+
Z-
```

Serial communication:

```text
9600 baud
```

Movement calibration:

```cpp
#define STEPS_PER_MM 533.33333334
```

---

### Arduino_finder.py

Responsibilities:

- Detect the Arduino automatically
- Identify the correct COM port
- Save the port number to `arduino_port.txt`

Run:

```bash
python Arduino_finder.py
```

Expected output:

```text
COM3 - USB Serial Device (COM3)

Found Arduino on port: COM3
```

---

### Path_maker.py

Responsibilities:

- Generate the robot trajectory
- Define the measurement volume
- Define point spacing
- Visualize the trajectory
- Generate `path.csv`

Current settings:

```python
RADIUS_MM = 30
STEP_MM = 25
```

To increase the number of measurement points:

```python
STEP_MM = 5
```

To increase the measurement volume:

```python
RADIUS_MM = 50
```

---

### Arduino_runner.py

Responsibilities:

- Initialize the Hall probe
- Load coordinates from `path.csv`
- Move the robot
- Communicate with the gaussmeter
- Record magnetic field values
- Generate `data.csv`

Run:

```bash
python Arduino_runner.py
```

Expected output:

```text
[INFO] motion link on COM3

[INFO] initializing probe...

[INFO] probe ready

[INFO] 625 points loaded from path.csv

[INFO] finished - data in data.csv
```

---

## Path Generation

Example measurement points:

```text
x y z

0 0 -30

-29 -4 -5

-4 -29 -5

-4 -4 -5

-4 21 -5

21 -4 -5

-22 3 20

3 -22 20

3 3 20
```

---

## Output Files

### path.csv

Contains the robot trajectory.

```text
x y z

0 0 -30

-29 -4 -5

3 3 20
```

---

### data.csv

Contains the magnetic field measurements.

```text
x y z reading

0 0 -30 456.10

-29 -4 -5 456.09

3 3 20 456.11
```

---

## Magnetic Field Units

Gaussmeter readings may be displayed in:

```text
Tesla (T)

Millitesla (mT)

Gauss (G)
```

Conversions:

```text
1 T = 10,000 G

1 T = 1,000 mT

1 G = 0.0001 T

1 G = 0.1 mT
```

Excel conversion from Gauss to Tesla:

```excel
=D2/10000
```

---

## Field Homogeneity

Peak-to-peak homogeneity:

```text
Homogeneity = ((Bmax - Bmin) / Bcenter) × 1,000,000
```

where:

```text
Bmax = Maximum magnetic field

Bmin = Minimum magnetic field

Bcenter = Magnetic field at the center of the imaging volume
```

---

## Troubleshooting

### Arduino upload failed

Error:

```text
No DFU capable USB device available
```

Solution:

- Reinstall the UNO R4 driver
- Reconnect the USB cable
- Reinstall the board package
- Exit bootloader mode

---

### Access denied

Error:

```text
PermissionError(13)
```

Solution:

- Close the Arduino IDE
- Close the Serial Monitor
- Run the Python script again

---

### Robot does not move

Check:

- Driver LEDs
- 24 V power supply
- Motor wiring
- Arduino ground connections
- DIP switch configuration
- Stepper motor coil pairs

---

### Motor only buzzes

Possible causes:

- Incorrect microstepping
- Incorrect coil wiring
- Mechanical binding
- Incorrect current settings

---

### Probe timeout

Error:

```text
[WARN] timeout - resetting probe
```

Solution:

- Reinitialize the probe
- Verify the COM port
- Check the USB connection
- Confirm the gaussmeter is transmitting data

---

## Future Improvements

- Increase measurement density
- Generate higher-resolution field maps
- Implement real-time visualization
- Improve homogeneity calculations
- Create 3D magnetic field heat maps
- Optimize the low-field MRI imaging volume