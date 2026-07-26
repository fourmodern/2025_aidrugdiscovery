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
| T004 | LLM Co-Scientist | LangChain + Google Gemini를 활용한 NSCLC 표적 탐색 AI 에이전트 | | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day02/T004_nsclc_llm_coscientist.ipynb) |
| t041 | MolGen | 사전 학습된 분자 생성 모델로 SMILES/SELFIES 기반 신규 분자 생성 | 권장 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day02/t041_molgen.ipynb) |
| t042 | MolT5 | T5 기반 분자-텍스트 상호 변환 (분자 설명 생성, 텍스트에서 분자 생성) | | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day02/t042_molt5.ipynb) |
| t043 | VAE 분자 생성 | 변분 오토인코더(VAE)로 약물 분자의 잠재 공간 학습 및 신규 분자 생성 | 권장 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day02/t043_vae.ipynb) |
| t045 | DiffDock | 확산 모델 기반 AI 분자 도킹 (DiffDock) | 필수 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day02/t045_diffdock.ipynb) |
| t046 | RFdiffusion | RFdiffusion으로 목표 기능을 가진 단백질 구조를 de novo 설계 | 필수 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day02/t046_rfdiffusion.ipynb) |
| t047 | Chai-1 | Chai-1 모델로 단백질-리간드 복합체 3D 구조 예측 | 필수 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day02/t047_chai1.ipynb) |
| t048 | DDPM | Denoising Diffusion Probabilistic Model 원리 학습 및 이미지 생성 실습 | 권장 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day02/t048_DDPM.ipynb) |
| t050 | RAG | PDF 문서 기반 로컬 RAG 파이프라인 구축 (신약개발 질의응답) | 필수 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day02/t050_simple-local-rag.ipynb) |
| t110 | ESM-2 | ESM-2 단백질 언어 모델로 펩타이드 결합 최적화 | 권장 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day02/t110_esm2_peptide_optimization_tutorial.ipynb) |
| t111 | **ESM3** | 생성형 멀티모달 단백질 모델로 서열 설계(inpainting) + 구조 예측 + 신뢰도(pTM/pLDDT) | | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day02/t111_esm3_protein_design.ipynb) |

강의 슬라이드 PDF 3종이 `Day02/` 폴더에 포함되어 있습니다.

`t111`은 워크숍 이후 추가된 실습입니다. `t110`(ESM-2)이 masked LM 스코어링 방식이라면, `t111`(ESM3)은 생성형 멀티모달 방식으로 같은 문제에 접근합니다.

---

## Day 3 - ADMET 특성 예측

약물의 흡수·분포·대사·배설·독성(ADMET)을 데이터로 예측합니다. hERG 심장독성을 사례로 baseline과 딥러닝을 같은 조건에서 비교합니다.

| 번호 | 주제 | 설명 | Colab |
|------|------|------|-------|
| T012 | ADMET / hERG 심장독성 | 데이터 정제 → ECFP+XGBoost baseline vs 그래프 신경망(D-MPNN)을 scaffold split으로 비교, SHAP 해석과 conformal 불확실성까지 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/Day3/T012_ADMET_hERG_Prediction.ipynb) |

강의 슬라이드 PDF 4종이 `Day3/` 폴더에 포함되어 있습니다.

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
