# Test Portfolio Strategy for Coding Agents

기준일: 2026-10-05

이 문서는 AI coding agent가 테스트를 무작정 추가하지 않고 변경 위험도와 검증 비용에 따라 테스트 계층을 선택하도록 하기 위한 연구 노트다.

## 1. 핵심 문제

좋은 테스트 전략은 테스트 개수를 최대화하는 것이 아니다.

목표는 다음 세 가지를 동시에 만족하는 것이다.
- 빠른 피드백
- 실제 결함 탐지
- 유지 가능한 테스트 비용

Agent가 변경마다 E2E를 추가하면 품질보다 CI 시간이 먼저 무너질 수 있다.

## 2. 변경 위험도 기반 테스트 선택

### Low Risk
- pure function
- formatter
- mapper
- local validation

권장:
- unit
- property-based test

### Medium Risk
- service business rule
- repository query
- serialization
- external API client
- transaction boundary

권장:
- unit
- integration
- contract
- 필요 시 mutation

### High Risk
- authentication / authorization
- payment / billing
- data migration
- concurrency
- distributed workflow
- cross-service orchestration
- user-critical end-to-end flow

권장:
- unit/integration
- contract
- mutation/fault injection
- critical E2E
- held-out acceptance

## 3. Change Surface 기반 테스트

Agent는 변경된 파일 수만 보지 않고 영향 범위를 판단해야 한다.

예:
- Controller만 변경: API contract와 authorization 우선
- Service 계산 로직 변경: unit + mutation
- SQL 변경: 실제 DB integration
- Kafka producer/consumer 변경: integration + schema/contract
- 로그인/session 변경: integration + critical E2E
- 공통 라이브러리 변경: dependency graph 기반 광범위 회귀

## 4. Test Budget

테스트마다 실행비용을 가진다고 본다.

대략적인 비용 계층:

Unit < Property < Component < Integration < Contract < E2E < Cross-browser/System

Agent는 같은 결함을 더 싼 테스트 계층에서 잡을 수 있으면 비싼 테스트를 추가하지 않는다.

## 5. PR / Main / Nightly 분리

### PR Fast Gate
- changed unit tests
- affected integration tests
- contract tests
- critical E2E smoke
- static anti-pattern checks

### Main Gate
- broader regression
- extended integration
- wider E2E

### Nightly
- full E2E
- browser matrix
- mutation
- fuzz
- nondeterminism perturbation
- long-running fault injection

핵심은 모든 검증을 생략하는 것이 아니라 피드백 시점을 분리하는 것이다.

## 6. Test Prioritization

전체 suite가 길다면 다음 순서로 우선 실행한다.
1. 변경 파일과 직접 연결된 테스트
2. 최근 실패 이력이 높은 테스트
3. critical business flow
4. dependency graph상 영향 범위
5. 나머지 회귀

## 7. Test Suite Growth Rule

새 테스트를 추가할 때 기존 테스트와 중복되는지 확인한다.

다음 경우 새 테스트 대신 기존 테스트 보강을 우선한다.
- 같은 input domain
- 같은 assertion 목적
- 같은 system boundary
- 동일한 regression 원인

## 8. 2026 연구와 연결

ISSTA 2026의 TestDecision 연구는 테스트 suite 생성에서 각 단계의 marginal gain을 고려하는 suite-level 관점을 강조한다.

보고된 결과:
- branch coverage +38.15~52.37%
- execution pass rate +298.22~558.88%
- vanilla base LLM 대비 bug finding +58.43~95.45%

핵심 해석:

테스트를 하나씩 독립적으로 많이 만드는 것보다 현재 suite가 무엇을 이미 검증하고 있는지 보고 다음 테스트의 추가 이득을 판단해야 한다.

## 9. Agent Rule 후보

Agent가 새 테스트를 추가하기 전에 다음을 답한다.
1. 이 테스트가 잡으려는 failure mode는 무엇인가?
2. 기존 테스트가 이미 잡고 있지 않은가?
3. 가장 낮은 테스트 계층은 무엇인가?
4. 이 테스트의 실행비용은 어느 정도인가?
5. PR마다 실행해야 하는가?
6. critical path인가?
7. mutation 또는 held-out으로 품질을 검증할 필요가 있는가?

## 10. 완료 기준

테스트 개수 증가를 성과로 보지 않는다.

다음 지표를 함께 본다.
- changed-code coverage
- mutation score
- regression reproduction
- flaky rate
- p50/p95 test duration
- suite wall-clock time
- E2E critical-path count
- duplicated test ratio
- held-out pass rate