# codeclimate-standalone

[![Tests](https://github.com/erzz/codeclimate-standalone/actions/workflows/tests.yml/badge.svg)](https://github.com/erzz/codeclimate-standalone/actions/workflows/tests.yml)

## Purpose

Code Climate is a great service with pricing and plans for all kinds of users. However there are cases where you may not want to send results to an external service or just want a quick PASS / FAIL based on simple thresholds directly within the workflow with no frills

This action produces a json report with pass / fail thresholds for the different severities of finding and optionally a readable HTML report that you can upload as a job artifact.

The action uses the container version of the codeclimate CLI and is configured to your tastes using the same configuration file and settings you would use for the full service.

## Code Climate Configuration

Code Climate has a comprehensive ability to configure via .codeclimate.yml at the root of your project (or using a custom path with this action - see inputs below).

Although a configuration is not required (Code Climate will attempt to discover languages used and apply some standard rules), it is highly recommended you provide a configuration that suits your needs as it will provide more satisfactory results and speed up execution as the job will not need to try and discover languages etc.

For details of Code Climate configuration see:

- https://docs.codeclimate.com/docs/default-analysis-configuration
- https://docs.codeclimate.com/docs/advanced-configuration

## Available Inputs

None of the inputs are currently mandatory!

| Input                | Default          | Details                                                                                     |
| -------------------- | ---------------- | ------------------------------------------------------------------------------------------- |
| `config_file`        | .codeclimate.yml | Optional relative path to custom location of Code Climate config file (must be yaml format) |
| `html_report`        | false            | Set to true if you wish to also have an HTML format report produced                         |
| `info_threshold`     | 0                | The number of findings of severity INFO allowed before the job returns a failure            |
| `minor_threshold`    | 0                | The number of findings of severity MINOR allowed before the job returns a failure           |
| `major_threshold`    | 0                | The number of findings of severity MAJOR allowed before the job returns a failure           |
| `critical_threshold` | 0                | The number of findings of severity CRITICAL allowed before the job returns a failure        |
| `blocker_threshold`  | 0                | The number of findings of severity BLOCKER allowed before the job returns a failure         |

## Outputs

Some simple outputs are provided for use in later steps / jobs

| Output              | Details                                     |
| ------------------- | ------------------------------------------- |
| `info_findings`     | The number of findings of severity INFO     |
| `minor_findings`    | The number of findings of severity MINOR    |
| `major_findings`    | The number of findings of severity MAJOR    |
| `critical_findings` | The number of findings of severity CRITICAL |
| `blocker_findings`  | The number of findings of severity BLOCKER  |

## Examples

### Run a default codeclimate scan

The main thing to ensure is that you **MUST** checkout your code in a preceding step otherwise there would be nothing to scan!

If you place your `.codeclimate.yml`at the root of your project then no further configuration is required by default

```yaml
jobs:
  code-quality:
    name: Code Climate Standalone
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Run Code Climate
        uses: erzz/codeclimate-standalone@v0
```

### Provide your own pass / fail thresholds

```yaml
jobs:
  code-quality:
    name: Code Climate Standalone
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Run Code Climate
        uses: erzz/codeclimate-standalone@v0
        with:
          info_threshold: 10
          minor_threshold: 5
          major_threshold: 1
          critical_threshold: 0
          blocker_threshold: 0
```

### Run a codeclimate scan with additional HTML report

There are some limitations with the CLI in that it is not possible to generate two reports from a single scan (AFAIK!). So the first execution will produce a json report which is easier to parse for a pass/fail result.

This action provides the option `html_report` (defaults to false) to enable a second scan to be executed that produces an additional, much more readable, HTML report which you can upload as an artifact for the developer to use when there are findings.

The second execution does mean the job takes a little longer, but not by much. Most of the time in the first execution is the Code Climate CLI pulling the various docker images it needs and setting up. As the images are then already pulled by the time a second execution starts - rerunning the scan for the HTML report typically only added 10-20s

In basic testing of a tiny project (this one!) execution time is typically

- ~1m 50s without HTML report
- ~2m 10s with HTML report

```yaml
jobs:
  code-quality:
    name: Code Climate Standalone
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Run Code Climate
        uses: erzz/codeclimate-standalone@v0
        with:
          html_report: true

      - name: Upload Report
        uses: actions/upload-artifact@v2
        if: always()
        with:
          name: Code Climate Report
          path: codeclimate-report.html
```
