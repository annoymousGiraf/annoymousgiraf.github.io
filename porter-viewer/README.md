# Porter robot — interactive 3D prototype

Self-contained browser visualization of the **Porter** removalist robot spec (Australian pantech / house-move workflow). No build step.

## Files

| File | Purpose |
|------|---------|
| `docs/porter-robot-3d.html` | Primary viewer (Three.js via CDN) |
| `media/porter-robot-3d.html` | Same viewer (copy for media folder) |

## How to open locally

Browsers block ES-module imports from `file://`. Serve the folder over HTTP:

```bash
cd /cursor/stores/bc-2ec988dd-49b1-4fca-93e9-2a0e4a60701e
python3 -m http.server 8765 --bind 127.0.0.1
```

Then open:

**http://127.0.0.1:8765/docs/porter-robot-3d.html**

(Or use `media/porter-robot-3d.html` if you prefer that path.)

Requires network access on first load to fetch Three.js from jsDelivr CDN.

## Controls

| Input | Action |
|-------|--------|
| Left drag | Orbit camera |
| Right drag | Pan |
| Scroll | Zoom |
| **Show AU door frame** | Toggle 820 × 2040 mm reference frame beside the robot |
| **Door-fit overlay** | Light blue plane on the door opening |
| **Tilt carton for door clearance** | Preset ~22° forward tilt on the carton |
| **Carton tilt slider** | Manual tilt 0–35° |
| **700 mm envelope** | Translucent width guide with box in front |
| **Floor dimension guides** | Base width and envelope labels |
| **Hover door / envelope parts** | Optional raycast on reference geometry (off by default) |
| **Run demo** / **Stop** | Scripted removalist loop (park → drive → floor pickup → door tilt → drop zone → return). Default **stopped** |
| **Demo speed** | 0.5×–2× time scale for the animation loop (~62 s per cycle at 1×) |
| **Follow robot** | Optional gentle `OrbitControls` target tracking (off by default — orbit stays manual) |

## Demo animation

Time-based **state machine** in `animate()` drives a repeatable workflow:

1. **Park** at the staging pose (rear-left of the grid).
2. **Drive** along **piecewise straight segments** with **turn-in-place** at waypoints to a **pickup approach stop** (belly ~350&nbsp;mm in front of the floor carton — not driven over the box).
3. **Pick up** — lower lift, belly tilt-in, then a **~0.4&nbsp;s world-space slide** of the floor carton into the tray before parenting to `bellyGroup`; lift raises.
4. **Carry** toward the AU door reference, with a short **~22° tilt** preset through the door line.
5. **Drop off** at the **drop zone** — lower, **slide carton down** to the pad, then release (stays in the scene).
6. **Return** toward park and **loop** (clears dropped carton for the next cycle).

Implementation details:

- Phases live in `demoScript[]` with per-phase `duration`, `enter()`, and `update(t)` where `t ∈ [0, 1]`.
- Driving uses `expandDrivePath()` / `updateDrivePath()` (separate **x/z** straight legs and **in-place** `rotation.y` turns; no simultaneous position+heading arc).
- `PICKUP_BOX` / `PICK_APPROACH` share one floor-carton anchor; orange **approach line** on the grid marks the final leg.
- Scissor height uses `applyLiftRatio`; belly uses `applyDemoBellyTilt` (carton + clamp lip on `cartonPivot`, U-tray frame ≤ 8°); pickup/drop cartons lerp in world space via `scene.attach()` before re-parenting.
- **Hover** raycast is paused while the demo runs (resumes on **Stop**).
- **Follow robot** lightly lerps the orbit target toward the robot — camera orbit/zoom remain user-controlled.

## Hover materials

Move the pointer over any **robot** mesh (throttled raycast, ~48&nbsp;ms). A **fixed panel** (top-right) and a **cursor tooltip** show:

- **Part name** — build component (chassis, wheels, scissor links, mast rails, belly U-tray, carton, beacon, sensor head, etc.)
- **Material** — suggested real-world stock for an MVP removalist build (e.g. 3&nbsp;mm steel RAL&nbsp;7035, PU 100&nbsp;mm castors, 40×40 anodised extrusion)
- **Note** — one-line fabrication or usage hint

Meshes carry `userData = { label, material, note, hoverable }` via the `tagPart()` helper. **Door frame** and **700&nbsp;mm envelope** guides are excluded unless **Hover door / envelope parts** is checked.

Visual tweaks in this revision: **`sensorStack`** group (neck strut + head + lidar window, contiguous from chest top), **beacon on rear chassis corner** (not floating above the head), **torus-profile tyres** with a darker tread band, **M8 bolts** on the scissor platform plate.

## Scale notes

All geometry uses **1 scene unit = 1 metre**.

| Element | Dimension |
|---------|-----------|
| Total robot height | 1.75 m |
| Wheeled base width | 550 mm |
| Typical carton | 450 × 400 × 600 mm (W × D × H) |
| Max envelope (with carton forward) | 700 mm wide |
| AU door reference | 820 mm clear width × 2040 mm height |

Styling: matte grey warehouse-tug chassis, connected scissor struts, U-shaped belly tray (belly clamp), orange status strip and bumper, cardboard-brown carton, neutral light-grey studio background.

## Troubleshooting

**Sensor head floating above the neck** — Earlier builds placed the head at a fixed `ROBOT_HEIGHT - 0.09` while the neck cap sat near `liftTop + 0.48`, leaving a visible air gap (~17&nbsp;cm). Fix: parent neck, head, and lidar window under a `sensorStack` group with contiguous local Y (neck base on chest top at `liftTop + 0.47`, tall neck strut, head seated on the strut). Prefer physical connection over pinning the head to an arbitrary 1.75&nbsp;m line.

**Carton clipping through sensor stack on tilt** — Rotating the whole `bellyGroup` swung the carton top into the fixed chest `sensorStack`. Fix: `cartonPivot` inside the belly holds the clamp lip + carton (0–35° slider; demo door pass ~22°); the U-tray frame only tilts up to 8°. The 700&nbsp;mm envelope guide fades slightly at high carton tilt (it stays world-fixed).

## Technology

- [Three.js](https://threejs.org/) r170 (import map → jsDelivr)
- `OrbitControls` for navigation
- Procedural primitives only (boxes, cylinders, planes, line helpers)