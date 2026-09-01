# 에이전트 하네스 2종 — 결과 보고서

> 9/5 「대규모 언어모델과 신약개발」 강의용. `0905_agent/` 안의 두 Claude Code 하네스
> — **저분자(PDE5)** 와 **항체(HER2)** — 를 실제로 실행해 확인한 결과와 한계를 정리한다.
>
> **무-날조 원칙**: 이 문서의 모든 수치·식별자·서열은 **실행으로 확인된 값**이거나
> **원본 저장소·공식 문서에서 확인한 인용값**이다. 실행하지 않은 것은
> `미검증` 으로 명시했다. 추정·근사·"대략적인 값"은 쓰지 않았다.
>
> **🔄 2026-09-01 갱신 — RunPod GPU 실행으로 설계 2경로를 실제로 완주했다.**
> 이전 판에서 `미검증(GPU)` 이던 항목 다수가 실측값으로 대체되었고,
> **"ESMFold2 는 버전 스큐로 공개 릴리스에서 실행 불가"라는 이전 판단은 틀렸다** —
> 정정 내용은 §6.9 재현성 교훈 참조.
>
> **실행 환경 A — CPU 평가 (2026-08-31)**
> - 일시: 2026-08-31 (UTC) — 봉투 `provenance.timestamp` 기준
> - 인터프리터 A: `/home/hjpark/lecture_drug1/.venv_slides/bin/python` — Python 3.12.3, RDKit 2026.03.4, BioPython 1.88
> - 인터프리터 B: `/share/anaconda3/bin/python3` — Python 3.8.6, RDKit 2024.03.5 (버전 차이 검증용)
> - GPU: `nvidia-smi` 는 PATH 에 있으나 `torch.cuda` 는 **사용 불가** (`torch 2.13.0+cpu`)
>
> **실행 환경 B — RunPod GPU 설계 (2026-08-31 UTC 저녁 ~ 2026-09-01)**
> - 경로 B (RFantibody): **A40, $0.44/hr** — 3단계 전부 완주
> - 경로 A (ESMFold2 inversion): **H100, $3.29/hr** — 설계 성공.
>   `esm 3.4.0` (commit `827ec128`, 2026-08-27), `torch 2.11.0+cu130`, Python 3.12
> - 총 비용: H100 2회 + A40 1회 = **약 $5**. 종료 후 `list` 로 잔여 pod **0** 확인
> - 원본 로그: `0905_agent/runpod_logs/` (`rfantibody/{rfab.log, 3_rf2.sc}`,
>   `esmfold2_design_{run.log,results.json}`, `esmfold2_design.log`)

---

## 목차

