# Shared image workflow (GHCR, amd64 + arm64)

One reusable workflow builds every app image the same way:

- **One native runner per architecture** (amd64 on `ubuntu-latest`, arm64 on `ubuntu-24.04-arm`), no emulation.
- **One multi-arch image** in GitHub Container Registry: `ghcr.io/<org>/<repo>`.
- **Immutable tags:** `sha-<7 chars>` for every commit, plus the branch name (`main`, `staging`) and semver tags on releases.
- **Optional Coolify deploy** after the image is published.

## Use it in a repo

Copy `workflow-templates/docker-publish.yml` into the repo's `.github/workflows/` (it also appears under
Actions, New workflow, for the org). It calls:

```yaml
uses: Cascadian-Technology-CAS-Tech/.github/.github/workflows/docker-publish.yml@main
```

The caller must grant `contents: read` and `packages: write`.

## Inputs

| Input | Default | Meaning |
|---|---|---|
| `image-name` | the calling repo, lowercased | Path under `ghcr.io` |
| `context`, `dockerfile` | `.`, `./Dockerfile` | Build context and Dockerfile |
| `platforms` | `["linux/amd64","linux/arm64"]` | Use `["linux/amd64"]` until the app is tested on arm64 |
| `build-args` | empty | Non-secret `KEY=value` lines |
| `push` | `true` | `false` builds only (pull requests) |
| `deploy-method` | `GET` | Method for the Coolify webhook |

| Secret | Meaning |
|---|---|
| `build-secrets` | `KEY=value` lines, read with `RUN --mount=type=secret,id=KEY` (never use build args for secrets) |
| `coolify-webhook`, `coolify-token` | Trigger a deploy after publishing; skipped when empty |

## Run the image on a Coolify host

Each host needs a read-only login once: `docker login ghcr.io -u <user> --password-stdin` with a token that has
`read:packages`. In Coolify, set the application to use the image `ghcr.io/<org>/<repo>:sha-<commit>` so a deploy
and a rollback are just a tag change.

## Before turning on arm64 for an app

Third-party images are already multi-arch. Check the app's own Dockerfile for native modules and pinned x86
binaries, build it with `platforms: ["linux/amd64","linux/arm64"]`, and run the arm64 image on a staging host first.
