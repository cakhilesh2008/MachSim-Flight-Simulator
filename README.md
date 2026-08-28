# MachSim: High-Speed Flight Simulator

A Unity flight simulator that flies a fighter jet over a streamed, photoreal model of the
real Earth. Rather than shipping hand-authored terrain, MachSim uses
[Cesium for Unity](https://cesium.com/platform/cesium-for-unity/) to pull elevation data and
satellite imagery on demand from Cesium ion, so the entire planet is flyable from a single
scene.

## Features

- Fly anywhere on Earth over real terrain and satellite imagery
- Pitch, roll, yaw and throttle controls driving a rigidbody flight model
- Four switchable camera viewpoints (chase, high chase, nose, reverse)
- Heads-up display showing throttle, airspeed, altitude, altitude-hold state and fuel
- Fuel system that progressively degrades engine performance as it depletes
- Altitude hold and landing assist modes

## Tech Stack

| Component | Version / Choice |
| --- | --- |
| Engine | Unity **6000.1.7f1** (Unity 6) |
| Render pipeline | Universal Render Pipeline (URP) 17.1.0 |
| Geospatial runtime | Cesium for Unity 1.16.2 |
| Tile source | Cesium ion — World Terrain (asset 1) + Bing Maps Aerial imagery (asset 2) |
| Language | C# (MonoBehaviour scripting) |
| HUD text | TextMesh Pro |
| Input | Unity legacy Input Manager (custom `Pitch` / `Roll` / `Yaw` axes) |
| Aircraft model | Raptor3D FA_N26, FBX with LOD0 / LOD1 and five liveries |

## Requirements

- **Unity 6000.1.7f1** (other Unity 6 versions will likely work but are untested)
- A free [Cesium ion](https://ion.cesium.com/) account — required for terrain and imagery
- An internet connection at runtime; all terrain is streamed, none is bundled

## Getting Started

1. **Clone the repository**

   ```bash
   git clone https://github.com/cakhilesh2008/MachSim-High-Speed-Flight-Simulator.git
   ```

2. **Open the project in Unity Hub** using Unity 6000.1.7f1. The first import takes several
   minutes while Unity builds its asset database.

3. **Add your Cesium ion token** (see below) — without it the terrain renders as nothing.

4. **Open the scene** `Assets/Scenes/SampleScene.unity`. This is the playable scene and the one
   referenced in Build Settings. Opening a different scene will not give you an aircraft.

5. **Press Play.** The aircraft starts near Los Angeles (33.93512, -118.4048). Hold `Space` to
   build throttle, then pull back with `S` once you have airspeed.

## Cesium ion Token Setup

Access tokens are deliberately **not** committed to this repository, so you must supply your own.

1. Sign up at [ion.cesium.com](https://ion.cesium.com/) (the free tier is sufficient).
2. In Unity, open the Cesium panel from the menu bar: **Cesium → Cesium**.
3. Click **Connect to Cesium ion** and sign in. This writes a project-wide default token into
   `Assets/CesiumSettings/Resources/CesiumRuntimeSettings.asset`.

The tileset and imagery overlay components ship with empty token fields, which means they fall
back to that project default — so connecting once is enough. Alternatively, select the
**Google Photorealistic 3D Tiles** GameObject (child of `CesiumGeoreference`) and paste a token
directly into the `Ion Access Token` field on both the `Cesium3DTileset` and
`CesiumIonRasterOverlay` components.

> **Note:** if you fork this project, keep your token out of commits. Unity serializes it into
> the scene and settings asset, so a blanket `git add -A` will publish it.

## Controls

| Input | Action |
| --- | --- |
| `W` / `S` | Pitch down / up |
| `A` / `D` | Roll left / right |
| `Left Arrow` / `Right Arrow` | Yaw left / right |
| `Space` | Throttle up |
| `Left Ctrl` | Throttle down |
| `1` `2` `3` `4` | Switch camera viewpoint |
| `J` | Toggle altitude hold (engage once airborne) |
| `K` | Toggle landing mode |
| `L` | Toggle the landing-gear sphere colliders |

Altitude hold captures your current altitude and holds it. It releases automatically if you give
a deliberate pitch input, engage landing mode, or run low on fuel — the pilot always has authority.

## Project Structure

```
Assets/
  Scenes/SampleScene.unity        The playable scene
  Plane Scripts/
    PlaneController.cs            Flight model, fuel, HUD, autopilot modes
    SoftBodyCageDriver.cs         Joint-based deformation rig (work in progress)
    SoftBodyNodeLink.cs           Data holder for the soft-body rig
  CameraController.cs             Camera viewpoint switching
  Deformation.cs                  Runtime mesh deformation solver (not currently wired up)
  Raptor3D/FA_N26/                Aircraft mesh, prefabs, materials, textures
  CesiumSettings/                 Cesium ion runtime settings
ProjectSettings/
  InputManager.asset              Custom Pitch / Roll / Yaw axes
```

## Flight Model

Forces are applied to a 8,936 kg rigidbody each physics step:

- **Thrust** along the aircraft's forward axis, scaled by throttle
- **Control torques** for pitch, yaw and roll, with roll given double authority
- **Lift** along the aircraft's *own* up axis, scaled by forward airspeed and capped at a
  configurable multiple of aircraft weight — so banking and pitching genuinely redirect lift
- **Velocity and angular velocity clamps** to keep the aircraft controllable

This is a force approximation rather than an aerodynamic simulation: there is no angle-of-attack,
drag curve, or stall modelling.

## Known Limitations

- No `CesiumOriginShift`, so flying very far from the georeference origin will eventually cause
  floating-point precision jitter
- The soft-body deformation rig is incomplete and disabled; `Deformation.cs` is not attached to
  any GameObject
- The engine-status HUD is implemented in code but its Inspector fields are unassigned
- No stall, drag, collision damage or landing gear logic

## Roadmap

- Realistic collision and damage response
- Origin shifting for unlimited-range flight
- Stall and drag modelling
- Improved camera system

## License

Released under the MIT License.
