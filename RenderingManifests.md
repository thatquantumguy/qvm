## Client-Server Rendering Manifest (Conceptual Overview)

Published 25 Aug 2025.  
Updated 28 Aug 2025.  
Companion repository "https://github.com/thatquantumguy/qva"

### Context
In the system for which this approach was conceived, every type the server evaluates is a packed index into something else — whether that’s a function in an L1‑cache‑resident, work‑queue‑native kernel instance, or a rich volumetric texture in client VRAM.

Because of this, it is both natural and efficient to treat all server‑side state as an approximation of geometry, concepts, and attributes. The server’s role is not to replicate full‑fidelity world state, but to resolve indices into compact, meaningful references from which the client can construct a controlled view into it.

View‑volume‑driven occlusion culling, omnidirectional perspective projection, and moreover the metadata derived from these operations are compiled into rendering manifests. These manifests act as highly concise scene descriptions, allowing client‑side asset references and shader parameters to be resolved locally.

---

### Server‑Side Process (In Practice)

1. **Occlusion / Selection** – Determine which entities, surfaces, or volumetric regions are relevant to the current observer context.  
2. **Perspective Projection** – Transform those selections into a normalized cube‑map — a discrete representation of a spherical probe — producing bounded, observer‑centric viewpoints.
3. **Metadata Derivation** – Extract volumetric material subscripts, server‑side environmental lighting data, viewpoint relative per‑face normals, and the underlying geometric topology of the to‑be‑rendered scene.  
4. **Packaging** – Compile, compress, and package this data into a network‑efficient form.  
5. **Networking** – Collate and batch transmit render manifests for all clients at a consistent rate, preferring UDP where possible.

---

### Client‑Side Rendering

From the client’s perspective, it receives regular 360‑degree views into world space from the server. Between each next/last pair of such reference points in space/time, it is possible to derive intermediate reference points with the same qualities.  
> **Note:** This interpolation does not work with frustum‑culled viewpoints, since the missing peripheral data prevents accurate reconstruction.

---

### Client-Side Process (In Practice)

1. **Interpolation Authority** – Maintain authority over interpolation between active viewpoint pairs, ensuring smooth motion and responsiveness without waiting for server updates.  
2. **Local Clipping Authority** – Determine clipping volume for rendering, allowing efficient culling of non-visible viewpoint metadata in the current view.  
3. **Local Orientation Control** – Manage pitch, yaw, and roll locally for immediate camera response.  
4. **State Relay to Server** – Send locally determined values (clipping, orientation, interpolation state) to a predictive server-side handler for integration into future manifest generation.  
5. **No Simulation Authority** – While controlling presentation and interpolation, defer simulation truth to the server to ensure consistency across all connected clients.

## Division of Responsibilities

The following table outlines the trust boundary and workflow between server and client components.  
It’s designed for clarity at a glance, ensuring each side’s authority and duties are unambiguous.

| **Server Responsibilities** | **Client Responsibilities** |
|------------------------------|------------------------------|
| Maintain authoritative simulation state in approximated world space. | Maintain authoritative interpolation between active viewpoint pairs. |
| Perform view‑volume‑driven occlusion culling and radial perspective projection. | Apply local clipping authority to cull non‑visible viewpoint metadata within the current view. |
| Derive metadata (material subscripts, lighting data, normals, topology) from selected geometry. | Control local pitch, yaw, and roll for immediate camera responsiveness. |
| Collate, compress, and buffer rendering manifests into regular network‑efficient transmissions. | Resolve server‑provided indices into local assets and shader parameters. |
| Manage per-client synchronization runlevel per network health telemetry. | Relay local orientation, clipping, interpolation and other input state to the server’s predictive handler. |
| Remain authoritative over simulation truth and validate all client inputs. | Render the scene as metadata derived, CPU generated 2D topology without altering simulation truth. |

---

### Visualizing a Clipped Rendering Manifest

You can picture a clipped rendering manifest by starting with a photograph — or better yet, a painting — and then reducing it to the minimum viable metadata needed to drive shaders.

