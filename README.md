# Autonomous / Semi-Autonomous 6-DOF Subsea ROV
**Engineering Design File — NovaTech Environmental & Infrastructure Inspection**  
**Author:** Soufiane Azim — Ensatronics / ENSA Tanger  
**Status:** Day-1 Architecture & Documentation Baseline  

---

## 1. Project Charter & Mission Scope

### Mission Statement
Design and prototype a **6-degree-of-freedom (6-DOF) underwater Remotely Operated Vehicle (ROV)** for environmental monitoring and hazardous subsea inspection (port structures, dam walls, coastal runoff, and underwater pipelines). The system reduces human diver exposure in high-risk waters while detecting structural anomalies and water-quality shifts.

### Target Specifications
* **Test Depth:** Up to 10 m prototype rating (tank and shallow-water validation).
* **Fabrication Constraint:** Structural brackets, buoyancy mounts, and end-cap flanges fabricated using **FDM 3D printing only** (PETG or ASA), paired with non-porous sealing interfaces (cast acrylic/PVC cylinder, rubber O-rings, and marine epoxy).
* **Control & Stabilization:** Deterministic **100 Hz** PID stabilization for depth and heading, plus a **6 x 6 thruster allocation matrix** for holonomic motion.
* **Telemetry & Vision:** Tethered link (Cat5e/Ethernet + telemetry) to a topside Ground Control Station (GCS) with live video HUD and sensor logging.

### Scope & Safety Boundaries
* **In Scope:** Waterproof pressure hull, 6-thruster vector control, auto-depth and auto-heading loops, tethered video/telemetry dashboard, and basic anomaly/turbidity logging.
* **Out of Scope (Phase 1):** Deep-sea pressure ratings (>30 m), untethered acoustic navigation, or heavy manipulator arms.
* **Battery & Electrical Safety:** Dedicated bottom-hull leak sensor wired to an immediate interrupt/cutoff, pre-dive vacuum/dry checks, and safe handling of onboard LiPo packs.

---

## 2. Mechanical Hull & 3D-Printer-Only Waterproofing

### 4-Layer Waterproofing Architecture
1. **Primary Pressure Hull:** Cylindrical housing (100 mm–110 mm cast acrylic or Schedule 40/80 PVC) with double nitrile (NBR) O-ring end caps lubricated with silicone grease, or a solvent-welded PVC end + PTFE-sealed threaded access cap.
2. **Epoxy-Potted Cable Pass-Throughs:** IP68 nylon PG cable glands with O-ring flange seals, stripped conductor cores, and marine two-part epoxy potting inside the gland barrel to block capillary wicking along wire strands.
3. **Board & Motor Protection:** Acrylic/silicone conformal coating on PCBs/ESCs (masking connectors and pressure sensor ports), silica gel desiccant packs inside the sealed hull, and fresh-water rinsing for flooded brushless thrusters.
4. **Active Leak Detection & Pre-Dive Checks:** Bottom-hull moisture/leak sensor + pre-dive vacuum hold test (-0.3 to -0.5 bar for 10 minutes) + unweighted dry-hull submersion test before installing electronics or LiPo batteries.

### 3D Printing Rules (PETG / ASA Only)
* **Never rely on bare FDM layer lines as a pressure seal.** Sand O-ring grooves smooth and coat printed end-cap/flange surfaces with 2 thin coats of low-viscosity two-part epoxy or marine resin.
* **Print Settings:** PETG or ASA (avoid PLA due to moisture uptake and creep), **6 perimeter walls**, **40–50% Gyroid infill** for brackets / **100% concentric** for end flanges, and **+5 °C** nozzle temperature for strong inter-layer bonding.
* **Buoyancy Trim:** Place positive buoyancy foam above the center of gravity so the ROV passively rights itself in roll and pitch.

