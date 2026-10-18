# app-build-info

A shared convention for identifying what a web app is running: the exact Git
commit, the latest published release, and the environment. Every deployment
is traceable, but not every deployment has to become a release.

An app that follows it:

- serves `GET /build-info.json` with a fixed schema (environment, commit SHA,
  build time, latest published release or `null`);
- shows a compact footer label such as `v1.0.0 · 05f2e39`, `Build 05f2e39` or
  `Preview · 05f2e39`, never commit counts like `v1.0.0+2`;
- takes its release from the latest published GitHub release, read at runtime,
  so publishing a release needs no rebuild.

## Contents

- [SKILL.md](SKILL.md): the specification, written as a
  [Claude Code skill](https://docs.claude.com/en/docs/claude-code/skills).
- [references/stack-patterns.md](references/stack-patterns.md): integration
  patterns for Docker/Compose, Next.js, FastAPI, static sites and GitHub Actions.
- [scripts/](scripts/): the canonical host-side scripts.
  - `build-env.sh` prints the validated commit SHA and build time.
  - `update-release-info.sh` writes `release-info.json` from the latest
    published release.

## Using it

As a Claude Code skill, clone it into your skills directory:

```sh
git clone https://github.com/Marcos-A/app-build-info ~/.claude/skills/app-build-info
```

Apps copy the scripts unchanged into their own repository (each copy carries an
`app-build-info vN` header) rather than calling them from a shared location, so
every repository still builds from a fresh clone. Check for drift with `diff`.
