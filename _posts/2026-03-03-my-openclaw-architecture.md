---
layout: post
title: My OpenClaw Architecture — How I Built a DevRel AI Agent
date: 2026-03-03 12:00:00 +0100
tags:
  - ai
  - devrel
  - automation
  - buildinpublic
---

I've been running an AI agent on my laptop for a few weeks now. It handles my LinkedIn content pipeline, drafts my weekly newsletter, manages my Notion databases, and checks my calendar. I didn't buy a SaaS tool for any of it. I just gave an AI agent access to my stuff.

Here's the full architecture.

---

## The Setup

The agent runs on [OpenClaw](https://openclaw.ai), an open-source framework that lets you run Claude as a persistent, local AI agent. It has access to:

- My Google Workspace (Gmail + Calendar via `gog` CLI)
- My Notion databases (LinkedIn posts + newsletter drafts)
- My X/Twitter bookmarks
- The web (research, fetching content)
- FeedHive (for scheduling LinkedIn posts)

The agent has a name: **Orion Dev Rel**. It's Claude Sonnet 4.6 running with a custom persona, memory files, and a set of skills tailored to my DevRel workflow.

---

## The Architecture

![OpenClaw Architecture Diagram](/assets/images/openclaw-architecture.png)

Three zones: **Sources** (what feeds the agent), **Skills** (what the agent uses to process), and **Outputs** (where things end up).

---

## The Main Pipeline: LinkedIn Content

This is the one I use most.

**Flow:** X Bookmark → Research → 4 Post Variations → Humanize → Notion → FeedHive → LinkedIn

1. I bookmark a post on X that I find interesting
2. Orion fetches my bookmarks via the `linkedin-pipeline` skill
3. It researches the topic (web search + RAG across my notes/emails)
4. It writes 4 variations: 2 short, 2 long — in my voice
5. It runs a humanize pass to strip AI patterns
6. It pushes all 4 to my Notion **Posts DB** with status `To Review`
7. I review, approve one → status becomes `To Publish`
8. Orion schedules it on FeedHive for the next Mon–Thu slot at 11am Paris time

The whole thing used to take me 45 minutes per post. Now it takes 5.

**Deduplication built-in:** Before adding anything, the agent checks all Notion cards (including Discarded ones) against the source URL. If a bookmark is already in there — even if I discarded it — it gets skipped.

---

## The Newsletter Pipeline

Every Monday I publish *Building with AI* on Substack.

**Flow:** Notion Newsletter DB → Draft → Cover Image → LinkedIn Promo

The newsletter has its own Notion DB. Orion manages the full pipeline:

- Drafts the weekly issue based on what's been happening (my notes, bookmarks, what I've built)
- Generates a dark-mode cover image using a custom Playwright script (hardcoded design — dark navy, purple/cyan accents, typography-only)
- Writes a LinkedIn promo post
- Stores everything back in Notion for me to review before publishing

I write the personal section myself. Everything else is Orion.

---

## Memory Architecture

Every session, the agent wakes up fresh. These files are its continuity:

```
workspace/
├── SOUL.md          # Who Orion is. Personality, tone, principles.
├── MEMORY.md        # Long-term curated memory (like a human's mental model)
├── USER.md          # Who I am, my goals, my writing style
├── HEARTBEAT.md     # What to check periodically
└── memory/
    └── YYYY-MM-DD.md   # Daily session logs
```

`SOUL.md` defines the vibe — no filler words, have opinions, be resourceful before asking. `MEMORY.md` is the distilled essence of everything important that's happened. Daily files are raw logs.

Every few days, during a heartbeat, Orion reviews the daily logs and updates `MEMORY.md` with what's worth keeping long-term. The rest gets left in the daily files.

---

## The Heartbeat System

Orion doesn't just sit there waiting for me to talk to it.

OpenClaw has a heartbeat system — periodic polls that let the agent check in proactively. I configured it to batch-check a few things without burning too many tokens:

- **Email** — any urgent unread messages?
- **Calendar** — events coming up in the next 48h?
- **Pipeline status** — anything stuck in `To Publish` in Notion?

It only reaches out if something actually needs attention. Late night (11pm–8am), it stays quiet. No alerts for nothing.

---

## Skills

Everything the agent can do comes from installable **skills** — SKILL.md files that give it instructions, CLIs, and APIs to use.

My current setup:

| Skill | What it does |
|---|---|
| `linkedin-pipeline` | Fetches X bookmarks, drafts posts, syncs to Notion, schedules on FeedHive |
| `newsletter` | Drafts weekly newsletter, generates cover, writes LinkedIn promo |
| `rag` | Searches across my notes, Gmail, and bookmarks for context |
| `gog` | Google Workspace CLI — Gmail, Calendar, Drive |
| `notion` | Read/write Notion databases |
| `github` | GitHub ops via `gh` CLI |
| `excalidraw` | Architecture diagrams on a local canvas (used to make the diagram above) |

Skills are just markdown + scripts. They're shareable. The community publishes them at [clawhub.com](https://clawhub.com).

---

## What It's Not

This isn't magic. It's not an autonomous agent running 24/7 rebuilding my entire life.

It's more like a very reliable collaborator that:
- Knows my workflow
- Knows my voice
- Has the right tools installed
- Doesn't forget things (files > memory)

I still review everything before it goes public. I still make the calls. Orion handles the grunt work.

---

## What's Next

A few things I'm building toward:

- **People CRM** — tracking connections, conversations, follow-ups inside Notion
- **Music pipeline** — lyrics drafts, session notes, release checklist
- **Startup tracker** — syncing progress updates to a Notion dashboard

The architecture is the same for all of it: give the agent access, write a skill, let it do the boring parts.

If you're building something similar, [OpenClaw is open-source](https://github.com/openclaw/openclaw). Skills are at [clawhub.com](https://clawhub.com).
