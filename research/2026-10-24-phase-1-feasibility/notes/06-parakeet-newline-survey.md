# 06. Can any Parakeet (or other NVIDIA NeMo ASR) model emit a newline or paragraph break?

Asked by Daniel on 2026-10-24: "regarding the paragraph breaks, which is really important ... see if there's any other Parakeet models that support this". Agent 06, 2026-10-24. Every claim is tagged **Confirmed** (I ran it or read it at source) or **Inferred**.

## Answer

**No. None of the 22 NVIDIA NeMo ASR models on Hugging Face in the Parakeet, Canary and Nemotron-speech families can emit a newline as shipped.** I pulled the actual SentencePiece `tokenizer.model` out of every `.nemo` archive, using HTTP range requests and a gzip stream for nemotron-3.5, and round-tripped `\n`, `\n\n`, `¶` and U+2029. All 22 tokenizers use `nmt_nfkc` or `nmt_nfkc_cf` normalisation, and none has byte fallback. None has a piece containing `\n`, `\r`, tab, U+2028/2029, `¶` or `§`. `\n` never survives encode→decode, and `¶` always becomes `<unk>` [S0601][S0602] (Confirmed). No model card mentions paragraphs or newlines [S0616] (Confirmed). The one NVIDIA ASR model whose tokenizer *can* hold a newline is **canary-qwen-2.5b**. It decodes with Qwen3-1.7B, whose vocab has `Ċ` 198 and `ĊĊ` 271. But it is 2.5B parameters, and its ASR mode is trained on ordinary transcripts [S0613]. Confirmed that the tokenizer has the tokens; Inferred that it does not emit them in ASR mode.

Two findings change the routes compared with note 01:

1. **NVIDIA already ships a Parakeet with tokens added after training, and NeMo has a recipe for it.** `parakeet_realtime_eou_120m-v1` has `<EOU>` and `<EOB>` appended as USER_DEFINED pieces at ids 1024/1025. The base vocab is 1,024, and the trainer spec's `user_defined_symbols` is empty, so they were edited into the proto after training. Its config carries `token_init_method: constant`, `token_init_bias_value: -1000.0` [S0602][S0608] (Confirmed). The NeMo Speech repo has `scripts/asr_eou/tokenizers/add_special_tokens_to_sentencepiece.py`, which appends USER_DEFINED pieces. It also has `examples/asr/asr_eou/speech_to_text_rnnt_eou_train.py::init_from_pretrained_nemo`, which copies every trained row of the encoder, the decoder embedding and the joint output and initialises only the new rows [S0605][S0606][S0607] (Confirmed). Token surgery is therefore NVIDIA's own pattern, not a hack. It was written for plain RNN-T, though, and its row slicing (`[:-3]`, `[-1]` = blank) is **wrong for TDT**, where the 5 duration logits come after the blank (see Routes).
2. **A literal `\n` piece works if it is added as a USER_DEFINED symbol.** I appended `"\n"` to the v2 tokenizer by the NeMo script's method. `"end of point.\n\nNext point."` then encodes as `['\n','\n','N',…]` and decodes back exactly. SentencePiece matches user-defined pieces before NFKC normalisation eats the newline [S0604] (Confirmed). The same holds for `¶`, U+2029 and `<|para|>`. So surgery can give Proofed a real newline token, not just a sentinel.

There is also a third route that needs no surgery at all. **parakeet-tdt-0.6b-v3, canary-1b-v2 and canary-1b-flash/180m-flash ship 30 reserved, never-used `<|spltoken0|>`…`<|spltoken29|>` tokens** that are already in the output layer. Examples: v3 ids 244–273 in an 8,192 vocab that the joint fully covers, and canary-1b-v2 ids 1133–1162. They encode as one piece and decode back literally [S0603] (Confirmed). Repurposing `<|spltoken0|>` as "paragraph" on v3 needs no code change, but it means leaving v2, which is the better English base (note 03).

**Recommendation:** stay on v2 and do **TDT-aware token surgery**: add one USER_DEFINED `"\n"` piece (or `<|para|>`, swapped for `\n\n` after decoding) at id 1024. Adapt NVIDIA's EOU warm-start to the TDT row layout, and run the `^` sentinel as the zero-surgery control arm in the same experiment. Details below.

## Table

Params are safetensors totals where the repo has them, otherwise ≈ `model_weights.ckpt` bytes / 4 (fp32). "Survives?" means encode→decode of `"Hello.\nNew paragraph."` and `"Hello.\n\nNew paragraph."` with the tokenizer from the `.nemo` file [S0602]. "dbl-space" means the double space that `\n\n` turns into is kept as an extra `▁` piece (`remove_extra_whitespaces=False`). Sentinel candidates are single-character non-alphanumeric NORMAL pieces that round-trip exactly in `"Hello. X New"`. Tokenizers that are byte-identical (md5) are marked = [S0602].

