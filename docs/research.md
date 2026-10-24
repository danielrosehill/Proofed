# Proofed: research

The phase 1 exit test (`development-plan.md`). The full report, with every source and the
confirmed/inferred status of each claim, is in
[`research/2026-10-24-phase-1-feasibility/`](../research/2026-10-24-phase-1-feasibility/):

- [report.md](../research/2026-10-24-phase-1-feasibility/report.md) and its
  [PDF](../research/2026-10-24-phase-1-feasibility/Proofed-research-report.pdf), 17 pages,
  completed 2026-10-24 14:43 +03:00 (Claude Code, Opus 5.5).
- `notes/01`–`06`: one note per sub-question (paragraph token, prior art, base models,
  feasibility, datasets, Parakeet-family newline survey), each with Answer, Evidence, Gaps and Confidence sections.
- `sources.md`: 125 entries, S0101–S0542.

## Phase 1 (2026-10-24) in brief

| Question | Answer | Report |
|---|---|---|
| Newline or paragraph token in Parakeet v2? | No (1,024-piece BPE; `nmt_nfkc` turns `\n` into a space). `^` (id 918) works as a sentinel. | §3.1 |
| ASR models that emit paragraphs natively? | None documented; AssemblyAI and Deepgram add paragraphs after recognition, on the API side. | §3.1 |
| Other Parakeet/Canary models with a newline? (note 06, added later on 2026-10-24) | None of 22 (tokenizers tested). Add a `"\n"` token to v2 at id 1024, as NVIDIA did for `<EOU>` in parakeet_realtime_eou_120m-v1; `^` as control; v3 reserved token as fallback. | notes/06 |
| Prior art | Disfluency removal end-to-end on Switchboard (2020: worse than pipeline; 2023 with pretraining: better); FluentWhisper (2026, Whisper LoRA, synthetic). No Parakeet or transducer version; nothing on dictation self-corrections, same-speaker asides or paragraphs. | §3.2 |
| Base model | Keep parakeet-tdt-0.6b-v2 (Open ASR Leaderboard 6.02). Comparison arm: Qwen3-ASR-0.6B (6.24, LLM decoder, newlines, Android via sherpa-onnx). | §3.3 |
| Feasible? | Yes for the core tier; deletion learned by CTC, attention and RNN-T models; ~5–50 h with replay. Acted-to-natural transfer untested. Forgetting: replay or joint adapters. | §3.4 |
| Training data | ~42 min own in-scope takes (0 self-corrections). Redistributable: DisfluencySpeech, AMI, ICSI, LibriSpeech-PC and other replay sets. Excluded: SPGISpeech. Synthesis for self-corrections, asides, numbers. | §3.5 |
