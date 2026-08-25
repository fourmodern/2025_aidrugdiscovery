# 화합물 표현 방법 & QSAR — 3교시 + 실습

7/23 강의(화학정보학·분자구조표현·SMILES·Molecular Descriptor 4종)를 **화합물 표현 2시간 + QSAR 1시간 = 3교시**로 압축·재구성했습니다. 구조를 컴퓨터로 표현하는 법 → 문자열·기술자 표현 → 그 표현으로 활성/물성을 예측하는 QSAR까지의 하나의 스토리입니다.

## 학습 자료 (슬라이드)

| 교시 | 주제 | 파일 |
|------|------|------|
| 1교시 | **화학정보학 & 분자구조 표현** — 화학정보학 개요·신약 파이프라인, 분자 그래프·connection table·InChI·2D/3D·파일포맷 | `1교시_화학정보학과_분자구조표현.pdf` (53p) |
| 2교시 | **SMILES & 분자 기술자** — SMILES 문법·canonical·SMARTS, 분자 기술자(0D~3D)·지문(ECFP) | `2교시_SMILES와_분자기술자.pdf` (50p) |
| 3교시 | **QSAR** — 기술자→모델, 워크플로, 모델(MLR/PLS/RF/GBM/DL), **검증(CV·y-scrambling·외부검증)·적용범위·OECD 5원칙** | `3교시_QSAR.pdf` (52p) |

## 실습 노트북 (Colab)

| 노트북 | 내용 | Colab |
|--------|------|-------|
| QSAR & ADMET 실습 | SMILES→**RDKit 기술자/ECFP 지문**→**QSAR 회귀**(용해도 logS)+**ADMET 분류**(혈뇌장벽 BBBP)→**검증(CV·y-scramble·적용범위·ROC-AUC)**→시각화. 실제 ESOL(1128)·BBBP(2039) 데이터 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/20260825/notebooks/qsar_practical.ipynb) |

### 실습 실측 결과(재현 시 값은 소폭 변동)
- **QSAR(회귀·용해도)**: 테스트 R² 기술자+RF **0.87**·Ridge 0.77·ECFP+RF 0.72, 5-fold CV **0.89±0.02**, **y-scramble −0.21**(우연상관 아님), 적용범위 **내부 RMSE 0.71 < 외부 1.32**, 주요 기술자 **logP·MW·TPSA**
- **ADMET(분류·혈뇌장벽 BBBP)**: ECFP+RF **ROC-AUC 0.938**·정확도 0.917, 5-fold CV ROC-AUC **0.921±0.009**
- → 회귀든 분류든 **동일한 "기술자→모델" 파이프라인**

> ⚠️ 데이터·수치는 모두 실제 측정/계산값(무-날조). 교육용 데모이며 실제 QSAR는 scaffold split·외부검증·불확실성 정량이 필요하고, 예측은 실험 검증 전까지 결론이 아닙니다.

**참고문헌**: Delaney 2004(ESOL); Hansch & Fujita 1964, Free & Wilson 1964; Todeschini & Consonni(기술자); Rogers & Hahn 2010(ECFP); Tropsha 2010, Cherkasov 2014, OECD 2007(QSAR 검증/원칙); RDKit.
