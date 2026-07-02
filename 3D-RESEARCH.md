# 3D Platform Research

> **Goal of this document:** scope the technology, hardware, and knowledge needed to build:
> 1. A **3D simulation app** (physics, movement, environments)
> 2. **3D modeling apps** (create/edit 3D geometry)
> 3. A **website** that presents real estate, ships, and physical buildings in **interactive 3D**
> 4. A pipeline to **turn real-world physical objects/buildings into 3D**
>
> Scope: tech-stack recommendations, minimum hardware, and a beginner→advanced learning path. **No code yet** — this is the map before the build.

---

## 0. TL;DR — The one-paragraph version

You are really describing **three connected products** that share a foundation: (a) a **web 3D viewer** for real estate/ships/buildings, (b) a **simulation app** with physics, and (c) a **capture pipeline** that converts real buildings into 3D assets. The pragmatic modern stack is: **React + React Three Fiber (Three.js)** for the web viewer, **glTF** as your universal asset format, a **Gaussian Splatting + photogrammetry** pipeline to digitize real buildings, and either **Three.js/Babylon.js** (web) or **Unreal/UNIGINE** (native) for the heavy simulation work depending on how realistic the physics must be. Start with the web viewer — it's the smallest slice that delivers value and teaches you everything else.

---

## 1. Breaking the vision into buildable pieces

Trying to build "a 3D everything app" at once will fail. Decompose it into capabilities. Each row below is something you can build and ship independently.

| # | Capability | What it does | Hardest part |
|---|-----------|--------------|--------------|
| A | **Web 3D viewer** | Show a building/ship/property in the browser, orbit/zoom/walk | Performance on low-end devices |
| B | **Asset pipeline** | Convert models/scans into a clean, optimized format | File formats, optimization, LODs |
| C | **Real-world capture** | Turn a physical building/ship into a 3D asset | Photogrammetry / Gaussian Splatting |
| D | **Modeling tool** | Let users create/edit geometry | 3D math, UI, undo/history, geometry kernel |
| E | **Simulation** | Physics: gravity, collision, water, vehicles | Real-time physics + performance |
| F | **Backend/platform** | Accounts, storage, sharing, streaming large assets | Serving GB-scale 3D data cheaply |

**Recommended build order:** A → B → C → (E or D). Do **A first**. It is the smallest thing that produces a visible result and forces you to learn coordinates, cameras, meshes, materials, and lighting — the vocabulary everything else builds on.

---

## 2. Core concepts (beginner → advanced glossary)

Read top-to-bottom. Each term builds on the previous ones.

### 2.1 Beginner — the vocabulary you cannot skip
- **Vertex / Edge / Face / Mesh** — a 3D model is points (vertices) connected into triangles (faces). A **mesh** is the whole collection.
- **Polygon count (poly count / tris)** — how many triangles. More = more detail but slower. Budgeting this is 80% of real-time 3D.
- **Coordinate system** — X (right), Y (up), Z (depth). Note: engines disagree on which axis is "up" and handedness — a constant source of bugs.
- **Transform** — an object's **position**, **rotation**, and **scale**.
- **Camera** — your viewpoint. **Perspective** (realistic, things shrink with distance) vs **Orthographic** (no perspective, used in CAD).
- **Material / Texture** — the surface look. A **texture** is an image wrapped onto a mesh; a **material** describes how it reacts to light.
- **UV mapping** — how a 2D texture image is unwrapped onto a 3D surface.
- **Light types** — ambient, directional (sun), point (bulb), spot.
- **Render / Rendering** — turning the 3D scene into a 2D image on screen, ~60 times per second.

### 2.2 Intermediate — how it actually runs
- **PBR (Physically Based Rendering)** — the modern standard for realistic materials (metalness/roughness). glTF uses it; learn it.
- **Normal / bump maps** — fake surface detail (bricks, bumps) without extra polygons.
- **LOD (Level of Detail)** — swap a high-poly model for a low-poly one when far away. Essential for big scenes (a whole building, a city block).
- **Draw calls** — each object sent to the GPU costs a "call." Too many = slow. **Batching/instancing** reduces them.
- **Scene graph** — the tree of all objects, parent/child relationships (a door is a child of a building).
- **glTF / GLB** — the "JPEG of 3D," the web-standard delivery format. `.glb` is the single-file binary version. **This is your target format.**
- **Frame budget** — 60 FPS = **16.6 ms per frame** to do everything. This is the constraint you design around.
- **Culling** — don't render what the camera can't see (frustum culling, occlusion culling).

