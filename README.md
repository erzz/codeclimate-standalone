# codeclimate-standalone

## Purpose

Code Climate is a great service with pricing and plans for all kinds of users. However there are cases where you may not want to send results to them, just want a quick PASS / FAIL based on simple thresholds directly within the workflow with no frills

This action produces a json report with pass / fail thresholds for the different severities of finding and optionally a readable HTML report that you can upload as a job artifact.

The action uses the container version of the codeclimate CLI and is configured to your tastes using the same configuration file and settings you would use for the full service.

## Inputs

## Outputs

## Examples

### Run a default codeclimate scan

### Run a codeclimate scan with additional HTML report

### Provide your own pass / fail thresholds
