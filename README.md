# Shared GitHub Actions and Workflows

This folder contains reusable GitHub Actions and workflows that can be shared across multiple repositories.

## Available Workflows

### Docker Build and Publish (`docker-build-publish.yml`)

A reusable workflow for building and publishing Docker images to GitHub Container Registry (GHCR).

#### Features

- Multi-architecture builds (linux/amd64, linux/arm64)
- Automatic tagging based on git refs (branch, PR, SHA, semver)
- Docker layer caching for faster builds
- Comprehensive metadata and labels
- Security scanning ready
- Detailed build summaries

#### Usage

```yaml
name: Build and Publish Docker Image

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  docker-build-publish:
    permissions:
      contents: read
      packages: write
      id-token: write
    uses: aimeerivers/shared/.github/workflows/docker-build-publish.yml@main
    with:
      service_name: your-service-name
      dockerfile_path: ./Dockerfile  # optional, defaults to ./Dockerfile
      context_path: .               # optional, defaults to .
      registry: ghcr.io            # optional, defaults to ghcr.io
      platforms: linux/amd64,linux/arm64  # optional, defaults to both
    secrets:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

#### Input Parameters

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `service_name` | ✅ | - | Name of the service (used for image naming) |
| `dockerfile_path` | ❌ | `./Dockerfile` | Path to the Dockerfile relative to repository root |
| `context_path` | ❌ | `.` | Build context path relative to repository root |
| `registry` | ❌ | `ghcr.io` | Container registry URL |
| `platforms` | ❌ | `linux/amd64,linux/arm64` | Target platforms for multi-arch builds |

#### Required Secrets

| Secret | Description |
|--------|-------------|
| `GITHUB_TOKEN` | GitHub token for authentication (automatically available in GitHub Actions) |

#### Generated Tags

The workflow automatically generates the following tags:

- `latest` (only on default branch)
- Branch name (e.g., `main`, `develop`)
- PR number (e.g., `pr-123`)
- Git SHA (e.g., `sha-abcd1234`)
- Semver tags (e.g., `v1.2.3`, `1.2`, `1`) if pushing a version tag

#### Image Location

Images are published to: `ghcr.io/{owner}/{service_name}`

For example: `ghcr.io/aimeerivers/watchthis-home-service`

#### Usage in watchthis-e2e

After the image is published, it can be used in your end-to-end tests:

```bash
# Pull the latest image
docker pull ghcr.io/aimeerivers/watchthis-home-service:latest

# Or use a specific version
docker pull ghcr.io/aimeerivers/watchthis-home-service:v2.2.9
```

In `docker-compose.yml`:

```yaml
services:
  home-service:
    image: ghcr.io/aimeerivers/watchthis-home-service:latest
    ports:
      - "7279:7279"
```

## Repository Setup

To use these shared workflows, ensure your repository has the necessary permissions:

1. **Packages permissions**: The `GITHUB_TOKEN` needs `packages: write` permission
2. **Contents permissions**: The `GITHUB_TOKEN` needs `contents: read` permission

These are automatically granted when using `secrets.GITHUB_TOKEN` in the workflow.

## Contributing

When adding new reusable workflows:

1. Place them in `.github/workflows/` 
2. Use `workflow_call` trigger
3. Document inputs, outputs, and usage
4. Include comprehensive examples
5. Update this README

## Security Notes

- Images are published to GHCR with public visibility by default
- Use proper secret management for any sensitive build arguments
- Consider using OIDC for enhanced security in production environments