# 03. Candidate base models: which small English ASR models besides parakeet-tdt-0.6b-v2 are good bases for Proofed?

## Answer

No small model clearly beats `nvidia/parakeet-tdt-0.6b-v2` on English accuracy. On the current Open ASR Leaderboard (version 21-10-2026, page last updated 22 October 2026), only `Qwen/Qwen3-ASR-0.6B` (6.24) and `ibm-granite/granite-speech-5.0-470m-turboctc` (6.38) come near v2 (6.02) among open models of about 1B parameters or less on sherpa-onnx or similar runtimes [S0323]. `Edge0/ARK-ASR-0.6B` scores 5.93, but it is 1.3B parameters and has no phone runtime [S0322][S0327]. What sets the candidates apart for Proofed is the decoder and the tokenizer, not WER. Every NVIDIA model whose tokenizer I could test strips `\n` during normalisation: parakeet-tdt-0.6b-v3 and nemotron-speech-streaming-en-0.6b (test below). Granite 5.0 470M also strips it. Byte-level or byte-fallback tokenizers keep it: Whisper, Moonshine v1 and v2, Kyutai STT, Granite 4.0 speech, and Qwen3-ASR [S0328]. Of those, only Qwen3-ASR-0.6B (LLM decoder, Apache-2.0, int8 sherpa-onnx build that runs on Android) and Moonshine run on a phone today through sherpa-onnx. For Moonshine that is v1 only; v2 Streaming has no sherpa-onnx build yet [S0324][S0326]. The one direct comparison of architectures on fluent-transcript output I found is old (Lou & Johnson 2020). In it, attention models (seq2seq, Transformer) removed disfluencies better than CTC, and all end-to-end models trailed a pipeline [S0329]. I found no study comparing a transducer (RNN-T/TDT) with an AED on this task. That is a gap, not a finding. My ranked shortlist, with v2 kept as the default: (1) Qwen3-ASR-0.6B, (2) canary-180m-flash, (3) nemotron-speech-streaming-en-0.6b.

## Evidence

### Comparison table

WER columns: **OAL-now** is the live Open ASR Leaderboard, version 21-10-2026 [S0323]. It uses cleaned AMI, Earnings22, GigaSpeech and VoxPopuli plus URGENT24, Voice Arena and private sets, so it **cannot be compared** with **Card avg**, the average of the original 8 Open ASR datasets as reported on each model card. "Newline" is the result of my own round-trip test of the published HF `tokenizer.json`: encode then decode `"…here.\n\nSecond…"` and check whether `\n` survives [S0328]. "n/t" means not tested, because the tokenizer exists only inside the `.nemo` file.

