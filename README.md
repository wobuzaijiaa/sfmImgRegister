# Images Registration Pipeline: SFM Images Register & PortViewer

A lightweight, two-stage pipeline to register newly captured images into an existing COLMAP sparse model and interactively visualize the resulting camera poses.

![Screenshot Stage 1 window](snapshot_s1.png)
![Screenshot Stage 2 window](snapshot_s2.png)

## Motivation & Core Principle
Sparse models and downstream 3D Gaussian Splatting (3DGS) models are typically static. If you capture new photos of an environment, they lack camera poses and cannot be placed into the existing spatial context.

This repository bridges COLMAP and spatial visualization. Because the downstream 3DGS model is trained directly from the COLMAP sparse model, they share an identical 3D coordinate system. By registering new images to the sparse model, the calculated 6-DoF poses are mathematically guaranteed to align within the 3DGS scene.

The pipeline provides two standalone software packages to register new images, export their poses, and navigate them via a custom 2D/3D web interface.

## Architecture & Dataflow

The pipeline is cleanly divided into two standalone software packages:

| Component | Role | Input | Output |
| :--- | :--- | :--- | :--- |
| **SFM Images Register**<br>(Stage 1 Package) | Core Registration | Existing Sparse model, DB, new images | `cameras.json`, `supersplat_manifest.json`, Session folders |
| **PortViewer**<br>(Stage 2 Package) | Visualization Workspace | Stage 1 Output, 3DGS `.ply` model | Local HTTP Servers, Interactive 2D Overlay, `topdown.png/json` |

## Standard Output Format (Stage 1)
The **SFM Images Register** outputs a timestamped folder (e.g., `imgRegister_20261012_120000/`). This folder serves as the direct input for PortViewer and contains:
~~~text
imgRegister_20261012_120000/
├── cameras.json               # 3DGS/COLMAP camera poses for new images
├── supersplat_manifest.json   # Web UI sidebar data
├── supersplat_settings.json   # 3D annotations for the viewer
└── sessions/                  # Grouped by capture session
    └── <session_name>/
        ├── cameras.json
        └── images/            # Thumbnails for the Web UI
~~~

## Features
* **Intelligent Auto-Calibration (Stage 1):** Analyzes image resolutions and EXIF data to automatically group mixed devices and adjust COLMAP matching thresholds adaptively.
* **Session Management (Stage 1):** Groups images by subfolders, exports individual/session-wide camera poses, and builds a hierarchical web manifest.
* **Built-in Editor & Viewer (Stage 2):** PortViewer bundles a customized SuperSplat Editor (to generate top-down snapshots) and a SuperSplat Viewer (for 3D exploration with synchronized sidebars).
* **Top-Down 2D Overlay (Stage 2):** Reprojects 3D camera poses into a 2D orthographic snapshot. Clicking an image in the web UI instantly highlights its corresponding marker (red dot) on the 2D map.
* **Coordinate Transformations (Stage 2):** Built-in support for multiple axis conventions (OpenGL/PlayCanvas, COLMAP/OpenCV, WebXR, Blender) to ensure perfect alignment.
* **Portable & Standalone:** Both tools are provided as standalone Windows executables (`.exe`). No Node.js, Python, or CUDA installation is required at runtime.

## Requirements (End-Users)
* **COLMAP** (Must be downloaded separately and accessible locally. Required by Stage 1).
* A modern web browser (Chrome or Edge) with WebGPU support (Required by Stage 2).

## How to Use

### Stage 1: Register New Images
Run the **SFM Images Register** executable (e.g., `sfm_images_register_v0_1_0.exe`).
1. Set the **Sparse Model** (`sparse/0` directory), **Database.db**, and the **Base Output dir**.
2. Set the **New Images dir** (can contain subfolders representing different capture sessions).
3. Locate your local **COLMAP** executable (e.g., `colmap.bat`).
4. Click **Register Images**. The tool will output the standard registration folder.

### Stage 2: Visualize with PortViewer
Run the **PortViewer** executable (e.g., `portviewer_v0_1_0.exe`).

**Step A: Generate the Top-Down Snapshot**
1. Click **▶ Start Editor** in PortViewer to open the integrated SuperSplat Editor.
2. Load your pre-trained 3DGS `.ply` model.
3. Adjust the camera to a strict orthographic top-down view.
4. Click **Render -> Top-Down Snapshot** to export `topdown.png` and `topdown.json`.

**Step B: Interactive Visualization**
1. In the PortViewer GUI, select the **Output Folder** generated in Stage 1.
2. Select your trained **3DGS Model** (`.ply`, `.splat`, `.sog`).
3. Select the **Snapshot PNG** and **Metadata JSON** generated in Step A.
4. Click **▶ Start Viewer**. 
5. Use the web interface to click through images and verify their 3D positions on the 2D top-down map.
