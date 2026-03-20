# CR-MFDIG-01 — Camera Stack & Capture Node Hardware Compatibility

**Document type:** Compatibility Report
**Project:** MFDIG — Microfiche Digitizer
**Status:** In Progress
**Last updated:** 2026-03-16

---

## 1. Purpose

This document records the results of compatibility testing and evaluation for camera hardware and capture node software on the target platforms used in the microfiche digitizer project. It captures what works, what does not, the reasons why, and the resulting hardware/software decisions. This document should be updated as further testing is completed.

---

## 2. Scope

- Raspberry Pi HQ Camera Module (IMX477, 12.3 MP)
- Arducam IMX477 UVC Camera Adapter Board
- Raspberry Pi 3B+ as capture node
- Raspberry Pi 5 as capture node
- OctoPrint camera stack (webcamd / mjpg_streamer)
- Standalone app capture path (rpicam-still / raspistill)

---

## 3. Hardware Under Test

| Item | Description |
|---|---|
| Camera | Raspberry Pi HQ Camera Module, Sony IMX477, 12.3 MP |
| Adapter | Arducam IMX477 UVC Camera Adapter Board (CSI-to-USB) |
| Node A | Raspberry Pi 3B+ (1 GB RAM, USB 2.0, 100 Mbps Ethernet) |
| Node B | Raspberry Pi 3B+ (1 GB RAM, USB 2.0, 100 Mbps Ethernet) |
| (Reserved) | Raspberry Pi 5 — not used for this project; reserved for other work |
| Server | Dell Optiplex MFF, Intel Core i5 9th gen, 16 GB DDR4 |

---

## 4. Findings

### 4.1 Arducam IMX477 UVC Adapter Board

**Conclusion: Not suitable for tile capture. Acceptable for preview/alignment only.**

The adapter board presents the IMX477 sensor as a UVC-compliant USB device. This approach has an architectural limitation that makes it unsuitable for archival tile capture regardless of platform:

**Data format is MJPG only.** The board has no RAW, uncompressed YUV, or PNG output path. JPEG compression is applied inside the adapter firmware before the image is accessible to the host system. This means every captured frame has already been lossy-compressed before any host software can act on it. For archival microfiche digitization, where resolving fine document text at full sensor resolution is the primary requirement, this generation loss is unacceptable.

The MJPG-only constraint is not a driver or software problem — it is a physical bandwidth limitation of the USB 2.0 interface. A full-resolution uncompressed frame from the IMX477 at 4032×3040 would require approximately 3.5 GB/s; USB 2.0 can sustain roughly 40 MB/s in practice. The MJPG compression (~20:1 ratio) is the only way to fit the data through the interface.

**Adapter board specifications for reference:**

| Parameter | Value |
|---|---|
| Sensor | Sony IMX477, 12.3 MP |
| Data format | MJPG only |
| Max resolution | 4032×3040 @ 10 fps |
| Interface (host) | USB 2.0 |
| Interface (camera) | 22-pin 0.5 mm CSI |
| Driver requirement | None — UVC-compliant, standard OS drivers |

**Alternative use:** The adapter is fully functional as a focus/alignment preview tool on a laptop or desktop PC during digitizer setup. Standard V4L2 (Linux), DirectShow (Windows), or AVFoundation (macOS) drivers work without any additional software. It is not used in the automated capture pipeline.

---

### 4.2 Raspberry Pi 5 + OctoPrint Camera Stack

**Conclusion: Incompatible without significant workaround. OctoPrint approach abandoned.**

The Raspberry Pi 5 uses the libcamera framework exclusively. The legacy V4L2 camera path that earlier Pi models exposed is not available on Pi 5.

OctoPrint's camera stack (`webcamd`) depends on `mjpg_streamer`, which requires the legacy V4L2 camera path. The result is a hard failure at stream initialization:

```
mjpg_streamer[1610]: init_VideoIn failed
```

The attempted workaround chain was:

1. Install `libcamera-apps` — package not present in OctoPi image by default. Resolved with `apt install`.
2. Replace `mjpg_streamer` with `ustreamer` + `libcamerify` shim. `ustreamer` available via apt. `libcamerify` not available via apt.
3. Manually download `libcamerify` from Raspberry Pi GitHub. First URL (libcamera-apps repo) returned 404. Second URL (rpicam-apps repo) provided — **outcome not confirmed before project was paused.**

