# Test Impact Analysis and PR Fast Gate

기준일: 2026-10-05

이 문서는 AI coding agent가 만든 변경에 대해 전체 테스트를 매번 실행하지 않고도 빠른 피드백과 회귀 탐지력을 유지하는 방법을 연구한다.

## 1. 핵심 질문

PR 하나가 올라올 때 어떤 테스트를 먼저 실행해야 하는가?

나쁜 두 극단이 있다.

1. 모든 PR에서 모든 테스트를 항상 실행한다.
2. 변경된 파일과 직접 연결된 테스트만 실행한다.

첫 번째는 느리고 비싸다.
두 번째는 간접 영향과 공통 라이브러리 변경을 놓칠 수 있다.

따라서 목표는 다음 구조다.

diff -> change surface -> dependency impact -> ranked test set -> fast gate -> broader regression

## 2. 2026 Google Regression Test Selection at Scale

Google Research의 2026 연구는 대규모 core library 변경의 regression test selection을 다룬다.

대상:
- high-impact core libraries: 10
- 평균 test suite: 약 220,000 tests

문제:
core library의 작은 변경도 수많은 reverse dependency에 영향을 줄 수 있다. 그러나 pre-submit 단계에서 모든 downstream test를 실행하는 것은 현실적으로 비싸다.

접근:
- hybrid call graph
- per-commit dependency candidates
- graph structural features
- ML ranking
- fixed test budget

결과:
- commit당 2,000 tests 선택
- 전체 실행 비용의 약 0.9%
- failing commit의 약 40% regression을 pre-submit에서 탐지

### 책에서의 의미

테스트 선택은 단순 file-name matching이 아니라 dependency graph와 failure likelihood를 이용한 ranking 문제로 볼 수 있다.

중요한 점은 0.9%의 테스트로 모든 결함을 잡았다는 뜻이 아니다.
Fast Gate에서 높은 위험의 회귀를 먼저 잡고 이후 broader regression이 보완하는 계층 구조가 필요하다.

## 3. Change And Cover (ChaCo), ICSE 2026

ChaCo는 전체 repository coverage가 아니라 PR에서 실제 변경된 코드의 patch coverage gap을 겨냥한다.

대상:
- 145 PRs
- SciPy
- Qiskit
- Pandas

결과:
- 30%의 PR을 full patch coverage로 개선
- 평균 비용 약 $0.11 per PR
- human reviewer 평가
  - worth adding: 4.53/5
  - well integrated: 4.20/5
  - relevant: 4.70/5
- test context 제공 시 coverage 약 2배
- 제출한 12개 테스트 중 8개 merge
- previously unknown bugs 2개 발견 및 수정

### 책에서의 의미

Agent가 테스트를 추가할 때 전체 coverage 숫자를 올리는 것보다 변경된 코드의 uncovered branch/line을 먼저 찾아야 한다.

즉:

repository coverage -> changed-code / patch coverage

로 중심 지표를 바꿀 필요가 있다.

## 4. AI-generated PR의 Changed-Code Coverage

MSR 2026 연구는 AIDev v2의 2,314개 Python PR을 분석했다.

modified tests를 실행했을 때 변경된 non-test source line의 diff-coverage:
- AI-Only PR 평균: 19.96%
- Coauthor: 17.35%
- Human: 13.09%

non-zero diff coverage:
- AI-Only: 80.4%
- Coauthor: 62.3%
- Human: 66.2%

assertion quality 분석에서는 functional assertion이 주류였고, L3+L4 high-quality assertion은 세 그룹 모두 test file의 90% 이상에서 나타나 저자 유형별 통계적 유의 차이가 없었다.

### 주의

AI PR의 diff coverage가 상대적으로 높다는 결과를 '충분히 테스트됐다'로 해석하면 안 된다.
평균 절대 diff coverage 자체는 낮다.

따라서 Agent 작업 완료 조건은 다음처럼 보는 편이 적절하다.

