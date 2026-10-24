# Proofed

Proofed: integrated ASR + post-processing model, from speech straight to clean, proofed text.

A stock speech-recognition model (NVIDIA Parakeet TDT 0.6B v2 to start) fine-tuned only on
cleanup tasks, so the transcript comes out already proofed: no ums, repeats or false starts,
numbers as digits, self-corrections applied, remarks to someone else in the room left out, and
whispered speech handled. One small model, one pass, no LLM afterwards. Domain vocabulary is out
of scope: that is a later layer on top.

**Status (2026-10-24): phase 1 research done (viable).** No model yet.

## Project brief

The project started from a 25-minute voice note:

- [Listen](context/voice-notes/2026-10-24-proofed-brief/voice-note.mp3) (MP3, 24:40)
- [Transcript, cleaned](context/voice-notes/2026-10-24-proofed-brief/transcript-clean.md) ·
  [raw ASR output](context/voice-notes/2026-10-24-proofed-brief/transcript-raw.txt)
- [Development plan](docs/development-plan.md) · [Tasks](docs/tasks.md)
- [Project brief (PDF)](context/voice-notes/2026-10-24-proofed-brief/Proofed-project-brief.pdf): overview, plan and the edited transcript in one document
- [Phase 1 research (PDF)](research/2026-10-24-phase-1-feasibility/Proofed-research-report.pdf): feasibility, prior art, base models, training data · [summary](docs/research.md)
- [Brief and research (PDF)](docs/Proofed-brief-and-research.pdf): both of the above in one document, 29 pages

## Design

- [Concept note (PDF)](docs/Proofed-concept-note.pdf) ([source](docs/concept.md)): the idea, challenges, task table, one-pass vs two-pass architecture options with diagrams, paragraph-token routes, questions for reviewers
- [Task taxonomy](docs/taxonomy.md) ([JSON](data/taxonomy.json)): every post-processing task, its operation type and tier
- [Paragraph token survey](research/2026-10-24-phase-1-feasibility/notes/06-parakeet-newline-survey.md): newline support across all NVIDIA Parakeet, Canary and Nemotron speech models
- [Recording targets](docs/recording-targets.md): takes per subtask for the first fine-tune
- [Training data on Hugging Face](https://huggingface.co/datasets/danielrosehill/Proofed-ASR-Cleanup-Training-Data): public, CC-BY-4.0; first snapshot 2026-10-24 (600 takes, ~75 min)
