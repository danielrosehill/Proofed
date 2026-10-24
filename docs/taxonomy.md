# Proofed: task taxonomy

What Proofed bakes into the ASR model, and what it leaves out. Daniel asked for this in the
voice note (narrow, modular tasks, vn 17:54) and again on 2026-10-24 ("build out our own taxonomy
... of the specific tasks, post-processing tasks to be incorporated into the model").
Machine-readable copy: [`data/taxonomy.json`](../data/taxonomy.json). The dataset category ids are
the recorder's (`Hebrish-App/data/training-categories.json`, `Voice-Training-Recorder` `TASKS`).

## The admission rule

A transformation belongs in Proofed only if it is **right for every user, every time** (vn 15:17):

- it doesn't depend on what the text is for (no email formatting, no style);
- it doesn't depend on who is speaking or their field (no domain vocabulary, vn 23:54);
- the clean output is one well-defined string, so it can be scored.

Everything else is a later layer on top (vn 13:55).

## Operation types

Each task is classified by **what it does to the token sequence**. This matters because the decoder
architecture decides which operations are easy (see the [concept note](concept.md), "Architecture"):

| Op | Meaning | Transducer (TDT) difficulty |
|---|---|---|
| **delete** | drop spans; the rest is unchanged and in order | easy: emit blank over the span |
| **delete-ctx** | drop a span whose deletion is only decidable from *later* audio | medium: needs right context (offline encoder has it; streaming does not) |
| **rewrite** | replace words by their written form, same order | easy–medium: monotonic |
| **reorder** | the written form moves a token (fifty dollars → $50) | hard: output order ≠ audio order |
| **structure** | insert layout (paragraph break) | medium: needs a token, plus discourse context |
| **filter** | drop speech not meant for the transcript | hard: a judgement, prosodic and semantic |
| **robust** | no text change; the audio condition differs | acoustic only |

## Taxonomy

| Id | Family | Task | Op | Example (said → written) | Tier | Dataset category |
|---|---|---|---|---|---|---|
| A1 | Disfluency | Filled pauses | delete | "I, uh, think so" → "I think so" | core | `disfluency` |
| A2 | Disfluency | Repetitions | delete | "the the report" → "the report" | core | `repetitions` |
| A3 | Disfluency | False starts / restarts | delete | "And going to the — I'm going to the cinema" → "I'm going to the cinema" | core | `false-starts` |
| A4 | Disfluency | Abandoned sentences | delete-ctx | "I think I had an idea — I want to go to the movies" → "I want to go to the movies" | core | `false-starts` |
| B1 | Repair | Self-corrections | delete-ctx | "Haifa today. I mean Tiberias" → "Tiberias today" | ambitious | `self-corrections` |
| B2 | Repair | Editing commands | delete-ctx | "Jerusalem. Scratch that, Tel Aviv" → "Tel Aviv" | ambitious | `self-corrections` |
| C1 | Written form | Numbers, dates, units, times | rewrite | "twenty twenty-six" → "2026"; "forty-five seconds" → "45 seconds" | core | `numerics` |
| C2 | Written form | Money | reorder | "fifty dollars" → "$50"; "fifty shekels" → "50 NIS" | core | `currency` |
| D1 | Structure | Paragraph breaks | structure | (pause, topic shift) → blank line | ambitious | `paragraphs` |
| E1 | Filtering | Asides to someone else | filter | "the API docs are here — just leave it by the door — now" → "the API docs are here now" | experimental | `asides` |
| F1 | Robustness | Whispered speech | robust | (whispered) → same text | robustness | `whispered` |

**Inherited, not trained:** sentence punctuation and casing come from the base model (Parakeet v2 is
PnC). Proofed must keep them, and the eval scores them.

**Negative controls, inside each task's prompts:**
- intentional repetition ("very, very good", "bye bye") stays;
- quoted or reported disfluency stays when it is content;
- numbers that are names or idioms stay words ("one of a kind", "Formula One").

Over-deletion is the failure mode the eval watches, as WER on the kept text.

## Open: written-form conventions (decide before phase 3)

These pass the admission rule for most users, but they touch vocabulary. Plan open question 3.

| Id | Task | Op | Example | Category |
|---|---|---|---|---|
| C3 | Web addresses | rewrite | "example dot com slash page" → "example.com/page" | `web-addresses` |
| C4 | Spelled-out letters | rewrite | "B R E N N A N" → "Brennan" | `spelled-out` |
| C5 | Acronyms in standard form | rewrite | "A P I" → "API" | `acronyms` |
| C6 | Expanded chat shorthand | rewrite | "E O D" → "end of day" | `expanded-acronyms` |
| A5 | Discourse markers and editing terms | delete | "so, like, you know, it works" → "so it works" | none yet; risky, because these words often carry meaning |

## Out of scope

| What | Why | Where it goes |
|---|---|---|
| Domain vocabulary: code-switching, tech terms, names, brands | user-specific (vn 23:54) | a layer on top of Proofed (Hebrish lines) |
| Style and format: email, lists, formal/casual | use-specific (vn 15:17) | a text model or LLM afterwards |
| Translation, summarisation | not transcription | out |
| Singing, put-on voices, name cipher | other experiments | Hebrish robustness sets |

## Evaluation per task

Each task is scored on its own (plan rule: one task per category):

| Score | What it measures |
|---|---|
| **Task success** | filler gone; repeat gone; correction applied (kept version present, replaced version absent); digits and currency exact; aside absent; paragraph break F1 |
| **WER on the rest** | errors in the words that should be kept |
| **Regression** | WER on plain English (`general`, LibriSpeech-PC test-clean/other) |
