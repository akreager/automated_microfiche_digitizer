# Microfiche Digitizer

A custom 3D printer-based system for automated microfiche digitization.

## Components

- **capture-node** â€” Lightweight Python agent running on RPi 3B+ nodes. Handles motion control, tile capture, local staging, and batch transfer to the processing server.
- **processing-server** â€” FastAPI + SQLite server running on the Dell Optiplex. Handles ingest, image processing pipeline, fiche database, and web dashboard.

## Documentation

See [docs/README.md](docs/README.md) for the full document register.

## Hardware

| Role | Hardware |
|---|---|
| Capture Node (Ã—2) | Raspberry Pi 3B+ |
| Processing Server | Dell Optiplex MFF, i5 9th gen, 16 GB DDR4 |
| Camera | Raspberry Pi HQ Camera Module (IMX477, 12.3 MP) |
