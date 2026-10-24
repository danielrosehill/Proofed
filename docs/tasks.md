# Proofed: tasks

Seeded 2026-10-24 from the [development plan](development-plan.md). Tick and date as done.
**(D)** = needs Daniel.

## Phase 1: research

- [x] 2026-10-24 ([research](../research/2026-10-24-phase-1-feasibility/report.md) §3.1: no newline; `^` sentinel works). Check whether `nvidia/parakeet-tdt-0.6b-v2`'s tokenizer has a newline or paragraph token
      (load the `.nemo`, inspect the SentencePiece vocab). Start here: it decides the paragraph task.
- [x] 2026-10-24 ([research](../research/2026-10-24-phase-1-feasibility/report.md) §3.2). Survey prior art: ASR models trained to output clean or formatted text directly (unified
      ASR + post-processing), with sources and dates, confirmed vs inferred.
- [x] 2026-10-24 ([research](../research/2026-10-24-phase-1-feasibility/report.md) §3.1: none documented). Survey ASR models that emit paragraph breaks natively.
- [x] 2026-10-24 ([research](../research/2026-10-24-phase-1-feasibility/report.md) §3.3: keep v2; Qwen3-ASR-0.6B as comparison arm). Shortlist candidate base models besides Parakeet (on-device size, English, PnC), with a
      reason to switch or not.
- [x] 2026-10-24 ([research](../research/2026-10-24-phase-1-feasibility/report.md) §3.4). Feasibility note: can an RNN-T/TDT decoder learn deletions (asides, self-corrections) from
      read-aloud data? Look for evidence.
- [x] 2026-10-24 Write `docs/research.md`.

## Phase 2: data audit

- [ ] Count takes and minutes per in-scope category (`disfluency`, `repetitions`, `false-starts`,
      `numerics`, `currency`, `self-corrections`, `paragraphs`, `asides`, `whispered`) in the
      kitchen-sink data; `split_subsets.py --metadata-only` in
      `audio-dictation/English-Hebrew-Codeswitching-Parakeet-Finetune` does this.
- [ ] Check that these categories' targets are the clean text (not the spoken text).
- [ ] Write `docs/data.md` with the gaps.
- [ ] **(D)** Set a minimum per category and record the missing minutes.
- [ ] Record the gaps in the recorder's **Proofed** training mode (train.hebrish.app, release 25):
      self-corrections (0 takes), asides, paragraphs, whispered and false starts first.
- [x] Publish `danielrosehill/Proofed-ASR-Cleanup-Training-Data` (public, ungated): Daniel's
      in-scope takes, no code-switching rows, source split kept. **Done 2026-10-24**, commit
      2c51919f (600 takes, ~75 min, 9 tasks; built from Read-Aloud-Voice-Sentences @ c544f891).
      Rebuild and re-upload as new takes land.
- [ ] Synthesis plan: TTS self-corrections (Disfl-QA patterns), asides, and numbers/currency
      (NeMo TN verbalising digit targets); voice per the answer to the question below.
- [ ] **(D)** Training-data licence policy: permissive only, or also NC/LDC data (research §5).
- [ ] **(D)** Synthetic voice: Kokoro voices or Daniel's voice clone.

## Phase 3: spec

- [ ] **(D)** Answer the plan's open questions 1–3 (English-only base, paragraphs route, tiers).
- [ ] Write `docs/spec.md`: tasks, targets, metrics, blend and replay ratio, baselines.

## Phase 4: prototype

- [ ] Build the core + whispered blend (ASR Dataset Blender or the Hebrish blend scripts).
- [ ] Baseline evals: stock Parakeet; stock + regex + `ASR-Paragraph-Breaks`.
- [ ] First fine-tune on Modal; per-category eval.
