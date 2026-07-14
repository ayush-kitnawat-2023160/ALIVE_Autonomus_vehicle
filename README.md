# ALIVE: Autonomous Vehicle Stack — Restoration & Adaptation

> **B.Tech Project | IIIT Delhi | 2025–2026**  
> Restoration and adaptation of a legacy ROS-based autonomous driving stack for the **Mahindra e2o** electric vehicle and **Golf Cart** platforms.

**Authors:** Ayush Kitnawat (2023160) · Dev (2023189)  
**Advisors:** Dr. Saket Anand · Prof. Sanjit Krishan Kaul  
**Mentor:** Rehan Fazal (PhD Student)


https://github.com/user-attachments/assets/2b18edaa-df30-461a-80f2-16a76b72180b


---

## Project Overview

The ALIVE (Autonomous Last Mile Vehicle) stack is a full-pipeline autonomous driving system built on ROS, originally developed over multiple student cohorts at IIIT Delhi. This project restored it from a **fragmented, non-functional state** to a **field-tested, outdoor-capable autonomous vehicle**.

The project proceeded in three phases:
1. **CARLA Simulation Validation** — end-to-end stack verification in a safe virtual environment
2. **Mahindra e2o Hardware Integration** — real vehicle bringup, CAN bus fix, single-LiDAR adaptation
3. **Golf Cart Integration** — hardware mounting and drive-by-wire fault diagnosis

**Results:** 80% navigation success in open environments, 70% in constrained scenarios — fully demonstrated to faculty in live outdoor trials.

---

## System Architecture

```
LiDAR / Camera
      │
      ▼
  PERCEPTION                    LOCALIZATION
(Point Clouds,             (Pose Estimation via
Obstacle Detection)           FLOAM + EKF)
      │                              │
      └──────────┬───────────────────┘
                 ▼
             PLANNING
      (Global Path + Local
       Trajectory Generation)
                 │
                 ▼
             CONTROL
      (Steering, Throttle,
       Velocity Commands)
                 │
                 ▼
     CAN BUS / DRIVE-BY-WIRE
                 │
                 ▼
        VEHICLE PLATFORM
     (Mahindra e2o / Golf Cart)
```

**Simulation environment:** CARLA 0.9.11 (used for full-stack validation before hardware deployment)

---

## ROS Stack — Key Packages

| Package | Role |
|---|---|
| `car_data` | Converts CARLA/vehicle odometry → `/pose`, `/odometry/filtered/global`, TF |
| `point_cloud_transformer` | Merges, transforms, and filters LiDAR streams into obstacle clouds |
| `estimator` | Fuses LiDAR + camera + virtual obstacles into a `/grid_map` for planning |
| `map_publisher` | Loads Lanelet2 HD map (CARLA town or IIITD campus) |
| `structured_planner` | Trajectory rollout planner — publishes `/trajectory_rollout` |
| `cascaded_pid` | PID trajectory tracker — publishes `/pid_info` |
| `alive_behavior_tree` | High-level behavior arbitration (traffic lights, goal-reach, stop) |
| `hybrid_astar` | Optional reverse/recovery planner |
| `yolop` / `object_detection` | Camera-based perception for lane segmentation and object detection |
| `virtual_obs_pose_pub` | Publishes scenario-defined virtual obstacles for repeatable tests |

---

## Data Flow

```
CARLA / Real Sensors
        │
        ▼
  carla_ros_bridge / Velodyne Driver
        │
   ┌────┴────────────────┐
   ▼                     ▼
car_data_node     point_cloud_transformer
   │  /pose              │  /global_velodyne_obstacles
   │                     ▼
   │              estimator (/grid_map)
   │                     │
   └──────────┬──────────┘
              ▼
    structured_planner
    (/trajectory_rollout)
              │
              ▼
      cascaded_pid (/pid_info)
              │
              ▼
   CARLA adapter / CAN interface
              │
              ▼
       Vehicle Actuators
```

---

## Key Engineering Contributions

### 1. Codebase Restoration
- Audited a fragmented multi-repository ROS codebase
- Resolved deprecated Python dependencies (numpy, scipy API mismatches)
- Achieved a clean, reproducible build across all core packages

### 2. CARLA Simulation Validation
- Fixed a **race condition** in the 3-LiDAR synchronization node causing empty point cloud outputs
- Corrected a **ROS topic name mismatch** in the control pipeline (vehicle was stationary despite active planner)
- Fixed **velocity scaling factor** miscalibrated for CARLA's physics model
- Verified full perception → planning → control loop with goal-directed autonomous runs

### 3. CAN Bus Fix (Mahindra e2o)
- Diagnosed CAN autonomous-mode rejection using `candump` traffic capture
- Compared manual vs. autonomous CAN frame sequences to identify the incorrect message format
- After correcting and rebuilding CAN dependencies: vehicle's steering wheel responded to software commands for the first time

### 4. Single-LiDAR Adaptation
- Patched a **null pointer dereference** in the single-LiDAR code path (uninitialized merged cloud output → segfault)
- Fixed array indexing made conditional on active LiDAR count
- Stack ran stably for extended outdoor sessions without crashes

### 5. Field Testing (Outdoor, IIIT Delhi Campus)
- Demonstrated **goal-directed navigation**, **obstacle avoidance**, and **repeated goal publication** to faculty
- Isolated two remaining edge-case bugs: oscillatory planner recovery and goal-reach overshoot (both documented with workarounds)

### 6. Golf Cart — Fault Diagnosis
- Mounted hardware (LiDAR, NUC, networking) and confirmed ROS sensor topics
- Identified **Controllio drive-by-wire fault state** triggered by state-transition command
- Hypothesized missing safety handshake; documented as primary blocker for next phase

---

## System Requirements

| Component | Version |
|---|---|
| OS | Ubuntu 20.04 |
| ROS | Noetic (full desktop) |
| CARLA | 0.9.11 |
| CUDA | 11.6 |
| cuDNN | 8.4 |
| OpenCV | 4.5.5 (CUDA build) |
| LibTorch | 1.12.0 |
| NVIDIA Drivers | 510 |

---

## Future Work

- **Golf Cart:** Reverse-engineer Controllio initialization handshake → enable autonomous mode
- **Multi-LiDAR:** Restore 360° coverage on golf cart platform
- **Planner tuning:** Fix oscillatory recovery and goal-reach overshoot for single-LiDAR cost parameters
- **Data collection:** Structured rosbag logging (LiDAR + camera + odometry) for perception research
- **Complex environments:** Pedestrian interaction, dynamic obstacles, multi-agent scenarios

---

## References

1. Dosovitskiy et al., *CARLA: An Open Urban Driving Simulator*, CoRL 2017
2. Quigley et al., *ROS: An Open-Source Robot Operating System*, ICRA 2009
3. Pendleton et al., *Perception, Planning, Control, and Coordination for Autonomous Vehicles*, Machines 2017
4. Bosch, *CAN Specification 2.0*, 1991

---

*IIIT Delhi · B.Tech CSE · Engineering*
