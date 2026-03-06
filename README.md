# EO Geometric Simulation Toolbox

**Pixel-accurate ground projection for Earth Observation optical payloads.**

This toolbox implements the full geometric chain from pixel coordinates to ground footprint, including instrument model, payload–platform interfaces, attitude representation, and Earth surface intersection.

Developed by [Flora Gabriagues](https://floragabriagues.github.io) — Optical Payload & Image Quality Engineer.

---

## What it does

Given a pixel index, an instrument model, an attitude, and an orbit position, the toolbox computes where that pixel projects on the ground — and what shape it takes.

This enables:
- **Ground footprint simulation** for any pixel or region of the focal plane
- **Geometric deformation analysis** as a function of off-nadir angle, roll, pitch, yaw
- **Sensitivity studies** on boresight, instrument–platform misalignment, attitude errors
- **Cal/Val preparation** — expected vs. observed ground position of known GCPs

---

## Geometric chain

```
Pixel (i, j)
    │
    ▼
LOS in R_opt          ← pinhole camera model (f, pitch, principal point)
    │
    ▼
LOS in R_inst         ← boresight correction (µrad–mrad)
    │
    ▼
LOS in R_sat          ← instrument–platform misalignment
    │
    ▼
LOS in LVLH           ← attitude (quaternion or Roll/Pitch/Yaw)
    │
    ▼
LOS in ECI            ← LVLH → ECI via orbital mechanics
    │
    ▼
LOS in ECEF           ← ECI → ECEF (Earth rotation, astropy)
    │
    ▼
Ground intersection   ← plane / sphere / WGS84 ellipsoid
    │
    ▼
Lat / Lon + pixel footprint polygon
```

---

## Key features

**Instrument model**
- Pinhole camera: focal length, pixel pitch, principal point
- Per-pixel or per-corner LOS computation
- Boresight offset (small-angle rotation, µrad–mrad range)

**Payload–platform interface**
- Instrument-to-satellite misalignment (3-axis small-angle rotation)
- Independent from boresight — matches real ICD structure

**Attitude**
- Input: quaternion `[w, x, y, z]` (body → ECI) or Roll/Pitch/Yaw angles (deg)
- RPY → quaternion conversion (ZYX convention)
- Quaternion → DCM with norm validation

**Earth model**
- Flat plane (local approximation)
- Spherical Earth (R = 6371 km)
- WGS84 ellipsoid (a = 6378.137 km, b = 6356.752 km)

**Visualization**
- Ground point projection on OpenStreetMap basemap
- Pixel footprint polygon (4-corner projection)
- Multi-pixel swath simulation
- Geometric deformation as a function of attitude angles

---

## Example outputs

### Single pixel projection — attitude sensitivity
Ground projection of the central pixel (i=512, j=512) for 5 different attitudes: nadir, roll 10°, roll 20°, pitch 10°, pitch 20°. Orbit: 500 km SSO, sphere intersection model.
Roll displaces the pixel cross-track (longitude), pitch displaces it along-track (latitude). At 500 km altitude, a 10° roll corresponds to ~88 km of ground displacement. The residual latitude offset on roll cases reflects Earth curvature effects on the spherical intersection model.

<img width="652" height="789" alt="image" src="https://github.com/user-attachments/assets/edd7424b-1e45-4f05-9873-3f9f4ae7aef0" />


### Pixel footprint deformation — roll, pitch and combined attitude
Ground projection of the 4 corners of the central pixel (i=512, j=512) for 4 attitude cases: nadir, roll 10°, pitch 10°, and combined roll+pitch 10°. Coordinates are expressed in metres relative to the pixel center. Orbit: 500 km SSO, sphere intersection model.
Roll shears the pixel along the cross-track axis, pitch along the along-track axis. The combined case shows how both effects accumulate, producing a pixel that is both displaced and geometrically distorted. At 500 km altitude with f=1.5m and 3.6µm pitch, the ground sampling distance is ~1.2m — the deformation, while sub-metre, is measurable and mission-relevant for geolocation accuracy and cal/val budget.

<img width="1589" height="536" alt="image" src="https://github.com/user-attachments/assets/668457f2-17b0-4343-9736-38d56694f1b0" />


### Attitude sensitivity

Ground position shift as a function of roll angle (0° → 30°), for a 500 km orbit.  
Typical output: ~9 km/degree of roll at nadir.

---

## Instrument configuration

```python
instrument = {
    "f"  : 1.55,       # focal length (m)
    "px" : 3.2e-6,     # pixel pitch along i (m)
    "py" : 3.2e-6,     # pixel pitch along j (m)
    "i0" : 1023.5,     # principal point — row
    "j0" : 1023.5,     # principal point — col
    "boresight": {
        "bx": 0.0,     # rotation around X_inst (rad)
        "by": 0.0,     # rotation around Y_inst (rad)
        "bz": 0.0      # rotation around Z_inst (rad)
    }
}
```

## Satellite & orbit configuration

```python
satellite = {
    "position_sat_ECI" : position_ECI,          # (3,) array, metres
    "velocity_sat_ECI" : velocity_ECI,           # (3,) array, m/s
    "t_utc"            : np.datetime64("2025-05-01T14:00:00"),
    "att_quat_RPY"     : [10.0, 0.0, 0.0]       # RPY in degrees, or quaternion [w,x,y,z]
}
```

## Usage

```python
lat, lon = project_pixel_center_to_ground(
    i=512, j=512,
    instrument=instrument,
    IF_misalignment=IF_misalignment,
    satellite=satellite,
    att_mode="RPY",          # "RPY" or "quaternion"
    earth_model="ellipsoid", # "plane", "sphere", "ellipsoid"
    plot=True
)
```

---

## Modules

| Module | Content |
|---|---|
| `LOS.py` | Full projection chain, pixel footprint |
| `attitude.py` | RPY → quaternion, quaternion → DCM |
| `earth.py` | ECI ↔ ECEF, ECEF → lat/lon, LVLH → ECI, ellipsoid intersection |
| `satellite.py` | Orbital velocity, LVLH frame construction |
| `display.py` | Basemap visualization, footprint rendering |

---

## Dependencies

```
numpy
matplotlib
geopandas
shapely
contextily
astropy
pymap3d
```

---

## About

This toolbox is part of my freelance activity in **optical payload performance and image quality** for Earth Observation missions.

I work with engineering teams on:
- System specification and performance budgets
- In-orbit image quality analysis
- Calibration & validation planning
- Geometric and radiometric performance tools

→ [floragabriagues.github.io](https://floragabriagues.github.io)

---

*The full toolbox, including advanced features and customization for specific mission configurations, is available as part of consulting engagements.*
