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
| `version` | string | - | the version of this schema | 1.1.0 |
| `update_date` | date | yyyy-mm-dd | the date on which this version updated | 2024-05-18 |
| `channels` | list | - | a list of [channel](#channel) specified metadata | - |

#### channel
| FIELD | TYPE | UNIT | EXPLAINATION | EXAMPLE
| ------------- | ------------- | ------------- |------------- |------------- |
| `created_time` | string | - | time when file created, in [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) format | 2024-05-18T00:00:00Z |
| `caption` | string | - |{client_id}-{sample_id} | XJY-TY1-2 |
| `channel_name` | string | - | name of channel | 488nm_10X |
| `wave_length` | string | nanometer | laser wavelength | 488 |
| `power` | string | milliwatt | laser power | 20.1 |
| `filter` | string | nanometer/nanometer |optical filter info, central wavelength /  bandwidth, for example, 520/40 presents 520nm±(40/2)nm i.e. 500-540nm | 520/40 |
| `exposure` | string | milliseconds | exposure time | 0.3 |
| `max_volts` | string | Volt | microscope scanner configuration | 2.3 |
| `volts_offset` | string | Volt | scanner offset | 0.2 |
| `s_route` | int | - | s_route, 1 for 'S' route, 0 for 'E' route | 1 |
| `velocity` | string | mm/s | velocity in x direction | 2 |
| `move_y` | string | millimeter | position moved in y direction | 2 |
| `12bit` | int | - | 1 indicates each pixel (16 bit) were truncated to 12 bit | 1 |
| `slices_index` | string | - | slice index | 3 |
| `slides_index` | string | - | slide index | 1 |
| `image_size` | string | pixel | width x height | 2048x788 |
| `pixel_size` | string | micrometer/pixel | micrometer per pixel | 1 |
| `topleft_x` | string | - | slice ROI starting coordinate ("top-left") x | 10.375 |
| `topleft_y` | string | - | slice ROI starting coordinate ("top-left") y | 5.208 |
| `topleft_z` | string | - | slice ROI starting coordinate ("top-left") z | 3.472 |
| `bottomright_x` | string | - | slice ROI ending coordinate ("bottom-right") x | 12.282 |
| `bottomright_y` | string | - | slice ROI ending coordinate ("bottom-right") y | 6.211 |
| `bottomright_z` | string | - | slice ROI ending coordinate ("bottom-right") z | 5.337 |
| `positions` | list | - | a list of position coordinates in millimeter, index of sublist indicates Stack Index, index within sublist indicates Frame Index, filled in (-1.0,-1.0) for missing Frames | - |
| `version` | string | - | version of microscope control software | 2.8.7 |
| `mode` | string | - | TO_BE_CLARIFY | - |
| `slash_fixed` | string | - | TO_BE_CLARIFY | - |
| `time_seconds` | int | - | TO_BE_CLARIFY | - |
