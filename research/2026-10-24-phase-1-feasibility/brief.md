# Brief: Proofed phase 1 research

Started 2026-10-24T14:18:08+03:00. Skill `modular-research:research-report`.

## Request (verbatim)

> yes please

…in reply to the offer to run Proofed's phase 1 research (`docs/tasks.md`, Phase 1) through the
skill: Parakeet's paragraph token, prior art, candidate base models, feasibility.

## Central question

Can a small on-device ASR model (Parakeet TDT 0.6B v2 first) be fine-tuned to output clean,
proofed text directly (disfluencies, repeats, false starts, self-corrections and asides removed,
numbers in digits, paragraph breaks, whispered speech handled), and what is the best base model
and route for each task?

## Sub-questions

1. **Paragraphs and the tokenizer:** does `nvidia/parakeet-tdt-0.6b-v2`'s tokenizer have a
   newline or paragraph token? If not, what are the routes (special token + vocabulary change
   in NeMo, a sentinel string, a different base), and which ASR models emit paragraph breaks
   natively?
2. **Prior art:** end-to-end ASR trained to output clean or written-form text (disfluency
   removal, inverse text normalisation, self-correction, removing non-directed speech) instead
   of a separate post-processing step. Papers, models, products.
3. **Candidate base models:** small English ASR models besides Parakeet v2 suited to on-device
   dictation and fine-tuning (size, PnC, licence, NeMo/HF fine-tuning support, streaming).
4. **Feasibility:** evidence that transducer/CTC/AED decoders learn deletions and reformatting
   from training targets; how much data; acted (read-aloud) vs natural disfluencies; whispered
   speech ASR.

## Scope

In: English ASR, public sources, current to 2026-10. Out: domain vocabulary (code-switching,
tech terms), LLM text-only post-processors except as baselines.

## Already established in the repo (not re-researched)

- Plan and tiers: `docs/development-plan.md`. Text-side baselines: `ASR-Paragraph-Breaks`
  (Gemma 3 270M, break F1 0.78), `ASR-Post-Processor-Model` (S1-mini LoRA, planning).
- Daniel's existing HF datasets `danielrosehill/Parakeet-Dictation-Cleanup-Pairs` and
  `-Eval` (in the local HF cache), and the task categories in
  `audio-dictation/Hebrish-App/data/training-categories.json`.

## Assumptions

- "Base model" means a model small enough for a phone (≤ ~1B parameters); larger ones are
  noted only as references.
