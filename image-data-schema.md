# Image Data Schema

## Version
1.2.0

## Update Date
2024-08-02

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
 {SAMPLE_ID}                                            (e.g. BB001)
 ├── VISoR_Raw_Images
 |   └── Slice_{SLICE_INDEX}.zarr                       (1-based)
 |       ├── .zattrs
 |       └── {CHANNEL}                                  (e.g. 405nm_10X)
 |           └── Stack_{STACK_INDEX}                    (0-based)
 |               └── Resolution_Level_{RESOLUTION_LEVEL}
 |                   ├── .zarray
 |                   ├── {CHUNKS}
 |                   └── ...
 └── VISoR_Reconstructed_Images (optional)
     └── {VERSION}                                      ('-' separated identifiers, e.g. xxx-20240801)
         ├── metadata.json
         ├── Transform
         |   └── {SRC_SPACE}_to_{DES_SPACE}.{FILE_EXT}  (e.g. Raw_to_Slice_{SLICE_INDEX}.zarr | Slice_to_Whole_Sample.pth | ...)
         └── Image (optional)
             └── {IMAGE_TYPE}                           (e.g. Whole_Sample | Slice_{SLICE_INDEX} | ...)
                 └── {CHANNEL}                          (e.g. 405nm_10X)
                     └── {RESOLUTION}                   (e.g. 1um)
                         ├── .zarray
                         ├── {CHUNKS}
                         └── ...
```

## Typical values

Demo code:
```python
f_path = r'Z:\mnt\VISoR_Data_r\zarr\N1779\Slice_006.zarr'
zg = zarr.open(f_path, mode='r')
a = zg['640nm_10X']['Stack_1']['Resolution_Level_1']
```

||||
|---|---|---|
| Number of stacks | 3 | `len(zg['640nm_10X'].keys())` |
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
