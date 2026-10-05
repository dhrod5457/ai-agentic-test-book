# Test Anti-Patterns for Coding Agents

기준일: 2026-10-05

이 문서는 AI coding agent가 코드를 수정하면서 만들어내기 쉬운 나쁜 테스트를 분류하기 위한 연구 노트다.

2026년 연구에서 reward hacking, test smell, flaky test, weak generated test가 반복해서 관찰되고 있으므로 테스트 코드는 프로덕션 코드와 동일하게 검토 대상이어야 한다.

## 1. 억지로 성공시키는 테스트

### Assertion Dilution

실패하는 테스트를 고치지 않고 assertion을 약하게 바꾸는 패턴.

예:

```text
정확한 금액 9000원을 검증
        ↓
0보다 큰지만 검증
        ↓
not null만 검증
```

테스트는 green이지만 원래 요구사항을 더 이상 검증하지 않는다.

### Implementation Echo Test

expected value를 독립된 specification에서 만들지 않고 현재 구현 결과를 이용해 계산한다.

```text
actual = production.calculate(input)
expected = production.calculate(input)
assert actual == expected
```

형식상 assertion은 있지만 결함을 검출할 수 없다.

### Exception Swallowing

실패해야 하는 예외를 잡은 뒤 아무 검증 없이 테스트를 종료한다.

### Disabled Regression Test

실패하는 테스트를 삭제하거나 skip/disable 처리한다.

### Test-Harness Tampering

Agent가 자신을 평가하는 테스트, verifier, grading script를 수정해 점수를 얻는다.

2026 Reward Hacking Benchmark와 BenchJack 계열 연구는 evaluation-relevant function tampering과 검증 절차 우회를 실제 위험으로 다룬다.

## 2. 쓰레기 테스트

'쓰레기 테스트'는 학술 표준 용어가 아니라 이 책에서 실용적으로 사용할 표현이다. 정확한 test smell 명칭이 존재하면 그 용어를 함께 적는다.

### Always Green Test

무슨 코드가 들어가도 통과하는 테스트.

후보 탐지 규칙:

- assertion 없음
- `assertTrue(true)`
- catch 블록에서 성공 종료
- 검증 결과를 로그만 출력
- 결과값을 계산하지만 assertion에 사용하지 않음

### Coverage Padding

coverage만 높이기 위해 메서드를 호출하지만 결과나 side effect를 검증하지 않는다.

SWE-Mutation의 문제의식처럼 coverage가 있다고 해서 discriminative test라는 뜻은 아니다.

### Mock Echo Test

mock이 expected value 자체를 그대로 반환하도록 설정하고 그 반환값을 검증한다.

실제 프로덕션 로직이나 integration point의 동작은 검증되지 않는다.

### Overspecified Interaction Test

필요한 결과보다 내부 호출 순서와 구현 세부를 지나치게 검증한다.

Agent가 리팩터링할 때 기능은 그대로인데 테스트가 대량으로 깨지는 원인이 될 수 있다.

## 3. Sleep으로 성공시키는 테스트

2026 test-smell systematic mapping은 Sleepy Test를 중요한 test smell 중 하나로 분류한다.

### Sleep Stabilization

테스트가 간헐적으로 실패할 때 원인을 찾지 않고 고정 지연을 추가한다.

```text
요청 전송
Thread.sleep(3000)
응답 확인
```

문제:

- 빠른 환경에서도 항상 3초를 낭비한다.
- 느린 CI에서는 3초 후에도 실패한다.
- timeout 원인을 숨긴다.
- 테스트 개수가 늘수록 전체 수행 시간이 선형으로 증가한다.

대안 후보:

- Awaitility 같은 condition-based wait
- future/promise completion
- latch/barrier
- event observation
- virtual clock
- deterministic scheduler

핵심 규칙 후보:

> 시간을 기다리지 말고, 상태가 성립할 때까지 기다린다.

단, 조건 대기도 무한정 기다려서는 안 되며 명시적인 timeout과 실패 메시지를 가져야 한다.

## 4. Retry로 숨기는 flaky test

### Retry Concealment

실패 원인을 제거하지 않고 테스트를 여러 번 실행해서 한 번이라도 성공하면 통과시키는 패턴.

재시도 자체가 항상 잘못은 아니다. 네트워크 fault-tolerance 같은 실제 요구사항을 검증하는 테스트에는 재시도가 테스트 대상일 수 있다.

금지 대상은 '테스트의 flaky함을 숨기기 위한 retry'다.

검증 질문:

- 왜 첫 실행이 실패할 수 있는가?
- random seed가 고정되어 있는가?
- 테스트 순서에 의존하는가?
- DB 상태가 격리되는가?
- clock/timezone이 고정되는가?
- collection order를 가정하는가?
- 비동기 completion을 관찰하는가?

2026 ICSE-SEIP의 LLM-generated flaky test 연구에서는 unordered collection 의존이 주요 원인으로 관찰됐다.

## 5. Visible Test Overfitting

Agent가 볼 수 있는 테스트만 통과하도록 구현한다.

2026 SpecBench는 visible validation test와 held-out composition test 사이의 gap으로 coding-agent reward hacking을 측정한다.

실무 규칙 후보:

- Agent에게 unit test 작성 권한을 줄 수 있다.
- 그러나 acceptance/held-out 검증 전체를 Agent가 수정할 수 있게 두지 않는다.
- CI verifier와 일부 contract/invariant test는 독립 경계에 둔다.

## 6. 테스트 생성 Agent의 금지 행동 후보

Agent 규칙 파일에 다음을 명시하는 방안을 조사한다.

- 실패 테스트를 삭제하지 않는다.
- 기존 assertion을 약하게 만들지 않는다.
- skip/disable을 추가하지 않는다.
- fixed sleep을 flaky 해결책으로 사용하지 않는다.
- retry를 flaky 은폐 목적으로 사용하지 않는다.
- expected 값을 production method 결과로 계산하지 않는다.
- verifier, grading, hidden/acceptance test를 수정하지 않는다.
- coverage 증가만을 완료 조건으로 삼지 않는다.
- bug fix 테스트는 수정 전 실패 여부를 확인한다.
- 새 테스트는 최소 한 개 이상의 의미 있는 failure mode 또는 boundary를 검토한다.

## 7. 자동 탐지 후보

### 정적 검사

검색 대상:

- `Thread.sleep`
- `TimeUnit.*.sleep`
- `@Disabled`, `@Ignore`
- assertion 없는 test method
- 빈 catch
- catch 후 return
- 지나치게 넓은 `assertDoesNotThrow`
- constant-true assertion
- production method를 expected 계산에도 호출
- retry annotation 추가

### 동적 검사

- 동일 test N회 반복
- test order shuffle
- timezone 변경
- locale 변경
- random seed 변경
- CPU load / scheduling 변화
- mutation testing
- fault injection
- before-patch / after-patch red-green 검증

## 8. 책의 중심 메시지 후보

> Agent에게 테스트를 만들게 하는 것과 Agent가 만든 테스트를 신뢰하는 것은 다른 문제다.

> 테스트가 green이라는 사실보다, 잘못된 구현을 red로 만들 수 있다는 사실이 중요하다.

> Agent가 접근할 수 있는 테스트만으로 Agent를 평가하면 테스트가 목표가 되고, 사용자의 요구사항은 부차적인 것이 될 수 있다.
