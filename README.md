# AI Home Lab - Part Deux

Overview of my AI homelab and how I use local models, hosted models, agents, dashboards, and routing in practice.

This did not start as an efficiency project. It started because I wanted to see whether a local model setup was actually for me, how far I could push it, and where it made sense to keep things local versus hand them off to a stronger hosted model. What followed was a year of building, breaking, rebuilding, and learning that the models themselves are only part of the story. The routing, isolation, and observability around them matter just as much. This repo covers what is running today, how the pieces fit together, what tools are involved, and the lessons that came out of it.

## Then and now

The stack did not start where it is now. Here is the arc at a glance.

| | Where it started | Where it landed |
| --- | --- | --- |
| **Inference** | Ollama in Docker on Unraid. GPU passthrough was fragile, restarting a container meant reloading models, and multi-instance setups were unreliable. | llama-server running via systemd on a dedicated Linux box. Single persistent service, clean restarts, journald logging, 51% faster than the Ollama setup on the same hardware. |
| **Model lanes** | Five Ollama aliases running simultaneously: fast models, heavy models, coder models, research models. Most were never actually used. | One active lane serving a single high-quality model with a generous context window. Additional lanes for coding and research are planned, but only with a clear use case. |
| **Agent isolation** | Profiles were "read-only" via configuration flags. Trusted the config to enforce boundaries. A read-only agent found and used credentials it should not have had access to. | Separate OS-level users per local profile. SSH-only terminal access for restricted agents. Watchdog automation that monitors for credential exposure. Scoped credentials enforced at the environment level, not just the profile config. |
| **Observability** | No centralized logging. No metrics. When something broke, I found out by noticing it was broken. Debugging meant SSHing into multiple machines and grepping logs. | Loki for log aggregation, Prometheus for metrics, Grafana for dashboards. Inference latency, throughput, container health, and agent status are visible at a glance. Alerts fire before I notice the problem. |
| **Routing** | One model trying to do everything. Local models pushed beyond what they were good at. No clear split between read-only analysis and mutation. | Mutation goes through a single orchestrator with cloud-hosted models. Read-only analysis stays local. A local model does the first pass well enough that the next step has less work to do. |
| **Hardware** | One Mac Mini running everything plus Ollama contending for resources on the Unraid server. | Mac Mini handles orchestration. Dedicated Linux box handles inference. Unraid handles Docker services. Each machine has one job. |
| **Knowledge** | Context lived in chat history. Restart a session, lose the context. | Obsidian as the durable knowledge layer. Notes, decisions, changelogs, and operating details survive session restarts. |

## What is running today

The current setup is split across five named pieces, plus a supporting observability stack.

### Gengar

Gengar is the primary orchestrator. It runs Hermes Agent on a Mac Mini and is the only part of the system with broad mutation access. It handles file and configuration changes, cron job management, GitHub workflows, service inspection and control, Obsidian vault updates, and bridging Discord and Telegram. If something needs to change state, it goes through Gengar, which uses cloud-hosted models for tasks that need stronger reasoning, wider tool access, or direct mutation capability.

### Ditto

Ditto handles local inference and read-only analysis on a dedicated Linux box with unified memory. It runs the kinds of tasks that benefit from staying local: summarizing logs and alerts, reading dashboards and system state, research and synthesis, first-pass incident analysis, and drafting prompts for follow-up work. Ditto also powers a Discord bot for local image generation on the same hardware and inference runtime.

The runtime is llama-server (llama.cpp), managed via systemd as a persistent service. This was a deliberate migration away from Ollama. For a multi-agent setup where uptime matters, a systemd-managed inference service with a single well-chosen model turned out to be simpler, faster, and more predictable than running multiple model aliases through a containerized orchestrator. Benchmarking the same model under both runtimes showed a 51% performance gain under llama-server.

Currently Ditto serves one active model lane. Additional lanes for coding and research tasks are planned, and the hardware has the headroom with model files already on disk. The move to llama-server made it straightforward to spin up separate endpoints for different workloads without the overhead that made multi-model setups painful under the previous runtime.

### Gengar Cockpit

Gengar Cockpit is the operations dashboard. It brings together host health, Ditto telemetry and model performance, cron job status, backup state, service checks, and project-specific signals. Once you add automation and multiple agents, you need a fast way to see whether the system is healthy, and the Cockpit is that surface.

### Alakazam

Alakazam handles scheduled reporting: morning digests and background summaries that pull together overnight state into something scannable. The collection and summarization are split into separate steps: a collector gathers structured state first, and a second pass reads that data and turns it into a digest. This keeps the reporting layer cleaner and makes it easier to reason about what the model is actually summarizing. Alakazam is deliberately constrained, running as a read-only profile with limited tool access, scoped credentials, and watchdog automation that monitors for unexpected behavior.

