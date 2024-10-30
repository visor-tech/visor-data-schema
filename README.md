# VISoR Image Data Schema

## Version
2024.11.0

## Update Date
2024-11-01

## Terms
|||
|---|---|
| `Slice` | Brain Slice, contains multiple 'Stacks' |
| `Stack` | A collection of 'Frames' |
| `Frame` | A 2D picture took by microscopy camera |

## Data Schema
```
{SAMPLE_ID}                                              # Sample may contain multimodal data
 |                                                       # visor_raw and visor_reconstructed images are separate zarr files
 |                                                       # share the same SAMPLE_ID
 |                                                       # e.g. BB001
 |
 ├── visor_raw_image.zarr
 |   |
 |   ├── .zgroup
 |   ├── .zattrs
 |   |
 |   ├── 0                                               # Resolution levels
 |   |   ...
 |   ├── n
 |   |   |
 |   |   ├── .zarray
 |   |   |
 |   |   └── slice_{SLICE_INDEX}_stack_{STACK_INDEX}     # VISoR Raw Image specific dimension
 |   |       |                                           # Align with "t" in ome-zarr spec v0.4
 |   |       |                                           # both SLICE_INDEX and STACK_INDEX are 1-based
 |   |       |                                           # e.g. slice_1_stack_1
 |   |       |
 |   |       └── {CHANNEL}                               # Align with "c" in ome-zarr spec v0.4
 |   |           |                                       # e.g. 405nm_10X
 |   |           └─ z
 |   |              └─ y
 |   |                 └─ x
 |   └── labels                                          # TBD
 |
 |
 ├── visor_reconstructed_image.zarr                      # Optional
 |   |
 |   ├── .zgroup
 |   ├── .zattrs
 |   |
 |   ├── 0                                               # Resolution levels
 |   |   ...
 |   ├── n
 |   |   |
 |   |   ├── .zarray
 |   |   |
 |   |   └── {PERSON_ID}_{ROI}_{DATE}                    # VISoR Reconstructed Image specific dimension
 |   |       |                                           # Align with "t" in ome-zarr spec v0.4
 |   |       |                                           # e.g. xxx_brain_20241101 or xxx_slice_1_20241101
 |   |       |
 |   |       └── {CHANNEL}                               # Align with "c" in ome-zarr spec v0.4
 |   |           |                                       # e.g. 405nm_10X
 |   |           └─ z
 |   |              └─ y
 |   |                 └─ x
 |   └── labels                                          # TBD
 |
 |
 └── visor_reconstruction_transforms                     # TBD
```

## Typical values
|||
|---|---|
| Number of stacks | 3
| Stack shape | (1474, 788, 2048)
| Frame shape | (788, 2048)
| dtype       | dtype('uint16')
||||
| zarr related  |---
| Chunk Size    | (256, 256, 256)
| Default fill pixel value | 0
| Memory order  | 'C'
||||

## Metadata
.zarr/.zattrs
```
{
    "multiscales": [
        {
            "version": "0.4",
            // use .zarr file name
            "name": "visor_raw_image",
            "axes": [
                {"name": "t", "type": "visor"},
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
                    "path": "0/slice_1_stack_1",
                    "coordinateTransformations": [{
                        // TBD, how to fill in these values?
                        "type": "translation",
                        "translation": [1.0, 1.0, 1.0, 61258.1, 20264.7]
                    }]
                },
                ...
                {
                    "path": "1",
                    "coordinateTransformations": [{
                        // the voxel size for the second scale level (downscaled by a factor of 2 -> 2 micrometer)
                        "type": "scale",
                        "scale": [1.0, 1.0, 2.0, 2.0, 2.0]
                    }]
                }
                ...
            ],
            "type": "coarsen",
            "metadata": {
                "description": "the fields in metadata depend on the downscaling implementation. Here, the parameters passed to the dask function are given",
                "method": "dask.array.coarsen",
                "version": "2024.9.1",
                "args": "[true]",
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

