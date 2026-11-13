# Changelog

All notable user-visible changes to the Verdaccio TOS application package.

## 7.1.0-1 — 2026-11-14

### Added
- First release. Verdaccio (`verdaccio/verdaccio:7.1.0`, Docker Hub official image, unmodified)
  packaged as a TOS 7 Docker application, published on port 8007 and running
  as a non-root user (1000:1000).
- Persistent data is kept under `/Volume1/DockerAppData/verdacciodocker/`.