### Observability stack

The observability layer is built on Loki, Prometheus, and Grafana, running alongside the rest of the homelab services. It provides centralized log aggregation across hosts and containers, metrics on inference performance and system health, and dashboards that surface what is happening without needing to SSH into individual machines. This stack was not part of the original design, but it became necessary once the system grew past the point where checking logs manually was a viable debugging strategy.

## Architecture at a glance

The architecture is simple in principle. A single orchestrator handles all mutations while local models handle the work they are good at and cloud-hosted models handle tasks that need frontier reasoning. Scheduled jobs manage recurring checks and reporting, a dedicated observability layer provides visibility into all of it, and Obsidian acts as the long-term knowledge layer.

![AI Home Lab architecture](assets/architecture_diagram.png)

## How work gets routed

The routing model comes down to three questions:

1. Does this need to change something? If yes, it goes to Gengar.
2. If it is read-only, can a local model do it well enough? If yes, it usually goes to Ditto.
3. If a local model can do it, is there still a good reason to use cloud? If no, keep it local.

A few real examples:

| Task | Route | Why |
| --- | --- | --- |
| Summarize an alert or dashboard | Ditto | Fast, local, read-only |
| Review a config or script | Ditto first | Good first-pass analysis |
| Restart a service or edit files | Gengar | Needs mutation access |
| Create or update repo content | Gengar | Needs Git and file access |
| Deep orchestration or multi-step tool work | Gengar | Better tool and reasoning coverage |

The point is not to build a dramatic multi-layer routing engine. It is to keep the boundary clear.

## Infrastructure and tools

**Hermes Agent** is the backbone for the agent workflows. It ties together profiles, cron jobs, messaging platforms, skills, persistent memory, tool access, and automation glue. Without it, this would be a pile of scripts and one-off commands.

**llama.cpp (llama-server)** provides the local inference runtime on the dedicated Linux box, running as a systemd service with a single high-quality model and a generous context window.

**Obsidian** is the long-term knowledge layer where notes, decisions, project context, and operating details live so the system is not relying only on transient chat history.

**Docker and Unraid** provide the underlying homelab service layer, roughly two dozen containers covering media automation, home automation, monitoring, search, and supporting infrastructure.

**SearXNG** gives agents a privacy-respecting search path without forcing everything through a commercial API.

**Loki, Prometheus, and Grafana** provide centralized logging, metrics, and dashboards for the entire stack.

**Frigate** handles local NVR with object detection for cameras, and **ComfyUI** powers local image generation workflows on the inference box.

## Lessons learned

These are the things I would tell someone starting a similar project. Each one represents something I had to break before I understood it.

### Ollama is great until it is not (for this kind of setup)

I started with Ollama because everyone starts with Ollama. It is the obvious first choice: simple to install, easy to pull models, works out of the box. For running a single model and chatting with it locally, Ollama is excellent.

For running a multi-agent system where models need to be available as reliable services, the friction adds up. Docker GPU passthrough on Unraid was fragile, model management felt bolted onto a container orchestration model that was not designed for it, and restarting a container meant waiting for models to reload. Running multiple model aliases meant juggling memory in ways that were hard to predict.

The migration to llama-server, running directly via systemd on a dedicated Linux box, changed the reliability equation: one well-chosen model, one persistent service, no container layer between the agent and inference. The systemd integration means the service restarts cleanly, logs go to journald, and resource limits are explicit. Benchmarking the same model under both runtimes showed a 51% performance gain under llama-server compared to the Ollama setup. That is the difference between a local model feeling usable and feeling like you are waiting on it.

The migration is less convenient up front than `ollama pull`, but it is dramatically more predictable to operate. If you are exploring local models for the first time, start with Ollama. If you are building something that needs to run unattended and survive reboots, a thinner runtime with native service management may be a better fit.

### Read-only is harder than it sounds

I configured a profile to be read-only. It was supposed to have limited tool access, scoped credentials, and no ability to mutate anything. I trusted the configuration. It found a key anyway.

The specifics are less important than the lesson: a determined agent with filesystem access and tool access can discover things you forgot existed. Credentials in environment files, keys in config directories, tokens cached where you did not expect them. A read-only profile can still read them, and if it can read them, it can use them. This was not a failure of the agent. It was a failure of my assumption that restricted tools and read-only mode were the same thing as actual isolation. They are not.

### Locking down local agents

