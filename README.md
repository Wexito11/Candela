# CANDELA (Лесной страж) - autonomous wildfire monitoring drone

Original team repository: [milady-Winter/Lesnoi_strazh](https://github.com/milady-Winter/Lesnoi_strazh)

Team project from the **«Engineering Challenge» hackathon** (Bauman League, April 2026). A quadcopter patrols a forest area, detects fire with a camera and a temperature sensor, investigates the spot, returns to base when the battery is low and lands precisely on an ArUco marker.

## My part

- **Hardware design and assembly** - 7" Mark4 frame, SpeedyBee F405 flight controller, Raspberry Pi 5 as the onboard computer, takeoff weight 1.18 kg. Selected the components, made the **budget**, drew the **electrical schematic**, soldered and assembled the drone.
- **Sensors** - Arduino Nano with a VL53L0X rangefinder (down), TOF250 (forward), MAX6675 thermocouple and KY-037 microphone, connected to the Raspberry Pi.
- **Mission logic and flight control** (`simulator/mission.py`, `simulator/flight_controller.py`) - Python + MAVLink (pymavlink). A finite state machine with 7 states:
  `IDLE → TAKEOFF → PATROL → INVESTIGATE → RETURN → LANDING`, plus `EMERGENCY` for any failure. Includes return-to-base on low battery and a manual override from the TX16S remote.
- **Simulation** (`simulator/`) - the full mission runs against ArduPilot SITL (Mission Planner) with simulated sensors, fire events and landing corrections, so the logic can be tested without flying.

I also helped train the YOLO fire detector. Teammates led the fire detection and ArUco landing modules; the mission code receives their results over UDP.

## Architecture

```
 Camera ──▶ fire detection (YOLO) ──┐
                                     │ UDP
 Camera ──▶ ArUco landing ───────────┤
                                     ▼
 Arduino Nano sensors ──serial──▶ Raspberry Pi 5: mission FSM ──MAVLink──▶ SpeedyBee F405 (ArduPilot)
 (distance, temperature, sound)                                              ▲
                                                                  TX16S manual override
```

## Run the simulation

1. Install [Mission Planner](https://firmware.ardupilot.org/Tools/MissionPlanner/) and start SITL: *Simulation → MultiRotor → Start*.
2. Install dependencies: `pip install -r simulator/requirements_sim.txt`
3. In separate terminals:
   ```bash
   cd simulator
   python main_sim.py                              # mission
   python fire_simulator.py --angle 35 --delay 10  # simulate a fire
   python landing_simulator.py                     # simulate ArUco landing corrections
   ```

What is tested: takeoff and patrol route, fire detection and investigation, return to base on low battery, precise ArUco landing, switching between manual and autonomous mode.

## Repository structure

- `simulator/` - mission FSM, MAVLink flight controller wrapper, sensor reader and simulators
- `landing/` - ArUco precision landing (OpenCV)
- `drone/`, `fire_detection/`, `base_station/`, `database/` - onboard entry point, detection, ground station, fire log database
- `Dockerfile` - container for the onboard software

## Stack

Python · pymavlink · ArduPilot (SITL) · OpenCV / ArUco · YOLO · Raspberry Pi 5 · Arduino Nano · Docker
