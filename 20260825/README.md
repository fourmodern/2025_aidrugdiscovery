# 화합물 표현 방법 & QSAR — 3교시 + 실습

7/23 강의(화학정보학·분자구조표현·SMILES·Molecular Descriptor 4종)를 **화합물 표현 2시간 + QSAR 1시간 = 3교시**로 압축·재구성했습니다. 구조를 컴퓨터로 표현하는 법 → 문자열·기술자 표현 → 그 표현으로 활성/물성을 예측하는 QSAR까지의 하나의 스토리입니다.

## 학습 자료 (슬라이드)

| 교시 | 주제 | 파일 |
|------|------|------|
| 1교시 | **화학정보학 & 분자구조 표현** — 화학정보학 개요·신약 파이프라인, 분자 그래프·connection table·InChI·2D/3D·파일포맷 | `1교시_화학정보학과_분자구조표현.pdf` (53p) |
| 2교시 | **SMILES & 분자 기술자** — SMILES 문법·canonical·SMARTS, 분자 기술자(0D~3D)·지문(ECFP) | `2교시_SMILES와_분자기술자.pdf` (50p) |
| 3교시 | **QSAR** — 기술자→모델, 워크플로, 모델(MLR/PLS/RF/GBM/DL), **검증(CV·y-scrambling·외부검증)·적용범위·OECD 5원칙** | `3교시_QSAR.pdf` (52p) |
| 4교시 | **약물 최적화 & ADMET** — ~10^60 화학공간에서 **약효만이 아니라 다중 파라미터 최적화(MPO)**, ADMET(A/D/M/E/T)·리간드효율·CNS MPO·QED·Pareto·DMTA·attrition | `4교시_약물최적화_ADMET.pdf` (49p) |

## 실습 노트북 (Colab) — 주제별 3종 (특징 → 모델 → 응용 순서)

| # | 노트북 | 내용 | Colab |
|---|--------|------|-------|
| 01 | **분자 특징 생성 & 분석 (EDA)** | SMILES→기술자+ECFP 계산 → **특징 EDA**(분포·target 상관·상관 히트맵·화학공간 PCA/UMAP·ECFP 비트 통계) → **low-variance + 공선성 필터링**(2048→368). 모델링 전 특징을 만들고 이해 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/20260825/notebooks/01_feature_engineering_analysis.ipynb) |
| 02 | **QSAR 모델링 & 결과분석 + 전이학습** | 01 특징으로 회귀(Ridge/RF/GB) → 검증(CV·y-scramble·적용범위) → **8종 결과분석 시각화**(Williams/leverage plot 등) → **ChemBERTa(=MolCLR류) 전이학습**·표현 4종 비교 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/20260825/notebooks/02_qsar_modeling.ipynb) |
| 03 | **ADME/T 예측 (다중 엔드포인트)** | **BBBP**(혈뇌장벽·분류·ROC-AUC) + **Lipophilicity**(logD·회귀) — 같은 기술자→모델, 엔드포인트/지표만 다름 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/20260825/notebooks/03_admet_prediction.ipynb) |

### 실측 결과(재현 시 값은 소폭 변동)
- **01 특징**: ESOL 1128분자, logS 상관 Top3 **logP 0.83·MolMR 0.70·MW 0.64**, ECFP 필터 **2048→368**(82%↓, train에서만 학습해 누수 방지)
- **02 모델링/전이학습**: 기술자+GB R² **0.875**, 5-fold CV **0.896±0.015**, **y-scramble −0.21**(우연상관 아님), AD 적용범위 94%; 표현별 best R² 기술자 0.874·ECFP 0.72·필터ECFP 0.72·**ChemBERTa 0.785**
- **03 ADME/T**: BBBP **ROC-AUC 0.938**(CV 0.921±0.009), Lipophilicity **R² 0.673**(CV 0.666±0.009)

> ⚠️ 모든 수치는 실제 공개 데이터(ESOL·BBBP·Lipophilicity)로 노트북이 직접 계산한 값(무-날조). ChemBERTa는 자기지도 사전학습 분자 LM으로 전이학습을 시연하며, 그래프 기반 **MolCLR**(Wang 2022)은 동종 접근으로 노트북에 개념 소개.

> ⚠️ 데이터·수치는 모두 실제 측정/계산값(무-날조). 교육용 데모이며 실제 QSAR는 scaffold split·외부검증·불확실성 정량이 필요하고, 예측은 실험 검증 전까지 결론이 아닙니다.

**참고문헌**: Delaney 2004(ESOL); Hansch & Fujita 1964, Free & Wilson 1964; Todeschini & Consonni(기술자); Rogers & Hahn 2010(ECFP); Tropsha 2010, Cherkasov 2014, OECD 2007(QSAR 검증/원칙); RDKit.
