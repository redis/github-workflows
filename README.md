# github-workflows

Reusable GitHub Actions workflows and composite actions for Redis Java projects.

## Workflows

### `build.yml`

Reusable Gradle build workflow.

Inputs:

- `split-jobs`: run separate lint, unit test, integration test, and package jobs
- `gradle-tasks`: legacy single-job task list
- `coverage-results-path`: optional JaCoCo XML report glob to publish from the legacy single-job path
- `lint-gradle-tasks`: task list for lint and static analysis
- `test-gradle-tasks`: task list for the default JVM test suite, typically `test`
- `integration-gradle-tasks`: task list for additional JVM test suites, typically `integrationTest`
- `package-gradle-tasks`: task list for packaging and assembly verification
- `unit-coverage-results-path`: optional JaCoCo XML report glob to publish from the unit test job
- `integration-coverage-results-path`: optional JaCoCo XML report glob to publish from the integration test job
- `coverage-artifact-prefix`: optional prefix for uploaded coverage artifacts when coverage publishing is enabled
- `sonar-gradle-tasks`: optional SonarQube Cloud Gradle task list appended to `gradle-tasks` in the legacy single-job path when the `sonar-token` secret is provided
- `sonar-project-key`: optional SonarQube Cloud project key
- `sonar-organization`: optional SonarQube Cloud organization key
- `sonar-host-url`: SonarQube Cloud host URL, defaulting to `https://sonarcloud.io`
- `sonar-region`: optional SonarQube Cloud region
- `sonar-quality-gate-wait`: optional value for `sonar.qualitygate.wait`
- `sonar-coverage-report-paths`: optional comma-separated JaCoCo XML report paths passed to the scanner

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
      unit-coverage-results-path: ''
      integration-coverage-results-path: 'build/reports/jacoco/jacocoRootReport/jacocoRootReport.xml'
```

When a coverage path is configured for a split test job, the reusable workflow will:

- publish a JaCoCo summary in the job output
- upload the matching XML plus any adjacent JaCoCo HTML report as an artifact

This is intended for repositories that already generate coverage as part of an existing test job and want to avoid a separate coverage-only workflow job that reruns tests.

## SonarQube Cloud

For SonarQube Cloud, prefer the legacy single-job path so the Gradle scanner runs in the same workspace and invocation as compilation, tests, and coverage generation. This follows SonarSource's recommended Gradle pattern and avoids a second test run solely for analysis.

Example:

```yaml
jobs:
  build:
    uses: redis/github-workflows/.github/workflows/build.yml@main
    with:
      split-jobs: false
      java-version: '17'
      gradle-tasks: --parallel test jacocoTestReport integrationTest jacocoIntegrationTestReport assemble
      sonar-gradle-tasks: sonar
      coverage-results-path: |
        **/build/reports/jacoco/test/jacocoTestReport.xml
        **/build/reports/jacoco/jacocoIntegrationTestReport/jacocoIntegrationTestReport.xml
      sonar-coverage-report-paths: '**/build/reports/jacoco/test/jacocoTestReport.xml,**/build/reports/jacoco/jacocoIntegrationTestReport/jacocoIntegrationTestReport.xml'
    secrets:
      sonar-token: ${{ secrets.SONAR_TOKEN }}
```

If `sonar-gradle-tasks` is set but `sonar-token` is not provided, the workflow emits a notice and runs only `gradle-tasks`.

## Recommended Gradle Test Layout

For Gradle projects, prefer the built-in JVM test suites API instead of keeping integration coverage inside `src/test`.

Default suite:

- `src/test/java`
- task: `test`

Additional suite:

- `src/integrationTest/java`
- task: `integrationTest`

In the caller repository, update `build.gradle` or `build.gradle.kts` so Gradle actually defines the extra suite before enabling `integration-gradle-tasks`.

Typical caller-side `build.gradle` change:

```groovy
plugins {
    id 'java-library'
}

testing {
    suites {
        test {
            useJUnitJupiter()
        }

        integrationTest(JvmTestSuite) {
            useJUnitJupiter()

            dependencies {
                implementation project()
                // Additional suites do not inherit testImplementation/testRuntimeOnly.
                // Add the libraries your integration tests actually use here.
                implementation 'org.assertj:assertj-core'
                implementation 'org.testcontainers:junit-jupiter'
                implementation 'org.testcontainers:testcontainers'
                runtimeOnly 'org.junit.platform:junit-platform-launcher'
            }

            targets {
                all {
                    testTask.configure {
                        shouldRunAfter(test)
                        systemProperty 'io.lettuce.core.epoll', 'false'
                        systemProperty 'io.lettuce.core.kqueue', 'false'
                        systemProperty 'io.lettuce.core.iouring', 'false'
                    }
                }
            }
        }
    }
}

tasks.named('check') {
    dependsOn tasks.named('integrationTest')
}
```

What this caller-side change does:

- creates the `integrationTest` task and the `src/integrationTest/java` source set
- gives the new suite its own dependencies instead of reusing `testImplementation`
- keeps unit tests in `test` and slower coverage in `integrationTest`
- includes `integrationTest` in `check` and therefore in `build`

If the project already configures the default `test` task, keep that configuration and make sure the same JVM settings are applied to `integrationTest`. One way is to configure every `Test` task:

```groovy
tasks.withType(Test).configureEach {
    useJUnitPlatform()
    systemProperty 'io.lettuce.core.epoll', 'false'
    systemProperty 'io.lettuce.core.kqueue', 'false'
    systemProperty 'io.lettuce.core.iouring', 'false'
}
```

Move tests in the caller repository like this:

- keep fast, isolated tests in `src/test/java`
- move Docker, Testcontainers, network, and cross-process tests to `src/integrationTest/java`
- add any integration-only fixtures under `src/integrationTest/resources`

Once the caller Gradle build defines `integrationTest`, map the workflow inputs like this:

- `test-gradle-tasks: --parallel test`
- `integration-gradle-tasks: --parallel integrationTest`

## Migration Guidance For Consuming Repositories

1. Update the caller repo's `build.gradle` to define an `integrationTest(JvmTestSuite)`.
2. Move slower tests from `src/test` to `src/integrationTest`.
3. Keep any shared `Test` task configuration applied to both `test` and `integrationTest`.
4. Enable `split-jobs: true` in the caller workflow.
5. Wire `test-gradle-tasks` to `test` and `integration-gradle-tasks` to `integrationTest`.
6. If the caller repo does not yet have a separate integration suite, set `integration-gradle-tasks: ''` until the Gradle layout is updated.

## Notes

- The reusable workflow does not create `integrationTest` automatically. The caller repository must define the suite in Gradle.
- Do not use `check` as the lint bucket unless you want tests to run there too.