| Model | Params | Decoder | Tokenizer / vocab | Newline survives? | Sentinel candidates (single piece, round-trip) | Licence | Notes |
|---|---|---|---|---|---|---|---|
| parakeet-tdt-0.6b-v2 (baseline) | ≈618M | TDT, durations 0–4 | SP BPE 1,024, `nmt_nfkc`, no byte-fallback, 0 specials | No; `\n\n`→dbl-space `▁` | `^` 918, `¡` 1012, `¿` 904, `€` `£` `$` `%` `/` `:` `-` `!` | CC-BY-4.0 | Agent 01's result, reproduced with md5 `5bf007e9…` |
| parakeet-tdt-0.6b-v3 | 627M | TDT, 0–4 | SP BPE 8,192, 273 USER_DEFINED (canary-style prompt tokens, `<|spltoken0-29|>`, `<|spkchange|>`) | No; dbl-space | `¡` `¿` `€` `£` `$` `%` `/` `:`; **reserved `<|spltoken0..29|>` ids 244–273** | CC-BY-4.0 | Reserved tokens are already output classes (joint `num_classes` 8192) |
| parakeet-tdt-1.1b | ≈1.07B | TDT, 0–4 | SP BPE 1,024, **`nmt_nfkc_cf` (lower-cases)**; = rnnt/ctc 0.6b/1.1b | No; `.` and `\n` → `<unk>` | none (no punctuation pieces) | CC-BY-4.0 | No PnC tokenizer; unsuitable |
| parakeet-rnnt-0.6b / -1.1b | 617M / 1.07B | RNN-T | same lower-case 1,024 as tdt-1.1b | No | none | CC-BY-4.0 | No PnC |
| parakeet-ctc-0.6b / -1.1b | 609M / 1.06B | CTC | same lower-case 1,024 | No | none | CC-BY-4.0 | No PnC |
| parakeet-tdt_ctc-110m | ≈115M | hybrid TDT+CTC | SP BPE 1,024 `nmt_nfkc`, rmws=True; = tdt_ctc-1.1b | No (whitespace collapsed too) | `.` `,` `?` `!` `'` only | CC-BY-4.0 | No spare symbols at all |
| parakeet-tdt_ctc-1.1b | ≈1.07B | hybrid TDT+CTC | = 110m | No | as 110m | CC-BY-4.0 | |
| parakeet-unified-en-0.6b | ≈618M | RNN-T (unified offline/streaming) | SP BPE 1,024 `nmt_nfkc`; = nemotron-speech-streaming, multitalker | No; dbl-space | `^` 1022, `_` 1021, `=` 1002, `*` 1016, `&` 1015, `+` 1019, `¥` 1023, `"` `€` `£` `$` `%` `/` | NVIDIA Open Model License | Training pipeline not released (note 03) |
| nemotron-speech-streaming-en-0.6b | 618M | cache-aware RNN-T | = unified | No; dbl-space | as unified | NVIDIA Open Model License | Source model of NVIDIA's EOU recipe |
| multitalker-parakeet-streaming-0.6b-v1 | ≈622M | multitalker RNN-T | = unified | No | as unified | NVIDIA Open Model License | |
| parakeet_realtime_eou_120m-v1 | ≈115M (card: 120M) | streaming RNN-T | SP BPE 1,024 `nmt_nfkc_cf` + **2 USER_DEFINED `<EOU>` 1024, `<EOB>` 1025, added after training** | No | none (lower-case, no PnC) | NVIDIA Open Model License | **Precedent: tokens added to a trained Parakeet** |
| nemotron-3.5-asr-streaming-0.6b | 638M | RNN-T with language prompt | SP UNIGRAM 13,087 merged, 1 USER_DEFINED `<bg-BG>`, `nmt_nfkc` | No; dbl-space | `<` 1844, `>` 1845, `·` 9595, `¿` `!` `"` `-` `'` | OpenMDW-1.1 | `.nemo` is gzip; streamed. `<` `>` single pieces: `<EOU>` round-trips as text |
| parakeet-ctc-0.6b-Vietnamese | ≈609M | CTC | SP BPE 1,024 `nmt_nfkc` | No | `.` `,` `?` `!` | NVIDIA Open Model License | Vietnamese; noted only |
| parakeet-tdt_ctc-0.6b-ja | ≈622M | hybrid TDT+CTC | SP UNIGRAM 3,072 | No | none useful | CC-BY-4.0 | Japanese; noted only |
| parakeet-rnnt-110m-da-dk | ≈113M | RNN-T | SP BPE 44 (characters) | No | none | NVIDIA Open Model License | Danish; noted only |
| canary-180m-flash | ≈184M | AED (Transformer dec.) | aggregate: spl_tokens UNIGRAM 1,152 (1,150 USER_DEFINED) + 4×1,024 BPE (en/de/es/fr), head 5,248; en rmws=True | No (whitespace collapsed) | en: `¡` `¿` `«` `»` `@` `*` `[` `]` `` ` `` `°` `®` `&` `+` …; **reserved `<|spltoken0..29|>`** | CC-BY-4.0 | `^` is `<unk>` here |
| canary-1b-flash | 811M | AED | = 180m-flash (same md5s for spl and en) | No | as 180m-flash | CC-BY-4.0 | |
| canary-1b-v2 | 979M | AED | single SP BPE 16,384, 1,162 USER_DEFINED, rmws=False | No; dbl-space | `¡` `¿` `€` `£` `$` `%` `/` `:`; **reserved `<|spltoken0..29|>` ids 1133–1162** | CC-BY-4.0 | Same special-token scheme as v3 |
| canary-1b | ≈1.02B | AED | aggregate: spl 32 + en 1,024 | No | en: `.` `,` `?` `'` only; `<|spltoken0..17|>` | **CC-BY-NC-4.0** | Non-commercial |
| canary-qwen-2.5b | 2.56B | FastConformer + Qwen3-1.7B (LoRA) | Qwen3 byte-level BPE 151,643; **`Ċ` 198, `ĊĊ` 271** | Tokenizer: **yes** | n/a | CC-BY-4.0 | No `.nemo` in the repo; too big for a phone (Inferred) |

