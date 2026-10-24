# Proofed: tasks

Seeded 2026-10-24 from the [development plan](development-plan.md). Tick and date as done.
**(D)** = needs Daniel.

## Phase 1: research

- [ ] Check whether `nvidia/parakeet-tdt-0.6b-v2`'s tokenizer has a newline or paragraph token
      (load the `.nemo`, inspect the SentencePiece vocab). Start here: it decides the paragraph task.
- [ ] Survey prior art: ASR models trained to output clean or formatted text directly (unified
      ASR + post-processing), with sources and dates, confirmed vs inferred.
- [ ] Survey ASR models that emit paragraph breaks natively.
- [ ] Shortlist candidate base models besides Parakeet (on-device size, English, PnC), with a
      reason to switch or not.
- [ ] Feasibility note: can an RNN-T/TDT decoder learn deletions (asides, self-corrections) from
      read-aloud data? Look for evidence.
- [ ] Write `docs/research.md`.

## Phase 2: data audit

- [ ] Count takes and minutes per in-scope category (`disfluency`, `repetitions`, `false-starts`,
      `numerics`, `currency`, `self-corrections`, `paragraphs`, `asides`, `whispered`) in the
      kitchen-sink data; `split_subsets.py --metadata-only` in
      `audio-dictation/English-Hebrew-Codeswitching-Parakeet-Finetune` does this.
- [ ] Check that these categories' targets are the clean text (not the spoken text).
- [ ] Write `docs/data.md` with the gaps.
- [ ] **(D)** Set a minimum per category and record the missing minutes.

## Phase 3: spec

- [ ] **(D)** Answer the plan's open questions 1–3 (English-only base, paragraphs route, tiers).
- [ ] Write `docs/spec.md`: tasks, targets, metrics, blend and replay ratio, baselines.

## Phase 4: prototype

- [ ] Build the core + whispered blend (ASR Dataset Blender or the Hebrish blend scripts).
- [ ] Baseline evals: stock Parakeet; stock + regex + `ASR-Paragraph-Breaks`.
- [ ] First fine-tune on Modal; per-category eval.
