# 🏠 AI Home Lab — Part Deux

> *Phase 1 of assimilation. The homelab was the sandbox. Everything else is next.*

A local-first, privacy-respecting multi-agent AI infrastructure that started with homelab automation and is expanding into work intelligence, family logistics, consulting discovery, and home automation — powered by a network of specialized AI agents on dedicated hardware.

---

## 🧠 How AI Changed the Game

### Before: The Traditional Homelab

A typical homelab is a collection of services running on repurposed hardware — Plex for media, Home Assistant for automation, Frigate for cameras, a pile of Docker containers for everything else. It works, but it's fundamentally **reactive and manual**:

- **You watch the dashboards.** Netdata, Uptime Kuma, Homepage — dozens of status tiles that you have to remember to check.
- **You troubleshoot alone.** A container crashes at 3 AM? You find out when you wake up. Then you SSH in, dig through logs, Google the error, piece together a fix.
- **You write every automation.** Want a daily health summary? Write a script. Camera alerts? Write another script. Each one is a bespoke snowflake that only you understand.
- **You're the bottleneck.** Every integration, every alert rule, every config change goes through you. The homelab is only as smart as the time you have to spend on it.

This was the state of things for years. Services ran. Things mostly worked. But the homelab wasn't a partner — it was a collection of appliances waiting to be told what to do.

### After: The AI-Native Homelab

Adding Gengar (a cloud-powered AI orchestrator) and Ditto (a local LLM server) transformed the homelab from a **static infrastructure** into an **intelligent operating environment**. The shift wasn't incremental — it was categorical.

#### ⚡ Gengar: The Conductor

Gengar is the **first thing that could actually do things** — not just report status, but act on it. It has full shell access, Docker control, SSH to every host, file system access, and Git. It doesn't just tell you a container is down — it investigates why, checks related services, reads the logs, and either fixes it or drafts exactly what you need to do.

What used to take 45 minutes of context-switching (SSH, grep, Docker inspect, log spelunking) is now a single sentence in Discord: *"Frigate is spiking memory again — check it out."* Gengar does the spelunking, correlates across systems, and presents findings.

**But the real unlock is what Gengar builds while you sleep.** It writes monitoring scripts. Creates dashboards. Sets up cron jobs that watch other cron jobs. It maintains an Obsidian knowledge base so nothing is lost between sessions. It backs itself up to GitHub every 12 hours. The homelab went from *"I maintain this"* to *"this maintains itself — and tells me when it needs me."*

#### 👻 Ditto: Privacy Without Compromise

Ditto solved the problem that keeps most people from going all-in on AI assistants: **not everything should go to the cloud.**

Ditto runs entirely on local hardware — a BOSGAME M5 with 128GB of unified memory and an AMD Ryzen AI Max+ 395. Five specialized model lanes cover everything from quick answers (Gemma 4 at 55 tok/s) to deep reasoning (Qwen 3.6 35B MoE at 46 tok/s). Zero API keys. Zero cloud calls. Zero privacy concerns.

The insight was architectural: **not every task needs gpt-5.4.** Summarizing logs? Researching a topic? Analyzing a dashboard? Ditto does it locally, instantly, and for free. When a task genuinely needs frontier-model reasoning, Ditto drafts a precise prompt and hands it to Gengar. This isn't a fallback — it's a **task routing system** where the right model handles the right job.

**The MoE discovery was a breakthrough.** Mixture-of-Experts models achieve ~6x higher throughput than dense models of the same parameter class on AMD unified memory hardware. This single insight turned Ditto from "a fun local toy" into a genuinely capable daily driver.

#### 🔄 The Multiplier Effect

Gengar and Ditto together create something neither could alone:

