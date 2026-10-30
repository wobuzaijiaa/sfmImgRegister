# Images Registration Pipeline: SFM Images Register & PortViewer

A lightweight pipeline for registering new photographs against an existing COLMAP reconstruction and browsing them in spatial context through a 3DGS viewer, without rebuilding the scene.

It decouples facility-level spatial reconstruction from ongoing photographic data acquisition, enabling flexible documentation and incremental information updates for applications such as facility inspection, asset management, site surveys, and spatial information retrieval.

![Stage 1](snapshot_s1.png)
![Stage 2](snapshot_s2.png)

## Motivation

Spatial reconstruction and subsequent image acquisition often serve different purposes. Reconstruction prioritizes geometric coverage and visual overlap, while new photographs may focus on specific objects, details, or conditions without meeting those requirements.

This pipeline reuses an existing COLMAP reconstruction to spatially register new photographs independently of scene reconstruction. Images that cannot be registered directly, such as close-ups of assets or defects, can remain accessible through approximate **session-proxy associations**, provided at least one image in the same session is successfully registered.

This allows spatial context to be reused across different acquisition workflows and updated incrementally without routinely rebuilding the 3DGS scene.

## Architecture

| Component | Function |
|---|---|
| **SFM Images Register (Stage 1)** | Registers images using COLMAP and exports camera poses, session metadata, and images |
| **PortViewer (Stage 2)** | Visualizes registered and session-proxy images using SuperSplat Viewer and a top-down snapshot |

### Stage 1 Output

~~~text
imgRegister_<timestamp>/
├── cameras.json                 # Optional combined registered poses
├── supersplat_manifest.json     # Session/image metadata
├── supersplat_settings.json     # Viewer annotations
└── sessions/
    └── <session_name>/
        ├── cameras.json         # Registered camera poses
        ├── report.json
        ├── report.txt
        ├── target.json          # When applicable
        └── images/              # Registered and failed images
~~~

Camera positions use COLMAP world coordinates, with 3×3 Camera-to-World (C2W) rotation matrices.

## Features

- **Image registration:** Registers new images into an existing COLMAP sparse reconstruction, with camera configuration adapted to capture devices and image characteristics.
- **Session-based organization:** Preserves image associations and registration results across capture sessions.
- **Session-proxy visualization:** Assigns approximate locations to unregistered images using registered cameras in the same session.
- **3D visualization:** Integrates SuperSplat Viewer for camera annotations and session-based navigation.
- **Top-down overlay:** Projects camera positions onto an orthographic snapshot with synchronized image/session highlighting.
- **Visual distinction:** Green for registered images, amber for session proxies, and dark red for images without poses.
- **Local operation:** Supports offline use with locally available dependencies and Viewer/Editor assets.

## Requirements

- COLMAP installed locally (Stage 1).
- Compatible 3DGS model (`.ply`, `.splat`, or `.sog`).
- Chrome or Edge with the rendering capabilities required by the Viewer build.
- Local SuperSplat Viewer and customized Editor assets (Stage 2).

## How to Use

### Stage 1 — Register Images

Run **SFM Images Register** (`sfm_images_register_v0_1_0.exe`).

1. Select the existing COLMAP sparse model and `database.db`.
2. Select the new images directory (subfolders represent sessions).
3. Configure COLMAP and the output directory.
4. Click **Register Images** to generate the registration results.

### Stage 2 — Visualize Images

Run **PortViewer** (`portviewer_v0_1_0.exe`).

1. Start the customized SuperSplat Editor and load the 3DGS model.
2. Establish an orthographic top-down view and export `topdown.png` and `topdown.json` using **Render → Top-Down Snapshot**.
3. Select the Stage 1 output folder, 3DGS model, and snapshot files in PortViewer.
4. Click **Start Viewer** to browse sessions, images, and spatial annotations.
5. Select a session to highlight its top-down markers and navigate its available 3D annotations.

## Scope and Limitations

- Successful registration requires sufficient overlap with the existing COLMAP reconstruction.
- Session-proxy poses are **approximate associations**, not recovered camera poses or verified object locations. Sessions without registered cameras cannot receive proxies.
- Proxy markers use small, fixed world-space display offsets to avoid overlap; these do not alter registration data.
- The Viewer supports up to 25 annotations. Images without individual 3D annotations remain accessible through the session browser and, when a pose is available, the top-down overlay.
- Spatial accuracy depends on the existing reconstruction's coordinate system and scale. The pipeline does not update 3DGS geometry, identify objects, or perform image-content analysis.
- Stage 2 proxy poses are derived at runtime and are not persisted in Stage 1 outputs.