- 테스트를 수정했는가? X
- coverage가 기존보다 올랐는가? X
- 변경한 위험 코드가 실제로 테스트됐는가? O
- 의미 있는 assertion이 있는가? O
- fault detection 능력이 있는가? O

## 5. Test Impact Analysis

Test Impact Analysis(TIA)는 코드와 테스트의 연결관계를 이용해 변경의 영향을 받을 가능성이 있는 테스트를 우선 선택하는 기법이다.

연결정보 후보:
- static dependency graph
- call graph
- runtime coverage mapping
- module dependency
- historical failure correlation
- changed API contract
- ownership/domain

### 단순 방식

changed file -> directly associated test

장점:
- 구현 단순
- 매우 빠름

문제:
- reflection
- dynamic dispatch
- shared configuration
- DB schema
- dependency injection
- event/message flows
- indirect common-library impact

을 놓칠 수 있다.

### 더 안전한 방식

changed symbols
-> direct dependents
-> transitive dependents
-> runtime-covered tests
-> historical failures
-> risk ranking

으로 확대한다.

## 6. Java/Spring 영향도 예

### Controller 변경
우선:
- controller/API tests
- authorization tests
- contract tests

### Service 변경
우선:
- service unit tests
- directly dependent controller/use-case tests
- affected integration tests

### Repository / MyBatis / JPA SQL 변경
우선:
- actual DB integration tests
- affected service tests

mock repository unit test만으로 승인하지 않는다.

### DTO / JSON Schema 변경
우선:
- serialization
- API contract
- consumer contract

### SecurityConfig 변경
우선:
- authn/authz integration
- representative E2E

변경 줄 수가 적다고 low risk로 분류해서는 안 된다.

## 7. PR Fast Gate 후보

### Stage 0: Integrity
- production/test diff 확인
- disabled test 증가 확인
- sleep/retry 추가 확인
- protected verifier 변경 확인

### Stage 1: Compile / Static
- compile
- lint/static analysis
- architecture rule

### Stage 2: Changed Tests
- directly mapped unit tests
- affected package tests

### Stage 3: Impact Tests
- dependency graph selected tests
- runtime coverage selected tests
- historical failure ranking

### Stage 4: Changed-Code Coverage
- modified executable line coverage
- changed branch coverage
- uncovered error handling 확인

### Stage 5: Critical Gates
- contract
- DB integration
- auth/security
- critical E2E smoke

### Stage 6: Deferred Broad Regression
Main/Nightly에서:
- full integration
- broad E2E
- mutation
- fuzz
- flaky perturbation

## 8. Selection Failure 방지

TIA의 가장 큰 위험은 '필요한 테스트를 선택하지 못하는 것'이다.

따라서 selection algorithm 자체를 검증해야 한다.

권장:
- 일정 비율 random full-regression shadow run
- nightly full suite
- missed regression 기록
- false negative rate 측정
- dependency map freshness 측정

즉 test selection도 테스트 대상이다.

## 9. Fallback Rule

다음 변경은 aggressive selection을 피하고 broader suite를 실행한다.

- build configuration
- dependency version
- framework upgrade
- common utility
- shared DTO/schema
- auth/security
- database migration
- global configuration
- serialization
- dependency injection wiring

Agent가 영향 범위를 자신 있게 좁힐 수 없는 경우 테스트를 줄이는 대신 확대한다.

## 10. Agent Rule 후보

Agent가 PR 검증 계획을 만들 때 다음을 출력하도록 한다.

- changed production symbols
- affected modules
- selected tests
- selection reason
- skipped expensive suites
- deferred suites and execution stage
- changed-code coverage gap
- risk fallback 여부

## 11. 핵심 원칙

> 테스트 선택은 테스트 생략 기능이 아니라 빠른 실패 탐지 순서를 설계하는 기능이다.

> 변경량보다 blast radius가 regression 범위를 결정한다.

> 전체 coverage보다 changed-code coverage가 Agent 변경 검증에 직접적이다.

> PR Fast Gate에서 선택하지 않은 테스트는 사라지는 것이 아니라 Main/Nightly에서 보완되어야 한다.