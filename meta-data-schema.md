# Meta Data Schema

## Version
1.1.0

## Update Date
2024-05-18

## File Location
`.zattrs` file in root directory of .zarr file

## Meta Data Fields Explaination
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
| `positions` | list | - | a list of position coordinates in millimeter, index of sublist indicates Stack Index, index within sublist indicates Frame Index, filled in (-1.0,-1.0) for missing Frames | [[[20.2647, 61.2581], [20.2682, 61.2581]],[[20.2647, 63.2581], [-1.0, -1.0]]] |
| `version` | string | - | version of microscope control software | 2.8.7 |
