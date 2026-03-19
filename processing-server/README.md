# Processing Server

FastAPI + SQLite server running on the Dell Optiplex MFF. This is the single point of operator interaction for the entire system.

## Responsibilities

- Receiving and ingesting complete fiche tile batches from capture nodes
- Running the image processing pipeline (stitching, segmentation, normalization)
- Maintaining the fiche database
- Serving the web dashboard to operators
- Exposing the REST API consumed by capture node agents
- Monitoring capture node heartbeats and status

## Hardware

Dell Optiplex MFF, Intel Core i5 9th gen, 16 GB DDR4
