# RAG 실습 코퍼스 (t050)

`Day06_LLM_Agent/notebooks/t050_simple-local-rag.ipynb` 이 쓰는 PDE5 논문 8편이다.

## 왜 여기에 두나

원래 노트북은 EuropePMC 의 `europepmc.org/articles/<PMCID>?pdf=render` 에서
직접 내려받았다. 2026-11 현재 그 경로는 Cloudflare managed challenge 로 막혀
PDF 대신 챌린지 HTML(403)을 돌려준다. 그래서 같은 파일을 여기에 고정해 둔다.

강의 저장소(`fourmodern/2026_aidrugdiscovery`)는 수강생이 Claude Science 로
가져오는 저장소라 100 MB 제한을 지켜야 해서, 18 MB 인 이 코퍼스는 미러인
이 저장소에 둔다.

## 라이선스

전 편 **CC BY** (Creative Commons Attribution) 이며 EuropePMC 의 오픈액세스
서브셋이다. CC BY 는 출처를 표기하면 재배포를 허용한다. 아래가 그 출처 표기다.
원문과 최신 판본은 각 DOI 를 따라가면 된다.

| PMCID | 제목 | 저널 | 연도 | DOI | 라이선스 |
|---|---|---|---|---|---|
| [PMC12454544](https://europepmc.org/article/PMC/PMC12454544) | An overview on pharmaceutical applications of phosphodiesterase enzyme 5 (PDE5) inhibitors | Molecular diversity | 2025 | [10.1007/s11030-024-11016-2](https://doi.org/10.1007/s11030-024-11016-2) | CC BY |
| [PMC12673857](https://europepmc.org/article/PMC/PMC12673857) | The impact of PDE5 inhibitors on cardiovascular outcomes | Global cardiology science & practice | 2025 | [10.21542/gcsp.2025.31](https://doi.org/10.21542/gcsp.2025.31) | CC BY |
| [PMC11763789](https://europepmc.org/article/PMC/PMC11763789) | Phosphodiesterase Type 5 Inhibitors in Male Reproduction: Molecular Mechanisms and Clini… | Cells | 2025 | [10.3390/cells14020120](https://doi.org/10.3390/cells14020120) | CC BY |
| [PMC10201782](https://europepmc.org/article/PMC/PMC10201782) | Phosphodiesterase 5 (PDE-5) inhibitors (sildenafil, tadalafil, and vardenafil) effects o… | BMC gastroenterology | 2023 | [10.1186/s12876-023-02787-3](https://doi.org/10.1186/s12876-023-02787-3) | CC BY |
| [PMC11346399](https://europepmc.org/article/PMC/PMC11346399) | Polypharmacological Potential of Phosphodiesterase 5 Inhibitors for the Treatment of Neu… | Aging and disease | 2024 | [10.14336/ad.2023.1129](https://doi.org/10.14336/ad.2023.1129) | CC BY |
| [PMC12841899](https://europepmc.org/article/PMC/PMC12841899) | Repurposing PDE5-Inhibitors: Sildenafil Drives Arteriogenesis via Localized Regenerative… | International journal of molecular sciences | 2026 | [10.3390/ijms27020985](https://doi.org/10.3390/ijms27020985) | CC BY |
| [PMC13024863](https://europepmc.org/article/PMC/PMC13024863) | Treatment with Sildenafil Promotes Angiogenesis and Modulates Immune Response in Ischemi… | Current issues in molecular biology | 2026 | [10.3390/cimb48030283](https://doi.org/10.3390/cimb48030283) | CC BY |
| [PMC13111112](https://europepmc.org/article/PMC/PMC13111112) | Phosphodiesterase-5 inhibitors for cerebral small vessel disease-related ischemic stroke… | Frontiers in neurology | 2026 | [10.3389/fneur.2026.1776589](https://doi.org/10.3389/fneur.2026.1776589) | CC BY |

합계 8편 · 18.9 MB · 서지 원본은 `corpus.json`.
