# Changelog

<!-- format ref: https://github.com/vweevers/common-changelog -->

## [v2024.11.2]

- Use "s" custom dimension for visor_stacks, created indices mappings for "s" and "c" dimensions
- Updated .visor and .zattr metadata, removed .source metadata

[v2024.11.2]: https://github.com/visor-tech/visor-data-schema/releases/tag/v2024.11.2


## [v2024.11.1]

- Align with [OME-Zarr spec v0.4](https://ngff.openmicroscopy.org/0.4/index.html)
- VISoR-specific metadata are stored in a .visor file and "visor" field in .zattr file
- Raw, processed, and reconstructed images and transforms are each organized in separate, dedicated directories

[v2024.11.1]: https://github.com/visor-tech/visor-data-schema/releases/tag/v2024.11.1
