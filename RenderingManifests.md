## Client‑Server Rendering Manifest (Conceptual Overview)

### Context
In the system for which this approach was conceived, every type the server evaluates is a packed index into something else — whether that’s a function in an L1‑cache‑resident, work‑queue‑native kernel instance, or a rich volumetric texture in client VRAM.

Because of this, it is both natural and efficient to treat all server‑side state as an approximation of geometry, concepts, and attributes. The server’s role is not to replicate full‑fidelity world state, but to resolve indices into compact, meaningful references from which the client can construct a controlled view into it.

View‑volume‑driven occlusion culling, omnidirectional perspective projection, and moreover the metadata derived from these operations are compiled into rendering manifests. These manifests act as highly concise scene descriptions, allowing client‑side asset references and shader parameters to be resolved locally.

---

### Server‑Side Process (In Practice)

1. **Occlusion / Selection** – Determine which entities, surfaces, or volumetric regions are relevant to the current observer context.  
2. **Perspective Projection** – Transform those selections into a normalized cube‑map — a discrete representation of a spherical probe — producing bounded, observer‑centric viewpoints.
3. **Metadata Derivation** – Extract volumetric material subscripts, server‑sent environmental lighting data, viewpoint relative per‑face normals, and the underlying geometric topology of the to‑be‑rendered scene.  
4. **Packaging** – Compile, compress, and package this data into a network‑efficient form.  
5. **Networking** – Collate and batch transmit render manifests for all clients at a consistent rate, preferring UDP where possible.

---

### Client‑Side Rendering

From the client’s perspective, it receives regular 360‑degree views into world space from the server. Between each next/last pair of such reference points in space/time, it is possible to derive intermediate reference points with the same qualities.  
> **Note:** This interpolation does not work with frustum‑culled viewpoints, since the missing peripheral data prevents accurate reconstruction.

---

### Client‑Side Process (In Practice)

1. **Interpolation Authority** – Maintain authority over interpolation between active viewpoint pairs, ensuring smooth motion and responsiveness without waiting for server updates.  
2. **Local Clipping Authority** – Determine clipping planes for rendering, allowing efficient culling of non‑visible viewpoint metadata in the current view.  
3. **Local Orientation Control** – Manage pitch, yaw, and roll locally for immediate camera response.  
4. **State Relay to Server** – Send locally determined values (clipping, orientation, interpolation state) to a predictive server‑side handler for integration into future manifest generation.  
5. **No Simulation Authority** – While controlling presentation and interpolation, defer simulation truth to the server to ensure consistency across all connected clients.

## Division of Responsibilities

The following table outlines the trust boundary and workflow between server and client components.  
It’s designed for clarity at a glance, ensuring each side’s authority and duties are unambiguous.

| **Server Responsibilities** | **Client Responsibilities** |
|------------------------------|------------------------------|
| Maintain authoritative simulation state in approximated world space. | Maintain authoritative interpolation between active viewpoint pairs. |
| Perform view‑volume‑driven occlusion culling and omnidirectional perspective projection. | Apply local clipping authority to cull non‑visible viewpoint metadata within the current view. |
| Derive metadata (material subscripts, lighting data, normals, topology) from selected geometry. | Control local pitch, yaw, and roll for immediate camera responsiveness. |
| Compile, compress, and package rendering manifests into a network‑efficient form. | Resolve server‑provided indices into local assets and shader parameters. |
| Collate and batch transmit manifests at a consistent rate, preferring UDP where possible. | Relay local orientation, clipping, interpolation and other input state to the server’s predictive handler. |
| Remain authoritative over simulation truth and validate all client inputs. | Render the scene as metadata derived, CPU generated meshes without altering simulation truth. |

---

### Visualizing a Clipped Rendering Manifest

You can picture a **clipped** rendering manifest by starting with a photograph — or better yet, a painting — and then reducing it to the minimum viable metadata needed to drive shaders.

On the client:

- **Vertex shaders** ingest CPU‑generated 2D meshes that cover the client‑local world space in a planar fashion.  
  Think of it as a mural: a flat surface subdivided by targeted shader type, onto which the scene will be painted. 
- **Fragment shaders** then “paint” onto this mural using volumetric textures, with HDRI-like lighting.
- The result is passed through an orthographic projection in an otherwise ordinary 2D graphics pipeline.

This approach treats the manifest as a compact, metadata‑driven recipe — using little more client-server bandwidth than typical state replication methods — allowing the client to reconstruct the intended view with minimal bandwidth while preserving artistic and spatial fidelity.

---

## Natural Projection

Natural Projection is the client‑side realization method for rendering manifests, designed to preserve perceptual fidelity while using a standard 2D pipeline.

A spherical-to-planar split-pipeline method that mirrors human vision by concentrating detail at the center and preserving undistorted peripheral view, through a standard 2D client pipeline.

Conventional 3D graphics pipelines treat perspective as a radial projection of world geometry onto a flat plane — a simplification that introduces distortions, especially at wide fields of view.  

**Natural Projection** takes a different approach, one that mirrors how we actually see. From the viewpoint, the scene is projected radially onto a sphere's inner surface (the spherical clipping volume), using viewpoint-relative \((w, x, y, z)\) coordinates. The resulting spherical image is then clipped client‑side with an oval‑shaped “frustum”, producing a mural‑like rectangular 2D mesh whose faces map directly to shader invocations.  

This mesh passes through an otherwise standard 2D graphics pipeline, resolved with an orthographic projection. The outcome is an image where detail is naturally concentrated toward the center — just as in human vision — while the periphery remains stable and undistorted.

---

## Usage / Intended Usage

This client–server rendering manifest system was originally developed as part of an **in‑development game platform**, designed to support high‑fidelity, low‑latency world simulation across distributed clients.  
Its architecture emerged from the need to:

- Keep the **server authoritative** over simulation truth while minimizing bandwidth by transmitting compact, metadata-centric rendering manifests instead of full‑fidelity world state.
- Allow **clients** to render scenes locally, interpolate between server‑provided viewpoints, and maintain responsive presentation without compromising consistency.
- Support **scalable multiplayer environments** where view‑volume‑driven culling and metadata-only rendering manifests ensure efficient rendering pipelines.

### Integration with QVM

This model is now included in the (QVM) repository as the default client–server reference implementation.  
It serves as:

- A **baseline networking and rendering pattern** for QVM‑powered simulations and games.
- A **template** for developers to adapt or extend for their own distributed virtual worlds.
- A **demonstration** of how QVM can integrate with real‑time rendering and asset resolution.

By shipping this system as part of the QVM repo, it provides both a practical starting point for new projects and a canonical example of QVM’s intended use in interactive, networked environments.

---

## Terminology Note:

The term “QVM” is used here as a generic descriptor for a quantum virtual machine architecture.  
It is not intended to refer to any specific commercial product or trademark.  

---

© 2025 thatquantumguy. This document is licensed under the Apache License, Version 2.0. See the [LICENSE](./LICENSE) file for full terms.
