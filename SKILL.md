---
name: readwise-research
description: Use when starting research, a landscape review, "find the best X for Y", or a personal computing question where Tejas says "you don't understand me" or "you don't understand my thesis" — before comparing products, read his own words and the system he already built. For AI or agent design, search AI Engineer talks alongside Readwise.
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

## Personal systems before product rankings

When recommending a tool or platform for Tejas's own computing, read the primary statement of his objective, his relevant notes or journal, and the installed system he already uses. Derive the decision criteria from those sources before ranking market options. Keep his exact words, installed behavior, and agent-authored interpretations distinct. A feature comparison is insufficient when the requested outcome is authorship over data, algorithms, interfaces, or habits: name which layer he can actually change and which remains controlled by a service. Use the smallest live trial that separates the leading candidates, and say what existing function a migration would lose.

Incident, 2026-10-22: An answer to his phone question recommended iPhone blocking products and a stock Pixel from app availability and screen-time features before reading his manifesto or thnkr.ing's Journal Lock, Focus, meditation, and capture paths. He rejected it: “I'm not happy at all with the solution. I think the agent doesn't understand what personal computing means here.” He clarified: “This is not just about screen time or shaping the phone. Sovereignty over computing, sovereignty over my portal to the internet.” The wrong first theory was that stronger limits solved his phone problem; the actual decision is which substrate lets his own system shape the portal while preserving daily functions.

## The Full Protocol (open-topic tasks)

**Before web search, before reading the codebase, before drafting an answer:**

1. Generate **6+ keyword variations** for the topic. Cover:
   - The literal topic
   - Synonyms and paraphrases
   - Adjacent concepts (techniques, patterns, frameworks in the space)
   - People and companies known to write in the space

2. **First, his own index** (server; Mac agents prefix `ssh remote-box`). thnkr.ing's Reader holds
   what he saves and, as the Readwise import proceeds, his Readwise library and highlights, cut into
   passages and searched by words and meaning at once. Pass up to six variations in one call; hits
   are merged by passage, each with title, author, source link and a Reader link to that passage:
   ```bash
   library search "<variation 1>" "<variation 2>" ... --limit 20
   library document <document id>          # full text as Markdown
   library highlights <document id>         # his highlights in it
   library session <document id>            # that article's conversation, for sessions ask
   ```
   On his own research questions this matched or beat Readwise's search in the 2026-10-21
   measurement, but that comparison was judged by a model and is exploratory (thnkr.ing
   `docs/plans/2026-10-21-reader-search.md`).

3. **Then Readwise, for what the import has not reached yet.** For each variation run BOTH:
   ```bash
   readwise reader-search-documents --query "<variation>"
   readwise readwise-search-highlights --vector-search-term "<variation>"
   ```
   and for every relevant hit not already found, fetch full content:
   ```bash
   readwise reader-get-document-details --document-id <id>
   ```

4. Cite Reader or Readwise document IDs alongside web sources in the output.

Then — and only then — extend to web search.

## AI Engineer Talks — the Second Library (AI engineering and agent design)

When the topic touches AI engineering — agents, harnesses, memory, context engineering, evals,
RAG, coding agents, tool use, MCP, skills, voice, inference, AI product design — also search the
AI Engineer conference talks (World's Fair, Summit, Code, Europe; about 1,200 talks, every one
with a transcript). Tejas used to add these to Readwise by hand so agents would find them; this
replaces that, so do not import talks into his Readwise.

Run it with the same keyword variations, in the same step as the Readwise sweep:

```bash
~/.local/bin/ai-engineer-talks search <variation> --limit 8      # on the server
ssh remote-box '~/.local/bin/ai-engineer-talks search <variation>'   # from the Mac
~/.local/bin/ai-engineer-talks talk <slug> --query "<words>"  # read more of one hit
```

Each hit gives the title, speakers, event (the conference and year), length, the talk page, a
summary, and the best transcript passages with a link that opens the video at that moment. Cite
talks by title, speaker and that timestamp link. The summary is ai.engineer's description, not
a quotation; quote only passage text. Transcripts are automatic and can garble names. Words are
matched literally (with plural/stem folding), so vary terms as you would for Readwise. Exact
flags: `ai-engineer-talks --help`. A staleness warning means the server's daily refresh broke;
say so and continue.

When dispatching, put the `~/.local/bin/ai-engineer-talks search` lines in the `Step 0` block next to the
Readwise commands for any AI engineering topic.

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
- `ai-engineer-talks --help` — the talk search; its row in the `capability-map` skill.
