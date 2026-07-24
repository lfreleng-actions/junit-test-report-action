<!--
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: 2026 The Linux Foundation
-->

# 🧪 JUnit Test Report Action

<!-- prettier-ignore-start -->
<!-- markdownlint-disable-next-line MD013 -->
[![Linux Foundation](https://img.shields.io/badge/Linux-Foundation-blue)](https://linuxfoundation.org/) [![Source Code](https://img.shields.io/badge/GitHub-100000?logo=github&logoColor=white&color=blue)](https://github.com/lfreleng-actions/junit-test-report-action) [![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0) [![pre-commit.ci status badge]][pre-commit.ci results page] [![OpenSSF Scorecard](https://api.scorecard.dev/projects/github.com/lfreleng-actions/junit-test-report-action/badge)](https://scorecard.dev/viewer/?uri=github.com/lfreleng-actions/junit-test-report-action)
<!-- prettier-ignore-end -->

Summarise JUnit XML test results in the job summary, report counts through
action outputs, and upload the matched report files as a workflow artefact.

The action parses the JUnit XML format rather than a specific build tool, so
it serves Maven (Surefire and Failsafe) and Gradle alike. The default paths
match the report directories that both tools produce, so most callers add the
action as a step without further configuration.

## Usage

<!-- markdownlint-disable MD046 -->

```yaml
steps:
  - name: "Build and test"
    run: mvn --batch-mode verify

  - name: "Summarise test results"
    if: always()
    uses: lfreleng-actions/junit-test-report-action@main
```

<!-- markdownlint-enable MD046 -->

Run the action with `if: always()` so it reports results even when the build
step fails.

## Inputs

<!-- markdownlint-disable MD013 -->

| Name            | Required | Default             | Description                     |
| --------------- | -------- | ------------------- | ------------------------------- |
| report-paths    | False    | Maven + Gradle dirs | Globs matching JUnit XML files  |
|                 |          |                     | (whitespace or newline          |
|                 |          |                     | separated)                      |
| summary         | False    | true                | Write a results table to the    |
|                 |          |                     | job summary                     |
| artifact-upload | False    | true                | Upload the matched files as an  |
|                 |          |                     | artefact                        |
| artifact-name   | False    | ""                  | Artefact name (defaults to      |
|                 |          |                     | `junit-test-reports-<job>`)     |
| fail-on-failure | False    | false               | Exit non-zero when results hold |
|                 |          |                     | failures or errors              |

<!-- markdownlint-enable MD013 -->

The default `report-paths` covers `**/target/surefire-reports/*.xml`,
`**/target/failsafe-reports/*.xml`, and `**/build/test-results/**/*.xml`.
Override it to point at a bespoke report location.

## Outputs

<!-- markdownlint-disable MD013 -->

| Name          | Description                                             |
| ------------- | ------------------------------------------------------- |
| total         | Total number of tests across all report files           |
| passed        | Number of tests that passed                             |
| failures      | Number of failed tests                                  |
| errors        | Number of tests that errored                            |
| skipped       | Number of skipped tests                                 |
| time          | Total test execution time in seconds                    |
| report_count  | Number of JUnit XML report files parsed                 |
| result        | Result: success, failure, or none                       |
| artifact_name | Name of the uploaded artefact (empty when not uploaded) |

<!-- markdownlint-enable MD013 -->

## Implementation Details

- The action sums the `tests`, `failures`, `errors`, `skipped`, and `time`
  attributes of every opening `<testsuite>` tag, and skips the
  `<testsuites>` wrapper so aggregated totals never count twice.
- A parser that treats `<` as the record separator reads attributes that
  wrap across lines, and a leading-whitespace match stops an attribute such
  as `runtime` from matching `time`.
- When no report files match, the action reports `result=none` with zero
  counts and exits zero, so a job that skips tests stays green.
- `fail-on-failure` turns a run that holds failures or errors into a failed
  step. The build step gates the job in most workflows, so the default
  leaves this behaviour off and keeps the action a pure reporter.
- The shell step needs Bash 4 or newer for `globstar` and associative
  arrays. GitHub-hosted runners include it; on macOS or self-hosted
  runners with older Bash the step stops with a clear error message.

## Notes

Embed the action as a reporting step inside a build workflow. Because the
default paths already match Surefire, Failsafe, and Gradle output, the same
step works for Maven and Gradle projects without change.

[pre-commit.ci results page]: https://results.pre-commit.ci/latest/github/lfreleng-actions/junit-test-report-action/main
[pre-commit.ci status badge]: https://results.pre-commit.ci/badge/github/lfreleng-actions/junit-test-report-action/main.svg
