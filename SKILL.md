---
name: app-build-info
description: >
  Implement standardized build and release identification in web applications.
  Use when adding or improving application version displays, Git commit
  identifiers, deployment metadata, release tracking, or build-information
  endpoints. Supports preview and production environments, private GitHub
  repositories, and deployments that occur independently of formal releases.
---

# Application Build and Release Identification

## Purpose

Implement a consistent, lightweight, and reusable system for identifying
deployed versions of web applications.

The system must distinguish between:

1. **Release version:** The latest formally published application release.
2. **Build identifier:** The exact Git commit used to build the deployed application.
3. **Environment:** The deployment environment, such as production or preview.

These identifiers serve different purposes and must remain independent.

The system should provide reliable deployment traceability without requiring
formal releases for every production deployment.

Favor simplicity, automation, maintainability, and minimal dependencies.

---

## 1. Core Principles

### 1.1 Every deployment must be identifiable

Every deployed application must expose the Git commit SHA corresponding to
the source revision used to build it.

The commit SHA is the primary technical identifier.

Requirements:

- Obtain the SHA automatically during the build or deployment process.
- Preserve the full SHA internally.
- Display the first 7 characters by default.
- Never require developers to update commit identifiers manually.
- Ensure the SHA corresponds to the actual deployed source revision.
- Do not use timestamps or sequential build numbers as the primary identifier.
- Do not generate artificial commit identifiers.

The same commit can be deployed to multiple environments.

Consequently, a commit SHA identifies a source revision rather than a unique
deployment event or compiled artifact.

### 1.2 Releases and deployments are independent

Applications may be deployed to production multiple times between formal
GitHub releases.

This is expected behavior.

A production deployment must not:

- Automatically create a GitHub release.
- Automatically increment the semantic version.
- Require a new Git tag.
- Modify formal release information unless explicitly instructed.

Formal releases are milestones published deliberately by the developer.

### 1.3 Keep release versions simple

Use conventional semantic versions for formal releases:

Examples:

- `v1.0.0`
- `v1.1.0`
- `v2.0.0`

Do not append deployment counters, commit distances, or automatically
generated build numbers to these versions.

In particular, avoid formats such as:

- `v1.0.0+2`
- `v1.0.0+15`
- `v1.0.0-2-g05f2e39`
- `v1.0.0-build.42`

The `+N` suffix must never be used to indicate the number of commits since
the latest release.

The displayed release version must remain unchanged until a new formal
release is published.

Build identification is handled separately using the commit SHA.

### 1.4 Do not invent release versions

If the application has no formally published release, do not invent one
solely for display purposes.

For example:

```text
Build 05f2e39
```

Do not automatically display `v0.0.0`, `v0.1.0`, or another placeholder
unless that version has genuinely been established as a formal release.

---

## 2. Version and Build Display

### 2.1 Production environment

When a formal release exists, the preferred footer format is:

```text
v1.0.0 · 05f2e39
```

Where:

- `v1.0.0` is the latest published release.
- `05f2e39` is the currently deployed Git commit.

If production contains changes introduced after the latest formal release,
the displayed release version remains unchanged while the commit SHA changes.

Example:

```text
Initial release:

v1.0.0 · a83f21c

Subsequent deployment:

v1.0.0 · b72c415

Another deployment:

v1.0.0 · 05f2e39

New formal release:

v1.1.0 · 05f2e39
```

The final example assumes that the new release tag points to the same
source revision already deployed.

If release metadata is updated independently, changing the displayed
release version must not require rebuilding the application code.

### 2.2 Preview environment

Preview deployments should clearly identify their environment.

Preferred format:

```text
Preview · 05f2e39
```

Preview environments do not need to display the latest formal release.

Do not automatically assign prerelease semantic versions such as
`v1.1.0-beta` unless the project explicitly uses such a release process.

### 2.3 Production without a formal release

When no formal release exists:

```text
Build 05f2e39
```

### 2.4 Formatting conventions

Use these conventions consistently:

- Prefix formal releases with `v`.
- Display the first 7 characters of the commit SHA.
- Separate the release and build identifiers using ` · `.
- Do not include commit counts.
- Do not include `+N` build metadata.
- Do not display timestamps in the default footer.
- Avoid redundant labels such as `Version:` or `Commit:` in the compact UI.
- Keep the presentation consistent across applications.

---

## 3. User Interface

### 3.1 Placement

Integrate the identification information unobtrusively into the existing
application interface.

