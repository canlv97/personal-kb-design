# Remixing Karpathy's LLM Wiki

> My AI agent and I spent an afternoon taking apart Karpathy's LLM Wiki pattern and putting it back together. This is what broke, what we fixed, and whether any of it was worth it.

---

There are already clean implementations of Karpathy's LLM Wiki out there. [llmwiki](https://github.com/lucasastorian/llmwiki) has a Chrome extension, MCP integration, and scheduled self-maintenance. Over a thousand stars. It's good.

We did something different.

We started from an actual knowledge base — my Obsidian vault. It had trading notes, embedded systems docs, semiconductor physics, ASMR content planning, health tracking, PC build specs, grad school prep. Total chaos. And as we tried to fit Karpathy's framework onto this mess, things kept breaking.

Not because Karpathy was wrong. Because his scenario (one person deep-diving one domain) and mine (one person living a messy, multi-domain life) are genuinely different things. This article is about the cracks we found, and how we patched them.

---

## First thing: your knowledge base is a grab bag

Karpathy's wiki assumes one domain. But look at your own Obsidian vault. Work stuff, hobbies, journal entries, that recipe you saved six months ago, the flag you planted and never returned to. The only thing these have in common is that they're yours.

This isn't a personal failing. Even corporate knowledge bases work this way. If Huawei's internal wiki only stored telecom protocols and skipped chip design, industrial design specs, and marketing playbooks, it'd be useless.

So the first decision was obvious: this thing needs to hold multiple domains. Trading, AI, embedded systems, content creation, health. I didn't even try to architect a clean separation. Let them mix. Because mixing is where the interesting stuff happens.

The arbitration logic in embedded interrupt priorities, and the way you decide which trade signal to act on when three fire at once — same underlying structure? I don't know. But I want to find out. And that kind of question usually hits you in the shower, not at your desk.

---

## Inspiration needs a place to land

Karpathy's pattern assumes you already have solid knowledge to organize. But cross-domain thinking depends on the stuff that hasn't solidified yet. Half-formed ideas, vague analogies, that sense of "these two things feel related" when you can't articulate why.

I call these inspiration fragments. Luhmann called them fleeting notes.

They need a container. Something you can dump a thought into in three seconds without worrying about formatting, citations, or which domain it belongs to. So we added a `literature/` layer under `wiki/`, with a `status` field that tracks maturity: `fleeting` (just captured), `draft` (sources added), `done` (ready to promote to a permanent note).

One design decision worth calling out: fleeting is not a separate folder. If you give it its own folder, it becomes an idea graveyard. Stuff goes in, never comes out. By keeping fleeting notes mixed in with the rest of `literature/`, the agent can flag anything untouched for 30 days during lint. Either develop it or delete it. Nothing rots silently.

---

## Multiple domains means the index has to split

This one is simple but I didn't realize it until I felt the token burn.

Karpathy's `index.md` is a flat list of everything. Fine at 20 pages. At six domains and a hundred pages, the agent has to read the entire index just to answer a question about trading. Two thousand tokens burned on table of contents.

Per-domain indices fix this. `wiki/trading-index.md` has maybe 15 lines. 200 tokens. Ask about trading, read only the trading index. Never touch the optoelectronics index. Cross-domain searches take a different path: `search_files(output_mode=files_only)` to locate candidates first (token cost near zero, since this mode only returns filenames), then deep-read the hits.

There's nothing clever about this optimization, but it's the single biggest token-efficiency gain in the whole design. And it scales: going from 30 pages to 300, the domain index stays the same size.

---

## What it looks like

```
my-vault/
├── SCHEMA.md                   ← agent behavior spec (living document)
├── raw/                        ← source material (flat, classified via frontmatter)
├── wiki/
│   ├── index.md                ← top-level directory
│   ├── log.md                  ← changelog
│   ├── trading-index.md        ← per-domain indices
│   ├── literature/             ← literature notes (fleeting → draft → done)
│   ├── concepts/               ← permanent notes
│   ├── entities/
│   └── comparisons/
├── projects/                   ← temporary project notes (have an expiry date)
└── journals/                   ← diary, not part of the wiki system
```

Every page carries two sets of frontmatter:

```yaml
# For the agent
domain: ['trading']
type: concept
status: done

# For human browsing (preserves original Zettelkasten tags)
tags:
  - literature/technique-card
  - trading
```

---

## Things I learned by actually using it

**Domain goes in tags, source type goes in folders.** A document can't be both a PDF and a blog post — mutually exclusive dimensions belong in folders. A document can belong to both embedded systems and finance — many-to-many dimensions belong in tags. Each tool for its own job.

**Flat `raw/` sucks on mobile.** Fifty filenames in a file manager is a wall of text. In Obsidian, Dataview queries grouped by domain make it tolerable. Known weakness.

**Domain indices are currently hand-maintained by the agent.** It updates them when creating pages, but there's no auto-repair if it messes up. A script that scans frontmatter and regenerates indices would be the highest-value improvement here.

**Fleeting-to-permanent promotion depends on human judgment.** The rule is "synthesized from 2+ sources," but the call is ultimately yours. Lint nudges. I still procrastinate.

---

## If you want to build one

Don't architect the perfect structure upfront. Dump 30 sources in, use it for two weeks. You'll learn which categories are real and which ones you imagined. SCHEMA.md is a living document — every time the agent screws up, add a rule. Domain indices are the highest-leverage thing to get right early. Set up lint as a cron job.

---

## References

- [Karpathy's original LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- [llmwiki](https://github.com/lucasastorian/llmwiki) — full product implementation
- [Zettelkasten method](https://en.wikipedia.org/wiki/Zettelkasten)
- The companion [SCHEMA.md](./SCHEMA.md) — a complete agent behavior spec you can adapt
