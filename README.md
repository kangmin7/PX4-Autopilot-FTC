# PX4-Autopilot-FTC

Custom [PX4-Autopilot](https://github.com/PX4/PX4-Autopilot) with Fault Tolerant Control (FTC) for multirotors, targeting Gazebo SITL simulation.

**Demo**:
[![Demo](https://img.youtube.com/vi/oBGaT4WjUKI/maxresdefault.jpg)](https://www.youtube.com/watch?v=oBGaT4WjUKI)
[![Demo](https://img.youtube.com/vi/v-Z2dhzyuTM/maxresdefault.jpg)](https://www.youtube.com/watch?v=v-Z2dhzyuTM)

---

## Changes

- Added motor failure detection and fault tolerant emergency landing for quadrotor (x500)
- Added motor failure detection and fault tolerant control for hexarotor (Typhoon H480 from gazebo-classic)
- Tuned PX4 parameters for stable descent/control after motor failure

---

## Running

**Quadrotor:**
```bash
cd ~/PX4-Autopilot-FTC
make px4_sitl gz_x500
```

**Hexarotor:**
```bash
cd ~/PX4-Autopilot-FTC
make px4_sitl gz_typhoon_h480
```

Then in the PX4 console, trigger motor failure:
```
failure motor off -i 1
```
