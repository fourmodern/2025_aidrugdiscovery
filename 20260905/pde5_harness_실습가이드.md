# PDE5 저해제 탐색 에이전트 하네스 — 수강생 실습 가이드

> 6일차 실습 · 대규모 언어모델과 신약개발 (2026)
> 이 가이드는 `0905_agent/pde5_harness/` 하네스를 **Claude Code / Claude Desktop** 으로 열어
> "자연어 지시 하나"로 자율 실행(PLAN → 검증 게이트 → 보고서)하는 절차를 단계별로 설명한다.

---

## 0. 이 실습이 보여주는 것

`nb1_scientific_agent.ipynb`(무료 노트북)에서 손으로 만든 에이전트 골격을, 실무 코딩 에이전트인
**Claude Code 하네스**로 구현한 버전이다. 차이는:

| | nb1 노트북 | pde5_harness (이 실습) |
|---|---|---|
| 실행 주체 | 노트북 안의 파이썬 루프 | **Claude Code 에이전트** (파일·Bash·훅을 직접 다룸) |
| 계획 | 스크립트 고정 or LLM | `CLAUDE.md` 규약 기반 LLM 자율 PLAN |
| 검증 | 함수 반환값 점검 | **PostToolUse 훅** + `verify.py` 단계별 게이트 |
| 도구 | 파이썬 함수 | `.claude/skills/*` + `scripts/*.py` |
| 비용 | **무료**(무키 폴백/Gemini 무료) | **Claude Code 유료 구독 필요** → 강사 데모용 |

> 💡 **무료 대안 안내:** Claude Code 구독이 없으면 이 하네스는 강사 데모로 참관하고,
> 직접 실습은 `notebooks/nb1_scientific_agent.ipynb`(무료 Colab)로 진행하면 동일한 개념을 익힐 수 있다.

---

## 1. 사전 준비 (로컬 .venv 모드 — Docker 불필요)

Docker 없이 로컬 파이썬 가상환경만으로 돌린다.

```bash
cd /home/hjpark/lecture_drug1/0905_agent/pde5_harness

# (권장) 가상환경 + 의존성
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt          # rdkit, requests, chembl-webresource-client

# 환경 점검 — rdkit ok / 스크립트 존재 / rdkit 실계산 smoke test
python run_harness.py check
```

`run_harness.py check` 가 아래처럼 **PASS** 를 출력하면 준비 완료:

```
[dep] rdkit                        ok
[dep] requests                     ok
[rdkit] smoke test: ethanol MW=46.07  ok
결과: PASS — 자율 실행 준비 완료
```

> `rdkit` 이 MISSING이면 `pip install rdkit` (구버전 pip은 `rdkit-pypi`).
> `chembl-webresource-client` 는 옵션이다 — 없으면 `chembl_actives.py` 가 **오프라인 데모 모드**
> (공개 대표 저해제 ChEMBL 실 ID, "실데이터 아님" 표기)로 폴백한다. 무-날조 원칙상 수치를 지어내지 않는다.

### (선택) 스크립트만 직접 돌려 보기 — Claude Code 없이도 동작 확인
```bash
python run_harness.py run            # target → chembl → mol-properties → selectivity 순차 실행
# 또는 개별:
python scripts/target_lookup.py                                  # UniProt O76074 실조회
python scripts/chembl_actives.py 10 | python scripts/mol_properties.py --stdin
python scripts/selectivity.py
```

---

## 2. Claude Code / Desktop 로 자율 실행하기

### 2-1. 폴더 열기
```bash
cd /home/hjpark/lecture_drug1/0905_agent/pde5_harness
claude                # Claude Code CLI
```
Claude Desktop을 쓰면 이 폴더를 워크스페이스로 추가한다.
진입 시 Claude Code가 `CLAUDE.md`(에이전트 규약)와 `.claude/settings.json`(검증 훅)을 자동 로드한다.

### 2-2. 컨텍스트 읽히고 자율 실행 지시
아래 자연어 프롬프트 하나면 된다:

> **"SETUP(README)·CLAUDE.md 규약을 읽고, PDE5 저해제 후보를 조사해 보고서까지 자율 실행해줘."**

또는 짧게:

> **"PDE5 저해제 후보 조사해 보고서 써줘"**

