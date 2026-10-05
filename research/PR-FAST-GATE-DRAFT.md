# PR Fast Gate Draft

상태: 연구 기반 초안
기준일: 2026-10-05

## 목표

Agent 변경에 대해 가장 위험한 실패를 가장 짧은 시간 안에 발견한다.

## Input
- git diff
- changed symbols
- dependency graph
- test-to-code map
- test duration history
- test failure history

## Selection

1. 직접 연관 테스트
2. dependency 영향 테스트
3. 최근 실패 가능성이 높은 테스트
4. critical business/security tests
5. changed-code coverage gap을 메우는 테스트

## Mandatory Escalation

다음 변경은 wider regression으로 자동 승격한다.
- security/auth
- common library
- schema/migration
- global config
- dependency upgrade
- shared serialization
- framework wiring

## Gate

Integrity -> Compile -> Selected Unit -> Selected Integration -> Changed-Code Coverage -> Contract/Security -> Critical E2E

## Deferred

Main/Nightly:
- broad regression
- full E2E
- mutation
- fuzz
- flaky perturbation

## Metrics

- PR fast-gate wall clock
- selected / total tests
- changed-line coverage
- changed-branch coverage
- selection miss rate
- post-merge regression escape rate
- p50/p95 selected test duration

## Safety Rule

selection confidence가 낮으면 테스트를 줄이지 않는다.

Fast Gate의 목적은 green을 빨리 만드는 것이 아니라 red를 빨리 발견하는 것이다.