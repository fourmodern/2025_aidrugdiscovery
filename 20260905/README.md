# 6일차 실습 — 대규모 언어모델과 신약개발: LLM 에이전트 (2026)

> 9/5 강의 6일차 **실습 자료 허브**. 자연어 목표 하나로 **타겟 조사 → 후보 검색 → ADMET 평가 → 보고서**
> 전체 사이클을 자율 수행하는 신약개발 LLM 에이전트를, (1) **직접 만드는 무료 노트북** 과
> (2) **Claude Code 하네스**(강사 데모) 두 방식으로 실습한다.

[![No-Fabrication](https://img.shields.io/badge/policy-no--fabrication-red.svg)](#무-날조-원칙-반드시-읽기)
[![Free-tier](https://img.shields.io/badge/practice-free%20(0원)-brightgreen.svg)](#무료로-실습하는-법-0원)
[![RDKit](https://img.shields.io/badge/RDKit-cheminformatics-green.svg)](https://www.rdkit.org/)

---

## 실습 목표 (PDF 커리큘럼 6일차)
1. **도구 등록(function calling):** PubMed · UniProt · ChEMBL · RDKit · 규칙기반 ADMET.
2. **자율 실행:** 자연어 프롬프트 → 타겟 조사 → 후보 검색 → ADMET → 보고서 전체 사이클.
3. **에이전트 고도화:** 도구 추가 · 실패 디버깅(재시도/폴백) · 안전(권한 최소화·키 격리·감사로그).

---

## (a) 이론 5교시 — 강의 슬라이드

이론 강의는 `lecture_materials/` 의 아래 슬라이드로 진행한다 (2026 모델 지형 갱신본):

| 교시 | 파일 |
|------|------|
| 1교시 | `lecture_materials/0912_1교시.pdf` / `.pptx` |
| 2교시 | `lecture_materials/0912_2교시.pdf` / `.pptx` |
| 3교시 | `lecture_materials/0912_3교시.pdf` / `.pptx` |
| 4교시 | `lecture_materials/0912_4교시.pdf` / `.pptx` |
| 5교시 | `lecture_materials/0912_5교시.pdf` / `.pptx` |

2026 모델 지형(프론티어 LLM·단백질/화학 LM·치료제 파운데이션·에이전트/자율과학)의 검증된 사실 단일 소스는
[`2026_landscape.md`](2026_landscape.md) 참고. 핵심 메시지: **"챗봇 → 워커(도구 실행 에이전트)"**,
그리고 **단일 LLM → 다중 에이전트 co-scientist** 로의 이동.

---

## (b) 실습 노트북

모든 노트북은 **Colab 무료 런타임**에서 동작한다. Colab 배지를 클릭해 바로 실행.

| 노트북 | 내용 | 모델/도구 | Colab |
|--------|------|-----------|-------|
| **`notebooks/nb1_scientific_agent.ipynb`** ⭐신규 | 직접 만드는 "Claude Code식" 과학 에이전트 — function calling + ReAct 루프 + 엔드투엔드 자율 사이클 | PubMed/UniProt/ChEMBL/RDKit(무료), Gemini 무료 티어 or 무키 폴백 | [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/20260905/notebooks/nb1_scientific_agent.ipynb) |
| `t042_molt5.ipynb` (재사용) | 화학 언어모델 — MolT5 (SMILES↔텍스트 번역) | MolT5 (오픈, HF) | [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/20260905/notebooks/t042_molt5.ipynb) |
| `t110_esm2_peptide_optimization_tutorial.ipynb` (재사용) | 단백질 언어모델 — ESM-2 펩타이드 최적화 | ESM-2 (오픈, HF) | [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/20260905/notebooks/t110_esm2_peptide_optimization_tutorial.ipynb) |
| `t050_simple-local-rag.ipynb` (재사용) | RAG — 로컬 문서 검색 증강 생성 | 오픈 임베딩/LLM | [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/20260905/notebooks/t050_simple-local-rag.ipynb) |
| `T004_pde5_llm_coscientist.ipynb` (재사용) | 다중 에이전트 co-scientist (PDE5/PAH 사례) | LLM 에이전트 | [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/20260905/notebooks/T004_pde5_llm_coscientist.ipynb) |

> 재사용 노트북 4종의 원본은 `../../2026_aidrugdiscovery/Day06_LLM_Agent/` 에 있다.
> Colab 배지는 강의 리포(`fourmodern/2025_aidrugdiscovery`, `20260905/notebooks/`)에 업로드된 것을 가정한 경로다.

### nb1 노트북 셀 구성 (핵심 신규)
1. 에이전트 개념(LLM+도구+계획+검증, ReAct/function calling) + 2026 맥락.
2. 설치(rdkit/requests) & 임포트.
3. **도구 5종 정의** — 전부 실 API/실계산: `search_pubmed`, `lookup_uniprot`, `search_chembl_actives`, `mol_properties`, `admet_flags`.
4. **함수 호출 스키마 등록**(name/description/parameters, 3-벤더 공통 형식).
5. **에이전트 루프** — 무료 우선: Gemini 무료 티어(실 LLM) → 유료(OpenAI/Anthropic) → 무키 결정론 폴백.
6. **엔드투엔드 데모** — 자연어 목표 → 자율 실행 → 마크다운 보고서 + RDKit 구조 그림.
7. **고도화** — 도구 추가(PubChem 예시) · 재시도/폴백 디버깅 · 안전 3원칙 + 감사 로그.
8. 한계·주의(환각·규칙기반 ADMET 한계·검증 필요).

---

## (c) PDE5 하네스 실습 (Claude Code)

`pde5_harness/` — nb1의 원리를 실무 코딩 에이전트 **하네스**로 구현한 버전.
`CLAUDE.md` 규약 + PostToolUse 검증 훅으로 "자연어 지시 하나 → PLAN → 단계별 검증 게이트 → 보고서"를 자율 실행한다.

- 상세 단계별 안내: **[`pde5_harness_실습가이드.md`](pde5_harness_실습가이드.md)**
- 빠른 시작 (로컬 .venv, Docker 불필요):
  ```bash
  cd pde5_harness
  python run_harness.py check     # rdkit ok / 스크립트 점검
  claude                          # Claude Code로 폴더 열기
  # 프롬프트: "SETUP·CLAUDE.md 읽고 harness 자율 실행해줘"
  ```
- ⚠️ **Claude Code는 유료 구독** 이 필요하다 → 이 파트는 **강사 데모**로 진행한다.
  구독이 없는 수강생의 **무료 대안은 위 `nb1_scientific_agent.ipynb`**(동일 개념, 무료 Colab).

---

## 하네스 없이 재현하기 — Claude for Science 용 프롬프트

로컬 파일(`CLAUDE.md`·스킬·스크립트) 없이 같은 연구를 수행하려면
[**`pde5_harness/prompts/claude_for_science.md`**](pde5_harness/prompts/claude_for_science.md)
하나만 열면 된다. 하네스가 파일로 강제하던 계약을 프롬프트 본문이 대신 진다.

- **A. 마스터 프롬프트** (약 2,000단어) — 그대로 붙여넣는다
- **B. 각 조항이 무엇을 막는가** — 21개 조항을 실제 실패 사건과 짝지은 표
- **C. 다른 표적으로 바꿀 때** — 치환할 자리 · 이 연구에 맞춰진 값 · 바꾸면 안 되는 다섯

> 이 연구는 같은 표적·같은 도구로 세 판본을 냈고 결론이 세 번 달랐다. 세 번 모두 자동
> 검증 게이트를 전부 통과했으며, 오류를 잡은 것은 매번 외부 비평이었다.

---

## 무료로 실습하는 법 (0원)

이 실습은 **완전 무료**로 완주할 수 있다.

1. **노트북 = 0원.** Colab 무료 런타임 + **오픈 모델**(MolT5 화학LM, ESM-2 단백질LM, ChemBERTa류)
   + **무료 공개 API**(PubMed/NCBI E-utilities, UniProt REST, ChEMBL REST) + **오픈소스 RDKit**.
   API 키 없이도 전부 돌아간다.
2. **에이전트 LLM = 무료 또는 무키.**
   - **무료 실-LLM:** Google AI Studio(https://aistudio.google.com/apikey)에서 **Gemini 무료 티어** 키 발급 →
     `GOOGLE_API_KEY` 로 넣으면 진짜 LLM이 도구를 선택한다. (무료 티어는 분당/일일 rate limit 있음 — 429 시 잠시 대기.)
   - **무키 폴백:** 키가 하나도 없으면 결정론적 ReAct 시연으로 자동 폴백해 그대로 완주된다.
     > 정직하게: **무키 폴백은 '진짜 LLM 추론'이 아니라 스크립트 시연**이다. 단, 도구는 실제 공개 API를
     > 호출하므로 반환되는 PMID/SMILES/물성 값은 **real** 이다. LLM 추론 자체를 무료로 보려면 Gemini 키를 쓴다.
   - OpenAI/Anthropic 키는 **선택(유료)** 이며 있으면 사용된다.
3. **PDE5 하네스 = Claude Code 유료 구독 필요** → 강사 데모. 무료 대안은 nb1 노트북.

키는 `getpass`/`os.environ` 로만 다루며 **코드·노트북·깃에 하드코딩하지 않는다**.

---

## 무-날조 원칙 (반드시 읽기)

- **모든 수치·식별자는 실데이터/실계산만.** PMID·UniProt accession·ChEMBL ID·SMILES·물성값은
  실제 API 응답 또는 RDKit 실계산에서만 가져온다. LLM이 수치·ID를 **지어내지 않는다**.
- **규칙기반 ADMET의 한계:** nb1의 `admet_flags` 와 하네스의 관련 게이트는 **Lipinski/Veber/lead-like
  규칙 필터**다. 실제 흡수·분포·대사·배설·독성 **ML "예측"이 아니다.** "예측"이라 과장하지 말 것 —
  실제 ADMET은 전용 ML 모델(ADMET-AI/TxGemma류)이나 실험이 필요하다.
- **결과 = 가설.** 이 실습은 알려진 화학공간 후보의 조사·선별·정리 **시연**이며 신약 발견이 아니다.
  합성·효소 어세이 실검증은 별도다.
- **환각 재검증:** LLM이 말한 수치는 항상 도구로 재확인한다. 데이터 부재 시 값을 만들지 않고 "확인 필요"로 표기.

---

## 디렉토리
```
0905_agent/
├── README.md                     # 이 파일
├── 2026_landscape.md             # 2026 모델 지형(검증된 사실 단일 소스)
├── notebooks/
│   └── nb1_scientific_agent.ipynb   # ⭐신규 핵심 노트북(실행·출력 임베드 완료)
├── pde5_harness/                 # Claude Code 하네스(강사 데모)
├── pde5_harness_실습가이드.md     # 하네스 단계별 실습 가이드
└── assets/                       # 강의 보조 그림/슬라이드
```

---

*무-날조 원칙 준수: 모든 값은 실제 API 응답 또는 RDKit 실계산이다. 규칙기반 ADMET은 예측이 아니다.*
