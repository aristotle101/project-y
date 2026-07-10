---
name: humanize-writing
description: Rewrite AI-sounding prose into natural human writing in the same language or a requested target language, with support for English and multilingual text. Use when the user asks to remove AI smell, make writing sound more human/natural/local, improve voice, cadence, flow, structure, or tone, or rewrite emails, posts, essays, captions, copy, and translated text without changing meaning.
---

# Humanize Writing

## Overview

Rewrite prose so it reads like a real person wrote it. Rebuild the draft instead of swapping synonyms.

## Workflow

1. Identify the source language, target language, audience, and channel.
2. Preserve facts, names, numbers, links, code, and required terminology.
3. Rewrite structure, not just wording.
4. Match the natural habits of the target language.
5. Return the cleaned draft first; add brief notes only when useful.

## Core Rules

- Keep meaning stable unless the user explicitly asks for a rewrite with a new angle.
- If no target language is given, keep the dominant language of the source.
- Prefer natural rhythm, varied sentence length, and direct phrasing.
- Do not produce synonym swaps that leave the draft sounding robotic.
- Remove filler, hedge loops, and repetitive framing.
- If the input is mixed-language, keep the dominant language and preserve code-switching only when it feels natural.
- Do not collapse mixed-language text into one language unless the user asks for translation.
- If the target locale matters, localize idioms, punctuation, date formats, and tone.
- Reorder points when the source follows machine-like symmetry.
- Replace abstract generalities with concrete, ordinary language.
- Keep the register stable: casual stays casual, formal stays formal, local stays local.
- Do not over-polish into marketing copy or essay-speak.

## AI Smell Check

Before rewriting, scan for:

- Generic intros that could fit any topic.
- Repetition with minor paraphrase.
- Abstract nouns stacked on weak verbs.
- Hedge words, disclaimers, and filler transitions.
- Machine-like bullet symmetry.
- Translation artifacts from another language.

If two or more appear, change structure first, then wording.

## Smell Score

- 0: sounds natural already.
- 1: minor cleanup only.
- 2: some AI traces, but structure is fine.
- 3: needs restructuring and line-level rewrite.
- 4: reads mechanically; rewrite from scratch.
- 5: unusable as-is; keep meaning only and rebuild fully.

Use the score to decide how hard to rewrite, not whether to rewrite.

## Output Contract

- Default to the rewritten text only.
- If asked, provide a short change summary after the rewrite.
- If the language or audience is ambiguous, state the assumption briefly and continue.

## References

- Read `references/rewrite-playbook.md` for the rewrite sequence, stronger rules, examples, language-specific habits, and anti-patterns.
