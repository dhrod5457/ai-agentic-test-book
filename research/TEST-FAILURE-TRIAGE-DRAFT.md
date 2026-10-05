# TEST-FAILURE-TRIAGE Draft

상태: 연구 기반 초안
기준일: 2026-10-05

## On Failure

Agent MUST NOT immediately weaken or delete the failing test.

## Classification

Choose one primary class:
- PRODUCT_REGRESSION
- TEST_DEFECT
- FLAKY
- ENVIRONMENT
- EXTERNAL_DEPENDENCY
- STATE_POLLUTION
- UNRELATED_FAILURE
- HARNESS_DEFECT

## Required Evidence

- failing test/error
- relation to diff
- rerun result
- clean-environment result when needed
- historical failure evidence when available
- root-cause evidence

## Required Order

Preserve failure -> classify -> gather evidence -> choose fix target -> modify -> rerun original failure -> run affected regression.

## Flaky

Do not resolve with sleep or retry-only changes.
Identify root-cause category first.

## Environment

Do not change production behavior to accommodate broken CI infrastructure.

## Test Defect

Fix the test only if the existing assertion/setup conflicts with the intended specification.

## Product Regression

Keep or add a regression test that fails before the product fix and passes after it.

## Low Confidence

If the cause is uncertain, gather more evidence rather than making broader code changes.

## Final Report

Report:
- classification
- confidence
- evidence
- fix
- validation result