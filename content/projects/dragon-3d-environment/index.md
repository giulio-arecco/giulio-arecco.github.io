---
title: "Dragòn - 3D Environment"
date: 2024-02-25
summary: "A 3D cathedral environment modeled in Blender and rendered in Cycles, featuring procedural geometry nodes, custom props, and physically-based materials."
tags: ["Computer Graphics", "3D Modeling", "Animation",]
---

This project is a 3D environment set inside a gothic cathedral, created in Blender and rendered using the Cycles engine. Developed as complementary visual art for the [Dragòn - 2D OpenGL Arcade Game](projects/opengl-dragon-game) project, the scene showcases fantasy props, procedural asset modeling, and atmospheric lighting.

**Project Overview:**

* **Role:** 3D Artist (Team of 3).
* **Context:** 3D Environment Art / Offline Rendering.
* **Tools Used:** Blender, Cycles Render Engine.
* **Responsibilities:** Procedural modeling via Geometry Nodes, prop modeling, procedural shading, scene lighting.

## Contribution Overview

* **Dragon Egg (Geometry Nodes):** procedural distribution, scaling, and alignment of overlapping scales across the egg surface using a custom Geometry Nodes graph.
* **Prop Modeling:** 3D modeling of the floor wrought-iron candelabras, altar candleholders, wax candles, and the carved dragon fang resting on the ceremonial pillow.
* **Procedural Materials:** node-based procedural shading, including subsurface scattering for the translucent candle wax, metallic surfaces for candleholders and candelabras, and organic texturing for the dragon fang and egg.
* **Lighting Setup:** multi-source illumination setup balancing warm local point lights at candle wicks and the glowing lava contained in the dragon vessel with cool, low-intensity ambient light streaming from the cathedral windows.

## Gallery

{{< gallery >}}
  <img src="img/dragon-3d-environment/dragon_env_vertical.png" alt="Dragòn 3D Environment - Render Screenshot" class="grid-w75" />
  <img src="img/dragon-3d-environment/dragon_env_panoramic.png" alt="Dragòn 3D Environment - Render Screenshot" class="grid-w75" />
{{< /gallery >}}
