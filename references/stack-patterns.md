# Stack integration patterns

These are patterns, not drop-in code. Adapt them to what the inspection
step found, and prefer the application's existing conventions over these
sketches. All patterns produce the schema defined in SKILL.md section 5.

## Configuration inputs

| Input | Source | When |
|---|---|---|
| `GIT_COMMIT` | `git rev-parse HEAD` or CI (`GITHUB_SHA`) | build time |
| `BUILD_TIME` | `date -u +%Y-%m-%dT%H:%M:%SZ` | build time (optional) |
| `APP_ENV` | `production` / `preview` from the deployment's env config | runtime |
| `REPO_URL` | e.g. `https://github.com/<owner>/<repo>` | runtime config (optional) |
| `REPO_VISIBILITY` | `public` / `private`, set explicitly | runtime config |
| release metadata | `release-info.json` written by the release workflow | runtime file |

`APP_ENV` is a runtime setting, not a build argument, so that a preview
artifact can be promoted to production unchanged (SKILL.md section 7.1).
Never infer `REPO_VISIBILITY` from whether GitHub is reachable.

## Build-time capture

Copy [../scripts/build-env.sh](../scripts/build-env.sh) into the
application unchanged (SKILL.md section 8.4) and call it from the existing
deploy path. It prints `GIT_COMMIT` and `BUILD_TIME` as `KEY=value` lines,
fails on an invalid SHA, and refuses a dirty tree unless `ALLOW_DIRTY=1`.

A dirty working tree means the SHA does not match the deployed source
(SKILL.md section 1.1), so production builds should refuse it. Preview
builds may set `ALLOW_DIRTY=1` if the project's preview workflow needs it.

## Release metadata

The release has to come from an explicit, published source (SKILL.md
section 5.4), never from `git describe` or the nearest tag. GitHub's
"latest release" excludes drafts and prereleases.

Copy [../scripts/update-release-info.sh](../scripts/update-release-info.sh)
into the application unchanged and run it on the host on each deploy and
after publishing a release:

```sh
scripts/update-release-info.sh build-meta/release-info.json "$GIT_COMMIT"
```

It writes `{"version", "tag", "commit"}` for the latest published release,
or `null` when there is none, it cannot be verified, or the deployed commit
predates it (a rollback).

- `gh` authentication stays on the host. No token reaches the image or
  the frontend.
- `release.commit` is the commit the tag points to, resolved with
  `rev-list` so annotated tags are dereferenced.
- If `gh` is unavailable or fails, write `null`. Don't guess.

The application reads this file at request time, and a missing or invalid
file means `release: null`. Publishing a release then only requires
re-running the script, with no rebuild (SKILL.md section 6.3). If the
application can't read a runtime file, bake the release into the build
instead and document that publishing a release requires a redeploy.

## Docker / Docker Compose

In the Dockerfile's final stage, declare the build arguments as late as
possible (after dependency installation) so the layer cache survives:

```dockerfile
ARG GIT_COMMIT
ARG BUILD_TIME
ENV GIT_COMMIT=$GIT_COMMIT BUILD_TIME=$BUILD_TIME
LABEL org.opencontainers.image.revision=$GIT_COMMIT \
      org.opencontainers.image.created=$BUILD_TIME
```

Compose:

```yaml
build:
  args:
    GIT_COMMIT: ${GIT_COMMIT:?GIT_COMMIT not set; run scripts/build-env.sh}
    BUILD_TIME: ${BUILD_TIME:-}
environment:
  APP_ENV: production          # or from the existing env file
volumes:
  - ./build-meta:/app/build-meta:ro   # contains release-info.json
```

- The `:?` syntax makes Compose fail when the SHA is missing, which
  satisfies SKILL.md section 9.
- Mount a **directory**, not a single file. Bind-mounting a file that
  doesn't exist yet makes Docker create a directory in its place.
- Deploy script: `set -a; eval "$(scripts/build-env.sh)"; set +a`, then
  the project's usual `docker compose … build/up` with its required
  flags (such as `--env-file` or `-f`).
- Don't store `GIT_COMMIT` in `.env` files, because it changes on every
  build.
- Verify the image with `docker inspect --format '{{ index .Config.Labels "org.opencontainers.image.revision" }}' <image>`.
- Worktree-based preview/production setups: run the capture script in the
  worktree being built.

## Next.js (App Router)

- `lib/build-info.ts` (server-only) reads `process.env.GIT_COMMIT`,
  `BUILD_TIME`, `APP_ENV`, `REPO_URL` and `REPO_VISIBILITY`, and reads
  `release-info.json` from a configured path. It returns the schema object
  plus a label helper.
- `app/build-info.json/route.ts` is dynamic, so it is evaluated per request.
  Return `Response.json(info, { headers: { "Cache-Control": "no-cache" } })`.
  Exclude this path from the `middleware.ts` matcher if middleware enforces
  auth.
- Render the footer from a server component in the shared layout. Pick the
  label variant using the existing session or role check (SKILL.md section
  4.1).
- With `output: "standalone"`, environment variables are read at runtime,
  so `GIT_COMMIT` must be set as `ENV` in the runtime stage. Avoid
  `NEXT_PUBLIC_*` unless a client component truly needs it.

## FastAPI / Python

- `app/build_info.py` reads environment variables once and re-reads
  `release-info.json` on each call (or caches it by file mtime).
- Expose the route with `@app.get("/build-info.json", include_in_schema=False)`,
  returning `JSONResponse(info, headers={"Cache-Control": "no-cache"})`,
  and keep it outside auth dependencies.
- For Jinja2 templates, expose the label helper as a template global and
  render it in the base template's footer.

## Static sites (Caddy / nginx)

- At deploy time, write `build-info.json` into the published directory,
  combining the capture script's output with the contents of
  `release-info.json`. Write it last, through a temporary file and `mv`, so
  a partial deploy never advertises the new commit.
- Allow revalidation by setting `Cache-Control: no-cache` on that path.
  Caddy: `header /build-info.json Cache-Control "no-cache"`.
- A small inline script fetches `/build-info.json` and fills the footer.
  This is acceptable only because there is no server rendering.

## SPA bundles and cache mismatch

If the SHA is also embedded in the JS bundle, the cached client code and
the server's `/build-info.json` can disagree after a deploy. Either have
the UI display only the fetched metadata, or compare the two and treat a
mismatch as "new version available". Check service workers for caching
rules that would pin an old `build-info.json`.

## GitHub Actions (only if the app already builds in CI)

Use `${{ github.sha }}` as `GIT_COMMIT`. The release script needs tags, so
check out with `fetch-depth: 0`. Do not add a step that creates releases
during deployment.

## Links

- Commit: `${REPO_URL}/commit/<full sha>`
- Release: `${REPO_URL}/releases/tag/<tag>`
- Only render links when `REPO_URL` is set, and respect the private
  repository visibility rules in SKILL.md section 4.
- External links use `target="_blank" rel="noopener noreferrer"`, or follow
  the application's existing convention.