Not tested: the older `stt_en_fastconformer_*` checkpoints (e.g. `stt_en_fastconformer_hybrid_large_pc`). They are outside the families asked about, and are older, smaller-data predecessors.

**Canary prompt format** [S0603] (Confirmed). The canary2 prompt slots are `source_lang`, `target_lang`, `emotion`, `pnc`, `itn`, `diarize` and `timestamp`, plus `decodercontext`. The non-language specials are `<|nospeech|>`, `<|startofcontext|>`, `<|pnc|>`/`<|nopnc|>`, `<|itn|>`/`<|noitn|>`, `<|timestamp|>`, `<|diarize|>`, `<|spkchange|>`, `<|audioseparator|>`, `<|emo:*|>`, `<|spk0-15|>`, 900 numeric `<|n|>` timestamp tokens, and `<|spltoken0-29|>`. Nothing is paragraph-like. `<|spkchange|>` is the closest structural break, but it means speaker change and is tied to the diarize mode.

## Evidence

- **Confirmed:** the HF API lists 15 `parakeet*`, 5 `canary*` and 2 `nemotron*-speech/asr` ASR repos under `nvidia`, and none is gated. The `pipeline_tag=automatic-speech-recognition` listing (89 repos) adds only the `stt_en_fastconformer_*` series [S0601].
- **Confirmed:** I extracted the tokenizers from all 21 `.nemo` files without downloading the weights. Members come before `model_weights.ckpt` in every archive except canary-1b-v2, where the walker skipped over the 2.5 GB timestamps ckpt by its header. 34 `tokenizer.model` files were inspected [S0602].
- **Confirmed:** the normaliser is `nmt_nfkc` in 16 tokenizers and `nmt_nfkc_cf` (case-folding) in the 6 older or lower-case ones. `byte_fallback=False` everywhere. Newline-like pieces: none in any of them [S0602].
- **Confirmed:** byte-identical tokenizer groups: {rnnt-0.6b, rnnt-1.1b, ctc-0.6b, ctc-1.1b, tdt-1.1b}, {tdt_ctc-110m, tdt_ctc-1.1b}, {nemotron-speech-streaming, parakeet-unified, multitalker}, and canary-180m-flash = canary-1b-flash for all 5 sub-tokenizers. v2, v3, EOU, nemotron-3.5 and canary-1b-v2 are each unique [S0602].
- **Confirmed:** v3 and canary-1b-v2 encode `<|spltoken3|>` and `<|spkchange|>` as single pieces and decode them back literally. The NeMo `SentencePieceTokenizer` in non-legacy mode (`legacy=False`, the default) passes ids to `sp.encode`/`sp.decode` without its own special-token handling [S0603][S0612].
- **Confirmed:** adding `\n`, `¶`, U+2029 or `<|para|>` as a USER_DEFINED piece to the v2 proto makes each one a single id (1024) with an exact round trip [S0604].
- **Confirmed (code):** NVIDIA's EOU warm-start copies `dec_rnn`, `joint.enc` and `joint.pred` in full. In the decoder embedding it copies `[:-3] ← old[:-1]` and `[-1] ← old[-1]` (blank). In the joint final Linear it copies weight and bias the same way, then sets the new rows to a constant, or to the min/max/mean of the old rows. The shipped EOU model used weight `null` (left at random init) and bias −1000 [S0606][S0608].
- **Confirmed (code):** the NeMo Speech `main` (commit b19b1a7, 2026-10-23) `change_vocabulary` still runs `del self.joint` / `del self.decoder` and rebuilds the loss as `RNNTLoss(num_classes=self.joint.num_classes_with_blank - 1)`. Here `num_classes_with_blank` includes the extra outputs (`_num_classes = num_classes + 1 + num_extra_outputs`) and no `loss_name="tdt"` is passed. Inferred: on a TDT model this gives a plain RNN-T loss whose blank index is off by 5 [S0610].
- **Confirmed (code):** TDT greedy decoding splits the joint output as `logits[..., :-len(durations)]` (tokens + blank) and `logits[..., -len(durations):]` (durations). Any new token row must go in at index N, before the blank, and the last 5 rows must stay the duration rows [S0611].
- **Confirmed:** NVIDIA maintainer nithinraok, NeMo issue #15793 (2026-06-25): "Unfortunately with tokenizer re-initialization along with decoder, its not possible to retrieve prior accuracy". Extending the tokenizer while keeping the ids is not supported out of the box in the training scripts, and he gave the manual row mapping (old rows 0..N−1, blank N→M, new rows random). He did not mention TDT duration rows [S0609].
- **Confirmed:** NeMo issue #15936 (canary-1b-v2 drops Greek `ς`) shows how this fails in practice: a character with no piece is silently lost on output, with no error. A newline in a v2 manifest fails the same way [S0614].
- **Confirmed (negative):** GitHub issue searches on NVIDIA-NeMo/Speech for "newline", "paragraph", "spltoken" and "special tokens parakeet" found nothing about ASR paragraph or newline output. Web searches for Parakeet paragraph/newline output and NVIDIA release notes (Speech NIM 26.02) found nothing either [S0615][S0618]. None of the 22 model cards mentions paragraphs or newlines [S0616].

