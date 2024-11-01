# VISoR Data Schema

## Version
2024.11.0

## Update Date
2024-11-01

## Terms
|||
|---|---|
| `sample` | Biomedical sample, e.g. a brain, may contain multiple 'slices' |
| `slice`  | Sample slice, may contain multiple 'stacks' |
| `stack`  | A stack of 'frames' |
| `frame`  | A 2D picture took by microscopy camera |

## Data Schema
```
{SAMPLE_ID}                            # Top level container directory of a sample
 |                                     # each mode has a subdirectory
 |                                     # SAMPLE_ID e.g. BB001
 |
 ├── visor_raw_images
 |   |
 |   ├── slice_1.zarr                  # One zarr file per slice
 |   |   ...                           # with slice level metadata
 |   └── slice_m.zarr                  # slice index is 1-based
 |       |
 |       ├── .zgroup
 |       ├── .zattrs
 |       |
 |       ├── 0                         # resolution levels
 |       |   ...
 |       └── n
 |           |
 |           ├── .zarray
 |           |
 |           └── stack_{STACK_INDEX}   # visor_raw_image specific dimension
 |               |                     # align with "t" in ome-zarr spec v0.4
 |               |                     # STACK_INDEX is 1-based
 |               |                     # e.g. stack_1
 |               |
 |               └── {CHANNEL}         # align with "c" in ome-zarr spec v0.4
 |                   |                 # e.g. 405nm_10X
 |                   └─ z
 |                      └─ y
 |                         └─ x
 |
 |
 |
 ├── visor_reconstructed_images        # optional
 |   |
 |   └── {VERSION}.zarr                # version format: {PERSON_ID}_{ROI_ID}_{DATE}
 |       |                             # e.g. xxx_brain_20241101 or xxx_slice_1_20241101
 |       ├── .zgroup
 |       ├── .zattrs
 |       |
 |       ├── 0                         # Resolution levels
 |       |   ...
 |       └── n
 |           |
 |           ├── .zarray
 |           |                         # note "t" in ome-zarr spec v0.4 is not used
 |           └── {CHANNEL}             # align with "c" in ome-zarr spec v0.4
 |               |                     # e.g. 405nm_10X
 |               └─ z
 |                  └─ y
 |                     └─ x
 |
 |
 └── visor_reconstruction_transforms   # optional
     |
     └── {VERSION}                     # version format: {PERSON_ID}_{ROI_ID}_{DATE}
                                       # e.g. xxx_brain_20241101 or xxx_slice_1_20241101
```

## Typical values
|||
|---|---|
| number of stacks | 3
| stack shape | (1474, 788, 2048)
| frame shape | (788, 2048)
| dtype       | dtype('uint16')
||||
| Zarr related  |---
| chunk Size    | (256, 256, 256)
| default fill pixel value | 0
| memory order  | 'C'
||||

## Metadata
Example: visor_raw_images/slice_1.zarr/.zattrs
```JSON
{
    "multiscales": [
        {
            "version": "0.4",
            "name": "slice_1",
            "axes": [
                {"name": "t", "type": "visor_stack"},
                {"name": "c", "type": "channel"},
                {"name": "z", "type": "space", "unit": "micrometer"},
                {"name": "y", "type": "space", "unit": "micrometer"},
                {"name": "x", "type": "space", "unit": "micrometer"}
            ],
            "datasets": [
                {
                    "path": "0",
                    "coordinateTransformations": [{
                        // the voxel size for the first scale level (1 micrometer)
                        "type": "scale",
                        "scale": [1.0, 1.0, 1.0, 1.0, 1.0]
                    }]
                },
                {
                    "path": "1",
                    "coordinateTransformations": [{
                        // the voxel size for the second scale level (downscaled by a factor of 2 -> 2 micrometer)
                        "type": "scale",
                        "scale": [1.0, 1.0, 1.0, 2.0, 2.0]
                    }]
                }
            ],
            "coordinateTransformations": [{
                // the voxel size for the first scale level (1 micrometer)
                "type": "scale",
                "scale": [1.0, 1.0, 3.5, 1.0, 1.0]
            },{
                // physical position of the slice
                "type": "translation",
                "scale": [1.0, 1.0, 1.0, 61258.1, 20264.7]
            }],
            "type": "mean",
            "metadata": {
                "description": "the fields in metadata depend on the downscaling implementation. Here, the parameters passed to the dask function are given",
                "method": "dask.array.coarsen",
                "version": "2024.9.1",
                "args": "[np.mean]",
                "kwargs": {"multichannel": true}
            }
        }
    ],
    "visor": {
    }
}
```
Example: visor_reconstructed_images/xxx_slice_1_20241101.zarr/.zattrs
```JSON
{
    "multiscales": [
        {
            "version": "0.4",
            "name": "xxx_slice_1_20241101",
            "axes": [
                {"name": "c", "type": "channel"},
                {"name": "z", "type": "space", "unit": "micrometer"},
                {"name": "y", "type": "space", "unit": "micrometer"},
                {"name": "x", "type": "space", "unit": "micrometer"}
            ],
            "datasets": [
                {
                    "path": "0",
                    "coordinateTransformations": [{
                        // the voxel size for the first scale level (1 micrometer)
                        "type": "scale",
                        "scale": [1.0, 1.0, 1.0, 1.0, 1.0]
                    }]
                },
                {
                    "path": "1",
                    "coordinateTransformations": [{
                        // the voxel size for the second scale level (downscaled by a factor of 2 -> 2 micrometer)
                        "type": "scale",
                        "scale": [1.0, 1.0, 2.0, 2.0, 2.0]
                    }]
                }
            ],
            "type": "mean",
            "metadata": {
                "description": "the fields in metadata depend on the downscaling implementation. Here, the parameters passed to the dask function are given",
                "method": "dask.array.coarsen",
                "version": "2024.9.1",
                "args": "[np.mean]",
                "kwargs": {"multichannel": true}
            }
        }
    ]
}
```

## References
[Ome NGFF Spec](https://ngff.openmicroscopy.org/latest/)

[Zarr Spec](https://zarr-specs.readthedocs.io/en/latest/specs.html)

[Ome NGFF Paper](https://www.nature.com/articles/s41592-021-01326-w)

