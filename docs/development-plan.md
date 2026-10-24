# Proofed: development plan

Status 2026-10-24: planning. No data split, no training run, no model yet.
Source: [voice note 2026-10-24](../context/voice-notes/2026-10-24-proofed-brief/transcript-clean.md)
(cited as `vn mm:ss`). Where this plan and the note disagree, the note wins; record the decision
here.

**Decided** = said by Daniel in the note. **Inferred** = this plan's reading, to confirm.

## What it is

An ASR fine-tune that writes clean text directly: a stock ASR base model (Parakeet first) trained
only on post-processing tasks, so that the transcript comes out without disfluencies, with numbers
as digits, without asides to other people, and (if the base model allows) with paragraphs. One
forward pass, no second LLM step. (Decided, vn 12:34, 19:27.)

It is a research prototype: the question is *to what extent a stock ASR model can be fine-tuned to
bake in a series of post-processing transformations* (vn 12:34). Published on Hugging Face (vn
20:25).

## Why

- Audio-native LLMs (Gemini with a system prompt) can do any transformation, but they are too big
  for phones and unusable for dictation even on a 12 GB desktop GPU (vn 04:55).
- Dedicated text post-processors (S1-mini by Superwhisper) still need a second model (vn 05:53).
  Daniel's own text-side attempts: `danielrosehill/ASR-Post-Processor-Model` (planning, S1-mini
  LoRA) and `danielrosehill/ASR-Paragraph-Breaks` (Gemma 3 270M, break F1 0.78, released
  2026-10-21).
- Regex post-inference cleanup works for some of this; the aim is to fix it at source (vn 16:03).
- The Hebrish kitchen-sink run showed that combining training domains did not degrade either
  (vn 10:58). Inferred: that is some evidence task subsets can be stacked.

## Scope

**In (decided):** post-processing transformations that are right for every user, every time
(vn 15:17). Narrow, modular tasks, each measurable on its own, because some may not train at small
scale (vn 17:54).

**Out (decided):** all domain vocabulary, including code-switching, tech vocab, names (vn 23:54).
Anything use-specific, such as formatting output as an email (vn 15:17).

**Tiers.** Daniel names core, ambitious and experimental tiers (vn 20:25) but assigns only some
tasks. Proposed mapping onto the existing task categories
(`audio-dictation/Hebrish-App/data/training-categories.json`, 2026-10-21):

| Tier | Category id | Task | Basis |
|---|---|---|---|
| core | `disfluency` | drop uh / um / eh | decided, vn 09:55, 15:17 |
| core | `repetitions` | drop restart repeats ("orange, orange juice") | decided, vn 17:54 |
| core | `false-starts` | drop abandoned starts and unfinished sentences | decided, vn 17:54 |
| core | `numerics` | numbers, dates, units in digits | decided, vn 09:55 |
| core | `currency` | spoken amounts as typed (50 NIS) | inferred: general formatting |
| ambitious | `self-corrections` | "Haifa, I mean Tiberias" → Tiberias; "scratch that" | inferred tier; task decided, vn 18:32 |
| ambitious | `paragraphs` | paragraph breaks | decided task, vn 16:19; tier inferred, blocked by tokenizer (below) |
| experimental | `asides` | drop speech aimed at someone else | decided, vn 17:04 |
| robustness | `whispered` | whispered speech | decided, must be included, vn 21:03 |
| open | `web-addresses`, `spelled-out`, `acronyms`, `expanded-acronyms` | written-form conventions | unclear whether "post-processing" or "domain"; ask |
| out | `singing`, `weird-voices`, `name-cipher` | other experiments | inferred out: not mentioned |

Plus plain-English replay (`general`, LibriSpeech-PC) so the base does not forget; ratio per
`ai-models/ASR-Dataset-Blending` (start ~10%).

## Approach

1. **Base model:** `nvidia/parakeet-tdt-0.6b-v2` (English, punctuated and cased). Daniel prefers
   to stay on Parakeet (vn 22:47). Survey alternatives anyway (task in `tasks.md`).
2. **Data:** from the kitchen-sink recordings (`danielrosehill/Read-Aloud-Voice-Sentences` and
   the Hebrish blends, e.g. `Hebrish/Parakeet-V2-Blend-All`), keep only the post-processing
   categories above, stratified by category. Same term-disjoint train/test split as the Hebrish
   repos. Measure minutes per category first: Daniel expects some need more recording
   (vn 22:00).
3. **Training:** NeMo fine-tune, as in `audio-dictation/English-Hebrew-Codeswitching-Parakeet-Finetune`
   (Modal GPU).
4. **Evaluation:** per category, against the clean target: task success (filler gone, digits
   right, aside removed, correction applied) plus WER on the rest of the sentence, and WER on
   `general` and LibriSpeech-PC for regression. Compare with stock Parakeet, and stock Parakeet +
   regex + `ASR-Paragraph-Breaks` as the two-model baseline.
5. **Layering (later):** fine-tune Proofed again with a domain-vocab layer (tech, code-switching)
   and compare with the one-shot kitchen-sink model (vn 13:55).

## Phases and exit tests

| Phase | Work | Exit test |
|---|---|---|
| 0. Brief | this repo, cleaned transcript, plan, tasks | done 2026-10-24 |
| 1. Research | prior art (unified ASR + post-processing models), ASR models with a paragraph/newline token, feasibility of training these tasks into an ASR decoder, candidate base models | `docs/research.md` with sources, dates, and a confirmed/inferred status per claim |
| 2. Data audit | minutes and takes per in-scope category; gaps | table in `docs/data.md`; every core category ≥ the minimum Daniel sets |
| 3. Spec | final task list and tiers, targets, metrics, blend ratios | `docs/spec.md` agreed |
| 4. Prototype | first fine-tune on core + whispered | per-category eval vs stock Parakeet; no regression over 1 pt WER on `general` |
| 5. Extend | ambitious and experimental tasks one at a time | each task's eval, kept or dropped with a reason |
| 6. Publish | HF model, card, eval table | model card live |

## Open questions (need Daniel)

1. **English-only base?** Raised at vn 23:39. With domain vocab out of scope (vn 23:54), inferred
   answer: yes, the English-only Parakeet v2 is fine; code-switching comes back only in a later
   layer, and that layer's base question can wait. Confirm.
2. **Paragraphs.** Parakeet's tokenizer may have no newline token (vn 16:19, "if I'm not
   mistaken"); to verify in phase 1. If not: add a token and resize the decoder, change base model,
   or leave paragraphs to `ASR-Paragraph-Breaks`?
3. **Tiers.** Confirm the table above, especially the "open" row.
4. **One model or two passes inside one?** vn 19:27 asks whether one model can hold an ASR pass
   and a post-processing pass. Treated as a research question in phase 1, not a design commitment.
5. **Recording budget.** How many more minutes per thin category is Daniel willing to record?

## Risks

- Tasks such as asides and self-corrections need context the ASR decoder may not model well at
  this size; they may not train (Daniel expects some won't, vn 17:54).
- Cleaning in the decoder can over-delete real words; the eval must measure WER on the kept text,
  not only task success.
- Read-aloud disfluency data is acted, not natural; it may not transfer to real dictation.
