# Pipeline

Image processing pipeline. Designed to be independently testable without standing up the web application.

## Stages

1. **Normalization** — CLAHE contrast enhancement, polarity detection and inversion, flat-field correction
2. **Segmentation** — Hough line detection, document grid extraction, per-document tile manifest generation
3. **Stitching** — ORB/AKAZE feature detection at downscale, RANSAC homography, full-resolution warp, multiband blending
