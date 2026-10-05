# Changelog

<!-- format ref: https://github.com/vweevers/common-changelog -->
## [v2026.9.1]

- defined the four spaces (raw / ortho / slice / sample) with axes, units and origins; documented raw-space axes (vs/ch/z/y/x; z = frame index along the stage scan)
- renamed space `brain` to `sample` (specimens are not limited to brain); `brain` accepted as a legacy alias
- specified transform direction mathematically: `A_to_B` is a point map T: A → B; resampling is pull-back; optional `direction` field in transforms.json declares the stored mapping direction
- documented the transform directory inner layout: per stack+channel, per channel, and slice-level
- recommended per-slice transform grouping; whole-sample transforms (raw_to_sample) derived on demand, not stored
- added optional extension blocks: per-slice quality.json and anchor.json; optional parameters field in recon.json
- updated LICENSE from BSD 3-Clause to Apache License 2.0 (patent grant;
  all copyright holders consented)

[v2026.9.1]: https://github.com/visor-tech/visor-data-schema/releases/tag/v2026.9.1

## [v2025.6.1]

- use slice directory name instead of path

[v2025.6.1]: https://github.com/visor-tech/visor-data-schema/releases/tag/v2025.6.1

## [v2025.5.3]

- updated visor_recon_transforms schema

[v2025.5.3]: https://github.com/visor-tech/visor-data-schema/releases/tag/v2025.5.3

## [v2025.5.2]

- added custom projection images in raw image, for imaging quality assurance

[v2025.5.2]: https://github.com/visor-tech/visor-data-schema/releases/tag/v2025.5.2

## [v2025.5.1]

- update to adopt OME-Zarr spec v0.5 and zarr v3
- use zarr.json, deprecated .zattrs .zgroup .zarray
- added "c" zarr group
- renamed "s" to "vs", "c" to "ch"

[v2025.5.1]: https://github.com/visor-tech/visor-data-schema/releases/tag/v2025.5.1


## [v2024.11.3]

- use .vsr extension for VISoR image type
- new info.json metadata file contains sample info
- new visor_raw_images/selected.json metadata file contains selected slices and channels
- moved source images to .zattrs['sources']
- removed .visor metadata file
- updated LICENSE to BSD 3-Clause License

[v2024.11.3]: https://github.com/visor-tech/visor-data-schema/releases/tag/v2024.11.3


## [v2024.11.2]

- Use "s" custom dimension for visor_stacks, created indices mappings for "s" and "c" dimensions
- Updated .visor and .zattr metadata, removed .source metadata

[v2024.11.2]: https://github.com/visor-tech/visor-data-schema/releases/tag/v2024.11.2


## [v2024.11.1]

- Align with [OME-Zarr spec v0.4](https://ngff.openmicroscopy.org/0.4/index.html)
- VISoR-specific metadata are stored in a .visor file and "visor" field in .zattr file
- Raw, processed, and reconstructed images and transforms are each organized in separate, dedicated directories

[v2024.11.1]: https://github.com/visor-tech/visor-data-schema/releases/tag/v2024.11.1
