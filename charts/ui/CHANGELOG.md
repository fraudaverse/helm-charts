# Changelog

## v0.1.0

### Added

* Support for displaying rule instructions
* Functionality to manage defined risk lists
* Persmission to have custom layouts. Users without permission to have custom layouts will always "inherit" the default layout without the ability to change it. Only the `super` user has the permission to change layouts, both `admin` and `demo` don't have it. This is currently "hardcoded" because there is no system to deliver permissions yet from an IAM system. The functionality is working though.

### Fixed

* Investigation filters now support filter for the "New" status and combinations of multiple values for the "not equal" operator are fixed

## v0.0.7 (backend and frontend)

### Added

- Case actions on close codes

## v0.0.5 (backend and frontend)

### Changed

- Various fixes of findings during deployment
## v0.0.4

### Changed

- Update ui-frontend image to version 0.0.4 which which uses a specific nginx version with additional statements to make it rootless
## v0.0.3

### Changed

- Update ui-frontend image to version 0.0.3 which which uses the rootless nginx image


## v0.0.2

### Changed

- Update ui-frontend image to version 0.0.2 which allows to run nginx as unprivileged