Preferred locations, in order:

1. Existing application footer.
2. Application information or About section.
3. Settings or administration interface.

Do not introduce a new persistent navigation item solely for build
information.

Do not add unnecessary banners, badges, dialogs, or visual distractions.

### 3.2 Visual design

The identification element should:

- Use existing typography and design tokens.
- Use muted text colors.
- Remain visually secondary to primary application content.
- Respect the application's existing spacing and alignment.
- Work on desktop and mobile layouts.
- Meet applicable accessibility requirements.

Avoid introducing new colors or decorative components solely for
version information.

### 3.3 Additional information

Where appropriate, provide a tooltip or accessible details view.

Example:

```text
Latest release: v1.0.0
Deployed commit: 05f2e39
Environment: Production
```

The full commit SHA may be exposed in the details view.

Do not include commit counts or commit-distance indicators.

### 3.4 Links

When a GitHub repository is configured:

- The release version may link to its corresponding GitHub release.
- The commit identifier may link to its corresponding GitHub commit.
- Use the full commit SHA when constructing commit URLs.
- Use ordinary HTTPS URLs.
- Open external links using safe browser conventions.
- Ensure keyboard accessibility.

Do not make external links visually prominent.

The application must remain functional if GitHub is unavailable.

---

## 4. Private GitHub Repositories

Private repositories require special consideration.

GitHub enforces repository access permissions, so an ordinary link to a
private commit does not grant unauthorized access.

However, exposing links to inaccessible resources can create unnecessary
confusion for regular application users.

### 4.1 Recommended visibility

When the application supports user roles:

**Regular users**

Display the latest formal release:

```text
v1.0.0
```

If no formal release exists, the technical build identifier may be omitted
from the regular-user interface.

**Administrators**

Display the full compact identification:

```text
v1.0.0 · 05f2e39
```

The commit identifier may link to the corresponding GitHub commit.

**Preview environments**

Display:

```text
Preview · 05f2e39
```

### 4.2 Public repositories

For public repositories, the application may display both release and
build identifiers to all users.

### 4.3 Applications without roles

Do not introduce an authentication or authorization system merely to
control version-information visibility.

If roles do not exist, preserve the existing application architecture.

Choose an appropriate default based on the application's intended
audience and existing UI conventions.

### 4.4 Security

Never embed GitHub access tokens, credentials, or authenticated URLs
in frontend code.

Do not attempt to bypass GitHub repository permissions.

Hiding a commit link is a usability decision, not a security boundary.

If build metadata is publicly accessible, assume that the commit SHA
and any included repository information are public.

Repository visibility must be configurable rather than inferred from
the ability to access GitHub during the build.

---

## 5. Build Metadata

Every application should expose build information using a standardized
metadata structure.

Recommended schema:

```json
{
  "schemaVersion": 1,
  "app": "alias",
  "environment": "production",
  "build": {
    "sha": "05f2e39a83f21c92d4e",
    "shortSha": "05f2e39",
    "builtAt": "2026-10-18T14:30:00Z"
  },
  "release": {
    "version": "1.0.0",
    "tag": "v1.0.0",
    "commit": "a83f21c92d4e"
  }
}
```

The SHA values above are illustrative.

### 5.1 Required fields

The following fields are required:

- `schemaVersion`
- `app`
- `environment`
- `build.sha`
- `build.shortSha`

### 5.2 Optional fields

The following fields are optional:

- `build.builtAt`
- `release`
- Repository URL
- Additional deployment metadata

When no formal release exists, use:

```json
{
  "schemaVersion": 1,
  "app": "alias",
  "environment": "production",
  "build": {
    "sha": "05f2e39a83f21c92d4e",
    "shortSha": "05f2e39"
  },
  "release": null
}
```

Do not populate optional fields with fabricated data.

### 5.3 Metadata generation

Generate build metadata automatically.

Prefer:

- Build-time environment variables.
- Git information available during the build.
- CI/CD-provided commit identifiers.
- Explicit release metadata maintained by the release workflow.

Do not require the production application to execute Git commands.

Do not require the `.git` directory to exist in the deployed container.

Do not make runtime requests to GitHub merely to determine build or
release information.

### 5.4 Release metadata integrity

A Git tag is not necessarily equivalent to a published GitHub release.

Do not assume that:

- The nearest reachable Git tag is the latest published release.
- The highest semantic-version tag has necessarily been published.
- The package version matches the latest published release.
- The deployed commit is itself tagged.

