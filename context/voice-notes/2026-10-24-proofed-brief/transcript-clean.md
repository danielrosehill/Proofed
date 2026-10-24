# Proofed: project brief (cleaned transcript)

Voice note recorded 2026-10-24 by Daniel Rosehill (24:40). Cleaned from
[`transcript-raw.txt`](transcript-raw.txt) (AssemblyAI universal, unedited). Fillers, false starts
and repetitions are removed and obvious mishearings fixed (listed under Corrections at the end).
The words, order and points are the speaker's. Timestamps are `mm:ss` into
[`voice-note.mp3`](voice-note.mp3).

## Background: voice as the way in (00:00)

This voice note was recorded on 24 October 2026. It's context for a project that builds on what
I've been working on for the past week, and really on a couple of preceding years of using voice
dictation all day, every day, across different devices, as my preferred way of entering text into
a computer: trying out my own fine-tunes, trying out different apps, seeing what works and what
breaks, and seeing how the available models have changed and are changing. That goes from the
amazing Whisper, an outstanding model, to the directions people have been taking for that final
push in ASR and voice dictation utility: from "this is very impressive, very accurate, but I still
need to put effort into this before I send it [unclear: "to the ladder"]" to something you don't
have to touch.

I think this is a very exciting area right now, because what I think is essential is the action
taken on the back of voice notes. If it stops at the transcription, a huge amount of the potential
utility of dictation and voice as an interface for getting things done is lost, maybe even most of
it. With easy-to-use connectors for email, calendar, contacts, you name it, available in apps like
ChatGPT, the complication and bugginess of MCP is no longer something that most people, or anyone
for that matter, has to wade through just to get these things done.

## The basic contract of ASR (01:58)

The basic contract of ASR is extremely limited in scope: taking what people say, phonetics, sounds,
and rendering that into words. What's underappreciated is that everything bolted onto that very
bare level of scope is a decision and an addition. It's work that someone undertook to go beyond
the basic contract. Someone said: hang on, we need sentences, we need punctuation, we need to detect
sentence boundaries, and so on.

## From post-processing prompts to audio-native models (02:52)

There have been a couple of changes I've been looking at for my own tools. Over the course of
building these out, I developed a bank of what I call text transformation prompts. Initially that
was the classic post-processing step for ASR plus LLM: the second step gets the ASR text back and
says, restructure this text as an email, a formal email, a casual email, a shopping list, a task
list. When I was drafting these, I realised the number of permutations was enormous, since you can
build them one on top of the other.

I did create an app for my own use that did exactly that. Under the hood it was prompt
concatenation. I had boxes for "make this more formal", "less formal", "polite business email";
some of them instructing the output format, some of them instructing the style. When the multimodal
models started appearing, the Gemini models that could process audio tokens and take a system
prompt, it struck me that this is gigantic for dictation, because I can create basically any
instruction, even "translate this into formal Hebrew". In other words, I can mop up even
translation tools.

My thought at that point was: what's the point of having ASR and cleanup models if you have these?
I've been thinking about that a lot since. Is that the future?

## Why that isn't the whole answer: size (04:55)

One of the negatives of that line of prediction is that, to the best of my knowledge, there aren't
very many small models that can do audio. On-device ASR has quickly matured, through excellent
models from Moonshine and NVIDIA. The multimodal audio models are still just too big for most
people. I don't even know what phone could run them; they're not practical for the most part. Even
on the desktop with 12 GB of VRAM, I haven't found one that runs in any way that's usable for
dictation, which is a separate concern.

I'm tracing the full evolution of the field as I see it, to explain where I see this idea fitting
in.

## Dedicated post-processing models (05:53)

Then there's a model called S1-mini by Superwhisper, which is a dedicated post-processing model. In
other words, instead of taking a general-purpose LLM and adding your own instructions to an
instruction-tuned model, it's something actually trained to expect ASR text, with a defined system
prompt honed on this specific task. That's a good idea.

## What I've been fine-tuning (06:11)

In the last couple of weeks I've been fine-tuning Parakeet and Nemotron on a code-switching task. I
live in Israel, and when I began using Whisper very ardently, one of the first things that jumped
out at me was that it was going to fail on all the Hebrewisms, as it were: "I'm going to the shuk
and going to Bituach Leumi after this." So my first fine-tune for Whisper was code-switching.

Then, again just because I'm using these things day in, day out, I noticed that tech vocab and new
brand names are a problem. That makes sense, because they're still fairly niche, and beyond being
niche they change very fast. I might begin talking about S1-mini from Superwhisper a lot, or ROCm,
as in ROCm for the AMD stuff.

For this round of fine-tuning I moved from Whisper to Parakeet, because I found that stock Parakeet
was far, far superior on my phone, which is what I've happened to be using for the past few months
rather than my desktop.

## One model across devices (07:52)

I'm personally in favour of using one model across different devices. You might have a model that's
the best choice for GPU acceleration on one machine. But this is my philosophy: if you're working
towards extremely high accuracy and exactly the type of output you want, there's more to be said
for knowing exactly the limits, the pros and the strengths of one model than for using a couple of
them. It gets hard, it gets confusing: which model doesn't handle the disfluencies, which does. So
I'm going all in on Parakeet and the NVIDIA models generally.

