# 2026 Code Context and Prompting Verification

기준일: 2026-10-05

## S002 Impact of code context and prompting strategies on automated unit test generation with modern general-purpose LLMs

상태: VERIFIED_PARTIAL+REPLICATION

출판 정보:
- Journal of Systems and Software
- Volume 237, July 2026, Article 112834
- DOI: 10.1016/j.jss.2026.112834

확인 범위:
- ScienceDirect 공식 본문 페이지의 highlights, abstract, introduction, methodology/result/conclusion snippets
- 공개 arXiv preprint metadata
- 저자 공개 GitHub replication repository peetery/LLM-analysis
- prompt strategy, prompt template, AST context extractor 구현 확인

연구 질문:
- RQ1: code context 수준이 테스트 품질에 어떤 영향을 주는가
- RQ2: simple vs sequential multi-turn prompting의 차이
- RQ3: 모델별 compilation / branch coverage / mutation score / uniqueness 비교
- RQ4: LLM과 human tester가 공통으로 놓치는 systematic gap

실험 context levels:
- CF1 Interface: method signatures only
- CF2 Interface + Docstring: signatures + behavioural documentation
- CF3 Full Context: complete implementation

핵심 결과:
- Interface-only 대비 docstring 포함 시 branch coverage +19.67 percentage points
- compilation success +9.16 percentage points
- full implementation까지 추가했을 때의 incremental gain은 docstring 추가 대비 작음
- sequential multi-turn prompting에서 최대 96.3% branch coverage
- 평균 mutation score 57%
- near-perfect compilation success
- simple prompting 대비 computational cost +187%
- Gemini 2.5 Pro는 full context 조건에서 mutation score 87%
- 비교한 practitioner baseline mutation score 44%
- 모든 평가 LLM이 None, inf, NaN 같은 special-value robustness test를 체계적으로 놓침

중요한 해석:
1. 테스트 생성에는 구현 세부보다 behavioural specification이 더 큰 가치를 줄 수 있다.
2. context를 많이 넣는 것이 무조건 좋은 것이 아니다.
3. sequential analysis -> scenario planning -> implementation 방식은 품질을 높일 수 있지만 비용이 크다.
4. special values와 robustness는 명시적으로 요구하지 않으면 LLM과 사람 모두 놓칠 수 있다.

공개 replication repository에서 확인한 구조:
- simple prompting: 한 번에 complete test suite 요청
- sequential strategy: analyze -> plan -> implement 3단계
- context extractor:
  - interface
  - interface_docstring
  - full_context
- AST 기반으로 public methods, helper types, relevant imports를 추출
- full context에서는 helper types와 실제 source를 포함
- prompt는 typical / edge / invalid input / exception을 명시적으로 요구

replication README에서 확인한 파이프라인:
Generated Test
 -> compilation check
 -> test execution
 -> branch coverage
 -> mutation testing
 -> quality metrics

주의:
- 공개 repository의 현재 README는 논문 출판 당시 실험 설정과 이후 repository 발전이 섞여 있을 수 있다.
- README에는 현재 720 CLI experiments와 최신 CLI model 이름이 기록되어 있어, journal paper의 여섯 모델 실험과 그대로 동일하다고 간주하지 않는다.
- journal paper 결과는 ScienceDirect 공식 본문 수치를 우선한다.

## Agent Context Rule로 변환

테스트 생성 Agent에게 다음 순서로 context를 제공하는 것을 권장한다.

### 1. Requirement / Behaviour First

가장 먼저 제공:
- acceptance criteria
- method/class contract
- docstring
- valid input domain
- invalid input
- exception contract
- boundary conditions

### 2. Interface

- public methods
- parameter types
- return types
- public DTO/schema

### 3. Relevant Dependencies Only

- 실제 호출하는 collaborator contract
- helper types
- DB/API/message schema
- existing usage examples

### 4. Implementation Selectively

full implementation은 다음 경우에만 추가:
- branch discovery가 필요한 경우
- undocumented behavior를 조사해야 하는 경우
- internal condition이 테스트 설계에 필요하지만 specification이 부족한 경우

구현 전체를 기본 입력으로 넣지 않는다.

## Special-Value Checklist

LLM이 체계적으로 놓친 항목을 Agent 규칙에 명시한다.

숫자:
- zero
- negative
- min/max
- overflow boundary
- NaN
- Infinity

reference/container:
- null / None
- empty
- single item
- duplicate
- unordered

string:
- empty
- blank
- Unicode
- very long
- malformed

time:
- timezone
- DST
- boundary second/day/month/year

## Multi-Step Generation Rule

중요 로직에서는 다음 흐름을 사용할 수 있다.

1. Analyze behaviour and constraints
2. List test scenarios without code
3. Check missing boundaries and failures
4. Generate tests
5. Execute
6. Mutation / coverage feedback
7. Revise

다만 모든 단순 테스트에서 multi-turn을 강제하지 않는다.
논문에서 +187% computational cost가 관찰되었으므로 risk-based로 적용한다.

## 핵심 원칙

> 테스트 생성 Agent에게 가장 먼저 줘야 할 것은 구현 코드가 아니라 행동 명세다.

> Full context는 공짜가 아니다. 더 많은 token과 implementation bias를 가져올 수 있다.

> Edge case를 잘 찾을 것이라고 기대하지 말고, 중요한 robustness class를 규칙으로 명시한다.
