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
### "multiscales"
algin with "multiscales" in ome-zarr spec v0.4
### "visor"
| FIELD | TYPE | UNIT | EXPLAINATION | EXAMPLE
| ------------- | ------------- | ------------- |------------- |------------- |
| `version` | string | - | the version of schema | 1.1.0 |
| `update_date` | date | yyyy-mm-dd | the date when version updated | 2024-05-18 |
| `channels` | list | - | a list of [channel](#channel) specified metadata | - |

#### channel
| FIELD | TYPE | UNIT | EXPLAINATION | EXAMPLE
| ------------- | ------------- | ------------- |------------- |------------- |
| `created_time` | date | - | time when file created, in [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) format | 2024-05-18T00:00:00Z |
| `caption` | string | - | {client_id}-{sample_id} | USTC_THY1-YFP_1779 |
| `wave_length` | string | nanometer | laser wavelength | 488 |
| `power` | string | milliwatt | laser power | 60 |
| `filter` | string | nanometer/nanometer |optical filter info, central wavelength /  bandwidth, for example, 520/40 presents 520nm±(40/2)nm i.e. 500-540nm | 520/40 |
| `exposure` | string | milliseconds | exposure time | 4 |
| `max_volts` | string | Volt | microscope scanner configuration | 2.2 |
| `volts_offset` | string | Volt | scanner offset | 0.45 |
| `s_route` | int | - | s_route, 1 for 'S' route, 0 for 'E' route | 1 |
| `velocity` | string | mm/s | velocity in x direction | 0.875 |
| `move_y` | string | millimeter | position moved in y direction | 2 |
| `12bit` | int | - | 1 indicates each pixel (16 bit) were truncated to 12 bit | 1 |
| `slices_index` | string | - | slice index | 3 |
| `slides_index` | string | - | slide index | 1 |
| `image_size` | string | pixel | width x height | 2048x788 |
| `pixel_size` | string | micrometer/pixel | micrometer per pixel | 1.03 |
| `topleft_x` | string | - | slice ROI starting coordinate ("top-left") x | 20.2647 |
| `topleft_y` | string | - | slice ROI starting coordinate ("top-left") y | 61.2581 |
| `topleft_z` | string | - | slice ROI starting coordinate ("top-left") z | 14.2395 |
| `bottomright_x` | string | - | slice ROI ending coordinate ("bottom-right") x | 24.5047 |
| `bottomright_y` | string | - | slice ROI ending coordinate ("bottom-right") y | 62.9141 |
| `bottomright_z` | string | - | slice ROI ending coordinate ("bottom-right") z | 14.2390 |
| `positions` | list | - | a list of position coordinates in millimeter, index of sublist indicates stack index | [[20.2647, 61.2581], [20.2647, 63.2581]] |
| `version` | string | - | version of microscope control software | 2.8.7 |

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
        "version": "2024.11.0",
        "update_date": "2024-11-01",
        "channels": [{
            "created_time": "2024-05-18T00:00:00Z",
            "caption": "USTC_THY1-YFP_1779",
            "wave_length": "488",
            "power": "60",
            "filter": "520/40",
            "exposure": "4",
            "max_volts": "2.2",
            "volts_offset": "0.45",
            "s_route": 1,
            "velocity": "0.875",
            "move_y": "2",
            "12bit": "1",
            "slices_index": "3",
            "slides_index": "1",
            "image_size": "2048x788",
            "pixel_size": "1.03",
            "topleft_x": "20.2647",
            "topleft_y": "61.2581",
            "topleft_z": "14.2395",
            "bottomright_x": "24.5047",
            "bottomright_y": "62.9141",
            "bottomright_z": "14.2390",
            "positions": [[20.2647, 61.2581],[20.2647, 63.2581]],
            "version": "2.8.7"
        }]
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