## How the task list grew (08:37)

For the fine-tune I started training last week, after I moved over, I didn't even consider using my
previous Whisper dataset, because I thought it was a bit old. So I started recording new
code-switched sentences. And while doing that I thought: if I'm going to sit down to do this... It's
worth it when you start seeing the evals move in the right direction, but it's kind of weird to be
sitting in bed reading these random sentences, and it's tedious.

So I thought, if I'm going through this process again, I may as well think of everything I want:
everything that's wrong with this model as I see it, or everything it does poorly. After
code-switching I added recognition for tech vocab, and then I asked what else would be nice for
this model to do, and began drawing up a list as I transcribed with it. These are things I've just
observed; it's not as strong as Whisper in some ways, maybe because it's a smaller model.

One was number rendering. The classic example: "the year is 2026" could be rendered in spelled-out
numbers, but no one writes a date that way, so it should be 2026 in numerals. Then just disfluency
removal: Parakeet very faithfully renders all the ums and ahs.

As I built up these tasks and datasets and trained on them, I realised I had crossed a juncture, or
a division.

## Combining domain fine-tunes didn't hurt (10:58)

My first experiment was to see whether two sets of fine-tuned vocab would degrade each other's
accuracy, which my evals said they didn't. That was very encouraging, because I try to balance my
desire to open source with the time I have to dedicate to a project. In an ideal world I'd maintain
a tech one, because that could be useful for a lot of people, and Parakeet fine-tuned for
code-switching plus tech for my own use. But in the real world, to the extent that I can afford to
spend time on my open-source projects, it's biased towards the tools I'm actually going to use and
benefit from most myself, which is the all-in-one.

I called it the kitchen-sink training dataset: tech plus Hebrew. That was a surprising result. I
assumed, and I'm not sure why, it just seemed common sense, that if you distribute the weights
you're increasing recognition for across tech vocab and Hebrew, the jack of all trades is going to
be master of none. And even if the eval had shown a 5% decrease in accuracy on both domain
datasets, I would have said that's a good trade-off, because the most useful model for me is the
one that does code-switching and tech without overfitting on either niche domain.

## The idea: Proofed (12:34)

Coming back to the thought I was finishing earlier: when I began expanding the dataset, because who
wants to do this multiple times, I realised that the first two datasets are training on the classic
ASR fine-tuning task of domain vocabulary. The second group, which I'm building out now, crossed
over once I added disfluency removal and number rendition: we're now integrating a post-processing
step into the model instead of doing it separately.

So what I want to train with this model, which I'm calling Proofed (I couldn't think of a better
name right now, but it's a decent one), is an ASR model trained only on a base ASR model plus
cleanup. I'm not sure this is actually the fine-tuning methodology that makes sense, but it would
nevertheless be very worthwhile, just to see how well this works in isolation. This is the
isolated model, to test to what extent a stock ASR model can be fine-tuned to bake in a series of
post-processing transformations.

## Layering (13:55)

Then, can we take a layering approach and fine-tune the fine-tune to add the second layer of needs,
like the tech vocab? In other words, you take a model like Parakeet; Proofed is Parakeet plus
fine-tuning for post-processing and formatting; and then you build on top of it. It's a higher
workload from a training perspective, since the easiest way to get to any result is one dataset and
one training run, but it could be a very elegant way to do it.

What I want to do for this one, maybe not for my own use but as an experiment, is throw in as many
of the datasets I've been creating specifically for post-processing as I can. I've deliberately
created a mixture, starting with what I'd call the core ones.

## Where the line is (15:06)

There is a limit, a definite line you don't want to cross. If you're going to bake in deterministic
post-processing, you want it to stay general. (And maybe this is where it could pair with a second
pass.) If, for argument's sake, Proofed were trained to format every output as an email, it would be
useless for anyone who wants it for any other purpose. So the set of post-processing
transformations I'm trying to train on are the ones that make sense for everyone. For my use, I
can't think of a single time I've wanted disfluencies like "um" to appear in a transcript. They're
always unwanted, and that's the classic example of the type of thing I want to bake in here.

Those can be handled at the regex level, meaning post-inference. But the idea here is to make that
unnecessary in the first place, to tackle it at source.

## Paragraph inference (16:19)

Another one I feel strongly about is paragraph inference. I find it's very overlooked, in ASR and
even in post-processing: sentence boundary detection is one thing, but adding paragraph breaks is
another. This is a bit of a problem with Parakeet, because if I'm not mistaken the core model
doesn't have a token for a blank line, which makes training for paragraph inference impossible in
the raw model. So I might need to find a different base model, or add that into the decoder.

## The experimental one: non-directed speech (17:04)

