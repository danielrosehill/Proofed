# Overview

## What it is

Proofed is a speech-recognition model that writes clean text directly. It starts from a stock,
phone-sized ASR model (NVIDIA Parakeet TDT 0.6B v2) and fine-tunes it only on post-processing
tasks, so the transcript comes out already proofed, in a single forward pass, with no LLM step
afterwards (12:34, 19:27).

## The problem

The basic contract of ASR is narrow: render sounds as words. Everything beyond that
(punctuation, sentence boundaries, removing "um") is an addition someone chose to build (01:58).
Today those additions come from a second model: a general LLM with instructions, or a dedicated
text post-processor such as Superwhisper's S1-mini (05:53). Audio-native multimodal models can
do any transformation but are far too big for a phone and unusable for dictation even on a
desktop GPU with 12 GB of VRAM (04:55). Regex cleanup after inference works for some of it; the
aim is to fix it at source (16:03).

## The objective

Find out to what extent a stock ASR model can be fine-tuned to bake in a set of post-processing
transformations (12:34). Same result for the user as ASR plus cleanup, from one small model.
Published on Hugging Face as an experimental prototype (20:25).

## Scope

In: transformations that are right for every user every time (15:17):

- removing fillers ("um", "uh"), repeated words, false starts and unfinished sentences
- numbers written as digits
- applying self-corrections and editing instructions ("Haifa, I mean Tiberias"; "scratch that")
- paragraph breaks
- leaving out speech aimed at someone else in the room (the experimental task)
- whispered speech, so dictation works when others are asleep (21:03)

Out: any domain vocabulary, including code-switching and tech terms (23:54), and anything
use-specific, such as formatting every output as an email (15:17).

## Approach

Take Daniel's existing "kitchen sink" recordings, already stratified by task type, keep only
the post-processing tasks, and tier them as core, ambitious and experimental (20:25). Define
each task narrowly so each can be evaluated alone, since some may not train well at small scale
(17:54). Later, test a layered approach: fine-tune Proofed again with a domain-vocabulary layer
and compare with a one-shot model (13:55).

## Background

Proofed grows out of Daniel's Parakeet and Nemotron fine-tunes for English-Hebrew
code-switching and tech vocabulary (06:11). While adding number rendering and disfluency
removal to that dataset, he noticed the work had crossed from domain-vocabulary fine-tuning
into post-processing (12:34). An earlier finding encouraged him: combining two domain
fine-tunes in one "kitchen sink" dataset did not degrade either (10:58).

## Open questions from the note

- Does Parakeet have a token for a blank line? If not, paragraphs need a different base model
  or a decoder change (16:19).
- Can one model hold both an ASR pass and a post-processing pass internally (19:27)?
- Can the model be English-only (23:39)? With domain vocabulary out of scope, probably yes.
- Which existing attempts at a unified ASR and post-processing model are there (22:00)?
