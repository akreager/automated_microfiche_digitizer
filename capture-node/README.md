# Capture Node

Lightweight Python agent software for the Raspberry Pi 3B+ capture nodes.

## Design Philosophy

The capture node is intentionally thin. It does not run a web server or any user-facing interface. All serving, processing, and operator interaction is handled by the processing server.

The agent is responsible for:
- Receiving job instructions from the processing server
- Sending G-code over serial to the printer motion controller
- Capturing tiles via `raspistill` / `rpicam-still`
- Staging a complete fiche capture locally before transfer
- Transferring the complete batch to the processing server on success
- Sending periodic heartbeat and status to the server

A fiche dataset is only transferred when the full capture completes without error. Partial captures remain in local staging and are never sent to the server.

## Hardware

- Raspberry Pi 3B+
- Raspberry Pi HQ Camera Module (IMX477) via CSI
- USB serial connection to 3D printer motion controller
