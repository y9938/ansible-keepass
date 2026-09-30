# Changelog

All notable changes to this collection are documented here.

## [0.8.0] - 2026-09-30

### Added

- Keyfile-only authentication for the `attachment` module.
- Execution Environment dependency metadata for controller-side `pykeepass`.
- GitHub Actions checks for collection sanity, integration, lint, and build.

### Changed

- Require `ansible-core >=2.19.0` (previously `>=2.10`).
- Update repository metadata and installation instructions for the `y9938` fork while retaining the `viczem.keepass` FQCN.
- Document `pykeepass` requirements separately for the controller and the Python environment executing the module.

### Fixed

- Correct the attachment module example and the documented environment variable names.
- Make attachment exports idempotent while preserving file attribute changes.
- Report missing `pykeepass` consistently from the lookup and standalone socket command.

## [0.7.6] - 2026-09-30

### Fixed

- Replace deprecated Templar `_available_variables` access with the public `available_variables` property.
