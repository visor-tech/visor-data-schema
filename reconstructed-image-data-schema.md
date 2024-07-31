# Reconstructed Image Data Schema

## Version
1.1.0

## Update Date
2024-08-01

## Data Directory Hierarchy
### Reconstructed Slice
```bash
 {SAMPLE_ID}                   (e.g. BB001)
 └── Slice_{SLICE_INDEX}.zarr  (1-based)
     ├── .zattrs
     └── {CHANNEL_ID}          (e.g. 405nm_10X)
         └── Resolution_Level_{RESOLUTION_LEVEL}
             ├── .zarray
             ├── {CHUNKS}
             └── ...
```
### Reconstructed Sample (Brain)
```bash
 {SAMPLE_ID}.zarr  (e.g. BB001)
 ├── .zattrs
 └── {CHANNEL_ID}  (e.g. 405nm_10X)
     └── Resolution_Level_{RESOLUTION_LEVEL}
         ├── .zarray
         ├── {CHUNKS}
         └── ...
```
