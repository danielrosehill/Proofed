# 05: Public audio datasets for a merged Proofed training blend

Research agent 05, 2026-10-24. Model: claude-opus-5-5. Sources S05xx are in `notes/05-sources.md`.
"Confirmed" means I read the dataset card, licence file, catalogue page or paper text (or its
search summary where noted). "Inferred" means my reading, not stated by the source.

## Answer

No single public corpus covers Proofed. The usable material splits three ways:

1. **Redistributable now** (permissive licence, audio can go inside a new public HF dataset):
   - **DisfluencySpeech** (Apache-2.0, ~10 h, one acted speaker). It ships graded clean targets for
     fillers, editing terms and false starts [S0501, S0502].
   - **AMI** and **ICSI** meetings (CC BY 4.0, ~100 h and ~70 h). Their verbatim transcripts keep
     "um", "uh" and repeats, so you would derive the clean targets yourself [S0507–S0509].
   - **People's Speech** (CC-BY/CC-BY-SA, 30k h, lowercase spoken form) [S0510].
   - **LibriSpeech / LibriSpeech-PC** and **Libriheavy** (CC BY 4.0 / Apache-2.0, read speech,
     PnC from the source books) [S0505, S0506, S0533].
   - **VoxPopuli-en** (CC0, 543 h) [S0532].
   - **Granary-en** (CC BY 4.0 labels) [S0531].
   - Daniel's own **Hebrish** task subsets (CC BY 4.0, tens of rows each): fillers, repeats,
     self-corrections, asides, currency [S0534].
   - Any TTS audio you generate yourself from permissive text with an Apache-2.0 voice (Kokoro)
     [S0540].
2. **Training-time only** (do not redistribute the audio; publishing a model trained on it is
   possible but restricted or grey):
   - Switchboard/Fisher (LDC non-member licence, non-commercial research only) and the NXT
     disfluency annotations (CC BY-NC-SA 3.0) [S0516–S0519].
   - GigaSpeech (non-commercial research terms; SpeechColab does not own the audio) [S0511].
   - Earnings-21/22 (CC BY-SA 4.0 on the repo, but the audio is claimed only as fair use)
     [S0513, S0514].
   - The CC BY-NC whisper sets: EARS whisper ~3.2 h and Expresso whisper ~2.8 h [S0523, S0524].
   - Common Voice (CC0, but the 2025 Mozilla Data Collective terms ban re-hosting) [S0512].
   - wTIMIT (licence unspecified) [S0520].
3. **Exclude or eval-only:**
   - **SPGISpeech**, the best written-form corpus. Its terms allow "academic research purposes and
     internal use only", with no redistribution and no derivative works [S0504].
   - FluentWhisper's `pradachan/auto-whisper-disfluency` (no licence declared) [S0503].
   - LibriStutter and LLM-Dys (no licence found, and stutter-clinical rather than dictation) [S0527,
     S0528].
   - SEP-28k (audio not distributed; podcast owners keep copyright) [S0530].
   - Unofficial HF re-uploads of Switchboard/Fisher (Inferred: unlicensed redistribution of LDC data).

Real gaps:

- **No public audio corpus covers dictation-style explicit self-corrections, asides to another
  person, or paragraph-break targets.** These need TTS-plus-recording, built from text resources
  such as Disfl-QA (CC BY 4.0, ~12k disfluent questions, >90% corrections or restarts) [S0537], and
  from Daniel's own recordings.
- **Numbers and currency in digits from natural speech** have one strong source, SPGISpeech, and it
  is restricted. The practical route is NeMo-text-processing (Apache-2.0) to verbalise written text,
  then TTS, with the written form as the target [S0538]. Earnings-21/22 serve as a natural numeric
  eval set.
- **Whispered speech:** the only clearly open option is to make it yourself. WhispSynth (MIT,
  ~118 h en+zh) is announced, but its repo was empty on 2026-10-24 [S0520].

## Candidate table

Abbreviations. Tasks: F = fillers, R = repeats, FS = false starts, SC = self-corrections,
A = asides, N = numbers/currency, P = paragraphs, W = whispered, REP = general replay/anti-forgetting.
"Model?" = train a model and publish it openly on HF. "Audio?" = redistribute the audio inside a
new public HF dataset. C = Confirmed, I = Inferred.

