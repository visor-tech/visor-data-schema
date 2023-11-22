# Image Data Schema

## Version
1.0.0

## Update Date
2023-11-22

## Terms
|||
|---|---|
| `Slice` | Brain Slice, contains multiple 'Stacks' |
| `Stack` | A collection of 'Frames' |
| `Frame` | A 2D picture took by microscopy camera | 

## Data Format
[Zarr](https://zarr.readthedocs.io/en/stable/spec/v2.html) / [Ome-NGFF](https://ngff.openmicroscopy.org/latest/)

[The Paper](https://www.nature.com/articles/s41592-021-01326-w)

## Data Directory Hierarchy
```bash
 {SAMPLE_ID} (e.g. BB001)
 ├── Slice_{SLICE_INDEX}.zarr
 │   └── Resolution_Level_{RESOLUTION_LEVEL}
 │       ├── Stack_{STACK_INDEX}
 │       │   ├── {CHANNEL_ID} (e.g. 405nm_10X)
 │       │   │   ├── {CHUNKS}
 │       │   │   ├── ...
```

## Chunk Size
256x256x256

## **!IMPORTANT!** Design Considerations 
- One zarr file per Slice, with Slice level metadata
- In a Slice, all Stacks have same number of Frames
- Filled in all zeros for missing Frames