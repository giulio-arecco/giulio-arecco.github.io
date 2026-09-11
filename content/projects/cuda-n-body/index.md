---
title: "CUDA N-Body Simulation"
date: 2025-02-12
summary: "A CUDA-accelerated gravitational N-Body Simulation based on the all-pairs approach. Features optimized kernels utilizing shared memory and loop unrolling, a benchmark suite, and OpenGL real-time visualization."
tags: ["C++", "Parallel Computing", "Systems Programming"]
---

{{< katex >}}

<div class="flex flex-wrap gap-2 mb-6">
  {{< button href="https://github.com/LienoPC/N-BodySimulation" target="_blank" >}}
    {{< icon "github" >}} View Source Code
  {{< /button >}}
</div>

This project implements a GPU-accelerated simulation of an N-body system utilizing the all-pairs approach. Developed collaboratively, the application computes gravitational interactions using NVIDIA CUDA, supplemented by real-time visualization and performance benchmarking.

**Project Overview:**
* **Role:** Software Engineer (Team of 2).
* **Context:** Parallel Computing / GPU Programming.
* **Responsibilities:** Shared memory optimizations, numerical validation harness, and hardware diagnostic utilities.

## Contribution Overview

* **Shared Memory Tiling:** parametric tiling that decouples tile width from thread block size, combined with register-level state caching to minimize global memory traffic.
* **CPU Validation:** a multi-threaded OpenMP reference implementation providing lockstep verification and adaptive tolerance analysis for floating-point calculations.
* **Hardware Diagnostics:** runtime GPU introspection that queries streaming multiprocessor limits to perform pre-flight validation on kernel launch parameters.

## Engineering Highlights

### Parametric Tiling and Register Caching
Evaluating all-pairs gravitational forces requires \(O(N^2)\) interactions, which quickly saturates memory bandwidth if threads query global memory directly. To optimize data reuse, a parametric shared memory tiling system was implemented. By decoupling tile width from the thread block dimension, the kernel stages larger contiguous chunks of particle data into fast on-chip shared memory while maintaining coalesced global reads. In addition, particle positions and velocities are cached directly in thread-local registers during the numerical integration step. This confines intermediate Leapfrog updates to registers, reducing global memory traffic to a single read at kernel launch and a single write upon completion.

### Numerical Validation and Adaptive Tolerance
Because floating-point addition is non-associative, varying warp execution orders introduce numerical deviations that make it difficult to distinguish expected rounding errors from actual concurrency bugs. To verify algorithmic correctness, a deterministic multi-threaded CPU reference implementation was developed using OpenMP. Both implementations include a Plummer softening factor (\(\epsilon^2\)) to prevent numerical divergence when particle distances approach zero. A testing harness runs the CPU and GPU simulations in lockstep, using page-locked host memory (`cudaMallocHost`) for fast DMA transfers. Rather than relying on fixed epsilon thresholds, which fail across large coordinate spans, the validator evaluates discrepancies using an adaptive relative tolerance, preventing false positives while catching true mathematical divergence.

### Hardware Diagnostics and Launch Validation
Arbitrary thread block configurations and shared memory allocations risk exceeding device limits, causing runtime launch failures (`cudaErrorLaunchOutOfResources`). To prevent this, a diagnostic module queries the GPU's hardware properties (including register availability, shared memory capacity per SM, and maximum grid dimensions) via the CUDA Runtime API. A pre-flight validation check evaluates kernel requirements against these physical limits before launch, throwing explicit exceptions if resource budgets are exceeded. Additionally, the execution pipeline separates computational benchmarking from display rendering, enabling headless execution to measure raw kernel throughput without display synchronization constraints.