1. [개요](#1-개요)
2. [두 하네스 비교](#2-두-하네스-비교)
3. [공통 설계 원칙](#3-공통-설계-원칙)
4. [하네스 ① PDE5 (저분자)](#4-하네스--pde5-저분자)
5. [하네스 ② 항체 (바이오로직스)](#5-하네스--항체-바이오로직스)
6. [설계 2경로 — ESMFold2 inversion vs RFantibody](#6-설계-2경로--esmfold2-inversion-vs-rfantibody)
   - 6.6 [GPU 실측 ① 경로 B RFantibody (A40)](#66-gpu-실측--경로-b-rfantibody-a40--3단계-완주)
   - 6.7 [GPU 실측 ② 경로 A ESMFold2 inversion (H100)](#67-gpu-실측--경로-a-esmfold2-inversion-h100--설계-성공)
   - 6.8 [설계 2경로 실측 비교](#68-설계-2경로-실측-비교)
   - 6.9 [재현성 교훈 — 이전 오판 정정](#69-재현성-교훈--이전-오판-정정)
7. [검증 매트릭스](#7-검증-매트릭스)
8. [무-날조가 실제로 작동한 사례](#8-무-날조가-실제로-작동한-사례)
9. [강의 활용법](#9-강의-활용법)
10. [재현 방법](#10-재현-방법)
11. [미검증 항목 총목록](#11-미검증-항목-총목록)

---

## 1. 개요

`0905_agent/` 에는 **같은 뼈대를 공유하는 두 개의 에이전트 하네스**가 있다.

| | 하네스 | 경로 | 도메인 |
|---|--------|------|--------|
| ① | PDE5 저해제 탐색 | `0905_agent/pde5_harness/` | 저분자 (small molecule) |
| ② | 항체 설계 | `0905_agent/antibody_harness/` | 바이오로직스 (antibody) |

"하네스(harness)"란 **폴더 하나가 곧 에이전트**가 되도록 구성한 것이다.
`CLAUDE.md`(에이전트 지시서) + `.claude/skills/*/SKILL.md`(도구 카드) +
`scripts/*.py`(실제 계산) + `.claude/hooks/*.py`(사후 점검 훅) 의 조합이며,
Claude Code 가 이 폴더를 열면 자연어 지시 하나로 계획 → 단계별 검증 → 보고서를 자율 수행한다.

두 하네스를 나란히 두는 이유는 명확하다. **도메인이 저분자에서 단백질로 바뀌어도
계약(표준 봉투)·게이트·무-날조 규약은 그대로 재사용된다**는 것을 보이기 위해서다.
바뀌는 것은 도구(RDKit ↔ BioPython)와 데이터 소스(ChEMBL ↔ RCSB PDB)뿐이다.

---

## 2. 두 하네스 비교

| 항목 | ① PDE5 (저분자) | ② 항체 (바이오로직스) |
|------|------------------|------------------------|
| **표적** | PDE5A — UniProt `O76074`, ChEMBL `CHEMBL1827` | HER2/ERBB2 — UniProt `P04626` (ECD 23–652) |
| **참조 물질** | sildenafil `CHEMBL192` · tadalafil `CHEMBL779` · vardenafil `CHEMBL1520` | 트라스투주맙 `1N8Z` · 퍼투주맙 `1S78` |
| **주 계산 라이브러리** | RDKit (Descriptors / QED / Lipinski / sascorer) | BioPython (ProtParam · PairwiseAligner + BLOSUM62) |
| **데이터 소스** | UniProt REST · ChEMBL (webresource client) | UniProt REST · RCSB Search+Data REST API |
| **파이프라인 단계** | target → chembl → mol-properties → selectivity → report (4+1) | antigen → antibody-search → cdr → developability → humanness → report (5+1) |
| **설계(생성) 단계** | 없음 (알려진 화학공간 선별만) | **있음** — 경로 A ESMFold2 inversion / 경로 B RFantibody |
| **스킬 수** | 6 (`plan-research`, `target-lookup`, `chembl-actives`, `mol-properties`, `selectivity-check`, `report-writer`) | 9 (평가 5 + 설계 3 + `report-writer`) |
| **스크립트 수** | 7 (`verify` + 4 필수 + `mol_utils`·`docking` 옵션) | 11 (`verify`·`seq_utils` + 평가 5 + 설계 3) |
| **하드 규칙 조항 수** | 5 (README `하드 규칙`) | **12** (CLAUDE.md `하드 규칙`) |
| **게이트 수 (실행 확인)** | 4 PASSED | 5 PASSED |
| **실행 환경** | CPU · 노트북에서 즉시 동작 | 평가 = CPU 즉시 / **설계 = GPU 필요** (실측: 경로 A H100 · 경로 B A40) |
| **GPU 오케스트레이션** | 없음 | RunPod (`RUNPOD_가이드.md`, 외부 `runpod_ctl.py` 재사용) — **2026-09-01 실행 완료, 총 약 $5** |
| **컨테이너** | `Dockerfile` 제공 | 없음 (`.venv` 모드 + RunPod 이미지) |
| **옵션 도킹/구조** | `mol_utils.py`(RCSB PDB 다운로드), `docking.py`(smina/vina 래퍼) | — |
| **오프라인 폴백 정책** | 공개 참조값으로 폴백 + `OFFLINE DEMO … 실데이터 아님` 표기, **게이트는 통과** | 1N8Z 실서열 캐시로 폴백하되 **`passed=false` 유지 (게이트 통과 아님)** |
| **강의 위치** | 강사 데모 (기존 덱 `0912_하네스데모` 28장) | 확장 사례 (본 비교 덱) |

> **설계 철학의 차이 한 줄**: PDE5 하네스는 "알려진 것을 정직하게 정리"하고,
> 항체 하네스는 거기에 "새로 만드는 단계(de novo 설계)"를 얹은 뒤
> **설계물과 기존 항체를 같은 자로 재는 것**을 목표로 한다.

---

## 3. 공통 설계 원칙

### 3.1 표준 봉투 — `{result, provenance, verification}`

두 하네스의 **모든** 과학 스크립트는 동일한 JSON 봉투를 stdout 으로 반환한다.

```json
{
  "result": ...,
  "provenance": {"source": "...", "query": "...", "timestamp": "...Z"},
  "verification": {"passed": true, "checks": [{"check": "...", "passed": true}], "notes": "..."}
}
```

실측 예 (항체 하네스 `antigen_lookup.py P04626`):

```json
"provenance": {"source": "UniProt REST",
               "query": "https://rest.uniprot.org/uniprotkb/P04626.json",
               "timestamp": "2026-08-31T15:17:37.511938Z"}
```

이 계약 하나에서 **체이닝**(stdout → 다음 스크립트 stdin), **자동 검사**(`gate()`),
**역추적**(source/query/timestamp) 이 전부 나온다.

### 3.2 게이트 — `verification.passed=false` 면 다음 단계 금지

`scripts/verify.py` 의 `gate(envelope, step)` 이 stderr 에 한 줄을 찍고 bool 을 돌려준다.

```
[VERIFY-GATE:target-lookup] PASSED
[VERIFY-GATE:...] FAILED → 다음 단계 진행 금지. 실패 항목: [...]
```

`make_result()` 안에서 `passed = all(ok for _, ok in checks)` — **check 하나라도 False 면 실패**다.
게이트 통과 조건은 `CLAUDE.md` 의 표(문서)와 `verify.py`(코드) **양쪽에** 박혀 있다.

### 3.3 Provenance — 값마다 출처 등급이 붙는다

`source` 문자열만 봐도 라이브 조회인지 폴백인지 구분된다. 실측 사례:

| 실측된 `source` 문자열 | 의미 |
|---|---|
| `UniProt REST` | 라이브 조회 성공 |
| `UniProt REST (FAILED)` | 조회 실패 → `result: null`, `passed: false` |
| `UniProt REST [local cache]` | 로컬 캐시 사용 (humanness germline pool) |
| `RCSB PDB Search + Data REST API` | 라이브 조회 성공 |
| `offline cache (PDB 1N8Z, RCSB Data API 2026-08-31 취득)` | 실서열 캐시, **`passed: false`** |
| `ChEMBL (OFFLINE DEMO — 공개 대표 저해제, 실데이터 아님)` | 오프라인 폴백 |

### 3.4 무-날조 (No-Fabrication)

두 하네스 모두 "**모르는 것을 모른다고 말하는 것**"을 성공 조건으로 삼는다.

- PDE5: 수치는 도구 실계산값만 / 모든 사실 주장에 출처 / 불확실은 불확실로 / 결과=가설 (5조)
- 항체: 위에 더해 **서열 창작 금지 · API 창작 금지 · GPU 없이 돌린 척 금지 ·
  근사를 정확이라 말하지 않기 · 지표 발명 금지 · 규칙≠예측 · 키 출력 금지** (12조)

항체 하네스의 12조는 저분자에 없던 실패 양식(서열을 "대략 이렇다"로 만들어내기,
모델 API 함수명을 지어내기, GPU 없이 설계 결과를 상상하기)을 겨냥해 추가된 것이다.

---

## 4. 하네스 ① PDE5 (저분자)

### 4.1 구성

```
pde5_harness/
├── CLAUDE.md              에이전트 지시서 (47줄)
├── README.md              (163줄)
├── run_harness.py         check / run / list (124줄)
├── Dockerfile
├── scripts/
│   ├── verify.py          make_result · gate · valid_smiles · numbers_backed (63줄)
│   ├── target_lookup.py   UniProt O76074
│   ├── chembl_actives.py  ChEMBL CHEMBL1827
│   ├── mol_properties.py  RDKit 물성 + Ro5/QED/SA 게이트
│   ├── selectivity.py     PDE5 vs PDE6 (정성)
│   ├── mol_utils.py       (옵션) RCSB PDB 다운로드
│   └── docking.py         (옵션) smina/vina 래퍼
└── .claude/{skills,hooks,settings.json}
```

### 4.2 실측 결과

**환경 점검** — `python run_harness.py check` → **PASS**

```
[dep] rdkit                        ok
[dep] requests                     ok
[dep] chembl_webresource_client    missing(옵션)
[script] verify/target_lookup/chembl_actives/mol_properties/selectivity  전부 ok
[rdkit] smoke test: ethanol MW=46.07  ok
결과: PASS — 자율 실행 준비 완료
```

**파이프라인** — `python run_harness.py run` → 4 게이트 전부 PASSED

```
[VERIFY-GATE:target-lookup]   PASSED
[VERIFY-GATE:chembl-actives]  PASSED
[VERIFY-GATE:mol-properties]  PASSED
[VERIFY-GATE:selectivity-check] PASSED
```

**① target_lookup.py** (UniProt REST 라이브)

| 필드 | 값 |
|---|---|
| accession | `O76074` |
| gene | `PDE5A` |
| protein | `cGMP-specific 3',5'-cyclic phosphodiesterase` |
| length | `875` |
| function | "… catalyzes the specific hydrolysis of cGMP to 5'-GMP (PubMed:15489334, PubMed:9714779) …" |

**② chembl_actives.py** — 이번 환경에서는 `chembl_webresource_client` 미설치 →
**오프라인 폴백**으로 동작했다. `source` 에 `ChEMBL (OFFLINE DEMO — 공개 대표 저해제, 실데이터 아님)` 표기.
반환된 3건은 ID·SMILES 자체는 실재값이며 RDKit 파싱 3/3 통과.

| molecule_chembl_id | pref_name |
|---|---|
| `CHEMBL192` | SILDENAFIL |
| `CHEMBL779` | TADALAFIL |
| `CHEMBL1520` | VARDENAFIL |

**③ mol_properties.py** — RDKit **2026.03.4** 실계산값

| 화합물 | MW | logP | HBD | HBA | TPSA | QED | SA | Ro5_pass | gate_pass |
|---|---|---|---|---|---|---|---|---|---|
| sildenafil | 474.6 | 1.61 | 1 | 7 | 113.4 | 0.553 | 2.74 | true | true |
| tadalafil | 389.4 | 2.21 | 1 | 4 | 74.9 | 0.693 | 3.28 | true | true |
| vardenafil | 488.6 | 2.07 | 1 | 7 | 112.9 | 0.516 | 2.77 | true | true |

게이트 상수는 코드에 박힌 **교육용 데모값** — `QED_MIN = 0.5`, `SA_MAX = 6.0`,
Ro5 는 4항목 중 3개 이상 충족(위반 1개 허용).

> ⚠️ **새로 확인된 재현성 이슈**: 같은 SMILES·같은 스크립트인데
> **RDKit 2024.03.5 에서는 sildenafil/vardenafil 의 HBA 가 8** 로 나온다(2026.03.4 에서는 7).
> `Lipinski.NumHAcceptors` 의 정의가 버전 간 달라진 결과다. MW·logP·TPSA·QED·SA 는 두 버전에서 동일했다.
> → **디스크립터 값을 인용할 때는 라이브러리 버전을 함께 적어야 한다.**
> 이 하네스는 provenance 에 `source: "RDKit Descriptors/QED"` 만 남기고 **버전은 남기지 않는다** → 개선 여지.

**④ selectivity.py**

```json
"result": {"quantitative_selectivity_predicted": false,
           "assessment": "정성(qualitative) — 정량 fold-selectivity 미산출", ...}
"verification": {"passed": true}
```

첫 번째 check 항목이 **"정량 fold-selectivity 미날조(정성 보고)"** 다.
즉 **억지 수치를 내면 오히려 게이트가 실패**하도록 설계돼 있다.
참고문헌은 실재하는 2편만 반환한다 (Boolell 1996 *Int J Impot Res* 8(2):47-52;
Ghofrani 2006 *Nat Rev Drug Discov* 5(8):689-702, doi:10.1038/nrd2030).

**⑤ docking.py (옵션)** — 실행 결과 `docking_executed: false`

```json
"result": {"docking_executed": false,
           "reason": "전제 조건 미충족 — 도킹 미실행(구조·smina 필요)",
           "missing": ["수용체 파일(--receptor, PDBQT)", "리간드 파일(--ligand, PDBQT)",
                       "도킹 박스 중심(--center X Y Z)", "도킹 박스 크기(--size SX SY SZ)"],
           "smina_found": "/home/hjpark/vina/smina"}
"verification": {"passed": true}   // "도킹 미실행을 정직하게 보고" 가 통과 조건
```

이 환경에는 smina 바이너리가 **실제로 있었지만**(`/home/hjpark/vina/smina`)
수용체/리간드 PDBQT 와 박스 파라미터가 없어 미실행으로 종료했다.

### 4.3 한계 (PDE5)

1. **`docking.py` 는 도킹 엔진이 아니다.** PATH 의 smina/vina 를 `subprocess` 로 부르는 **래퍼**이며,
   전제 4가지(바이너리·수용체·리간드·박스) 중 하나라도 없으면 `docking_executed: false` 로 끝난다.
   스코어를 만들어내지 않는다.
2. **정량 선택성 없음.** PDE5 vs PDE6 는 정성 서술 + "확인 필요 근거 목록"만 반환한다.
   fold-selectivity 는 병렬 IC50/Ki 어세이 또는 인용된 문헌 수치가 있어야만 말할 수 있다.
3. **오프라인 폴백이 게이트를 통과한다.** `chembl_actives.py` 의 3개 check
   (레코드 존재 / SMILES 유효 / ID 형식) 는 **데이터의 출처 등급을 보지 않는다.**
   `source` 문자열에 `OFFLINE DEMO … 실데이터 아님` 이 찍히지만 `passed: true` 다.
   → 항체 하네스는 이 지점을 고쳐 폴백 시 `passed: false` 를 유지한다 (§5.3 참조).
4. **디스크립터 버전 의존성** (위 §4.2 박스).
5. **임계값이 교육용 상수** — `QED_MIN=0.5`, `SA_MAX=6.0` 은 데모값이지 도메인 기준이 아니다.
6. **결과 = 가설.** 알려진 화학공간의 선별·정리 시연이며 신약 발견이 아니다.

---

## 5. 하네스 ② 항체 (바이오로직스)

### 5.1 구성

```
antibody_harness/
├── CLAUDE.md            에이전트 지시서 (255줄, 하드규칙 12조)
├── README.md            (234줄)
├── RUNPOD_가이드.md      GPU 실행 절차 (274줄)
├── run_harness.py       check / design-check / run / list (357줄)
├── scripts/
│   ├── verify.py        (155줄)   seq_utils.py (372줄)
│   ├── antigen_lookup.py (168)  antibody_search.py (295)
│   ├── cdr_analysis.py   (207)  developability.py (298)  humanness.py (302)
│   ├── design_esmfold2.py  (448)  ← 경로 A
│   ├── design_rfantibody.py (493) ← 경로 B
│   └── compare_designs.py  (288)
├── vendor/binder_design.py   공식 프로토콜 원본 사본 (1497줄, sha256 검증)
└── outputs/  01_antigen · 02_antibodies · 03_cdr · 04_developability · 05_humanness · design_a_dryrun
```

### 5.2 실측 결과 — 평가 파이프라인 (CPU, 실행 검증됨)

**`run_harness.py check` → PASS**

```
[dep] requests ok / Bio ok / abnumber missing(옵션) / anarci missing(옵션)
[script] 11종 전부 ok
[biopython] ProtParam smoke: MW=2414.69 pI=4.53  ok
[biopython] Aligner smoke: BLOSUM62 local score=49.0  ok
[cdr] 휴리스틱 자기검증(트라스투주맙 Kabat CDR 6종 재현): ok
[cdr] 번호매김 백엔드: heuristic  (휴리스틱 근사 — IMGT 정확 번호 아님)
[verify] 표준 봉투 계약 self-test: ok
결과: PASS — 자율 실행 준비 완료
```

**`run_harness.py design-check` → PASS** (§6 에서 상세)

**`run_harness.py run P04626` → 5 게이트 전부 PASSED**

```
[VERIFY-GATE:antigen-lookup]  PASSED
[VERIFY-GATE:antibody-search] PASSED
[VERIFY-GATE:cdr-analysis]    PASSED
[VERIFY-GATE:developability]  PASSED
[VERIFY-GATE:humanness]       PASSED
```

**① antigen_lookup.py P04626** (UniProt REST 라이브, 7 checks)

| 필드 | 값 |
|---|---|
| accession / uniprot_id | `P04626` / `ERBB2_HUMAN` |
| gene (synonyms) | `ERBB2` (HER2, MLN19, NEU, NGL) |
| protein_name | `Receptor tyrosine-protein kinase erbB-2` |
| organism / taxon | `Homo sapiens` / 9606 |
| length | `1255` |
| 세포외 도메인 | 23–652 (항체 접근 가능 영역 특징 1건 검출) |

**② antibody_search.py P04626 6** (RCSB Search + Data REST, 실시간)

쿼리: `UniProt accession=P04626 AND polymer_entity_count_protein>=2; max_entries=6`

| PDB | 해상도(Å) | 방법 | 항체 사슬 |
|---|---|---|---|
| `1N8Z` | 2.52 | X-ray | light 214 aa · heavy 220 aa (Herceptin Fab) |
| `1S78` | 3.25 | X-ray | light 214 · heavy 226 (pertuzumab) |
| `3BE1` | 2.90 | X-ray | heavy 230 · light 218 (dual specific bH1 Fab) |
| `3H3B` | 2.45 | X-ray | light(scFv) 259 |
| `3N85` | 3.20 | X-ray | light 217 · heavy 224 |
| `3WLW` | 3.088 | X-ray | heavy 217 · light 217 |

합계 **6 entry / 항체 사슬 11건**. 1N8Z heavy 앞 40자 (RCSB 원값):

```
EVQLVESGGGLVQPGGSLRLSCAASGFNIKDTYIHWVRQA
```

> 중쇄/경쇄 판별은 **보존 Ig 모티프 휴리스틱**이다 (실험적 확인 아님).
> 근거가 봉투에 남는다: `["heavy FR4 motif 'WGQGT' at index 109", "FR1-Cys..FR2-Trp motif at index 21"]`

**③ cdr_analysis.py** — 백엔드 `heuristic (approximate) — Kabat-like, conserved-motif regex`

| 사슬 | CDR1 | CDR2 | CDR3 |
|---|---|---|---|
| 1N8Z_B (heavy) | `DTYIH` | `RIYPTNGYTRYADSVKG` | `WGGDGFYAMDY` |
| 1N8Z_A (light) | `RASQDVNTAVA` | `SASFLYS` | `QQHYTTPPT` |
| 1S78_DF (heavy) | `DYTMD` | `DVNPNSGGSIYNQRFKG` | `NLGPSFYFDY` |
| 1S78_CE (light) | `KASQDVSIGVA` | `SASYRYT` | `QQYYIYPYT` |

1N8Z 6종은 **트라스투주맙 문헌 Kabat CDR 공개값과 일치**한다
(`run_harness.py check` 의 자기검증 항목이 바로 이 6개를 하드코딩해 대조한다 → ok).

**④ developability.py** — BioPython ProtParam 실계산 + 정규식 liability 스캔 (사슬 11건)

| 사슬 | MW (Da) | pI | GRAVY | instability | liability(총) | CDR 내부 |
|---|---|---|---|---|---|---|
| **1N8Z_B** (heavy) | **23403.96** | **8.83** | **-0.18** | **40.32** | 19 | **6** |
| 1N8Z_A (light) | 23442.81 | 7.76 | -0.4523 | 54.43 | 9 | 1 |
| 1S78_DF (heavy) | 24135.77 | 8.77 | -0.2177 | 39.21 | 15 | 2 |
| 1S78_CE (light) | 23525.94 | 7.74 | -0.4294 | 47.79 | 8 | 0 |
| 3BE1_H / 3BE1_L | 24533.20 / 23933.36 | 8.74 / 7.76 | -0.23 / -0.4647 | 42.56 / 51.31 | 20 / 9 | 6 / 1 |
| 3H3B_CD (scFv-L) | 27536.13 | 9.00 | -0.4293 | 38.01 | 14 | 3 |
| 3N85_H / 3N85_L | 23830.45 / 23875.28 | 8.79 / 7.76 | -0.1353 / -0.423 | 43.65 / 56.38 | 22 / 12 | 7 / 4 |
| 3WLW_CH / 3WLW_DL | 23129.57 / 23065.15 | 8.33 / 6.34 | -0.2576 / -0.406 | 37.76 / 52.35 | 19 / 10 | 5 / 0 |

전체 liability 규칙 히트 **157건 (CDR 내부 35건)**.
instability index 판정 기준은 Guruprasad 1990 (>40 = unstable).

**1N8Z_B (트라스투주맙 중쇄) CDR 내부 liability 6건 상세**:

| liability | 모티프 | 위치(1-based) | CDR | severity |
|---|---|---|---|---|
| isomerization_moderate | `DT` | 31 | CDR-H1 | moderate |
| **deamidation_NG** | `NG` | **55** | **CDR-H2** | **high** |
| isomerization_moderate | `DS` | 62 | CDR-H2 | moderate |
| oxidation_Trp | `W` | 99 | CDR-H3 | moderate |
| **isomerization_DG** | `DG` | **102** | **CDR-H3** | **high** |
| oxidation_Met | `M` | 107 | CDR-H3 | moderate |

요약: `n_high_severity: 2`, `n_moderate_severity: 17`, `n_hydrophobic_patches: 8`.

> 이 hotspot 들은 트라스투주맙에 대해 **문헌에 알려진 것과 일치**한다(특히 CDR-H2 의 NG 탈아미드화).
> 즉 규칙 스캔이 "실제로 문제가 되는 자리"를 짚었다 — 다만 아래 한계를 반드시 함께 읽을 것.

**⑤ humanness.py** — germline pool **143개** (UniProt reviewed IGHV/IGKV/IGLV), BLOSUM62 local alignment

| 사슬 | nearest germline | accession | identity(%) | alignment len |
|---|---|---|---|---|
| **1N8Z_A** (light) | **IGKV1-39** | P01597 | **86.32** | 95 |
| **1N8Z_B** (heavy) | **IGHV3-66** | A0A0C4DH42 | **80.61** | — |
| 1S78_CE | IGKV1D-33 | P01593 | 86.81 | — |
| 1S78_DF | IGHV3-23 | P01764 | 77.55 | — |
| 3N85_L | IGKV1D-12 | — | 91.11 | — |
| 3WLW_CH | IGHV1-18 | — | 91.84 | — |
| 3WLW_DL | IGLV2-14 | — | 97.98 | — |

pool 분해: heavy 도메인 대조 pool 59개(IGHV), light 84개(IGKV+IGLV).

정의 그대로 인용: `germline_identity_percent = 100 × (동일 잔기 수) / (BLOSUM62 local alignment 길이)`.
**"휴먼성 점수" 같은 발명 지표는 만들지 않는다** — 정의가 명확한 identity(%) 하나만 보고한다.

### 5.3 네트워크 차단 시 동작 (실제로 막아서 확인)

프록시를 막고(`HTTPS_PROXY=http://127.0.0.1:9`) 재실행한 결과:

| 스크립트 | result | passed | source |
|---|---|---|---|
| `antigen_lookup.py P04626` | `null` | **false** | `UniProt REST (FAILED)` |
| `antibody_search.py P04626 6` | 1 entry (1N8Z 실서열 캐시) | **false** | `offline cache (PDB 1N8Z, RCSB Data API 2026-08-31 취득)` |

**값을 만들어내지 않았다.** 그리고 캐시 폴백조차 `passed: false` 를 유지해
게이트 통과로 취급되지 않는다 — PDE5 하네스의 오프라인 폴백보다 엄격하다.

### 5.4 한계 (항체)

1. **CDR 추출은 휴리스틱 근사다.** `anarci`/`abnumber` 미설치 → 보존 모티프 정규식 기반.
   봉투에 `method: "heuristic (approximate)"` 와 경고가 그대로 남는다
   (예: *"CDR-L2 는 Kabat L50-56 고정 길이(7)를 가정한 근사값"*).
   **자기검증은 트라스투주맙·퍼투주맙 2건으로만 했다** — 비정형 프레임워크·삽입(insertion)·
   비인간 항체에서는 경계가 틀릴 수 있다. **"IMGT 번호매김으로 추출했다"고 쓰면 안 된다.**
2. **liability 는 규칙이지 예측이 아니다.** 정규식 모티프 플래그이며 응집·안정성 예측 모델이 아니다.
   서술은 "이 항체는 응집할 것이다"가 아니라 "NG 탈아미드화 모티프가 CDR-H2 55번에 있다(규칙 기반 플래그)".
3. **germline identity 는 면역원성(ADA) 예측이 아니다.** 서열 유사도 상관 지표다.
   local alignment 이므로 germline signal peptide 와 항체 CDR3/FR4 는 정렬 밖에 있다.
4. **중쇄/경쇄 판별도 휴리스틱**(보존 Ig 모티프)이다.
5. **설계 단계는 이 환경에서 실행되지 않았다** — §6·§7 참조.
6. 이 하네스가 **하지 않는 것**: 친화도(KD) 예측 · 면역원성 예측 · 에피토프 자동 발견 ·
   발현/정제 성공 예측. in silico 결과는 **가설**이고 wet-lab(SPR/BLI·DSF·SEC·PK)이 유일한 판정자다.

---

## 6. 설계 2경로 — ESMFold2 inversion vs RFantibody

### 6.1 개념 비교

| 항목 | **경로 A — ESMFold2 inversion** (주축) | **경로 B — RFantibody** (고전 비교군) |
|---|---|---|
| 원리 | 구조 예측 모델을 **역방향으로 미분**해 바인더 서열을 gradient 최적화 | RFdiffusion(백본) → ProteinMPNN(서열) → RoseTTAFold2(구조 검증) 3단계 |
| 모델 수 | 1개 (folding model 하나로 설계) | 3개 (단계별 다른 모델) |
| 저장소 / 라이선스 | `github.com/Biohub/esm` — **MIT** | `github.com/RosettaCommons/RFantibody` — **MIT** |
| 프로토콜 | `cookbook/tutorials/binder_design.py` (엔드투엔드) | 공식 README 의 CLI 3종 |
| 가중치 | HF `biohub/ESMFold2`, `ESMFold2-Fast`, `ESMC-6B` | `bash include/download_weights.sh` |
| 프리프린트 | biorxiv `10.64898/2026.06.03.729735` | biorxiv `2024.03.14.585103v1` |
| 입력 규격 | preset 항원 5종(`cd45`·`ctla4`·`egfr`·`pd-l1`·`pdgfr`) 또는 `--target-sequence` | **HLT 포맷 PDB** (체인 H→L→T, `REMARK PDBinfo-LABEL` CDR 주석) |
| 바인더 preset | `minibinder`, `trastuzumab_framework_vhvl`, `atezolizumab_framework_vhvl`, `ocankitug_framework_vhvl` | 프레임워크 PDB 사용자 제공 |
| 필터 | 프로토콜 내부 critic 랭킹 (ipTM 등) | **저자 권장 최소 필터**: `RF2 pAE < 10`, `RMSD < 2 Å`, (선택) `Rosetta ddG < -20` |
| GPU 요구 | A100 40GB 이상 (80GB 권장) | NVIDIA GPU, CUDA 11.8+ |
| VRAM (문서 인용) | 공식 주석: `REUSE_ESMC=True` 27GB / `False` 51GB (batch_size=1 기준) | **문서에 수치 명시 없음 — 미확인** |
| 하네스의 API 안전장치 | 공식 `binder_design.py` 를 `vendor/` 로 받아 **그대로 import** (재구현 없음) + sha256 대조 | 공식 README CLI 를 **그대로 호출** (인자 창작 없음) |

### 6.2 경로 A — 검증 성적 (원 저작자 보고, 인용값)

- **wet-lab hit rate**: 5개 타깃(**EGFR · PDGFRβ · PD-L1 · CTLA-4 · CD45**)에서
  **항체 포맷 15–29%**, **미니바인더 36–88%**, nM 친화도.
- **FoldBench**: 항체–항원 DockQ pass-rate 에서 **AF3 상회**
  (`2026_landscape.md`: single-seq 기준 **50% vs AF3 47%** — *2차 소스, 확인 권장*).

> 이 수치들은 **하네스가 계산한 값이 아니라 원 저작자 보고의 인용값**이다.

### 6.3 경로 B — 저자가 스스로 밝힌 한계 (원문 인용)

> *"The lack of an effective filter is the main limitation of the RFantibody pipeline at the moment."*

- 일부 타깃은 **95 designs** 로 VHH binder 를 찾았지만,
  일반적으로는 **10k 규모 캠페인**이 필요할 것으로 저자가 예상한다.
- 따라서 **소수 설계의 순위는 약한 증거다.** 강의에서 반드시 짚을 지점.

### 6.4 두 경로를 하나의 점수로 합치지 않는다

`compare_designs.py` 의 명시 규칙 (코드 주석·봉투 notes 원문):

> "경로 A 의 ipTM 과 경로 B 의 pAE 를 하나의 **종합 점수로 합치지 않는다.
> 그런 통합 점수는 근거 없는 발명이다.**"

정당한 공통 비교축은 **서열만으로 계산되는 지표**뿐이다 —
CDR 길이 / MW / pI / GRAVY / instability index / liability 히트 수 / germline identity(%).
경로별 신뢰도 지표는 **원본 이름 그대로** 병기한다.
이것이 "평가 3종(cdr·developability·humanness)을 설계 서열에도 똑같이 적용한다"는 설계의 이유다.

### 6.5 실행 검증 상태 — `design-check` (GPU 불필요, PASS)

> 아래는 **GPU 노드 배포 이전(2026-08-31 CPU 환경)** 의 사전 점검 결과다.
> 실제 GPU 실행 결과는 §6.6–6.8 에 있다.

```
-- 경로 A: ESMFold2 inversion --
  passed=True
    [ok] 입력 설정 유효 (항원/바인더 조합)
    [ok] 공식 프로토콜 스크립트 확보
    [ok] 프로토콜 내 기대 API 심볼 존재
  공식 프로토콜: 기존 vendor 사본 사용 (sha256 일치=True)
  API 심볼: 공식 프로토콜에서 기대 심볼 전부 확인
  GPU: CUDA 사용 불가

-- 경로 B: RFantibody --
    rfdiffusion / proteinmpnn / rf2 / qvscorefile   전부 missing — GPU 노드에서 설치 필요

결과: PASS — 설계 경로 설정 검증 완료 (실제 설계는 GPU 노드 필요)
```

봉투에 남은 검증 앵커:

| 항목 | 값 |
|---|---|
| vendor 프로토콜 경로 | `antibody_harness/vendor/binder_design.py` (1497줄) |
| sha256 | `28d672a3b1ff722e6d3d50f7538b806bbaf04291c70d83454fa6c912869cd3d3` |
| `matches_verified_sha256` | `true` |
| 검증된 upstream commit | `827ec128e4cdaf80f7d6f95fb367a08980b34918` |
| 환경 | `torch 2.13.0+cpu`, `cuda: false`, `esm 미설치` |
| dry-run notes | "DRY-RUN — 설계를 수행하지 않았습니다. … 설계 서열/점수는 생성하지 않습니다(무-날조)." |

### 6.6 GPU 실측 ① 경로 B RFantibody (A40) — 3단계 완주

**환경**: RunPod A40, `$0.44/hr`. 원본 로그 `runpod_logs/rfantibody/rfab.log` (2733줄) ·
점수표 `runpod_logs/rfantibody/3_rf2.sc` (헤더 1 + 16행).

**실행한 명령 (로그 `+ ` 트레이스 원문)**

```bash
uv run rfdiffusion --target scripts/examples/example_inputs/flu_HA.pdb \
  --framework scripts/examples/example_inputs/h-NbBCII10.pdb \
  --output-quiver .../1_rfdiffusion.qv --num-designs 4 \
  --design-loops H1:7,H2:6,H3:5-13 --hotspots B146,B170,B177 \
  --diffuser-t 50 --deterministic
uv run proteinmpnn --input-quiver .../1_rfdiffusion.qv \
  --output-quiver .../2_proteinmpnn.qv --seqs-per-struct 4 --temperature 0.2
uv run rf2 --input-quiver .../2_proteinmpnn.qv \
  --output-quiver .../3_rf2.qv --hotspot-show-prop 0.0 --num-recycles 10
```

- 타깃은 **인플루엔자 HA** (`flu_HA.pdb`), 프레임워크는 **나노바디** `h-NbBCII10.pdb`
  (저장소 공식 예제 입력 — HLT 포맷). 경쇄 없음 → 로그 `loopL: []`, 점수표 L1/L2/L3 = `NaN`.
- `--deterministic` 을 켰으므로 같은 입력에서 재현 가능하다.

**① RFdiffusion — 4 designs, 설계당 벽시계 시간과 핫스팟 거리 (로그 원값)**

| design | Finished design in | Overall min distance (Å) | Average min distance (Å) |
|---|---|---|---|
| 0 | 3.83 분 | 4.899 | 7.063 |
| 1 | 3.22 분 | 4.954 | 5.681 |
| 2 | 3.08 분 | 5.563 | 6.521 |
| 3 | **3.15 분** | **4.835** | **5.551** |

> 4 designs 합계 **13.28 분** (로그의 네 값을 더한 것). "3.15 분 / 최소 4.83 Å / 평균 5.55 Å"
> 은 **마지막 design(3) 의 값**이며 전체 평균이 아니다 — 인용할 때 반드시 구분할 것.
> 핫스팟 잔기 서열은 로그에 `Sequence of Hotspot Residues: WIV` 로 기록됐다.

**② ProteinMPNN** — `seqs-per-struct 4`, `temperature 0.2` → 4 구조 × 4 = **16 서열**.
구조당 3–7초. 설계 루프 길이는 구조마다 달랐다 (H3 가변 `5-13` 이므로):
design 0 = 18 잔기, design 1 = 22, design 2 = 19, design 3 = 23.

**③ RF2** — `--num-recycles 10` (= 로그상 11 cycle) → **16 예측**.
마지막 예측(`samples_design_3_dldesign_3`)의 pLDDT 궤적: cycle 1 `0.884` → cycle 11 `0.902`
(best `0.902`) — **재활용이 실제로 수렴한다는 증거**.

**16개 설계 점수 통계 (`3_rf2.sc` 를 직접 집계)**

| 지표 | min | max | avg |
|---|---|---|---|
| `pred_lddt` | 0.90 | 0.92 | **0.91** |
| `pae` | 3.12 | 9.82 | 8.18 |
| `interaction_pae` | **6.22** | 22.24 | 18.57 |
| `framework_aligned_cdr_rmsd` (Å) | 0.76 | 2.03 | 1.23 |
| `framework_aligned_H3_rmsd` (Å) | 0.64 | 3.01 | 1.61 |

**최고 설계**: `samples_design_1_dldesign_3_best` — `interaction_pae` **6.22**,
`pae` 3.12, `pred_lddt` 0.91, `target_aligned_antibody_rmsd` 1.59 Å,
`framework_aligned_cdr_rmsd` 0.76 Å.

**⭐ 저자 권장 필터를 실제로 적용한 결과 (이번 실행의 가장 중요한 발견)**

`interaction_pae < 10` 을 16개에 그대로 적용하면 **통과 1건 / 16건 (6.3%)** —
`samples_design_1_dldesign_3_best` 하나뿐이다. 나머지 15건은 17.67–22.24 로 필터 밖이다.
`pred_lddt` 는 16건 전부 0.90–0.92 로 **거의 구분력이 없었다**.

> 이것이 §6.3 에서 인용한 저자 문장
> *"The lack of an effective filter is the main limitation…"* 과
> *"일반적으로 10k 규모 캠페인이 필요할 것"* 을 **우리 손으로 재현한 숫자**다.
> 4 designs 로는 필터를 통과하는 설계가 1개밖에 나오지 않는다.

**산출물 크기**: `1_rfdiffusion.qv` 431 KB · `2_proteinmpnn.qv` 1.7 MB ·
`3_rf2.qv` 6.5 MB · `3_rf2.sc` (16행 TSV).

---

### 6.7 GPU 실측 ② 경로 A ESMFold2 inversion (H100) — 설계 성공

**환경**: RunPod H100, `$3.29/hr`. `esm 3.4.0` (upstream commit `827ec128`, 2026-08-27),
`torch 2.11.0+cu130`, Python 3.12.
원본 로그 `runpod_logs/esmfold2_design_run.log` · 결과 `esmfold2_design_results.json`.

**설계 설정** (어댑터 `antibody_harness/esmfold2_public_release_adapter.py` 의 `app.design(...)` 호출 원문)

```python
app.load(False)                              # use_scaling_critics=False
seq, traj, results = app.design(
    target_name="pd-l1", binder_name="trastuzumab_framework_vhvl",
    target_sequence=None, binder_sequence=None, is_antibody=True,
    seed=0, batch_size=1, target_hotspot_ids=None, epitope_contact_distance=12.0)
```

**시간 실측**

| 구간 | 값 | 근거 |
|---|---|---|
| `MODEL_LOAD_SEC` (최초, 가중치 캐시 없음) | **100.1 초** | 1차 실행 로그 |
| `MODEL_LOAD_SEC` (2차, HF 캐시 존재) | **29.3 초** | `esmfold2_design_run.log` |
| `DESIGN_SEC` | **191.3 초** | 같은 로그 |
| 최적화 궤적 길이 | **150 스텝** (`TRAJ_LEN: 150`) | 같은 로그 |
| 설계 루프 VRAM | `peak_allocated_gib` **25.08** / `peak_reserved_gib` **25.59** | 스텝 로그 매 줄 |

> VRAM 25.08/25.59 GiB 는 **`use_scaling_critics=False`, `batch_size=1`, PD-L1(115aa) 타깃**
> 조건의 값이다. §6.1 표의 27GB/51GB 는 공식 주석 인용값이므로 조건이 다르며 직접 비교 불가.

**설계 산출 서열** — scFv **244 aa** = VH 119 + `(GGGS)×4` 링커 16 + VL 109 (119+16+109=244).

```
VH  EVQLVESGGGLVQPGGSLRLSCAASGPFEGHIIYIHWVRQAPGKGLEWVARILVLVGATRYADSVKGRF
    TISADTSKNTAYLQMNSLRAEDTAVYYCSREDLTEHETDWGQGTLVTVSS
VL  DIQMTQSPSSLSASVGDRVTITCRVSNYDSDDMLIDWYQQKPGKAPKLLIYETDELASGVPSRFSGSRS
    GTDFTLTISSLQPEDFATYYCFDDENFPLTFGQGTKVEIK
```

프레임워크는 트라스투주맙 그대로 유지되고 **CDR 만 재설계**됐음이 눈으로 확인된다 —
예: CDR-H3 가 트라스투주맙 `WGGDGFYAMDY` → 설계 `SREDLTEHETD` 로 바뀌었다.
(`EVQLVESGGGLVQPGGSLRLSCAAS…`, `…WGQGTLVTVSS`, `DIQMTQSPSSLSASVGDRVTITC…` 등
프레임워크 구간은 원본과 동일.)

**⭐ critic 4종의 ipTM 편차 — 이번 실행의 두 번째 중요 발견**

**같은 서열, 같은 `final_loss` 3.611** 인데 채점자를 바꾸면 점수가 크게 달라진다.

| critic | ipTM | distogram ipTM proxy | CDR distogram ipTM proxy |
|---|---|---|---|
| `ESMFold2-Experimental-Fast` | **0.8938** | 0.8843 | 0.8967 |
| `ESMFold2-Experimental-Fast-Cutoff2025` | 0.8615 | 0.7119 | 0.7357 |
| `ESMFold2-Experimental` | **0.5218** | 0.5927 | 0.6125 |
| `ESMFold2-Experimental-Cutoff2025` | 0.7879 | 0.6721 | 0.6781 |

최고 0.8938 vs 최저 0.5218 — **동일 설계에 대해 0.37 의 폭**이다.

> **교훈**: "ipTM 0.89 를 얻었다"는 문장은 **어느 critic 인지 밝히지 않으면 무의미하다.**
> 하나만 골라 인용하면 그 자체가 cherry-picking 이다. 이 하네스가 §6.4 에서
> "두 경로 점수를 합치지 않는다"고 못 박은 이유가 **한 경로 안에서도** 성립한다.
> `Cutoff2025` 계열은 2025년 컷오프 데이터로 학습된 별도 체크포인트이므로
> 학습 데이터 누출(leakage) 통제 여부에 따라 점수가 달라지는 것으로 보인다 —
> **다만 그 인과는 우리가 검증하지 않았다(가설).**

---

### 6.8 설계 2경로 실측 비교

**주의**: 아래 표는 §6.4 규칙대로 **두 경로 점수를 하나로 합치지 않는다.**
타깃도 포맷도 다르므로 (HA/나노바디 vs PD-L1/scFv) **성능 비교가 아니라 실행 특성 비교**다.

| 항목 | 경로 A — ESMFold2 inversion | 경로 B — RFantibody |
|---|---|---|
| GPU / 단가 | H100 · `$3.29/hr` | A40 · `$0.44/hr` |
| 타깃 | PD-L1 preset (115 aa) | 인플루엔자 HA (`flu_HA.pdb`) |
| 바인더 포맷 | scFv 244 aa (VH 119 + 링커 16 + VL 109) | 나노바디 VHH (`h-NbBCII10`, 경쇄 없음) |
| 모델 로드 | 100.1 초 (최초) / 29.3 초 (캐시) | (별도 계측 없음 — 3단계 CLI 각각 로드) |
| 설계 시간 | **191.3 초** (150 스텝, 설계 1건) | **13.28 분** (RFdiffusion 4 designs 합계) + MPNN 초 단위 + RF2 |
| 산출 설계 수 | 1 서열 (batch_size=1) | 16 서열 (4 백본 × 4 서열) |
| 결정론 | `seed=0` | `--deterministic` |
| 신뢰도 지표 | **ipTM** (critic 4종) — 0.5218 / 0.7879 / 0.8615 / 0.8938 | **RF2 interaction_pae / pae / pred_lddt** |
| 필터 적용 결과 | critic 간 편차 0.37 → 단일 임계값 설정 불가 | `interaction_pae<10` → **1/16 통과 (6.3%)** |
| 구조 자기일관성 | (이번 실행에서 미산출) | `framework_aligned_cdr_rmsd` 0.76–2.03 Å (avg 1.23) |
| 필요했던 코드 수정 | **4건** (§6.9) — 알고리즘 수정 0 | 0건 — 공식 CLI 그대로 |
| 이번 실행 비용 | H100 2회 | A40 1회 |

**두 경로 모두에서 확인된 공통점**

1. **필터가 병목이다.** 경로 B 는 16건 중 1건만 통과했고, 경로 A 는 critic 을 바꾸면
   같은 설계가 0.52 도 되고 0.89 도 된다. 설계를 만드는 것보다 **고르는 것**이 어렵다.
2. **소수 설계의 순위는 약한 증거다.** 4 designs / 1 설계는 둘 다 통계적 주장을 할 수 있는
   규모가 아니다 — 저자가 말한 10k 캠페인과 3–4 자릿수 차이다.
3. **둘 다 in silico 가설 생성기다.** wet-lab 결합 데이터는 이번 실행에 **없다.**

**총 비용**: H100 2회 + A40 1회 = **약 $5**. 작업 후 pod 전부 종료, 잔여 **0** 확인.

---

### 6.9 재현성 교훈 — 이전 오판 정정

> ### ⚠️ 정정: 이전 판의 "ESMFold2 는 버전 스큐로 실행 불가" 는 **틀렸다**
>
> 이전 판 보고서와 강의 슬라이드는 공개 릴리스(`esm 3.4.0`)의 `forward()` 시그니처가
> 쿡북 프로토콜과 맞지 않는 것을 보고 **"버전 스큐 — 공개 릴리스에서는 실행 불가"** 로
> 결론지었다. 최초 실패 로그는 이랬다 (`runpod_logs/esmfold2_design.log`):
>
> ```
> TypeError: EsmFold2ExperimentalModel.forward() got unexpected keyword
>            argument(s) ['calculate_confidence', 'disto_cond', 'disto_cond_mask']
> ```
>
> **그 판단은 성급했다.** 저장소 전체(모델 코드 · 테스트 · Modal 이미지 정의)를 뒤진 결과
> 이 세 인자는 **추론에서 쓰이지 않거나 자동 계산되는 것**이었고,
> 아래 **4가지 최소 어댑테이션만으로 공개 릴리스에서 그대로 돌았다.**
> **알고리즘은 한 줄도 수정하지 않았다.**

**필요했던 것은 이 4가지뿐이다**

| # | 조치 | 근거 (저장소에서 확인) |
|---|---|---|
| 1 | `disto_cond` / `disto_cond_mask` 인자 **제거** | 저장소 자체 테스트 `tests/models/esmfold2_inputs_test.py` 의 `FAST_PATH_OMITS` 가 *"inference does not use"* 라고 명시. 게다가 `prepare_input` 은 conditioning 미지정 시 **전부 0/False** 를 반환 → **제거는 no-op** |
| 2 | `calculate_confidence` 인자 **제거** | `esm/models/esmfold2/experimental.py` 의 `forward` 가 `if self.confidence_head is not None:` 로 **confidence 를 자동 계산**한다 → 인자만 빼면 값은 그대로 나온다 |
| 3 | `model._esmc` → `model.esmc` | 공개 릴리스에서 속성이 개명됨 (private → public) |
| 4 | **`abnumber` 설치** (+ `hmmer`, `anarci`) | **실제 마지막 블로커.** 공식 Modal 이미지는 micromamba 로 이것들을 깔지만, uv/pip 경로에는 들어오지 않는다. `is_antibody=True` 경로가 CDR 번호매김에 이걸 쓴다 |

1·2 의 근거는 어댑터 파일 `antibody_harness/esmfold2_public_release_adapter.py` 의
docstring 에 원문 그대로 남겨 두었다. 어댑터는 `fold_and_get_distogram` 을 감싸는
얇은 shim 하나이며 `kwargs.pop(...)` 3줄이 전부다.

**추가로 얻은 환경 교훈**

| 교훈 | 상세 |
|---|---|
| `esm` 은 **Python ≥ 3.12** 를 요구한다 | 3.11 이하로는 설치 자체가 되지 않는다 |
| venv 는 **로컬 디스크**에 만들어야 한다 | RunPod 의 `/workspace` 는 네트워크 FS — 여기에 venv 를 만들면 패키지 설치가 극심하게 느려진다. `/root/v312` 처럼 로컬 경로 사용 (실제 로그 경로가 `/root/v312/lib/python3.12/...` 인 이유) |
| `modal` 패키지가 **로컬 실행에도 필요**하다 | Modal 을 쓰지 않아도 쿡북 모듈이 import 하므로 설치해야 한다 |
| 커널 폴백 경고는 정상이다 | `transformer-engine` / `xformers` / `flash-attn` 미설치 시 순수 PyTorch 폴백 경고가 뜬다. 로그는 **최종 LayerNorm 이후 몇 ULP 차이, perplexity 는 rounding noise 내**라고 명시 — 결과 무효 사유가 아니다 |

**이 사건에서 가져갈 방법론**

1. **"버전 스큐라서 안 된다"는 결론은 저장소를 다 읽기 전에는 내리면 안 된다.**
   답은 저장소의 **자체 테스트 파일** 안에 문장으로 적혀 있었다 (`FAST_PATH_OMITS`).
2. **에러 메시지가 가리키는 곳이 원인이 아닐 수 있다.** `TypeError` 3개를 다 없앤 뒤에도
   막힌 진짜 원인은 전혀 다른 곳 — **의존성 하나(`abnumber`)** 였다.
3. **컨테이너 정의 파일은 1급 문서다.** Modal 이미지 정의가 "이 코드가 실제로 필요로 하는
   시스템 의존성 목록"이었다. README 에는 없었다.
4. **틀린 결론은 지우지 말고 정정으로 남긴다.** 이 절 자체가 그 실천이다 —
   이전 판을 본 사람이 "왜 말이 바뀌었나"를 추적할 수 있어야 한다.

---

## 7. 검증 매트릭스

범례: ✅ 실행으로 확인 · 🆕 **2026-09-01 RunPod 실행으로 새로 검증됨** ·
🟡 여전히 미검증 · ⚠️ 확인했으나 한계 있음 · 📄 문서/원저자 인용값

### 7.1 PDE5 하네스

| # | 항목 | 상태 | 근거 |
|---|------|------|------|
| 1 | `run_harness.py check` | ✅ PASS | rdkit ok, requests ok, 스크립트 5종 ok, ethanol MW=46.07 |
| 2 | `run_harness.py run` 4 게이트 | ✅ 전부 PASSED | stderr `[VERIFY-GATE:*] PASSED` ×4 |
| 3 | target_lookup (UniProt O76074) | ✅ | PDE5A / 875 aa / cGMP-specific 3',5'-cyclic phosphodiesterase |
| 4 | chembl_actives 3건 | ⚠️ **오프라인 폴백** | ID·SMILES 는 실재값, 라이브 조회 아님. 그런데도 `passed: true` |
| 5 | mol_properties (sildenafil 등 3건) | ✅ | MW/logP/HBD/HBA/TPSA/QED/SA 실계산 (§4.2 표) |
| 6 | HBA 버전 의존성 (7 vs 8) | ✅ **재현 확인** | RDKit 2026.03.4 → 7 / 2024.03.5 → 8 |
| 7 | 후보 0건 → `passed: false` | ✅ | `mol_properties.py` 인자 없이 실행 → `result: []`, checks 2개 모두 false |
| 8 | selectivity 정성 보고 | ✅ passed=true | `quantitative_selectivity_predicted: false` 가 통과 조건 |
| 9 | 정량 fold-selectivity | ❌ 미산출(설계상) | 병렬 어세이/문헌 인용 필요 |
| 10 | docking 실행 | 🟡 미실행 | `docking_executed: false`, smina 는 있으나 PDBQT·박스 없음 |
| 11 | 실제 도킹 스코어 | 🟡 미검증 | 래퍼일 뿐 — 엔진 아님 |
| 12 | Docker 이미지 빌드 | 🟡 미검증 | `Dockerfile` 존재하나 이번에 빌드하지 않음 |

### 7.2 항체 하네스

| # | 항목 | 상태 | 근거 |
|---|------|------|------|
| 1 | `run_harness.py check` | ✅ PASS | BioPython 1.88, ProtParam/Aligner smoke ok, verify self-test ok |
| 2 | CDR 휴리스틱 자기검증 | ✅ ok | 트라스투주맙 Kabat CDR 6종 재현 |
| 3 | `design-check` | ✅ PASS | 경로 A dry-run passed=True, sha256 일치 |
| 4 | `run P04626` 5 게이트 | ✅ 전부 PASSED | stderr `[VERIFY-GATE:*] PASSED` ×5 |
| 5 | antigen_lookup P04626 | ✅ 7 checks | ERBB2_HUMAN / 1255 aa / ECD 23–652 |
| 6 | antibody_search (RCSB 실시간) | ✅ 6 checks | 6 entry · 11 사슬 · 1N8Z 2.52 Å 등 |
| 7 | CDR 추출 11 도메인 | ⚠️ **휴리스틱 근사** | `method: heuristic (approximate)`; 검증은 2건(트라스투주맙·퍼투주맙)뿐 |
| 8 | developability 11 사슬 | ✅ 실계산 | ProtParam MW/pI/GRAVY/instability (§5.2 표) |
| 9 | liability 157건(CDR 35) | ⚠️ **규칙 플래그** | 정규식 모티프 — 예측 모델 아님 |
| 10 | humanness 11 도메인 | ✅ 실계산 | germline pool 143, BLOSUM62 local identity |
| 11 | germline identity ↔ 면역원성 | ⚠️ 상관 지표 | ADA 예측 아님 |
| 12 | 네트워크 차단 시 무-날조 | ✅ 확인 | `result: null` / 캐시도 `passed: false` |
| 13 | 경로 A 설계 실제 실행 | 🆕 ✅ **검증됨** | H100 · `esm 3.4.0` · `DESIGN_SEC 191.3` · scFv 244 aa 산출 (§6.7) |
| 14 | 경로 B 3단계 실제 실행 | 🆕 ✅ **검증됨** | A40 · RFdiffusion 4 → MPNN 16 → RF2 16 완주 (§6.6) |
| 15 | HF 가중치 다운로드 | 🆕 ✅ **검증됨** | `Fetching 2/11 files` 로그 + 캐시 재사용으로 로드 100.1초→29.3초 |
| 16 | 실행시간 / RunPod 비용 | 🆕 ✅ **검증됨** | A40 `$0.44/hr` · H100 `$3.29/hr` · 총 **약 $5** · 잔여 pod 0 |
| 17 | VRAM | 🆕 ✅ **부분 실측** | 경로 A 설계 루프 `peak_allocated 25.08 GiB` / `reserved 25.59 GiB` (PD-L1·bs=1·critics off). 27/51GB 는 여전히 📄 공식 주석 인용값 |
| 18 | 경로 A wet-lab hit rate 15–29% / 36–88% | 📄 원저자 보고 | 하네스 산출값 아님 — **이번 실행에도 wet-lab 데이터 없음** |
| 19 | FoldBench DockQ AF3 상회 | 📄 2차 소스 | `2026_landscape.md` — 확인 권장 |
| 20 | 경로 B 저자 한계 인용문 | 🆕 ✅ **실측으로 재현** | `interaction_pae<10` 통과 **1/16 (6.3%)** — 필터가 병목임을 우리 손으로 확인 (§6.6) |
| 21 | `compare_designs.py` 실제 두-경로 비교 | 🟡 **여전히 미검증** | 두 경로를 **다른 타깃**(HA vs PD-L1)으로 돌려 동일 타깃 비교 입력이 아직 없음 |
| 22 | 경로 A critic 간 ipTM 편차 | 🆕 ✅ **검증됨** | 동일 서열·동일 `final_loss` 3.611 에서 0.5218–0.8938 (폭 0.37) (§6.7) |
| 23 | 공개 릴리스(`esm 3.4.0`)에서 경로 A 실행 가능 여부 | 🆕 ✅ **가능 — 이전 "불가" 판단 정정** | 최소 어댑테이션 4건만 필요, 알고리즘 수정 0 (§6.9) |
| 24 | 설계 서열에 대한 평가 3종(cdr·developability·humanness) 적용 | 🟡 **여전히 미검증** | 설계 서열은 확보했으나 평가 파이프라인에 아직 투입하지 않음 |

---

## 8. 무-날조가 실제로 작동한 사례

### 사례 ① 후보 0건 → 빈 표도 만들지 않는다 (PDE5)

```bash
$ python scripts/mol_properties.py        # 입력 SMILES 없음
```
```json
{"result": [],
 "verification": {"passed": false,
   "checks": [{"check": "후보 존재", "passed": false},
              {"check": "값 sanity 범위 내", "passed": false}],
   "notes": "물성 계산 0건, 게이트(Ro5≥3·QED≥0.5·SA≤6.0) 통과 0건."}}
```

여기서 파이프라인은 멈춘다. **빈 표도, 평균값도, "대표적인 PDE5 저해제의 전형적 물성"도 만들지 않는다.**

### 사례 ② 네트워크 차단 → 서열을 지어내지 않는다 (항체)

```bash
$ HTTPS_PROXY=http://127.0.0.1:9 python scripts/antigen_lookup.py P04626
```
```json
{"result": null,
 "provenance": {"source": "UniProt REST (FAILED)"},
 "verification": {"passed": false,
   "notes": "UniProt 조회 실패: 네트워크 오류: ProxyError: … rest.uniprot.org …"}}
```

`antibody_search.py` 는 1N8Z **실서열 캐시**로 폴백하되 `passed: false` 를 유지한다
(`source: "offline cache (PDB 1N8Z, RCSB Data API 2026-08-31 취득)"`).
**캐시는 "값이 있음"이지 "게이트 통과"가 아니다.**

### 사례 ③ (보너스) GPU 없으면 설계 결과를 상상하지 않는다

`design_esmfold2.py --dry-run` 봉투 notes 원문:

> "DRY-RUN — 설계를 수행하지 않았습니다. 설정·환경·공식 API 심볼만 검증.
> **설계 서열/점수는 생성하지 않습니다(무-날조).** GPU: CUDA 사용 불가"

**후일담 (2026-09-01)**: GPU 노드를 확보한 뒤 이 자리는 **상상이 아니라 실측값**으로 채워졌다
(§6.6–6.8). 무-날조의 목적은 "영원히 값을 비워 두는 것"이 아니라
**값이 생길 때까지 빈칸을 빈칸으로 유지하는 것**이다.

### 사례 ④ 틀린 결론을 지우지 않고 정정으로 남긴다

이전 판에서 "ESMFold2 는 버전 스큐로 공개 릴리스에서 실행 불가"라고 단정했으나
**그 판단이 틀렸음이 실행으로 드러났다** (§6.9). 이 보고서는 그 문장을 조용히 삭제하지 않고
**정정 절을 신설해 오판 사실·원인·정정 근거를 함께 남긴다.**
무-날조 규약은 "틀린 말을 하지 않는 것"만이 아니라 **"틀렸던 것을 틀렸다고 기록하는 것"**까지다.

---

## 9. 강의 활용법

### 9.1 강사 데모 트랙 (Claude Code 유료 구독 필요)

| 순서 | 할 일 | 소요 | 보여줄 것 |
|---|---|---|---|
| 1 | `cd pde5_harness && python run_harness.py check` | 10초 | 환경 PASS · ethanol MW=46.07 (실계산 증거) |
| 2 | `python scripts/target_lookup.py` | 3초 | 표준 봉투 3필드 · `query` 에 URL 원문 |
| 3 | `python scripts/mol_properties.py` (인자 없이) | 1초 | **`passed: false`** — 무-날조의 실물 |
| 4 | `claude` → *"CLAUDE.md 읽고 PDE5 저해제 후보 조사해 보고서까지 자율 실행해줘"* | 5–10분 | PLAN → 게이트 로그 → 보고서 |
| 5 | `cd ../antibody_harness && python run_harness.py run P04626` | 1–2분 | **같은 봉투·같은 게이트, 다른 도메인** |
| 6 | `python run_harness.py design-check` | 5초 | sha256 대조 + `CUDA 사용 불가` → 설계 미실행 |

> **팁**: 3번과 6번이 이 강의의 핵심 장면이다. "실패를 실패라고 말하는 출력"을
> 직접 보여주는 것이 슬라이드 열 장보다 강하다.
> 네트워크를 일부러 끊어 §8 사례 ②를 재현하면 더 좋다.

### 9.2 수강생 무료 대안 (0원)

| | `nb1_scientific_agent.ipynb` (무료) | `*_harness/` (강사 데모) |
|---|---|---|
| 실행 환경 | Colab 무료 런타임 | 로컬 Claude Code (유료 구독) |
| LLM | Gemini 무료 티어 또는 **무키 결정론 폴백** | Claude |
| 도구 | 5종 — `search_pubmed`·`lookup_uniprot`·`search_chembl_actives`·`mol_properties`·`admet_flags` | 스킬 6~9종 + 스크립트 7~11종 |
| 에이전트 루프 | **손으로 구현** (`call_tool`·`audited_call`·`with_retry`·`run_agent`) | Claude Code 가 담당 |
| 검증 | 감사 로그 + 재시도/폴백 | PostToolUse 훅 + `verify.py` 게이트 |
| 배우는 개념 | 동일 — 도구 등록 · 표준 반환 · 재시도 · 출처 기록 · 검증 | 동일 |

> 정직하게: 무키 폴백은 **진짜 LLM 추론이 아니라 스크립트 시연**이다.
> 다만 도구는 실제 공개 API 를 호출하므로 반환되는 PMID/SMILES/물성 값은 real 이다.

### 9.3 두 하네스를 붙여 쓰는 강의 서사

1. **PDE5** — "에이전트가 저분자 파이프라인을 정직하게 돌린다" (기존 28장 덱)
2. **항체** — "같은 뼈대가 바이오로직스에서도 그대로 돈다" (도메인 이식성)
3. **설계 2경로** — "2026 SOTA(ESMFold2 inversion) vs 고전(RFantibody), 그리고
   **저자 스스로 밝힌 한계**를 그대로 인용하는 태도" (비판적 읽기)
4. **검증 매트릭스** — "무엇을 확인했고 무엇을 확인하지 않았는가를 표로 남긴다" (연구 위생)
5. **GPU 실측과 정정** — "미검증이던 칸을 실제로 채웠고, 그 과정에서 **이전 결론이 틀렸음**을
   발견해 정정했다" (§6.6–6.9). 강의의 마지막 메시지는 여기다:
   빈칸을 지키는 것과, 채운 뒤 틀린 걸 고백하는 것이 **같은 규약의 앞뒷면**이다.

---

## 10. 재현 방법

### 10.1 PDE5 하네스

```bash
cd /home/hjpark/lecture_drug1/0905_agent/pde5_harness

# (권장) 검증에 쓴 인터프리터
PY=/home/hjpark/lecture_drug1/.venv_slides/bin/python   # Python 3.12.3 · RDKit 2026.03.4

$PY run_harness.py check      # → PASS, ethanol MW=46.07
$PY run_harness.py run        # → 4 게이트 PASSED
$PY run_harness.py list

# 개별 도구
$PY scripts/target_lookup.py
$PY scripts/chembl_actives.py 3
$PY scripts/mol_properties.py "CCCc1nn(C)c2c1nc([nH]c2=O)-c1cc(ccc1OCC)S(=O)(=O)N1CCN(C)CC1"
$PY scripts/selectivity.py
$PY scripts/docking.py         # → docking_executed:false (정직한 스킵)

# 무-날조 사례 재현
$PY scripts/mol_properties.py  # 인자 없음 → passed:false
```

Docker 도 준비돼 있다 (이번 검증에서는 빌드하지 않음):
```bash
docker build -t pde5-harness . && docker run --rm -it -v "$PWD/outputs:/workspace/outputs" pde5-harness python scripts/target_lookup.py
```

### 10.2 항체 하네스 (CPU 평가 파이프라인)

```bash
cd /home/hjpark/lecture_drug1/0905_agent/antibody_harness

# 권장: 전용 venv
python3 -m venv .venv && ./.venv/bin/pip install -r requirements.txt
PY=./.venv/bin/python        # 또는 검증에 쓴 /home/hjpark/lecture_drug1/.venv_slides/bin/python

$PY run_harness.py check         # → PASS (CDR 자기검증 ok 확인)
$PY run_harness.py design-check  # → PASS (sha256 일치, CUDA 사용 불가)
$PY run_harness.py run P04626    # → 5 게이트 PASSED, outputs/01~05_*.json 생성
$PY run_harness.py list

# 무-날조 사례 재현 (네트워크 차단)
HTTPS_PROXY=http://127.0.0.1:9 HTTP_PROXY=http://127.0.0.1:9 $PY scripts/antigen_lookup.py P04626
HTTPS_PROXY=http://127.0.0.1:9 HTTP_PROXY=http://127.0.0.1:9 $PY scripts/antibody_search.py P04626 6
```

정식 IMGT/Kabat 번호매김이 필요하면 `pip install anarci` 또는 `pip install abnumber` 후
`cdr_analysis.py` 를 재실행하면 백엔드가 바뀐다 (`numbering_backend()` 로 확인).

### 10.3 GPU 설계 단계 (2026-09-01 실행 완료 — 실제로 밟은 절차)

`RUNPOD_가이드.md` 의 표준 흐름: `deploy` → `wait` → `put` → `ssh`(가중치+설계) → `get` → **`rm`**.

**경로 B — RFantibody (A40, `$0.44/hr`)**

```bash
cd /workspace/RFantibody
OUT=scripts/examples/example_outputs/nb_ha_demo && mkdir -p $OUT
uv run rfdiffusion --target scripts/examples/example_inputs/flu_HA.pdb \
  --framework scripts/examples/example_inputs/h-NbBCII10.pdb \
  --output-quiver $OUT/1_rfdiffusion.qv --num-designs 4 \
  --design-loops H1:7,H2:6,H3:5-13 --hotspots B146,B170,B177 \
  --diffuser-t 50 --deterministic
uv run proteinmpnn --input-quiver $OUT/1_rfdiffusion.qv \
  --output-quiver $OUT/2_proteinmpnn.qv --seqs-per-struct 4 --temperature 0.2
uv run rf2 --input-quiver $OUT/2_proteinmpnn.qv \
  --output-quiver $OUT/3_rf2.qv --hotspot-show-prop 0.0 --num-recycles 10
```

**경로 A — ESMFold2 inversion (H100, `$3.29/hr`)** — 환경 구성이 절반이다:

```bash
# 1) venv 는 반드시 로컬 디스크에 (네트워크 FS /workspace 에 만들면 설치가 극도로 느림)
python3.12 -m venv /root/v312          # esm 은 Python >= 3.12 요구
/root/v312/bin/pip install esm modal   # modal 은 로컬 실행에도 import 됨
# 2) 실제 마지막 블로커였던 의존성 (Modal 이미지는 micromamba 로 깔지만 pip 경로엔 없음)
/root/v312/bin/pip install abnumber    # + hmmer, anarci
# 3) 공개 릴리스 호환 어댑터로 실행 (알고리즘 수정 없음, 인자 3개 pop + 속성명 1개)
/root/v312/bin/python esmfold2_public_release_adapter.py
```

> 파드 위에서는 이 어댑터가 `/workspace/run_design2.py` 로 저장돼 실행됐다
> (로그의 `ResourceWarning: /workspace/run_design2.py:43` 이 그 흔적).
> 저장소에 보관된 사본이 `antibody_harness/esmfold2_public_release_adapter.py` 다.

> ⚠️ 이 4가지가 왜 필요했는지, 그리고 이전 판의 "실행 불가" 결론이 왜 틀렸는지는
> **§6.9 재현성 교훈**에 근거와 함께 정리했다.

> ⚠️ **작업 후 `rm` 하지 않으면 pod 이 켜져 있는 동안 계속 과금된다.**
> `list` 로 살아 있는 pod 이 없는지 마지막에 반드시 확인.
> RunPod 제어는 이 하네스에 새로 구현하지 않고 기존 검증 도구
> `/home/hjpark/foundation_model_research/projects/_shared_infra/runpod_ctl.py` 를 재사용한다.
> 키(`RUNPOD_API_KEY`, `HF_TOKEN`)는 **이름만** 쓰고 값은 문서·로그·코드에 남기지 않는다.

---

## 11. 미검증 항목 총목록

### 11.1 ✅ 해소됨 — 2026-09-01 RunPod 실행으로 검증

이전 판의 미검증 16건 중 **9건이 실측값으로 대체**되었다.

| 이전 # | 항목 | 실측 결과 (근거 절) |
|---|---|---|
| 1 | 경로 A 실제 설계 실행 | H100 · `esm 3.4.0` 로 완주 (§6.7) |
| 2 | 경로 A 산출 설계 서열 · ipTM · 랭킹 | scFv 244 aa · critic 4종 ipTM 0.5218–0.8938 (§6.7) |
| 3 | 경로 A `model_load_seconds` · `elapsed_s` | 로드 100.1초(최초)/29.3초(캐시) · `DESIGN_SEC 191.3` (§6.7) |
| 4 | 경로 B RFdiffusion 실행 | 4 designs · 각 3.08–3.83분 · 합계 13.28분 (§6.6) |
| 5 | 경로 B ProteinMPNN 실행 | 16 서열 (4 구조 × 4) · 구조당 3–7초 (§6.6) |
| 6 | 경로 B RF2 실행 · pAE/RMSD 산출 | 16 예측 · `3_rf2.sc` 전체 통계 (§6.6) |
| 7 | 경로 B 저자 권장 필터 실적용 | `interaction_pae<10` → **1/16 통과 (6.3%)** (§6.6) |
| 9 | HF 가중치 다운로드 | 로그 `Fetching 2/11 files` + 캐시 재사용 확인 |
| 10·11 | 실행 시간 / RunPod 비용 | A40 `$0.44/hr` · H100 `$3.29/hr` · 총 **약 $5** · 잔여 pod 0 |
| 12 | VRAM (부분) | 경로 A `peak_allocated 25.08 GiB` / `reserved 25.59 GiB` |

### 11.2 🟡 여전히 미검증 — 값을 적지 않는다

| # | 항목 | 왜 아직 미검증인가 |
|---|---|---|
| 1 | `compare_designs.py` 실제 두-경로 비교 산출물 | 두 경로를 **서로 다른 타깃**(HA/나노바디 vs PD-L1/scFv)으로 돌렸다. 동일 타깃 대조 입력이 없으므로 비교 스크립트를 돌리면 근거 없는 대조가 된다 |
| 2 | 설계 서열에 대한 평가 3종(cdr·developability·humanness) 적용 | 설계 서열은 확보했으나 평가 파이프라인에 아직 투입하지 않음 |
| 3 | 경로 B `--self-consistency` (설계 백본 vs RF2 예측 Cα RMSD) | 이번 실행에서 미사용. `3_rf2.sc` 의 `framework_aligned_*_rmsd` 는 다른 정렬 기준 |
| 4 | Rosetta ddG < -20 필터 | Rosetta 미설치 — 저자 권장 필터 3개 중 이 항목만 미적용 |
| 5 | VRAM 27GB / 51GB (공식 주석 조건) | 우리 실측 25.08 GiB 는 `use_scaling_critics=False`·`batch_size=1`·PD-L1(115aa) 조건. 공식 주석의 `REUSE_ESMC` 조건과 다르므로 그 값 자체는 여전히 인용값 |
| 6 | 경로 A wet-lab hit rate (15–29% / 36–88%) | 📄 원저자 보고. **이번 실행에도 wet-lab 결합 데이터는 없다** |
| 7 | FoldBench DockQ AF3 상회 (50% vs 47%) | 📄 2차 소스 (`2026_landscape.md`) — 1차 확인 권장 |
| 8 | 설계물의 실제 결합 여부 | in silico 지표만 산출. **wet-lab 이 유일한 판정자** |
| 9 | PDE5 `docking.py` 실제 도킹 스코어 | 수용체/리간드 PDBQT · 박스 파라미터 부재 |
| 10 | PDE5 Docker 이미지 빌드·실행 | 이번 검증 범위 밖 |
| 11 | Claude Code 에이전트 자율 실행 전 과정 (PLAN→REPORT) | 본 검증은 스크립트 직접 실행으로 수행 |

**상태**: 설계 2경로의 **실행 자체는 검증 완료**. 남은 것은 (a) 동일 타깃 대조 실험,
(b) 설계 서열의 평가 파이프라인 통과, (c) wet-lab 검증 — (c)는 계산으로 해소되지 않는다.

---

## 참고 출처

- **PDE5**: UniProt `O76074`; ChEMBL `CHEMBL1827`;
  Boolell M et al. *Int J Impot Res.* 1996;8(2):47-52;
  Ghofrani HA et al. *Nat Rev Drug Discov.* 2006;5(8):689-702 (doi:10.1038/nrd2030); RDKit.
- **항체**: UniProt `P04626`; RCSB PDB `1N8Z`·`1S78`·`3BE1`·`3H3B`·`3N85`·`3WLW`; BioPython (ProtParam,
  PairwiseAligner/BLOSUM62); instability index 기준 Guruprasad 1990.
- **설계 경로 A**: `github.com/Biohub/esm` (MIT), `cookbook/tutorials/binder_design.py`,
  commit `827ec128e4cdaf80f7d6f95fb367a08980b34918`, biorxiv `10.64898/2026.06.03.729735`.
- **설계 경로 B**: `github.com/RosettaCommons/RFantibody` (MIT), biorxiv `2024.03.14.585103v1`.
- **2026 모델 지형**: `0905_agent/2026_landscape.md` (2차 소스 항목은 "확인 권장(2026.9)" 표기).
- **하네스 원본**: `0905_agent/pde5_harness/`, `0905_agent/antibody_harness/`
  (CLAUDE.md · README.md · RUNPOD_가이드.md · scripts/ · outputs/).
- **GPU 실행 원본 로그 (2026-08-31~09-01)**: `0905_agent/runpod_logs/` —
  `rfantibody/rfab.log` (2733줄) · `rfantibody/3_rf2.sc` (16행 TSV) ·
  `esmfold2_design_run.log` · `esmfold2_design_results.json` ·
  `esmfold2_design.log` (최초 실패 `TypeError` 기록).
  공개 릴리스 호환 어댑터: `antibody_harness/esmfold2_public_release_adapter.py`.
- **결합 구조 그림**: PDE5A+실데나필 `PDB 1UDT`, HER2+트라스투주맙 Fab `PDB 1N8Z` —
  PyMOL(open-source)로 실제 PDB 파일에서 렌더, 접촉 잔기는 **4.5 Å 기준 실계산**.
  파일: `0905_agent/figures/{pde5_1UDT_sildenafil, her2_1N8Z_trastuzumab, her2_1N8Z_interface}.png`.

---

*무-날조 원칙 준수: 이 문서의 실측값은 2026-08-31(UTC) CPU 실행 및
2026-08-31~09-01 RunPod GPU 실행(A40 · H100) 결과이며, 원본 로그는 `0905_agent/runpod_logs/` 에 있다.
실행하지 않은 항목은 §11.2 에 "여전히 미검증" 으로 명시했다. 추정값·대표값·전형값은 사용하지 않았다.
이전 판의 잘못된 결론("ESMFold2 실행 불가")은 삭제하지 않고 §6.9 에 정정으로 남겼다.*