Those are the standard ones on my list. The experimental one, as I call it, is non-directed speech,
which is kind of a reasoning task. For example, you're recording and you say: "So the API documents
are here now... yeah, just leave it by the door." The idea is to infer that "just leave it by the
door" wasn't meant for the transcription: the speaker was talking to somebody else, and it
shouldn't be included. That's an edge case: if it worked, it might be very useful; if it didn't,
not so much.

## Keep tasks narrow and modular (17:54)

I think it makes sense to define these tasks very narrowly and modularly, because in seeing what
actually works, it could be that certain tasks just can't be trained reliably at a small scale and
some can.

The other common disfluency is repeated words: "I'm putting the orange, orange juice in the
fridge." You repeat words when you're distracted. False starts are another: "And going to the... I'm
going to the cinema." Another example is sentences you never finish: "I think I had an idea... I
want to go to the movies now."

Another is inferred editing instructions. A self-correction: "I need to go to Haifa today. I mean
Tiberias." Whereas an editing instruction might be: "I need to go to Jerusalem. Scratch that, Tel
Aviv." These have been handled differently by different approaches and models in ASR and speech,
and the question is really scope: which of these makes sense?

## The objective (19:27)

What I'd like for my own use: rather than a huge multimodal model capable of processing everything,
and therefore very heavy, take a pure ASR model and adapt it so that in a single forward pass it
not only does the phonetic rendering but also reasons and produces text that follows these
instructions. I don't know if this is architecturally possible: one model that encapsulates two
passes, the first pure ASR, the second post-processing. Same result for the user; internally, two
different processes going on. I'm not sure, but that's the idea.

## The prototype (20:25)

For the prototype on Hugging Face: take my training dataset, which is basically the kitchen sink,
everything I've recorded, stratified by all these types. Isolate only the post-processing tasks,
and within those, isolate the ones I've described here as core, ambitious and experimental.

## Whispered sentences (21:03)

There's one in there I've called whispered sentences, and it's an important one for me. Here's the
use case. Since I became a voice-first user, sometimes I'm in bed and my wife is saying "stop
talking to your phone". I remember asking ChatGPT way back whether Whisper could transcribe
whispered speech. Then I just tried it, and it was quite good. I thought, this is actually super
useful: maybe you have to write something discreetly, maybe you're trying not to wake the kids, or
the wife, or the husband. Why not include an actual small corpus of whispered sentences in the
training subset, so there's explicit training data on whispered speech? That's why I created a
whispered dataset, and I'd very much like to have it included for that reason.

## Data and prior art (22:00)

The datasets I have at the moment might need to be beefed up for all of these tasks, paragraph
inference and the others. I'd also like a survey of prior art, and parallel art as well: ASR models
that support a paragraph break as a token, and anything else. I'd be surprised if there weren't
other attempts out there to create a unified ASR post-processing model. (I don't even like that
name very much, but it's accurate to the lineage, and that's the objective.)

## Stages (22:47)

There are a few stages. First, setting up the repository with a cleaned-up version of this
transcript and a spec from it. Then research on existing attempts, research on feasibility, and
research on candidate ASR models that might be better fits than Parakeet, although I'd be reluctant
not to use it, because I've become very fond of it.

## Open question: English-only? (23:39)

The other question I have is within the Parakeet models. This would be an English model. If I bake
in my code-switching support, I presume it can still be a model that explicitly supports only
English? That's an open question: whether it has to be multilingual if code-switching is part of
it.

## Correction: domain vocab is out of scope (23:54)

Actually, I'll correct myself and end by clarifying: the idea for this one is to isolate, in the
proof of concept at least, the post-processing elements, so that future trainings of a model that
integrates niche domain vocab (say my tech model) build on an established base that does the
post-processing reliably. So any domain vocab is out of scope for this one. This is the
experimental prototype, just for the post-processing tasks combined.

## Corrections

Mishearings fixed against `transcript-raw.txt`:

| Raw | Clean | Note |
|---|---|---|
| "into a compute out my own fine tunes" | "into a computer: trying out my own fine-tunes" | |
| "send it to the ladder" | kept, marked [unclear] | possibly "send it off" or "send it to the LLM" |
| "Parakeet Nematron" | "Parakeet and Nemotron" | both lines are being fine-tuned (Hebrish Shakshuka is the Nemotron one) |
| "Beto Akhlaomi" | "Bituach Leumi" | |
| "Whisperer" | "Whisper" | |
| "ROCM" | "ROCm" | |
| "evalues" | "evals" | |
| "tac flow cap" | "tech vocab" | |
| "the m's" | "ahs" | |
| "full starts" | "false starts" | |
| "adding instruction" | "editing instruction" | |
| "Tiberius" | "Tiberias" | |
| "Candidus ASR models" | "candidate ASR models" | |
| "S1 Mini by Superwhisper" | "S1-mini by Superwhisper" | |
| "experimentary" | "experimental" | the speaker's own coinage; same meaning |
| "Moonshine and Nvidia" | "Moonshine and NVIDIA" | |

At 23:54 the speaker revises the scope: domain vocabulary, including code-switching, is out of
scope for Proofed. The question at 23:39 is left as spoken; `docs/development-plan.md` records the
resulting decision.
