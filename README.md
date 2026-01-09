# Publish Labspace Action

A GitHub Action that packages and deploys your Labspace.

## What it does

This action:

1. Sets up a known-working version of Docker Compose
2. Creates a base64-encoded tarball of the content in the repository
3. Generates a `.labspace/compose.pipeline.yaml` file with:
   - A `project_source_tar` config containing the base64-encoded tarball
   - Mounts the tarball into the `configurator` service and sets the `PROJECT_TAR_PATH` environment variable pointing to the mounted tarball
4. Publishes the Compose file using `docker compose publish`

## Prerequisites

Your repository must have a `.labspace/compose.overrides.yaml` file, following the convention set in the dockersamples/labspace-starter repo.

## Configuration

### Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `target_repo` | The repo where the Compose file should be published | Yes | - |
| `target_tag` | The tag for the Compose file should be published | No | `latest` |
| `labspace_base_version` | The version of the base Labspace Compose file | No | `latest` |
| `labspace_overrides_file` | Path to the Labspace Compose overrides file | No | `.labspace/compose.overrides.yaml` |

## Usage

```yaml
name: Publish Labspace

on:
  push:
    branches: [ main ]

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Log in to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Publish Labspace
        uses: dockersamples/publish-labspace-action@v1
        with:
          target_repo: dockersamples/labspace-demo
```

## How it works

The action creates a Docker Compose override file that embeds your project source code as a base64-encoded tarball. This tarball is then available to the `configurator` service at runtime through the `PROJECT_TAR_PATH` environment variable.

The generated `compose.pipeline.yaml` structure:

```yaml
configs:
  project_source_tar:
    content: |
      <base64-encoded-tarball>

services:
  configurator:
    configs:
      - source: project_source_tar
        target: /etc/labspace-support/content.tar.base64
    environment:
      PROJECT_TAR_PATH: /etc/labspace-support/content.tar.base64
```

