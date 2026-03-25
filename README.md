# DDPM_MRI
A practical tutorial demonstrating score-based diffusion models (DDPM) for accelerated MRI reconstruction — with k-space data conditioning and optional data consistency (DC) enforcement, applied to mask undersampling on the CMRxRecon dataset.


# MRI Reconstruction with Diffusion Models

A hands-on workshop notebook demonstrating **accelerated MRI reconstruction** using a Denoising Diffusion Probabilistic Model (DDPM). Notebook used in the context of the IDRE-ECR workshop: https://idre.ucla.edu/calendar-event/physic-informed-diffusion-models-for-medical-imaging 

> Author: Thomas Coudert
> 
> Collaborators: Amit Rand, Zhengyang Ming  
> IDRE Workshop

---

## Overview

In clinical MRI, scan time is precious. Acquiring a fully-sampled k-space can take minutes — time patients may not be able to hold still, and time that limits scanner throughput. A common workaround is **undersampling**: acquire only a fraction of k-space measurements, then use a reconstruction algorithm to recover the full image.

This notebook shows how a **conditioned DDPM** trained on the [CMRxRecon](https://cmrxrecon.github.io/) cardiac MRI dataset can reconstruct high-quality images from undersampled k-space, with optional **hard k-space data consistency (DC)** enforcement.

---
## Workflow
```
Fully-sampled k-space from val set 
       │
       │  ← apply undersampling mask  (Cartesian / Radial / Poisson)
       ▼
Undersampled k-space  →  Zero-filled reconstructed image  (aliased, blurry)
       │
       │  ← reverse DDPM sampling (500 steps) with/w.o Data Consistency
       ▼
DDPM Reconstruction  VS  Ground truth
```
---

## Key Concepts

| Concept | Description |
|---|---|
| **Forward diffusion** | Adds Gaussian noise to the missing k-space region over T steps |
| **Reverse diffusion** | Iteratively denoises from pure Gaussian noise using a UNet that predicts $\hat{\varepsilon}$ |
| **Data conditioning** | The observed (undersampled) image is concatenated as UNet input channels at every step — the model cannot hallucinate away from acquired data |
| **Hard Data Consistency (DC)** | At each reverse step: FFT the current estimate, replace sampled lines with the measured k-space, then IFFT back — enforces strict fidelity to raw measurements |
| **Mask choice** | Cartesian, Radial, and Poisson patterns produce different aliasing; incoherent patterns (Poisson, Radial) are generally easier for the model to remove |

---

## Notebook Contents

| Section | Description |
|---|---|
| 0 — Imports & Paths | Setup |
| 1 — Select Subject | Pick a validation subject from CMRxRecon |
| 2 — Visualise Ground Truth | RSS coil combination, log-scale k-space display |
| 3 — Undersampling Strategy | Choose Cartesian / Radial / Poisson mask |
| 4 — Load Pretrained DDPM | Load model weights |
| 5 — Noise Schedule | Visualise the cosine beta schedule |
| 6 — Run Reconstruction | Full reverse diffusion chain (`dc_mode='none'` or `'hard'`) |
| 7 — Results | Side-by-side comparison: GT / zero-filled / DDPM |
| 7b — Hard DC | Enforce k-space data consistency at every denoising step |
| 8 — Denoising Trajectory | Watch the image emerge from noise (workshop demo) |
| 10 — Summary | Take-home messages |

---

## Requirements

- Python 3.8+
- PyTorch
- [CMRxRecon](https://cmrxrecon.github.io/) validation data correctely preprocessed or slice example (provided here)
- Pretrained DDPM checkpoint (provided separately)

---

## Citation

If you use this work, please acknowledge this workshop material.
