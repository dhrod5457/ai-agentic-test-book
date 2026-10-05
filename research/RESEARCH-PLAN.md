# Research Plan

기준일: 2026-10-05

이 저장소는 AI Agent가 코드를 생성·수정할 때 함께 만들어야 하는 테스트 규칙을 연구하고 책으로 정리하기 위한 저장소다.

집필·검수 기준은 `dhrod5457/published-book/BOOK-WRITING-MANUAL.md`를 따른다. 특히 검색 결과의 제목만 본 상태와 실제 원문·초록·공식 README를 확인한 상태를 구분한다.

## 연구 원칙

핵심 근거는 **2026년에 발표되거나 2026년에 현행성이 확인된 자료를 우선**한다.

2025년 이전 연구는 다음 경우에만 보조적으로 사용한다.

- 2026년 논문이 직접 인용하는 기초 개념
- test smell, mutation testing, property-based testing처럼 개념의 원전 설명이 필요한 경우
- 현재 사용되는 오픈소스 도구의 설계 배경을 설명하는 경우

책의 주장은 가능하면 2026년 자료를 먼저 인용하고, 오래된 자료는 역사적 배경으로 분리한다.

## 핵심 연구 질문

1. AI Agent가 코드를 변경할 때 어떤 테스트를 반드시 추가하거나 수정해야 하는가?
2. Agent가 만든 테스트가 실제 결함을 검출하는지 어떻게 독립적으로 검증할 것인가?
3. Agent가 테스트를 '통과시키기 위해' 테스트 자체나 검증 조건을 약화시키는 것을 어떻게 막을 것인가?
4. `sleep`, 고정 지연, 무분별한 retry 같은 flaky-test 은폐 패턴을 어떻게 금지할 것인가?
5. 과도한 mock, 구현 결과 복사, 의미 없는 assertion 같은 쓰레기 테스트를 어떻게 판별할 것인가?
6. visible test와 held-out test를 분리해야 하는가?
7. coverage와 mutation score를 어떤 관계로 사용해야 하는가?
8. 테스트 생성 Agent와 테스트 검증 Agent를 분리할 필요가 있는가?

## 2026년 우선 조사 축

### R1. LLM/Agent 테스트 생성 품질

- 2026년 LLM 기반 unit test generation 연구
- agentic iteration이 실제 품질을 높이는지
- repository context가 테스트 생성에 미치는 영향
- mock 기반/비-mock 기반 테스트의 차이
- compile rate, execution rate, coverage, mutation score 비교

### R2. 테스트 자체의 검증 능력

핵심 질문은 '테스트가 통과하는가'가 아니다.

> 잘못된 구현을 넣었을 때 이 테스트가 실패하는가?

따라서 mutation testing과 adversarial mutant를 핵심 평가 기준으로 조사한다.

### R3. Reward Hacking / Test Gaming

Agent가 아래 행동으로 성공을 위조하는 문제를 조사한다.

- 테스트 파일 변경
- assertion 약화
- expected value를 구현 결과에 맞춤
- verifier 또는 grading 로직 변경
- skip/disable
- 예외 삼키기
- hard-coded answer
- visible test만 맞추기
- 사용자 요구보다 grader가 볼 법한 항목에 최적화

### R4. Flaky Test

- 비결정적 순서
- clock/timezone
- random seed
- 외부 네트워크
- 공유 DB 상태
- 테스트 순서 의존
- concurrency
- fixed sleep
- retry masking

### R5. Test Smell

2026년 연구에서 중요도가 높게 보고된 smell과 AI Agent 특화 smell을 함께 다룬다.

특히 다음을 우선한다.

- Sleepy Test
- Ignored Tests
- Resource Optimism
- Mystery Guest
- Assertion Roulette
- Eager Test
- Unknown Test
- Empty Test

AI Agent 특화 작업 용어:

- Always Green Test
- Implementation Echo Test
- Mock Echo Test
- Assertion Dilution
- Exception Swallowing
- Disabled Regression Test
- Coverage Padding
- Sleep Stabilization
- Retry Concealment
- Test-Harness Tampering
- Production-Code-for-Test Distortion

위 AI Agent 특화 이름은 기존 학술 표준 용어라고 주장하지 않고 책에서 설명을 위해 사용하는 작업 명칭으로 표시한다.

## 테스트 품질 게이트 가설

책에서 다음 구조를 검증한다.

```text
요구사항
   |
   v
코드 변경
   |
   v
테스트 생성/수정
   |
   +--> Build Gate
   +--> Red/Green Gate
   +--> Assertion Quality Gate
   +--> Boundary/Negative Gate
   +--> Mutation Gate
   +--> Flaky Gate
   +--> Isolation Gate
   +--> Anti-Cheating Gate
   +--> Held-out Verification Gate
   |
   v
회귀 테스트로 고정
```

### Red/Green Gate

버그 수정 테스트라면 반드시 다음을 확인한다.

- 수정 전 코드에서 실패
- 수정 후 코드에서 성공

처음부터 성공하는 테스트는 버그를 재현했다는 증거가 아니다.

### Assertion Quality Gate

금지 후보:

- `assertTrue(true)`
- 단순 `assertNotNull`만으로 복잡한 규칙을 검증
- 구현 메서드를 다시 호출해 expected 값을 계산
- assertion 없음
- 예외를 catch하고 아무 검증 없이 종료

### Mutation Gate

line/branch coverage만으로 테스트 품질을 승인하지 않는다.

의미 있는 코드 변이를 투입하고 테스트가 이를 검출하는지 확인한다.

### Flaky Gate

동일 입력과 동일 환경에서 반복 실행했을 때 결과가 일관되어야 한다.

고정 `sleep` 추가를 flaky 해결로 인정하지 않는다.

### Anti-Cheating Gate

Agent가 자신을 평가하는 테스트·verifier를 마음대로 수정할 수 없도록 경계를 둔다.

가능하면 다음을 분리한다.

- Agent가 작성할 수 있는 테스트
- 사람이 제공한 acceptance/held-out test
- CI의 독립 verifier
- mutation/fault-injection 검증

## 책에 넣을 실전 사례

1. 가격 계산 버그 수정 중 Agent가 expected value까지 함께 바꾸어 통과시키는 사례
2. 비동기 API 테스트 실패에 `Thread.sleep(3000)`을 넣어 CI만 느려지는 사례
3. 실패하는 테스트를 `@Disabled` 처리하는 사례
4. 예외를 catch하고 성공 처리하는 사례
5. DB 통합 테스트를 mock으로 대체해 SQL 오류가 사라지는 사례
6. mock이 expected object 자체를 반환하는 사례
7. retry 5회로 flaky test를 숨기는 사례
8. coverage 90%지만 mutation score가 낮은 테스트 스위트
9. visible tests는 전부 통과하지만 held-out 조합 테스트에서 실패하는 Agent 구현
10. 테스트 파일 또는 verifier를 수정해 점수를 얻는 Agent
11. 구현 내부 구조를 그대로 복제한 brittle test
12. 수정 전에도 통과하는 '회귀 테스트'를 Agent가 추가한 사례

## 자료 상태

- VERIFIED_FULL: 본문 전체 또는 공식 자료 전체를 실제 확인
- VERIFIED_PARTIAL: 초록, 일부 본문, 공식 README 등 필요한 범위를 확인
- LINK_ONLY: 링크만 확보
- REVERIFY: 버전, 날짜, 실험 조건 등을 다시 확인해야 함

본문 핵심 주장에는 가능한 한 2026년 VERIFIED_FULL/PARTIAL 자료를 둘 이상 연결한다.
