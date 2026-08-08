---
layout: post
title: I Built a DevRel AI Agent With OpenClaw — Here's the Full Architecture
date: 2026-03-03 12:00:00 +0100
tags:
  - ai
  - devrel
  - automation
  - buildinpublic
---

A few weeks ago I started running an AI agent on my laptop. It handles my entire LinkedIn content pipeline, drafts my weekly newsletter, manages my Notion databases, and checks my calendar.

I didn't buy a SaaS tool. I gave Claude access to my stuff, defined what it knows, and told it what to do.

Here's exactly how it works — and how you can build something similar.

---

## What is OpenClaw?

[OpenClaw](https://openclaw.ai) is an open-source framework for running Claude as a persistent, local AI agent. It's not a chatbot wrapper. It's infrastructure for a real assistant that:

- Has access to your tools (APIs, CLIs, files)
- Has persistent memory across sessions
- Can run on a schedule (heartbeats, cron)
- Can be extended via community-built skills

You run it on your own machine. You control what it touches. It runs as a background process, wakes up when you message it, and keeps state between sessions via plain text files.

If you want to understand OpenClaw quickly: it's what you get if you gave Claude a home directory, a shell, and a job description.

---

## My Agent: Orion Dev Rel

My agent is named **Orion Dev Rel** (📡). It runs on Claude Sonnet 4.6 and is scoped specifically to my DevRel workflow.

It has access to:

- **Google Workspace** (Gmail + Calendar via `gog` CLI)
- **Notion** (two databases: LinkedIn posts + newsletter drafts)
- **X/Twitter bookmarks** (via the `linkedin-pipeline` skill)
- **The web** (research, fetching content, search)
- **FeedHive** (for scheduling LinkedIn posts via MCP)
- **GitHub** (issues, PRs, repo ops via `gh` CLI)

Every agent in OpenClaw has a workspace directory with a few key files:

```
workspace/
├── SOUL.md       # Who the agent is. Persona, tone, principles.
├── USER.md       # Who it's helping. Background, goals, preferences.
├── MEMORY.md     # Long-term memory — curated, updated over time.
├── HEARTBEAT.md  # What to check periodically without being asked.
└── memory/
    └── YYYY-MM-DD.md   # Raw daily session logs
```

This is how continuity works. The agent wakes up fresh every session and reads these files to re-establish context. No database, no vector store — just files.

---

## The Full Architecture

![OpenClaw Architecture Diagram](/assets/images/openclaw-architecture.png)

Three zones: **Sources** (what feeds the agent), **Skills** (what it uses to process), and **Outputs** (where things end up).

---

## Pipeline 1: LinkedIn Content

This is the most-used workflow.

**Full flow:**

```
X Bookmark → fetch → research → 4 variations → humanize → Notion → review → FeedHive → LinkedIn
```

Step by step:

1. I bookmark a post on X that catches my attention
2. Orion fetches my bookmarks via the `linkedin-pipeline` skill
3. It researches the topic: web search + RAG across my own notes and emails
4. It writes **4 post variations** — 2 short, 2 long — in my voice
5. It runs a **humanize pass** to strip AI-writing patterns (no em dashes, no "leverage", no rule-of-three)
6. All 4 land in my Notion **Posts DB** with status `To Review`
7. I pick one, change status to `To Publish`
8. Orion schedules it on FeedHive: next available Mon–Thu slot, 11am Europe/Paris

**Deduplication is built in.** Before adding anything, the agent queries all Notion cards — including Discarded — and skips any bookmark already in there. Discarded cards act as a permanent blocklist.

What used to take 45 minutes per post now takes 5.

---

## Pipeline 2: Newsletter

Every Monday I publish *Building with AI* on Substack. Orion runs the production side.

**Flow:**

```
Notion Newsletter DB → draft → cover image → LinkedIn promo → Substack
```

The newsletter has its own Notion DB. Orion:

- Drafts the weekly issue based on recent bookmarks, notes, and what I've built
- Generates a **dark-mode cover image** using a custom Playwright script (hardcoded design — dark navy, purple/cyan accents, typography-only, no AI slop)
- Writes a LinkedIn promo post
- Stores both back in Notion, linked to the issue card

I write the personal section myself. Everything else is automated.

---

## Memory Architecture

Most AI agents lose context between sessions. OpenClaw solves this with files, not embeddings.

```
SOUL.md      → who the agent is (persona, values, how it communicates)
USER.md      → who it's helping (goals, background, preferences, writing style)
MEMORY.md    → distilled long-term memory — like a human's mental model
daily logs   → raw session notes (what happened, what was decided)
HEARTBEAT.md → what to check periodically without being asked
```

`SOUL.md` is the most important file. Mine tells Orion to skip filler words, have opinions, be resourceful before asking, and treat my private data with respect. It shapes every single response.

`MEMORY.md` gets updated over time. During slow heartbeat cycles, Orion reviews the daily logs and distills what's worth keeping long-term — decisions, context, lessons. The rest stays in the daily files.

This design is intentional. Plain text files are auditable, portable, and version-controllable. You know exactly what the agent knows because you can read it.

---

## The Heartbeat System

Orion doesn't just sit there waiting for messages. OpenClaw has a **heartbeat system** — periodic polls that let the agent act proactively.

I configured `HEARTBEAT.md` to batch-check a few things:

- **Email** — anything urgent?
- **Calendar** — events in the next 48 hours?
- **Pipeline** — anything stuck in `To Publish` in Notion?

It only reaches out when something needs attention. Late night (11pm–8am), it stays silent. No noise.

This is one of the most underrated parts of OpenClaw. You're not just building a chatbot — you're building something that watches things for you.

---

## Skills: How Extension Works

Everything the agent can do comes from **skills** — installable modules that give it instructions, CLIs, and APIs to work with.

A skill is a folder with a `SKILL.md` file. That file tells the agent what the tool does, how to call it, and when to use it. Some skills include shell scripts or config. It's simple by design.

My current skill setup:

| Skill | What it does |
|---|---|
| `linkedin-pipeline` | Fetches X bookmarks, drafts posts, syncs to Notion, schedules on FeedHive |
| `newsletter` | Drafts weekly issues, generates cover image, writes LinkedIn promo |
| `rag` | Searches across my notes, emails, and bookmarks for research context |
| `gog` | Google Workspace CLI — Gmail, Calendar, Drive |
| `notion` | Read/write Notion databases |
| `github` | GitHub ops via `gh` CLI (issues, PRs, CI) |
| `excalidraw` | Architecture diagrams on a local canvas (used to generate the diagram above) |

Skills are shareable. The community publishes them at [clawhub.com](https://clawhub.com). Installing one is a single command:

```bash
clawhub install linkedin-pipeline
```

---

## How to Get Started

If you want to try this yourself:

1. **Install OpenClaw** — [openclaw.ai](https://openclaw.ai), runs on Node.js
2. **Bootstrap your workspace** — OpenClaw guides you through creating `SOUL.md` and `USER.md`
3. **Install a skill** — start with something simple (`weather`, `github`, `notion`)
4. **Add a heartbeat** — even a weekly calendar check is a good first automation
5. **Expand from there** — the architecture stays the same as you add more

The learning curve is low if you're comfortable with CLIs and markdown. The ceiling is high.

---

## What's Next for Me

A few pipelines I'm building:

- **People CRM** — tracking connections, conversations, follow-ups inside Notion
- **Music pipeline** — lyrics drafts, session notes, release checklist for my Spotify tracks
- **Startup tracker** — syncing progress updates across projects to a single dashboard

The architecture is the same for all of it: give the agent access to the right tools, write (or install) a skill, and let it handle the repetitive parts.

OpenClaw is [open-source on GitHub](https://github.com/openclaw/openclaw). Community skills are at [clawhub.com](https://clawhub.com). If you build something with it, I'd love to see it.