| Model | Params | Arch | PnC | Vocab / newline | Licence | Fine-tune route | Streaming | On-device runtimes | Card avg WER | OAL-now |
|---|---|---|---|---|---|---|---|---|---|---|
| parakeet-tdt-0.6b-v2 (baseline) | 600M [S0301] | FastConformer-TDT | yes [S0301] | n/t (agent 01) | CC-BY-4.0 | NeMo | no (offline, ≤24 min) | sherpa-onnx int8/fp16 [S0324] | 6.05 [S0301] | 6.02 [S0323] |
| parakeet-tdt-0.6b-v3 | 627M [S0327] | FastConformer-TDT, 25 langs | yes | 8,192; **newline stripped** (its `Ċ` token is the Maltese letter, not GPT-2's newline; inference) [S0302][S0328] | CC-BY-4.0 | NeMo; HF Transformers | chunked inference script [S0302] | sherpa-onnx int8 [S0324] | 6.34 [S0302] | 6.18 [S0323] |
| parakeet-tdt_ctc-110m | ~114M [S0303] | hybrid FastConformer TDT+CTC | yes | 1,024 per card table (labelled v2.0) [S0303]; n/t | CC-BY-4.0 | NeMo | not stated | sherpa-onnx (CTC and TDT builds) [S0324] | 7.49 [S0303] | not listed |
| parakeet-unified-en-0.6b (new, 2026-04-07) | 600M | Unified FastConformer **RNN-T**, one model for offline and streaming | yes | n/t | NVIDIA Open Model License | **inference only; training pipeline "soon"** [S0306] | yes, 0.16–2.08 s latency | sherpa-onnx int8 offline and streaming [S0324] | 5.91 offline; 6.52 at 0.56 s [S0306] | not listed |
| nemotron-speech-streaming-en-0.6b | 618M [S0327] | FastConformer cache-aware RNN-T | yes [S0304] | 1,024 (+3); **newline stripped** [S0328] | NVIDIA Open Model License [S0304] | NeMo (owner has done it) | yes, chunks of 80/160/560/1120 ms | sherpa-onnx int8 per chunk size [S0324]; GGUF [S0304] | 6.93 at 1.12 s; 8.43 at 80 ms [S0304] | 6.60 [S0323] |
| nemotron-3.5-asr-streaming-0.6b | ~600M | same + language-ID prompt, 40 locales | yes | n/t | OpenMDW-1.1 [S0305] | NeMo (NVIDIA blog) | yes | sherpa-onnx int8 [S0324] | English FLEURS only, 7.91–9.43 [S0305] | 9.75 [S0323] |
| canary-1b-flash | 883M (card), 811M (safetensors) [S0307][S0327] | AED: FastConformer, 32 enc + 4-layer Transformer dec | optional (prompt) | concatenated SentencePiece; size not stated; n/t | CC-BY-4.0 | NeMo | no | **no sherpa-onnx build found** [S0324] | 6.35 [S0307] | not listed |
| canary-180m-flash | 182M [S0308] | AED: 17 enc + 4 dec | optional (`pnc`) | concatenated SentencePiece; n/t | CC-BY-4.0 | NeMo | no | sherpa-onnx int8 (2025-07-07) [S0324] | 7.12 [S0308] | not listed |
| Moonshine v1 tiny / base | 27M / 61M [S0310] | AED (enc-dec Transformer) | not stated | 32,768 byte-fallback BPE [S0310]; **newline kept** [S0328] | MIT | HF (no official recipe) | no | sherpa-onnx int8/quantized [S0324] | 12.66 / 10.07 [S0310] | removed 31 Jul 2026 [S0323] |
| Moonshine v2 Streaming tiny / small / medium (Jan–Feb 2026) | 34M / 123M / 245M [S0312] | AED with sliding-window streaming encoder | not stated | same tokenizer [S0312]; **newline kept** [S0328] | MIT [S0327] | HF Transformers; no official fine-tune recipe [S0311] | yes, 80 ms lookahead [S0311] | Moonshine Voice; community GGUF; **no sherpa-onnx v2 build found** [S0324] | 12.01 / 7.84 / 6.65 [S0312] | not listed |
| Whisper small | 242M [S0327] | AED | yes | 51,865 byte-level; **newline kept** [S0328] | Apache-2.0 [S0327] | HF (well documented) | no | sherpa-onnx, whisper.cpp | not found | not listed |
| distil-small.en | 166M [S0314] | AED (distilled) | yes | Whisper tokenizer | MIT | HF distil-whisper training code | no | whisper.cpp, sherpa-onnx [S0314][S0324] | short-form OOD 12.1 (distil metric) [S0314] | not listed |
| whisper-large-v3-turbo | 809M, 4 decoder layers [S0313] | AED | yes | byte-level (Whisper) | MIT | HF | no | sherpa-onnx "turbo", whisper.cpp [S0324] | — | 7.64 [S0323] |
| distil-large-v3.5 | 756M [S0327] | AED | yes | Whisper | MIT | HF | no | sherpa-onnx [S0324] | — | 6.76 [S0323] |
| Kyutai stt-1b-en_fr | 989M [S0327] | decoder-only over Mimi audio tokens (Moshi) | yes [S0315] | 8,000 unigram, byte-fallback; **newline kept** [S0328] | CC-BY-4.0 | moshi / Transformers ≥4.53; no fine-tune recipe on card | yes, 0.5 s delay | PyTorch, Transformers (card); no sherpa-onnx | not on card | not listed |
| granite-speech-5.0-470m-turboctc (Aug 2026, multilingual update 21 Oct 2026) | 473M [S0327] | Conformer **CTC** (non-autoregressive) | yes [S0316] | 16,384 BPE; **newline stripped** [S0328] | Apache-2.0 | HF Transformers ≥5.16, Colab notebook | not stated | mlx-audio; no sherpa-onnx [S0316][S0324] | 5.51 (card's own) [S0316] | 6.38 [S0323] |
| granite-4.0-1b-speech | 2.31B (safetensors) despite the name [S0327] | Conformer enc + Q-former + Granite LLM (LoRA) | n/s | 100k byte-level; **newline kept** [S0328] | Apache-2.0 | HF notebook | no | vLLM, mlx-audio [S0317] | 5.52 [S0317] | 6.41 [S0323] |
| Qwen3-ASR-0.6B (2026-01-29) | 938M [S0327] | audio encoder + Qwen3 **LLM decoder** | not stated on card | 151,643 byte-level BPE; `Ċ`/`ĊĊ` newline tokens [S0328] | Apache-2.0 | official `finetuning/` dir [S0319]; owner has fine-tuned the 1.7B | yes (vLLM only) [S0318] | **sherpa-onnx int8, Android arm64** [S0324][S0326] | card: LS 2.11/4.55, TED 3.85, VoxPop 9.96 [S0318] | 6.24 [S0323] |
| Voxtral Mini 4B Realtime (2602) | ~4.4B [S0320][S0327] | streaming LLM | yes | Tekken | Apache-2.0 | none on card | yes, 480 ms recommended | ExecuTorch (untested), voxtral.c, MLX [S0320] | — | 8.44 [S0323] |
| Voxtral Mini 3B (2507) | 4.68B safetensors [S0327] | LLM | yes | Tekken | Apache-2.0 | HF | no | MLX | — | 7.25 [S0323] |
| Phonon-2 (Fermion, 2026-09-28) | parakeet-v3 compressed to 164 MB [S0321] | TDT (derivative) | as v3 | as v3 | CC-BY-4.0 | not stated; a quantised derivative is a poor fine-tuning base (inference) | — | MLX, Core ML, Android arm64 CPU libs [S0321] | 5.21 (vendor, 7 sets) [S0321] | 6.87 [S0323] |
| ARK-ASR-0.6B (Edge0, May 2026) | 1.30B safetensors [S0327] | Whisper-style enc + Qwen2 0.6B decoder | n/s | Qwen2 byte-level (inferred) | Apache-2.0 | remote code; OPD recipe | no | Transformers only [S0322] | 6.55 [S0322] | 5.93 [S0323] |

### Further points

- The Open ASR Leaderboard is at version "21-10-2026", page last updated 22 October 2026. On 31 July 2026 it **removed older models, including `usefulsensors/moonshine-base`**, and switched to cleaned datasets plus private sets. Moonshine, Canary-flash, Kyutai, Whisper small and parakeet 110M therefore have no current OAL row [S0323]. Confirmed from the space's rendered config.
- On OAL-now, parakeet-tdt-0.6b-v2 (6.02) beats v3 (6.18) and nemotron streaming (6.60) [S0323]. The owner's own dictation use agrees: "v2 was more accurate than v3 for English dictation" [S0331]. Confirmed.
- Granite 5.0 470M: the model card claims 5.51 average, but OAL-now shows 6.38. The `-nc` variant scores 6.02 but is non-commercial [S0316][S0323]. The two numbers come from different dataset versions, so this is not a contradiction.
- sherpa-onnx `asr-models` release assets include, among others: parakeet v2/v3, 110M, nemotron-speech-streaming at four chunk sizes (2026-04-25), parakeet-unified streaming and offline (2026-04/05), canary-180m-flash, qwen3-asr-0.6B int8 (2026-03-25), moonshine v1 tiny/base, Whisper small, turbo and distil, and cohere-transcribe. There is **no canary-1b-flash, Granite, Kyutai, Voxtral or Moonshine v2 Streaming** asset [S0324][S0325]. Confirmed (498 assets listed via GitHub API).
- sherpa-onnx lists Android (arm64-v8a, armv7, x86, x86_64) for Qwen3-ASR. It gives no RTF on a phone [S0326]. Speed on a phone is not confirmed.
- **Tokenizer and newline** [S0328]. Own test, 2026-10-24, HF `tokenizers` library, on the HF-converted `tokenizer.json`:
  - `\n\n` survives the round trip for Whisper-small, Moonshine v1 and v2, Kyutai STT, Granite-4.0-1b-speech, and Qwen3-ASR (by vocab inspection: `Ċ` and `ĊĊ` are present).
  - `\n` is normalised to a space for parakeet-v3, nemotron-streaming and Granite 5.0 470M.
  - Caveat 1: NeMo trains from the SentencePiece `.model` inside the `.nemo` file, not this HF conversion. I assume the two normalise the same way (inferred).
  - Caveat 2: a tokenizer that *can* encode a newline does not mean the model was trained to emit one (inferred).
- **AED/LLM vs transducer for deletions.** Lou & Johnson (2020) trained CTC (Jasper), LSTM seq2seq and Transformer models end-to-end on Switchboard to produce fluent transcripts. Disfluent-word error rate (DER) on test was 26.2 for CTC, 25.6 for seq2seq and 24.0 for the Transformer. The Transformer pipeline scored 22.2. The paper says the Transformer is best at disfluency removal and attributes this, speculatively, to self-attention [S0329]. Confirmed, but the models are pre-2021 and small. No transducer was tested.
- Futami et al. (ICASSP 2023) did joint streaming ASR and disfluency detection with **Transformer encoder-decoder** models. They found that inline disfluency tags cause latency problems in streaming [S0330]. This was detection (tagging), not deletion. Confirmed from the abstract; full text not read.
- Inference, not sourced: Proofed's core tasks are monotonic. Fillers, repeats, false starts and self-corrections are deletions, and number words → digits keeps the order. A transducer emits blank or nothing over deleted spans, so monotonic alignment should not block them. Two kinds of task are harder for a transducer:
  1. Edits that **reorder** or need lookahead, such as "fifty shekels" → "50 NIS" when the symbol comes first, or "$50", or a paragraph break decided by what follows.
  2. Edits that depend on what comes *later*, such as "Haifa, I mean Tiberias": the "Haifa" tokens must be withheld until "I mean" is heard.

  Offline TDT (v2) has a full-context encoder, so the encoder can carry the later context. Cache-aware streaming models (nemotron, parakeet-unified at small chunks) cannot look that far ahead, so they are the riskiest for self-corrections. An AED or LLM decoder conditions on the whole utterance and can reorder freely. Its known costs are hallucination and repetition, and the Moonshine v2 card warns of exactly that on short or noisy audio [S0311].
- Canary-flash paper: a smaller decoder "may be less resilient to inaccurate labels" [S0309]. This matters for canary-180m-flash with a 4-layer decoder on a small, hand-labelled Proofed set (inference).

## Disagreements and gaps

- Parameter counts disagree:
  - canary-1b-flash: 883M on the card vs 811M in safetensors [S0307][S0327].
  - Granite-4.0-**1b**-speech: 2.31B in safetensors [S0327], so it is not a ≤1B model.
  - Qwen3-ASR-**0.6B**: 0.94B. Its "0.6B" refers to the LLM.
  - Voxtral "Mini": 4–4.7B. It is out of scope for a phone.
- canary-1b-flash `config.json` says `vocab_size: 1024`. That file is an encoder-only HF stub, and I did not trust it [S0307]. Canary vocab sizes and newline handling remain unknown; the tokenizer is only inside the `.nemo` file.
- PnC for Qwen3-ASR and the Moonshine models is not stated on their cards. That they are cased and punctuated is an inference from the byte-level tokenizers and typical outputs; not confirmed.
- No WER found for Whisper small on the current leaderboard, and none for Kyutai stt-1b-en_fr anywhere.
- Phonon-2: the vendor reports 5.21; OAL-now shows 6.87; a secondary blog independently got 7.23. Only the OAL figure was read at source.
- Not checked: the GGUF or transcribe.cpp status of each model beyond what the cards say, and Android latency for any model other than what the owner already ships.
- Searched and not found: any paper comparing RNN-T/TDT with AED on producing clean, written-form transcripts.

## Confidence

Medium. Parameters, licences, OAL-now rows, sherpa-onnx assets and the tokenizer newline test were read or run directly from the sources. Some model-card numbers came through a WebFetch summariser rather than raw text. The architecture claim (AED/LLM handles deletions and paragraphs better) rests on one 2020 paper plus inference. The tokenizer test used HF conversions, not the NeMo `.model` files.

## Ranked shortlist (with parakeet-tdt-0.6b-v2 kept as the default base)

1. **Qwen/Qwen3-ASR-0.6B.** The only phone-deployable candidate (sherpa-onnx int8, Android) that has an LLM decoder and a byte-level vocab with newline tokens. It is Apache-2.0, scores 6.24 on OAL-now, and has an official fine-tuning directory. The owner has already fine-tuned the 1.7B. Use it as the AED/LLM control arm for paragraphs and self-corrections [S0318][S0319][S0323][S0324][S0328].
2. **nvidia/canary-180m-flash.** An AED decoder inside the owner's NeMo and Modal tooling, CC-BY-4.0, 182M parameters, with a sherpa-onnx int8 build. It allows a cheap TDT-vs-AED comparison on the same data. Its newline handling is untested [S0308][S0324].
3. **nvidia/nemotron-speech-streaming-en-0.6b.** The streaming option in the same NeMo tooling, already fine-tuned and shipped by the owner, with a sherpa-onnx build for each chunk size. Expect it to be the weakest at edits that need right context, such as self-corrections (inference) [S0304][S0323][S0331].

Not shortlisted:
- **parakeet-tdt-0.6b-v3.** Worse English than v2 on OAL-now and in the owner's use, and it also strips newlines.
- **Moonshine v2 Streaming.** MIT, newline-capable, 6.65 at 245M, but no sherpa-onnx build and no fine-tuning recipe. Next in line if its runtime support arrives.
- **granite-speech-5.0-470m.** CTC, strips newlines, no sherpa-onnx build.
- **parakeet-unified-en-0.6b.** Cannot be fine-tuned yet.
- **Granite-4.0-1b-speech, ARK-ASR, Voxtral.** Over 1B real parameters or no phone runtime.