### 2.3 Advanced — the deep end
- **Shaders (GLSL / WGSL)** — small programs that run on the GPU to compute every pixel/vertex. Where custom visual effects and performance live.
- **WebGL vs WebGPU** — WebGL is the mature browser graphics API; **WebGPU** is its successor (better performance, compute shaders), production-ready in 2025–26 in both Three.js (r171+) and Babylon.js (v5+).
- **Rasterization vs Ray tracing** — rasterization (fast, real-time, games) vs ray tracing (accurate light/reflections, slower, film & high-end).
- **Physics engine** — rigid bodies, collision detection, constraints, buoyancy (for ships), vehicle dynamics.
- **NeRF / Gaussian Splatting** — AI/neural methods to reconstruct photoreal 3D scenes from photos (see §5).
- **BIM / IFC** — Building Information Modeling; the data-rich standard for architecture/construction. Relevant if "buildings" means real architectural data, not just visuals.
- **Digital twin** — a live, data-connected 3D replica of a real asset (building, ship, factory).
- **BVH / spatial partitioning** — data structures that make "what did I click / what's near me" fast at scale.

---

## 3. Recommended tech stacks

There is no single stack — it depends on the piece. Below are concrete, opinionated recommendations per capability.

### 3.1 Web 3D viewer (Capability A) — **start here**

| Layer | Recommendation | Why |
|-------|---------------|-----|
| Rendering engine | **Three.js** (or **React Three Fiber** if using React) | Largest ecosystem (~3.5M weekly downloads), lightest (~168 kB gzipped), most learning material. R3F is the industry standard for declarative 3D in React. |
| Framework | **React + React Three Fiber (R3F)** + **@react-three/drei** | Drei gives you cameras, controls, loaders, and helpers out of the box. |
| Physics (light) | **Rapier** (`@react-three/rapier`) | Fast Rust/WASM physics, integrates cleanly with R3F. |
| Asset format | **glTF / GLB** | Web-native, PBR, compressible. |
| Compression | **Draco** (geometry) + **KTX2/Basis** (textures) + **meshopt** | Shrinks assets 5–10×; critical for load times. |
| Alternative engine | **Babylon.js** | Choose if you want a batteries-included engine (built-in physics, XR, visual editor, strongest WebGPU) or a large enterprise scene loader. |

**When to pick Babylon over Three.js:** large scenes with dozens of models loading in parallel, you want a built-in editor, or you value an all-in-one "framework" feel over Three's "library" flexibility.

### 3.2 Real estate / walkthrough website

- **Viewer:** R3F + Drei (as above).
- **For photoreal captured spaces:** a **Gaussian Splatting** web viewer (e.g. renderers that plug into Three.js) — best-in-class for "walk through this actual house" realism.
- **For designed/CAD buildings:** glTF exported from Blender/Revit/SketchUp.
- **Meta-note:** the incumbent commercial product here is **Matterport**; your differentiator is either price, self-hosting, or a custom experience.

### 3.3 3D modeling app (Capability D) — hardest; consider deferring

- **Web-based:** Three.js/Babylon.js for viewport + your own tools, or fork/learn from the open-source **three.js editor**.
- **Reality:** a real modeling app needs a **geometry kernel** (boolean ops, NURBS/B-rep for CAD, or subdivision for organic). Options: **OpenCASCADE** (CAD/B-rep, has WASM builds), or mesh-based tooling.
- **Strong recommendation:** don't build a modeling app from scratch early. Instead, **integrate Blender** (free, open-source, scriptable via Python) into your pipeline and build a thin custom UI only for the operations your users actually need.

### 3.4 Simulation app (Capability E) — pick by realism required