## Routes ranked for Proofed

### 1. Token surgery on v2 (recommended)

Steps:
1. Run `add_special_tokens_to_sentencepiece.py --input_file parakeet-tdt-0.6b-v2.nemo --tokens "\n"` (or `<|para|>`) [S0607]. This gives a 1,025-piece tokenizer with the new piece at id 1024, USER_DEFINED. The script needs `sentencepiece_model_pb2`, or the `sentencepiece` pip package's bundled `sentencepiece_model_pb2`, which is what I used [S0604]. A literal `"\n"` is cleanest: decoding yields real newlines with no post-processing, and `\n\n` is two consecutive tokens. `<|para|>`, swapped for `\n\n` in a string replace, is more robust, because manifests, Lhotse and JSON tooling cannot strip it (Inferred).
2. Build the new model from v2's config with `tokenizer.dir` pointed at the new tokenizer, `decoder.vocab_size: 1025` and `joint.num_classes: 1025`. Keep `num_extra_outputs: 5` and `loss_name: tdt`. **Do not call `change_vocabulary`**, because it reinitialises decoder and joint and drops the TDT loss [S0610].
3. Warm-start, adapting NVIDIA's EOU `init_from_pretrained_nemo` [S0606] to the TDT layout. Decoder embedding (1025+1 rows): new[0:1024] ← old[0:1024]; new[1025] (blank) ← old[1024]; new[1024] = new token. Joint final Linear, old 1,030 rows = 1024 tokens + blank + 5 durations, new 1,031 rows: new[0:1024] ← old[0:1024]; new[1025] ← old[1024] (blank); **new[1026:1031] ← old[1025:1030] (durations)**; new[1024] gets weight = mean of the token rows (or random) and bias ≈ −10 to −1000. Using the EOU script unchanged would copy a duration row into the blank slot (Inferred from [S0606]+[S0611]).
4. Train with replay of ordinary data, as the EOU README advises blending, so that WER does not drop [S0605]. Before any training, check that the new model's output on a held-out set is identical to stock v2 when the new token's bias is −1000. That is a cheap test that the surgery is correct.

