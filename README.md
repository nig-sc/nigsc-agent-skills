# Agent Skills for the NIG supercomputer

[Agent Skills](https://www.anthropic.com/news/donating-agent-skills-to-the-agentic-ai-foundation)
(`SKILL.md`) that teach an AI agent how to run work on the **NIG
supercomputer** operated by DDBJ at the National Institute of Genetics
(Mishima, Japan). They work with any agent harness that reads `SKILL.md`,
such as Claude Code, Claude Science, Codex CLI, Gemini CLI and GitHub
Copilot.

This repository was renamed from `nig-sc/claude-science-skills`; the old URL
redirects here. They encode the cluster's usage conventions —
SLURM scheduling, the `/usr/local/biotools` BioContainers/apptainer image
collection, and the pre-installed databases — so an agent can submit jobs
correctly instead of guessing.

The NIG supercomputer is split into two **physically separate** networks, so
there is one skill per zone:

| Skill | Zone | Use it for |
|-------|------|-----------|
| [`nig-general-analysis`](./skills/nig-general-analysis/) | General analysis (一般解析区画) | Open, non-personal-genome work. Direct SSH, shared SLURM partitions (`epyc`, `rome`, …), public reference databases. |
| [`nig-personal-genome`](./skills/nig-personal-genome/) | Personal genome analysis (個人ゲノム解析区画) | Controlled-access human genome data. SSL-VPN + 2FA access, `-pg` accounts, node-lease + shared L40S GPU SLURM, Parabricks, NBDC-compliant data handling. |

Each skill is self-contained and covers the same three functional areas for its
zone: **(a) SLURM / scheduling, (b) tools via `/usr/local/biotools`,
(c) databases & data transfer**.

## Design principle: the live machine is the source of truth

The published DDBJ documentation can lag the running cluster (general-zone
compute nodes are lent out to labs, so partitions appear and disappear; database
and tool versions change). Both skills therefore instruct the agent to **trust
live `sinfo` / `ls` output over any documented table**, and treat the official
docs as background/orientation. Links to the authoritative DDBJ pages are
included at the bottom of each skill.

## What these skills deliberately do NOT contain

`/usr/local/biotools` holds **2,000+ tools and 90,000+ container images**, and
the database trees are large and versioned. The skills do **not** enumerate
them — they teach the *layout* and the *search/run idiom* (`ls
/usr/local/biotools/<letter>/ | grep <tool>`, `apptainer exec <img> <cmd>`) so
the agent finds what it needs at run time. This keeps the skills small and
prevents them from going stale.

## Installing

Install only the skill for the zone you use.

**Claude Code.** This repository is a Claude Code plugin marketplace, one plugin
per zone:

```
/plugin marketplace add nig-sc/nigsc-agent-skills
/plugin install nig-general-analysis@nigsc-agent-skills
/plugin install nig-personal-genome@nigsc-agent-skills
```

`/plugin marketplace update nigsc-agent-skills` fetches later fixes.

**Claude Science.** Point your Claude Science skill registry at this repository,
or copy a skill directory into your skills location.

**Other harnesses.** Copy the skill directory (for example
`skills/nig-general-analysis/`, which holds a single `SKILL.md`) into the
skills directory your harness reads; see your harness's documentation for
where that is.

Each skill directory contains a single `SKILL.md` with YAML front-matter
(`name`, `description`) plus the guidance body.

## Zones at a glance

**General analysis zone** — direct SSH to a login node (e.g. `nig-a003`),
shared SLURM (`epyc` default), tools from `/usr/local/biotools`, public DBs
under `/usr/local/resources` and `/usr/local/shared_data`.

**Personal genome zone** — a hardened, NBDC-compliant, **separate network**:
reached only through SSL-VPN (FortiClient) + two-factor auth, accounts carry a
`-pg` suffix, compute is leased per-node with a shared SLURM only for the scarce
NVIDIA **L40S GPU** nodes (`--partition=l40s --account=l40s`), and GPU genomics
runs on **Parabricks**. It does not share the general zone's database layout.

## Source

Authored against the DDBJ NIG supercomputer (2025 system) documentation at
<https://sc.ddbj.nig.ac.jp/> and verified against the live general-analysis
zone. Contributions/corrections welcome — especially verified specifics from
*inside* the personal-genome zone (leased-node database paths, live partition
list), which are marked as TODO in that skill.
