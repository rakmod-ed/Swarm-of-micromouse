# Swarm of Micromouse

## Overview

This project involves designing, building, and programming three autonomous robotic mice (Micromouse) that can navigate an unknown maze, map it collaboratively, and find the optimal path from the starting point to a destination in the shortest time possible. The distinguishing feature of this project over a single-robot approach is that all three mice find the target position simultaneously, sharing real-time information so that no section of the maze is mapped twice by more than one robot.

---

## Objectives

- Build three autonomous robots capable of navigating an unknown maze without human intervention.
- Implement path planning algorithms for single-robot maze solving.
- Extend to a multi-agent swarm system where robots explore the maze independently but in a coordinated, non-redundant manner.
- Establish wireless peer-to-peer communication between robots using the ESP-NOW protocol.
- Avoid inter-robot collisions while maximizing exploration efficiency.

---

## Project Workflow

### 1. Path Planning Algorithms (Single Mouse)

Before moving to multi-agent behavior, path planning was studied and implemented for a single robot.

**Breadth-First Search (BFS)**
BFS was the first algorithm implemented. Using the PyMaze library, a randomly generated maze was solved by an agent navigating from one corner to the opposite corner. BFS guarantees the shortest path in an unweighted graph, making it a reliable baseline.

**Depth-First Search (DFS)**
DFS was tested with varying configurations, including different loop percentages and movement preferences (e.g., ENSW, WSNE ordering). While DFS successfully navigates mazes, it does not guarantee the shortest path, which was confirmed experimentally during testing.

**A* (A-Star)**
A* was implemented in Python to simulate maze solving. It uses a heuristic function (typically Manhattan distance) to guide the search, combining the cost-so-far and estimated cost-to-goal. This makes it significantly more efficient than BFS in large mazes. A* is the intended algorithm for the swarm robots in the final implementation.

---

### 2. Multi-Agent Simulation

After validating single-robot algorithms, the focus shifted to simulating multiple agents navigating a maze simultaneously.

The simulation uses BFS across all agents in parallel. Key constraints enforced in the simulation:
- No two agents map the same cell twice (shared map state).
- Agents do not collide with each other.
- There is no fixed goal cell; agents explore until the maze is fully mapped.

This phase validated the coordination logic before it was implemented on physical hardware.

---

### 3. Hardware Design

Each Micromouse was built from scratch using the following components:

| Component | Specification |
|---|---|
| Microcontroller | ESP32 |
| Distance Sensors | Ultrasonic sensors x3 (front, left, right) |
| Drive Motors | GM20 DC Motors, 6V, with encoders x2 |
| Motor Driver | L293D |
| Power Source | LiPo battery, 6V |
| Voltage Regulation | Buck converter |

The ESP32 was chosen as the microcontroller because it provides onboard Wi-Fi and Bluetooth, supports the ESP-NOW protocol natively, and has sufficient GPIO pins and processing power for real-time sensor reading and motor control.

The three ultrasonic sensors are mounted to detect walls on the front and both sides of the robot, enabling the robot to determine available paths at each cell.

The GM20 motors with encoders allow precise odometry, meaning the robot can track how far it has traveled and correct its position within the maze.

---

### 4. Circuit Design

The circuit was first designed and verified on TinkerCad before physical assembly.

**Power distribution:**
- A 6V LiPo battery powers the motors via pin 8 (Vcc2) of the L293D motor driver.
- The Arduino/ESP32 5V output powers the L293D logic supply via pin 16 (Vcc1) and also supplies the ultrasonic sensors via the breadboard 5V rail.
- All components share a common ground reference to prevent floating ground issues.

The L293D motor driver handles bidirectional motor control for both drive motors, with enable and direction pins connected to the ESP32's GPIO outputs.

---

### 5. CAD Model and 3D Printing

The mechanical chassis was designed as a CAD model scaled to the dimensions of the maze being used. Once the design was finalized, the chassis was 3D printed. The design accounts for sensor placement (front and sides), motor mounting, wheel alignment, and the PCB/breadboard positioning.

