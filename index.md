This is the image data schema of VISoR `(pronounced /ˈvaɪ.zər/)` technology, align with [OME-Zarr spec v0.5](https://ngff.openmicroscopy.org/0.5/index.html).

## Version
2025.5.1

## Version Date
2025-05-04

## Terms
| TERM | DEFINITION |
|---|---|
| `sample` | Biomedical sample, e.g. a brain, may contain multiple 'slices' |
| `slice`  | Sample slice, may contain multiple 'stacks' |
| `stack`  | A stack of 'frames' |
| `frame`  | A 2D picture took by microscopy camera |

## Data Schema
```
{SAMPLE_ID}.vsr                        # use .vsr extension for VISoR image type
 |                                     # each subtype/mode is organized into its own subdirectory
 |                                     # we only define visor_ images schema in this spec
 |                                     # file name example: BB001.vsr
 |
 ├── info.json                         # sample info metadata, see "info.json"
 |
 ├── visor_raw_images
 |   |
 |   ├── selected.json                 # selected raw images metadata, see "selected.json"
 |   |
 |   ├── slice_1_{PARAMETERS}.zarr     # each slice is an independent image (.zarr file)
 |   |                                 # slice index is 1-based
 |   |                                 # PARAMETERS format: {MAGNIFICATION}[_{MULTI_ANGLE}][_{VERSION}]
 |   |   ...                           # e.g. slice_1_10x
 |   |                                 # e.g. slice_23_40x_4a90, _4a90 means the 90 degree of 4-angle imaging
 |   |                                 # e.g. slice_23_40x_1, _1 is a version indicator, typically an integer
 |   |
 |   └── slice_m_{PARAMETERS}.zarr
 |       |
 |       ├── zarr.json                 # contains custom slice level metadata, see zarr.json
 |       |
 |       ├── 0                         # resolution levels
 |       |   ...
 |       └── n
 |           |
 |           ├── zarr.json
 |           |
 |           └── c                     # "c" is zarr v3 group for "chunks"
 |               └── vs                # "vs" is a custom dimension specific to visor_raw_images, see "visor_stacks"
 |                   └─ ch             # "ch" is wavelength channel, see "channels"
 |                       └─ z          # "z" is frame number
 |                          └─ y
 |                             └─ x
 |
 |
 ├── [visor_{PROCESS_TYPE}_images]     # optional, processed images
 |   |                                 # PROCESS_TYPE definitions:
 |   |                                 # "recon": reconstruction, i.e. visor_recon_images
 |   |                                 # "compr": compression, i.e. visor_compr_images
 |   |                                 # "icorr": illumination correction, i.e. visor_icorr_images
 |   |                                 # "projn": projection, i.e. visor_projn_images
 |   |                                 #  ...
 |   |
 |   └── {VERSION}.zarr                # VERSION format: {PERSON_ID}_{ROI_ID}_{DATE}
 |       |                             # e.g. xx_brain_10x_20241101
 |       |                             # e.g. xxx_slice_1_40x_icorr_20241101
 |       |
 |       ├── zarr.json                 # contains custom roi level metadata, see zarr.json
 |       |
 |       ├── 0                         # resolution levels
 |       |   ...
 |       └── n
 |           |
 |           ├── zarr.json
 |           |
 |           └── c                     # "c" is zarr v3 group for "chunks"
 |               └── vs                # "vs" is optional; for example, "stacks" are no longer present after reconstruction
 |                   └─ ch             # "ch" is wavelength channel, see "channels"
 |                       └─ z          # "z" is frame number
 |                          └─ y
 |                             └─ x
 |
 |
 └── [visor_recon_transforms]          # optional, reconstruction transforms
     |
     └── {VERSION}                     # VERSION format: {PERSON_ID}_{ROI_ID}_{DATE}
                                       # e.g. xxx_brain_10x_20241101
                                       # e.g. xxx_slice_1_40x_icorr_20241101
```

## Metadata

Metadata formats are based on [OME-Zarr spec v0.5](https://ngff.openmicroscopy.org/0.5/index.html), with VISoR specifc extensions.

### Structure Overview
| DIRECTORY | SAMPLE LEVEL | ROI LEVEL |
|---|---|---|
| {SAMPLE_ID}.vsr | [info.json](#quotinfojsonquot) ||
| {SAMPLE_ID}.vsr/visor_raw_images | [selected.json](#quotselectedjsonquot) | [zarr.json](#quotzarrjsonquot) |
| {SAMPLE_ID}.vsr/visor_{PROCESS_TYPE}_images || [zarr.json](#quotzarrjsonquot) |

### Fields comparison with OME-Zarr spec v0.5
| File | OME-Zarr v0.5 | VISoR |
|---|---|---|
| zarr.json | [multiscales](https://ngff.openmicroscopy.org/0.5/index.html#multiscale-md) | [multiscales](#multiscales) |
|| - | [visor_stacks](#visorstacks) |
|| - | [channels](#channels) |
|| - | [sources](#sources) |
|| - | [transforms](#transforms) |
| info.json | - | [info.json](#quotinfojsonquot) |
| selected.json | - | [selected.json](#quotselectedjsonquot) |

### "info.json"
Information of the `sample`.
| FIELD | DESCRIPTION | EXAMPLE |
|---|---|---|
| `animal_id` | id of animal | "T070" |
| `project_name` | name of project | "BCP" |
| `species` | species | "Mouse" |
| `subproject_name` | name of subproject | "HSYN-EGFP-1E7-3W" |

### "selected.json"
A list of selected slices and channels. For raw images, a slice, or a channel, may be imaged multiple times; it is recommended to use the selected version listed here.
| FIELD | DESCRIPTION | EXAMPLE |
|---|---|---|
| `path` | path to slice zarr file, relative to visor_raw_images directory | "slice_1_10x.zarr" |
| `channels` | list of wavelength channels | ["488","561"] |

### "zarr.json"

#### multiscales
Align with "multiscales" in OME-Zarr spec v0.5.

#### visor_stacks
A list of VISoR stacks with corresponding axis index mappings.
- Axis indices are 0-based and increment sequentially
- Stack indices are 1-based with some stacks potentially missing

| FIELD | TYPE | UNIT | DESCRIPTION | EXAMPLE |
|---|---|---|---|---|
| `index` | int | - | visor_stack axis index | 0 |
| `label` | string | - | stack identifier | "stack_1" |
| `position` | list[float] | millimeter | 2D physical position coordinates [top_left_x, top_left_y] | [20.2647, 61.2581] |

#### channels
A list of wavelength channels with corresponding axis index mappings.
- Axis indices are 0-based and increment sequentially
- The order of channels is not guaranteed

| FIELD | TYPE | UNIT | DESCRIPTION | EXAMPLE |
|---|---|---|---|---|
| `index` | int | - | channel axis index | 0 |
| `wavelength` | string | nanometer | laser wavelength | "488" |
| `slice_index` | int | - | slice index | 3 |
| `slide_index` | int | - | slide index | 1 |
| `hardware_id` | string | name of microscope | "VISoR19" |
| `power` | float | milliwatt | laser power | 60.0 |
| `filter` | string | nanometer/nanometer | optical filter info, central wavelength /  bandwidth, for example, 520/40 presents 520nm±(40/2)nm i.e. 500-540nm | "520/40" |
| `exposure` | float | milliseconds | exposure time | 4.0 |
| `max_volts` | float | volt | microscope scanner configuration | 2.2 |
| `volts_offset` | float | volt | scanner offset | 0.45 |
| `s_route` | int | - | s_route, 1 for 'S' route, 0 for 'E' route | 1 |
| `velocity` | float | mm/s | velocity in x direction | 0.875 |
| `move_y` | float | millimeter | position moved in y direction | 2.0 |
| `12bit` | int | - | 1 indicates each pixel (16 bit) were truncated to 12 bit | 1 |
| `image_size` | string | pixel | width x height | "2048x788" |
| `pixel_size` | float | micrometer/pixel | micrometer per pixel | 1.03 |
| `roi` | list[float] | millimeter | 3D physical roi position coordinates for the slice, [top_left_x, top_left_y, top_left_z, bottom_right_x, bottom_right_y, bottom_right_z] | [20.2647, 61.2581, 14.2395, 24.5047, 62.9141, 14.2390] |
| `v_software` | string | - | the version of microscope control software | "2.8.7" |
| `v_schema` | string | - | the version of schema | "2025.5.1" |
| `created_time` | string | - | time when file created, in [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) format | "2024-05-18T00:00:00Z" |
| `personnel` | string | - | name initials of the microscopist | "YY" |

#### sources
A list of source images, on which the current process is based.
| FIELD | TYPE | DESCRIPTION | EXAMPLE |
|---|---|---|---|
| `path` | string | path to source image directory, relative to {SAMPLE_ID}.vsr directory | "visor_raw_images/slice_1_10x.zarr" |
| `channels` | list[string] | list of wavelength channels | ["488","561"] |

#### transforms
A list of reconstruction transforms.
| FIELD | TYPE | DESCRIPTION | EXAMPLE |
|---|---|---|---|
| `path` | string | path to transform parameter directory, relative to {SAMPLE_ID}.vsr directory | "visor_recon_transforms/xxx_brain_10x_20241101" |
| `roi` | list[float] | 3D roi coordinates in after-transform space, [x1,y1,z1,x2,y2,z2] | [0.0,0.0,0.0,256.0,256.0,256.0] |

### Examples

Example: info.json
```json
{
    "animal_id": "T070",
    "project_name": "BCP",
    "species": "Mouse",
    "subproject_name": "HSYN-EGFP-1E7-3W"
}
```

Example: visor_raw_images/selected.json
```json
[
    {
        "path": "slice_1_10x.zarr",
        "channels": ["488","561"]
    },
    {
        "path": "slice_1_10x_1.zarr",
        "channels": ["405","640"]
    },
    ...
    {
        "path": "slice_23_40x.zarr",
        "channels": ["405","488","561","640"]
    }
]
```

Example: visor_raw_images/slice_1_10x.zarr/zarr.json
```json
{
    "zarr_format": 3,
    "node_type": "group",
    "attributes": {
        "ome": {
            "version": "0.5",
            "multiscales": [
                {
                    "name": "slice_1_10x",
                    "axes": [
                        {"name": "vs", "type": "visor_stack"},
                        {"name": "ch", "type": "channel"},
                        {"name": "z", "type": "space", "unit": "micrometer"},
                        {"name": "y", "type": "space", "unit": "micrometer"},
                        {"name": "x", "type": "space", "unit": "micrometer"}
                    ],
                    "datasets": [
                        {
                            "path": "0",
                            "coordinateTransformations": [{
                                "type": "scale",
                                "scale": [1.0, 1.0, 1.0, 1.0, 1.0]
                            }]
                        },
                        {
                            "path": "1",
                            "coordinateTransformations": [{
                                "type": "scale",
                                "scale": [1.0, 1.0, 1.0, 2.0, 2.0]
                            }]
                        }
                        ...
                    ],
                    "coordinateTransformations": [{
                        "type": "scale",
                        "scale": [1.0, 1.0, 3.5, 1.03, 1.03]
                    }],
                    "type": "mean",
                    "metadata": {
                        "method": "dask.array.coarsen",
                        "version": "2025.4.1",
                        "args": "[np.mean]",
                        "kwargs": {"multichannel": true}
                    }
                }
            ]
        },
        "visor": {
            "visor_stacks": [
                {"index": 0, "label": "stack_1", "position": [20.2647, 61.2581]},
                {"index": 1, "label": "stack_3", "position": [20.2647, 65.2581]}
            ],
            "channels": [{
                "index": 0,
                "wavelength": "488",
                "slice_index": 3,
                "slide_index": 1,
                "hardware_id": "VISoR19",
                "power": 60.0,
                "filter": "520/40",
                "exposure": 4.0,
                "max_volts": 2.2,
                "volts_offset": 0.45,
                "s_route": 1,
                "velocity": 0.875,
                "move_y": 2.0,
                "12bit": 1,
                "image_size": "2048x788",
                "pixel_size": 1.03,
                "roi": [20.2647, 61.2581, 14.2395, 24.5047, 62.9141, 14.2390],
                "v_software": "2.8.7",
                "v_schema": "2025.5.1",
                "created_time": "2024-11-12T00:00:00Z",
                "personnel": "YY"
            }]
        }
    }
}
```

Example: visor_projn_images/xxx_slice_1_10x_20241101.zarr/zarr.json
```json
{
    "zarr_format": 3,
    "node_type": "group",
    "attributes": {
        "ome": {
            "version": "0.5",
            "multiscales": [
                {
                    "name": "slice_1_10x",
                    "axes": [
                        {"name": "vs", "type": "visor_stack"},
                        {"name": "ch", "type": "channel"},
                        {"name": "y", "type": "space", "unit": "micrometer"},
                        {"name": "x", "type": "space", "unit": "micrometer"}
                    ],
                    "datasets": [
                        {
                            "path": "0",
                            "coordinateTransformations": [{
                                "type": "scale",
                                "scale": [1.0, 1.0, 1.0, 1.0]
                            }]
                        },
                        {
                            "path": "1",
                            "coordinateTransformations": [{
                                "type": "scale",
                                "scale": [1.0, 1.0, 2.0, 2.0]
                            }]
                        }
                        ...
                    ],
                    "coordinateTransformations": [{
                        "type": "scale",
                        "scale": [1.0, 1.0, 1.03, 1.03]
                    }],
                    "type": "mean",
                    "metadata": {
                        "method": "dask.array.coarsen",
                        "version": "2025.4.1",
                        "args": "[np.mean]",
                        "kwargs": {"multichannel": true}
                    }
                }
            ]
        },
        "visor": {
            "visor_stacks": [
                {
                    "index": 0,
                    "label": "stack_1"
                },
                {
                    "index": 1,
                    "label": "stack_3"
                }
            ],
            "channels": [
                {
                    "index": 0,
                    "wavelength": "488"
                },
                {
                    "index": 1,
                    "wavelength": "561"
                }
            ],
            "sources": [
                {
                    "path": "visor_icorr_images/slice_1_10x.zarr",
                    "channels": ["488","561"]
                }
            ]
        }
    }
}
```

Example: visor_recon_images/xxx_brain_40x_20241101.zarr/zarr.json
```json
{
    "zarr_format": 3,
    "node_type": "group",
    "attributes": {
        "ome": {
            "version": "0.5",
            "multiscales": [
                {
                    "name": "xxx_slice_1_20241101",
                    "axes": [
                        {"name": "ch", "type": "channel"},
                        {"name": "z", "type": "space", "unit": "micrometer"},
                        {"name": "y", "type": "space", "unit": "micrometer"},
                        {"name": "x", "type": "space", "unit": "micrometer"}
                    ],
                    "datasets": [
                        {
                            "path": "0",
                            "coordinateTransformations": [{
                                "type": "scale",
                                "scale": [1.0, 1.0, 1.0, 1.0, 1.0]
                            }]
                        },
                        {
                            "path": "1",
                            "coordinateTransformations": [{
                                "type": "scale",
                                "scale": [1.0, 1.0, 2.0, 2.0, 2.0]
                            }]
                        }
                        ...
                    ],
                    "type": "mean",
                    "metadata": {
                        "method": "dask.array.coarsen",
                        "version": "2025.4.1",
                        "args": "[np.mean]",
                        "kwargs": {"multichannel": true}
                    }
                }
            ]
        },
        "visor": {
            "channels": [
                {
                    "index": 0,
                    "wavelength": "488"
                },
                {
                    "index": 1,
                    "wavelength": "561"
                }
            ],
            "sources": [
                {
                    "path": "visor_raw_images/slice_1_40x.zarr",
                    "channels": ["488"]
                },
                {
                    "path": "visor_raw_images/slice_1_40x_1.zarr",
                    "channels": ["561"]
                },
                ...
                {
                    "path": "visor_raw_images/slice_23_40x.zarr",
                    "channels": ["488","561"]
                }
            ],
            "transforms": [{
                "path": "visor_recon_transforms/xxx_brain_20241101",
                "roi": [0.0, 0.0, 0.0, 256.0, 256.0, 256.0]
            }]
        }
    }
}
```

## Typical values

| DESCRIPTION | VALUE |
|---|---|
| number of stacks | 3 |
| stack shape | (1474, 788, 2048) |
| frame shape | (788, 2048) |
| data_type | 'uint16' |
| chunk_grid | (1, 1, 64, 64, 64) |
| raw data shard shape | (1, 1, 8192, 832, 2048) |
| recon data shard shape | (1, 1, 2048, 2048, 2048) |
| chunk_key_encoding | os.sep |
| default fill pixel value | 0 |
| memory order | 'C' |

## References

[Ome NGFF Spec](https://ngff.openmicroscopy.org/latest/)

[Zarr Spec](https://zarr-specs.readthedocs.io/en/latest/specs.html)

[Ome NGFF Paper](https://www.nature.com/articles/s41592-021-01326-w)

