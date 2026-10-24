# 04 Feasibility: can CTC, transducer and AED models learn deletions and reformatting from targets? How much data? Acted vs natural disfluencies, whispered speech, forgetting

## Answer

Yes, with caveats. Every major E2E family has been trained on fluent or formatted targets and has learned it. Jamshid Lou & Johnson trained CTC, LSTM-AED and Transformer models on Switchboard/Fisher fluent transcripts. All three learned to remove disfluencies, though each scored slightly worse than an ASR-plus-detector pipeline. The Transformer was best on corrections and CTC was worst [S0401]. An Amazon RNN-T learned to drop partial words and repetitions from 47 h of disfluent data mixed into ~2,300 h. Even a tenth of that data helped [S0403]. Sony/CMU's streaming end-to-end removal model nearly matched joint detection models on DR-WER [S0404]. A HuBERT-CTC end-to-end fluent model beat the two-stage baseline [S0402].

For formatting, Conformer-CTC trained on SPGISpeech (non-verbatim, digits, PnC) came within 0.3 WER points of an attention model [S0405]. So a monotonic CTC decoder plus a full-context encoder does not block written-form output, apart from reorderings such as "ten dollars" → "$10" (inferred). Pure deletion fits CTC and transducers naturally, because they emit blanks over frames. Self-corrections and reorderings are where label-context or attention decoders do better [S0401, S0408]. Parakeet TDT is a transducer, which puts it between the two (inferred).

Data needs are modest for deletion, roughly tens of hours with replay [S0403]. Written-domain numerics in an RNN-T needed large-scale TTS augmentation (370 h, 10% of batches) [S0406]. No study measures transfer from acted to natural disfluencies. That is the largest gap for Proofed: the one acted single-speaker corpus flags acting as a limitation but never tests ASR [S0407]. Whispered speech is learnable from ~11–26 h [S0409, S0410]. Forgetting is real: full fine-tunes on narrow data can wreck the original domain, while replay and adapters contain it [S0411, S0412].

## Evidence

**(a) Architectures learn deletion and reformatting**

- Confirmed. CTC (Jasper), LSTM Seq2Seq and Transformer, trained end-to-end on fluent Switchboard targets (260 h gold plus ~2,000 h Fisher with silver labels), all learned removal. Their SWBD test DER was 26.2 / 25.6 / 24.0 end-to-end against 23.3 / 22.6 / 22.2 for the pipeline ("slightly worse"). On corrections, end-to-end DER was 33.5 for CTC, 29.5 for Seq2Seq and 25.8 for the Transformer. End-to-end models sometimes fail to remove repetitions [S0401].
- Confirmed. A HuBERT-large Conformer-CTC fine-tuned to fluent Switchboard targets reached 12.2% WER, against 13.1% for the two-stage system. A wav2vec2-initialised model did much worse, so the choice of pretraining matters [S0402].
- Confirmed. A transducer case: an Amazon RNN-T (~2,300 h ordinary plus 47 h disfluent, partial words deleted from targets) got 19–21% relative WER reduction on disfluent test and 13–14% on a stutter test, with no change on ordinary test. The baseline's failure mode was insertions [S0403].
- Confirmed. A streaming Transformer AED with CTC 0.3, used as an end-to-end fluent-only model, scored DR-WER 21.7 on Switchboard. Joint multi-task scored 21.3 and the BERT pipeline 22.6. Its drawback is that deletions cannot be undone and the verbatim text is lost [S0404].
- Confirmed. SPGISpeech is 5,000 h of non-verbatim, fully formatted transcripts: fillers removed, "ninety two year" written as "92-year". Conformer AED reached 5.7% orthographic WER and Conformer-CTC 6.0%, and about 1/2 to 2/3 of the error is formatting. Currency slices were excluded [S0405].
- Confirmed. Written-domain RNN-T on numerics: adding TTS numeric data gave 65% relative WER improvement on short numbers. The best results came from spoken-domain training plus a neural denormer, not from direct written-domain output [S0406].
- Confirmed (ITN tagger, not ASR). NVIDIA's Thutmose ITN tagger assumes monotonic alignment, drops non-monotonic pairs and encodes the few common reorderings ("$" moved before the number) with special tags [S0408]. Inferred: CTC and transducer outputs face the same reordering constraint at the output layer, while the full-context encoder can still carry future context.
- Confirmed. Parakeet TDT 0.6B v2: ~120k h (10k human, 110k pseudo-labelled), PnC transcripts, "robust performance on spoken numbers". The card says nothing about verbatim/disfluency policy or ITN [S0413].

**(b) How much data**

- Confirmed. Amazon RNN-T, by fraction of the 47 h disfluent set (with 2,300 h replay): 1/10 (~4.7 h) gave 8.7% relative WERR, 1/4 gave 13.9%, 1/2 gave 18.3%, the full set 22.5%. Ordinary test changed by +0.3 to 0.0 [S0403].
- Confirmed. TTS numeric data: 370 h synthesized, sampled as 10% of each batch, on a 43M-utterance base [S0406].
- Inferred. The deletion tasks look feasible at the 5–50 h scale with replay. Reformatting with a long tail (numbers, currency) probably needs synthetic volume beyond what one speaker can record.

**(c) Acted vs natural, synthetic injection**

