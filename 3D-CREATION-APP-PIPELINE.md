# Building a 3D Creation App: From Uploaded Images to Walkable Buildings

> Research compiled 2026-07-04. Companion to [3D-REAL-ESTATE-RESEARCH.md](3D-REAL-ESTATE-RESEARCH.md).
> The question this answers: *"How can users upload captured images (e.g., the front of a house, then its interior) into an app, and have the app layer them so a visitor can scroll from outside to inside?"*

---

## 0. First, an important reality check

A single flat photo **cannot** become a walkable 3D space by itself — a photo has no depth information. Every app that turns uploads into 3D (Matterport, Polycam, Zillow 3D Home) does one of these:

| What the user uploads | What the app can produce | Technique |
|---|---|---|
| 1 ordinary photo | A rough 3D guess of a single *object*, or a flat "billboard" backdrop | AI single-image reconstruction |
| 20–200 overlapping photos of the same subject | A real textured 3D mesh | Photogrammetry (Structure-from-Motion) |
| A phone video / photo orbit | A photoreal "splat" scene you can move through | 3D Gaussian Splatting |
| 360° panoramas (one per room/position) | A clickable virtual tour with scene-to-scene movement | Linked panorama nodes |
| Photos **+** depth (LiDAR phone, Matterport cam) | A full doll-house model + smooth walkthrough | Mesh + panorama hybrid (how Matterport works) |