| Capability | Before | After |
|-----------|--------|-------|
| **Incident response** | Discover at morning coffee, fix when free | Watchdogs detect → Ditto analyzes → Gengar fixes or escalates |
| **System visibility** | Check dashboards manually | Morning digest in Discord with overnight summary |
| **Service uptime** | Reactive restarts | 1-minute watchdogs with silent-on-healthy pattern |
| **Knowledge retention** | Memory and scattered notes | Obsidian vault maintained by agents, Git-backed |
| **New capabilities** | Weeks of scripting | Describe what you want, Gengar builds it |
| **Privacy-sensitive tasks** | Cloud-only or nothing | Ditto handles locally, Gengar handles the rest |
| **Daily overhead** | 20-30 min checking systems | One Discord message, 30 seconds to scan |

### The Takeaway

Adding AI to a homelab isn't about the models — it's about **closing the gap between observation and action.** Traditional homelabs are great at collecting data (metrics, logs, alerts) but terrible at doing anything with it without human intervention. Gengar closes that gap on the action side. Ditto closes it on the privacy side. Together, they turn a collection of services into something that feels less like infrastructure and more like a teammate.

---

## 🧬 Architecture Overview

![AI Home Lab Architecture](assets/architecture_diagram.png)

> **[Interactive SVG version](assets/architecture_diagram.html)** — open in any browser for the full dark-themed diagram with grid background, color-coded components, and animated pulse indicator. Available offline — zero dependencies.

---

## 🖥️ Hardware

### Gengar — The Orchestrator
**Mac Mini M4** (Apple Silicon, 16GB unified memory)

