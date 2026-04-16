# github-workflows

Reusable GitHub Actions workflows and composite actions for Redis Java projects.

## Workflows

### `build.yml`

Reusable Gradle build workflow.

Inputs:

- `split-jobs`: run separate lint, unit test, integration test, and package jobs
- `gradle-tasks`: legacy single-job task list
- `lint-gradle-tasks`: task list for lint and static analysis
- `test-gradle-tasks`: task list for the default JVM test suite, typically `test`
- `integration-gradle-tasks`: task list for additional JVM test suites, typically `integrationTest`
- `package-gradle-tasks`: task list for packaging and assembly verification

When `split-jobs` is enabled, any split stage can be disabled by passing an empty string.

Example:

```yaml
name: Build

on:
  pull_request:
  workflow_dispatch:
    inputs:
      test-gradle-tasks:
        description: 'Gradle tasks to run for unit tests'
        required: false
        default: '--parallel test'
        type: string
      integration-gradle-tasks:
        description: 'Gradle tasks to run for integration tests'
        required: false
        default: '--parallel integrationTest'
        type: string

permissions:
  contents: read
  checks: write
  pull-requests: write
  id-token: write

jobs:
  build:
    uses: redis/github-workflows/.github/workflows/build.yml@main
    with:
      split-jobs: true
      java-version: '17'
      lint-gradle-tasks: ''
      test-gradle-tasks: ${{ inputs.test-gradle-tasks || '--parallel test' }}
      integration-gradle-tasks: ${{ inputs.integration-gradle-tasks || '--parallel integrationTest' }}
      package-gradle-tasks: '--parallel assemble'
```

## Recommended Gradle Test Layout

For Gradle projects, prefer the built-in JVM test suites API instead of keeping integration coverage inside `src/test`.

Default suite:

- `src/test/java`
- task: `test`

Additional suite:

- `src/integrationTest/java`
- task: `integrationTest`

Example Gradle configuration:

```groovy
testing {
    suites {
        test {
            useJUnitJupiter()
        }

        integrationTest(JvmTestSuite) {
            useJUnitJupiter()

            dependencies {
                implementation project()
            }

            targets {
                all {
                    testTask.configure {
                        shouldRunAfter(test)
                    }
                }
            }
        }
    }
}
```

This layout maps cleanly to the reusable workflow:

- `test-gradle-tasks: --parallel test`
- `integration-gradle-tasks: --parallel integrationTest`

## Migration Guidance For Consuming Repositories

1. Keep fast unit tests in `src/test`.
2. Move slower Docker, Testcontainers, network, or cross-process coverage into `src/integrationTest`.
3. Define an `integrationTest` JVM test suite in Gradle.
4. Enable `split-jobs: true` in the consuming workflow.
5. Wire `test-gradle-tasks` to `test` and `integration-gradle-tasks` to `integrationTest`.
6. If the project does not yet have a separate integration suite, set `integration-gradle-tasks: ''` to skip that job until the Gradle layout is updated.

## Notes

- The reusable workflow does not create `integrationTest` automatically. The consumer repository must define the suite in Gradle.
- Do not use `check` as the lint bucket unless you want tests to run there too.
