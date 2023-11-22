# Meta Data Schema

## Version
1.0.0

## Update Date
2023-11-22

## File Location
`.zattrs` file in root directory of .zarr file

## Meta Data Fields Explaination
||||
| ------------- | ------------- | ------------- |
| `version` | string | the version of this schema |
| `update_date` | date | the date on which this version updated |
| `channels` | list | a list of [channel](#channel) specified metadata |

#### channel
||||
| ------------- | ------------- | ------------- |
| `12bit` | int | 1 indicates each pixel (16 bit) were truncated to 12 bit |
| `all_images` | object | see [image](#image) below |
| `caption` | string | caption, inherited from '.flsm' file |
| `created_time` | string | created_time, inherited from '.flsm' file |
| `exposure` | string | exposure, inherited from '.flsm' file |
| `filter` | string | filter, inherited from '.flsm' file |
| `image_height` | string | image_height, inherited from '.flsm' file |
| `image_size` | string | image_size, inherited from '.flsm' file |
| `images` | string | images, inherited from '.flsm' file |
| `lefttop_x` | string | lefttop_x, inherited from '.flsm' file |
| `lefttop_y` | string | lefttop_y, inherited from '.flsm' file |
| `lefttop_z` | string | lefttop_z, inherited from '.flsm' file |
| `max_volts` | string | max_volts, inherited from '.flsm' file |
| `mode` | string | mode, inherited from '.flsm' file |
| `move_y` | string | move_y, inherited from '.flsm' file |
| `pixel_size` | string | pixel_size, inherited from '.flsm' file |
| `positions` | list | a list of position coordinates in millimeter, index of sublist indicates Stack Index, index within sublist indicates Frame Index, filled in (-1.0,-1.0) for missing Frames |
| `power` | string | power, inherited from '.flsm' file |
| `rightbottom_x` | string | rightbottom_x, inherited from '.flsm' file |
| `rightbottom_y` | string | rightbottom_y, inherited from '.flsm' file |
| `rightbottom_z` | string | rightbottom_z, inherited from '.flsm' file |
| `s_route` | int | s_route, inherited from '.flsm' file |
| `slash_fixed` | string | slash_fixed, inherited from '.flsm' file |
| `slices_index` | string | slices_index, inherited from '.flsm' file |
| `slides_index` | string | slides_index, inherited from '.flsm' file |
| `time_seconds` | int | time_seconds, inherited from '.flsm' file |
| `velocity` | string | velocity, inherited from '.flsm' file |
| `version` | string | version, inherited from '.flsm' file |
| `volts_offset` | string | volts_offset, inherited from '.flsm' file |
| `wave_length` | string | wave_length, inherited from '.flsm' file |
| `channel_name` | string | e.g. 405nm_10X, 640nm_10X etc. |

#### image
||||
| ------------- | ------------- | ------------- |
| `overlap` | object | see [image fields](#image-fields), inherited from '.flsm' file |
| `projection` | object | see [image fields](#image-fields), inherited from '.flsm' file |
| `raw` | object | see [image fields](#image-fields), inherited from '.flsm' file |
| `side_projection` | object | see [image fields](#image-fields), inherited from '.flsm' file |
| `thumbnail` | object | see [image fields](#image-fields), inherited from '.flsm' file |

#### image fields
||||
| ------------- | ------------- | ------------- |
| `factor` | string | factor, inherited from '.flsm' file  |
| `path` | string | path, inherited from '.flsm' file  |

