# CLAUDE.md

Public repo for Proofed: a fine-tune of a stock ASR model (Parakeet TDT 0.6B v2 first) trained only
on post-processing tasks (disfluencies, repeats, false starts, numbers in digits, self-corrections,
asides, paragraphs, whispered speech), so it writes clean text in one pass with no LLM step.
Research prototype for Hugging Face. Status 2026-10-24: phase 1 research done (viable); no data split, run or model.

## Where things are

- Brief: `context/voice-notes/2026-10-24-proofed-brief/`: `voice-note.mp3` (polished),
  `transcript-raw.txt` (AssemblyAI output, unedited; never "fix" it), `transcript-clean.md`,
  `processing.json` (Drive id, AssemblyAI id).
- Plan, scope, tiers, open questions: `docs/development-plan.md`. Tasks: `docs/tasks.md`.
- Research: `docs/research.md` (summary), `research/2026-10-24-phase-1-feasibility/` (report, notes,
  sources).
- Recording: the recorder app (`~/repos/github/ai-models/Voice-Training-Recorder`,
  train.hebrish.app) has a **Proofed** training mode since release 25 (2026-10-24), its own button on the Record mode bar (STT | Proofed | Other) since release 29. Takes land in
  `danielrosehill/Read-Aloud-Voice-Sentences` and the `Hebrish/*` task subsets.
- Design: `docs/concept.md` → `docs/Proofed-concept-note.pdf` (render with the modular-research
  `render_pdf.sh`; diagrams `docs/diagrams/*.mmd` → `docs/img/*.png` via mermaid-cli 11.4.2),
  taxonomy `docs/taxonomy.md` + `data/taxonomy.json`, paragraph-token survey `notes/06`.
- Recording targets per subtask: `docs/recording-targets.md`.
- Later voice notes go in their own `context/voice-notes/YYYY-MM-DD-<slug>/` (skill
  `ideation-planning:voice-note-kickoff`).

## Hard rules

- **Domain vocabulary is out of scope:** no code-switching, tech, names or brand-name training
  data in Proofed blends (voice note 23:54). Those come later as a layer on top.
- Only transformations right for every user every time. Nothing use-specific (email formatting,
  style).
- One task per category, each evaluated on its own.
- Source-of-truth order: the voice note, then `docs/development-plan.md`, then everything else.
  When they disagree, record the decision in the plan.

## Skills for this repo

- `skill:modular-research/research-report`: research tasks (e.g. phase 1 in `docs/tasks.md`)
  go to `research/YYYY-MM-DD-<slug>/` as a timestamped executive report PDF, emailed.
- `skill:ideation-planning/voice-note-kickoff`: further voice notes about Proofed.

## Related repos

- `~/repos/github/audio-dictation/English-Hebrew-Codeswitching-Parakeet-Finetune`: the NeMo
  training setup, sentence bank and `split_subsets.py`.
- `~/repos/github/audio-dictation/Hebrish-App/data/training-categories.json`: category ids used here.
- `~/repos/github/audio-dictation/ASR-Paragraph-Breaks`, `ASR-Post-Processor-Model`: the text-side
  alternatives (baselines).
- `~/repos/github/ai-models/ASR-Dataset-Blending`: replay ratios.
