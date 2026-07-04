# 3D Real Estate Rendering for the Web — Research Guide

> Research compiled 2026-07-04. Hardware context: MacBook M1, 16GB RAM, 256GB SSD.

## The Big Picture

There are three ways real estate gets shown in "3D" on websites, and you'll likely combine them:

1. **True interactive 3D** — a building model rendered live in the browser with WebGL (Three.js). Visitors can orbit, zoom, walk through, click on apartments. This is the core skill to learn.
2. **360° virtual tours** — panoramic photos stitched into a clickable tour (the "Matterport style"). Much easier, very in-demand, and free to build yourself.
3. **Pre-rendered visuals** — photorealistic stills/videos made in Blender, embedded as media. Highest visual quality, no browser performance limits.

**Recommended stack:** Blender (create/optimize models) → export **glTF/GLB** → **Three.js** (or react-three-fiber with React) to render on the website. Everything in this stack is free and runs natively on Apple Silicon.

---

## 1. Core Web Technology

| Tool | What it is | Link |
|---|---|---|
| Three.js | The standard JavaScript 3D library; free docs + hundreds of live examples | https://threejs.org/ |
| react-three-fiber + drei | React wrapper for Three.js; `useGLTF` + `OrbitControls` gets a building on screen in ~20 lines | https://r3f.docs.pmnd.rs/tutorials/loading-models |
| gltfjsx | Converts a GLB into a reusable React component | (part of the pmndrs/R3F ecosystem) |
| `<model-viewer>` | Google's single HTML tag that displays a GLB with orbit controls + AR — the zero-code embed option | https://modelviewer.dev/ |

