# 단백질 표현법 (Protein Representation) — 3교시 + 실습

단백질을 **어떤 벡터로 표현하느냐**가 downstream 성능을 좌우합니다. 구조·성질(토대) → 수작업 표현 → 학습된·구조 표현 → 생성모델 잠재공간까지, 실제 데이터로 실습합니다.

## 학습 자료 (슬라이드)

| 교시 | 주제 | 파일 |
|------|------|------|
| 1교시 | 단백질의 이해 — 아미노산·1~4차 구조·폴딩·종류·국재화 (모든 표현의 토대) | `1교시_단백질의이해.pdf` |
| 2교시 | 수작업 표현 — one-hot·조성(AAC)·물성(ProtParam)·PSSM·예측 툴 | `2교시_수작업표현.pdf` |
| 3교시 | 학습된 표현(PLM: ESM-2·ProtT5) · 구조 표현(contact/graph/AlphaFold) · 생성모델 잠재공간(RFdiffusion·ProteinMPNN) · 실습 | `3교시_학습된구조표현.pdf` |

## 실습 노트북 (Colab)

| 노트북 | 내용 | GPU | Colab |
|--------|------|-----|-------|
| 표현법 비교 | 실제 단백질 20개(4 family)로 one-hot·AAC·물성 vs **ESM-2** 임베딩을 family 구분으로 비교 (LOO 1-NN·실루엣) | 선택 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/20260824/notebooks/protein_representation_practical.ipynb) |
| 서열→구조+위치 | ESM-2 임베딩→**localization 예측**(4클래스) + **Boltz-2**(AF3급)로 **구조 예측**(ESMFold API 무GPU 폴백) | 구조=필수 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/20260824/notebooks/protein_structure_localization.ipynb) |
| de novo 설계 | **RFdiffusion**(백본 생성)→**ProteinMPNN**(서열 설계)→ESMFold 재접힘 self-consistency | 필수 | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/fourmodern/2025_aidrugdiscovery/blob/main/20260824/notebooks/protein_design_rfdiffusion.ipynb) |

### 방법 요약
- **localization**: ESM-2(단백질 언어모델) 임베딩 → 로지스틱 회귀. UniProt 실라벨 4클래스(Nucleus/Secreted/Mitochondrion/CellMembrane), 5-fold 교차검증.
- **구조**: Boltz-2(GPU, `--no_kernels`로 환경 독립 실행) 주 경로 + ESMFold API(무GPU) 폴백.
- **설계**: RFdiffusion + ProteinMPNN (ColabDesign 레시피). GPU 필수.

> ⚠️ 모든 결과는 **후보/예측(가설)** 이며, 수치는 실제 계산값(무-날조)입니다. 소규모 교육용 데모이므로 전용 SOTA(DeepLoc·AlphaFold3 등)와 다를 수 있고, 예측 구조·설계는 실험(합성·assay) 검증 전까지 결론이 아닙니다.
