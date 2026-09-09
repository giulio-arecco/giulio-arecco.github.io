---
title: "Project Mecha-Ball"
date: 2025-07-14
summary: "A 3D puzzle game developed in Unity, centered on spatial problem-solving."
tags: ["Unity", "C#", "Game Development"]
---

<div class="flex flex-wrap gap-2 mb-6">
  {{< button href="https://lienopc.itch.io/project-mecha-ball" target="_blank" >}}
    {{< icon "itch-io" >}} Play on Itch.io
  {{< /button >}}
</div>

*Project Mecha-Ball* is a 3D tactical puzzle game built in Unity, designed around simulated AI training scenarios and spatial problem-solving. Developed as a collaborative group project for the "Game Design" exam at Politecnico di Torino.

**Project Overview:**
*   **Role:** Game Programmer, Technical Game Designer (Team of 6).
*   **Context:** Collaborative Game Design and Development Project.
*   **Responsibilities:** Game concept and design, execution architecture, custom tooling, serialization improvements, teleport implementation, platforms and buttons mechanics.

## Contribution Overview

*   **Execution Architecture:** a custom polling system utilizing the Observer pattern to manage the execution order of game state updates reliably.
*   **Trigger Optimization:** procedural bounding-box generation tools and zero-allocation overlap querying systems to bypass the standard need for `Rigidbody` components on kinematic triggers.
*   **Teleportation Infrastructure:** teleportation logic featuring vector-preserving momentum calculations and object caching to prevent infinite teleport cycles.
*   **Custom Editor Tooling:** reflection-driven property drawers and polymorphic lists serialization allowing level designers to construct puzzle interactions directly within the Unity Inspector.

## Engineering Highlights

### Custom Update Loop
Relying heavily on distributed native update methods can make it challenging to maintain a predictable execution order. To ensure determinism, the architecture implements centralized update managers utilizing the Observer pattern. Active objects register to pending buffers to safely handle state changes mid-frame. The execution array is dynamically sorted by priority and iterated backwards, naturally accommodating real-time observer unsubscription.

### Triggers Optimization
Standard Unity trigger callbacks typically require at least one interacting object to possess a `Rigidbody` component, which did not align with the needs of our specific kinematic platforming mechanics. To accommodate this, custom bounding boxes (acting like trigger colliders) are calculated to automatically extrude and snap to a visual mesh's top vertex, adapting dynamically as level designers scale objects in the Editor. To maintain performance, these custom triggers manage physics interactions using pre-allocated buffers. Collisions are evaluated using `Physics.OverlapBoxNonAlloc()`, preceded by a `Physics.SyncTransforms()` call to ensure accurate spatial queries without overlap latency on high-velocity platforms.

### Teleporters Implementation
Puzzle mechanics required teleportation systems to preserve momentum accurately while preventing actors from getting trapped in infinite interaction loops. Upon entering the teleporter trigger, the player's velocity is cached, and the exact output trajectory is calculated using `Quaternion.FromToRotation`. This mirrors the object's inertia relative to the receiving portal's spatial orientation. To manage the teleportation state securely, a caching structure temporarily flags the actor on the receiving end, enforcing a cooldown loop that breaks the transfer cycle until the entity fully exits the teleporter trigger.

### Editor Serialization Tools
To decouple puzzle triggers (such as buttons) from receiver logic (such as platforms), the system relies on an interface-driven bridge. To empower level designers to configure these connections directly in the Unity Inspector, a custom Property Drawer was developed. By utilizing C# reflection, the tool dynamically discovers scripts inheriting from a base effect class, and then generates polymorphic concrete classes on the fly within the Inspector. This allows designers to assign diverse visual and logical effects without writing new code.

## Gallery

{{< gallery >}}
  <img src="img/project-mecha-ball/project-mecha-ball-1.jpg" alt="Project Mecha Ball - Example Screenshot" class="grid-w50" />
  <img src="img/project-mecha-ball/project-mecha-ball-2.jpg" alt="Project Mecha Ball - Example Screenshot" class="grid-w50" />
  <img src="img/project-mecha-ball/project-mecha-ball-3.jpg" alt="Project Mecha Ball - Example Screenshot" class="grid-w50" />
  <img src="img/project-mecha-ball/project-mecha-ball-4.jpg" alt="Project Mecha Ball - Example Screenshot" class="grid-w50" />
{{< /gallery >}}
