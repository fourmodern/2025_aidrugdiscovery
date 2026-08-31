# 2026 모델 지형 레퍼런스 (교안 갱신 기준, 무-날조)

> 9/5 강의(2026)용. 2024–25 낡은 모델을 현재로 갱신하기 위한 **검증된 사실 단일 소스**.
> 규칙: 실제 존재하는 모델/버전만. 버전이 빠르게 바뀌는 항목·2차 블로그 근거는 **"확인 권장(2026.9)"** 표기.
> 개념·원리(RNN→Transformer→GPT/BERT, self-attention, 사전학습/정렬)는 시대 무관 → 선생님 교안 그림 그대로 사용.

## 1. 범용(프론티어) LLM — 2026
- **OpenAI GPT-5 계열** — GPT-5.x. 코딩 특화 GPT-5.3-Codex(2026.2). *세부 버전 확인 권장*
- **Anthropic Claude 5 계열** — Claude Opus 5 / Sonnet 5, Haiku 4.5, Opus 4.8. `Claude for Life Sciences`(과학 특화 제품).
- **Google Gemini 3** — Gemini 3 Pro 등. `Gemini for Science`.
- **오픈 가중치** — Meta Llama 4, DeepSeek(V3/R1 계열, reasoning), Qwen3, Mistral.
- **2026 흐름**: (a) reasoning(추론형) 기본 탑재, (b) **agentic — 모델이 '챗봇'에서 '워커(도구 실행)'로**. → 3교시 에이전트로 연결.
- 표기: "버전은 빠르게 갱신 — 2026.9 시점 확인 권장".
- 근거: Wikipedia(GPT-5.3-Codex); Anthropic/Google 발표; codingscape 2026 정리(블로그, 확인 권장).

## 2. 단백질 언어모델 / 구조 — 2026
- **ESM3** (EvolutionaryScale, 2024) — 멀티모달 생성형 단백질 'GPT'(서열·구조·기능 통합 track), open 1.4B(esm3-open). 신규 esmGFP 설계.
- **ESM Cambrian / ESMC** (2024–25) — **표현(representation) 특화** LM. 약 2.8B 서열 학습, 최대 6B 파라미터. ESM3와 병렬 계열(생성=ESM3 / 표현=ESMC).
- **ESMFold2** — FoldBench에서 AF3 대등/상회 주장(antibody–antigen DockQ 50% vs AF3 47%, single-seq). *확인 권장(2차 소스)*
- **AlphaFold3**(2024, DeepMind/Isomorphic), **Boltz-2**(2025, MIT — 3일차 실습 도구), **Chai-1**(2024).
- 메시지: 단백질 LM = '이해(ESMC 임베딩)' + '생성(ESM3)'. MSA-free·초고속이 대규모 스캔의 열쇠.
- 근거: evolutionaryscale.ai/blog/esm-cambrian; Hayes 2024(ESM3); neurohive/arxiv(ESMFold2, 확인 권장).

## 3. 화학 언어모델 / 분자 파운데이션 — 2026
- **표현(이해)**: ChemBERTa(-2), MolT5, Chemformer, **SMI-TED**(IBM) — SMILES 자기지도.
- **화학 파운데이션 LLM**: **ChemDFM**(chemistry foundation LLM, ScienceDirect 2025), **ChemLLM**.
- **멀티모달**: **ChemMLLM**(text+SMILES+분자 이미지 통합 이해·생성, Cell Rep Phys Sci 2026) — 이미지 직접 입력·생성 분자 시각화 end-to-end.
- **대화형 최적화**: DrugAssist(2025, Llama-2-7B).
- 메시지: 'SMILES 문자열 LM' → '멀티모달(구조 이미지·스펙트럼) 통합'으로 이동. 다수 최신은 preprint/2026 — 벤치·재현 확인 권장.
- 근거: cell.com(ChemMLLM 2026, S2666-3864(26)00216-X); ScienceDirect(ChemDFM); arxiv 2505.16326.

## 4. 치료제·오믹스 파운데이션 — 2026
- **TxGemma**(Google, 2025.4) — Gemma-2 기반 2B/9B/27B, TDC 66개 태스크. **Tx-LLM 대비 45/66 향상**. TxGemma-Predict/Chat. → 구 Tx-LLM 대체.
- **TxAgent / Agentic-Tx**(Google) — Gemini 2.0 기반 generalist 치료 에이전트(추론·도구·외부지식).
- **단일세포 파운데이션**: **scGPT**(3300만+ 세포, generative), **Geneformer**(rank encoding·masked gene), **scFoundation**(human cell atlas, asymmetric transformer), **CellFM**, UCE, SCimilarity.
  - ⚠ 정직한 한계: 섭동(perturbation) 예측 벤치에서 **"단순 PCA/baseline이 여전히 우세"**한 경우 다수(파운데이션 우위 과장 금지). → 5교시 멀티오믹스에서 비판적으로 제시.
- 근거: arxiv 2504.06196(TxGemma); huggingface google/txgemma; Nature s12276-025-01547-5(single-cell FM 리뷰); arxiv 2410.13956(perturbation 벤치).

## 5. 신약개발 LLM 에이전트 / 자율과학 — 2026
- **Google AI co-scientist**(Gemini 2.0, 2025.2, arxiv 2502.18864) — 다중 에이전트 가설 생성.
- **TxAgent / Agentic-Tx**(Google) — 치료 태스크 자율 실행.
- **다중 에이전트 신약**: **PharmAgents**(2025.3), **MADD**(multi-agent drug discovery, EMNLP 2025), **DiscoVerse**(traceable pharma co-scientist, arxiv 2511.18259, 2025.11), **RAG-enhanced collaborative LLM agents**(arxiv 2502.17506).
- **산업 배치**: ChatInvent @ AstraZeneca(2026.1 보고) — *확인 권장*.
- **평가/서베이**: "Agentic Science" 서베이(arxiv 2508.14111); "Beyond SMILES: Evaluating Agentic Systems for DD"(arxiv 2602.10163); text-guided molecular discovery 서베이(2505.16094); Frontiers LLM in DD & precision medicine(2026).
- 메시지: 2025–26 = **단일 LLM → 다중 에이전트 co-scientist**로 이동. 추적성(traceability)·검증·안전이 핵심 과제.

## 갱신 원칙 (덱 반영 시)
1. **원리 슬라이드(선생님 그림)**: 갱신 불필요 — 그대로 사용.
2. **지형/최신세대/벤치/사례 슬라이드**: 위 2026 사실로 교체. 옛 이름(GPT-4, Claude 3.5, Gemini 1.5, Tx-LLM 단독)은 "→ 현행 계열"로.
3. 모든 최신·2차소스 수치엔 "확인 권장(2026.9)" 병기. 날조·가짜 버전번호 금지.
4. 슬라이드 하단 ref에 위 근거 축약 인용.
