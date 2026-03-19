# Stitching

Per-document tile stitching pipeline.

ORB/AKAZE feature detection at downscale → RANSAC homography → full-resolution warp → multiband blend.

Handles both positive and negative fiche polarities. Supports seeding initial transforms from known grid geometry.
