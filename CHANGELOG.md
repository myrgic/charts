# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Commit-message convention: [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)
(`feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`, `ci:`).

## [Unreleased]

### Added

### Changed

- README: documented that no `ghcr.io/myrgic/*` container images are published yet, and pointed `helm install` / `docker compose up` users at the build-from-source path instead.

### Fixed

- Kernel port default (`charts/cogos-kernel/values.yaml`, `charts/cogos-node/values.yaml`, `charts/cogos-mod3/values.yaml`, `docker-compose.yml`) was still `5200` after the README examples were fixed to `6931`. Chart and compose defaults now match the kernel's actual default port.

<!--
Release template — copy this block, bump the version, date it, and move
Unreleased entries into the new release section:

## [X.Y.Z] - YYYY-MM-DD

### Added
- ...

### Changed
- ...

### Fixed
- ...

### Removed
- ...

### Security
- ...
-->
