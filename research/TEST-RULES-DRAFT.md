# TEST-RULES Draft

상태: 연구 기반 초안
기준일: 2026-10-05

이 문서는 AI coding agent에 적용할 테스트 규칙 후보를 정리한다.

## 1. 기본 원칙
Agent는 코드를 변경할 때 가장 낮고 빠른 테스트 계층에서 요구사항을 검증한다.

우선순위: Unit -> Property -> Integration/API -> Contract -> E2E

전체 시스템 연결이 필요한 경우에만 E2E를 사용한다.

## 2. Bug Fix Rule
버그 수정 테스트는 가능하면 반드시 수정 전 코드에서 실패하고 수정 후 코드에서 성공해야 한다.
수정 전에도 통과하는 테스트는 해당 버그의 regression test로 인정하지 않는다.

## 3. Forbidden Test Weakening
Agent는 테스트를 통과시키기 위해 다음 행동을 하지 않는다.
- assertion 삭제
- assertion 범위 약화
- expected value를 현재 구현 결과로 변경
- failing test 삭제
- @Disabled / @Ignore 추가
- exception swallowing
- verifier/grader 변경
- hidden/acceptance test 변경
- retry 횟수 증가만으로 flaky 처리
- fixed sleep 추가로 timing failure 은폐

## 4. Assertion Rule
금지 후보:
- assertTrue(true)
- 의미 없는 assertNotNull 단독 검증
- production method 결과를 expected 계산에도 사용
- assertion 없는 test
- catch 후 아무 검증 없이 종료

## 5. Mutation Rule
critical business logic은 mutation testing 대상 후보로 분류한다.
coverage가 높아도 mutation score가 낮으면 테스트 품질 완료로 보지 않는다.

## 6. Flaky Rule
다음 nondeterminism을 명시적으로 검토한다.
- time
- timezone
- locale
- randomness
- thread scheduling
- unordered collection
- database order
- shared state
- external network
- filesystem
- test order

순서가 specification에 없으면 test도 순서를 요구하지 않는다.

## 7. Sleep Rule
Thread.sleep, TimeUnit.sleep, page.waitForTimeout, arbitrary fixed wait는 기본 금지한다.
대신 상태 또는 event를 기다리며 explicit timeout과 clear failure message를 둔다.

## 8. Retry Rule
retry는 product requirement를 검증하는 경우에만 허용한다.
flaky test를 통과시키기 위한 retry는 허용하지 않는다.

## 9. Held-out Rule
중요 기능은 agent-visible tests와 independent held-out tests를 분리한다.
held-out 후보: feature composition, cross-module behavior, architecture invariant, authorization, error handling, boundary combinations.

## 10. E2E Rule
E2E 추가 전 다음을 검토한다.
1. 더 낮은 계층에서 검증할 수 없는가?
2. 실제로 검증하는 system boundary는 무엇인가?
3. UI setup을 API/fixture로 줄일 수 없는가?
4. fixed sleep이 없는가?
5. 병렬 실행 가능한가?
6. 독립 상태인가?
7. critical user journey인가?

## 11. Test Integrity Rule
다음 영역은 CI 또는 repository policy로 보호하는 것을 권장한다.
- acceptance tests
- hidden tests
- grading scripts
- CI verifier
- mutation configuration
- critical architecture tests

Agent가 변경한 경우 별도 승인 대상으로 처리한다.

## 12. Completion Gate
Build -> Unit/Integration Green -> Before/After Regression Proof -> Assertion Quality -> Mutation/Fault Detection -> Flaky Check -> Held-out Check -> E2E Critical Flow -> Test Integrity Check

green 하나만으로 완료를 선언하지 않는다.