**This entire problem class is eliminated by the standalone app approach.** Without OctoPrint, there is no `webcamd` service, no `mjpg_streamer`, and no camera service of any kind running in the background. Tile capture becomes a direct subprocess call:

```python
subprocess.run([
    "rpicam-still",
    "--output", str(output_path),
    "--encoding", "png",
    "--width", "4056",
    "--height", "3040",
    "--nopreview"
])
```

This requires no camera service, no streaming shim, and no background process to conflict with. **Verification of this path on Pi 5 is pending** but expected to be straightforward.

**Note:** The Raspberry Pi 5 has been reassigned away from this project (see Section 5). This finding is retained for reference.

---

### 4.3 Raspberry Pi 3B+ + raspistill (CSI path)

**Conclusion: Known working baseline. Recommended for capture nodes.**

The Raspberry Pi 3B+ using the CSI interface and the legacy `raspistill` tool is the confirmed working configuration from initial project development. Key characteristics:

- `raspistill` outputs lossless PNG or uncompressed formats directly
- No libcamera dependency — legacy stack works natively on Raspbian/Bookworm (32-bit) and earlier
- USB serial to printer controller works without issues
- 100 Mbps Ethernet (shared USB 2.0 bus) is sufficient for tile file transfer at typical scan dwell times; a single 4056×3040 PNG is approximately 12–25 MB depending on content, taking ~3–10 seconds to transfer — well within acceptable limits at normal scan intervals

**Pending verification:** Confirm `raspistill` behaviour on current Bookworm builds. The `raspistill` binary was deprecated in favour of `rpicam-still` on newer OS images. On Pi 3B+ running a current Bookworm image this may require either using the `rpicam-still` equivalent or pinning to a Bullseye image. This should be tested and documented before committing to a software stack.

---

## 5. Hardware Architecture Decision

Based on the above findings, the following hardware topology has been adopted:

| Hardware | Assigned Role | Rationale |
|---|---|---|
| RPi 3B+ × 2 | Capture nodes (Node A, Node B) | Known working CSI camera path; dedicated to this project; age makes them unsuitable for other demanding uses |
| Dell Optiplex MFF | Processing server | Runs FastAPI server, SQLite database, stitching pipeline, Hough segmentation, web dashboard. Provides serving and processing for all nodes |
| RPi 5 | Not used in this project | Reserved for a project that benefits from its processing capability |
| Arducam UVC adapter | Bench tool only | Preview/focus alignment during setup. Not in capture pipeline |

### 5.1 Capture Node Design Philosophy

The capture nodes are **intentionally thin.** They do not run a web server, web UI, or any serving infrastructure. All of that is the responsibility of the processing server. The rationale:

- Reduces software complexity and resource load on the RPi 3B+ (1 GB RAM)
- Eliminates the need to maintain or secure a web-facing service on each node
- Keeps the node software to a single focused concern: motion control + image capture + file delivery
- Makes nodes interchangeable — the same software runs on any node

The nodes run a lightweight Python agent process (not a web server) that:
- Receives job instructions from the processing server
- Sends G-code over serial to the printer controller
- Captures tiles via `raspistill` / `rpicam-still`
- Transfers captured image files to the processing server
- Sends periodic heartbeat and status information to the server

The processing server is the single point of operator interaction for all nodes.

---

## 6. Open Items

| # | Item | Priority |
|---|---|---|
| 1 | Verify `raspistill` availability and behaviour on current Bookworm (Pi 3B+) | High |
| 2 | Verify `rpicam-still` lossless PNG output on Pi 5 standalone app (no OctoPrint) | Medium |
| 3 | Measure actual tile transfer time RPi 3B+ → Optiplex over LAN for a full-res PNG | Medium |
| 4 | Confirm Arducam UVC adapter functions as expected for preview use on Linux (V4L2) | Low |

---

## 7. Related Documents

| Document | Title |
|---|---|
| DN-MFDIG-01 | Architecture & Component Decisions |
| DN-MFDIG-04 | OctoPrint Evaluation & Migration Rationale |

---

*This document should be updated when open items are resolved or new compatibility findings are made.*