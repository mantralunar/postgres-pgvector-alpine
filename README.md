# PostgreSQL 17 Alpine with pgvector

This repository needs only one workflow file. GitHub Actions creates the temporary container build instructions, compiles pgvector against `postgres:17-alpine`, tests it, and publishes the resulting `linux/amd64` image to GitHub Container Registry.

## Repository contents

```text
.github/workflows/publish.yml
README.md
```

## Setup

1. Create a GitHub repository.
2. Add `.github/workflows/publish.yml` from this repository.
3. Optionally add this README.
4. Push to the default branch.
5. Open **Settings → Actions → General** and give workflows read and write permissions.
6. Open **Actions**, select **Build PostgreSQL Alpine with pgvector**, and choose **Run workflow**.
7. Open the generated package and make it public if anonymous pulls are required.

The workflow uses GitHub's automatically provided `GITHUB_TOKEN`; no registry password or repository secret is required.

## Published image

For a repository at `github.com/example/postgres-pgvector`, pull:

```sh
podman pull ghcr.io/example/postgres-pgvector:17-alpine
```

The workflow publishes three tags:

- `17-alpine` — latest successful build;
- `<pgvector-version>-pg17-alpine` — latest PostgreSQL Alpine base for that pgvector release;
- `<pgvector-version>-pg17-alpine-<digest>` — immutable upstream combination.

Use the digest-suffixed tag for reproducible production deployments.

## Automatic updates

The workflow runs daily at 04:23 UTC. It checks:

- the latest stable pgvector Git tag;
- the current multi-platform digest behind `postgres:17-alpine`.

A new image is built when either upstream value changes. If the exact combination already exists in GHCR, the scheduled run exits without rebuilding it. You can also run the workflow manually at any time.

## Using the image

Recreate the PostgreSQL container using the new image and the same existing PostgreSQL 17 Alpine data volume. Do not delete or initialize the volume again.

Enable pgvector only in the database that needs it:

```sh
podman exec postgres \
  psql -U postgres -d bookorbit \
  -c 'CREATE EXTENSION IF NOT EXISTS vector;'
```

Adding pgvector to the image only makes the extension available. It does not enable it automatically in the other databases.
