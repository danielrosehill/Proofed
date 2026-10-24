# Proofed

Proofed: integrated ASR + post-processing model, from speech straight to clean, proofed text.

A stock speech-recognition model (NVIDIA Parakeet TDT 0.6B v2 to start) fine-tuned only on
cleanup tasks, so the transcript comes out already proofed: no ums, repeats or false starts,
numbers as digits, self-corrections applied, remarks to someone else in the room left out, and
whispered speech handled. One small model, one pass, no LLM afterwards. Domain vocabulary is out
of scope: that is a later layer on top.

**Status (2026-10-24): planning.** No model yet.

## Project brief

The project started from a 25-minute voice note:

- [Listen](context/voice-notes/2026-10-24-proofed-brief/voice-note.mp3) (MP3, 24:40)
- [Transcript, cleaned](context/voice-notes/2026-10-24-proofed-brief/transcript-clean.md) ·
  [raw ASR output](context/voice-notes/2026-10-24-proofed-brief/transcript-raw.txt)
- [Development plan](docs/development-plan.md) · [Tasks](docs/tasks.md)
- [Project brief (PDF)](context/voice-notes/2026-10-24-proofed-brief/Proofed-project-brief.pdf): overview, plan and the edited transcript in one document
