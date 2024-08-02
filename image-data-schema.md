# Image Data Schema

## Version
1.1.0

## Update Date
2024-05-20

## Terms
|||
|---|---|
| `Slice` | Brain Slice, contains multiple 'Stacks' |
| `Stack` | A collection of 'Frames' |
| `Frame` | A 2D picture took by microscopy camera | 

## Data Format References
[Zarr](https://zarr.readthedocs.io/en/stable/spec/v2.html) / [Ome-NGFF](https://ngff.openmicroscopy.org/latest/)

[The Paper](https://www.nature.com/articles/s41592-021-01326-w)

## Data Directory Hierarchy
```bash
 {SAMPLE_ID}                                      (e.g. BB001)
 ├── VISoR_Raw_Images
 |   └── Slice_{SLICE_INDEX}.zarr                 (1-based)
 |       ├── .zattrs
 |       └── Resolution_Level_{RESOLUTION_LEVEL}
 |           └── Stack_{STACK_INDEX}              (0-based)
 |               └── {CHANNEL_ID}                 (e.g. 405nm_10X)
 |                   ├── .zarray
 |                   ├── {CHUNKS}
 |                   └── ...
 └── VISoR_Reconstructed_Images (optional)
     └── {VERSION}-{CHANNEL}.zarr                 (e.g. xxx-20240801-405nm_10X.zarr)
         ├── .zattrs
         └── {ROI} (e.g. Whole_Sample / Slice_{SLICE_INDEX} / or arbitrary ROI name)
             ├── .zattrs
             ├── Transform
             |   ├── .zarray
             |   ├── {CHUNKS}
             |   └── ...
             └── Image
                 └── {RESOLUTION}                     (e.g. 1um)
                     ├── .zarray
                     ├── {CHUNKS}
                     └── ...
```

## Typical values

Demo code:
```python
f_path = r'Z:\mnt\VISoR_Data_r\zarr\N1779\Slice_006.zarr'
zg = zarr.open(f_path, mode='r')
a = zg['Resolution_Level_1']['Stack_1']['640nm_10X']
```

||||
|---|---|---|
| Number of stacks | 3 | `len(zg['Resolution_Level_1'].keys())` |
| Stack shape | (1474, 788, 2048) | `a.shape`       |
| Frame shape | (788, 2048)       | `a[0].shape`    |
| dtype       | dtype('uint16')   | `a.dtype`       |
| Metadata    | {'version': '1.0.0',... | `dict(a.attrs)` |
||||
| zarr related  |---| `a.info` |
| Chunk Size    | (256, 256, 256) | `a.chunks`     |
| Default value | 0               | `a.fill_value` |
| Memory order  | 'C'             | `a.order` |
||||

See [meta-data-schema.md](meta-data-schema.md) for Metadata.

## **!IMPORTANT!** Design Considerations 
- One zarr file per Slice, with Slice level metadata
- In a Slice, all Stacks have same number of Frames
- Filled in all zeros for missing Frames