Use a reliable, explicit release metadata source.

If the latest published release cannot be established reliably, omit
the release information rather than guessing.

The release identifier must refer to a real published release.

---

## 6. Build Information Endpoint

When compatible with the application architecture, expose metadata at:

```text
/build-info.json
```

For example:

```text
https://example.com/build-info.json
```

Use the same JSON schema across applications.

### 6.1 Preferred implementation

For applications that serve static assets, prefer generating a JSON
file during the build or deployment process.

For server-rendered applications or applications with an existing
backend API, an equivalent endpoint is acceptable.

Choose the simplest approach compatible with the current architecture.

### 6.2 Caching

Build metadata must not become indefinitely stale.

Configure appropriate caching behavior.

Consider:

- Browser caches.
- Reverse proxies.
- CDNs.
- Service workers.
- Existing application caching strategies.

When possible, use HTTP caching mechanisms that allow clients to
revalidate metadata.

Ensure that the displayed information corresponds to the currently
served application build.

If frontend assets and metadata are deployed separately, account for
possible mismatches between cached client code and server metadata.

### 6.3 Updating release information

The system should support updating release metadata independently of
application code when practical.

For example, publishing a GitHub release corresponding to an already
deployed commit should not necessarily require a complete application
rebuild.

However, do not introduce unnecessary infrastructure solely to achieve
this.

If independent metadata updates are impractical, document the
alternative explicitly.

---

## 7. Deployment Workflow

Support the following workflow:

```text
Develop changes
       |
       v
Commit changes to Git
       |
       v
Deploy commit to preview
       |
       v
Test and approve
       |
       v
Deploy tested revision to production
       |
       +--> Build SHA updates automatically
       |
       +--> Release version remains unchanged
       |
       v
Publish GitHub release when appropriate
       |
       v
Update release metadata
```

### 7.1 Preview and production consistency

Prefer deploying the same tested commit to production.

When possible, reuse the same build artifact.

Avoid rebuilding from an unverified source revision.

Environment-specific configuration should not change the identity
of the source revision.

### 7.2 Build reproducibility

A commit SHA identifies a source revision, not necessarily an identical
compiled artifact.

Different build-time configuration or dependencies may produce
different artifacts from the same commit.

Do not claim that commit identification alone guarantees reproducible
builds.

### 7.3 Deployment automation

Integrate metadata generation into the existing deployment mechanism.

Possible environments include:

- Docker and Docker Compose.
- GitHub Actions.
- Node.js build scripts.
- Vite.
- Next.js.
- Other frontend or backend frameworks.

Do not introduce a new CI/CD platform merely to implement this skill.

Do not automatically create GitHub releases during deployment.

---

## 8. Reusability Across Applications

The system should work across different web applications with minimal
project-specific modifications.

### 8.1 Standardize the contract

Use consistent:

- Metadata field names.
- Version formatting.
- Commit identifier formatting.
- Environment naming.
- Endpoint conventions.
- Fallback behavior.

### 8.2 Adapt to existing architecture

Before implementation, inspect:

1. Application framework.
2. Build system.
3. Deployment workflow.
4. Hosting environment.
5. Existing version metadata.
6. Existing UI components.
7. Authentication and user roles.
8. Repository visibility.

Do not assume every application uses the same technology stack.

### 8.3 Prefer existing tools

Reuse existing build tooling wherever possible.

Avoid adding dependencies for functionality already available through
Git, the build system, or the deployment environment.

### 8.4 Shared implementation

The host-side scripts are shared by copying them, not by referencing them.
Their canonical copies live in this skill's `scripts/` directory:

- [scripts/build-env.sh](scripts/build-env.sh): validates and prints the
  commit SHA and build time.
- [scripts/update-release-info.sh](scripts/update-release-info.sh): writes
  `release-info.json` from the latest published GitHub release.

Rules:

- Copy both files into the application's repository unchanged (for example
  into its `scripts/` directory) and commit them. Do not rewrite them in the
  application.
- Never make an application call scripts from this skill directory or from
  another shared server path. Each repository must build and deploy on its
  own, from a fresh clone or in CI.
- Each copy starts with an `app-build-info vN` header. To check for drift,
  `diff` the application's copies against the canonical ones. Update an
  application's copies as a deliberate commit in that application.
- If an application genuinely needs a change, make it in the canonical copy
  (bumping the version when its behaviour changes), then re-copy it. Report
  any application that needed a local variant instead.
