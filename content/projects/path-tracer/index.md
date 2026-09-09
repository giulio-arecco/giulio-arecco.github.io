---
title: "Zig Path Tracer"
date: 2026-06-10
summary: "A CPU-based Monte Carlo path tracer written in Zig, featuring multi-bounce global illumination, BVH acceleration, and custom spatial denoisers."
tags: ["Zig", "Computer Graphics"]
featured: true
---

{{< button href="https://github.com/giulio-arecco/zig-path-tracer" target="_blank" >}}
  {{< icon "github" >}} View Source Code
{{< /button >}}

This project is a CPU-based Monte Carlo path tracer built entirely in Zig. Inspired by the [*Ray Tracing in One Weekend* series](https://raytracing.github.io/), encompassing the vast majority of concepts from the first two books, the codebase deliberately shifts away from traditional C++ object-oriented patterns. Instead, it was developed from the ground up to explore the Zig programming language and implement a multi-bounce global illumination engine from scratch without relying on external dependencies.

## Tech Stack

*   **Language:** Zig `0.16.0-dev`
*   **External Dependencies:** None (Standard Library only)

## Architectural Overview

The engine is built as a complete pipeline divided into the following sub-systems. For a comprehensive breakdown of the software architecture, please refer to the [repository's README](https://github.com/giulio-arecco/zig-path-tracer).

*   Application Configuration
*   Core Utilities and Mathematics
*   Camera, Rays, and Color
*   Scene and Materials
*   Geometry and Spatial Acceleration
*   Rendering
*   Frame Buffers and G-Buffers
*   Post-Processing and Denoising
*   Display and Image Output
*   Testing and Resource Management

## Engineering Highlights

### Zero-Cost Polymorphism via Comptime
To avoid the runtime overhead of virtual function calls typical of C++ inheritance, the rendering backend utilizes a Strategy Pattern implemented through tagged unions. By leveraging Zig's `inline else` prongs within the rendering pipepline orchestrator (`Renderer.render`), backend dispatching is resolved entirely at compile-time. Compile-time metaprogramming is also utilized in the I/O pipeline, specifically in evaluating PPM header lengths to bypass dynamic heap allocations during string formatting, as well as in many mathematical utility functions.

### Explicit Memory Ownership and State Isolation
The architecture strictly enforces data-oriented contexts. Persistent runtime configurations are decoupled from both the execution state and the user-dependent configurations they-re derived from. Memory buffers are injected into core algorithms as transient data structures, ensuring the renderers remain stateless and easily testable.

Memory management is handled explicitly via `std.mem.Allocator`. The `Scene` struct acts as the sole owner of all dynamically allocated entities on the heap, ensuring deterministic cleanup. Geometrical primitives hold only non-owning `*const` pointers to their materials, eliminating lifetime ambiguity and double-free vulnerabilities.

### Selective Denoising
The post-processing pipeline mitigates Monte Carlo noise at low sample counts using custom Joint Bilateral and À-Trous spatial filters. To guarantee memory safety during complex filtering operations, denoisers receive G-Buffer data through a dedicated read-only view. This leverages compile-time type coercion (`[]T` to `[]const T`) to prevent accidental state corruption.

Furthermore, the path tracer separates light transport into diffuse, specular, and emission components. This separation enables the engine to selectively bypass spatial denoising for perfectly glossy materials, preserving sharp, mirror-like reflections that would otherwise be incorrectly blurred.

## Gallery

### Denoising Results

{{< gallery >}}
  <img src="img/path-tracer/CornellBox-LowSamples.jpg" alt="50 Samples (No Filter)" class="grid-w33" />
  <img src="img/path-tracer/CornellBox-LowSamples-JointBilateral.jpg" alt="50 Samples (Joint Bilateral)" class="grid-w33" />
  <img src="img/path-tracer/CornellBox-LowSamples-ATrous.jpg" alt="50 Samples (A-Trous)" class="grid-w33" />
{{< /gallery >}}

### Other Renders
{{< gallery >}}
  <img src="img/path-tracer/CornellBox.jpg" alt="Cornell Box Render" class="grid-w33" />
  <img src="img/path-tracer/Final.jpg" alt="Final Scene Render" class="grid-w33" />
  <img src="img/path-tracer/ProceduralSpheres.jpg" alt="Spheres Scene Render" class="grid-w33" />
{{< /gallery >}}

## Key Takeaways
*   **Zig's Explicit Philosophy:** Prioritizing explicit control flow and zero hidden allocations proved highly effective in preventing code obfuscation as architectural complexity grew. Enforcing the explicit `std.mem.Allocator` pattern eliminated unexpected side effects, while leveraging the updated I/O and allocation interfaces in Zig 0.16 facilitated a modular and maintainable codebase.
*   **Engineering From Scratch:** Deliberately avoiding third-party dependencies required writing custom functions and libraries to handle tasks like vector math and raw `.ppm` image serialization. This low-level approach consolidated my theoretical computer graphics knowledge, granting absolute control over every stage of the execution pipeline.
*   **Expanding the Scope:** Venturing beyond standard light transport to implement custom spatial denoising filters provided hands-on experience with image-space data manipulation and lightweight post-processing algorithms.
*   **Application Polish:** Designing a robust command-line interface to parse custom execution arguments transformed the project from a standalone algorithmic exercise into a fully configurable rendering application.
*   **Future Improvements:** Integrating advanced sampling strategies (such as those detailed in *[Ray Tracing: The Rest of Your Life](https://raytracing.github.io/books/RayTracingTheRestOfYourLife.html)*) would significantly strengthen the mathematical foundations of the path tracer.