Single-bot testing was conducted on the physical hardware before swarm behavior was introduced.

---

### 6. Swarm Intelligence

Swarm intelligence allows multiple robots to operate as a collective system without centralized control. Each robot makes decisions based on its own local sensor data and information shared by the other robots. The system as a whole exhibits intelligent behavior through this decentralized coordination.

In this project, each Micromouse independently explores the maze while broadcasting its current position to the other two mice. When a robot receives position data from its peers, it avoids navigating to cells already being explored, effectively dividing the maze among the three robots without overlap.

**Properties of the swarm system:**
- Decentralized: no single robot is the master controller.
- Fault-tolerant: if one robot fails, the other two continue operating.
- Scalable: the communication model supports additional robots without architectural changes.

---

### 7. Communication Protocol: ESP-NOW

Two communication models were considered:

**Centralized (Master-Slave):** One robot acts as master, receives data from the others, makes decisions, and sends commands back. This introduces a single point of failure and does not scale well.

**Decentralized (Peer-to-Peer):** Each robot communicates directly with every other robot and makes its own decisions. This was the chosen approach.

**ESP-NOW** was selected as the communication protocol for the following reasons:

- **Low Latency:** Data is transmitted in near real-time, which is essential for coordinated movement.
- **No Infrastructure Required:** ESP-NOW works without a Wi-Fi router or access point, making it suitable for a closed arena environment.
- **Low Power Consumption:** Important for battery-operated robots where runtime matters.
- **Peer-to-Peer and Broadcast Support:** Supports both direct unicast messages between specific robots and broadcast to all robots simultaneously.
- **Acknowledgement (ACK):** ESP-NOW confirms data delivery, reducing the risk of a robot acting on stale or lost information.
- **Simple Integration:** ESP-NOW can be added directly on top of existing ESP32 motor and sensor code with minimal overhead.

Each robot is registered with the MAC addresses of the other two robots. When a robot moves to a new cell, it broadcasts its position. All robots maintain a shared map of explored cells, updated in real time as position data arrives.

---

## Repository Structure

```
swarm-micromouse/
├── simulation/
│   ├── bfs_single.py          # BFS on a single agent using PyMaze
│   ├── dfs_single.py          # DFS with configurable movement preferences
│   ├── astar_single.py        # A* algorithm simulation
│   └── multiagent_bfs.py      # Simultaneous multi-agent BFS exploration
├── hardware/
│   ├── circuit/               # TinkerCad circuit design files
│   └── cad/                   # CAD model files for 3D printing
├── firmware/
│   ├── main/                  # Main ESP32 firmware
│   ├── espnow/                # ESP-NOW communication implementation
│   └── motor_control/         # Motor driver and encoder logic
└── README.md
```

---

## Getting Started

### Simulation

1. Install dependencies:
   ```bash
   pip install pyamaze
   ```

2. Run a single-agent simulation:
   ```bash
   python simulation/bfs_single.py
   ```

3. Run the multi-agent simulation:
   ```bash
   python simulation/multiagent_bfs.py
   ```

### Firmware

1. Install the [Arduino IDE](https://www.arduino.cc/en/software) or [PlatformIO](https://platformio.org/).
2. Install the ESP32 board support package.
3. Open the firmware from the `firmware/main/` directory.
4. Flash to each ESP32, ensuring each robot is assigned its unique device ID and the MAC addresses of the other two robots are correctly configured in the ESP-NOW setup.

---

## Current Status

- Single-robot path planning (BFS, DFS, A*): Complete
- Multi-agent simulation with simultaneous exploration: Complete
- Hardware assembly and single-bot physical testing: Complete
- Circuit design on TinkerCad: Complete
- CAD model and 3D printing: Complete
- ESP-NOW communication implementation: In Progress

---

## Future Work

- Complete and test ESP-NOW swarm communication on physical hardware.
- Implement full maze mapping with real-time shared state across all three robots.
- Optimize path planning for the return run (shortest path after full maze is known).
- Add battery monitoring and low-power sleep modes between moves.
- Evaluate performance metrics: total exploration time, path length, number of redundant cell visits.
