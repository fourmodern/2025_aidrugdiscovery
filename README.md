# 2025 AI 신약개발 워크숍

AI 기반 신약개발의 전 과정을 다루는 집중 워크숍 실습 자료입니다. Day 1 ~ Day 3로 구성됩니다.

**일시:** Day 1 - Day 2: 2025년 7월 21일 - 22일 / Day 3: ADMET 특성 예측

---

## Day 1 - 화학정보학 및 구조 기반 약물 설계

화합물 데이터 수집부터 분자 도킹까지, 전통적인 CADD 파이프라인을 단계별로 실습합니다.

| 번호 | 주제 | 설명 | Colab |
|------|------|------|-------|
| T001 | ChEMBL 데이터 조회 | ChEMBL 데이터베이스에서 EGFR 표적의 활성 화합물을 조회하고 전처리 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day1/T001_ChEMBL_Data.ipynb) |
| T002 | ADME 필터링 | 리핀스키 Rule of 5 등 ADME 기준으로 약물 유사 화합물 필터링 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day1/T002_ADME_Filtering.ipynb) |
| T003 | 유해 부분구조 필터링 | PAINS 등 원치 않는 화학적 부분구조를 가진 화합물 제거 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day1/T003_Unwanted_Substructures.ipynb) |
| T004 | 화합물 유사도 | 분자 지문(fingerprint) 기반 화합물 간 유사도 계산 및 시각화 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day1/T004_Compound_Similarity.ipynb) |
| T005 | 리간드 약물단 | 3D 약물단(pharmacophore) 모델링을 통한 리간드 기반 가상 스크리닝 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day1/T005_Ligand_Pharmacophore.ipynb) |
| T006 | 랜덤 포레스트 | 분자 지문 특성을 활용한 Random Forest 분류기 기반 활성 예측 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day1/T006_Random_Forest.ipynb) |
| T007 | 신경망 | Keras 기반 인공신경망으로 화합물 활성(pIC50) 예측 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day1/T007_Neural_Networks.ipynb) |
| T008 | PDB 데이터 | Protein Data Bank에서 단백질 3D 구조 데이터 수집 및 분석 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day1/T008_PDB_Data.ipynb) |
| T009 | 결합 부위 탐지 | 단백질 표면에서 리간드 결합 부위를 자동으로 탐지하는 방법 실습 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day1/T009_Binding_Site_Detection.ipynb) |
| T010 | 분자 도킹 | smina를 사용한 단백질-리간드 분자 도킹 시뮬레이션 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day1/T010_Molecular_Docking.ipynb) |
| T011 | 상호작용 분석 | PLIP를 활용한 단백질-리간드 비공유 상호작용(수소결합, 소수성 등) 분석 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day1/T011_Interaction_Analysis.ipynb) |

강의 슬라이드 PDF 4종이 `Day1/` 폴더에 포함되어 있습니다.

---

## Day 2 - 심화 AI 모델 및 생성 기반 약물 설계

암 유전체 분석, 그래프 신경망, LLM, 생성 모델, 단백질 구조 예측 등 최신 AI 기법을 다룹니다.

| 번호 | 주제 | 설명 | GPU | Colab |
|------|------|------|-----|-------|
| T001_mod | TCGA 데이터 분석 | TCGA 암 유전체 데이터로 차등 발현 유전자 분석, PCA, 생존 분석 수행 | | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day02/T001_mod.ipynb) |
| T002 | GNN 딥러닝 | PyTorch Geometric 기반 그래프 신경망으로 유전자-표적 관계 예측 | 권장 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day02/T002_deeplearning.ipynb) |
| t041 | MolGen | 사전 학습된 분자 생성 모델로 SMILES/SELFIES 기반 신규 분자 생성 | 권장 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day02/t041_molgen.ipynb) |
| t043 | VAE 분자 생성 | 변분 오토인코더(VAE)로 약물 분자의 잠재 공간 학습 및 신규 분자 생성 | 권장 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day02/t043_vae.ipynb) |
| t045 | DiffDock | 확산 모델 기반 AI 분자 도킹 (DiffDock) | 필수 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day02/t045_diffdock.ipynb) |
| t046 | RFdiffusion | RFdiffusion으로 목표 기능을 가진 단백질 구조를 de novo 설계 | 필수 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day02/t046_rfdiffusion.ipynb) |
| t047 | Chai-1 | Chai-1 모델로 단백질-리간드 복합체 3D 구조 예측 | 필수 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day02/t047_chai1.ipynb) |
| t048 | DDPM | Denoising Diffusion Probabilistic Model 원리 학습 및 이미지 생성 실습 | 권장 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day02/t048_DDPM.ipynb) |

강의 슬라이드 PDF 3종이 `Day02/` 폴더에 포함되어 있습니다.