**Your "front photo → add interior photo as a layer" idea is exactly the linked-panorama-node model** (row 4). Matterport itself works this way: it builds a coarse 3D mesh from depth + images, then when you click forward it *transitions through the mesh* between two panorama viewpoints, which is why movement feels real ([how Matterport tours work](https://cgi-matter.com/3d-matterport-virtual-tour-for-exploring-a-place-as-if-you-were-there/), [Matterport](https://discover.matterport.com/)). A typical home tour is 30–40 linked scans.

So the app you're describing = **an upload pipeline + a scene graph (nodes & links) + a viewer with camera transitions.** The rest of this doc covers each part.

---

## 1. The four capture-to-3D pipelines your app can support

### A. Single image → 3D object (AI)
Good for: a prop, a facade block, a "quick preview." Not good for: full walkable interiors.

- [TripoSR](https://github.com/VAST-AI-Research/TripoSR) (Stability AI + Tripo, MIT license) — one image → mesh in under a second on server GPU. [Setup guide](https://triposr.org/open-source), [texture-baking fork](https://github.com/iffyloop/TripoSR-Bake)
- [Hunyuan3D-2.1](https://github.com/tencent-hunyuan/hunyuan3d-2.1) (Tencent, open source) — image → high-fidelity mesh **with PBR materials**; the current best open model ([v2](https://github.com/Tencent-Hunyuan/Hunyuan3D-2), [v1](https://github.com/Tencent-Hunyuan/Hunyuan3D-1))
- ⚠️ These need an NVIDIA GPU — run them as a **cloud processing step** (Replicate, Hugging Face Spaces, or a rented GPU), not on your M1.

### B. Many photos → real 3D mesh (photogrammetry)
Good for: building exteriors, rooms, objects — the classic pipeline.

- **On your M1 (free, local):** Apple **Object Capture** via PhotoCatch — see the companion research doc, section 7.
- **Open source, self-hostable (server side of your app):** [COLMAP](https://peterfalkingham.com/2017/05/26/photogrammetry-testing-11-visualsfm-openmvs/) does Structure-from-Motion (figures out where each photo was taken + sparse points), then **OpenMVS** densifies it into a mesh. COLMAP → OpenMVS is the standard free pipeline; [comparison of open-source reconstruction tools](https://www.triposrai.com/posts/open-source-3d-reconstruction-showdown/)
- [Nerfstudio](https://arxiv.org/pdf/2311.01659) — wraps COLMAP for camera poses, then trains NeRF/splat models; has a full CLI, ideal as a backend worker

**Rule of thumb for users of your app:** 60–80% overlap between consecutive photos, every surface seen from ≥3 angles, no zoom changes, diffuse lighting.

### C. Phone video → Gaussian Splat (the 2026 real-estate standard)
Photoreal, smooth free movement, captured with just a smartphone. Zillow ships this (SkyTours), and Apartments.com/Matterport added exterior splats ([industry overview](https://www.utsubo.com/blog/gaussian-splatting-guide)).

- Capture/processing: [Polycam](https://poly.cam/tools/gaussian-splatting) (free tier: 150 images/capture, cloud processing — no local GPU needed), Postshot, or Luma ([how to create splats](https://swyvl.io/blog/how-to-create-gaussian-splats/), [tool comparison](https://www.polyvia3d.com/guides/gaussian-splatting-tools-comparison))
- Web display: Three.js splat renderers like **Spark**, plus native support in Babylon.js 8 ([viewer roundup](https://swyvl.io/blog/best-gaussian-splat-viewers/)); works fine on M1 + in browsers
- This is the highest-payoff skill right now for real-estate 3D ([Gaussian splatting for real estate](https://www.thefuture3d.com/gaussian-splatting/real-estate/))

### D. 360° panoramas → linked virtual tour  ⭐ *your exact use case*
Each uploaded image is a **node**; nodes are connected by **links** (hotspots). Front-of-house node → front-door hotspot → living-room node → kitchen node…

- [Photo Sphere Viewer's VirtualTourPlugin](https://photo-sphere-viewer.js.org/plugins/virtual-tour.html) does literally this: *"a list of nodes which contains a panorama with one or more links to other nodes."* Nodes can be loaded all at once **or asynchronously as the user navigates**. Links are 3D arrows or custom markers; there are Gallery/Map/Plan/Compass plugins for a mini-map of the house ([npm package](https://www.npmjs.com/package/@photo-sphere-viewer/virtual-tour-plugin))
- Positioning links: **manual mode** (you place the arrow) or **GPS mode** (auto from coordinates)
- Alternatives: Pannellum "tour" config, Marzipano (see companion doc, section 6)

---

## 2. What a 3D creation app needs to function (the full anatomy)

```
┌──────────┐   ┌─────────────┐   ┌───────────────┐   ┌────────────┐   ┌─────────┐
│ Uploader │ → │ Validation   │ → │ Processing     │ → │ Asset      │ → │ Viewer  │
│ (browser)│   │ + Storage    │   │ queue/workers  │   │ delivery   │   │ (WebGL) │
└──────────┘   └─────────────┘   └───────────────┘   └────────────┘   └─────────┘
                      ↕                                     ↕
               ┌──────────────────────────────────────────────┐
               │ Database: projects → scenes → nodes → links  │
               └──────────────────────────────────────────────┘
```

### 2.1 Upload & ingestion
- Accept: JPEG/PNG/HEIC (photos), equirectangular JPEG (360s — detect via 2:1 aspect ratio or XMP `GPano` metadata), MP4 (video for splats), GLB/OBJ/FBX (ready-made models)
- Validate: file type, size caps, image resolution; strip EXIF GPS if privacy matters, but **keep** EXIF orientation/camera data (photogrammetry needs focal length)
- Store originals in object storage (S3 / Cloudflare R2 / Supabase Storage); never process in the request cycle

### 2.2 Processing queue & workers (the heart of the app)
Uploads trigger background jobs — this is the standard pattern: file lands in S3 → event triggers a worker (Lambda/Fargate container or a simple Redis queue + worker process) → worker converts → writes results back ([AWS OBJ→glTF pipeline](https://aws.amazon.com/blogs/iot/how-to-convert-and-compress-obj-models-to-glb-gltf-for-use-with-aws-iot-twinmaker/), [open-source example: ifc-pipeline — Flask + Docker Compose + processing queue for viewing building models](https://github.com/AECgeeks/ifc-pipeline)).

Jobs your workers run, depending on upload type:
1. **Image jobs:** resize, generate tiled multi-resolution versions of panoramas (big 360s should stream progressively), thumbnails
2. **Model jobs:** convert anything → glTF (`obj2gltf`), then optimize with `gltf-pipeline`/`gltf-transform`: **Draco** mesh compression + **KTX2** texture compression ([why glTF: Khronos](https://www.khronos.org/gltf/)); generate a USDZ copy if you want iOS AR Quick Look
3. **Reconstruction jobs (GPU):** COLMAP+OpenMVS, Nerfstudio, TripoSR/Hunyuan3D — run on cloud GPU, poll for completion
4. Always: write job status to DB so the UI can show "processing…"

### 2.3 The data model — this is your "layers"
The layering you described is a **graph of scenes**. Minimal schema:

```jsonc
{
  "project": { "id": "villa-23", "title": "Seaside Villa" },
  "nodes": [
    {
      "id": "exterior-front",
      "type": "panorama",             // panorama | model | splat | photo-billboard
      "asset": "s3://…/front-8k.jpg",
      "thumbnail": "s3://…/front-thumb.jpg",
      "position": [0, 1.6, 12],        // optional: real-world placement
      "links": [
        { "to": "living-room", "label": "Enter house",
          "markerPosition": { "yaw": 0.05, "pitch": -0.1 } }   // the front door
      ]
    },
    {
      "id": "living-room",
      "type": "panorama",
      "asset": "s3://…/living-8k.jpg",
      "links": [
        { "to": "exterior-front", "label": "Go outside" },
        { "to": "kitchen", "label": "Kitchen" }
      ]
    }
  ]
}
```

When the user uploads a second image, your editor UI just: creates a node → asks "where does this connect?" → user drops a hotspot on the front door → link saved. That's the whole "add a layer onto it" mechanic. This JSON maps 1:1 onto Photo Sphere Viewer's VirtualTourPlugin config.

### 2.4 The viewer & the outside→inside transition
Three ways to move the visitor from the exterior node into the interior, easiest first:

1. **Click/hotspot transition (do this first).** VirtualTourPlugin handles it: click the arrow on the door → cross-fade + zoom into the next panorama. Zero custom code.
2. **Scroll-driven camera (the "scroll from outside to inside" feel).** Fix the WebGL canvas, make the page tall (e.g. 500vh), and bind scroll progress to a camera path through a real 3D scene with **GSAP ScrollTrigger**: progress 0 = street view, 0.5 = at the front door, 1 = inside the living room. The standard recipe: [Codrops — cinematic 3D scroll experiences with GSAP](https://tympanus.net/codrops/2025/11/19/how-to-build-cinematic-3d-scroll-experiences-with-gsap/), [scroll-driven Three.js presentation](https://medium.com/@pablobandinopla/scroll-driven-presentation-in-threejs-with-gsap-a2be523e430a), [Frontend Horse episode](https://frontend.horse/episode/using-threejs-with-gsap-scrolltrigger/), [Next.js + Three.js + GSAP + Lenis build](https://dev.to/robinzon100/build-an-award-winning-3d-website-with-scroll-based-animations-nextjs-threejs-gsap-3630), [three.js forum thread on camera + ScrollTrigger](https://discourse.threejs.org/t/how-to-move-camera-using-gsap-and-onscroll-triggers/42336). To pass "through" the wall, animate the camera to the door, fade/dissolve at the threshold, and swap the environment (exterior panorama/model → interior panorama) mid-fade — that's the same trick Matterport's mesh transit performs.
3. **Hybrid mesh + panoramas (Matterport-grade, later).** Build a coarse mesh of the house (photogrammetry or splat), place each panorama node at its true position inside it, and animate through the mesh between nodes. This is what makes Matterport transitions feel like walking ([explainer](https://cgi-matter.com/3d-matterport-virtual-tour-for-exploring-a-place-as-if-you-were-there/)).

### 2.5 Asset delivery
- Serve everything through a CDN; lazy-load nodes as the visitor approaches them (VirtualTourPlugin supports async nodes natively)
- Budget: keep any single GLB under ~10MB (Draco+KTX2), panoramas tiled, first paint under 3s on 4G
- Formats per platform: glTF/GLB for web+Android, USDZ for iOS AR

---

## 3. Recommended MVP for you (all free, M1-friendly)

**"Outside→inside house tour builder" — buildable in 2–3 weekends:**

1. **Frontend:** Next.js/Vite + [Photo Sphere Viewer + VirtualTourPlugin](https://photo-sphere-viewer.js.org/plugins/virtual-tour.html)
2. **Capture:** shoot 360s of a real house (phone + Google Street View app, or borrow any 360 cam); one exterior, 2–3 interiors
3. **Upload:** simple form → Supabase Storage (free tier) → DB row per node
4. **Editor:** page that lists a project's nodes, lets you click a point on the panorama to place a link to another node (PSV gives you yaw/pitch from click events)
5. **Viewer:** public page rendering the node graph — visitor starts outside, clicks the door, lands inside
6. **v2:** add the scroll-driven exterior approach with GSAP ScrollTrigger over a Three.js scene; **v3:** accept a Polycam splat as a node type

**What NOT to build yourself at first:** GPU reconstruction (use Polycam/PhotoCatch/Replicate as the "worker" and just ingest their output files). Your app's real value is the upload → node graph → linked viewer, not reimplementing COLMAP.

---

## 4. Checklist: everything a functioning 3D creation app needs

- [ ] Multi-format upload (photos, 360s, video, GLB) with validation & progress UI
- [ ] Object storage for originals + processed derivatives
- [ ] Background job queue with per-job status (queued → processing → ready/failed)
- [ ] Converters: image resize/tiling, model → optimized GLB (Draco/KTX2), thumbnails
- [ ] Optional GPU reconstruction integration (photogrammetry / splats / AI single-image)
- [ ] Database: projects → nodes (type, asset, position) → links (target, marker position, label)
- [ ] Editor UI: create nodes from uploads, place hotspots, set the starting node
- [ ] Viewer: panorama/model/splat rendering, hotspot navigation, smooth transitions, mobile + touch
- [ ] CDN delivery, lazy node loading, file-size budgets
- [ ] Auth + per-user projects; share/embed links (iframe embed is how tours get onto listing sites)