- The key format is **glTF/GLB** — compact, fast-loading, the "JPEG of 3D." [Sketchfab](https://sketchfab.com/features/gltf) and Three.js both treat it as the default.
- Tutorials: [LogRocket's react-three-fiber guide](https://blog.logrocket.com/configure-3d-models-react-three-fiber/) · [Next.js + R3F + drei walkthrough](https://medium.com/@valentinagaravaglia/rendering-a-3d-model-with-next-js-13-typescript-react-three-fiber-and-react-three-drei-84476c3b6a5d)

## 2. Free Learning Materials

### Three.js
- Official Three.js manual + examples (free, the best single reference): https://threejs.org/
- [Learn With Hasan's 2026 Three.js guide](https://learnwithhasan.com/threejs-guide/) — interactive in-browser demos
- [Three.js Resources hub](https://threejsresources.com/) — curated tools/courses/examples
- [Three.js Journey](https://threejs-journey.com/) — the famous course is paid, but the first lessons and the beginning of every lesson are free
- Free YouTube course roundups: [Class Central](https://www.classcentral.com/report/best-three-js-courses/) · [10 free Three.js YouTube courses (2026)](https://invernessdesignstudio.com/10-free-three-js-youtube-courses-to-learn-3d-in-2026) · [Coursesity 25+ free courses](https://coursesity.com/free-tutorials-learn/three-js)

### Blender (free, native on M1)
- [Architectural Modeling in Blender — modeling from a floor plan](https://www.youtube.com/watch?v=Xb4ddBuiTU0) — blueprint image → 3D building
- [How to scale a floor plan in Blender](https://www.youtube.com/watch?v=8iMWUsqeopQ) — critical first step for real-world dimensions
- [3D floor plan beginner's guide](https://www.youtube.com/watch?v=4TCTAuv15Gs)
- [Exterior archviz overview](https://www.youtube.com/watch?v=HYkMd6FX8jM)
- [Blender 3D Architect](https://www.blender3darchitect.com/cad/sketching-floor-plan-blender/) — a whole site dedicated to architecture in Blender
- Free add-ons: **Archimesh/Archipack** (parametric walls, doors, windows — ships with Blender), **Import Images as Planes** (trace blueprints)

## 3. Free Blueprints & Floor Plans

Real CAD drawings to trace in Blender:

- [DWG NET](http://www.dwgnet.com/) — free small-house plans, no registration ([example set](http://www.dwgnet.com/2025/04/house-plan-drawings-free-download-for-small-family/))
- [FreeCadFloorPlans.com](https://freecadfloorplans.com/)
- [FreeCads.com](https://www.freecads.com/cad/free-floor-plan-cad-drawings-dwg-dxf-pdf-format/) — includes a free multi-unit apartment building plan
- [DWGModels.com](https://dwgmodels.com/223-house.html) · [FreeCadFiles.com](https://www.freecadfiles.com/2020/12/modern-house-plan-dwg.html) · [DWGFree.com](https://dwgfree.com/category/free-autocad-house-plans-drawings/) · [DWGShare](https://dwgshare.com/)

Opening DWG on Mac: use **ODA File Viewer** (free), or convert DWG→DXF and import into Blender; often easiest to export the plan as an image and trace it.

Browser floor-plan tools with free tiers: [Homestyler](https://www.homestyler.com/solution/design_floor-planner) · [Coohom](https://www.coohom.com/case/floor-planner)

## 4. Free 3D Models, Textures & Images

### Ready-made building models
- [Sketchfab](https://sketchfab.com/features/gltf) — 800k+ free CC-licensed models, all downloadable as glTF. Example: [modern residential building GLB](https://sketchfab.com/3d-models/modern-residential-building-glb-818c217cab6048898ae5f1848219e0c1)
- [TurboSquid free architecture GLB](https://www.turbosquid.com/3d-model/free/architecture?keyword=building+glb)
- [CGTrader floor plans/buildings](https://www.cgtrader.com/3d-models/floor-plan)
- [Get3DModels buildings](https://www.get3dmodels.com/architecture/building/)
- **3D Warehouse** (SketchUp) — huge library of real buildings; more sources in [Visody's roundup](https://visody.com/download-3d-models-for-free/)
- Always check the license per model (CC-BY needs attribution; some are personal-use only).

### Textures, materials, HDRIs (CC0 — free for anything, including commercial)
- [Poly Haven](https://polyhaven.com/) — photoscanned 8K PBR [textures](https://polyhaven.com/textures) + [HDRIs](https://polyhaven.com/hdris) (the trick for realistic lighting in Blender *and* Three.js). No registration ([FAQ](https://docs.polyhaven.com/en/faq))
- **ambientCG** (https://ambientcg.com/) — the biggest CC0 PBR material library: brick, concrete, plaster, roofing, wood floors
- Comparison of both: [Chaos free-textures roundup](https://blog.chaos.com/free-rendering-textures)

### Reference photos of real buildings
- Unsplash (https://unsplash.com/) and Pexels (https://www.pexels.com/) — free-license building/facade photography for reference and texturing.

## 5. Free GitHub Repos to Study & Build On

| Repo | Why it matters |
|---|---|
| [furnishup/blueprint3d](https://github.com/furnishup/blueprint3d) | Draw a 2D floor plan, see it in 3D, place furniture — the classic open-source starting point (older code, great to study) |
| [sumandeepb/OpenSpace](https://github.com/sumandeepb/OpenSpace) | 3D virtual tour web app built for real estate cataloging (Three.js + WebGL2) |
| [jpayansh/3d-building-visualizer](https://github.com/jpayansh/3d-building-visualizer) | Buildings on a real map: MapLibre GL + Three.js + Threebox, markers, routes — "building in its neighborhood" views |
| [CodeHole7/threejs-3d-room-designer](https://github.com/CodeHole7/threejs-3d-room-designer) | React + Three.js room planner / product configurator |
| [AyushGupta011/3d-real-estate](https://github.com/AyushGupta011/3d-real-estate) | Simple real-estate site with 3D property views — easy first codebase to read |
| [benna100/house-3d-model-three-js](https://github.com/benna100/house-3d-model-three-js) | Small single-house Three.js scene — perfect first-project scope |
| [devXprite/3d-house](https://github.com/devXprite/3d-house) | Another minimal 3D house in Three.js |

Inspiration & discussion: [Real Estate Showcasing Project (Three.js forum)](https://discourse.threejs.org/t/real-estate-properties-showcasing-project/56343) · [Archilogic three.js + React viewer case study](https://www.archilogic.com/project/building-a-3d-viewer-with-three-js-react-and-the-archilogic-embed-api) · browse [github.com/topics/real-estate](https://github.com/topics/real-estate)

## 6. 360° Virtual Tours (free Matterport alternative)

- [Pannellum](https://github.com/mpetroff/pannellum) — 21KB, MIT; hotspots link rooms into full tours; [React wrapper](https://github.com/farminf/pannellum-react)
- [Marzipano](https://www.marzipano.net/) — Google's free 360° viewer ([source](https://github.com/google/marzipano)) with a visual tour-building tool
- **Photo Sphere Viewer** (MIT) — most advanced hotspot/marker system; see the [2026 comparison of open-source tour libraries](https://portalzine.de/open-source-virtual-tour-360-panorama-libraries-in-javascript-2026/)
- Capture panoramas with any phone (e.g. Google Street View app) — no special camera needed to start.

## 7. Photogrammetry — Photos → 3D Models (M1 advantage)

Apple's **Object Capture API** runs natively on M1 and turns a set of photos into a textured 3D model in minutes:
- [Apple docs](https://developer.apple.com/documentation/realitykit/realitykit-object-capture/) · [WWDC session](https://developer.apple.com/videos/play/wwdc2021/10076/)
- [PhotoCatch](https://9to5mac.com/2021/06/23/photocatch-lets-you-easily-create-3d-models-using-apples-new-object-capture-api/) — free Mac app wrapping the API; shoot photos with your phone, drop them in, get a model
- Works for building exteriors too, including [from drone video](https://ar-code.com/blog/video-to-3d-modeling-photogrammetry-ar-code-object-capture-now-on-macbook-m-series); M1 workflow background: [Blender Artists thread](https://blenderartists.org/t/free-3d-photogrammetry-with-macos-monterey-on-m1-intel-chips/1335447)
- Skip **Meshroom** — it needs an NVIDIA/CUDA GPU, which Macs don't have.

## 8. City-Scale Context (bonus)

- [Cesium OSM Buildings](https://cesium.com/platform/cesium-ion/content/cesium-osm-buildings/) — streams 350M+ real-world buildings derived from OpenStreetMap, free with CesiumJS
- Tutorial: [visualize a proposed building in a 3D city](https://cesium.com/learn/cesiumjs-learn/cesiumjs-interactive-building/)
- [OSM Buildings](https://osmbuildings.org/) — lighter open-source viewer for the same data ([OSMF announcement](https://blog.openstreetmap.org/2020/08/17/osmf-and-cesium-joint-press-release-openstreetmap-a-map-of-buildings/))

## M1 (16GB / 256GB) Notes

- **Blender runs natively and well** with Metal GPU rendering. Use **Eevee** (real-time) for most work; reserve Cycles for final stills. Keep scenes under a few million polygons.
- **Web 3D is inherently light** — browser scenes must be small anyway, so the hardware is a non-issue for Three.js development.
- **The 256GB SSD is the real constraint.** Download 1–2K texture resolutions (not 8K) from Poly Haven, delete raw photogrammetry photo sets after processing, and compress models with **Draco** (geometry) + **KTX2** (textures) via `gltf-transform` — a 100MB Blender export often becomes a 5MB GLB.
- 16GB RAM: close browser tab hoards while rendering in Blender; otherwise fine.

## Suggested Learning Path (~8–10 weeks)

1. **Week 1–2 — Three.js fundamentals.** Scene, camera, lights, OrbitControls, load a free GLB from Sketchfab. *Goal: any building spinning in a browser.*
2. **Week 3–4 — Blender basics.** Trace a free DWG NET floor plan into a simple house, texture with ambientCG materials, light with a Poly Haven HDRI, export GLB.
3. **Week 5–6 — Your building on a real webpage.** react-three-fiber (or plain Three.js), clickable hotspots ("click a unit to see details"), Draco compression.
4. **Week 7–8 — Tours & capture.** Add a 360° tour with Pannellum; try PhotoCatch on a real object/building.
5. **Then — portfolio piece.** Study OpenSpace and blueprint3d, then build a listing page with an orbitable exterior, floor-plan switcher, and virtual tour. That's exactly what real-estate clients pay for.