- Confirmed. DisfluencySpeech has one speaker re-voicing ~10 h of Switchboard utterances with marked disfluencies. Three transcript tiers remove non-speech events, fillers and false starts. The authors call it "a form of acting rather than spontaneous speech production" that "might affect" models. It was used for TTS only, with no ASR transfer test [S0407].
- Confirmed (old, HMM era). In a read/spontaneous comparison on ATIS, the gap came entirely from disfluent utterances: 56% WER disfluent vs 32% fluent spontaneous vs 33% read [S0414].
- Confirmed. The Amazon stutter test set was 11 speakers reading prompts. Training on natural field disfluencies improved it, so the transfer shown there runs from natural to read. The reverse direction is untested [S0403].
- Confirmed (search). No paper found that trains on acted or injected disfluencies and tests on natural ones for fluent-output ASR. Searches on LibriStutter, "acted vs spontaneous" and "synthetic disfluent TTS" found evaluation-only synthetic sets (e.g. arXiv 2405.06150) and stuttering-ASR work, not this. This is a fact about the search.

**(d) Whispered speech**

- Confirmed. wTIMIT has parallel whispered and normal speech, ~26 h each, 48 speakers, 450 TIMIT sentences. The original split overlaps sentences between train and test, so it needs a re-split [S0410].
- Confirmed. An E2E BLSTM trained on small normal data scored 59.5% whispered PER vs 29.7% normal. Frequency-weighted SpecAugment brought whispered PER to 48.3%. The paper reports 19.8% relative PER and 44.4% relative CER reduction overall, and concludes that "a relatively small set of whispered speech may suffice" after normal or pseudo-whisper pretraining [S0410].
- Confirmed. Stock Whisper large-v2: 16.69% WER normal vs 18.80% whispered. WavLM fine-tuned on 17 h of wTIMIT (11 h whispered) plus 3 h of CHAINS normal (36 Irish speakers) reached 9.22% on whispered speech [S0409].
- Inferred. Modern large-data models show a small normal/whisper gap (~2 points for Whisper). Parakeet's gap is unmeasured and should be checked before deciding how much to record.

**(e) Forgetting and mitigation**

- Confirmed. NVIDIA (NeMo, Conformer transducer, LibriSpeech base at 5.12% test-other) found that a full fine-tune on Speech Commands sent test-other to 51.08%, and even the constrained fine-tune reached 29.78%. Encoder adapters held it at 6.24% and joint adapters at 5.79%. On UK/Irish dialects, a full fine-tune went 5.11 → 7.68 while joint adapters stayed at 5.40. Decoder/joint adapters adapt the "language" side with little loss [S0411].
- Confirmed. Continual-learning methods for E2E ASR closed more than 40% of the fine-tune-to-joint-training gap using 0.6% of the original data. Task-specific adapters with regularised shared parameters avoid stored data entirely [S0412].
- Confirmed. In the Amazon RNN-T, mixing the disfluent data into the full ordinary set (joint training / replay) gave no ordinary-test regression [S0403].
- Inferred. For Proofed, replay (LibriSpeech-PC, which the plan already includes) plus a low learning rate is the first line of defence, with decoder/joint adapters as the fallback. Because the deletion/format tasks are output-side, decoder/joint adapters may suffice for them, but whispered speech is acoustic and needs encoder capacity.

**Owner's datasets (local cache, live API)**

- Confirmed. `danielrosehill/Parakeet-Dictation-Cleanup-Pairs` (cached snapshot cae5e1cb, README only) and `-Eval` (sha 9cda6a24, lastModified 2026-10-16, files `.gitattributes` and `README.md`) are both placeholders with no data. Pairs is planned as ~400 synthetic plus 50 no-op text pairs (raw Parakeet-style text → light-cleaned text) for the S1-mini LoRA text post-processor. Eval is planned as 40 spontaneous single-speaker dictations, transcribed by stock Parakeet v2 and hand-cleaned. Pairs now returns HTTP 401 unauthenticated, so it is private or removed. Both are text-to-text with no audio, so neither can train Proofed. Eval, once recorded, would be a natural-disfluency test set for Proofed, which directly addresses gap (c) [S0415].

## Disagreements and gaps

- **Fluent-only vs tagged output:** Futami et al. argue against fluent-only end-to-end output because verbatim text is unrecoverable and over-deletion is permanent [S0404]. Mendelev found a tag for partial words better than deleting them (22.4 vs 21.0 WERR) [S0403]. A Proofed variant that emits a removable tag is worth an ablation.
- **Direct written form vs spoken plus denorm:** Peyser found spoken-domain training plus a denormer beat direct written-domain RNN-T on numerics [S0406]. SPGISpeech shows direct formatted output works [S0405]. They conflict on the hard tail (long numbers, currency).
- **Not found:** acted→natural transfer for ASR, any TDT-specific disfluency study, data-size curves for self-corrections or asides, and normal/whisper WER for Parakeet or any transducer.
- Jamshid Lou's architectures are from 2020 and small. Their CTC < AED ranking on corrections may not hold for a 0.6B transducer with a strong prediction network.
- The wTIMIT results have speaker overlap [S0410]. The WavLM paper's test set composition is not fully clear from the text [S0409].

## Confidence

Medium. That E2E models learn deletion and formatting from targets is supported by four or more independent papers across CTC, AED and RNN-T (high). The data-size estimates rest on one industrial RNN-T paper and one TTS-numerics paper, extrapolated to a single speaker (medium-low). Acted-vs-natural transfer has no direct evidence (low), and it is the claim Proofed's design leans on most.