> 언어모델 관련 실습(LLM Co-Scientist·MolT5·RAG·ESM-2·ESM3)은 **[Day 5](#day-5---언어모델-실습)** 로 이동했습니다.

---

## Day 3 - ADMET 특성 예측

약물의 흡수·분포·대사·배설·독성(ADMET)을 데이터로 예측합니다. hERG 심장독성을 사례로 baseline과 딥러닝을 같은 조건에서 비교합니다.

| 번호 | 주제 | 설명 | Colab |
|------|------|------|-------|
| T012 | ADMET / hERG 심장독성 | 데이터 정제 → ECFP+XGBoost baseline vs 그래프 신경망(D-MPNN)을 scaffold split으로 비교, SHAP 해석과 conformal 불확실성까지 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day3/T012_ADMET_hERG_Prediction.ipynb) |

강의 슬라이드 PDF 4종이 `Day3/` 폴더에 포함되어 있습니다.

---

## Day 4 - 생성모델 · 분자 생성 실습

생성모델(언어모델·VAE)로 새로운 분자(SMILES)를 **생성 → 평가(validity·uniqueness·novelty) → 필터(QED·SA)** 하는 파이프라인을 실습합니다. 최신 라이브러리(현행 Colab)에서 동작하도록 갱신했습니다.

| 번호 | 주제 | 설명 | Colab |
|------|------|------|-------|
| 분자생성 파이프라인 | 통합 실습 | char-RNN 언어모델로 SMILES 생성 → 유효성·다양성·신규성 평가 → QED·SA 필터·시각화 (CPU에서 실행 가능한 소규모 데모) | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day4/mol_generation_practical.ipynb) |
| t041 | MolGen | 사전학습 분자 생성 모델(MolGen-large, SELFIES)로 신규 분자 생성 + 선택적 파인튜닝 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day4/t041_molgen.ipynb) |
| t043 | VAE 분자 생성 | 그래프 VAE로 분자 잠재공간 학습 및 신규 분자 생성(QED 조건) | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day4/t043_vae.ipynb) |
| REINVENT4 | de novo 생성 + RL 최적화 | 산업계 표준 오픈소스 프레임워크(AstraZeneca). 사전학습 prior로 de novo 생성 → 평가·필터 → **QED 목적함수 강화학습(staged learning)**. GPU 필수 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day4/reinvent4_denovo_optimization.ipynb) |

> 생성 결과는 **후보 제안(가설)** 이며 실험(합성·assay) 검증 전까지 결론이 아닙니다. `mol_generation_practical`은 CPU로도 도는 개념 데모이고, `t041`(MolGen ~1.4GB 다운로드)·`t043`는 GPU 권장, `reinvent4`는 **GPU 필수**(설치 수 분 + 설치 직후 런타임 1회 재시작이 필요할 수 있음)입니다.

---

## Day 5 - 언어모델 실습

일반 LLM·에이전트부터 과학 언어모델(단백질 LM, 화학 LM)까지, 신약개발에서의 **언어모델 활용**을 실습합니다. 현행 Colab 환경에서 동작하도록 갱신했습니다.

| 번호 | 주제 | 설명 | GPU | Colab |
|------|------|------|-----|-------|
| T004 | LLM Co-Scientist | LangChain + LLM 기반 NSCLC 표적 탐색·검증 AI 에이전트 (API 키 필요) | | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day5/T004_nsclc_llm_coscientist.ipynb) |
| t042 | MolT5 (화학 LM) | T5 기반 분자↔텍스트 상호 변환 (분자 설명 생성, 텍스트→분자 생성) | | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day5/t042_molt5.ipynb) |
| t050 | RAG | PDF 문서 기반 로컬 RAG 파이프라인 (신약개발 질의응답) | 필수 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day5/t050_simple-local-rag.ipynb) |
| t110 | ESM-2 (단백질 LM) | ESM-2 단백질 언어모델로 펩타이드 결합 최적화 (masked LM 스코어링) | 권장 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day5/t110_esm2_peptide_optimization_tutorial.ipynb) |
| t111 | ESM3 (단백질 LM) | 생성형 멀티모달 단백질 모델로 서열 설계(inpainting) + 구조 예측 + 신뢰도(pTM/pLDDT) | 권장 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day5/t111_esm3_protein_design.ipynb) |

> `t110`(ESM-2)이 masked LM 스코어링 방식이라면, `t111`(ESM3)은 생성형 멀티모달 방식으로 같은 문제에 접근합니다. `T004`(LLM 에이전트)는 LLM API 키가 필요합니다.

### 심화 — PDE5 저해제 발굴 에이전트 하네스 (`dd-harness.zip`)

