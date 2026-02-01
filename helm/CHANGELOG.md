# Changelog

All notable changes to this Helm chart will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2026-02-01

### Added

- Initial release of Configarr Helm chart
- CronJob-based deployment for periodic configuration sync
- ConfigMap for main configuration file
- Secret management for API keys
- PersistentVolumeClaim for repository cache
- Comprehensive documentation (README.md, INSTALL.md)
- Example values file with multiple instance configuration
- Support for customizable:
  - CronJob schedule
  - Resource limits and requests
  - Security contexts
  - Node selection and affinity
  - Persistence settings
- Post-install NOTES.txt with helpful commands
- Helm template helpers for consistent naming

### Features

- Runs Configarr as a Kubernetes CronJob (default: hourly)
- Persistent storage for repository cache to avoid repeated downloads
- Secure secret management for API keys
- Full compatibility with Configarr configuration format
- Support for multiple Sonarr/Radarr instances
- TTY support for colored logs
- Configurable job history limits
- Concurrency control for job execution

[0.1.0]: https://github.com/raydak-labs/configarr/releases/tag/helm-0.1.0
