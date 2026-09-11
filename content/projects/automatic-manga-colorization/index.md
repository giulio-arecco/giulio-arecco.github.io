---
title: "Automatic Manga Colorization"
date: 2025-02-14
summary: "A deep learning framework in PyTorch for automatic manga and line art colorization, combining quantized color classification with adversarial training."
tags: ["Python", "Machine Learning"]
---

{{< katex >}}

<div class="flex flex-wrap gap-2 mb-6">
  {{< button href="https://github.com/LienoPC/Manga-Auto-Colorization" target="_blank" >}}
    {{< icon "github" >}} View Source Code
  {{< /button >}}

  {{< button href="/automatic_manga_colorization_report.pdf" target="_blank" >}}
    {{< icon "file-lines" >}} Read Full Report
  {{< /button >}}
</div>

This project implements an adversarial deep learning pipeline built in PyTorch for the automatic colorization of line art and black-and-white illustrations. Based on the methodology introduced in *[Colorful Image Colorization](https://richzhang.github.io/colorization/)* (Zhang et al.), the system pairs the convolutional generator with adversarial discriminators to counteract desaturation. For an in-depth analysis of the training dynamics, hyperparameter tuning, and comparative metrics (PSNR, SSIM), refer to the [full technical report](/automatic_manga_colorization_report.pdf).

**Project Overview:**
* **Role:** Machine Learning Engineer (Team of 2).
* **Context:** Computer Vision / Deep Learning.
* **Responsibilities:** Color space quantization, GPU soft-encoding, loss optimization, dataset preprocessing.

## Contribution Overview

* **CIELAB Gamut Quantization:** discretization of continuous chrominance space into 313 physically realizable color centroids using lattice filtering and k-means clustering.
* **Chunked Soft-Target Encoding:** a GPU-accelerated mapping pipeline converting continuous chrominance channels into probability distributions via Gaussian-weighted k-nearest neighbors.
* **Numerically Masked Multinomial Loss:** vectorized multinomial cross-entropy loss over continuous probability distributions, stabilized with log-domain zero masking.
* **Dataset Sanitization and Preprocessing:** data validation routines filtering corrupted image headers, standardizing multi-channel formats, and flattening alpha transparency channels.

## Engineering Highlights

### Gamut Sampling and Color Space Discretization

Treating colorization as an \(L_1\) or \(L_2\) regression problem leads to desaturated, sepia-toned predictions because the loss minimizes expected error by predicting the mean of multimodal color distributions. To avoid this, the architecture adopts a classification formulation across discrete color bins in the CIELAB color space (\(L^*a^*b^*\)). Because unconstrained Cartesian sampling across the \(a\) and \(b\) axes generates chromatic coordinates outside the displayable sRGB spectrum, candidate pairs are concatenated with a reference luminance (\(L^* = 50\)) and projected into sRGB to discard non-physical values. The valid in-gamut points are clustered via k-means down to \(Q = 313\) centroids, producing an empirical color lattice that preserves natural density across high-saturation regions.

### Memory-Bound Soft-Target Encoding

Assigning continuous \(ab\) channels to discrete bins using 1-hot encoding produces severe quantization boundaries and high gradient variance. The target pipeline instead uses a soft-encoding mapping (\(H^{-1}\)) that distributes probability mass across the \(k = 5\) nearest color centroids weighted through a continuous Gaussian kernel. In a standard training batch of 32 images downsampled to \(64 \times 64\), computing pairwise distances for \(131,072\) pixels simultaneously exceeds available GPU memory. To prevent out-of-memory faults, the target encoding runs in fixed chunk partitions. For each slice, pairwise Euclidean distances against all 313 centroids are computed, converted to Gaussian proximity weights, normalized across the probability simplex, and accumulated directly in VRAM.

### Numerically Masked Multinomial Loss

Standard cross-entropy implementations expect discrete class indices, whereas the soft-target colorization objective optimizes cross-entropy over continuous probability distributions:

$$L_{cl}(Z, \widehat{Z}) = - \sum_{h,w} \sum_{q=1}^{Q} Z_{h,w,q} \log(\widehat{Z}_{h,w,q})$$

Here, \(Z\) represents the soft-encoded target distribution and \(\widehat{Z}\) denotes the predicted class probabilities from the network's classification layer. Because manga panels are dominated by neutral backgrounds and text, standard class rebalancing was excluded, relying instead on adversarial training and auxiliary pixel loss to enhance color saturation. Evaluating \(\log(\widehat{Z})\) risks divergence when predicted probabilities approach zero, yielding \(-\infty\) or `NaN` values that corrupt backpropagation gradients. To maintain numerical stability, the calculation isolates strictly positive probabilities with a boolean mask before computing the logarithm, evaluating the reduction via a single vectorized tensor product.

### Dataset Sanitization and Preprocessing

Raw manga scans collected from public datasets present substantial variations in format, including corrupted file headers, 1-channel bilevel line art, and 4-channel RGBA scans. The sanitization process validates image metadata at the header level during dataset indexing, skipping corrupted or incompatible files without reading entire pixel arrays into host memory. An alpha-detection preprocessor converts RGBA scans to 3-channel RGB representations prior to spatial resizing, preventing tensor dimension mismatches during batch collation. Additionally, CPU execution limits were configured to restrict OpenMP worker allocation between Scikit-learn and PyTorch DataLoader workers, preventing thread contention and memory leaks during data preparation.