Colab이 아니라 **Claude Desktop(Code 탭)** 으로 돌리는 자율 에이전트 하네스입니다. 자연어 목표 → 계획 → 단계별 검증 게이트 → 투고 패키지까지, PDE5 저해제 후보를 조사·생성·평가합니다. **Docker 없이 맥·Windows 모두 매끄럽게 실행**됩니다 — Docker가 없으면 프로젝트 안에 전용 파이썬 환경(`.venv-harness`)을 자동 생성해 실행하고(시스템 오염 없음), Docker가 켜져 있으면 자동으로 컨테이너 격리 모드로 돕니다.

**사용법**

1. `dd-harness.zip`을 내려받아 **압축을 풉니다** → `pde5-harness/` 폴더 생성. (필요 요건: **Python 3.9+** 하나. 계산용 패키지는 자동 설치됨)
2. **Claude Desktop의 Claude Code로 그 폴더에 들어갑니다.**
3. 폴더에서 환경 점검 (OS 공통):
   ```
   python run_harness.py check
   ```
   → **`rdkit ok True`** 가 나오면 준비 완료. (최초 1회 환경 준비로 수 분, 이후 캐시)
4. Claude Desktop **Code 탭**에서:
   ```
   SETUP.md, run.md, CLAUDE.md 읽고 harness 자율 실행해줘
   ```

> - **Docker는 선택**입니다. 없으면 자동 로컬 모드, 있으면 자동 컨테이너 모드. Docker가 있어도 로컬로 강제하려면 `python run_harness.py check --no-docker`. (Docker 격리를 쓰려면 <https://www.docker.com/products/docker-desktop> 설치 후 실행)
> - `run_harness.py`가 OS(맥/Windows/Linux)와 실행 환경을 자동 감지하므로 자기 OS나 Docker 유무를 몰라도 됩니다.
> - 폴더 안의 **`SETUP.md`(설치·실행 가이드) / `run.md`(단계별 실행) / `CLAUDE.md`(에이전트 규약)** 에 상세가 있습니다.
> - 역할 분담: 계산(RDKit 생성·필터·채점, ChEMBL 수집)=격리 실행 환경, 문헌조사·판단·문서화(보고서/원고)=Claude 본체.
> - 결과는 **후보 제안(가설)** 이며 합성·assay 실험 검증 전까지 결론이 아닙니다.

---

## 단백질 표현법 (2026-08-24) — 3교시 + 실습

단백질 표현(one-hot·조성·물성 → ESM-2 임베딩 → 구조·생성모델 잠재공간)을 다루는 3교시 강의와 실습 노트북 3종(표현법 비교 / 서열→구조(Boltz-2)+localization / RFdiffusion+ProteinMPNN 설계). 상세·Colab 링크: **[`20260824/`](20260824/)**.

---

## 화합물 표현 & QSAR (2026-08-25) — 3교시 + 실습

화학정보학·분자구조표현·SMILES·기술자 → **화합물 표현 2교시 + QSAR 1교시**로 압축한 강의와 실습 노트북 3종(① 분자 특징 생성·분석 ② QSAR 모델링·결과분석+ChemBERTa 전이학습 ③ ADME/T 다중 엔드포인트). 상세·Colab 링크: **[`20260825/`](20260825/)**.

---

## 추가 자료

| 파일 | 설명 | Colab |
|------|------|-------|
| qwen_tcga_advanced_2024.ipynb | Qwen3를 TCGA 암 데이터에 선호도 최적화(DPO/ORPO)로 파인튜닝하는 고급 실습. GPU 필수. 학습 데이터는 Google Drive에 별도로 준비해야 합니다 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/qwen_tcga_advanced_2024.ipynb) |

---

## 실행 환경

모든 노트북은 Google Colab에서 바로 실행할 수 있습니다. 각 노트북 첫 셀에서 필요한 패키지를 자동으로 설치합니다.

노트북은 현재 Colab 환경(Python 3.12 / PyTorch 2.11 / TensorFlow 2.20)에서 동작하도록 갱신되어 있습니다. 주요 변경 사항:

- **3D 시각화는 py3Dmol을 사용합니다.** nglview는 ipywidgets 7 시대 라이브러리로 현재 Colab에서 렌더링되지 않습니다.
- **opencadd 의존성을 제거했습니다.** 유지보수가 중단되어 Python 3.12에서 설치 자체가 실패합니다. KLIFS는 REST API를 직접 호출하고, 구조 정렬은 MDAnalysis로 처리합니다.
- Colab에 이미 설치된 `torch`는 재설치하지 않습니다. 재설치하면 런타임의 CUDA 빌드와 어긋납니다.

GPU가 필요한 노트북은 위 표의 GPU 열을 참고하세요.

Colab GPU 설정: 메뉴 > 런타임 > 런타임 유형 변경 > T4 GPU 선택

**T004 (LLM Co-Scientist)** 실행 시 Google Gemini API 키가 필요합니다.