On the client:

- **Vertex shaders** ingest CPU‑generated 2D topologies that span screen space in a planar fashion — imagine a mosaic or the cracked surface of a sun‑baked desert, each face a distinct region of the picture’s surface.  
- The result is passed through an orthographic projection in an otherwise ordinary 2D graphics pipeline.  
- **Fragment shaders** color and shape this mosaic analogy into something more like a mural or oil painting.

This approach treats the manifest as a compact, metadata‑driven recipe — using little more client-server bandwidth than typical state replication methods — allowing the client to reconstruct the intended view using minimal bandwidth while preserving artistic and spatial fidelity.

---

## Natural Projection

Natural Projection is the client‑side realization method for rendering manifests, designed to preserve perceptual fidelity within a standard 2D pipeline.

It’s a spherical‑to‑planar, split‑pipeline approach that mirrors human vision — concentrating detail at the center while preserving an undistorted peripheral view.

Conventional 3D pipelines treat perspective as a radial projection of world geometry onto a flat near‑plane. This simplification often introduces distortion, especially at wide fields of view.

Natural Projection instead projects the scene radially onto the inner surface of a sphere (the spherical clipping volume) using viewpoint‑relative (w, x, y, z) coordinates. These spherical viewpoints inherently encode panoramic environment light. The resulting cube‑map image is clipped client‑side with a hemispherical viewing volume, producing a mosaic‑like square 2D mesh topology whose faces map directly to shader invocations. The result: a perceptually correct 180‑degree field of view.

Each mesh face stores viewpoint‑relative normal vectors, enabling accurate lighting and shading in the 2D pipeline using captured server‑side lighting metadata. The hemispherical projection volume matches the natural curvature of human vision and camera lenses, so when warped to a square, lens distortion is corrected and render time isn’t wasted on pixels that would be discarded.

This rendering‑manifest‑derived, CPU‑generated mesh topology — defining groupings of shader invocations — then passes through an otherwise standard 2D graphics pipeline, resolved with an orthographic projection. The output image concentrates detail at the center, as in human vision, while the periphery remains stable, undistorted, and lit with the full richness of the captured environment.

---

## Usage / Intended Usage

This client–server rendering manifest system was originally developed as part of an **in‑development game platform**, designed to support high‑fidelity, low‑latency world simulation across distributed clients.  
Its architecture emerged from the need to:

- Keep the **server authoritative** over simulation truth while minimizing bandwidth by transmitting compact, metadata-centric rendering manifests instead of full‑fidelity world state.
- Allow **clients** to render scenes locally, interpolate between server‑provided viewpoints, and maintain responsive presentation without compromising consistency.
- Support **scalable multiplayer environments** where view‑volume‑driven culling and metadata-only rendering manifests ensure efficient network usage.

## Integration with QVM/QVA

This model is now included in the /qvm/ and /qva/ repositories as a default client–server reference implementation.  
It serves as:

- A **baseline networking and rendering pattern** for QVA/QVM‑powered simulations and games.
- A **template** for developers to adapt or extend for their own distributed virtual worlds.
- A **demonstration** of how QVM can integrate with real‑time rendering and asset resolution.

By shipping this system as part these repos, it provides a canonical example of QVA/QVM’s intended use in interactive, networked environments.

---

## Rendering Module: Natural Projection (NP)

**Status:** Under evaluation — not integrated in this branch.  
**Notes:** Reserved for potential future rendering paradigm.  
Full description, implementation details, and licensing to be added upon validation.

## Terminology Note

The acronyms **QVM** (“quantum virtual machine”) and **QVA** (“quantum virtual architecture”) are used here as generic descriptors for abstract concepts in quantum computing.  
They are not intended to refer to any specific commercial product, implementation, or trademark.

---

© 2025 thatquantumguy.  This document and all contents of this repository are licensed under the Apache License, Version 2.0.  See the [LICENSE](./LICENSE) file for full terms.
