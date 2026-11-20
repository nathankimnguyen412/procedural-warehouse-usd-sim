# Procedural Warehouse Simulation

A procedural warehouse simulation environment generator built in SideFX Houdini for robotics training. Generates standards-based and physics-enabled industrial environments modeled after North American Class A distribution center specifications, with articulated assets (cabinets, carts, valves, equipment cases) and static shelving, exports USD with UsdPhysics schemas (rigid bodies, collision proxies, mass, articulation roots, joint hierarchies), and validates in NVIDIA Isaac Sim. Layout logic follows OSHA 1910.176, NFPA 13, NEC 110.26, and ANSI MH16.1 with configurable compliance parameters.

---

## Visual Showcase

### Procedural Generation Demo

<video src="https://github.com/user-attachments/assets/495b0fb1-6a8d-44d4-8ab3-477e3e7ac0ea" width="100%" autoplay loop muted playsinline></video>

### Warehouse Environment
<img src="screenshots/procedural_warehouse1.PNG"/>
<img src="screenshots/procedural_warehouse2.PNG"/>
<img src="screenshots/procedural_warehouse3.PNG"/>
<img src="screenshots/procedural_warehouse4.PNG"/>
<img src="screenshots/procedural_warehouse5.PNG"/>
<img src="screenshots/procedural_warehouse6.PNG"/>
<img src="screenshots/procedural_warehouse7.PNG"/>

### Procedural Networks
<img src="screenshots/sop_network.PNG"/>

### Fully Procedural Models
<table>
  <tr>
    <td align="center"><img src="screenshots/industrial_cabinet_HDA.PNG" width="360"/></td>
    <td align="center"><img src="screenshots/industrial_cabinet_HDA2.PNG" width="360"/></td>
  </tr>
</table>
<table>
  <tr>
    <td align="center"><img src="screenshots/industrial_cabinet_HDA3.PNG" width="360"/></td>
    <td align="center"><img src="screenshots/industrial_cabinet_HDA4.PNG" width="360"/></td>
  </tr>
</table>
<img src="screenshots/warehouse_shelving_HDA_visual.PNG" width="500"/>
<img src="screenshots/wheeled_cart_HDA_Solaris_viewport.PNG" width="500"/>
<img src="screenshots/lever_valve_viewport.PNG" width="500"/>
<img src="screenshots/equipment_case_viewport.PNG" width="500"/>

### Isaac Sim Validation
<img src="screenshots/cabinet_after_clicking_play2.PNG" width="500"/>
<img src="screenshots/after_clicking_play3.PNG" width="500"/>
<table>
  <tr>
    <td align="center"><img src="screenshots/final_wheeled_cart_before_play.PNG" width="360"/></td>
    <td align="center"><img src="screenshots/final_wheeled_cart_after_play.PNG" width="360"/></td>
  </tr>
</table>
<table>
  <tr>
    <td align="center"><img src="screenshots/Before_Clicking_Play.PNG" width="360"/></td>
    <td align="center"><img src="screenshots/After_Clicking_Play.png" width="360"/></td>
  </tr>
</table>
<img src="screenshots/lid_moving_correctly.PNG" width="500"/>
<video src="https://github.com/user-attachments/assets/b0351d79-1c28-449f-9fe1-353294766585" width="100%" autoplay loop muted playsinline></video>
<video src="https://github.com/user-attachments/assets/dff85f6e-f319-46d1-a9ca-30c10c653bd3" width="100%" autoplay loop muted playsinline></video>

---

## Features

- **Procedural warehouse layout generation** modeled after North American Class A distribution center specifications (OSHA 1910.176, NFPA 13, NEC 110.26, ANSI MH16.1) with configurable compliance parameters
- **Five fully procedural asset HDAs** with UsdPhysics schemas, four articulated and one static:
  - Equipment Case: revolute joint lid with angular limits, fixed latch
  - Lever Valve: revolute joint with angular limits, pipe-mounted
  - Warehouse Shelving: multi-tier pallet racking, adjustable heights (static, no joints)
  - Industrial Cabinet: prismatic drawer joints with travel limits
  - Wheeled Cart: revolute wheel joints on fixed casters, rolling platform
- **USD physics authoring** via Python LOPs:
  RigidBodyAPI, CollisionAPI (convexHull), MassAPI, ArticulationRootAPI, joint hierarchies with limits