| If you need… | Use | Notes |
|---|---|---|
| Web-based, moderate physics | **Three.js/Babylon.js + Rapier or Ammo.js** | Ships in a browser; good enough for interactive demos. |
| High-fidelity native (real ship/vehicle dynamics, training sims) | **Unreal Engine 5.x** or **UNIGINE** | UNIGINE is purpose-built for simulation/training, digital twins, CAD/BIM/GIS. Unreal has the best visuals + huge ecosystem. |
| Custom/scientific physics | **Unity** + **Havok**, or **bepuphysics2** (C#), or **Matali Physics** | Unity is the most popular training-sim platform. |
| Buoyancy/water for ships | Unreal water system, or custom buoyancy on Rapier/PhysX | Water + floating-body dynamics is genuinely hard. |

**Decision rule:** if it must run **in a browser with no install** → Three/Babylon. If it needs **film-grade visuals or hard-real physics** and a native app is acceptable → **Unreal or UNIGINE**.

### 3.5 Backend / platform (Capability F)

- **Frontend:** React/Next.js.
- **API:** Node.js or Python (FastAPI). Python is convenient because the capture/ML pipeline (§5) is Python-heavy.
- **3D asset storage:** object storage (S3/R2/GCS) + a **CDN** — 3D assets are large; serve them from the edge.
- **Streaming large models:** consider **3D Tiles** (OGC standard) for city/large-building streaming; **glTF + Draco** for individual assets.
- **Database:** Postgres for metadata; the heavy binary lives in object storage, not the DB.
- **GPU compute:** the capture pipeline (photogrammetry/splatting) needs GPUs — run it as offline jobs (cloud GPU or a local workstation), not in the request path.

---

## 4. File formats you must know

| Format | Use | Notes |
|--------|-----|-------|
| **glTF / GLB** | Web delivery | **Your primary target.** PBR, animations, compact. |
| **OBJ** | Simple interchange | Old, universal, no animation, poor materials. |
| **FBX** | DCC interchange (Maya/Max) | Common in games; proprietary (Autodesk). |
| **USD / USDZ** | Large-scene pipelines, Apple AR | Pixar's standard; increasingly the pro backbone. USDZ = iOS AR Quick Look. |
| **STL** | 3D printing / raw meshes | Geometry only, no color/material. |
| **IFC** | BIM / architecture | Data-rich building models (walls, doors as real objects). |
| **PLY / SPLAT / .ksplat** | Point clouds & Gaussian Splats | Output of scanning/splatting pipelines. |
| **3D Tiles** | Streaming huge scenes | Cities, large sites; served like map tiles. |

---

## 5. Turning real buildings & ships into 3D (Capability C)

This is the "digitize the real world" pipeline. In 2026 there are four main techniques — you'll likely use **two together**.

| Method | What it is | Best for | Trade-off |
|--------|-----------|----------|-----------|
| **Photogrammetry** | Reconstruct an **editable textured mesh** from overlapping photos | Facades, ground planes, assets that must enter a 3D pipeline; when geometric accuracy matters | Struggles with reflective/transparent surfaces; more manual cleanup |
| **Gaussian Splatting (3DGS)** | Captures the scene as photoreal "light"; renders in real time, **web-native** | Photoreal real-estate/interior tours; stunning visual presentation; reflections | Not a conventional editable mesh; harder to measure/edit |
| **NeRF** | Neural radiance fields (predecessor to 3DGS) | Research; being superseded | Slower to render than 3DGS |
| **LiDAR scanning** | Laser distance measurement → precise point cloud | Millimetre-accurate dimensions, engineering, large ships/structures | Hardware cost; no color by itself |

**2026 state of the art:** Gaussian Splatting has overtaken NeRF for photoreal real-time work because it renders far faster and runs natively on the web. Training a room-scale scene takes ~20–40 min on an **RTX 3060**. Most professional workflows use **both**: photogrammetry/LiDAR for accurate engineering geometry, **3DGS for visualization**, from the same drone/scanner captures.

**Practical recommendation for real estate + buildings + ships:**
1. **Capture:** phone/DSLR photos or drone flights (exterior); add **LiDAR** (many recent iPhones/iPads have it) for scale-accurate interiors.
2. **Reconstruct:** photogrammetry (e.g. RealityCapture, Metashape, or open-source **COLMAP** + **Meshroom**) **and/or** a Gaussian Splatting trainer.
3. **Deliver:** glTF mesh for editable/measurable models; a **splat viewer in Three.js** for photoreal web tours.

Tools to evaluate: **RealityCapture**, **Agisoft Metashape**, **Polycam** (mobile, easy), **Luma AI**, **Postshot / Nerfstudio / gsplat** (splatting), **COLMAP + Meshroom** (open-source).

---

## 6. Minimum hardware specifications

3D has **two very different hardware needs**: (1) the **development/creation** machine and (2) what **end users** need to view your product. The capture/AI pipeline is the demanding part.

### 6.1 Developer / creator workstation

| Tier | CPU | GPU | RAM | Storage | Good for |
|------|-----|-----|-----|---------|----------|
| **Minimum** | Modern 6-core (Ryzen 5 / Core i5, or Apple M-series) | **NVIDIA RTX 3060 (8–12 GB VRAM)** | **16 GB** | 512 GB SSD | Web 3D dev, small models, learning, entry Gaussian Splatting |
| **Recommended** | 8-core (Ryzen 7 / Core i7 / M-Pro) | **RTX 4070 / 4080 (12–16 GB VRAM)** | **32 GB** | 1 TB NVMe SSD | Photogrammetry, comfortable splatting, Unreal/Blender |
| **Ideal / heavy** | 12–16-core (Ryzen 9 / Core i9 / M-Max/Ultra) | **RTX 4090 / pro card (24 GB+ VRAM)** | **64 GB+** | 2 TB+ NVMe | Large photogrammetry sets, big scenes, fast splat training, high-fidelity sim |

**Why VRAM matters most:** Gaussian Splatting, photogrammetry, and large scenes are bottlenecked by GPU memory, not CPU. **Prioritize VRAM.** For the AI/capture pipeline, an **NVIDIA (CUDA) GPU is effectively required** — most tools assume CUDA. Apple Silicon is excellent for web dev and Blender but weaker for CUDA-only splatting/photogrammetry tools.

### 6.2 End-user (someone viewing your website)

- **Web 3D viewer:** any laptop/phone from the last ~4 years with a modern browser (Chrome/Edge/Safari/Firefox) that supports WebGL2/WebGPU. Design for **mobile** — most real-estate viewers are opened on phones.
- **Gaussian Splat web tours:** a bit heavier; a recent mid-range phone or laptop handles a single scene. Optimize splat count.
- **Rule:** your **optimization work** (LODs, compression, culling) is what lets low-end user devices run your content. Budget engineering time for it.

### 6.3 Capture hardware

- **Entry:** a recent smartphone (bonus if it has **LiDAR**).
- **Better exteriors:** a **drone** (e.g. DJI) for buildings and ships.
- **Professional/accurate:** a dedicated **LiDAR scanner** (terrestrial or handheld) for engineering-grade dimensions.

---

## 7. Learning path (beginner → advanced)

A concrete curriculum. Don't skip the math — it's the difference between copying tutorials and building real things.

### Phase 1 — Foundations (weeks 1–4)
- **Three.js Journey** (Bruno Simon) — the canonical, comprehensive course.
- Official **Three.js manual** + examples.
- Learn **Blender** basics (Blender Guru "Donut" tutorial) — even if you won't model, you must understand meshes, materials, UVs, export.
- Concepts: vertices, transforms, cameras, lights, materials, the render loop.

### Phase 2 — The math (ongoing, alongside Phase 1)
- **Vectors** (dot/cross product), **matrices**, **transforms**, **quaternions** (rotation without gimbal lock), **projection**.
- Resources: *3Blue1Brown* "Essence of Linear Algebra" (intuition), then a graphics-math book/course.
- You don't need to derive everything, but you must be fluent reading it.

### Phase 3 — Building the web viewer (weeks 4–10)
- React + **React Three Fiber** + **Drei**.
- Load **glTF** models, add **OrbitControls**, lighting, environment maps.
- Learn **Draco/KTX2 compression** and the **glTF pipeline**.
- Ship capability A end-to-end for one property/ship.

### Phase 4 — Capture pipeline (weeks 8–14)
- Photogrammetry with **Meshcape/RealityCapture/Polycam**.
- Gaussian Splatting with **Nerfstudio / gsplat / Postshot**.
- Get a splat rendering **in your Three.js viewer**.

### Phase 5 — Physics & simulation (weeks 12+)
- **Rapier** with R3F: gravity, collisions, rigid bodies.
- If native fidelity needed: pivot to **Unreal** or **UNIGINE** and learn its workflow.

### Phase 6 — Advanced (ongoing)
- **Shaders** (GLSL/WGSL) — start with *The Book of Shaders*.
- **WebGPU**, performance profiling, LOD systems, large-scene streaming (**3D Tiles**).
- **BIM/IFC** if real architectural data is in scope.

---

## 8. Key risks & realistic expectations

- **Scope is enormous.** Simulation + modeling + web + capture is *four startups*. Sequence them; ship the web viewer first.
- **Performance is the real product.** Anyone can load a model; making it run at 60 FPS on a phone is the hard, valuable part.
- **Modeling apps are the deepest hole.** A production modeling/CAD tool needs a geometry kernel and years of work. Integrate Blender/OpenCASCADE instead of reinventing it.
- **The capture pipeline needs an NVIDIA GPU** and is compute-heavy — budget for a workstation or cloud GPU.
- **Water/ship physics is genuinely hard.** Buoyancy, waves, and floating-body dynamics are a specialty; lean on Unreal's water system or accept simplification.
- **Large assets = large bills.** Serving GB-scale 3D needs a CDN and compression strategy from day one.

---

## 9. Concrete next steps

1. **Pick the first slice:** build a **web 3D viewer** (R3F + Drei) that loads **one** `.glb` of a building or ship and lets the user orbit it.
2. **Get one real capture done:** photograph a real building, run it through **Polycam** or **Meshroom**, and load the result into your viewer.
3. **Decide the simulation path:** in-browser (Three + Rapier) vs native (Unreal/UNIGINE) — this choice shapes everything downstream.
4. **Set up the workstation** per §6.1 (prioritize an NVIDIA GPU with ≥12 GB VRAM if you'll do capture).
5. Only **after** A–C work, evaluate whether you truly need a custom modeling app (D) or can integrate Blender.

---

## Sources

- [Gaussian Splatting vs Photogrammetry (2026) — Polyvia3D](https://www.polyvia3d.com/formats/3dgs-vs-photogrammetry)
- [Gaussian Splatting vs Photogrammetry vs NeRF vs LiDAR — Utsubo](https://www.utsubo.com/blog/gaussian-splatting-vs-photogrammetry-nerf-lidar)
- [Photogrammetry vs Gaussian Splatting — Splatware](https://splatware.com/learn/photogrammetry-vs-gaussian-splatting)
- [The State of Gaussian Splatting in 2026 — The Future 3D](https://www.thefuture3d.com/blog/state-of-gaussian-splatting-2026/)
- [Three.js vs React Three Fiber vs Babylon.js 2026 — PkgPulse](https://www.pkgpulse.com/guides/threejs-vs-react-three-fiber-vs-babylonjs-3d-webgl-2026)
- [Three.js Alternatives: Babylon.js vs PlayCanvas (2026) — Utsubo](https://www.utsubo.com/blog/threejs-vs-babylonjs-vs-playcanvas-comparison)
- [Three.js vs Babylon.js — LogRocket](https://blog.logrocket.com/three-js-vs-babylon-js/)
- [Web game engines in 2026: PlayCanvas vs Three.js vs Babylon.js vs Unity WebGL — Cinevva](https://app.cinevva.com/blog/2026-06-09-web-game-engines-2026-comparison.html)
- [UNIGINE — real-time 3D engine for simulation, digital twins, CAD/BIM/GIS](https://unigine.com/)
- [Unreal Engine](https://www.unrealengine.com/)
- [bepuphysics2 — pure C# real-time physics](https://github.com/bepu/bepuphysics2)

---

*Document created 2026-07-02. This is a research/planning document — no code included by design.*
