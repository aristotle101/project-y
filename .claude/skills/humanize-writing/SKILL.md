---
name: humanize-writing
description: Rewrite AI-sounding prose into natural human writing in the same language or a requested target language, with support for English and multilingual text. Use when the user asks to remove AI smell, make writing sound more human/natural/local, improve voice, cadence, flow, structure, or tone, or rewrite emails, posts, essays, captions, copy, and translated text without changing meaning.
---

# Humanize Writing

Rewrite prose so it reads like a real person wrote it. Rebuild structure, not just wording.

## Workflow

1. Identify source language, target language, audience, and channel.
2. Preserve facts, names, numbers, links, code, and required terminology.
3. Score the AI smell (0–5 scale below), then rewrite accordingly.
4. Return the cleaned draft first; add brief notes only when useful.

## Smell Score

- 0–1: sounds natural or needs minor cleanup only — fix wording in place, no restructure.
- 2: some AI traces, structure is fine — line-level fixes only.
- 3: needs restructuring and line-level rewrite — **read `references/rewrite-playbook.md` before continuing**.
- 4: reads mechanically — **read `references/rewrite-playbook.md` before continuing**.
- 5: unusable; keep meaning only and rebuild — **read `references/rewrite-playbook.md` before continuing**.

## Core Rules

- Keep meaning stable unless the user explicitly asks for a new angle.
- If no target language given, keep the dominant language of the source.
- Prefer natural rhythm, varied sentence length, direct phrasing.
- Remove filler, hedge loops, and repetitive framing.
- Don't over-polish into marketing copy or essay-speak.
- Keep register stable: casual stays casual, formal stays formal.
- For mixed-language input, keep dominant language; preserve code-switching only when natural.
- If target locale matters, localize idioms, punctuation, and tone.

## AI Smell Signals

Scan for these before scoring:
- Generic intros that could fit any topic
- Repetition with minor paraphrase
- Abstract nouns stacked on weak verbs
- Hedge words, disclaimers, filler transitions
- Machine-like bullet symmetry
- Translation artifacts

Two or more → change structure first, then wording.

## Output

- Default: rewritten text only.
- If asked: short change summary after the rewrite.
- If language/audience is ambiguous: state assumption in one line, then continue.
