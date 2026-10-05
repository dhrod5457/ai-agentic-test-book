# Java Spring Test Gate Stack

기준일: 2026-10-05

이 문서는 Java/Spring 프로젝트에 적용 가능한 Agent 테스트 검증 스택 후보를 정리한다.

## 1. Unit

도구 후보:
- JUnit 5
- AssertJ
- Mockito

원칙:
- pure business rule은 Spring context 없이 검증
- mock은 외부 경계를 끊는 용도로만 사용
- 내부 구현을 그대로 복제하는 mock verification은 피함

## 2. Property-Based

도구 후보:
- jqwik

적합한 대상:
- 계산
- 정렬/변환
- 범위 validation
- serialization invariants
- idempotency

Agent가 몇 개의 예시 값만 생성하는 대신 input space를 넓게 탐색하도록 사용한다.

## 3. Integration

도구 후보:
- Spring Boot Test
- Testcontainers

적합한 대상:
- SQL
- transaction
- Redis
- Kafka
- database constraints
- serialization/deserialization

중요 원칙:

DB query를 변경했는데 repository 전체를 mock하면 실제 SQL 오류를 발견할 수 없다.

## 4. Contract

도구 후보:
- Pact JVM
- OpenAPI schema validation

적합한 대상:
- service-to-service API
- provider/consumer compatibility
- message schema

E2E 전체를 돌리지 않고 서비스 경계의 계약을 빠르게 검증한다.

## 5. Architecture

도구 후보:
- ArchUnit

Agent가 기능 구현 과정에서 다음 구조를 깨뜨리는 것을 방지한다.
- package dependency
- layer direction
- forbidden dependency
- naming/annotation rule

## 6. Mutation

도구 후보:
- PIT / pitest

적합한 대상:
- business-critical rule
- authorization
- calculation
- branching-heavy service code

모든 PR에서 전체 mutation을 돌릴 필요는 없다.

권장:
- PR: changed package 또는 selected mutation
- nightly/main: wider mutation

## 7. Async

도구 후보:
- Awaitility

fixed sleep 대체 원칙:
- 상태를 기다린다.
- explicit timeout을 둔다.
- 실패 시 현재 상태를 알 수 있는 메시지를 제공한다.

## 8. E2E

도구 후보:
- Playwright
- Selenium
- Cypress (frontend stack에 따라)

Java backend 프로젝트라도 E2E는 backend unit/integration을 대체하지 않는다.

critical journey만 남긴다.

## 9. Flaky Gate

정적 후보:
- Thread.sleep 탐지
- @Disabled/@Ignore 변경 감지
- retry annotation 변경
- assertion 없는 테스트
- empty catch

동적 후보:
- repeat
- random test order
- timezone 변경
- locale 변경
- seed 변경
- unordered collection perturbation

2026 ChaosAPI 연구처럼 nondeterministic source를 직접 흔드는 접근을 향후 자동화 후보로 둔다.

## 10. CI Gate 예시

PR:
- compile
- unit
- changed integration
- contract
- architecture
- critical E2E smoke
- test anti-pattern scan

Main:
- broader integration
- broader E2E
- selected mutation

Nightly:
- full mutation
- full E2E
- cross-browser
- flaky perturbation
- fuzz/fault injection

## 11. E2E 병렬화 주의

ICSE 2026 journal-first로 발표된 system-level testing 연구는 테스트 병렬화에서 shared state dependency가 핵심 제약임을 보여준다.

무조건 parallel=true를 켜기 전에 다음을 분리한다.
- shared database rows
- singleton state
- common test account
- message queue topic
- filesystem
- external resource

병렬화 가능한 테스트는 독립 fixture와 독립 account/resource를 가져야 한다.

## 12. Uber AutoCover 사례

ICSE-SEIP 2026 Distinguished Paper인 Uber AutoCover는 LLM을 이용해 테스트를 생성, 검증, 수리하는 production system이다.

공개 논문 기준 AutoCover가 생성하고 리뷰되어 Uber 코드베이스에 추가되는 테스트는 전체 신규 테스트의 약 11%를 차지한다.

세 가지 사용 방식:
- CLI
- Headless repository-scale generation
- IDE human-in-the-loop

중요한 시사점:

산업 규모에서는 'LLM이 테스트 코드를 생성한다'만으로 끝나지 않는다.

생성 -> 검증 -> repair -> workflow integration -> review

전체 파이프라인이 필요하다.

## 13. Agent 기본 정책 후보

1. 테스트 생성보다 먼저 변경 위험도를 분류한다.
2. 가장 낮은 계층에서 결함을 잡는다.
3. 실제 DB/API 경계는 필요한 경우 mock하지 않는다.
4. sleep/retry로 실패를 숨기지 않는다.
5. critical logic은 mutation으로 test의 test를 수행한다.
6. critical acceptance는 held-out으로 보호한다.
7. 느린 테스트는 duration을 기록하고 budget을 초과하면 구조를 재검토한다.
8. 병렬화 전에 shared-state dependency를 제거한다.