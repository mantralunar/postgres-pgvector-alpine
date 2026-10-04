# PostgreSQL Alpine with pgvector

PostgreSQL 17 on Alpine Linux with the [pgvector](https://github.com/pgvector/pgvector) extension preinstalled.

The image is based directly on the official `postgres:17-alpine` image. It keeps the standard PostgreSQL entrypoint, environment variables, data directory, and initialization behaviour.

## Image

```text
ghcr.io/OWNER/postgres-pgvector-alpine:17-alpine
```

Replace `OWNER` with the GitHub account or organization that publishes the image.

The image supports `linux/amd64`.

## Tags

- `17-alpine` — latest successful PostgreSQL 17 Alpine build.
- `<pgvector-version>-pg17-alpine` — a specific pgvector release on the latest available PostgreSQL 17 Alpine base.
- `<pgvector-version>-pg17-alpine-<digest>` — an immutable pgvector and PostgreSQL base-image combination.

Use the digest-suffixed tag for reproducible deployments.

## Run a new instance

```sh
podman volume create postgres-data

podman run -d \
  --name postgres \
  -p 127.0.0.1:5432:5432 \
  -e POSTGRES_PASSWORD='replace-me' \
  -v postgres-data:/var/lib/postgresql/data \
  ghcr.io/OWNER/postgres-pgvector-alpine:17-alpine
```

For production, provide the password with a Podman secret rather than placing it directly in the command.

## Enable pgvector

pgvector is available in the image but is not enabled automatically. Enable it separately in each database that needs it:

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

For example:

```sh
podman exec postgres \
  psql -U postgres -d bookorbit \
  -c 'CREATE EXTENSION IF NOT EXISTS vector;'
```

Confirm the installed version:

```sql
SELECT extversion
FROM pg_extension
WHERE extname = 'vector';
```

## Use an existing PostgreSQL volume

This image can replace `postgres:17-alpine` while continuing to use the same PostgreSQL 17 Alpine data volume:

1. Stop the existing container cleanly.
2. Remove the container without removing its volume.
3. Recreate it with this image and the same environment, network, and volume settings.
4. Run `CREATE EXTENSION vector` in databases that require pgvector.

Do not mount a data directory created by another PostgreSQL major version. Do not switch an existing Alpine data directory directly to a Debian-based PostgreSQL image.

Installing pgvector does not modify or enable it in other databases in the same PostgreSQL instance.

## Included PostgreSQL extensions

The underlying PostgreSQL image also provides standard contributed extensions such as:

- `pg_trgm`
- `unaccent`
- `uuid-ossp`

Enable them in the required database in the same way:

```sql
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE EXTENSION IF NOT EXISTS unaccent;
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```

## Example vector query

```sql
CREATE TABLE items (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    embedding vector(3)
);

INSERT INTO items (embedding)
VALUES ('[1,2,3]'), ('[4,5,6]');

SELECT id, embedding <-> '[1,2,4]' AS distance
FROM items
ORDER BY distance;
```

## Updates

The moving `17-alpine` tag is rebuilt when a stable pgvector release or the official PostgreSQL 17 Alpine base image changes. Each build is tested by creating the extension, inserting a vector, and running a distance query before publication.
