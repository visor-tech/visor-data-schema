# Changelog

<!-- format ref: https://github.com/vweevers/common-changelog -->
## [v2025.5.1]

- update to adopt OME-Zarr spec v0.5 and zarr v3
- use zarr.json, deprecated .zattrs .zgroup .zarray
- added "c" zarr group
- renamed "s" to "vs", "c" to "ch"

[v2024.5.1]: https://github.com/visor-tech/visor-data-schema/releases/tag/v2024.5.1


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