### Planned CAD Deliverables (`/cad`)
* `01_internal_electronics_tray.stl` — Two-tier sled (LiPo/ESCs low for stability; compute/MCU/IMU higher).
* `02_front_viewport_flange.stl` — Epoxy-coated flange clamping a flat acrylic camera window with axial O-ring.
* `03_rear_penetrator_cap.stl` — Epoxy-coated rear cap with flat seats for thruster, tether, and sensor/vacuum glands.
* `04_45deg_thruster_brackets.stl` — Clamps for 4 vectored horizontal thrusters (45 deg) and 2 vertical thrusters.
* `05_buoyancy_pod_clamps.stl` — Top mounts securing buoyancy foam blocks.

---

## 3. Kinematics, Control & Telemetry

### Subsystem Partitioning
* **Real-Time Firmware (STM32 / ESP32):** Runs the 100 Hz control loop, reads the 9-DOF IMU and MS5837 pressure/depth sensor, monitors the leak sensor, computes thruster mixing, and outputs PWM/DShot signals to ESCs.
* **Companion Compute & Vision (Raspberry Pi 4/5 or Jetson):** Handles low-latency video streaming, telemetry forwarding, and lightweight computer vision / water-quality logging.
* **Topside Ground Control Station (GCS):** Web interface displaying live camera feed, depth/heading/leak status, and pilot gamepad inputs.

### Mathematical Model
* **6-DOF Thruster Allocation:**
  $$\mathbf{u} = \mathbf{T}^+ \boldsymbol{\tau}$$
  where $\boldsymbol{\tau} = [F_x, F_y, F_z, M_x, M_y, M_z]^T$ is the commanded force/torque vector, $\mathbf{T}$ is the $6 \times 6$ thruster configuration matrix, and $\mathbf{u}$ is the motor command vector.
* **Depth-Hold PID + Buoyancy Feedforward:**
  $$u_z(t) = K_p e_z(t) + K_i \int_0^t e_z(\tau)\,d\tau + K_d \frac{de_z(t)}{dt} + u_{\text{buoyancy}}$$
  with depth error $e_z = z_{\text{target}} - z_{\text{measured}}$ from hydrostatic pressure $P = P_0 + \rho g z$.

---

## 4. Bill of Materials (Prototype Tier)

| Subsystem | Component | Role |
| :--- | :--- | :--- |
| **Propulsion** | 6x Underwater brushless thrusters + 6x bi-directional ESCs | 6-DOF vectored thrust & reverse control |
| **Hull & Seals** | 100–110 mm acrylic or PVC tube, NBR O-rings, silicone grease, PG glands, marine epoxy | Waterproof housing & potted cable penetrators |
| **Compute & Control** | Raspberry Pi 4/5 (or Jetson) + STM32 / ESP32 | Vision/streaming + deterministic 100 Hz control |
| **Sensors** | MS5837 depth/pressure sensor, 9-DOF IMU, leak sensor, optional turbidity/pH probe | Depth hold, orientation, leak alarm, water check |
| **Vision & Lighting** | Wide-angle camera + waterproof LED lights | Subsea visual inspection |
| **Tether & Power** | Tethered Cat5e/Ethernet cable + onboard LiPo battery + gamepad | Low-latency comms, onboard power, piloting |

---
## External CAD References
*   **guidoschillaci/underwater-drone**: Clamp geometries and 45° thruster mounts adapted for the NovaTech ROV. Source: https://github.com/guidoschillaci/underwater-drone (CC-BY 4.0)

---
## 6. GitHub & Overleaf Workflow

### Repository Layout
```text
rov-subsea-inspector/
├── README.md
├── PROJECT_DESIGN_FILE.md
├── docs/
│   ├── main.tex
│   ├── charter.tex
│   ├── architecture.tex
│   ├── control.tex
│   ├── bom.tex
│   └── logbook.tex
├── cad/
│   ├── slicer-profiles/
│   └── stl/
├── firmware/
│   ├── mcu-pid-controller/
│   └── thruster-mixer/
├── vision-telemetry/
│   └── gcs-dashboard/
└── tests/
    └── pressure-leak-logs/
