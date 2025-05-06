---
title: Final Report
---

<script>
// configure delimiters for kramdown’s $$…$$ and \(...\)
window.MathJax = {
  tex: {
    inlineMath: [
        ['\\(', '\\)']
        ],
    displayMath: [
        ['$$', '$$'],
        ['\\[', '\\]']
    ]
  },
  svg: { fontCache: 'global' }
};
</script>
<script src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js" crossorigin="anonymous"></script>

# [Home]({{site.baseurl}}/) / Final Report

## Introduction

I set out to enhance our ray tracer by adding a dynamic depth of field (DOF) feature controlled by a provided f-stop parameter. Traditional depth of field effects in ray tracers rely on fixed aperture settings; my goal was to implement a realistic, physically-based DOF where the aperture radius is derived from the f-stop and focal length, giving the user intuitive photographic control over focus and blur.

### Goal Scene

<img src="{{site.baseurl}}/assets/img/DOF_Goal.png" width="800"/>

## Architecture

The core of the system builds on the thin-lens model: rays originate from a disk representing the camera aperture and converge at a focal plane determined by the focal length. I derived the aperture radius via the relation:

$$ r = \frac{f}{2 \times \mathrm{f\#}} $$

where \\(f\\) is the focal length and f# is the f-stop. To integrate this into the existing pipeline, I:

1. Extended the `Camera` class to support a `_apertureRadius` member which determines the radius of the sample disk.
2. Implemented a uniform disk sampler to generate random points across the aperture for each primary ray. Multiple DOF rays can be shot per primary ray.
3. Computed the focal point on the focal plane for each pixel-aligned ray, then traced rays from randomized disk samples through this point.
5. Leveraged multithreading and supersampling to manage noise and performance trade-offs.

## System

My implementation is in C# and builds on our existing ray tracer:

* **Camera.SetAperature**: Calculates aperture radius.
* **Camera.SetSampling**: Adjusts the number of rays being fired out for either supersampling or DOF.
* **Camera.RandomInUnitDisk**: Samples the aperture disk.
* **Camera.SampleColor**: Generates a number of DOF rays (determined above, defaults to 1).
* **ObjParser.cs**: Parses OBJ and MTL files and applies image shaders to textured objects.
* **Program.cs**: Integrates DOF sampling into the main render loop, with options for multithreading and supersampling.

## Previous Results (mid-semester)

Below are example renders demonstrating the effect of different f-stops on a scene with foreground and background objects.

### f/22 (small aperture, large depth of field)

<img src="{{site.baseurl}}/assets/img/Scene_DOF_22_Z.png" width="800"/>

### f/2.8 (medium aperture)

<img src="{{site.baseurl}}/assets/img/Scene_DOF_2_8.png" width="800"/>

### f/1.4 (large aperture, shallow depth of field)

<img src="{{site.baseurl}}/assets/img/Scene_DOF_1_4_Z.png" width="800"/>

## Final Results (mid-semester)

Below are example renders demonstrating the effect of different f-stops on the goal scene.

### Reducing Aliasing through Supersampling

<img src="{{site.baseurl}}/assets/img/Piano_Alias.png" width="800"/>

### Final Result

<img src="{{site.baseurl}}/assets/img/Piano_DOF.png" width="800"/>



I also benchmarked performance on a 1.1M-triangle mesh (the piano) with varying samples per pixel:

| Samples per pixel | Single-thread time | Multi-thread time |
| ----------------- | ------------------ | ----------------- |
| 5                 | 4 m 48 s           | 1 m 37 s          |
| 25                | 29 m 19 s          | 8 m 53 s          |
| 50                | *DNF*              | 17 m 41 s         |

## Future Work

* **Variable Focal Lengths**: Allow dynamic focal lengths per scene or camera animation.
* **Adaptive Sampling**: Concentrate more DOF rays where blur gradients are steep.
* **GPU Acceleration**: Offload DOF sampling to GPU for real-time previews.
