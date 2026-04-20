# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

PX4-Autopilot-FTC is a fork of [PX4 Autopilot](https://github.com/PX4/PX4-Autopilot) — an open-source flight control system supporting multirotors, fixed-wing, VTOL, rovers, and more. The "FTC" in the repo name stands for Fault Tolerant Control. It targets both real hardware (NuttX RTOS) and software-in-the-loop simulation (POSIX/Linux).

## Build Commands

```bash
# List all available build targets
make list_config_targets

# Build SITL simulation (most common for development)
make px4_sitl_default

# Build for specific hardware
make px4_fmu-v5_default
make px4_fmu-v6x_default

# Build with sanitizers
PX4_ASAN=1 make px4_sitl_default

# Run SITL
make px4_sitl_default gz_x500    # with Gazebo simulator
```

The Makefile wraps CMake. Build artifacts go to `build/<target>/`.

## Test Commands

```bash
# Run all unit tests
make tests

# Run with coverage
make tests_coverage

# Run integration tests (SITL-based)
make tests_integration

# Run MAVSDK-based flight tests
make px4_sitl_default mavsdk_tests
./test/mavsdk_tests/mavsdk_test_runner.py --speed-factor 20 test/mavsdk_tests/configs/sitl.json
```

Unit tests use Google Test and live alongside source as `*_test.cpp` files within module directories.

## Lint and Format

```bash
# Check formatting (must pass before merging)
make check_format

# Auto-fix formatting
make format

# Format only files changed vs HEAD
make format_changed

# Static analysis
make clang-tidy
make clang-tidy-fix

# Validate module YAML configs
make validate_module_configs
```

Formatting uses astyle (config in `Tools/astyle/`). Static analysis config is in `.clang-tidy`.

## Architecture

### Inter-Module Communication: uORB

All modules communicate through **uORB**, a publish/subscribe middleware. Message definitions live in `msg/` as YAML files and are compiled to C++ classes. There is a single writer per topic and multiple readers. uORB is DDS-compatible and bridges to ROS 2 via the `uxrce_dds_client` module.

```bash
make uorb_graphs   # visualize message flow
make msg_docs      # generate message documentation
```

### Source Layout

| Path | Purpose |
|------|---------|
| `src/modules/` | Flight controllers, estimators, safety logic, communication |
| `src/drivers/` | Hardware drivers (IMU, baro, mag, GPS, radio) |
| `src/lib/` | Reusable libraries (math, control, geo, parameters) |
| `src/systemcmds/` | CLI commands (`param`, `perf`, `ver`, etc.) |
| `msg/` | uORB message definitions |
| `platforms/nuttx/` | NuttX embedded OS port |
| `platforms/posix/` | Linux/macOS SITL |
| `platforms/ros2/` | ROS 2 integration |
| `boards/` | Per-board hardware configuration |
| `ROMFS/` | Startup scripts and airframe configs |
| `test/` | Integration and SITL tests |
| `cmake/` | CMake helper modules |

### Key Modules

**Control pipeline** (multicopter): `sensors` → `ekf2` → `mc_pos_control` → `mc_att_control` → `mc_rate_control` → `actuator_outputs`

- `commander/` — Flight state machine, arming, failsafe, mode management
- `ekf2/` — Extended Kalman Filter for position/attitude/velocity estimation
- `navigator/` — Autonomous navigation (missions, RTL, precision landing)
- `mc_att_control/`, `mc_pos_control/`, `mc_rate_control/` — Multicopter controllers
- `fw_att_control/`, `fw_lateral_longitudinal_control/` — Fixed-wing controllers
- `vtol_att_control/` — VTOL transition logic
- `mavlink/` — MAVLink protocol (submodule)
- `airspeed_selector/` — Selects trusted airspeed source (relevant to FTC work)
- `logger/` — On-board ulog logging

### Module Structure Pattern

```
src/modules/<name>/
├── CMakeLists.txt       # px4_add_module() call
├── module.yaml          # Metadata, params, subscriptions
├── Kconfig              # Kernel config options
├── <Module>.cpp/.hpp    # Implementation
└── <Module>_test.cpp    # Unit tests (gtest)
```

### Parameter System

Parameters are defined in `module_params.yaml` (per module), stored by `dataman`, and accessed via `param_get()`/`param_set()`. System defaults are in `ROMFS/px4fmu_common/init.d/`.

## Commit Convention

PX4 uses Conventional Commits. Use the `/commit` skill for guided commit creation.

```
type(scope): description

type: feat, fix, docs, refactor, perf, test, build, ci, chore
scope: affected module/driver (e.g., ekf2, mavlink, commander, airspeed_selector)
```

Breaking changes: append `!` before the colon (`feat(ekf2)!: ...`).

## Skills Available

- `/commit` — Create a conventional commit
- `/pr` — Create a pull request
- `/review` — Review a pull request
- `/rebase-onto-main` — Rebase branch onto main cleanly