Risks: (a) only one new row is learned from Proofed's small paragraph set; the EOU model shows that this works for a boundary-type token, but on 1,024 BPE with streaming data, not TDT (Inferred). (b) **TDT duration interaction:** when the new token is emitted, its frame also predicts a duration. A paragraph break usually sits at a pause, and the model must learn to emit the token with duration 0 or 1 at the pause frame and then skip. The duration head is shared and already trained, so this should be learned, but it is untested (Inferred). (c) Export: sherpa-onnx `tokens.txt` gains a line, and `\n` as a literal piece may break line-based `tokens.txt` parsing. Prefer `<|para|>` for on-device builds (Inferred; `tokens.txt` is one-token-per-line [S0102 in note 01]). (d) Hand-written loading code must be tested against the NeMo version the training repo pins.

### 2. Sentinel on v2 (zero surgery; control arm)

Steps: write `… end of point. ^ Next point …` in the target text and replace ` ^ ` with `\n\n` after decoding (note 01). `^` (918) is a NORMAL single piece. `¡` (1012) and `¿` (904) are backups [S0602].
Risks: (a) `^` has a trained output row whose meaning comes from Granary pseudo-labels; whether stock v2 ever emits it is still untested (grep a decoded test set). (b) It costs nothing in architecture, but a 3-piece pattern (`▁`, `^`, then `▁Next`) has to be learned, against 1 piece for a dedicated token (Confirmed piece count; learning cost Inferred). (c) A variant: training on raw `\n\n` silently becomes a lone `▁` piece (a double space) in v2, v3, unified and nemotron. This is fragile, because any whitespace normaliser destroys it, and canary-flash and tdt_ctc collapse it inside the tokenizer [S0602].
Use: run it alongside route 1 on the same data. If it matches route 1, ship it, since it is the simplest to deploy.

### 3. Switching base

- **parakeet-tdt-0.6b-v3 + `<|spltoken0|>`.** This needs no surgery and no code: the token is already an output class (its row was trained only as a negative, so in effect it is a pre-made new token). It decodes as literal `<|spltoken0|>`; replace it afterwards [S0603]. Costs: worse English than v2 (OAL 6.18 vs 6.02; owner's experience, note 03), an 8,192 vocab and 25 languages. Note 03 also shows sherpa-onnx int8 v3 exists. Second choice if surgery proves buggy.
- **canary-1b-v2 / canary-180m-flash + `<|spltoken0|>`.** Same trick, with an AED decoder that can reorder (note 03). For 180m-flash, check whether NeMo's multitask decoding strips `<|…|>` tokens from the output text; I did not confirm this (Gap).
- **parakeet-unified-en-0.6b, nemotron-speech-streaming.** Same 1,024 tokenizer family as the EOU recipe's source model, so NVIDIA's surgery script applies almost as is (plain RNN-T, no durations). Unified cannot be fine-tuned yet (note 03); nemotron streaming is weaker on right-context edits.
- **canary-qwen-2.5b.** The only NVIDIA model whose tokenizer already has newline tokens, but it is too big for Proofed's on-device goal (Inferred).
- Not suitable: tdt-1.1b, rnnt-*, ctc-* and EOU-120m (lower-case, no punctuation), and tdt_ctc-110m/1.1b (no spare symbols, whitespace collapsed).

## Gaps

- No route was trained or even loaded in NeMo; route 1's row mapping comes from reading code [S0606][S0610][S0611].
- I did not confirm whether stock v2 emits `^`, `¡` or `¿` on real audio.
- I did not confirm whether NeMo's Canary (multitask) decoding strips special tokens from the text, or how sherpa-onnx handles a literal `\n` piece in `tokens.txt`.
- I did not confirm whether NeMo or Lhotse manifest loading normalises whitespace or `\n` in `text` before tokenising (Inferred no for `\n` inside JSON strings; it matters only for the `\n` piece and the raw double-space variant).
- The NeMo fine-tuning docs (`init_from_nemo_model_exclude` decoder/joint) were read only from a search summary [S0617].
- The `stt_en_fastconformer_*` series was not tested.

## Confidence

High that no shipped NVIDIA NeMo ASR model in these families can emit a newline: I inspected every tokenizer directly. High that a USER_DEFINED `\n` or `<|para|>` piece round-trips, and that v3 and canary carry reserved `<|spltoken*|>` output classes (run directly). Medium on the TDT surgery mapping: it rests on two code paths (EOU warm-start, TDT logit split) and a maintainer's comment, never executed. Low on how well any route will *learn* paragraph placement; that belongs to note 04.
