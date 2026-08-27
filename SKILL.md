---
name: readwise-research
description: Use when starting any research, investigation, prior-art scan, landscape review, "find the best X for Y", or compare-options task — before web search or codebase reading
---

# Readwise Research Protocol

The user has curated his Readwise library for years. It is the highest-signal reference set for any topic he asks about. Skip the sweep and you will miss articles that would have shortcut the entire task.

## When to Use

Any task with words like *research, investigate, find prior art, scan the landscape, compare options, best practice for, deep dive, fork-or-build, evaluate*.

Whether you are doing the work yourself OR dispatching a subagent / Codex task — same rule.

## Scope Gate — Match Sweep Depth to Where the Answer Lives

Readwise is a curated library of *ideas* — essays, patterns, prior art, commentary. It is not a mirror of vendor documentation or our codebases. Before sweeping, ask: **where does the answer to this question canonically live?**

- **Open-topic research** — landscape scans, "best X for Y", architecture patterns, best practices, fork-or-build decisions, anything where a saved essay could reshape the approach → **Full protocol below: 6+ variations, both commands, done first.**
- **Canonical-source research** — a named product's own docs/API reference, our own codebase, a spec/RFC. The answer lives in the canonical source, not in an essay → **Trimmed sweep: 2–3 variations (the product name + its category, e.g. "Photon iMessage" + "iMessage API"), both commands.** Purpose: catch saved *commentary* you don't know about. If it returns nothing, move on — no ceremony. The trimmed sweep must never block or delay fetching the canonical docs — run it in parallel (delegated or solo).
- **Mixed tasks** — "evaluate vendor X as a replacement for vendor Y": trimmed sweep on each vendor name, plus full variations only for the *category* if the decision hinges on landscape knowledge.

When in doubt, sweep — a wasted sweep costs 60 seconds; a missed save costs hours. The gate trims ceremony; it does not license skipping.

## The Full Protocol (open-topic tasks)

**Before web search, before reading the codebase, before drafting an answer:**

1. Generate **6+ keyword variations** for the topic. Cover:
   - The literal topic
   - Synonyms and paraphrases
   - Adjacent concepts (techniques, patterns, frameworks in the space)
   - People and companies known to write in the space

2. For each variation, run BOTH:
   ```bash
   readwise reader-search-documents --query "<variation>"
   readwise readwise-search-highlights --vector-search-term "<variation>"
   ```

3. For every relevant hit, fetch full content:
   ```bash
   readwise reader-get-document-details --document-id <id>
   ```

4. Cite Readwise document IDs alongside web sources in the output.

Then — and only then — extend to web search.

## Dispatching Subagents / Codex

If the work is delegated, the agent prompt MUST contain a literal `## Step 0 — Readwise sweep` block listing the keyword variations — 6+ for open-topic tasks, 2–3 for canonical-source tasks (per the Scope Gate). The agent will not generate good variations on its own. Spell them out.

Do **not** instead inject "here are 3 saves from Readwise to look at." That treats Readwise as spoon-fed context and the agent will skip general search. The whole point is to discover saves you don't know about.

## Anti-Patterns

| ❌ | Why it fails |
|---|---|
| Dispatching with web-search-only instructions | Misses the curated library entirely |
| Listing 3 specific Readwise URLs and calling that "the Readwise step" | Agent treats it as context, not a search target |
| "I already know the answer, skip the sweep" | You don't know what was saved |
| "Quick task, not worth searching" | The sweep takes 60 seconds. The miss costs hours. Trim it (Scope Gate), don't skip it. |
| Searching only the literal topic phrase | Misses saves under synonyms / adjacent terms |
| Full 6+-variation protocol on a single-vendor docs lookup | Ceremony without payoff. Canonical-source tasks get the trimmed sweep — see Scope Gate. |

## Red Flags

- About to write a research agent prompt with no `Step 0 — Readwise sweep` block.
- About to call WebSearch / WebFetch as the first tool on a research task.
- About to recommend an approach without citing any Readwise document ID.

All of these mean: stop, run the sweep, then continue.

## Real-World Calibration — Both Poles

**Skipped when it shouldn't have been (2026-06):** six research agents dispatched on "agent-native codebase architecture." None searched Readwise. Two directly-on-target articles ("Making Our Monorepo Ergonomic for Agents" by Basis; "The self-driving codebase: Building Horizon at WorkOS") were both saved in Readwise and both missed. One day lost re-doing the lenses. Root cause: prompts handed agents specific URLs instead of mandating general Readwise search.

**Full protocol when trimmed was right (2026-07-22):** evaluating Photon (photon.codes) as a replacement for Linq — a canonical-source task (both vendors' docs + our codebase). The mandated 6+-variation sweep ran in two agents; no direct Photon coverage existed in the library. A 2–3-variation trimmed sweep would have surfaced the same four useful adjacent saves (Linq product page, OpenClaw-vs-Hermes comparison, hermes-agent README, Sendblue CLI tweet) at a fraction of the ceremony. This case created the Scope Gate.

## See Also

- `readwise-cli` skill — full CLI command reference.
