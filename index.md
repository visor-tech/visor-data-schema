This is the image data schema of VISoR technology, align with [OME-Zarr spec v0.4](https://ngff.openmicroscopy.org/0.4/index.html).

## Version
2024.11.0

## Version Date
2024-11-01

## Terms
| TERM | DEFINITION |
|---|---|
| `sample` | Biomedical sample, e.g. a brain, may contain multiple 'slices' |
| `slice`  | Sample slice, may contain multiple 'stacks' |
| `stack`  | A stack of 'frames' |
| `frame`  | A 2D picture took by microscopy camera |

## Data Schema
```
{SAMPLE_ID}                            # container directory of a sample
 |                                     # each mode has a subdirectory
 |                                     # we only define visor_ image schema in this spec
 |                                     # SAMPLE_ID example: BB001
 |
 ├── visor_raw_images
 |   |
 |   ├── .visor                        # sample level metadata
 |   |
 |   ├── slice_1_{PARAMETERS}.zarr     # each slice is an independent image (.zarr file)
 |   |                                 # slice index is 1-based
 |   |   ...                           # PARAMETERS format: {MAGNIFICATION}[_{MULTI_ANGLE}]
 |   |                                 # e.g. slice_1_10x | slice_23_40x_4a
 |   └── slice_m_{PARAMETERS}.zarr
 |       |
 |       ├── .zgroup
 |       ├── .zattrs                   # slice level metadata
 |       |
 |       ├── 0                         # resolution levels
 |       |   ...
 |       └── n
 |           |
 |           ├── .zarray
 |           |
 |           └── stack_{STACK_INDEX}   # visor_raw_image specific dimension
 |               |                     # align with "t" in OME-Zarr spec v0.4
 |               |                     # STACK_INDEX is 1-based
 |               |                     # e.g. stack_1
 |               |
 |               └── {CHANNEL}         # align with "c" in OME-Zarr spec v0.4
 |                   |                 # e.g. 405
 |                   |
 |                   └─ z              # "z" indicates frame number
 |                      └─ y
 |                         └─ x
 |
 |
 ├── [visor_{PROCESS_TYPE}_images]     # optional, processed images
 |   |                                 # PROCESS_TYPE definitions:
 |   |                                 # "comp": compression, e.g. visor_comp_images
 |   |                                 # "icor": illumination correction, e.g. visor_icor_images
 |   |                                 #  ...
 |   |
 |   ├── .source                       # metadata of the source images
 |   |
 |   └── ...                           # rest of the directory structure 
 |                                     # should match the corresponding raw images
 |
 |
 ├── [visor_recon_images]              # optional, reconstructed images
 |   |
 |   └── {VERSION}.zarr                # VERSION format: {PERSON_ID}_{ROI_ID}_{DATE}
 |       |                             # e.g. xxx_brain_10x_20241101
 |       |                             # e.g. xxx_slice_1_40x_icor_20241101
 |       ├── .zgroup
 |       ├── .zattrs                   # metadata
 |       |
 |       ├── 0                         # resolution levels
 |       |   ...
 |       └── n
 |           |
 |           ├── .zarray
 |           |
 |           |                         # note "t" in OME-Zarr spec v0.4 is not used
 |           |
 |           └── {CHANNEL}             # align with "c" in OME-Zarr spec v0.4
 |               |                     # e.g. 405
 |               |
 |               └─ z
 |                  └─ y
 |                     └─ x
 |
 |
 └── [visor_recon_transforms]          # optional, reconstruction transforms
     |
     └── {VERSION}                     # VERSION format: {PERSON_ID}_{ROI_ID}_{DATE}
                                       # e.g. xxx_brain_10x_20241101
                                       # e.g. xxx_slice_1_40x_icor_20241101
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

## Metadata

### .visor

#### "project_info"
information of the project
| FIELD | EXPLAINATION | EXAMPLE |
|---|---|---|
| `animal_id` | id of animal | T070 |
| `date` | date of imaging, format: yyyymmdd | 20240811 |
| `hardware_name` | name of microscope | VISoR19 |
| `personnel` | name initials of the microscopist | YY |
| `project_name` | name of project | BCP |
| `software_name` | name of imaging control software | fastLSM-2.8.7.exe |
| `species` | species | Mouse |
| `subproject_name` | name of subproject | HSYN-EGFP-1E7-3W |

#### "featured_slices"
list of featured slices, a slice may be imaged several times, use the version in this list is recommended
| FIELD | EXPLAINATION | EXAMPLE |
|---|---|---|
| `path` | path to slice directory, relative to visor_raw_images directory | slice_1_10x.zarr |

### .source

#### "paths"
if not provided, default to same file name in visor_raw_images
| FIELD | EXPLAINATION | EXAMPLE |
|---|---|---|
| `source` | path to source image, relative to {SAMPLE_ID} directory | visor_raw_images/slice_1_10x.zarr |
| `result` | path to result image, relative to {SAMPLE_ID} directory | visor_comp_images/slice_1_10x.zarr |

### .zattr

#### "multiscales"
align with "multiscales" in OME-Zarr spec v0.4

#### "visor"
microscope and imaging settings
| FIELD | TYPE | UNIT | EXPLAINATION | EXAMPLE |
|---|---|---|---|---|
| `version` | string | - | the version of schema | 2024.11.0 |
| `version_date` | date | yyyy-mm-dd | the date when version updated | 2024-11-01 |
| `channels` | list | - | a list of [channel](#channel) specified metadata | - |
##### channel
| FIELD | TYPE | UNIT | EXPLAINATION | EXAMPLE |
|---|---|---|---|---|
| `created_time` | string | - | time when file created, in [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) format | 2024-05-18T00:00:00Z |
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
| `stack_positions` | list | millimeter | a list of stack position coordinates in millimeter, [[stack_1_topleft_x, stack_1_topleft_y],...,[stack_m_topleft_x, stack_m_topleft_y]] | [[20.2647, 61.2581], [20.2647, 63.2581]] |
| `version` | string | - | version of microscope control software | 2.8.7 |

#### "transforms"
reconstruction transforms
##### transform
| FIELD | EXPLAINATION | EXAMPLE |
|---|---|---|
| `path` | path to transform parameter directory, relative to {SAMPLE_ID} directory | visor_recon_transforms/xxx_brain_10x_20241101/ |
| `roi_coordinates` | a list of position coordinates in after-transform space, [x1,y1,z1,x2,y2,z2] | [0,0,0,256,256,256] |

## Metadata Examples

Example: visor_raw_images/.visor
```json
{
    "project_info": {
        "animal_id": "T070",
        "date": "20240811",
        "hardware_name": "VISoR19",
        "personnel": "YY",
        "project_name": "BCP",
        "software_name": "fastLSM-2.8.7.exe",
        "species": "Mouse",
        "subproject_name": "HSYN-EGFP-1E7-3W"
    },  
    "featured_slices": [
        {
            "path": "slice_1_10x.zarr"
        },
        ...
        {
            "path": "slice_23_40x.zarr"
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
                "scale": [1.0, 1.0, 3.5, 1.0, 1.0]
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
    "visor": {
        "version": "2024.11.0",
        "version_date": "2024-11-01",
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
        "path": "visor_recon_transforms/xxx_brain_20241101/",
        "roi_coordinates": [0,0,0,256,256,256]
    }]
}
```

## References

[Ome NGFF Spec](https://ngff.openmicroscopy.org/latest/)

[Zarr Spec](https://zarr-specs.readthedocs.io/en/latest/specs.html)

[Ome NGFF Paper](https://www.nature.com/articles/s41592-021-01326-w)