- Both scripts locate the repository from their own location, so they work
  from any directory inside the checkout.

The application-side code (reading the metadata, the endpoint and the UI
label) stays per application, following
[references/stack-patterns.md](references/stack-patterns.md).

Do not create a shared package or library yet. That will be extracted
once the convention has been validated in several applications (see Step 6).

---

## 9. Reliability and Fallbacks

Build identification must never prevent the application from starting.

If metadata is unavailable at runtime, display a neutral fallback:

```text
Build unknown
```

Do not fabricate commit SHAs or release versions.

For production deployment pipelines, missing or invalid mandatory
build metadata should produce a clear diagnostic and preferably fail
the deployment before an unidentifiable build is released.

Runtime fallback behavior must not conceal deployment configuration
errors.

If GitHub is unavailable, continue displaying locally stored metadata.

If the repository URL is unavailable, display the commit identifier
without a link.

If a release cannot be verified, omit the release version.

---

## 10. Implementation Procedure

When this skill is invoked, follow these steps.

### Step 1: Inspect

Examine the existing application.

Determine:

- Framework and build system.
- Git repository configuration.
- Deployment mechanism.
- Environment configuration.
- Existing version information.
- Existing footer or About interface.
- Existing authorization model.

Also follow any project-specific deployment rules documented in the
project's `CLAUDE.md`, README, or runbooks.

### Step 2: Plan

Choose the smallest compatible implementation.

Prefer standardized metadata and minimal dependencies.

Do not introduce architectural changes unrelated to build identification.

Summarize the findings and the plan, including the files to be changed,
before editing. Confirm with the user before changing how production is
deployed.

### Step 3: Implement

Implement:

1. Automatic commit SHA detection.
2. Environment identification.
3. Optional formal release metadata.
4. Standardized metadata output.
5. User interface integration.
6. GitHub links when appropriate.

Copy the canonical scripts for items 1 and 3 from this skill (section 8.4)
rather than writing new ones.

See [references/stack-patterns.md](references/stack-patterns.md) for
integration patterns for common stacks.

### Step 4: Verify

Confirm that:

- The displayed SHA matches the deployed source revision.
- Preview and production are distinguishable.
- New deployments update the build identifier.
- Deployments do not automatically change the release version.
- Published release metadata is accurate.
- Private repository links respect the visibility conventions.
- Missing metadata is handled correctly.
- Existing application functionality remains unaffected.

Never push tags or publish GitHub releases during verification unless
the user explicitly asks.

### Step 5: Document

Document:

- How build metadata is generated.
- Where it is exposed.
- How environment identification works.
- How release metadata is updated.
- Any required deployment configuration.
- Any limitations or manual steps.

Keep documentation concise and practical.

### Step 6: Report extraction candidates

This skill is currently instruction-only. A shared library will be
extracted later, after the convention has been validated in real
applications.

While implementing, keep generic logic (SHA capture and validation,
release metadata generation, label formatting) separate from
application-specific glue (routes, layout components, deployment
scripts). At the end, report which parts appeared generic and which
required application-specific changes.

---

## 11. Acceptance Criteria

The implementation is complete when:

- [ ] Every deployment is associated with its actual Git commit.
- [ ] The commit SHA is generated automatically.
- [ ] The full SHA is preserved internally.
- [ ] The UI displays a short SHA when appropriate.
- [ ] Formal releases and deployments remain independent.
- [ ] Formal release versions use conventional semantic versioning.
- [ ] No `+N` suffix or commit-distance indicator is displayed.
- [ ] Preview and production are clearly distinguishable.
- [ ] Release metadata is not fabricated.
- [ ] The application uses the standardized metadata structure.
- [ ] Build metadata can be inspected programmatically when appropriate.
- [ ] GitHub links use the correct commit or release URL.
- [ ] Private repository visibility is handled appropriately.
- [ ] No GitHub credentials are exposed.
- [ ] Missing metadata produces a safe fallback.
- [ ] The implementation preserves the existing application architecture.
- [ ] The integration is documented.

---

## 12. Design Philosophy

This skill establishes a consistent convention for build identification
across independently developed web applications.

The primary objective is:

**Every deployment must be traceable, but not every deployment needs
to become a formal release.**

The system should remain:

- Lightweight.
- Automated.
- Framework-adaptable.
- Visually unobtrusive.
- Consistent across projects.
- Easy to maintain.

Avoid overengineering.

A reliable Git commit identifier and optional formal release version
are sufficient for most small and medium-sized web applications.
