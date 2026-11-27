# 6일차 실습 — 대규모 언어모델과 신약개발: LLM 에이전트 (2026)

> **11/28 강의 6일차 실습 자료 허브.** 수강생 전원이 각자 계정으로 **Claude Science** 를 써서
> PDE5 저해제를 **동향 정리 → 논문 → 활성·decoy 데이터셋 → 도킹 → 점수 분포 판정** 까지 직접 수행한다.
> 무료 대안(Colab 노트북)과 Claude Code 하네스(강사 데모)도 함께 둔다.

[![No-Fabrication](https://img.shields.io/badge/policy-no--fabrication-red.svg)](#무-날조-원칙-반드시-읽기)
[![Free-tier](https://img.shields.io/badge/practice-free%20(0원)-brightgreen.svg)](#무료로-실습하는-법-0원)
[![RDKit](https://img.shields.io/badge/RDKit-cheminformatics-green.svg)](https://www.rdkit.org/)

---

## 실습 목표
1. **도구를 쓰는 에이전트를 직접 운전한다** — 커넥터로 공개 DB(FDA·ChEMBL·PubChem·PDB·문헌)를 조회하고,
   샌드박스에서 RDKit·smina 를 실행시킨다.
2. **비교 실험을 설계한다** — 알려진 활성과 물성을 맞춘 decoy 를 같은 조건에 도킹해 점수 분포를 가른다.
3. **판정하고 한계를 적는다** — 판정 기준을 분석 전에 정하고, 크기 교란을 통제하고,
   '이 결과로 말할 수 없는 것'을 세 가지 이상 쓴다.

---

## (a) 이론 5교시 — 강의 슬라이드

이론 강의는 `lecture_materials/` 의 아래 슬라이드로 진행한다 (2026 모델 지형 갱신본):

| 교시 | 파일 | 장수 |
|------|------|------|
| 1교시 LLM 원리 & 과학 LM | `lecture_materials/1128_1교시.pdf` / `.pptx` | 64 |
| 2교시 LLM 활용 & 환각·RAG | `lecture_materials/1128_2교시.pdf` / `.pptx` | 71 |
| 3교시 에이전트 원리·사례 | `lecture_materials/1128_3교시.pdf` / `.pptx` | 53 |
| 4교시 하네스·Agentic 보안 | `lecture_materials/1128_4교시.pdf` / `.pptx` | 45 |
| 5교시 자율 과학 · world model · lab in the loop | `lecture_materials/1128_5교시.pdf` / `.pptx` | 54 |
| 실습 Claude Science · PDE5 | `lecture_materials/1128_실습_클로드사이언스.pdf` / `.pptx` | 40 |

2026 모델 지형(프론티어 LLM·단백질/화학 LM·치료제 파운데이션·에이전트/자율과학)의 검증된 사실 단일 소스는
[`2026_landscape.md`](2026_landscape.md) 참고. 핵심 메시지: **"챗봇 → 워커(도구 실행 에이전트)"**,
그리고 **단일 LLM → 다중 에이전트 co-scientist** 로의 이동.

---

## (b) 메인 실습 — Claude Science (수강생 전원)

**교재**: [`claude_science_pde5_practice.md`](claude_science_pde5_practice.md) — 단계별 프롬프트와 '확인할 것'.
**도구 사실 근거**: [`claude_science_facts_202611.md`](claude_science_facts_202611.md) — 공식 문서 확인(2026-11-27).

| 단계 | 하는 일 | 쓰는 것 |
|------|---------|---------|
| ① 준비 | 설치·커넥터·키·검토 켜기, 프로젝트 규약 심기 | Settings |
| ② 동향 | 승인 PDE5 저해제·물성·표적·활성 데이터 규모 | FDA · PubChem · ChEMBL · UniProt |
| ③ 논문 | 선행 도킹·가상스크리닝 연구 10~15편 + DOI 검증 | Literature Graph(OpenAlex) · PubMed · literature review 스킬 |
| ④ 데이터셋 | 1UDT 구조 + DUD-E 활성 16 · decoy 45 + 물성 맞춤 확인 | PDB · DUD-E · RDKit |
| ⑤ 도킹 | smina(CPU) 도킹 → 점수 분포·ROC·교란 통제 → 판정 | smina · RDKit |

**사전 준비(수강생)**: Pro 이상 계정 + 앱 설치 + 로그인까지 미리. 첫 실행에 약 5 GB를 내려받으므로
수업 중 설치는 금물. 수업 직전 체크 4개 — ChEMBL·PubMed 디렉터리 커넥터 추가 / OpenAlex 무료 키 /
연락 이메일 / **Auto-review 켜기**(Pro는 기본 off).

> **강사 사전 검증 실행**(실습 덱의 수치는 전부 여기서 나온다): 활성 16 · decoy 45 를 1UDT 에
> smina(exhaustiveness 8, seed 42)로 도킹 — 16 CPU 에서 약 10분. AUC 0.704, 95% CI [0.554, 0.838],
> EF10% 1.91. **점 추정은 0.7 을 넘지만 신뢰구간 하한이 0.554 라 사전 판정선으로는 '약한 신호'다.**
> 원자료: `lecture_materials/data/cs_screen_results.json`, 통계·그림: `gen_1128_cs_figs.py`.

---

## (c) 무료 대안 · 보조 실습 노트북

계정이 없거나 더 해보고 싶은 수강생을 위한 Colab 노트북. 모두 **무료 런타임**에서 동작한다.

| 노트북 | 내용 | 모델/도구 | Colab |
|--------|------|-----------|-------|
| **`notebooks/nb1_scientific_agent.ipynb`** | 직접 만드는 과학 에이전트 — function calling + ReAct 루프 | PubMed/UniProt/ChEMBL/RDKit, Gemini 무료 티어 or 무키 폴백 | [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/20260905/notebooks/nb1_scientific_agent.ipynb) |
| `t042_molt5.ipynb` | 화학 언어모델 — MolT5 (SMILES↔텍스트) | MolT5 | [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/20260905/notebooks/t042_molt5.ipynb) |
| `t110_esm2_peptide_optimization_tutorial.ipynb` | 단백질 LM — ESM-2 펩타이드 최적화 | ESM-2 | [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/20260905/notebooks/t110_esm2_peptide_optimization_tutorial.ipynb) |
| `t050_simple-local-rag.ipynb` | RAG — 로컬 문서 검색 증강 생성 | 오픈 임베딩/LLM | [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/20260905/notebooks/t050_simple-local-rag.ipynb) |
| `T004_pde5_llm_coscientist.ipynb` | 다중 에이전트 co-scientist (PDE5/PAH) | LLM 에이전트 | [![Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/20260905/notebooks/T004_pde5_llm_coscientist.ipynb) |

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
