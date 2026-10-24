# Proofed: recording targets

Set 2026-10-24 for the **first Proofed fine-tune** (phase 4 prototype). Recorded in the recorder app's
**Proofed** training mode (train.hebrish.app, release 25: Settings → Speech-to-text training mode →
Proofed). These are approximate targets, open to revision after the first per-category eval.

**How they were set:**
- **Evidence:** about 5–50 h of disfluent audio, with replay, moved an industrial RNN-T 9–23%
  relative ([research](research.md) §3.4). The Hebrish code-switching fine-tune moved code-switching
  WER from 9.5% to 4.5% on 77 min of single-speaker takes.
- **Own recordings:** each subtask aims for **10–60 min of your own audio**, enough to train a
  signal and to hold out ~10% for a per-category test. Synthetic TTS audio is planned on top
  (numbers, self-corrections, asides).
- **Prompt banks:** where a bank is smaller than the target, the target is the whole bank (marked
  "all"). The bank gets expanded after the first eval if the subtask shows promise.
- **Durations:** the average take lengths come from the takes so far (2026-10-24, from
  `Read-Aloud-Voice-Sentences` @ c544f891). The paragraphs figure is an estimate: the passages
  average 354 characters, about 60–75 s each.

| Subtask | Tier | Have (takes) | Target (takes) | Avg take | ≈ minutes at target | Bank size | Priority |
|---|---|---|---|---|---|---|---|
| ✏️ self-corrections | ambitious | 0 | 100 (all) | ~7 s | ~12 | 100 | **1** |
| 💬 asides | experimental | 2 | 100 (all) | ~10 s | ~16 | 100 | **1** |
| ↩️ false starts | core | 3 | 159 (all) | ~6 s | ~16 | 159 | **1** |
| 🤫 whispered | robustness | 4 | 140 (all) | ~7 s | ~16 | 140 | **1** |
| 📄 paragraphs | ambitious | 3 | 49 (all) | ~65 s (est.) | ~55 | 49 | 2 |
| 🔢 numerics | core | 41 (23 from the task bank) | 167 (all) | ~4.5 s | ~13 | 167 | 2 |
| 🔁 repetitions | core | 19 | 200 | ~12 s | ~40 | ~1,300 bank sentences | 2 |
| 🤔 disfluency | core | 55 | 250 | ~14.5 s | ~60 | ~1,300 bank sentences | 3 |
| 💰 currency | core | 116 | 140 (all) | ~6 s | ~14 | 140 | 3 (nearly done) |
| **Total** | | **243** | **~1,305** | | **≈ 4 h** | | |

**Priority** 1 means the subtask is empty or close to it. The Proofed mode already gives each
subtask an equal share of cards, so the priority-1 subtasks fill first without any setting. If you
want to focus, pick a single subtask in the Task sheet.

**After this round:**
- Expand the self-corrections, asides and false-starts banks to ~300 prompts each.
- Record the natural-dictation eval set (`Parakeet-Dictation-Cleanup-Eval`, 40 spontaneous
  dictations).
- Add synthetic audio for numbers, currency and self-corrections.

Replay (plain English) needs no new recording: 363 takes (~36 min) exist, and LibriSpeech-PC tops
them up.
