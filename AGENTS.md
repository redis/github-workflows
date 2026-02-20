# AGENTS.md

> Reusable GitHub Actions workflows and composite actions for Redis Java projects.

## What This Repository Is

A shared library of CI/CD components consumed by other Redis repositories via:
```yaml
uses: redis/github-workflows/.github/workflows/release.yml@main
uses: redis/github-workflows/.github/actions/jreleaser@main
```

## Repository Structure

```
.github/
├── workflows/           # Reusable workflows (workflow_call)
│   ├── build.yml        # Simple Gradle build + test
│   ├── release.yml      # Full release: tag, build, JReleaser, publish
│   ├── early-access.yml # Snapshot/pre-release builds
│   └── docs.yml         # Antora documentation build + GitHub Pages
└── actions/             # Composite actions
    ├── setup-gradle/    # Java + Gradle setup with caching
    ├── configure-aws/   # AWS credentials via OIDC or static keys
    ├── jreleaser/       # GitHub release, Maven Central, Docker, Slack
    ├── create-release-tag/  # Axion-based version tagging
    ├── build-docs/      # Antora documentation builder
    ├── clone-to-dist-repo/  # Clone release to -dist repository
    └── update-antora-version/  # Sync docs/antora.yml version
```

## Workflows

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `build.yml` | `workflow_call` | Run Gradle build and tests |
| `release.yml` | `workflow_call` | Create release tag, build, publish to Maven Central |
| `early-access.yml` | `workflow_call` | Build and publish snapshot releases |
| `docs.yml` | `workflow_call` | Build Antora docs, deploy to GitHub Pages |

## Actions

| Action | Purpose |
|--------|---------|
| `setup-gradle` | Setup Java (Temurin) + Gradle with caching |
| `configure-aws` | Configure AWS credentials via OIDC (preferred) or static credentials |
| `jreleaser` | Run JReleaser for releases, signing, publishing |
| `create-release-tag` | Create Git tag using Axion release plugin |
| `build-docs` | Build Antora documentation with Algolia search |
| `clone-to-dist-repo` | Copy release to a `-dist` repository |
| `update-antora-version` | Update `docs/antora.yml` after release |

## Key Integrations

- **Gradle + Axion**: Version management via `axion-release-plugin`
- **JReleaser**: GitHub releases, Maven Central, Docker Hub, Slack
- **Antora**: Documentation site generation
- **Algolia**: Documentation search indexing

## Common Patterns

### Consuming a workflow
```yaml
# In your repo's .github/workflows/release.yml
name: Release
on:
  workflow_dispatch:
    inputs:
      version:
        description: 'Release version'
        required: false

jobs:
  release:
    uses: redis/github-workflows/.github/workflows/release.yml@main
    with:
      version: ${{ inputs.version }}
      gradle-build-tasks: 'build publish'
    secrets:
      git-access-token: ${{ secrets.GIT_ACCESS_TOKEN }}
      gpg-secret-key: ${{ secrets.GPG_SECRET_KEY }}
      # ... other secrets
```

### Required secrets (release workflow)
- `git-access-token`: GitHub PAT with repo write access
- `gpg-secret-key`, `gpg-public-key`, `gpg-passphrase`: For artifact signing
- `sonatype-username`, `sonatype-password`: Maven Central publishing
- `slack-webhook`: Release notifications
- `docker-username`, `docker-password`: Docker Hub publishing

## Making Changes

1. Changes here affect all consuming repositories
2. Test workflows in a fork before merging
3. Use `@main` for latest, or pin to a commit SHA for stability
4. Document new inputs/secrets in action.yml descriptions

## References

- [GitHub Reusable Workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows)
- [JReleaser Documentation](https://jreleaser.org/guide/latest/)
- [Axion Release Plugin](https://axion-release-plugin.readthedocs.io/)
- [Antora Documentation](https://docs.antora.org/)

