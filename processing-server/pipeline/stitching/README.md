# Stitching

Per-document tile stitching pipeline.

ORB/AKAZE feature detection at downscale â†’ RANSAC homography â†’ full-resolution warp â†’ multiband blend.

Handles both positive and negative fiche polarities. Supports seeding initial transforms from known grid geometry.
