# Staging

Local tile staging, batch transfer, and lifecycle management.

## Staging Directory Layout

```
/home/pi/capture-staging/
└── FICHE-YYYYMMDD-NNN/
    ├── meta.json       # Fiche metadata, grid layout, capture parameters
    ├── tile_00_00.png
    ├── tile_00_01.png
    └── ...
```

A staging directory is created at the start of each scan. On successful capture and confirmed server acknowledgment, the directory is deleted. On failure, it remains for operator inspection and retry.