| id | hours | type | Proofed tasks | target form | licence | Model? | Audio? |
|---|---|---|---|---|---|---|---|
| `amaai-lab/DisfluencySpeech` | ~10 h (9.4 h per Nyra), 5,000 utts, 1 speaker (C) | acted (re-performed Switchboard) | F, R, FS (partial SC) | clean ready: transcript_b (fillers, editing terms, discourse markers removed) and _c (+ false starts) (C) | Apache-2.0 (C); text is Switchboard content (I: upstream LDC question) | Yes (C licence; I residual risk) | Yes (C licence) |
| `nyralabs/disfluency_speech_english` | 9.4 h, 4,957 utts (C) | acted (same audio) | F, R, FS, eval | paired verbatim / "intended" (C) | Apache-2.0 (C) | Yes | Yes. Use one of the two, not both (same audio) |
| `pradachan/auto-whisper-disfluency` (FluentWhisper) | 23,535 rows (18,628/2,328/2,329/250), hours not stated (C) | synthetic (Kokoro TTS on LibriSpeech text + LARD) [S0205] | F, R, FS, SC | clean `eval_reference` (C field) | **none declared** (C) | No (I: all rights reserved by default) | No. Regenerate instead |
| `kensho/spgispeech` | 5,000 h, ~50k speakers (C) | natural (earnings calls) | N, F/FS removal, PnC | non-verbatim, fully formatted, denormalised numbers (C, S0405) | Kensho terms: "academic research purposes and internal use only", no redistribution, no derivative works (C) | No without Kensho consent (I) | No (C) |
| LibriSpeech-PC (OpenSLR 145) + `openslr/librispeech_asr` | ~874 h train retained of 960 (C) | read audiobooks | P (via source books, I), PnC, REP | PnC restored from books; numbers mostly words (I) | CC BY 4.0 (C) | Yes | Yes |
| Libriheavy (`k2-fsa/libriheavy`) | 50,000 h (C) | read audiobooks (LibriLight) | REP, PnC, possibly P (I) | cased + punctuated book text plus pre_texts context (C) | Apache-2.0 repo (C); audio LibriVox public domain (I) | Yes | Yes (I) |
| `edinburghcstr/ami` (IHM/SDM) | ~100 h, mostly non-native (C) | natural meetings | F, R, FS (natural), A-like crosstalk (I) | Kaldi-normalised uppercase verbatim, fillers kept ("MM-HMM") (C) | CC BY 4.0 (C) | Yes | Yes |
| `JacobLinCool/ami-disfluent` | 20,018 / 2,483 / 1,892 utts (C) | natural (AMI subset) | F, R, FS | verbatim with UM/UH/repeats; clean target must be derived (C) | CC BY 4.0 (C) | Yes | Yes |
| ICSI Meeting Corpus (Edinburgh mirror) | ~70 h (C) | natural meetings | F, R, FS | orthographic verbatim; derive clean (I) | CC BY 4.0 for signals + transcripts (C) | Yes | Yes |
| Switchboard-1 R2 (LDC97S62) + Treebank-3 / NXT (LDC2009T26) | 260 h, 543 speakers, ~2,400 convs (C) | natural telephone, 8 kHz | F, R, FS, SC (repairs) | gold disfluency annotation (650 files) → fluent text by deleting reparanda (C, S0401) | LDC non-member: non-commercial research only (C); NXT CC BY-NC-SA 3.0 (C, search) | Grey: research model, NC (I) | No (C/I) |
| Fisher English Part 1 (LDC2004S13/T19) + Part 2 | 984 h Part 1 (C); ~1,960 h total (I) | natural telephone | F, R, FS | verbatim; silver fluent labels only (S0401) | LDC non-member (C) | Grey (I) | No |
| `MLCommons/peoples_speech` | 30,000+ h (C) | natural (gov, legal, oral history) | REP; some F/R (verbatim-ish "i i i") | lowercase, no PnC, spoken numbers; some training labels are ASR output (C) | CC-BY and CC-BY-SA subsets (C) | Yes (attribution; SA on data) | Yes, with matching licence per subset |
| `speechcolab/gigaspeech` | 10,000 h labelled (C) | natural (podcasts, YouTube, audiobooks) | REP | uppercase spoken form, `<COMMA>` tags, fillers kept (C) | non-commercial research and education; audio not owned by SpeechColab (C) | Grey (NC) (I) | No (C) |
| Common Voice (Mozilla Data Collective) | en ~ thousands of h (I) | read prompts | REP (accents) | read sentence text, written form as prompted (I) | CC0 label but MDC adds "will not re-host or re-share" (C, search) | Yes (I) | No under MDC terms (C, search) |
| Earnings-21 / Earnings-22 (`revdotcom/speech-datasets`) | 39 h / 119 h (C) | natural earnings calls | N eval, F/FS eval | Rev verbatim with fillers and false starts; .nlp with entity/normalisation tags; digits likely (C/I) | CC BY-SA 4.0 badge (C); audio claimed as fair use (C, paper) | Eval only recommended (I) | No (I: Rev doesn't own audio) |
| `facebook/voxpopuli` (en) | 543 h, 1,313 speakers (C) | natural parliament speech | REP, N (raw_text) (I) | normalized_text + raw_text (C) | CC0 (C) | Yes | Yes |
| `nvidia/Granary` (en: LibriLight ~23.5k h + YODAS/YTC) | ~643k h all langs (C) | natural/read, pseudo-labelled | REP (matches Parakeet's own data) | PnC restored by Qwen-2.5-7B (C) | CC BY 4.0 labels; YODAS-Granary CC-BY-3.0; source audio per source (C) | Yes | YODAS part yes (I); others via source |
| wTIMIT | ~26 h whispered + ~26 h normal, 48 speakers, 450 sentences (C, S0410) | read, real whisper | W | TIMIT prompts (C) | unspecified, "N.A." (C, S0520) | Grey (I) | No (I) |
| CHAINS (UCD / LDC2008S09) | whisper subset 2.55 h, 36 speakers (C, S0520) | read, real whisper | W | read text (C) | conflicting: original UCD CC "free commercial and non-commercial, share-alike" vs LDC non-member vs BY-ND (C, search) | Probably yes (I) | Unclear. Not until UCD terms are read |
| EARS (`facebookresearch/ears_dataset`) | 100 h total, 107 speakers; whisper style 3.22 h (C) | read, real whisper | W | read text (C) | CC BY-NC 4.0 (C) | NC model only (I) | NC dataset only (I) |
| Expresso (`ylacombe/expresso`) | 40 h, 4 speakers; whisper 2.8 h (C) | read + improvised | W | read transcripts (C) | CC BY-NC 4.0 (C) | NC model only (I) | NC only (I) |
| WhispSynth (arXiv 2603.14853) | ~118 h en+zh, 479 speakers (C) | synthetic whisper (DDSP + TTS) | W | source text (I) | MIT (C, paper table) | Yes when released (I) | Yes when released (I); repo empty 2026-10-24 (C) |
| `neilshahnms/Claris-Whispered-English` | 3,365 files (C) | unknown, nonnative "effort" whisper (I) | W | per-file .txt (C) | none declared (C) | No (I) | No |
| `Malfaro43/whisperedAudio-benchmark` | small (I) | EARS + Expresso, normal vs whisper, background-speaker mixes (C) | W eval, A-ish eval | text (C) | none on card; inherits NC (I) | eval only | No |
| LibriStutter | ~20 h, 50 speakers vs 100 h/10 speakers (sources disagree) (C, search) | synthetic stutters spliced into LibriSpeech | R (word reps) | labels by type, fluent text (I) | none found (C); HF mirror `stillerman/libristutter-4.7k` no licence (C) | No (I) | No |
| LLM-Dys (`tong0/LLM_Dys`) | ~12,790 h full, generate-it-yourself; samples on Drive (C) | synthetic (VITS/E2-TTS, VCTK voices) | R (word-level REP/INS only) | phoneme/word dysfluency labels (C) | none declared (C) | Code-regenerate only (I) | No |
| Libri-Dys / Libri-Co-Dys (SSDM) | not read | synthetic (StyleTTS2 on LibriTTS) | R | phoneme labels (C, search) | not stated (C, search) | Unclear | Unclear |
| SEP-28k / FluencyBank | 28k 3-s clips (I) | natural stuttering podcasts | none for Proofed (clinical) | event labels only (C) | Apple sample-code licence on labels; audio not distributed, copyright with podcast owners (C) | Exclude | No |
| Hebrish task subsets (Daniel's) | tens of rows each, e.g. Disfluent-Read 51/4 (C) | acted, 1 speaker (Irish accent, phone mic) | F, R, SC, A, N (currency) | clean `text` + `spoken_text`/`verbatim_text` (C) | CC BY 4.0 (C) | Yes | Yes (his own) |
| Rev multireferences (Earnings-22 subset10, Rev16) | not read | natural | F/FS eval: verbatim vs non-verbatim pairs (C, README) | two Rev transcript styles side by side (C) | repo CC BY-SA (I) | eval only | No (I) |

## Text-only resources for TTS synthesis

| resource | what | licence | note |
|---|---|---|---|
| LARD (`tatianapassali/artificial-disfluency-generation`) | injects repetitions, restarts, replacements into fluent text | **CC BY-NC-SA 4.0** (C, LICENSE.txt) | FluentWhisper used it [S0205]. NC/SA: reimplement the rules rather than ship its output in a CC BY dataset (I) |
| `stillerman/fdt-disfluency-synthetic` | DailyDialog + Persona-Chat with injected fillers, repeats, false starts, substitution repairs; word-level KEEP/DELETE tags | CC BY-NC-SA 4.0 (C) | good template for SC ("the bus, I mean, the train") |
| Disfl-QA (`google-research-datasets/disfl-qa`) | ~12k disfluent questions, >90% corrections/restarts | **CC BY 4.0** (C) | best permissive text source for self-correction patterns |
| NeMo-text-processing | WFST TN/ITN grammars, spoken ↔ written | Apache-2.0 (C) | write targets in digits/currency, verbalise with TN, TTS the spoken form |
| Sproat & Jaitly TN data (Kaggle) | 1.1B-word Wikipedia token-level before/after pairs | not confirmed (C: README points to Kaggle only) | check the Kaggle licence before use |
| Kokoro-82M | TTS voices (54 in FluentWhisper) | Apache-2.0 (C) | output audio redistributable (I) |
| VCTK | 110 speakers, voice-cloning references | CC BY 4.0 (C) | for zero-shot TTS speaker variety |

## Evidence

**Disfluency (acted/synthetic)**

- Confirmed. DisfluencySpeech: HF card `license: apache-2.0`; splits 4,500/250/250. Its transcript tiers: `transcript_a` keeps everything except non-speech events, `_b` removes filled pauses, explicit editing terms and discourse markers ("coordinating conjunctions and asides are left in"), `_c` also removes false starts [S0501].
- Confirmed. The Nyra re-format pairs `verbatim_transcript` with `intended_transcript`, e.g. "So [UH] we do have a very low tax rate, but you know, …" becomes "We do have a very low tax rate, …"; 9.4 h [S0502].
- Inferred. The audio is original recordings, but the words are Switchboard transcripts (LDC). Apache-2.0 on the card is the authors' grant. Whether they could grant it over Switchboard text is untested.
- Confirmed. `pradachan/auto-whisper-disfluency` has no `license` field. It stores Whisper `input_features` and `labels` token ids plus `eval_reference` and audio, 82.8 GB, created 2026-06-09 [S0503]. Inferred: the 250-row `probe` split is the DisfluencySpeech test set the README scores on [S0205].

**Disfluency (natural)**

- Confirmed. AMI on HF follows the Kaldi s5 recipe ("text is normalized"). Rows read like "MM-HMM" and "THEN IT'S PROBABLY QUICKER TO JUST" [S0507]. The AMI licence page states CC BY 4.0 [S0507].
- Confirmed. `JacobLinCool/ami-disfluent` rows read like "SO SO UM I WILL PROBABLY SEND UM JUST ONE FILE…", licence cc-by-4.0 [S0508]. Inferred: it is an AMI subset filtered for disfluent utterances. Its card has no description.
- Confirmed. ICSI: "All of the signals and transcription, and some of the annotations, have been released publicly under the Creative Commons Attribution 4.0 International Licence" [S0509].
- Confirmed. Switchboard LDC97S62: 260 h, 543 speakers, "LDC User Agreement for Non-Members" [S0516]. The LDC licensing page says non-member licensees "may use LDC data for noncommercial linguistic research and education only" and "cannot use LDC data to develop or test products for commercialization". It says nothing about trained models [S0518]. The NXT annotations are CC BY-NC-SA 3.0 for non-members (search summary) [S0519].
- Confirmed. Earnings-21 transcripts are Rev "verbatim", including "filler words, false starts, grammatical errors" [S0514].

**Written form / numbers**

- Confirmed. SPGISpeech gated terms (HF page): "provided for academic research purposes and internal use only"; "must not publish, display, transfer or redistribute the Content or any portions or derivative versions"; "shall not … create derivative works" [S0504].
- Confirmed. Earnings-22 HF rows are cased and punctuated ("The next question is come from Mohammed `<unk>` at Goldman Sachs.") [S0515]. Rev repo READMEs carry a CC BY-SA 4.0 badge [S0513].
- Confirmed. GigaSpeech `text_tn` is spoken-form uppercase with tags ("FOUR O'CLOCK TOMORROW AFTERNOON `<COMMA>` …"), and the README says conversational fillers are present [S0511].
- Confirmed. People's Speech sample: "…and so i i i think that's wonderful…", lowercase with no digits. The card warns some training transcripts "are likely to be the output of automatic speech recognition systems" [S0510].

**Whisper**

- Confirmed. WhispSynth Table 1 lists English whisper corpora with licence: wTIMIT 29.44 h, 48 spk, N.A.; CHAINs subset 2.55 h, 36 spk, BY-ND; EARs subset 3.22 h, 107 spk, BY-NC; Expresso subset 1.32 h, 4 spk, BY-NC; wSPIRE 18 h, 88 spk, N.A. WhispSynth itself is ≈118 h, MIT [S0520]. GitHub `tan90xx/cosywhisper` returned an empty-repository error from the contents API (C, 2026-10-24).
- Confirmed. EARS README: "The code and dataset are released under CC-NC 4.0" [S0523]. Expresso card: CC BY-NC 4.0, whisper 79 min read + 86 min improvised [S0524].
- Disagreement. CHAINS: the original 2006 announcement describes a CC licence permitting commercial and non-commercial research, with share-alike redistribution. LDC2008S09 lists the LDC non-member agreement. WhispSynth says BY-ND [S0521]. chains.ucd.ie refused connections (ECONNREFUSED) on 2026-10-24.

**Self-corrections, asides, device-directed speech, paragraphs**

- Confirmed (negative search). No public English audio corpus labelled for addressee or device-directedness turned up. The SAS paper's 600 h corpus is proprietary, and the 2310.15261 multimodal 530 h set has no stated release [S0541]. Google's primary/non-primary speaker work [S0210] used internal data.
- Confirmed. Hebrish subsets: `Self-Correction-Sentences` (`spoken_text` vs corrected `text`), `Spoken-Aside-Sentences` (`aside_text`, `aside_position`), `Currency-Sentences` (`50 NIS`, `$1.50`), `Disfluent-Read-Sentences` (51 train / 4 test), `Repeated-Word-Sentences`; all CC BY 4.0 [S0534].
- Inferred. Paragraph targets: no ASR corpus ships paragraph breaks. Libriheavy and LibriSpeech-PC align to the original book text, so paragraph boundaries could be recovered from the Gutenberg source for long-form segments. That is read prose, not dictation.

## Disagreements and gaps

- **LibriStutter size**: 20 h/50 speakers (2021 review) vs 100 h/10 speakers (diagnostics table) [S0527]. Licence not found. Excluded either way.
- **CHAINS licence**: three different statements (see Evidence). Treat as unclear until the UCD site is reachable.
- **DisfluencySpeech upstream rights**: Apache-2.0 card over Switchboard-derived text. Not resolved by any source.
- **Model trained on NC / LDC / GigaSpeech data, published openly**: no source addresses model weights. NVIDIA and others publish models trained on Fisher/Switchboard/GigaSpeech (Inferred from general practice, not checked in this pass). Treat as a policy decision for Daniel, and record it in `docs/development-plan.md`.
- **Common Voice**: CC0 on the data, but MDC's "will not re-host or re-share" clause comes from a search summary. The MDC terms of service were not read.
- **Not read**: Rev multireferences subfolders (git submodules or LFS, 404 via raw), Libri-Dys licence, Google TN Kaggle licence, `hhoangphuoc/ami-disfluency` and `ltrciiith/bhasik_Indian_English_disfluency_corpus` (both gated; tags say apache-2.0 and cc-by-nd-4.0, and the latter is Indian-English read disfluency, worth a look), hours for `pradachan` and Claris.
- **Natural self-corrections**: Switchboard repairs are conversational ("I went, I mean, we went"), not dictation edit commands. No source tests transfer to "Tuesday, no, Wednesday".

## Confidence

- High: licences for DisfluencySpeech, AMI, ICSI, LibriSpeech(-PC), VoxPopuli, People's Speech, Expresso, EARS, LARD, Disfl-QA, NeMo-TP, SPGISpeech terms, GigaSpeech terms, LDC non-member scope.
- Medium: Earnings audio redistributability; Granary per-source audio rights; Common Voice MDC clause; WhispSynth release status.
- Low: CHAINS licence; wTIMIT access terms; whether publishing a model trained on LDC/NC data is allowed; hours of the synthetic sets.
