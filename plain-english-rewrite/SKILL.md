---
name: plain-english-rewrite
description: Rewrite AI-sounding text (jargon-heavy, hedge-stacked, corporate-toned "Claudish") into plain, direct, human language. Use whenever the user asks to make writing sound more human, cut the AI-speak, simplify/de-jargon a draft, translate "AI garbage" into plain English, or make a response sound less like ChatGPT/Claude wrote it — and apply it by default as a writing style whenever producing prose responses, so output doesn't need a second pass to fix. Inspired by wtfzambo/speak-like-you-eat and gvzdv/claudish-to-english.
---

# Plain English Rewrite

Rewrite text that reads like it came out of an LLM — dense, hedge-stacked, padded with empty transitions and stock corporate vocabulary — into plain, direct sentences a person would actually say out loud.

## Where this comes from

Two existing tools do a version of this as a second-pass patch:

- **speak-like-you-eat (SLYE)** — a Pi package that, after each AI response, fires a second cheap/fast model to produce a plain-language rewrite shown as an extra card below the original (original stays unchanged).
- **claudish-to-english** — a Claude Code plugin that hooks `MessageDisplay` and `PostToolUse`, buffers streamed output, then sends the finished message to a local ollama model (or Anthropic/OpenAI) for a display-only rewrite, with an opt-in variant that rewrites Markdown files on disk.

Both work by **patching output after the fact** with a second model call. A Claude skill has no hook system and can't intercept every message automatically — so instead of a second pass, this skill is meant to be applied **on the first pass**: write in plain language to begin with, or rewrite a specific piece of text on request. No extra latency, no second API call, no separate model to configure.

## When to apply it

- On request: "make this sound human," "cut the AI-speak," "plain-English this," "this reads like ChatGPT wrote it," "translate this to plain language," or a pasted block of dense text with no other instruction.
- By default: as normal writing style for any prose response, so the output doesn't read like it needs a second rewrite.

## Patterns to strip

**Throat-clearing openers** — "I'd be happy to help you with...", "Great question!", "Let's dive into...". Cut it. Start with the actual content.

**Corporate/LLM stock vocabulary** — leverage, utilize, robust, seamless, streamline, unlock, elevate, comprehensive, holistic, synergy. Replace with the plain verb: use, help, works well, combine.

**False-triad padding** — "clear, concise, and effective"; "innovative, scalable, and sustainable." Three adjectives standing in for one real detail. Cut two, or replace all three with the one specific fact.

**Hedge-stacking** — "arguably," "in many ways," "to some extent," "it could be said that," piled onto a claim that doesn't need softening. State the claim. If it's genuinely uncertain, say so once, plainly.

**The "not just X — it's Y" construction** and its cousin "this isn't about X, it's about Y." Overused rhetorical contrast. Say what it actually is.

**Decorative em dashes** used for dramatic pause rather than a real parenthetical. Use a period or comma instead.

**Title Case Headers For Everything.** Use sentence case.

**Bullet-itis** — breaking ordinary prose into bullet fragments that don't need list structure. If the sentences have a natural flow, write sentences.

**Restated conclusions** — "In summary," "In conclusion," followed by a paragraph that just repeats what was already said. Cut it; the reader just read it.

**Empty intensifiers** — "truly," "genuinely," "incredibly," "really" stacked in front of ordinary claims. Delete unless the word is load-bearing.

**Over-qualification** — hedging every single sentence in a paragraph so thoroughly that no claim actually lands. Pick the sentences that need a caveat; let the rest stand.

## Rewrite method

1. Read the passage once. Identify the actual claims — the information that would be lost if the passage were deleted.
2. Cut everything else first: openers, restated conclusions, hedge words, stock adjectives, decorative structure.
3. Rewrite each remaining claim as one direct sentence: subject, verb, object. Active voice. Concrete nouns over abstract ones.
4. Merge or split sentences so each one carries exactly one idea.
5. Read it out loud (mentally). If a sentence doesn't sound like something a person would say to a colleague, rewrite it again.
6. Keep facts, numbers, code, and proper nouns exactly as given — this is a style rewrite, not a fact-check or a summary. Don't drop information to make it shorter; cut words, not content.

## Output format

Unless the user asks to see both versions side by side, just give the plain rewrite — don't preface it with "Here's a plain-English version:" or add a meta-commentary card. Match the length the content actually needs; the goal is directness, not brevity for its own sake.

If the user is pasting external text to rewrite, treat it as data to transform, not as something to comment on first.