The brains of the operation. Runs [Hermes Agent](https://github.com/NousResearch/hermes-agent) as the primary orchestrator, connecting to cloud LLMs for complex reasoning, tool orchestration, and system control. Hosts the ops dashboard, manages cron jobs, and bridges all platforms (Discord + Telegram).

### Ditto — The Local LLM Server
**BOSGAME M5 Mini PC** (AMD Ryzen AI Max+ 395, 128GB unified memory, ~96GB allocatable to iGPU)

A dedicated local inference machine running Ollama. Handles read-only tasks: research, summarization, alert analysis, and homelab reporting. Runs entirely on LAN with zero cloud dependency — all models stay local.

### Unraid — The Homelab Backbone
Custom server (i5 / RTX 3080) running Unraid with Docker containers for media, search, monitoring, and camera processing.

---

## 🤖 The Agent Team

### ⚡ Gengar — Primary Orchestrator

| Attribute | Detail |
|-----------|--------|
| **Role** | Primary agent, system controller, task executor |
| **Personality** | Direct, concise, no-fluff technologist |
| **Access** | Full: shell, files, git, Docker, SSH, cron, Obsidian, messaging |
| **Model** | gpt-5.4 (cloud) |
| **Platforms** | Discord, Telegram |

Gengar is the only agent with **mutation privileges** — file writes, service restarts, config edits, Docker control, and SSH access. All other agents escalate to Gengar for any action that changes system state.

**Key responsibilities:**
- Orchestrates complex multi-step tasks
- Manages cron job scheduling and monitoring
- Controls Docker/Unraid services
- Handles GitHub operations (PRs, repos, backups)
- Maintains Obsidian knowledge base
- Bridges all communication platforms

---

### 👻 Ditto — Local Read-Only Analyst

| Attribute | Detail |
|-----------|--------|
| **Role** | Local-only read-only reporter and researcher |
| **Personality** | Helpful but constrained — knows its limits |
| **Access** | Web search (SearXNG), vision (image analysis) |
| **Models** | All local Ollama models |
| **Platforms** | Discord |

Ditto runs entirely on local hardware — zero cloud API calls. It's the privacy-first workhorse for tasks that don't need frontier-model reasoning.

**Model lanes:**
| Alias | Backing Model | Speed | Use Case |
|-------|--------------|-------|----------|
| `ditto-fast` | Gemma 4 27B | ~55 t/s | Quick answers, simple tasks |
| `ditto-default` | Qwen 3.6 35B-A3B (MoE) | ~46 t/s | General chat, explanations, daily briefings |
| `ditto-research` | Qwen 3.6 35B-A3B (Q8) | ~46 t/s | Deep research, long-context synthesis |
| `ditto-coder` | Qwen 3 Coder 30B-A3B (MoE) | ~65 t/s | Code analysis, architecture review, prompt drafting |
| `ditto-heavy` | Llama 4 Scout 17B | ~17 t/s | Deep analysis, complex reasoning |

> **Architecture insight:** MoE (Mixture of Experts) models achieve ~6x higher throughput than dense models of the same parameter class on AMD unified memory hardware. This is why Ditto exclusively uses MoE-architecture models.

**Key responsibilities:**
- Summarize homelab alerts and monitoring data
- Web research with source attribution
- Draft precise prompts for Gengar when action is needed
- Analyze incidents and correlate signals
- Image analysis via vision

---

### 🦴 Cubone — Ops Dashboard

| Attribute | Detail |
|-----------|--------|
| **Role** | Real-time operations dashboard |
| **Access** | Read-only API polling |
| **Serves** | `gengar-cockpit` on LAN |

A dark-themed, real-time web dashboard that provides unified visibility into the entire AI home lab. Built with FastAPI + vanilla HTML/CSS/JS with zero framework overhead.

![Gengar Cockpit Dashboard](assets/gengar_cockpit.png)

**Shows:**
- Gengar system health (CPU, memory, disk, process count)
- Ditto telemetry (temps, VRAM, loaded models, inference speed)
- Ollama model alias map with live performance probes
- Cron job status and last-run times
- Rotus fitness app health
- Obsidian vault sync status
- OpenRouter usage and credits
- Firecrawl cloud status
- Backup job status

---

### 🥄 Alakazam — Morning Digest

| Attribute | Detail |
|-----------|--------|
| **Role** | Automated morning briefing agent |
| **Schedule** | Daily at 07:10 |
| **Model** | Qwen 3.6 35B-A3B (local, via Gengar) |

A two-stage pipeline that collects system state at 07:05, then generates a concise morning digest at 07:10 delivered to Discord. Covers homelab health, Docker container status, Unraid disk array, Ditto metrics, cron job summaries, and any overnight incidents.

**Architecture:** Collector script (no_agent cron) → JSON file → Digest agent reads file and summarizes. This avoids the `context_from` limitation with upstream no_agent jobs.

---

## ⚙️ Infrastructure

### Hermes Agent Framework

Everything runs on [Hermes Agent](https://github.com/NousResearch/hermes-agent) — an extensible AI agent framework with:

- **Multi-profile support** — Gengar and Ditto run as separate profiles with isolated configs and credentials
- **Cron scheduler** — 13 automated jobs from 1-minute watchdogs to daily digests
- **Multi-platform gateway** — Discord and Telegram with thread-aware routing
- **Skill system** — 50+ reusable skill modules for specialized tasks
- **MCP integration** — Modular tool servers for homelab, GitHub, and filesystem access
- **Persistent memory** — Cross-session context retention
- **Obsidian integration** — Read/write/search the knowledge vault

### Task Distribution (Cron Jobs)

| Job | Frequency | Engine | Delivery |
|-----|-----------|--------|----------|
| Homelab Monitor | Every 5 min | Shell script | Local |
| Ditto Watchdog | Every 10 min | Shell script | Discord (alerts only) |
| Firecrawl + Frigate Watchdog | Every 5 min | Python script | Discord (alerts only) |
| Camera Alert Handler Watchdog | Every 1 min | Python script | Local |
| Rotus Web Watchdog | Every 1 min | Python script | Local |
| Hermes Zombie Killer | Every 30 min | Shell script | Discord (alerts only) |
| Alakazam Digest Collector | Daily 07:05 | Python script | Local (file) |
| Alakazam Morning Digest | Daily 07:10 | Shell script | Discord |
| Ditto Daily Briefing | Daily 07:00 | Python collector + LLM | Discord |
| Rotus Daily Post | Daily 06:00 | Shell script | Discord |
| GitHub DR Backup | Every 12 hrs | Shell script | Local |
| Hermes Agent Backup | Every 12 hrs | Shell script | Local |
| Homelab Daily Digest | Daily | Shell script | Local |

> **Design pattern:** Watchdogs use the "silent when healthy" pattern — only produce output when something is wrong. This keeps Discord notifications signal, not noise.

---

## 🧰 Supporting Tools

### 🔍 SearXNG
Self-hosted, privacy-respecting metasearch engine on Unraid. Aggregates Google, Bing, DuckDuckGo, Brave, Reddit, and GitHub — with zero API keys, zero rate limits, and zero tracking. Both Gengar and Ditto use it as their primary web search backend.

### 📓 Obsidian
A structured knowledge vault with 29+ project notes across 6 project categories. Hermes can read, search, create, and update notes — making it a persistent, human-readable knowledge base that survives agent sessions. Git-backed to a private GitHub repo for DR.

**Project structure:**
```
Projects/
├── Gengar/       — Agent server state, decisions, bugs
├── Ditto/        — Local LLM config, research plans, incidents
├── Rotus/        — Fitness app architecture, API notes, deployment
├── Alakazam/     — Morning digest specifications
├── Enterprise AI Governance/ — Work-related templates and checklists
└── Gengar Cockpit.md — Dashboard documentation
```

### 🐳 Docker + Unraid
Docker on Unraid hosts: SearXNG, Frigate (camera AI), Firecrawl (web scraping), Netdata (monitoring), Plex (media), and more. Inspection-only Docker API proxy allows Gengar to monitor container health without mutation risk.

### 📊 Monitoring Stack
- **Netdata** on Unraid for real-time system metrics
- **Homelab Monitor** (custom) for service-level health checks
- **Gengar Cockpit** for agent and service visibility
- **Ditto Watchdog** for temperature/RAM/VRAM tracking on the local LLM box

---

## 🏃 Services Running

### Rotus — Personal Fitness Coach
A private, LAN-only running and fitness tracking app (FastAPI + SQLite). Tracks runs, weight, body metrics, and generates AI-powered coaching feedback. Posts a daily summary to Discord via cron. Implements conservative return-to-running plans and COROS-style heart rate zone training.

### Camera Alert Handler
Replaces noisy Home Assistant Discord notifications with a debounced, intelligent alert pipeline. Handles Frigate camera events with rate limiting, snapshot fetching, and Discord embed delivery.

### Disaster Recovery
Automated GitHub backups every 12 hours preserve: skills, cron definitions, sanitized configs, monitoring code, scripts, and local Hermes patches. The backup repo can rebuild the entire agent setup from scratch.

---

## 🗺️ The Task Routing Model

Every request that hits the system goes through a multi-factor routing decision. It's not just "is it read-only?" — it's a layered evaluation that considers **capability fit**, **permission requirements**, **cost efficiency**, and **privacy sensitivity**.

### The Decision Framework

```
User Request → Discord/Telegram
                    │
                    ▼
         ┌──────────────────────┐
         │ 1. PERMISSION CHECK  │
         │  What tools does     │
         │  this task need?     │
         └──────┬───────────┬───┘
                │           │
        Read-only       Mutation
        (search,        (write files,
         analyze,        restart services,
         summarise)     SSH, git push)
                │           │
                │           ▼
                │    ┌──────────────┐
                │    │   GENGAR     │
                │    │  (cloud)     │
                │    │  gpt-5.4     │
                │    │  Full access │
                │    └──────────────┘
                ▼
     ┌──────────────────────┐
     │ 2. CAPABILITY CHECK  │
     │  Is there a Ditto    │
     │  model that fits?    │
     └──────┬───────────┬───┘
            │           │
       Yes —            No —
       right model      need frontier
       exists           reasoning
            │               │
            ▼               ▼
     ┌──────────────┐ ┌──────────────┐
     │ 3. COST CHECK│ │   GENGAR     │
     │  Cloud call   │ │  (cloud)     │
     │  worth it?    │ │  gpt-5.4     │
     └──┬───────┬───┘ └──────────────┘
        │       │
   Cheap/     Expensive/
   trivial    complex
        │       │
        ▼       ▼
 ┌──────────┐ ┌──────────────┐
 │  DITTO   │ │   GENGAR     │
 │ (local)  │ │  (cloud)     │
 │  $0/call │ │  frontier    │
 └──────────┘ └──────────────┘
```

### Layer 1: Permission Check — "What can this task touch?"

This is the hard wall. If a task requires **any mutation capability** — file writes, Docker restarts, SSH access, Git pushes, cron modifications — it's Gengar or nothing. Ditto literally cannot do these things; its toolset is deliberately stripped of all mutation tools.

| Permission Level | Tools Available | Routed To |
|-----------------|----------------|-----------|
| **Read-only** | web_search, file reads, system polls, Docker inspect, API queries, image analysis | Ditto or Gengar (next layers decide) |
| **Mutation** | file write/edit, Docker control, SSH, git push, cron management, service restart | **Gengar only** |

### Layer 2: Capability Check — "Can a local model handle this?"

Even for read-only tasks, not all are created equal. The question isn't just "can Ditto read this?" — it's "do we have a local model with the right cognitive profile for this task?"

| Task Profile | Cognitive Requirement | Best Ditto Lane | Local Viable? |
|-------------|----------------------|-----------------|----------------|
| Quick lookup / FAQ | Low reasoning, fast response | `ditto-fast` (Gemma 4) | ✅ Always |
| Summarization / explanation | Medium reasoning, long context | `ditto-default` (Qwen 3.6 MoE) | ✅ Default choice |
| Research / deep reading | High reasoning, very long context | `ditto-research` (Qwen 3.6 Q8) | ✅ Yes |
| Code review / architecture | Domain expertise, pattern matching | `ditto-coder` (Qwen 3 Coder MoE) | ✅ Yes |
| Multi-step analysis | Heavy reasoning, chain-of-thought | `ditto-heavy` (Llama 4 Scout) | ⚠️ Slower but capable |
| Frontier reasoning | Novel problem-solving, complex chains | *None — escalate to Gengar* | ❌ No |
| Multi-agent orchestration | Spawning sub-agents, delegation | *None — escalate to Gengar* | ❌ No |
| Creative generation | High-quality prose, structured output | *Cloud usually better* | ⚠️ Hit or miss |

The model lane is selected **automatically** by Ditto's Hermes profile based on the task — the user just says what they want, and the right model is loaded.

### Layer 3: Cost Check — "Is this worth a cloud call?"

When a task *could* go to either, the third layer evaluates whether the cloud cost is justified. This matters at scale — a daily briefing that runs 7 days a week adds up.

| Task Class | Cloud Cost | Ditto Cost | Decision |
|-----------|-----------|-----------|----------|
| Morning digest (daily, 5-10K tokens) | ~$0.02-0.05/day | $0/day | → Ditto |
| Quick question (1-3K tokens) | ~$0.005-0.01 | $0 | → Ditto |
| System log analysis (bulk, daily) | ~$0.10-0.30/day | $0/day | → Ditto |
| Incident correlation (rare) | ~$0.05 | $0 | → Ditto first, escalate if needed |
| Complex orchestration (multi-step, 10K+ tokens) | ~$0.10-0.25 | N/A (can't do it) | → Gengar |
| GitHub PR / code changes | ~$0.05-0.15 | N/A (needs mutation) | → Gengar |
| Frontier reasoning (novel problems) | ~$0.15-0.50 | Not viable locally | → Gengar |

**The heuristic:** if Ditto can do it, Ditto *should* do it. The cost saving isn't dramatic per-call, but it compounds. More importantly, it keeps cloud API keys out of routine task processing — reducing the attack surface for credential exposure.

### The Ditto-to-Gengar Escalation Pattern

Even when Ditto handles the initial request, it knows its limits. When it encounters something it can't do:

1. **Draft mode** — Ditto produces a precise, structured prompt that captures all context it's gathered
2. **Handoff** — The prompt is passed to Gengar with the original user context intact
3. **Execute** — Gengar picks up where Ditto left off, no context lost

This isn't a failure mode — it's by design. Ditto is the **first line**, not the fallback.

### Routing Table Summary

| Task | Permissions | Ditto Model? | Cost Decision | → Route |
|------|------------|-------------|---------------|---------|
| "What's running on Unraid?" | Read-only | ✅ `ditto-fast` | Trivial | **Ditto** |
| "Summarize this Frigate alert" | Read-only | ✅ `ditto-default` | Trivial | **Ditto** |
| "Research MoE model benchmarks" | Read-only | ✅ `ditto-research` | Medium | **Ditto** |
| "Review this Python script" | Read-only | ✅ `ditto-coder` | Trivial | **Ditto** |
| "Analyze why Frigate crashed" | Read-only | ✅ `ditto-heavy` | Medium | **Ditto** |
| "Restart the Frigate container" | **Mutation** | N/A | N/A | **Gengar** |
| "Create a GitHub repo" | **Mutation** | N/A | N/A | **Gengar** |
| "Build me a new dashboard" | **Mutation** | N/A | N/A | **Gengar** |
| "Write an Obsidian research note" | **Mutation** | N/A | N/A | **Gengar** |
| "Design the family logistics pipeline" | Complex planning | ❌ Frontier needed | High | **Gengar** |
| Morning digest (cron) | Read-only | ✅ `ditto-default` | Compounds | **Ditto** (Alakazam) |
| Backup job (cron) | **Mutation** | N/A | N/A | **Gengar** (cron) |

> **The guiding principle:** Ditto handles what it can do well, locally, for free. Gengar handles everything Ditto can't — mutations, frontier reasoning, and complex orchestration. The boundary is crisp, the handoff is clean, and the user never has to think about it.

---

## 🔮 Future Plans & Next Steps

### Near Term — Expanding Beyond Infrastructure
- **Ditto Dashboard** — Dedicated Ditto-specific control plane separate from the ops cockpit
- **Model benchmarking suite** — Automated A/B testing of local models for different task types
- **Expanded local inference** — Test additional MoE models as they become available (Qwen 4, Llama 4 MoE)
- **Voice integration** — TTS/STT for agent interaction
- **UniFi integration** — Deeper network monitoring and client anomaly detection

### Medium Term — Multi-Agent Collaboration
- **Direct agent-to-agent handoff** — Complex workflows split across specialized agents
- **Home Assistant bridge** — Agent-driven smart home control with safety guardrails
- **Knowledge base RAG** — Local document ingestion and retrieval over the Obsidian vault
- **Enhanced dashboards** — Real-time WebSocket updates, historical trend graphs

### Exploratory
- **Additional agent personas** — Specialized agents for specific domains
- **Federated model routing** — Dynamic task-to-model assignment based on capability requirements
- **Local fine-tuning** — Custom LoRA adapters on Ditto for domain-specific tasks

---

## 🌐 Beyond the Homelab — Phase 1 of Assimilation

The homelab automation is where it started, but it's not where it ends. The same infrastructure — always-on agents, multi-platform messaging, local + cloud model routing, cron-driven intelligence, and Obsidian-backed persistent memory — extends naturally into every corner of daily life. Here's what's on the roadmap.

### 📡 World Event Monitoring & Work Impact Analysis

**Research agents that watch the world so you don't have to.**

- Monitor regulatory changes, cybersecurity incidents, and technology policy shifts that could impact enterprise cloud architecture
- Track competitor movements, funding rounds, and product launches in the AI governance and cloud security space
- Cross-reference breaking news against your current projects and flag "this affects you" items
- Deliver a concise daily briefing: *what happened, why it matters to your work, and what (if anything) you should do about it*

### 💼 Consulting Opportunity Discovery

**Agents that find opportunities while you focus on delivering.**

- Monitor government RFPs, consulting marketplaces, and industry forums for relevant opportunities
- Match requirements against your expertise profile (cloud architecture, AI governance, Zero Trust, identity platforms)
- Draft initial qualification summaries and flag high-fit opportunities for review
- Track submission deadlines so nothing falls through the cracks

### 👨‍👩‍👧 Family Logistics & Communication

**The assistant that keeps family life running.**

- Morning briefing to each family member via SMS/Discord: *their schedule for the day, weather forecast, and anything they need to bring*
- School calendar integration — early dismissal days, picture day, permission slip deadlines
- Activity coordination — sports schedules, practice times, who needs to be where
- "Dinner decision agent" — factors in schedules, dietary restrictions, what's in the fridge, and weather (grill or stove?)

### 🏠 Intelligent Home Automation

**Go beyond "lights on at sunset" to context-aware home intelligence.**

- Arrival detection — geofence triggers: disarm security, adjust thermostat, preheat based on dinner plan
- Energy optimization — time-of-use rate awareness, solar production monitoring, EV charging scheduling
- Preventative maintenance — dryer lint alerts, HVAC filter reminders, water leak detection escalation
- Pet safety — ingredient scanning against known triggers before purchasing food or treats (already in design)

### 💰 Personal Finance & Admin

**Automate the tedious stuff.**

- Receipt capture → categorization → expense report draft
- Subscription audit: what you're paying for, what you haven't used in 3 months
- Bill negotiation research: "here's what competitors charge, here's a script for retention"
- Tax prep document collection throughout the year instead of the March scramble

### 📚 Continuous Learning & Research

**An AI-powered research assistant that builds knowledge over time.**

- "Deep dive" research agents that spend hours reading papers, extracting insights, and building structured summaries
- Cross-reference new information against your existing Obsidian knowledge base
- Generate Anki/flashcard decks from anything you're learning
- Weekly "what's new in your field" synthesis from arXiv, blogs, and industry publications

### The Pattern

Every one of these use cases follows the same architecture that already works:

```
Trigger (time / event / user request)
    → Agent evaluates: Ditto-local or Gengar-cloud?
    → Agent gathers context (Obsidian, web search, APIs)
    → Agent executes or drafts for approval
    → Delivers to the right platform (Discord, SMS, email)
    → Records outcome for future reference
```

The homelab was the sandbox. Now it's the launchpad.

---

## 🔒 Privacy & Security

This is a **local-first, privacy-respecting** setup:

- ✅ All AI inference runs locally on Ditto (Ollama) for sensitive tasks
- ✅ SearXNG provides anonymous, self-hosted web search
- ✅ No public exposure — all services are LAN-only
- ✅ Outbound-only notifications (Discord webhooks/bots)
- ✅ Least-privilege credentials with environment-variable isolation
- ✅ Ditto profile has zero access to cloud API keys, SSH credentials, or mutation tools
- ✅ Automated secret scanning on DR backups
- ✅ No personal data, IPs, or hostnames in public documentation

---

## 📂 Repository Structure

```
ai-home-lab-part-deux/
├── README.md                        ← You are here
├── assets/
│   ├── architecture_diagram.png     ← Dark-themed SVG architecture diagram
│   ├── architecture_diagram.html    ← Interactive diagram (open in browser)
│   └── gengar_cockpit.png           ← Ops dashboard screenshot
└── docs/                            ← Detailed documentation (coming soon)
```

---

*Built with ❤️ and way too many late nights. Powered by Hermes Agent, Ollama, and an ever-growing collection of Pokémon-themed codenames.*
