# Desert Sandworm System — Complete User Guide & Technical Encyclopedia

**Version:** 1.0.0 | **Author:** LaurindoGames | **Engine Compatibility:** Unreal Engine 5.8 (Win64)  
**Plugin Content Mount Point:** `/DesertSandwormSystem/`

Welcome to the official documentation for the **Desert Sandworm System**. Whether you are a complete beginner opening Unreal Engine for the first time or an experienced technical artist looking to customize GPU simulation shaders, this guide walks you through every asset, slider, material parameter, and C++/Blueprint function included in the plugin.

---

## Table of Contents

1. [Overview & Key Features](#1-overview--key-features)
2. [Quick Start Guide (Zero Code — 2 Minutes)](#2-quick-start-guide-zero-code--2-minutes)
3. [Complete Materials & Shaders Encyclopedia (All 11 Materials)](#3-complete-materials--shaders-encyclopedia-all-11-materials)
4. [GPU Sand Deformation System (`ASandwormSandDeformer`)](#4-gpu-sand-deformation-system-asandwormsanddeformer)
5. [Autonomous AI & Combat (`USandwormCombatAIComponent`)](#5-autonomous-ai--combat-usandwormcombataicomponent)
6. [Audio & MetaSounds (`USandwormSandInteractionComponent`)](#6-audio--metasounds-usandwormsandinteractioncomponent)
7. [Showcase Director (`ASandwormShowcaseDirector`)](#7-showcase-director-asandwormshowcasedirector)
8. [Troubleshooting & FAQ](#8-troubleshooting--faq)

---

## 1. Overview & Key Features

The **Desert Sandworm System** is a complete, plug-and-play colossal creature and real-time GPU granular sand physics framework built natively in C++ and HLSL for Unreal Engine 5.8. It contains zero external gameplay dependencies and drops cleanly into any new or existing project.

### Core Pillars

* **136-Meter Colossal Creature (`SK_DesertSandworm`):** A rigged and textured leviathan equipped with a physics asset (`SK_DesertSandworm_PhysicsAsset`), 13 animation assets (11 primary combat and locomotion sequences plus extra locomotion variants), and a custom thread-safe C++ animation blending system (`USandwormBlendAnimInstance`) that eliminates hard pose cuts and adds natural spine wave inertia (`SpineWaveLagFraction = 0.38`) without requiring manual AnimGraph state machines.
* **Real-Time 1024x1024 GPU Ping-Pong Sand Physics (`ASandwormSandDeformer`):** A persistent `320 m x 320 m` (`32000 cm` extent, `~31.25 cm/texel`) sliding deformation window backed by two 16-bit floating-point Render Targets (`RT_SandPing` and `RT_SandPong`, `RTF_RGBA16F`). It carves deep trenches (`R`), raises displaced lateral sand berms (`G`), opens dark emergence tunnels (`B`), and runs a 30 Hz cellular automaton gravity solver (`M_SandPhysics_CollapseStep`) that simulates the natural angle of repose (`33°`) so steep crater walls avalanche and self-seal after the creature dives.
* **Plug-and-Play Dynamic Object Sand Interaction (`USandDeformInteractorComponent`):** Attach this lightweight component to any Player Character, Vehicle, Physics Cube, or Falling Rock—or let `ASandwormSandDeformer`'s automatic scanner (`bAutoDeformDynamicObjects = true`) detect moving primitives automatically. Objects only carve sand when physically touching the ground (`ContactToleranceCm = 4.0 cm`), scaling trench depth and berm rim width (`DynamicObjectBermWidthScale = 0.40`) proportionally to object size and downward impact speed.
* **Autonomous Free-Roam & Combat AI (`USandwormCombatAIComponent`):** A self-contained C++ state machine supporting organic desert exploration (`AutonomousSlither`, `AutonomousBurrowTravel`, `AutonomousSurveyOrRoar`) inside automatic safe Landscape boundaries (`RoamMaxRadiusCm = 45000 cm`, `LandscapeEdgeMarginCm = 25000 cm`), plus reactive combat perception via ground vibration (`VibrationSenseRadiusCm = 6500 cm`), line-of-sight (`VisionRangeCm = 9500 cm`), and Unreal's native damage system (`OnTakeAnyDamage`).
* **Reactive 27-Bone Spine Interaction & 4 Primary Niagara VFX (`USandwormSandInteractionComponent`):** Tracks 27 spine and jaw bones in real time against the terrain height to distinguish underground travel (`SubsurfaceBulge`) from surface carving (`CarveAndBerm`), dynamically drives sand dust accumulation on the worm's armor (`SandCoverage`), triggers screen-space sonic shockwaves during roars (`M_PP_RoarSonicWave`), and spawns 4 primary optimized Niagara VFX systems (`NS_Sand_EruptionStateless`, `NS_Sand_CollapseStateless`, `NS_Sand_ImpactStateless`, `NS_Sand_SlitherTrail`) alongside world-space wrapped ambient dune wind (`NS_Desert_GroundDriftStateless`).
* **12 Bespoke 48 kHz / 16-bit PCM MetaSounds & Colossal Spatial Attenuation:** Full soundscape (`MS_Sandworm_*`) with exposed real-time DSP inputs (`PitchShiftSemitones`, `CutoffFrequencyHz`, `GainScale`) paired with a 280-meter logarithmic distance attenuation asset (`ATT_Sandworm_Colossal`) and 9 frame-accurate animation delay sliders.
* **3-Block One-Take Showcase Director (`ASandwormShowcaseDirector`):** Press Play (`Alt + P`) in the demo map to experience an automated 5-act Boss Cinematic (Block 1), an 11-clip orbiting Animation Gallery (Block 2), and an infinite Autonomous AI Drone Safari (Block 3) linked by 3-step aerial camera transitions.

### System Requirements & Real-World Performance Benchmarks

| Specification | Detail |
| :--- | :--- |
| **Unreal Engine Version** | **Unreal Engine 5.8.0+** |
| **Supported Target Platforms** | **Win64** (Editor, Development, Shipping) |
| **Required Engine Plugins** | `Niagara`, `Metasound` (Enabled automatically by `DesertSandwormSystem.uplugin`) |
| **Measured Overall Frame Rate** | **`70–95+ FPS`** in gameplay / **`63.5–81.0+ FPS`** across full 3-block cinematic trailer at `1024x1024` RT resolution |
| **CPU Game Thread Overhead** | **`~0.27 ms`** (Strict tick isolation disables dormant creatures; zero AnimBlueprint VM overhead) |
| **GPU Sand Solver Cost** | **Fixed `30 Hz` simulation step** with automatic sleep after `25.0s` of inactivity (`0.00 ms` GPU cost when idle) |

---

## 2. Quick Start Guide (Zero Code — 2 Minutes)

Follow these four simple steps to explore the showcase map or drop the Sandworm and deformable sand into your own level without writing a single line of code.

### Step 1: Enable the Plugin & Make Plugin Content Visible
1. Open your Unreal Engine 5.8 project and go to **Edit -> Plugins**.
2. Search for **Desert Sandworm System** (under the *Gameplay* category) and ensure the **Enabled** checkbox is ticked. Restart the Editor if prompted.
3. Open the **Content Browser** (`Ctrl + Space`). Click the **Settings** gear icon in the top-right corner of the Content Browser and check **Show Plugin Content**.
4. In the left folder tree, scroll down to **Plugins -> Desert Sandworm System Content** (`/DesertSandwormSystem/`).

### Step 2: Play the Included Showcase Map
1. Navigate to `/DesertSandwormSystem/Maps/` and double-click **`L_Sandworm_DesertShowcase`**.
2. Press **Play** (`Alt + P`) or **Simulate** (`Alt + S`).
3. The `ASandwormShowcaseDirector` actor in the level will automatically launch in **`FullOneTakeTrailer`** mode, displaying live performance statistics (`bShowPerfStats = true`) in the top-right corner while guiding the camera through the Boss Arena, Animation Gallery, and Free-Roam AI Desert.

### Step 3: Add the Sandworm & GPU Sand Physics to Your Own Level
If you are starting in your own custom map with a Landscape:
1. **Assign the Deformable Sand Material:** Select your **Landscape** actor in the *Outliner*. In the *Details* panel under **Landscape Material**, assign `/DesertSandwormSystem/Materials/MI_DesertSand_Dynamic`.
2. **(Optional) Add the High-Density Follower Patch:** For ultra-crisp vertex displacement directly around the worm even on low-density Landscapes, drag `/DesertSandwormSystem/Terrain/SM_SandWormFollowerPatch` into your level, assign `/DesertSandwormSystem/Materials/MI_DesertSand_FollowerPatch` to its material slot, and set its Scale to `(1, 1, 1)`.
3. **Place the GPU Sand Deformer:** Open the **Place Actors** panel, search for **`SandwormSandDeformer`**, and drag it anywhere into your level.
   * By default, it automatically finds your Landscape, `RT_SandPing`, `RT_SandPong`, `MPC_SandDeformation`, `M_SandPhysics_Stamp`, and `M_SandPhysics_CollapseStep`.
   * If you added the optional `SM_SandWormFollowerPatch`, pick it in the **`FollowerPatchActor`** slot under `Sand Physics | Targets`.
4. **Spawn a Sandworm Actor:**
   * Drag `/DesertSandwormSystem/Creature/Meshes/SK_DesertSandworm` into your level (this creates a `SkeletalMeshActor`).
   * With the new Sandworm actor selected, click **+ Add** in the *Details* panel and add **`SandwormSandInteractionComponent`** (handles bone-to-sand carving, Niagara VFX, and MetaSounds).
   * Click **+ Add** again and add **`SandwormCombatAIComponent`** (handles movement, burrowing, and combat).
   * Select the **`SandwormCombatAIComponent`** in the component list and check **`bEnableAutonomousAI = true`** under `Sandworm | AI`.
   * Select your **`SandwormSandDeformer`** actor in the level and set **`TargetSandwormActor`** to your newly placed Sandworm. Press **Play**—your 136m Sandworm will immediately slither, burrow, and carve the desert!

### Step 4: Make Any Object (Player, Vehicle, Physics Prop) Carve the Sand
There are **two effortless ways** to make characters, vehicles, boulders, or physics crates leave realistic footprints and trenches in the sand:

* **Method A — Automatic Area Scanner (Zero Setup):**  
  `ASandwormSandDeformer` has **`bAutoDeformDynamicObjects = true`** enabled by default. Any movable object with collision or physics moving faster than `10 cm/s` (`DynamicObjectMinSpeedCmPerSec`) within `140 meters` (`DynamicObjectScanRadiusCm = 14000.0`) of the active sand window will automatically carve proportional tracks whenever its bottom touches within `4 cm` of the ground!
* **Method B — Dedicated Component for Hero Actors (`USandDeformInteractorComponent`):**  
  If you want fine-tuned control over a specific Player Character, Vehicle, or Hero Prop (including pulling the deformation window to distant dunes when the player walks far away from the Sandworm):
  1. Select your Player Character, Vehicle, or StaticMesh Actor.
  2. In the *Details* panel, click **+ Add** and search for **`SandDeformInteractor`** (`USandDeformInteractorComponent`).
  3. Adjust **`DepthMultiplier`** (default `1.0`) or **`BermMultiplier`** (default `1.0`) to make heavy vehicles sink deeper or light characters leave subtle prints. That's it!

---

## 3. Complete Materials & Shaders Encyclopedia (All 11 Materials)

Every shader in the **Desert Sandworm System** is authored for DirectX 12 Shader Model 6 (`PCD3D_SM6`) with strict instruction budgets and zero redundant texture lookups. Below is the complete beginner-friendly reference for all **11 Master Materials** (and their ready-to-use Material Instances).

### Quick Summary Table of All 11 Materials

| # | Material Name | Content Path | Domain / Blend / Shading | GPU Cost (VS / PS / Samplers) | Primary Purpose |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | `M_DesertSand_Dynamic` | `/DesertSandwormSystem/Materials/` | Surface / Opaque / Default Lit | **254 VS / 498 PS / 3 Samplers** | Main deformable desert terrain & follower patch shader (WPO + Analytical Normals + Micro-Glints). |
| **2** | `M_SandPhysics_Stamp` | `/DesertSandwormSystem/Materials/` | Surface / Opaque / Unlit (RT Solver) | **153 VS / 808 PS / 1 Sampler** | GPU Ping-Pong brush solver that stamps up to 8 bones/objects per draw call with UV window shifting. |
| **3** | `M_SandPhysics_CollapseStep` | `/DesertSandwormSystem/Materials/` | Surface / Opaque / Unlit (RT Solver) | **148 VS / 612 PS / 1 Sampler** | 30 Hz GPU cellular automaton gravity solver simulating sand avalanches (`33°` angle of repose) & burrow sealing. |
| **4** | `M_DesertSandworm` | `/DesertSandwormSystem/Creature/Materials/` | Surface / Opaque / Default Lit | **168 VS / 214 PS / 3 Samplers** | Creature PBR master shader for chitin armor, shells, and teeth with dynamic world-normal sand accumulation. |
| **5** | `M_PP_RoarSonicWave` | `/DesertSandwormSystem/Materials/` | Post Process / Opaque / Unlit | **148 VS / 91 PS / 1 Sampler** | Screen-space radial acoustic refraction & chromatic aberration shockwave triggered by the worm's roar. |
| **6** | `M_Sandstone_Outcrop` | `/DesertSandwormSystem/Materials/` | Surface / Opaque / Default Lit (Nanite) | **148 VS / 196 PS / 0 Samplers** | 100% procedural triplanar sedimentary desert rock shader (zero texture memory footprint). |
| **7** | `M_SandVFX_DustCloud` | `/DesertSandwormSystem/VFX/Materials/` | Surface / Translucent / Unlit | **312 VS / 238 PS / 1 Sampler** | Hybrid volumetric dust plume & sand clod sprite shader driven by vertex World Position Offset (WPO). |
| **8** | `M_SandVFX_GroundSandDrift` | `/DesertSandwormSystem/VFX/Materials/` | Surface / Translucent / Unlit | **445 VS / 185 PS / 2 Samplers** | Stationary world-space tile-wrapping ground wind veil that hugs dune heightmaps without camera dragging. |
| **9** | `M_SandVFX_SandClump` | `/DesertSandwormSystem/VFX/Materials/` | Surface / Opaque / Default Lit | **148 VS / 142 PS / 0 Samplers** | Ultra-fast opaque PBR shader for solid 3D sand clod mesh particles ejected during eruptions. |
| **10** | `M_SandVFX_SandSpray` | `/DesertSandwormSystem/VFX/Materials/` | Surface / Translucent / Unlit | **312 VS / 238 PS / 1 Sampler** | High-velocity ballistic sand jet & granular spray shader with directional solar backlighting. |
| **11** | `M_SandVFX_WindSheet` | `/DesertSandwormSystem/VFX/Materials/` | Surface / Translucent / Default Lit | **298 VS / 215 PS / 0 Samplers** | Lit volumetric directional wind sheet and ground-hugging dust curtain shader. |

---

### 3.1 `M_DesertSand_Dynamic` (Deformable Desert Terrain Shader)
* **Location:** `/DesertSandwormSystem/Materials/M_DesertSand_Dynamic`
* **Material Instances:** `MI_DesertSand_Dynamic` (assigned to the Landscape) and `MI_DesertSand_FollowerPatch` (assigned to `SM_SandWormFollowerPatch`).
* **What It Does (In Simple Terms):**  
  This is the heart of the visual sand surface. In the Vertex Shader (`Custom_SandWPO`), it reads the live deformation texture (`RT_SandDeformation`) and pushes vertices down where trenches are carved (`R` channel), pushes vertices up where sand berms or underground bulges are formed (`G` channel), and smoothly tucks the edges of the `SM_SandWormFollowerPatch` mesh underneath the Landscape so you never see a seam. In the Pixel Shader (`Custom_SandNormalWS`, `Custom_SandAlbedo`, `Custom_SandRoughSpec`), it computes crisp 3D shadows along trench walls, erases natural wind ripples wherever the sand has been churned, darkens deep wormholes, and adds glittering quartz micro-sparkles in the sun.
* **Key Editable Parameters & Visual Effects:**
  * `RT_SandDeformation` (*Texture*): Points to `RT_SandPing`. Reads the live GPU height and disturbance channels.
  * `DesertHeightmapTex` (*Texture*): Points to `T_DesertHeightmap_RG8`. Allows the follower patch mesh to match the exact elevation of the underlying Landscape.
  * `bIsFollowerPatch` (*Scalar `0.0` or `1.0`*): Set to `0.0` on the Landscape (`MI_DesertSand_Dynamic`) and `1.0` on the follower mesh (`MI_DesertSand_FollowerPatch`). When `1.0`, the shader snaps the mesh vertices to the heightmap and gently dips the outer border under the Landscape.
  * **Global Parameters (Read automatically from `MPC_SandDeformation`):**
    * `MaxCarveDepthCm` (Default `620.0`): How deep (in centimeters) a full red channel value (`R = 1.0`) sinks the sand vertices downward.
    * `MaxBermHeightCm` (Default `340.0`): How high (in centimeters) a full green channel value (`G = 1.0`) raises displaced sand ridges upward.
    * `DeformNormalStrength` (Default `1.35`): Controls how sharp and steep the lighting normals appear along carved trench walls and footprints (includes an automatic `smallImprintBoost` so smaller footprints remain crisp).
    * `BurrowDarkenStrength` (Default `0.94`): How dark the interior of an open emergence/submerge wormhole looks (simulating deep underground shadow).
    * `SandEdgeFadeFraction` (Default `0.12`): Smoothly fades deformation to zero across the outer 12% border of the `320 m x 320 m` window so there is never a hard square edge.
* **GPU Cost:** **254 VS instructions / 498 PS instructions / 3 Samplers** (1 Virtual/Linear Render Target sampler + 1 Heightmap sampler + shared procedural noise math).

---

### 3.2 `M_SandPhysics_Stamp` (GPU Multi-Stamp Brush Solver)
* **Location:** `/DesertSandwormSystem/Materials/M_SandPhysics_Stamp`
* **Where It Is Used:** Assigned to `StampMaterialAsset` on `ASandwormSandDeformer` and executed on the GPU via `DrawMaterialToRenderTarget`.
* **What It Does (In Simple Terms):**  
  Think of this material as a high-speed digital paintbrush that paints onto the sand memory texture. To maximize performance on mid-range GPUs, it processes up to **8 separate stamps in a single draw call** (`StampPosRadius0..7` and `StampStrength0..7`). Whenever the deformation window moves across the desert, this shader also shifts the existing sand pixels (`UVShift`) so older tracks stay locked to their exact world coordinates without ever wiping the texture.
* **Key Parameters (Driven automatically by `ASandwormSandDeformer`):**
  * `PrevSandRT` (*Texture*): The previous frame's sand state (`RT_SandPing` or `RT_SandPong`).
  * `UVShift` (*Vector*): `(DeltaU, DeltaV, TexelSizeUV, 0)` — shifts the stored heightfield when the sliding window moves by whole texel increments.
  * `WindowCenterWS` (*Vector*): `(CenterX, CenterY, DeformExtentCm, 0)`.
  * `StampPosRadius0` .. `StampPosRadius7` (*Vector*): `(U, V, RadiusUV, StampMode)`.
    * `StampMode = 0` (`CarveAndBerm`): Carves a central depression in `R` and pushes up a surrounding rim in `G`.
    * `StampMode = 1` (`SubsurfaceBulge`): Raises a smooth underground pressure wave in `G` without carving `R`.
    * `StampMode = 2` (`BurrowHole`): Carves a deep pit in `R`, raises a massive collar rim in `G`, and writes the dark tunnel mask in `B`.
    * `StampMode = 3` (`ImpactCrater`): Carves a wide radial shockwave crater from body slams.
  * `StampStrength0` .. `StampStrength7` (*Vector*): `(CarveStrength, BermStrength, BurrowStrength, BermWidthScale / BulgeStrength)`. When `StampMode == 0`, the `W` channel controls `BermWidthScale` (`0.0` for wide Sandworm ridges; `0.40` for tight rims around dynamic physics cubes and feet).
* **GPU Cost:** **153 VS instructions / 808 PS instructions / 1 Sampler** (Only runs when active stamps are queued).

---

### 3.3 `M_SandPhysics_CollapseStep` (30 Hz GPU Granular Gravity & Avalanche Solver)
* **Location:** `/DesertSandwormSystem/Materials/M_SandPhysics_CollapseStep`
* **Where It Is Used:** Assigned to `CollapseStepMaterialAsset` on `ASandwormSandDeformer`.
* **What It Does (In Simple Terms):**  
  Real desert sand cannot hold vertical cliffs—when a trench wall is steeper than the sand's **Angle of Repose** (`~33°`), gravity pulls the upper grains down until the slope stabilizes. This material inspects each sand pixel and its 8 neighbors at a fixed `30 Hz` rate. When a slope exceeds the threshold, it transfers sand mass from the raised berm (`G`) into the carved trench (`R`). Once a slope settles below the angle of repose, the flux drops to zero (`0.0`), preventing precision drift so footprints and stabilized craters remain permanently etched in the desert!
* **Key Parameters (Driven automatically via `ASandwormSandDeformer` sliders):**
  * `CurrentSandRT` (*Texture*): The latest stamped sand Render Target.
  * `PhysicsParams0` (*Vector*): `(TexelUV, StepCm, StepDeltaTime, Tan(AngleOfReposeDeg))`.
  * `PhysicsParams1` (*Vector*): `(MaxCarveDepthCm, MaxBermHeightCm, AvalancheDiffusionRate, BurrowClosureRate)`.
  * `PhysicsParams2` (*Vector*): `(AmbientInfillRate, BulgeSettleRate, SurfaceSmoothStrength, DisturbedMaskDecayRate)`.
  * `KeepOpenUV` (*Vector*): `(BurrowU, BurrowV, RadiusUV, KeepOpenStrength)` — holds an emergence or dive hole open while the 136m body is actively passing through the ground, then releases it so gravity collapses the crater behind the tail.
* **GPU Cost:** **148 VS instructions / 612 PS instructions / 1 Sampler** (Runs at a fixed `30 Hz` and sleeps completely at `0 Hz` when no movement has occurred for `CollapseIdleSleepSeconds = 25.0s`).

---

### 3.4 `M_DesertSandworm` (Creature Armor, Shells & Teeth PBR Master Material)
* **Location:** `/DesertSandwormSystem/Creature/Materials/M_DesertSandworm`
* **Material Instances Applied to `SK_DesertSandworm`:**
  * Slot 0: `MI_Sandworm_HeadAndArmor` (`T_Sandworm_Armor_BaseColor`, `_Normal`, `_ORM`)
  * Slot 1: `MI_Sandworm_ArmorShells` (`T_Sandworm_Shells_BaseColor`, `_Normal`, `_ORM`)
  * Slot 2: `MI_Sandworm_TeethAndSpines` (`T_Sandworm_Teeth_BaseColor`, `_Normal`, `_ORM`)
* **What It Does (In Simple Terms):**  
  Renders the Sandworm's chitinous plates, dorsal shells, and inner teeth using full Packed ORM (Ambient Occlusion in `R`, Roughness in `G`, Metallic in `B`) textures, overlaid with a dynamic **world-space sand layer**. When the worm bursts out of the ground, its body is coated in sand (`SandCoverage = 1.0`). As it stays above ground, sand gradually sheds off (`SandShedRatePerSec = 0.20`), remaining longest in upward-facing crevices and scale pockets.
* **Key Editable Parameters & Visual Effects:**
  * `BaseColorTex`, `NormalTex`, `ORMTex` (*Textures*): The 3 PBR texture maps for each body section.
  * `SandCoverage` (*Scalar, `0.0` to `1.0`*): Dynamically updated every frame by `USandwormSandInteractionComponent`. Increasing it blankets upward-facing armor normals in fine desert sand and softens specular highlights.
  * `SandTintColor` (*Vector Color*): The warm ochre color of the sand grains clinging to the creature's scales.
  * `SandRoughness` (*Scalar, default `~0.88`*): Makes sand-covered areas matte and dusty compared to the glossier exposed chitin and teeth.
* **GPU Cost:** **168 VS instructions / 214 PS instructions / 3 Samplers** (`BaseColor`, `Normal`, `ORM`).

---

### 3.5 `M_PP_RoarSonicWave` (Post-Process Roar Acoustic Shockwave)
* **Location:** `/DesertSandwormSystem/Materials/M_PP_RoarSonicWave`
* **Where It Is Used:** Assigned to the **PostProcessVolume** (under *Rendering Features -> Post Process Materials*) in `L_Sandworm_DesertShowcase`.
* **What It Does (In Simple Terms):**  
  When the Sandworm opens its jaws and roars, `USandwormSandInteractionComponent` projects the 3D world position of the worm's mouth onto your 2D screen (`RoarScreenCenterUV`) and drives `MPC_SandwormFX`. This post-process shader generates concentric acoustic pressure rings expanding outward from the worm's mouth, refracting the air and splitting RGB channels (chromatic aberration) at the edges of the screen while keeping the center of the worm's face crystal clear (`centralMask`).
* **Key Parameters (Read from `MPC_SandwormFX`):**
  * `RoarDistortionIntensity` (*Scalar `0.0` to `1.0`*): Overall strength of the screen refraction and RGB fringe. When `0.0`, the shader exits immediately with zero distortion.
  * `RoarWavePhase` (*Scalar*): Drives the outward radial propagation speed of the concentric acoustic rings.
  * `RoarScreenCenterUV` (*Vector*): `(ScreenU, ScreenV, FacingMask, DistanceAtten)` — the 2D screen coordinates of the worm's mouth.
  * `RoarFacingMask` (*Scalar*): Automatically fades out the shockwave if the camera turns away from the worm.
  * `RoarDistanceAtten` (*Scalar*): Scales the distortion down smoothly when the camera is far away.
* **GPU Cost:** **148 VS instructions / 91 PS instructions / 1 Sampler** (`SceneTexture:PostProcessInput0`).

---

### 3.6 `M_Sandstone_Outcrop` (Procedural Nanite Sedimentary Rock Shader)
* **Location:** `/DesertSandwormSystem/Materials/M_Sandstone_Outcrop`
* **Where It Is Used:** Applied to desert rock outcrops and cliff formations (`bUsedWithNanite = true`).
* **What It Does (In Simple Terms):**  
  Generates layered desert sandstone cliffs and wind-eroded rock formations using 100% mathematical world-space strata (`WorldPos`) and vertex normals (`BaseNormalWS`). Because it uses procedural math instead of bitmap textures, you can scale rocks to any colossal size without blurry pixels or UV tiling seams, and it consumes **0 bytes of VRAM texture memory**.
* **Visual Behavior:**  
  Automatically blends dark iron-rich sedimentary bands (`colDark`), warm sunlit sandstone strata (`colWarm`), and settled desert dust (`colDust`) on top-facing rock ledges.
* **GPU Cost:** **148 VS instructions / 196 PS instructions / 0 Samplers**.

---

### 3.7 `M_SandVFX_DustCloud` (Hybrid Volumetric Dust & Sand Clod Sprite Shader)
* **Location:** `/DesertSandwormSystem/VFX/Materials/M_SandVFX_DustCloud`
* **Material Instances:**
  * `MI_SandVFX_EruptionDust` & `MI_SandVFX_EruptionSpray` (used by `NS_Sand_EruptionStateless`)
  * `MI_SandVFX_CollapseDust` & `MI_SandVFX_CollapseSpray` (used by `NS_Sand_CollapseStateless`)
  * `MI_SandVFX_ShockwaveDust` & `MI_SandVFX_ShockwaveSpray` (used by `NS_Sand_ImpactStateless`)
  * `MI_SandVFX_SlitherTrail` (used by `NS_Sand_SlitherTrail`)
* **What It Does (In Simple Terms):**  
  In Unreal Engine 5.8, `UNiagaraStatelessEmitter` achieves ultra-low CPU overhead by running fixed kinematic particle modules. To get complex, organic eruption plumes, ground-hugging dust skirts (which hide the intersection seam where the 136m body enters the ground), and ballistic gravity arcs, this material calculates particle ballistic expansion directly in the Vertex Shader (`Custom_HybridSandWPO`). In the Pixel Shader (`Custom_HybridSandSurface`), it samples fractional Brownian motion noise (`T_SandNoise_FBM_512`) to render both soft billowing dust clouds and dense, granular sand clods in a single pass, complete with directional sunlight self-shadowing (`SunDirectionWS` from `MPC_SandDeformation`).
* **Key Editable Parameters & Visual Effects:**
  * `NoiseTex` (*Texture*): Points to `/DesertSandwormSystem/VFX/Textures/T_SandNoise_FBM_512`.
  * `SandTint` (*Vector Color*): Primary sunlit color of the dust plume.
  * `ClodTint` (*Vector Color*): Deeper, shadowed ochre tone for dense interior sand clumps.
  * `ClodMix` (*Scalar `0.0` to `1.0`*): Blends between soft misty dust (`0.0`) and sharp, heavy granular sand clods (`1.0`).
  * `OpacityScale` (*Scalar*): Overall transparency multiplier of the cloud.
  * `DesiredRadiusCm` (*Scalar*): Target world-space radius of the eruption ring or dust curtain.
  * `RadialPushCm` (*Scalar*): How far outward particles blast horizontally over their lifetime.
  * `UpwardJetCm` (*Scalar*): How high the vertical fountain shoots upward during an eruption or impact.
  * `GravityDropCm` (*Scalar*): Parabolic downward gravity pull at the end of the particle's life so sand arcs back down to the desert floor.
  * `WindDriftVec` (*Vector*): World-space directional wind offset applied as the dust dissipates.
  * `CameraDepthFadeOffset` & `CameraDepthFadeLength` (*Scalars*): Smoothly fades particles out when the camera flies directly through them, preventing harsh screen clipping.
* **GPU Cost:** **312 VS instructions / 238 PS instructions / 1 Sampler** (`T_SandNoise_FBM_512`).

---

### 3.8 `M_SandVFX_GroundSandDrift` (World-Space Tile-Wrapping Ground Wind Veil)
* **Location:** `/DesertSandwormSystem/VFX/Materials/M_SandVFX_GroundSandDrift`
* **Material Instance:** `MI_SandVFX_GroundSandDrift` (used by `NS_Desert_GroundDriftStateless`).
* **What It Does (In Simple Terms):**  
  In many games, ambient ground dust emitters are attached to the camera, which causes an unnatural "dragging" artifact whenever the camera rotates or moves quickly. `M_SandVFX_GroundSandDrift` solves this with **100% Stationary World-Space Tile Wrapping** (`Custom_GroundSandDriftWPO`). The Niagara actor sits completely still at the world origin `(0, 0, 300)`, while the vertex shader reads `ActiveCameraPosWS` from `MPC_SandDeformation` and wraps every wind ribbon infinitely inside a `160 m x 160 m` (`TileSizeCm = 16000.0`) box centered around the viewer. It samples the desert heightmap (`T_DesertHeightmap_RG8`) at each wrapped world XY coordinate so the wind streams hug every dune crest and valley (`6 cm` to `130 cm` above ground) and stay 100% locked to the world when you pan or rotate the camera!
* **Key Editable Parameters & Visual Effects:**
  * `DesertHeightmapTex` (*Texture*): Points to `T_DesertHeightmap_RG8` so wind ribbons follow terrain elevation in the vertex shader.
  * `NoiseTex` (*Texture*): Points to `T_SandNoise_FBM_512` for fine blowing sand grain streaks.
  * `TileSizeCm` (*Scalar, default `16000.0`*): Width of the infinite wrapping world-space box around `ActiveCameraPosWS`. The outer 15% border automatically fades smoothly via `tileFade`.
  * `RibbonLengthMinCm` / `RibbonLengthMaxCm` (*Scalars, default `1800`..`2400`*): Length of the sweeping wind ribbons across the dunes.
  * `RibbonHeightMinCm` / `RibbonHeightMaxCm` (*Scalars, default `180`..`260`*): Vertical thickness of the ground-skimming sand veil.
  * `OpacityScale` (*Scalar, default `0.24`*): Subtlety of the blowing sand layer.
  * `SandTint` & `WindDriftVec` (*Vector*): Color and wind scroll velocity.
* **GPU Cost:** **445 VS instructions / 185 PS instructions / 2 Samplers** (1 VS Heightmap lookup + 1 PS Noise lookup).

---

### 3.9 `M_SandVFX_SandClump` (3D Mesh Particle Sand Clod Material)
* **Location:** `/DesertSandwormSystem/VFX/Materials/M_SandVFX_SandClump`
* **Where It Is Used:** Assigned to 3D mesh particle emitters (`bUsedWithNiagaraMeshParticles = true`) for solid flying chunks of compressed desert crust.
* **What It Does (In Simple Terms):**  
  An ultra-lightweight opaque Default Lit material that gives flying 3D debris chunks natural desert shading without any texture sampling overhead.
* **Key Editable Parameters:**
  * `ClumpColor` (*Vector Color*): Base albedo tint of the compressed sand clods.
* **GPU Cost:** **148 VS instructions / 142 PS instructions / 0 Samplers**.

---

### 3.10 `M_SandVFX_SandSpray` (Ballistic Granular Sand Jet Shader)
* **Location:** `/DesertSandwormSystem/VFX/Materials/M_SandVFX_SandSpray`
* **Where It Is Used:** High-energy granular spray layers during explosive emergence, body slams, and lateral tail sweeps.
* **What It Does (In Simple Terms):**  
  Shares the ballistic vertex displacement architecture (`Custom_HybridSandWPO` and `Custom_HybridSandSurface`) tuned specifically for sharp, high-velocity curtains of flying sand grains with strong solar rim-lighting (`SunDirectionWS`) and fast parabolic gravity drop.
* **Key Editable Parameters:**
  * `NoiseTex` (`T_SandNoise_FBM_512`), `SandTint`, `ClodTint`, `ClodMix`, `OpacityScale`, `DesiredRadiusCm`, `RadialPushCm`, `UpwardJetCm`, `GravityDropCm`, `WindDriftVec`, `CameraDepthFadeOffset`, `CameraDepthFadeLength`.
* **GPU Cost:** **312 VS instructions / 238 PS instructions / 1 Sampler** (`T_SandNoise_FBM_512`).

---

### 3.11 `M_SandVFX_WindSheet` (Lit Volumetric Directional Wind Curtain Shader)
* **Location:** `/DesertSandwormSystem/VFX/Materials/M_SandVFX_WindSheet`
* **Material Instance:** `MI_SandVFX_DesertWind` (used by `NS_Desert_WindAmbient`).
* **What It Does (In Simple Terms):**  
  A Default Lit (`MSM_DefaultLit`, `TLM_VolumetricDirectional`) translucent shader supporting both Niagara Sprites and Niagara Ribbons (`Custom_SandVFXWPO`, `Custom_SandVFXSurface`). It renders wide, atmospheric desert dust sheets that respond naturally to your level's Directional Light and SkyAtmosphere time-of-day changes while softly fading near the ground (`SoftFadeDistance`) and camera (`CameraDepthFadeOffset` / `CameraDepthFadeLength`).
* **Key Editable Parameters:**
  * `SandTint` (*Vector Color*), `OpacityScale` (*Scalar*), `SoftFadeDistance` (*Scalar* — depth fade against terrain), `CameraDepthFadeOffset` & `CameraDepthFadeLength` (*Scalars*), `DesiredRadiusCm`, `RadialPushCm`, `UpwardJetCm`, `GravityDropCm`, `WindDriftVec`.
* **GPU Cost:** **298 VS instructions / 215 PS instructions / 0 Samplers** (100% analytical noise in PS).

---

## 4. GPU Sand Deformation System (`ASandwormSandDeformer`)

`ASandwormSandDeformer` is the central simulation manager that connects the 136m Sandworm and dynamic gameplay objects to the Landscape shader.

### 4.1 How the Sliding Window & Ping-Pong Render Targets Work

Instead of allocating a massive, slow texture for an entire 10 km open world, `ASandwormSandDeformer` maintains a high-resolution **`1024 x 1024` floating-point sliding window** (`DeformExtentCm = 32000.0 cm`, covering **`320 m x 320 m`** at **`~31.25 cm per pixel`**) centered on the action:

```text
+-----------------------------------------------------------------------+
|                       DESERT LANDSCAPE (2 km+)                        |
|                                                                       |
|          +-------------------------------------------------+          |
|          |   SLIDING DEFORMATION WINDOW (320m x 320m)      |          |
|          |   Resolution: 1024x1024 RGBA16f (~31.25cm/px)   |          |
|          |                                                 |          |
|          |      [RT_SandPing]  <=======>  [RT_SandPong]    |          |
|          |       (Stamp Pass)             (30Hz Collapse)  |          |
|          |                                                 |          |
|          |             Center: CurrentCenterXY             |          |
|          |     (Follows Worm or Active Ground Interaction) |          |
|          +-------------------------------------------------+          |
|                                                                       |
+-----------------------------------------------------------------------+
```

1. **RGBA Channel Packing in `RT_SandPing` & `RT_SandPong`:**
   * **`R` (Red — Carve Depth):** Normalized downward excavation depth (`0.0..1.0` maps to `0..MaxCarveDepthCm`, default `620 cm`).
   * **`G` (Green — Berm & Subsurface Bulge Height):** Normalized upward displaced sand height (`0.0..1.0` maps to `0..MaxBermHeightCm`, default `340 cm`).
   * **`B` (Blue — Burrow Hole Mask):** Marks deep emergence and submerge wormholes (`0.0..1.0`), used by `M_DesertSand_Dynamic` to darken tunnel interiors and by `M_SandPhysics_CollapseStep` to trigger crater rim collapse.
   * **`A` (Alpha — Persistent Churned Sand Mask):** Tracks disturbed sand where natural wind ripples have been erased by movement.
2. **Zero-Wipe Texel Shifting (`UpdateWindowCenter`):**  
   When the window moves by more than `WindowRecenterThresholdFraction` (`8%` of `320 m`), it quantizes the movement to exact texel steps and passes `UVShift` into `M_SandPhysics_Stamp`. Existing trenches and craters simply scroll across the texture—**`ClearRenderTarget2D` is never called during movement**, so your tracks never vanish!
3. **Altitude-Aware `AutoSmart` Window Mode (`ESandWindowFollowMode`):**
   * `AutoSmart` (*Default*): Follows the Sandworm (`TargetSandwormActor`) or the last active arena zone. If the camera moves close to the ground (`<= AutoSmartMaxCameraHeightAboveGroundCm = 2500 cm`) and a dynamic object or player physically touches the sand far away, `NotifyDynamicInteractionAt` smoothly pulls the window to the player. If you fly the camera high up (`50 m – 120 m` in the air) just to look around, the window stays anchored on the Sandworm's craters so they stay 100% intact from afar.
   * `FollowSandworm`: Locks the window exclusively to `TargetSandwormActor`.
   * `FollowActiveCamera`: Locks the window to the active player or editor viewport camera.

### 4.2 `MPC_SandDeformation` Parameters

Every frame, `ASandwormSandDeformer::SyncMaterialParameterCollection()` updates `/DesertSandwormSystem/Materials/MPC_SandDeformation`:

| Parameter Name | Type | Driven By (`ASandwormSandDeformer`) | Description |
| :--- | :--- | :--- | :--- |
| `SandDeformCenterWS` | Vector | `CurrentCenterXY` (`X, Y, 0, 1`) | World-space XY center of the `320 m x 320 m` deformation window. |
| `ActiveCameraPosWS` | Vector | `GetActiveCameraView` (`X, Y, Z, 1`) | Live camera world position (used by `M_SandVFX_GroundSandDrift` for world-space wind wrapping). |
| `SunDirectionWS` | Vector | Material Parameter Collection default | Directional sunlight vector for VFX self-shadowing and dune backlighting. |
| `SandDeformExtentCm` | Scalar | `DeformExtentCm` (`32000.0`) | World width and length of the deformation window in centimeters. |
| `SandRTResolution` | Scalar | `GetRenderTargetResolution()` (`1024.0`) | Resolution of `RT_SandPing` / `RT_SandPong` for analytical normal filtering. |
| `MaxCarveDepthCm` | Scalar | `MaxCarveDepthCm` (`620.0`) | Maximum physical trench depth in centimeters. |
| `MaxBermHeightCm` | Scalar | `MaxBermHeightCm` (`340.0`) | Maximum physical lateral berm and underground bulge height in centimeters. |
| `DeformNormalStrength` | Scalar | `DeformNormalStrength` (`1.35`) | Intensity of normal map perturbation along carved slopes. |
| `BurrowDarkenStrength` | Scalar | `BurrowDarkenStrength` (`0.94`) | Ambient occlusion darkening inside emergence/submerge tunnels. |
| `SandEdgeFadeFraction` | Scalar | `EdgeFadeFraction` (`0.12`) | Smooth border falloff fraction around the window perimeter. |

### 4.3 Complete `ASandwormSandDeformer` Properties & Sliders (`Details` Panel)

| Property (`UPROPERTY`) | Category | Default | What It Controls |
| :--- | :--- | :--- | :--- |
| `TargetSandwormActor` | `Sand Physics\|Targets` | Auto | The active Sandworm actor whose spine bones are tracked. |
| `TargetLandscape` | `Sand Physics\|Targets` | Auto | The desert `ALandscapeProxy` used for ground height line traces. |
| `FollowerPatchActor` | `Sand Physics\|Targets` | Auto | Optional `SM_SandWormFollowerPatch` mesh that snaps to the window center in `FollowerPatchSnapStepCm` (`200 cm`) steps. |
| `WindowFollowMode` | `Sand Physics\|Area` | `AutoSmart` | Selects `AutoSmart`, `FollowSandworm`, or `FollowActiveCamera`. |
| `DeformExtentCm` | `Sand Physics\|Area` | `32000.0` | Total width/length of the deformable area (`320 m`). |
| `MaxCarveDepthCm` | `Sand Physics\|Area` | `620.0` | Maximum depth (`6.2 m`) of Sandworm trenches and craters. |
| `MaxBermHeightCm` | `Sand Physics\|Area` | `340.0` | Maximum height (`3.4 m`) of displaced sand walls and underground waves. |
| `TrenchWidthMultiplier` | `Sand Physics\|Area` | `1.28` | Scales trench width around the Sandworm's 10–14m diameter body. |
| `DeformNormalStrength` | `Sand Physics\|Area` | `1.35` | Sharpness of lighting normals on carved sand slopes. |
| `BurrowDarkenStrength` | `Sand Physics\|Area` | `0.94` | Shadow intensity inside deep burrow holes. |
| `AngleOfReposeDeg` | `Sand Physics\|Simulation` | `33.0` | Critical slope angle (degrees). Slopes steeper than `33°` avalanche until they stabilize at `33°`. |
| `AvalancheDiffusionRate` | `Sand Physics\|Simulation` | `10.5` | How fast over-steepened sand walls slide downward into trenches. |
| `BurrowClosureRate` | `Sand Physics\|Simulation` | `0.28` | Speed at which deep wormholes collapse and seal after the tail submerges. |
| `AmbientInfillRate` | `Sand Physics\|Simulation` | `0.0` | Wind infill rate. Kept at `0.0` so tracks remain permanent instead of magically disappearing. |
| `SurfaceSmoothStrength` | `Sand Physics\|Simulation` | `0.12` | 3x3 Gaussian diffusion filter strength during active avalanches to prevent jagged pixels. |
| `CollapseStepFrequencyHz` | `Sand Physics\|Simulation` | `30.0` | Fixed GPU update rate (`30 Hz`) for the gravity collapse pass. |
| `CollapseIdleSleepSeconds` | `Sand Physics\|Simulation` | `25.0` | Puts the GPU solver to sleep (`0` GPU passes) `25s` after the last stamp. |
| `bAutoDeformDynamicObjects` | `Sand Physics\|DynamicObjects` | `true` | Automatically scans and carves sand beneath moving physics objects, rocks, and characters. |
| `DynamicObjectMinStampRadiusCm` / `Max` | `Sand Physics\|DynamicObjects` | `45.0` / `350.0` | Clamps automatic footprint/object stamp radii to realistic proportions. |
| `DynamicObjectContactToleranceCm` | `Sand Physics\|DynamicObjects` | `4.0` | Objects must be within `4 cm` of the ground surface to leave a mark (zero air stamping). |
| `DynamicObjectBermWidthScale` | `Sand Physics\|DynamicObjects` | `0.40` | Tightens the displaced sand rim right against the perimeter of small dynamic objects. |

### 4.4 Blueprint & C++ Functions on `ASandwormSandDeformer`

* **`ResetSandSurface()`** (*CallInEditor / BlueprintCallable*): Instantly clears `RT_SandPing` and `RT_SandPong`, restoring untouched desert dunes (also available as a one-click button in the *Details* panel).
* **`StampCarveAndBerm(WorldPos, RadiusCm, CarveStrength, BermStrength, BermWidthScale)`**: Stamps a custom trench and raised rim at any world location (e.g., for artillery explosions or vehicle wheels).
* **`StampSubsurfaceBulge(WorldPos, RadiusCm, BulgeStrength, TrailCarveStrength)`**: Stamps a rising underground sand wave without excavating a pit.
* **`StampBurrowHole(WorldPos, RadiusCm, CarveStrength, BermStrength, BurrowMaskStrength)`**: Carves a deep dark wormhole with an elevated collar rim.
* **`StampImpactCrater(WorldPos, RadiusCm, CarveStrength, BermStrength)`**: Stamps a wide radial shockwave crater.
* **`SetActiveBurrowKeepOpen(bEnabled, WorldPos, RadiusCm, Strength)`**: Prevents a burrow hole from collapsing while an actor is still emerging or diving through it.

---

## 5. Autonomous AI & Combat (`USandwormCombatAIComponent`)

`USandwormCombatAIComponent` gives any Sandworm actor full autonomous life in the desert, seamlessly blending between locomotion, deep underground burrowing, and multi-attack boss combat via `USandwormBlendAnimInstance`.

### 5.1 AI States (`ESandwormCombatAIState`)

| Enum Value | State Name | Behavior Description |
| :--- | :--- | :--- |
| `0` | **`PatrolSlither`** | Surface patrol around `HomeLocation` within `PatrolRadiusCm` (`9000 cm`). |
| `1` | **`SubsurfaceStalk`** | High-speed underground pursuit (`SubsurfaceSpeedCmPerSec = 2400 cm/s`) toward a detected vibration or threat, pushing up a live `SubsurfaceBulge` wave on the sand surface. |
| `2` | **`ThreatAlert`** | Erupts (`AnimEmergeV2`) or turns toward a newly detected threat and unleashes an intimidating `AnimRoar` with a sonic shockwave before attacking. |
| `3` | **`HitReaction`** | Triggered when taking heavy damage (`OnTakeAnyDamage`), blending smoothly into `AnimHitReact` with an internal cooldown so the boss cannot be stun-locked. |
| `4` | **`CombatAttack`** | Selects and executes one of 4 contextual attacks (`ESandwormAttackType`): **`Spit`** (`AnimAttackSpit` at long range `>= SpitRangeMinCm = 4800 cm`), **`Sweep`** (`AnimAttackSweep` when targets flank past `FlankAngleThresholdDeg = 38°`), **`BodySlam`** (`AnimAttackBodySlam`), or **`Lunge`** (`AnimAttackLunge`). |
| `5` | **`BurrowAmbush`** | 3-phase tactical reposition (`EBurrowAmbushPhase::Submerging` -> `UndergroundReposition` -> `Emerging`) executed after consecutive surface attacks to ambush the target from below. |
| `6` | **`Dead`** | Triggered when `CurrentHealth <= 0.0f`. Plays `AnimDeath`, triggers the seismic collapse audio/VFX, and halts AI locomotion. |
| `7` | **`AutonomousSlither`** | Organic free-roam surface slithering (`SurfaceSlitherSpeedCmPerSec = 1650 cm/s`) across dunes for `FreeRoamSlitherMinDuration` (`7.0s`) to `FreeRoamSlitherMaxDuration` (`14.0s`). |
| `8` | **`AutonomousBurrowTravel`** | Dives underground (`AnimSubmerge`), travels `65 m – 105 m` (`FreeRoamBurrowMinDistCm = 6500` .. `MaxDistCm = 10500`) beneath the sand while carving a surface bulge wave, and erupts (`AnimEmergeV2`) at the new destination. |
| `9` | **`AutonomousSurveyOrRoar`** | Pauses after emerging or slithering to rear up (`AnimIdleCombat`) or bellow across the desert (`AnimRoar`) before picking a new organic waypoint. |

### 5.2 Automatic Safe Patrol & Free-Roam Boundaries

You never have to worry about the 136m Sandworm slithering off the edge of your map:
* On `BeginPlay`, `ResolveSafeLandscapeBounds()` inspects the bounding box of your level's Landscape and subtracts **`LandscapeEdgeMarginCm`** (default `25000 cm` / `250 m`).
* In addition, waypoints are clamped within **`RoamMaxRadiusCm`** (default `45000 cm` / `450 m`) around `RoamBoundsCenterXY` (or the worm's starting position when `bUseInitialLocationAsRoamCenter = true`).
* As the worm approaches the boundary perimeter, `ApplySoftBoundaryAvoidance()` gently steers its heading (`CurrentHeading2D`) back toward the safe desert interior.
* You can update the safe zone at runtime from Blueprint or C++ by calling **`SetRoamBounds(InCenterXY, InMaxRadiusCm)`**.

### 5.3 Ready-to-Use Blueprint & C++ Combat & Damage Functions

Integrating the Sandworm with your project's weapons, spells, or vehicles requires zero custom AI coding:

1. **Standard Unreal Damage Support (`ApplyDamage`):**  
   `USandwormCombatAIComponent` automatically binds to the owning Sandworm actor's **`OnTakeAnyDamage`** event and enables query/damage collision on `SK_DesertSandworm_PhysicsAsset`. Calling Unreal's built-in **`UGameplayStatics::ApplyDamage(SandwormActor, DamageAmount, InstigatorController, DamageCauser, DamageTypeClass)`** from any projectile or melee trace automatically deducts `CurrentHealth` (from `MaxHealth = 2500.0`), triggers `HitReaction` or `Dead`, and sets the attacker as `CurrentTargetActor`!
2. **One-Click Editor & Blueprint Testing Functions:**
   * **`TriggerTestThreat(Location)`** (*CallInEditor / BlueprintCallable*): Simulates a loud footstep/vehicle vibration at `Location` (or 35m in front of the worm's head if left at `(0,0,0)`), immediately pulling the worm into `ThreatAlert` / `CombatAttack`.
   * **`ApplyTestDamage(Amount = 250.0, Instigator = nullptr)`** (*CallInEditor / BlueprintCallable*): Deals instant test damage to verify hit reactions, health bars, and death sequences directly in the Editor.
   * **`SetAutonomousAIEnabled(bEnabled, bResetBehavior = true)`** (*BlueprintCallable*): Toggles autonomous desert exploration on or off at runtime.
   * **`ForceAutonomousBurrowNow()`** (*CallInEditor / BlueprintCallable*): Commands the worm to immediately dive underground (`Submerging -> UndergroundReposition -> Emerging`).
   * **`GetHealthNormalized()`** (*BlueprintPure*): Returns `0.0..1.0` for easy binding to a Boss Health Bar UMG progress bar.
   * **`GetCurrentState()` / `GetCurrentAIStateName()` / `IsUnderground()` / `GetAITrackFocusPoint()`** (*BlueprintPure*): Query live AI state and ideal camera tracking coordinates.
   * **`OnStateChanged`** (*BlueprintAssignable Delegate*): Fires `(ESandwormCombatAIState OldState, ESandwormCombatAIState NewState)` whenever the AI transitions states.

---

## 6. Audio & MetaSounds (`USandwormSandInteractionComponent`)

The plugin ships with a **100% bespoke, copyright-free 48 kHz / 16-bit PCM soundscape** normalized to `-1.00 dBFS`, wrapped in 12 modular **MetaSounds** (`/DesertSandwormSystem/Audio/MetaSounds/`) with real-time DSP filtering.

### 6.1 Complete Table of All 12 MetaSounds & Sound Waves

Every `MS_Sandworm_*` MetaSound exposes three real-time input parameters (`PitchShiftSemitones`, `CutoffFrequencyHz`, and `GainScale`) feeding a `Wave Player -> One-Pole Low Pass Filter -> Multiply` audio graph:

| # | MetaSound Asset (`/DesertSandwormSystem/Audio/MetaSounds/`) | Underlying SoundWave (`/DesertSandwormSystem/Audio/Waves/`) | Channels / Length | Component Property Slot | Acoustic Profile & Trigger Event |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | `MS_Sandworm_Roar` | `SFX_Sandworm_Roar` | Mono / `5.45s` | `SFX_Roar` (`USandwormSandInteractionComponent`) | Guttural leviathan roar with `32–65 Hz` sub-bass braam and desert air pulse during `AS_Sandworm_Roar`. |
| **2** | `MS_Sandworm_Emerge` | `SFX_Sandworm_Emerge` | Mono / `3.30s` | `SFX_Emerge` (`USandwormSandInteractionComponent`) | Deep subterranean ground rupture (`30–185 Hz`) followed by a 3-second cascade of sand pouring off the carapace (`AS_Sandworm_Emerge_V2`). |
| **3** | `MS_Sandworm_Submerge` | `SFX_Sandworm_Submerge` | Mono / `3.50s` | `SFX_Submerge` (`USandwormSandInteractionComponent`) | Heavy downward sand suction (`115 -> 27 Hz`) and crater rim avalanche during `AS_Sandworm_Submerge`. |
| **4** | `MS_Sandworm_Attack_BodySlam` | `SFX_Sandworm_Attack_BodySlam` | Mono / `2.35s` | `SFX_AttackBodySlam` (`USandwormSandInteractionComponent`) | Colossal seismic body impact with a `130 -> 26 Hz` pitch drop and displaced sand wave (`AS_Sandworm_Attack_Body_Slam`). |
| **5** | `MS_Sandworm_Attack_Whoosh` | `SFX_Sandworm_Attack_Whoosh` | Mono / `1.30s` | `SFX_AttackWhoosh` (`USandwormSandInteractionComponent`) | Heavy Doppler air displacement (`130 -> 680 -> 140 Hz`) triggered during `AS_Sandworm_Attack_Sweep` and `Anim_Sandworm_Attack_Lunge`. |
| **6** | `MS_Sandworm_Attack_Spit` | `SFX_Sandworm_Attack_Spit` | Mono / `1.42s` | `SFX_AttackSpit` (`USandwormSandInteractionComponent`) | Viscous, resonant acid/venom expulsion blast during `AS_Sandworm_Attack_Spit`. |
| **7** | `MS_Sandworm_HitReact` | `SFX_Sandworm_HitReact` | Mono / `1.45s` | `SFX_HitReact` (`USandwormSandInteractionComponent`) | Chitinous armor crack fused with a low guttural pain grunt during `AS_Sandworm_Hit_React`. |
| **8** | `MS_Sandworm_Death` | `SFX_Sandworm_Death` | Mono / `5.10s` | `SFX_Death` (`USandwormSandInteractionComponent`) | Fading agonizing bellow (`-14.2 -> -22.0` semitones) culminating in a seismic ground collapse at `4.38s` (`AS_Sandworm_Death`). |
| **9** | `MS_Sandworm_Slither_Loop` | `SFX_Sandworm_Slither_Loop` | Mono / `6.00s` (Loop) | `SFX_SlitherLoop` (`USandwormSandInteractionComponent`) | Seamless granular friction loop of heavy chitin scales dragging across coarse sand; volume and pitch modulate dynamically with live spine speed. |
| **10** | `MS_Sandworm_Subsurface_Loop` | `SFX_Sandworm_Subsurface_Loop` | Mono / `6.00s` (Loop) | `SFX_SubsurfaceLoop` (`USandwormSandInteractionComponent`) | Seamless `30–65 Hz` tectonic sub-bass rumble and compressed earth friction while the worm travels underground. |
| **11** | `MS_Sandworm_DesertWind_Loop` | `SFX_DesertWind_Loop` | Stereo / `12.00s` (Loop) | `SFX_DesertWindLoop` (`ASandwormShowcaseDirector`) | Seamless spectral FFT desert wind loop (`48–720 Hz`) with slow harmonic gusts and zero harsh whistling. |
| **12** | `MS_Sandworm_Trailer_GTATransition` | `SFX_Trailer_GTATransition` | Stereo / `2.80s` | `SFX_TrailerGTATransition` (`ASandwormShowcaseDirector`) | 3 dry low-end camera step punches on ascent (`0.004s, 0.317s, 0.635s`), high-altitude stereo whoosh (`1.40s`), and 3 step punches on descent (`1.848s, 2.165s, 2.483s`). |

### 6.2 Colossal Spatial Attenuation (`ATT_Sandworm_Colossal`)

Standard Unreal attenuation presets are designed for human-sized footsteps (`10–30 m` falloff), which causes a 136-meter monster to go silent while still filling half the screen. `/DesertSandwormSystem/Audio/ATT_Sandworm_Colossal` (`USoundAttenuation`) is pre-calibrated for leviathan scale:
* **Distance Algorithm:** `Logarithmic`
* **Inner Radius (`100%` volume):** `3500 cm` (`35 meters`)
* **Falloff Distance (Audible Horizon):** `28000 cm` (`280 meters`)
* **Non-Spatialized Radius Start & Stereo Spread:** `1200 cm` inner blend with `450 cm` 3D stereo width so close-range roars envelop both speakers without hard single-ear panning.
* **Distance Low-Pass Air Absorption (LPF):** Smoothly rolls off high frequencies from `20000 Hz` down to `2600 Hz` between `45 m` and `280 m`, making distant eruptions sound naturally booming and atmospheric across the desert.

### 6.3 How to Replace Any Sound in the `Details` Panel

All audio slots on `USandwormSandInteractionComponent` (and `ASandwormShowcaseDirector`) use the base `USoundBase*` type, which accepts **MetaSounds (`UMetaSoundSource`), Sound Cues (`USoundCue`), or raw Sound Waves (`USoundWave`)**:
1. Select your Sandworm actor in the level and click **`SandwormSandInteractionComponent`** in the component tree.
2. Scroll to the **`Sandworm | Audio`** category in the *Details* panel.
3. Replace any of the 10 creature slots (`SFX_Roar`, `SFX_Emerge`, `SFX_Submerge`, `SFX_AttackBodySlam`, `SFX_AttackWhoosh`, `SFX_AttackSpit`, `SFX_HitReact`, `SFX_Death`, `SFX_SlitherLoop`, `SFX_SubsurfaceLoop`) with your own asset, or tweak `AudioMasterVolume` (default `1.0`), `SlitherLoopGain` (`0.55`), `SubsurfaceLoopGain` (`0.72`), and `AttackWhooshGain` (`0.75`).

### 6.4 Fine-Tuning the 9 Synchronized Animation Audio Delays

If you swap in custom animations or altered sound effects with different attack transients, you can adjust the exact playback trigger timestamps (in seconds from the start of each animation clip) under **`Sandworm | Audio`** on `USandwormSandInteractionComponent`:

| Delay Property (`UPROPERTY`) | Default (`sec`) | Target Animation (`Length`) | Normalized Time | Exact Visual Moment Synchronized |
| :--- | :--- | :--- | :--- | :--- |
| **`RoarDelaySec`** | **`0.62s`** | `AS_Sandworm_Roar` (`6.25s`) | `0.100` | Instant the colossal jaws snap open and the sonic wave erupts. |
| **`BodySlamImpactDelaySec`** | **`4.95s`** | `AS_Sandworm_Attack_Body_Slam` (`7.292s`) | `0.679` | Pre-triggers `0.15s` before the `5.10s` belly slam so peak sub-bass hits on ground impact. |
| **`SpitDelaySec`** | **`1.82s`** | `AS_Sandworm_Attack_Spit` (`5.208s`) | `0.350` | Forward head snap as the acid projectile is expelled. |
| **`SweepWhooshDelaySec`** | **`2.00s`** | `AS_Sandworm_Attack_Sweep` (`6.042s`) | `0.331` | Peak lateral whip acceleration across the dunes. |
| **`LungeWhooshDelaySec`** | **`1.35s`** | `Anim_Sandworm_Attack_Lunge` (`5.00s`) | `0.270` | Forward strike acceleration toward the target. |
| **`HitReactDelaySec`** | **`0.13s`** | `AS_Sandworm_Hit_React` (`3.125s`) | `0.042` | Initial flinch recoil upon taking weapon damage. |
| **`EmergeDelaySec`** | **`0.38s`** | `AS_Sandworm_Emerge_V2` (`6.25s`) | `0.061` | Pre-triggers `0.12s` before the `0.50s` surface breach so the explosion matches the sand eruption. |
| **`SubmergeDelaySec`** | **`1.95s`** | `AS_Sandworm_Submerge` (`6.25s`) | `0.312` | Vertical dive entry into the burrow funnel. |
| **`DeathDelaySec`** | **`0.55s`** | `AS_Sandworm_Death` (`7.50s`) | `0.073` | Starts the 5.10s agonizing death roar so its final seismic thud lands squarely on the `4.38s` body collapse. |

---

## 7. Showcase Director (`ASandwormShowcaseDirector`)

Placed in `/DesertSandwormSystem/Maps/L_Sandworm_DesertShowcase`, `ASandwormShowcaseDirector` orchestrates the 3 isolated desert zones, camera choreography, tick isolation, and on-screen telemetry.

### 7.1 Exhibition Modes (`ShowcaseMode`)

Select the `SandwormShowcaseDirector` actor in the level *Outliner* and change **`ShowcaseMode`** (under `Showcase | Playback`) to fit your needs:

1. **`ESandwormShowcaseMode::FullOneTakeTrailer` (*Default*):**  
   Runs the complete, unedited 3-block showcase in a single continuous take:
   * **Block 1 — Boss Cinematic (`34.0s`, `Block1DurationSeconds`):** 5-act scripted boss encounter (`SubterraneanApproach` -> `ExplosiveEruption` -> `Roar` with sonic wave -> `ColossalSlitherAndStrikes` -> `SubmergeAndCollapse`).
   * **GTA Transition #1 (`2.80s`, `GTATransitionDuration`):** 3-step aerial camera launch (`+25 m`, `+85 m`, `+200 m`), high-altitude flight arc over the desert while shifting the Sand Deformer window, and 3-step descent into Zone 2.
   * **Block 2 — Animation Gallery (`~28.7s`):** Showcases all 11 individual combat and locomotion animations on a clean dune ridge (`AnimationGalleryWorm`) with golden-angle orbiting camera work and `EvaluateAudioOnlyStep`.
   * **GTA Transition #2 (`2.80s`):** 3-step aerial transition from the Animation Gallery to the Open Desert (`FreeRoamWorm`).
   * **Block 3 — Autonomous Free-Roam AI (`bHoldFreeRoamIndefinitely = true`):** Hands control over to `USandwormCombatAIComponent` (`FreeRoamWorm`) and follows the autonomous creature indefinitely with an adaptive look-ahead drone camera (`DroneSlitherDistCm = 11400`, `DroneSubmergeDistCm = 11800`, `DroneEmergeDistCm = 12200`).
2. **`ESandwormShowcaseMode::BossCinematicOnly`:**  
   Loops the 54-second 5-act Boss Arena cinematic (`TotalDurationSeconds = 54.0`). By default, `bResetSandOnLoop = false`, allowing craters and trenches to accumulate across loops (or set `bResetSandOnLoop = true` for a fresh dune each cycle).
3. **`ESandwormShowcaseMode::FreeRoamAIDemo`:**  
   Skips the scripted cinematic and jumps straight into Block 3 (100% autonomous Free-Roam AI with the adaptive drone camera).

### 7.2 Live Performance Overlay (`bShowPerfStats`) & Pure-Slate Labels (`bShowTrailerLabels`)

* **`bShowPerfStats` (`Category = "Sandworm|Showcase"`, default `true`):**  
  Automatically enables Unreal Engine's native **`stat fps`** and **`stat unit`** counters on `BeginPlay` (safely checking `UGameViewportClient::IsStatEnabled` so it never accidentally toggles them off if already enabled). This lets you monitor live **FPS, Frame ms, Game Thread ms, Draw Thread ms, and GPU ms** even in fullscreen (`F11`) mode. Uncheck **`bShowPerfStats`** in the *Details* panel to hide the counters with one click.
* **`bShowTrailerLabels` (`Category = "Showcase|Trailer|Overlay"`, default `true`):**  
  Displays a clean, pure-Slate C++ title card overlay (`GetCurrentOverlayHeader()` / `GetCurrentOverlaySubtitle()`) identifying the active trailer block and animation clip without creating any UMG Blueprint widget clutter.

### 7.3 Customizing `GalleryEntries` & Aerial Transitions

* **`GalleryEntries` (`TArray<FSandwormGalleryEntry>` under `Showcase | Gallery`):**  
  Contains 11 pre-configured entries (`IDLE COMBAT`, `SLITHER`, `ROAR`, `ATTACK SWEEP`, `ATTACK BODY SLAM`, `ATTACK LUNGE`, `ATTACK SPIT`, `HIT REACT`, `EMERGE V2`, `SUBMERGE`, `DEATH`). Each entry lets you customize `DisplayName`, `AnimSequence`, `DurationSeconds`, `bLooping`, `StartNormalized`, `EndNormalized`, `CameraOrbitStartYawDeg`, `CameraOrbitSweepDeg`, `CameraDistanceCm` (`10800–12800 cm`), `CameraHeightCm` (`2500–3200 cm`), and `CameraFocalLengthMm` (`28 mm`).
* **Aerial Transition Tuning (`Showcase | Trailer | GTATransition`):**  
  Adjust step heights (`GTAStep1HeightCm = 2500`, `GTAStep2HeightCm = 8500`, `GTAStep3HeightCm = 20000`, `GTAAerialArcExtraHeightCm = 3500`) and optical camera punch (`GTAStepFocalPunchMm = 6.5`, `GTAStepFringeIntensity = 2.6`, `GTAStepSubtleSonicPulse = 0.14`).
* **One-Click Editor Controls (`Showcase | Controls` buttons in `Details`):**  
  Click **`StartShowcase`**, **`StopShowcase`**, **`ResetSandManual`**, or call **`JumpToTrailerPhase`** / **`EvaluateShowcaseAtTime(TimeSeconds)`** to scrub the choreography directly in the Editor viewport.

---

## 8. Troubleshooting & FAQ

### Q1: I placed the Sandworm and `ASandwormSandDeformer` in my own map, but the ground doesn't deform when the worm moves. What did I miss?
**Answer:** Check these three quick items:
1. Make sure your Landscape (or ground mesh) has **`MI_DesertSand_Dynamic`** assigned as its material (or `MI_DesertSand_FollowerPatch` if using `SM_SandWormFollowerPatch`). Standard non-deformable materials do not read `RT_SandPing` or `MPC_SandDeformation`.
2. Select your `ASandwormSandDeformer` actor and ensure **`TargetSandwormActor`** points to your active Sandworm actor and **`TargetLandscape`** points to your Landscape.
3. Make sure your Sandworm actor has **`USandwormSandInteractionComponent`** attached (which reads the 27 spine bones and sends stamps to the deformer)—or check **`bTrackBonesAutomatically = true`** on `ASandwormSandDeformer`.

### Q2: My physics cubes or player character aren't leaving tracks in the sand. Why?
**Answer:** By design, the system enforces **strict ground contact (`DynamicObjectContactToleranceCm = 4.0 cm`)** so objects hovering in the air never falsely dent the ground.
1. Ensure the bottom of your object's collision primitive actually touches within `4 cm` of the Landscape surface.
2. Ensure your object's Mobility is set to **Movable** and it generates overlap/collision queries (`Collision Enabled`).
3. If your player is more than `140 meters` (`DynamicObjectScanRadiusCm = 14000 cm`) away from the Sandworm, attach **`USandDeformInteractorComponent`** to your Player Pawn—it automatically calls `NotifyDynamicInteractionAt` to pull the `320 m x 320 m` deformation window to the player's location when `WindowFollowMode` is set to `AutoSmart`.

### Q3: How do I disable the cinematic camera and use my own Player Character in `L_Sandworm_DesertShowcase`?
**Answer:** Select the **`SandwormShowcaseDirector`** actor in the *Outliner*:
1. Uncheck **`bAutoPlayOnBeginPlay`** (or set `ShowcaseMode` to `FreeRoamAIDemo` and uncheck **`bDriveCinematicCamera`**, **`bAutoPossessShowcaseCameraOnBeginPlay`**, **`bHidePlayerPawnDuringShowcase`**, and **`bHideGameHUDDuringShowcase`** under `Showcase | Camera`).
2. Or simply delete `SandwormShowcaseDirector` from the map and enable **`bEnableAutonomousAI = true`** on the `USandwormCombatAIComponent` of the Sandworm you wish to fight!

### Q4: How do I turn off the green FPS / Frame / GPU stats in the top-right corner of the screen?
**Answer:** Select **`SandwormShowcaseDirector`** in the *Outliner*, look under the **`Sandworm | Showcase`** category in the *Details* panel, and uncheck **`bShowPerfStats`**.

### Q5: When I fly my camera very high up (`100+ meters` in the air) away from the Sandworm, why doesn't the sand deformation window follow my camera?
**Answer:** This is a deliberate protection feature of **`ESandWindowFollowMode::AutoSmart`** (`AutoSmartMaxCameraHeightAboveGroundCm = 2500.0 cm`). If the window followed a high-altitude flying camera into empty desert, the Sandworm's arena craters would scroll out of the `320 m x 320 m` Render Target window. Keeping the window anchored on the active zone when the camera is high in the air ensures you can view the Sandworm's trenches and craters from `120+ meters` away with zero loss of detail. If you *want* the window to always follow the camera regardless of altitude, change `WindowFollowMode` on `ASandwormSandDeformer` to **`FollowActiveCamera`**.

### Q6: Why don't I see the screen distortion wave when the Sandworm roars in my custom level?
**Answer:** The roar sonic shockwave requires a **PostProcessVolume** in your level with `/DesertSandwormSystem/Materials/M_PP_RoarSonicWave` added to its **Rendering Features -> Post Process Materials -> Array**. Also ensure your PostProcessVolume has **Infinite Extent (Unbound)** checked so the effect is visible everywhere in your desert.