- **Validated Houdini-to-Isaac Sim pipeline** with correct physics behavior
- **Checked against NVIDIA's SimReady Foundation specification** with the `simready-validate` tool (results below)
- **Modular two-layer HDA architecture** (SOP geometry / LOP physics)
- Configurable violation rate for realistic warehouse messiness
- Regional configuration support (US GMA pallet / EU EUR pallet standards)

---

## Technical Stack

- **Houdini**: VEX, Python, LOPs/Solaris, HDA development
- **USD/OpenUSD**: UsdPhysics schemas
- **NVIDIA Isaac Sim**: Physics validation, articulated asset testing
- **NVIDIA SimReady Foundation**: Specification conformance checks with `simready-validate`
- **Python**: USD authoring scripts, pipeline automation

---

## Architecture

The pipeline follows a two-layer architecture:

1. **SOP Layer**: Procedural geometry generation, layout logic, asset placement with collision avoidance and standards-compliant clearances
2. **LOP Layer**: USD scene assembly, UsdPhysics schema application (RigidBodyAPI, CollisionAPI, MassAPI, ArticulationRootAPI, joint definitions)

---

## SimReady Foundation Validation

Testing in Isaac Sim shows that the assets behave correctly. It does not show that the USD files are authored the way other tools and pipelines expect. To check that, the exported warehouse USD was validated against NVIDIA's [SimReady Foundation](https://nvidia.github.io/simready-foundation/latest/guides/getting_started.html) specification using its `simready-validate` command-line tool.

SimReady Foundation defines what makes a USD asset ready for simulation. It is organized as **requirements** (single checkable rules), grouped into **features**, grouped into **profiles**. An asset is validated against one profile.

| | |
| --- | --- |
| Asset validated | Full warehouse scene, 161 rigid bodies |
| Profile | `Prop-Robotics-Physx` v2.1.0 |
| Result | Does not pass the profile yet: 588 issues across 7 requirements |
| Physics | The rigid body and collider feature and the joint and articulation feature raised no issues |

All remaining issues are in visual geometry, visual materials, and asset packaging.

### Remaining issues

| Requirement | Count | What it means | How it will be addressed |
| --- | --- | --- | --- |
| `VG.027` Normals exist | 306 | A normal tells the renderer which way a surface faces. Each mesh must store normals or be marked as a subdivision surface. Affects 161 visual meshes and 145 collision meshes. | Author normals in SOPs before the USD export. |
| `VM.MAT.001` Material binding | 209 | Geometry has no visual material assigned, so it renders as default gray. Affects 161 visual meshes and 48 helper point prims that only exist to mark joint positions. | Assign materials to the visual meshes as part of environment texturing. Exclude the joint marker prims from the export once their positions have been read. |
| `VG.008` Coincident meshes | 69 | A part's visual mesh and collision mesh are identical geometry at the same transform. The validator flags this because it usually indicates an accidental duplicate. | Intentional for now: simple parts reuse their visual shape as the collider. Planned replacement with dedicated simplified collision shapes. |
| `VG.025` Asset origin positioning | 1 | The root prim carries a rotation. Pre-rotated assets are harder to place and instance in other scenes. | The rotation comes from converting Houdini's Y-up to Isaac Sim's Z-up on export. A planned pipeline rebuild will author natively in Z-up so the root needs no rotation. |
| `GSP.001` Grasp vector | 1 | The profile targets props a robot can pick up and expects a marker showing where and from which direction to grasp. | Does not apply to a whole scene. Grasp vectors will be added to the individual graspable assets when they are validated on their own. |
| `NP.005` Asset folder structure | 1 | SimReady expects a set folder layout, with the file name containing the asset root folder's name. | Export into a SimReady-style asset folder. |
| `NP.006` Metadata location | 1 | The asset has no SimReady metadata, which must be stored in the layer's custom data or in a sidecar JSON file. | Write the metadata block during the export's post-render step. |

---

## Next Steps

- **Cabinet doors**: add revolute door joints to the Industrial Cabinet
- **Materials**: author UsdShade materials and add material assignment to the LOP layer
- **TOPs/PDG batch layer**: add a third layer to the architecture for PDG-orchestrated batch generation of environment variants, with parameterized randomization across layout, assets, and materials
- **Karma XPU rendering**: render the generated environments with Karma XPU
- **Composition arcs**: move from a single flattened USD file to a reference-based export, where each asset is its own file referenced into the scene
- **SimReady Foundation conformance**: work through the remaining issues listed in the validation section above

---

## Status

**Work in progress.** Core procedural warehouse generation and asset library complete and validated in Isaac Sim. SimReady Foundation conformance, environment texturing, additional asset types, and sensor data integration in active development.
