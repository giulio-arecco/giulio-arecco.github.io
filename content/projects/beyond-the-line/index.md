---
title: "Beyond The Line"
date: 2026-02-21
summary: "A text-based serious game developed as a Master's thesis to explore Complex Problem Solving through resource management, narrative branching, and systemic uncertainty."
tags: ["Unity", "C#", "Game Development"]
featured: true
---

<div class="flex flex-wrap gap-2 mb-6">
  {{< button href="https://github.com/giulio-arecco/beyond-the-line" target="_blank" >}}
    {{< icon "github" >}} View Source Code
  {{< /button >}}

  {{< button href="https://giulio-arecco.itch.io/beyond-the-line" target="_blank" >}}
    {{< icon "itch-io" >}} Play on Itch.io
  {{< /button >}}

  {{< button href="/Masters_Thesis_Giulio_Arecco.pdf" target="_blank" >}}
    {{< icon "file-lines" >}} Thesis Document
  {{< /button >}}
</div>

*Beyond The Line* is a text-based interactive survival drama developed in Unity. It was designed as a serious game for my Master's Thesis to evaluate if and how serious games have the ability to foster Complex Problem Solving cognitive skills through interactive narrative frameworks and targeted information management strategies.

For a comprehensive breakdown of the design process, the theoretical research foundations (including the [CPS-GFC framework](https://www.sciencedirect.com/science/article/pii/S245195882500226X?via%3Dihub)), and the empirical evaluation methodology, please refer to the [thesis document](/Masters_Thesis_Giulio_Arecco.pdf).

## Tech Stack

*   **Engine:** Unity
*   **Language:** C#
*   **Narrative Scripting Language:** Ink

## Architectural Overview

The project is built around a decoupled, event-driven architecture structured into the following functional areas:

*   **Core Engine & Execution:** custom update managers and reflection-powered global stat tracking.
*   **Narrative Interoperability:** centralized story orchestration, narrative variable registries, and Ink-to-C# function binding.
*   **Entity & Storage Systems:** immutable `ScriptableObject` databases and generic `IStorage<T>` event-driven collections.
*   **UI Architecture:** stack-based layer navigation (`UINavigator`) and decoupled MVC view controllers.
*   **Auxiliary Utilities:** generic global state managers (`Singleton<T>`, `PersistentSingleton<T>`, `RegulatedSingleton<T>`) and custom interface serialization wrappers (`InterfaceReference<T>`).
*   **Telemetry Pipeline:** zero-allocation behavioral tracking arrays and unified JSON stat exporters.

## Engineering Highlights

### Unity and Ink Interoperability
To translate narrative choices into systemic gameplay consequences, the architecture implements a robust interoperability layer between Ink and Unity. A centralized `StoryManager` orchestrates text parsing and UI instantiation. To maintain global state persistence across branching paths, a `StoryVariablesRegistry` injects and extracts narrative variables into the Ink runtime. On top of that, a `StoryFunctionsBinder` utilizes reflection and type casting to safely execute C# backend logic triggered directly by Ink's `EXTERNAL` functions. This interoperability layer also leverages `ScriptableObjects` as centralized databases, allowing narrative scripts to directly fetch game assets (such as inventory items, companions or audio tracks) and update the global game state at the most appropriate narrative moments.

### Generic Storage and Type-Agnostic UI
The inventory and companion systems rely on a generic `IStorage<T>` interface. Under the hood, this architecture leverages C# generics to guarantee data integrity, preventing the creation of collections with heterogeneous concrete types. Simultaneously, it exposes a type-agnostic interface (`IStorage`) to the presentation layer; the UI can then dynamically render the collection contents without requiring any knowledge of the underlying data types.

### Layer-Based UI and MVC Pattern
The user interface is engineered using a Model-View-Controller (MVC) pattern to strictly isolate raw logic from its visual representation. Decoupled view controllers respond passively to backend C# events, dynamically repopulating panels without continuous polling. Navigation is governed by a custom stack-based `UINavigator` that manages overlapping layers.

### Custom Auxiliary Tooling
To overcome native engine limitations and streamline the development of complex systems, the architecture relies on a suite of custom foundational utilities. Because Unity does not natively support the serialization of C# interfaces, a custom `InterfaceReference<T>` wrapper was engineered to allow the assignment of scripts that implement a specific interface directly within the Inspector. Additionally, a set of generic base classes (`Singleton<T>`, `PersistentSingleton<T>`, and `RegulatedSingleton<T>`) was developed to expedite the creation of global managers while providing granular control over their instantiation and lifecycles across scene transitions.

### Unified Telemetry Infrastructure
To support the project's research objectives, the game implements a silent telemetry system designed to track player behavior without impacting performance. It unifies narrative outcomes captured from Ink's global variables with gameplay metrics tracked by the Unity engine. These C# `RuntimeStats` utilize low-overhead static arrays indexed by strongly typed enumerations, ensuring constant *O(1)* access time and no dynamic memory allocations. At the end of a session, a dedicated exporter aggregates these synchronized states into a unified JSON dataset for empirical analysis.

## Gallery

{{< gallery >}}
  <img src="img/beyond-the-line/oil_map.jpg" alt="Beyond The Line - Map" class="grid-w50" />
  <img src="img/beyond-the-line/oil_inventory.jpg" alt="Beyond The Line - Inventory Panel" class="grid-w50" />
  <img src="img/beyond-the-line/oil_mainmenu.jpg" alt="Beyond The Line - Main Menu" class="grid-w33" />
  <img src="img/beyond-the-line/oil_lirainvestigation.jpg" alt="Beyond The Line - Narrative" class="grid-w33" />
  <img src="img/beyond-the-line/oil_companions.jpg" alt="Beyond The Line - Companions Panel" class="grid-w33" />
{{< /gallery >}}
