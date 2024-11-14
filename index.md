This is the image data schema of VISoR technology, align with [OME-Zarr spec v0.4](https://ngff.openmicroscopy.org/0.4/index.html).

## Version
2024.11.2

## Version Date
2024-11-14

## Terms
| TERM | DEFINITION |
|---|---|
| `sample` | Biomedical sample, e.g. a brain, may contain multiple 'slices' |
| `slice`  | Sample slice, may contain multiple 'stacks' |
| `stack`  | A stack of 'frames' |
| `frame`  | A 2D picture took by microscopy camera |

## Data Schema
```
{SAMPLE_ID}                            # sample directory, each mode is organized into its own subdirectory
 |                                     # we only define visor_ image schema in this spec
 |                                     # SAMPLE_ID example: BB001
 |
 ├── visor_raw_images
 |   |
 |   ├── .visor                        # VISoR specific sample level metadata, see ".visor" metadata
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
 |       ├── .zattrs                   # OME-Zarr compatible slice level metadata
 |       ├── .zgroup
 |       |
 |       ├── 0                         # resolution levels
 |       |   ...
 |       └── n
 |           |
 |           ├── .zarray
 |           |
 |           └── s                     # "s" is a custom dimension specific to visor_raw_images, see "visor_stacks" metadata
 |               └─ c                  # "c" is wavelength channel, see "channels" metadata
 |                  └─ z               # "z" is frame number
 |                     └─ y
 |                        └─ x
 |
 |
 ├── [visor_{PROCESS_TYPE}_images]     # optional, processed images
 |   |                                 # PROCESS_TYPE definitions:
 |   |                                 # "recon": reconstruction, i.e. visor_recon_images
 |   |                                 # "compr": compression, i.e. visor_compr_images
 |   |                                 # "icorr": illumination correction, i.e. visor_icorr_images
 |   |                                 #  ...
 |   |
 |   └── {VERSION}.zarr                # VERSION format: {PERSON_ID}_{ROI_ID}_{DATE}
 |       |                             # e.g. xx_brain_10x_20241101
 |       |                             # e.g. xxx_slice_1_40x_icorr_20241101
 |       |
 |       ├── .visor                    # metadata contains source images
 |       ├── .zattrs                   # roi level metadata
 |       ├── .zgroup
 |       |
 |       ├── 0                         # resolution levels
 |       |   ...
 |       └── n
 |           |
 |           ├── .zarray
 |           |
 |           └── s                     # "s" is optional; for example, "stacks" are no longer present after reconstruction
 |               └─ c                  # "c" is wavelength channel, see "channels" metadata
 |                  └─ z               # "z" is frame number
 |                     └─ y
 |                        └─ x
 |
 |
 └── [visor_recon_transforms]          # optional, reconstruction transforms
     |
     └── {VERSION}                     # VERSION format: {PERSON_ID}_{ROI_ID}_{DATE}
                                       # e.g. xxx_brain_10x_20241101
                                       # e.g. xxx_slice_1_40x_icorr_20241101
```

## Metadata

Metadata formats are based on [OME-Zarr spec v0.4](https://ngff.openmicroscopy.org/0.4/index.html), with VISoR specifc extensions.

### Structure Overview
| SUBDIRECTORY | SAMPLE LEVEL | ROI/SLICE LEVEL |
|---|---|---|
| visor_raw_images | [.visor](#quotvisorquot) ||
||| [.zattr](#quotzattrquot) |
| visor_{PROCESS_TYPE}_images |||
||| [.zattr](#quotzattrquot) |
||| [.visor](#quotvisorquot) |

### Fields comparison with OME-Zarr spec v0.4
| File | OME-Zarr v0.4 | VISoR |
|---|---|---|
| .zattr | [multiscales](https://ngff.openmicroscopy.org/0.4/index.html#multiscale-md) | [multiscales](#multiscales) |
|| - | [visor_stacks](#visorstacks) |
|| - | [channels](#channels) |
|| - | [transforms](#transforms) |
| .visor | - | [project_info](#projectinfo) |
|| - | [selected_images](#selectedimages) |

### ".visor"

#### project_info
Information of the project
| FIELD | EXPLAINATION | EXAMPLE |
|---|---|---|
| `animal_id` | id of animal | "T070" |
| `project_name` | name of project | "BCP" |
| `species` | species | "Mouse" |
| `subproject_name` | name of subproject | "HSYN-EGFP-1E7-3W" |

#### selected_images
A list of selected images.
- For raw images, a slice may be imaged several times, use the version in this list is recommended
- For processed images, this is the source images on which the current process is based.

| FIELD | EXPLAINATION | EXAMPLE |
|---|---|---|
| `path` | path to image directory, relative to {SAMPLE_ID} directory | "visor_icorr_images/slice_1_10x.zarr" |

### ".zattr"

#### multiscales
Align with "multiscales" in OME-Zarr spec v0.4.

#### visor_stacks
A list of VISoR stacks with corresponding axis index mappings.
- Axis indices are 0-based and increment sequentially
- Stack indices are 1-based with some stacks potentially missing

| FIELD | TYPE | UNIT | EXPLAINATION | EXAMPLE |
|---|---|---|---|---|
| `index` | int | - | visor_stack axis index | 0 |
| `label` | string | - | stack identifier | "stack_1" |
| `position` | list[float] | millimeter | 2D physical position coordinates [top_left_x, top_left_y] | [20.2647, 61.2581] |

#### channels
A list of wavelength channels with corresponding axis index mappings.
- Axis indices are 0-based and increment sequentially;
- The order of channels is not guaranteed.

| FIELD | TYPE | UNIT | EXPLAINATION | EXAMPLE |
|---|---|---|---|---|
| `index` | int | - | channel axis index | 0 |
| `wavelength` | string | nanometer | laser wavelength | "488" |
| `slice_id` | int | - | slice id | 3 |
| `slide_id` | int | - | slide id | 1 |
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
| `v_schema` | string | - | the version of schema | "2024.11.2" |
| `created_time` | string | - | time when file created, in [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) format | "2024-05-18T00:00:00Z" |
| `personnel` | name initials of the microscopist | "YY" |

#### transforms
A list of reconstruction transforms.
| FIELD | TYPE | EXPLAINATION | EXAMPLE |
|---|---|---|---|
| `path` | string | path to transform parameter directory, relative to {SAMPLE_ID} directory | "visor_recon_transforms/xxx_brain_10x_20241101" |
| `roi` | list[float] | 3D roi coordinates in after-transform space, [x1,y1,z1,x2,y2,z2] | [0.0,0.0,0.0,256.0,256.0,256.0] |

### Examples

Example: visor_raw_images/.visor
```json
{
    "project_info": {
        "animal_id": "T070",
        "project_name": "BCP",
        "species": "Mouse",
        "subproject_name": "HSYN-EGFP-1E7-3W"
    },  
    "selected_images": [
        {
            "path": "visor_raw_images/slice_1_10x.zarr"
        },
        ...
        {
            "path": "visor_raw_images/slice_23_40x.zarr"
        }
    ]
}
```

Example: visor_recon_images/xxx_slice_1_40x_20241101.zarr/.visor
```json
{
    "selected_images": [
        {
            "path": "visor_icorr_images/xxx_slice_1_40x_20241012.zarr"
        }
    ]
}
```

Example: visor_recon_images/xxx_brain_40x_20241101.zarr/.visor
```json
{
    "selected_images": [
        {
            "path": "visor_raw_images/slice_1_10x.zarr"
        },
        ...
        {
            "path": "visor_raw_images/slice_23_40x.zarr"
        }
    ]
}
```

Example: visor_raw_images/slice_1_10x.zarr/.zattrs
```json
{
    "multiscales": [
        {
            "version": "0.4",
            "name": "slice_1_10x",
            "axes": [
                {"name": "s", "type": "visor_stack"},
                {"name": "c", "type": "channel"},
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
                "version": "2024.9.1",
                "args": "[np.mean]",
                "kwargs": {"multichannel": true}
            }
        }
    ],
    "visor_stacks": [
        {"index": 0, "label": "stack_1", "position": [20.2647, 61.2581]},
        {"index": 1, "label": "stack_3", "position": [20.2647, 65.2581]}
    ],
    "channels": [{
        "index": 0,
        "wavelength": "488",
        "slice_id": 3,
        "slide_id": 1,
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
        "v_schema": "2024.11.2",
        "created_time": "2024-11-12T00:00:00Z",
        "personnel": "YY"
    }]
}
```

Example: visor_recon_images/xxx_brain_40x_20241101.zarr/.zattrs
```json
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
                "version": "2024.9.1",
                "args": "[np.mean]",
                "kwargs": {"multichannel": true}
            }
        }
    ],
    "transforms": [{
        "path": "visor_recon_transforms/xxx_brain_20241101",
        "roi": [0.0, 0.0, 0.0, 256.0, 256.0, 256.0]
    }]
}
```

## Typical values

| DESCRIPTION | VALUE |
|---|---|
| number of stacks | 3 |
| stack shape | (1474, 788, 2048) |
| frame shape | (788, 2048) |
| dtype | dtype('uint16') |
| chunk size | (256, 256, 256) |
| default fill pixel value | 0 |
| memory order | 'C' |

## References

[Ome NGFF Spec](https://ngff.openmicroscopy.org/latest/)

[Zarr Spec](https://zarr-specs.readthedocs.io/en/latest/specs.html)

[Ome NGFF Paper](https://www.nature.com/articles/s41592-021-01326-w)

