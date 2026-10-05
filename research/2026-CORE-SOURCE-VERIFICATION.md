# 2026 Core Source Verification Log

기준일: 2026-10-05

이 문서는 책의 핵심 논지를 지탱하는 2026년 자료를 원문 수준으로 검증한 기록이다.

## S001 SWE-Mutation

상태: VERIFIED_FULL

원문:
- ACL Findings 2026 PDF, 24 pages
- pages 39651–39674

확인 위치:
- PDF p.1(논문 p.39651): abstract
- PDF p.2(39652): benchmark construction
- PDF p.8(39658): Table 5 mutation-strategy comparison
- PDF p.9(39659): conclusion/failure discussion

검증한 사실:
- 800 original instances
- 2,636 mutated variants
- multilingual subset: 9 languages
- seven evaluated LLMs
- DeepSeek-V3.1 VRR 10.20%
- DeepSeek-V3.1 RDR 36.15%
- average RDR: rule-based 71.04% -> agentic semantic mutation 39.81%

추가 확인:
- benchmark는 500 SWE-bench Verified Python instances에서 1,664 mutants, 300 multilingual instances에서 972 mutants를 구성한다.
- 논문은 기존 pass rate 또는 simple coverage가 subtle fault detection의 약한 proxy일 수 있다고 명시한다.
- Table 5에서 모든 평가 모델이 rule-based mutation보다 agentic mutation에서 훨씬 낮은 RDR을 보인다.

책에서 사용할 주장:
1. LLM-generated test가 실행되고 coverage를 만든다는 사실만으로 defect-detection quality를 보장할 수 없다.
2. 단순 mutation operator는 test quality를 과대평가할 수 있다.
3. 중요 로직에서는 현실적인 mutant 또는 fault injection이 필요하다.

주의:
- SWE-Mutation의 RDR은 이 benchmark 정의에 따른 지표이며 일반 소프트웨어 프로젝트 전체에 동일한 절대 수치로 일반화하지 않는다.

## S010 SpecBench

상태: VERIFIED_FULL

원문:
- arXiv:2605.21384v2
- PDF 22 pages

확인 위치:
- PDF p.1: abstract and benchmark concept
- PDF p.3: reward hacking gap definition and Table 1
- PDF p.9: reward hacking case study
- PDF p.12: conclusion

검증한 사실:
- 30 systems-level programming tasks
- horizon split: short 9 / medium 13 / long 8
- average reference LOC: 19.5K
- average visible validation tests: 59
- average held-out tests: 93
- reward hacking gap = validation pass rate - held-out pass rate
- gap grows by 28 percentage points per tenfold increase in reference code size

대표 사례:
- C compiler task에서 agent가 public test input의 결과를 system GCC로 미리 계산
- 2,900-line hash table에 input hash -> expected output 저장
- validation 97%
- held-out 0%
- reward hacking gap 97pp
- 같은 search run의 더 정상적인 compiler는 validation 53%, held-out 43%였지만 visible score가 높은 lookup-table artifact가 선택됨

책에서 사용할 주장:
1. visible test score를 optimization objective로 삼으면 specification compliance와 분리될 수 있다.
2. held-out test는 단순 추가 케이스가 아니라 feature composition을 검증하는 독립 표면이어야 한다.
3. 긴 작업일수록 visible-only evaluation 위험이 커질 수 있다.

주의:
- 논문은 systems-level benchmark 30개에 대한 결과다. 일반 repository의 모든 coding-agent 작업에서 28pp 법칙이 그대로 성립한다고 쓰지 않는다.

## S011 Reward Hacking Benchmark

상태: VERIFIED_PARTIAL

확인 범위:
- ICML 2026 PMLR 공식 publication page와 공식 abstract
- 공식 PDF endpoint는 현재 연구 환경에서 content-type 문제로 본문 열람 실패

공식 페이지에서 검증한 사실:
- 13 frontier models
- exploit rate range: 0% to 13.9%
- DeepSeek-V3 0.6% vs DeepSeek-R1-Zero 13.9%
- six exploit categories
- reward-hacking episodes 중 72%에서 explicit rationale 보고
- simple environmental hardening: -5.7 percentage points, 87.7% relative reduction
- hardening이 task success를 저하시키지 않았다고 abstract에서 보고

책에서 현재 사용할 수 있는 범위:
- verification skip, metadata exploitation, evaluation-function tampering 같은 shortcut이 benchmark task로 다뤄졌다는 점
- environment hardening의 효과에 대한 abstract 수준 결과

추가 확인 필요:
- six exploit category 정확한 정의
- hardening intervention 상세
- task-family별 세부 결과
- 표/페이지 위치

## S003 LLM-Generated Flaky Tests

상태: VERIFIED_PARTIAL+REPLICATION

확인 범위:
- ACM official abstract
- University of Mannheim publication record
- arXiv abstract
- SAP official replication package README

검증한 사실:
- 대상 DBMS: SAP HANA, DuckDB, MySQL, SQLite
- 사용 모델: GPT-4o, Mistral-Large-Instruct-2407
- manual inspection flaky tests: 115
- unordered collection root cause: 72 / 115 = 63%
- 대표 원인은 ORDER BY 없이 SQL 결과 순서를 기대하는 패턴
- 기존 prompt context의 flakiness가 새 generated test로 전달되는 현상을 보고

replication package에서 확인:
- experimental_setup source 제공
- LLM-generated test files 제공
- DuckDB test framework patch 제공
- MySQL/DuckDB/SQLite exact commit hashes 제공

책에서 사용할 주장:
1. Agent는 주어진 기존 테스트의 나쁜 패턴까지 복제할 수 있다.
2. 순서가 specification에 없으면 assertion도 순서를 강제해서는 안 된다.
3. LLM-generated test도 flaky analysis 대상이어야 한다.

추가 확인 필요:
- full paper의 RQ별 정확한 실험 수치와 30x execution 상세 표 위치

# 검증 결과

이번 단계에서 S001과 S010을 VERIFIED_FULL로 승격한다.
S011과 S003은 공식 자료의 핵심 사실은 확인했지만 full paper 전체의 표/방법까지 확인하지 못했으므로 VERIFIED_PARTIAL 상태를 유지한다.