The incident forced a rethinking of how the profiles are isolated. Each local-model profile now runs under a separate OS-level user, so filesystem permissions provide a real boundary instead of just a configuration flag. Restricted profiles use SSH-only terminal access, preventing direct local execution. Watchdog automation monitors for unexpected credential exposure and permissive key usage, and each profile gets exactly the secrets it needs and nothing more, enforced at the environment level rather than just in the profile config.

Local models feel safe because they are local, but that makes permissions more important, not less. A local model with broad filesystem access is not safe because it is on-premises. It is dangerous because it is on premises and someone forgot to scope its access.

### Routing matters more than models

I spent a lot of time early on chasing better models, bigger context windows, better benchmark scores. What actually improved the system was getting the routing right. The split between mutation and read-only matters more than which specific model is behind either role. Once the boundary is clear, you can swap models behind either side without changing the architecture.

Local models became much more useful once I stopped looking for one perfect model and started thinking about handoffs. A local model does not need to do everything. It needs to do the first pass well enough that the next step has less work to do, whether that next step is a human review or a handoff to a stronger model.

### Observability is not optional

The original stack had no observability layer. When something broke, I found out when I noticed it was broken, sometimes hours later, sometimes days. Once you have multiple agents running scheduled jobs, two dozen containers across multiple machines, and automation that fires without anyone watching, checking logs manually stops being a viable strategy. You need centralized log aggregation so you are not SSHing into three different machines to piece together what happened, metrics on inference latency and throughput so you can tell when the local model is degrading, and dashboards that surface health at a glance. Loki, Prometheus, and Grafana were not in the original scope, but flying blind is not sustainable for a system that runs unattended.

### One model lane is enough (for now)

When I was running multiple model aliases under Ollama, I had fast models, heavy models, coder models, and research models all running simultaneously. I assumed I needed all of them. The move to llama-server forced a different question: what if you just pick one really good model and run it well?

A single high-quality model with a generous context window covers most of what the local side actually needs to do. The specialized lanes sounded good in theory, but in practice they added complexity without adding enough value to justify it. That does not mean the other lanes will never come back. The hardware has the headroom, and additional endpoints for coding and research models are planned, but standing them up will be a deliberate choice with a clear use case rather than a default. Start with one well-chosen model, prove the value, and add lanes when you can point to a specific gap.

## Current capabilities

Scheduled morning digests and automated reporting. Alert analysis and summarization. Local-first research and synthesis. Dashboard-based visibility into host and model state. Git-backed configuration and recovery material. Persistent project context through Obsidian. Clean handoff between local analysis and cloud-hosted execution. Verified isolation between read-only and mutation-capable profiles.

## What I am still exploring

Current areas I am pushing on: additional local model lanes for coding and research tasks, a consolidated service dashboard that brings together homelab health, inference telemetry, and agent status in one view, more useful reporting that does not sound machine-generated, practical use of voice and scheduled background agents, deeper benchmarking of local models on real tasks instead of benchmark scores, and figuring out where local really earns its keep versus where hosted is still the better answer.

## Privacy and scope

This repo is intentionally sanitized. It does not include IP addresses, hostnames, secrets or tokens, environment-specific access details, private usage data, or specific commercial model or provider names. It is meant to show the shape of the system, not every private implementation detail.

## Technology glossary

A quick reference for the main technologies mentioned here.

- [Hermes Agent](https://github.com/NousResearch/hermes-agent): agent orchestration framework covering profiles, cron, tools, skills, and memory
- [llama.cpp](https://github.com/ggerganov/llama.cpp): local LLM inference runtime
- [Obsidian](https://obsidian.md): long-term knowledge management
- [Docker](https://www.docker.com): container runtime
- [Unraid](https://unraid.net): homelab operating system and NAS
- [SearXNG](https://docs.searxng.org): privacy-respecting metasearch engine
- [Loki](https://grafana.com/oss/loki/): log aggregation
- [Prometheus](https://prometheus.io): metrics collection and alerting
- [Grafana](https://grafana.com/oss/grafana/): observability dashboards
- [Frigate](https://frigate.video): local NVR with object detection
- [ComfyUI](https://github.com/comfyanonymous/ComfyUI): image generation workflow engine
- [systemd](https://systemd.io): Linux service manager
- [GitHub](https://github.com): version control and source hosting

## Repository structure

```text
ai-home-lab-part-deux/
├── README.md
└── assets/
    ├── architecture_diagram.png
```

## Notes

The names are a little ridiculous on purpose. The actual goal here is not to make a flashy AI system. It is to better understand what local AI, hosted AI, routing, automation, and agent workflows look like when they are attached to real infrastructure and used regularly. The lessons that came out of it, about isolation, observability, and picking the right runtime, were the real output. The stack is just what I built along the way.