### 2-3. 에이전트가 수행하는 것 (지켜볼 포인트)
`CLAUDE.md` 규약대로 다음 순서로 진행한다:

1. **PLAN** — 연구 계획을 구조화 JSON(objective/questions/tools/success_criteria/stopping_rules)으로 작성.
2. **EXECUTE (단계별 + 검증 게이트)**:
   - `target-lookup` → PDE5A(UniProt **O76074**) 실재·기능 확인
   - `chembl-actives` → **CHEMBL1827** 활성물질(실 ID+SMILES) 조회
   - `mol-properties` → RDKit 물성·QED·SA·Lipinski
   - `selectivity-check` → PDE5 vs PDE6 (정직한 정성 한계)
   - 각 단계는 `{result, provenance, verification}` 표준 봉투를 남기고, `verification.passed=false` 면 **다음 단계로 진행하지 않는다**(최대 2회 재시도 후 "미확인/플래그").
3. **REPORT** — `report-writer` 로 IMRAD 보고서(`outputs/report_pde5.md`) 작성. 근거 없는 수치 0.

각 Bash 호출 뒤 **PostToolUse 훅** 2개(`verify_provenance.py`, `no_fabrication_guard.py`)가
provenance/무-날조 위반을 점검해 리마인더를 주입한다(경고형, 비차단).

### 2-4. 결과 확인
```bash
cat outputs/report_pde5.md
```
표적·후보 표·검증 로그·한계·참고문헌이 담긴 보고서가 생성된다.

---

## 3. 고도화 실습 (에이전트 확장)

강의 요구사항인 "도구 추가·실패 디버깅"을 하네스에서 실습하는 과제:

1. **도구 추가:** `scripts/`에 새 스크립트(예: `pubchem_lookup.py`)를 만들고 표준 봉투를 반환하게 한 뒤,
   `.claude/skills/<name>/SKILL.md` 를 추가해 스킬로 등록한다. Claude에게
   "PubChem 조회 스킬을 추가하고 파이프라인에 넣어줘"라고 지시해 자율 확장을 관찰한다.
2. **실패 디버깅:** 네트워크를 끊거나(오프라인) 잘못된 accession을 주고, 에이전트가
   재시도 → 오프라인 폴백 → "미확인 플래그" 로 **수치를 지어내지 않고** 회복하는지 확인한다.
3. **검증 강화:** `verify.py` 의 단계별 통과 조건을 더 엄격히 바꾸고(예: QED 임계 상향),
   게이트가 실제로 다음 단계를 막는지 확인한다.

---

## 4. 무-날조(No-Fabrication) 하드 규칙 — 반드시 준수

- **수치는 도구 실계산값만.** LLM이 IC50·물성·ChEMBL ID·PMID를 지어내는 것 금지.
- **모든 사실 주장에 출처**(ChEMBL ID / UniProt / PMID / DOI). 없으면 "확인 필요".
- **단계별 검증 게이트** 통과 후에만 다음 단계.
- **결과 = 가설.** 이 하네스는 알려진 화학공간 후보의 선별·정리 **시연**이며 신약 발견이 아니다.
  실검증(합성·효소 어세이)은 별도다.
- **selectivity/도킹은 정성·가설.** 정량 예측 없으면 "정성·확인 필요"로 명시(억지 수치 금지).

---

## 5. 파일 구조 요약

```
pde5_harness/
├── CLAUDE.md              # 에이전트 규약(PLAN→게이트→REPORT, 무-날조)
├── README.md             # 개요·워크플로·오프라인 안내
├── run_harness.py        # 로컬 헬퍼: check / run / list
├── requirements.txt      # rdkit, requests, chembl-webresource-client
├── Dockerfile            # (옵션) 컨테이너 실행
├── .claude/
│   ├── settings.json     # PostToolUse 훅 등록
│   ├── hooks/            # verify_provenance.py, no_fabrication_guard.py
│   └── skills/           # plan-research, target-lookup, chembl-actives,
│                         #   mol-properties, selectivity-check, report-writer
├── scripts/              # verify.py + 각 단계 스크립트(표준 봉투 반환)
└── outputs/              # 생성 보고서(report_pde5.md) 저장 위치
```

---

*무-날조 원칙: 모든 수치·식별자는 도구(UniProt/ChEMBL/RDKit) 실계산·실API 값만 사용한다.*
