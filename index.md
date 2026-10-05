This is the image data schema of VISoR `(pronounced /ˈvaɪ.zər/)` technology, align with [OME-Zarr spec v0.5](https://ngff.openmicroscopy.org/0.5/index.html).

## Version
2026.9.1

## Version Date
2026-09-30

## Terms
| TERM | DEFINITION |
|---|---|
| `sample` | Biomedical sample, e.g. a brain or a whole body, may contain multiple 'slices' |
| `slice`  | Sample slice, may contain multiple 'stacks' |
| `stack`  | A stack of 'frames' |
| `frame`  | A 2D picture taken by microscopy camera |

## Spaces
Reconstruction transforms map points between four named spaces. A transform named `A_to_B` is a point map `T: A → B` (`p_B = T(p_A)`); composition follows `T_BC ∘ T_AB = T_AC`, and inverses are taken explicitly (`(T_AB)⁻¹ = T_BA`). **Resampling is pull-back**: to fill an output grid in space *A* from data defined on space *B*, sample the source at `T_A_to_B(p)` for every output point `p ∈ A`.

| SPACE | AXES / UNITS | ORIGIN | NOTES |
|---|---|---|---|
| `raw` | array axes `vs/ch/z/y/x`; x = frame width pixels (fast axis), y = frame height pixels, z = frame index along the stage scan (step = frame pitch, sign = scan direction); C memory order | (0, 0, 0) at the first frame corner, per stack, per channel | the stored frames themselves; each x–y plane is one `frame`, and the frame plane is oblique (±45° light sheet) to the physical slice plane — the obliquity is carried by `raw_to_ortho`, it is not part of raw space |
| `ortho` | micrometer, per stack | the stack's stage position (from acquisition metadata) | de-skewed, scaled, positioned; the ±45° de-skew lives in `raw_to_ortho` |
| `slice` | micrometer, per slice | the sample-wide min corner of the slice's stack bounds | the slice's stacks registered into one frame |
| `sample` | micrometer, whole specimen | slice *i* owns z ∈ [z_i, z_i + t_i); x/y origin 0 | slices stacked along z; the placement (fixed nominal thickness or measured per-slice thickness) is recorded with the transforms. `brain` is accepted as a legacy alias of `sample` (specimens are not limited to brain) |

Schema v2025.6.1 named the fourth space `brain`; `sample` supersedes it (`slice_to_brain` → `slice_to_sample`).

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
 |       ├── slide.tar                 # custom projection images for imaging quality assurance
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
     └── {VERSION}                     # VERSION format: {PERSON_ID}_{DATE}
         |                             # e.g. xxx_20250525
         |
         ├── recon.json                # reconstruction metadata, see "recon.json"
         |
         ├── slice_1_{PARAMETERS}      # each slice directory corresponds to its respective slice raw image
         |                             # contains an independent transform group
         |   ...                       # matches the raw image naming convention
         |
         └── slice_m_{PARAMETERS}
             |
             ├── transforms.json       # slice level transforms metadata, see "transforms.json"
             |
             ├── raw_to_ortho          # transform from visor raw image space to orthogonal space
             ├── raw_to_slice          # transform from visor raw image space to slice space
             ├── raw_to_sample         # transform from visor raw image space to sample space
             ├── slice_to_sample       # transform from slice space to sample space
             |                         # (brain is accepted as a legacy alias of sample, e.g. slice_to_brain)
             |
             ├── [quality.json]        # optional extension block: per-transform quality metrics
             └── [anchor.json]         # optional extension block: anchor frame + placement policy

```

## Metadata

Metadata formats are based on [OME-Zarr spec v0.5](https://ngff.openmicroscopy.org/0.5/index.html), with VISoR specific extensions.

### Structure Overview
| DIRECTORY | SAMPLE LEVEL | ROI LEVEL |
|---|---|---|
| {SAMPLE_ID}.vsr | [info.json](#quotinfojsonquot) ||
| {SAMPLE_ID}.vsr/visor_raw_images | [selected.json](#quotselectedjsonquot) | [zarr.json](#quotzarrjsonquot) |
| {SAMPLE_ID}.vsr/visor_{PROCESS_TYPE}_images || [zarr.json](#quotzarrjsonquot) |
| {SAMPLE_ID}.vsr/visor_recon_transforms | [recon.json](#quotreconjsonquot) | [transforms.json](#quottransformsjsonquot) |

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
| `name` | name of slice | "slice_1_10x" |
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
| `filter` | string | nanometer/nanometer | optical filter info, central wavelength /  bandwidth, for example, 520/40 represents 520nm±(40/2)nm i.e. 500-540nm | "520/40" |
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
| `v_schema` | string | - | the version of schema | "2025.6.1" |
| `created_time` | string | - | time when file created, in [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) format | "2024-05-18T00:00:00Z" |
| `personnel` | string | - | name initials of the microscopist | "YY" |

#### sources
A list of source images, on which the current process is based.
| FIELD | TYPE | DESCRIPTION | EXAMPLE |
|---|---|---|---|
| `path` | string | path to source image directory, relative to {SAMPLE_ID}.vsr directory | "visor_raw_images/slice_1_10x.zarr" |
| `channels` | list[string] | list of wavelength channels | ["488","561"] |

#### transform_version
Version of the reconstruction transform.
e.g. xxx_20250525

### "recon.json"
Information of the `reconstruction`.
| FIELD | DESCRIPTION | EXAMPLE |
|---|---|---|
| `personnel` | person who did reconstruction | "YY" |
| `create_time` | time when reconstruction finished, in the ISO 8601 format | "2024-05-18T00:00:00Z" |
| `spaces` | a list of available spaces | "sample" "slice" "ortho" "raw" |
| `keywords` | a list of reconstruction algorithms, libraries etc. | "b-spline" "elastic" "deep learning" |
| `slices` | list of slice transforms | see slices |
| `parameters` | optional; the effective parameter set of the reconstruction run (preset name + values), for reproducibility | {"preset": "mouse_body", ...} |
#### slices
A list of source images, on which the current process is based.
| FIELD | TYPE | DESCRIPTION | EXAMPLE |
|---|---|---|---|
| `name` | string | name of slice | "slice_1_10x" |
| `transforms` | list[string] | list of available transforms | ["raw_to_ortho","raw_to_sample"] |

### "transforms.json"
List of reconstruction transforms.
| FIELD | DESCRIPTION | EXAMPLE |
|---|---|---|
| `name` | name of transform directory, relative to slice directory | "raw_to_ortho" |
| `type` | type of transform | "affine" "b-spline" "dense displacement field" "neural network" |
| `format` | store format of transform | "npy" "zarr" "mha" "onnx" "tfm" |
| `direction` | optional; declares the actual mapping direction of the stored transform, `"{space}_to_{space}"`. Defaults to `name`. The name only identifies the entry; `direction` decides the mapping semantics (see "Transform layout") | "raw_to_slice" |

#### Transform layout
Inside one slice directory, a transform directory `{name}` holds:
```
{name}/{stack}/{channel}/{type}.{format}      per stack+channel (e.g. per-stack affines)
{name}/{channel}/{type}.{format}              per channel only
{name}/{type}.{format}                        slice-level (one transform for the whole slice)
```
- stack / channel are numeric indices (0-based), matching the `visor_stacks` and `channels` axis indices.
- `direction` matters when a transform is stored in the opposite direction of its name: readers must return a point map in the requested direction, inverting when necessary (affines invert analytically; dense displacement fields do not — store the required direction explicitly).

#### Per-slice grouping
Transforms are grouped by slice on purpose: resampling selects a ROI or reads per-slice chunks in parallel, and never needs a whole-sample transform at once. Whole-sample compositions (`raw_to_sample`) are therefore **derived on demand** and not stored — storing them would duplicate geometry.

#### Extension blocks
Optional per-slice sidecar files (JSON) follow the schema's extension idiom:
| FILE | CONTENT |
|---|---|
| `quality.json` | per-transform quality metrics (e.g. per-stack / per-interface / per-block NCC, SSIM, residuals) |
| `anchor.json` | anchor frame description + slice placement policy (nominal vs measured thickness) |
The effective parameter set of a reconstruction run may be recorded in `recon.json` as `parameters`.


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
        "name": "slice_1_10x",
        "channels": ["488","561"]
    },
    {
        "name": "slice_1_10x_1",
        "channels": ["405","640"]
    },
    ...
    {
        "name": "slice_23_40x",
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
                "v_schema": "2025.6.1",
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
            "transform_version": "xxx_20250525"
        }
    }
}
```

Example: visor_recon_transforms/xxx_20250525/recon.json
```json
{
    "personnel": "YY",
    "create_time": "2025-05-25T20:25:05Z",
    "spaces": ["raw","ortho","slice","sample"],
    "keywords": ["SimpleITK","Elastix"],
    "slices": [
        {
            "name": "slice_1_10x",
            "transforms": ["raw_to_ortho", "raw_to_sample"]
        }
    ]
}
```

Example: visor_recon_transforms/xxx_20250525/slice_1_10x/transforms.json
```json
[
    {
        "name": "raw_to_ortho",
        "type": "affine",
        "format": "tfm",
        "direction": "raw_to_ortho"
    },
    {
        "name": "sample_to_slice",
        "type": "dense displacement field",
        "format": "mha",
        "direction": "sample_to_slice"
    }
]
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

