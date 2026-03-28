# Azrael Security -- Master Handoff Document
**Schema version:** 1.0 (locked)
**Date:** 2026-03-28
**Supersedes:** azrael-handoff-q1-s033.md (Session 33)

---

## SESSION PROCEDURE (NON-NEGOTIABLE)
Session type must be declared at open: INFRA / RESEARCH / LEARNING / MIXED
Session close trigger: "session close" or "wrap up"
On close: ask five questions → output changed sections only → Claude Code applies surgical edits
INFRA session → always update Section 2 and Section 3
RESEARCH session → always update Section 3
ALL sessions → update Section 5 and Section 7
Output file: azrael-handoff-q1-sNNN.md
Commit: git commit -m "docs: session handoff YYYY-MM-DD SNNN"

---

## SECTION 0 -- STABLE REFERENCE

### Node Registry
| Name | Role | OS | WireGuard IP | LAN IP | SSH |
|---|---|---|---|---|---|
| **Cerberus** | Edge / infra node | Arch Linux (headless) | 10.0.0.1 (hub) | 192.168.1.251 (static) | `ssh cerberus` or `ssh -p 2121 foreverlx@192.168.1.251` |
| **NightForge** | Operator workstation | Arch Linux | 10.0.0.3 | 192.168.1.145 (DHCP) | `ssh nightforge` |
| **Tairn** | Attack node / C2 | NixOS 24.11 | 10.0.0.4 | 192.168.122.230 (libvirt NAT) | `ssh tairn` (WireGuard only, 10.0.0.4) |
| **Hermes** | Redirector VM | Alpine Linux 3.23.3 | 10.0.0.5 | 192.168.122.200 (static, libvirt NAT) | `ssh foreverlx@10.0.0.5` (WireGuard only) |
| **iPhone** | Mobile WireGuard client | iOS | 10.0.0.2 | -- | -- |

**WireGuard topology:** Hub-and-spoke via Cerberus. Hairpin routing enabled. ListenPort 51820 (Cerberus), 51821 (NightForge), 50555 (Tairn), dynamic (Hermes).

**WireGuard public keys:**
| Node | Public Key |
|---|---|
| Cerberus | SRmiMpAS6fVYvRKmVd4cjT6wDLNQqecWLktUNB28XmM= |
| iPhone | MSXEjVdA+sXW3hbkr4/iwxn7mWaXD96xyCwyAOd1GQ4= |
| NightForge | QsV69ujC9Er3ZIugFIrYVKjzuilhQP72LMT964jC8DQ= |
| Tairn | ZZKPbobyVPpIS05/iHDGB4pDe7bKDwm9NK6eEGfr6hM= |
| Hermes | 1MJchWRXNchXxm6BDHkyYBxetGzn8IVGME3As+aSjGg= |

**WireGuard topology rules (locked):**
- Tairn initiates to Cerberus only -- Cerberus has no endpoint for Tairn in wg0.conf
- NightForge SSH config uses 10.0.0.4 (WireGuard), not 192.168.122.230 (libvirt NAT)
- Cerberus cannot reach 192.168.122.x -- libvirt NAT is only reachable from NightForge
- All spokes must have AllowedIPs = 10.0.0.0/24 for Cerberus peer — not 10.0.0.1/32 — or new mesh peers will be unreachable

### Key Paths

**Cerberus:**
```
~/veil/
~/.config/containers/systemd/
/etc/containers/systemd/
~/caddy/Caddyfile
~/homepage/config/
/etc/wireguard/wg0.conf
/etc/nftables.conf
~/.config/systemd/user/nightforge-shield.service
~/scripts/suricata/suricata-shield.sh
~/scripts/noc/
~/noc-status/
/etc/pihole/hosts/local.lan
/etc/containers/secrets/pihole.env
```

**NightForge:**
```
~/Github/
~/Github/veil/
~/Github/veil/docs/skills/
~/Github/veil/docs/troubleshooting.md
~/Github/security-research/
~/Github/nightforge/
~/Github/azraelsec.dev/
~/Documents/azrael-ops/
~/Documents/azrael-vault/
~/lab/VMs/
~/Repos/3rd-party/
~/Repos/3rd-party/ccsm/
~/Repos/3rd-party/llmfit/
/etc/wireguard/wg0.conf
/etc/nftables.conf
/etc/systemd/system/vnet0-bridge-fix.service
~/.ssh/config
~/.claude/settings.json
~/.claude/CLAUDE.md
~/.claude/hooks/block-destructive.sh
~/.claude/agents/
~/.claude/skills/strategic-compact/
~/.claude/commands/
~/.claude/plugins/cache/trailofbits/
~/.claude/plugins/claude-hud/config.json
~/.claude.json
~/.config/Claude/claude_desktop_config.json
~/.config/fastfetch/config.jsonc
~/.config/starship.toml
~/.config/operator-terminal/operator-init.sh
```

**Tairn:**
```
/etc/nixos/configuration.nix
~/Mythic/
```

**Hermes:**
```
/etc/wireguard/wg0.conf
/etc/nginx/nginx.conf
/etc/network/interfaces
```

### Locked Technical Decisions
| Decision | Choice | Rationale |
|---|---|---|
| C2 framework | Mythic | Industry recognized, rich agent ecosystem |
| Primary agent | Poseidon (Go, Linux) | Aligns with Linux/kernel focus |
| Secondary agent | Apollo (Windows) | Installed on Tairn, available for Windows engagements |
| C2 network access | WireGuard-only (10.0.0.x) | Mirrors real engagement OPSEC |
| C2 VM OS | NixOS 24.11 | Declarative, reproducible, portfolio signal |
| Container runtime (Cerberus) | Podman rootless Quadlets | Security, no daemon |
| Container runtime (Tairn) | Docker | Mythic officially supports Docker |
| VM storage location | ~/lab/VMs/ | OPSEC + maintenance |
| WireGuard topology | Hub-and-spoke via Cerberus | Always-on edge node = natural hub |
| Shell prompt | Starship (all nodes) | Cross-node consistency, version controlled |
| Search engine | SearXNG (self-hosted) | Local-first, multi-engine, OPSEC aligned |
| DNS | Pi-hole on Cerberus | .lan domains + ad blocking |
| TLS | Caddy local CA | Simple, automatic for .lan |
| Flagship research | Container boundary weaknesses | Real infra, differentiating, connects RE to infra |
| Flagship research question | In rootless Podman deployments on RHEL-family Linux, does nvidia-container-runtime execute OCI hooks with privileges exceeding the container's user namespace mapping, and can a compromised agentic workload supply attacker-controlled input to that hook execution to achieve code execution in the host's namespace? | Locked S27c. Builds on CVE-2025-23266 (NVIDIAScape). Threat model: agentic workloads in GPU-enabled multi-tenant environments. |
| Research threat model | Compromised agentic workload in GPU-enabled multi-tenant environment | CI/CD dropped as primary framing. Agentic workloads are the 2026 deployment pattern confirmed by NemoClaw announcement. |
| ACLGuard | Archived | No passion, AI-built, not representative |
| Handoff storage | Private Gitea repo (azrael-ops) | Operational notes -- not public, version controlled |
| Career plan horizon | 12 months, hard review at end | Realistic, prevents over-planning |
| North star role | Vulnerability Researcher (runZero-level) | Practitioner-led culture, research-driven, quality over ease |
| Language stack sequencing | Python then C then Go then Rust (deferred) | Python closes nearest gaps, C required for exploitation depth, Go for north star alignment, Rust not before Q4 review |
| Write-first rule | Claude reviews, never generates, for programming skill work only | Gap closes through writing, not reading Claude output. Does not apply to copy/prose. |
| Q1 simultaneous tracks | Python scripting + Networking hands-on + CWL/CRTA | Reinforce each other, motivation maintained, early warning flag if any drops to zero for 2 weeks |
| Skill file storage | Version controlled in ~/Github/veil/docs/skills/ + uploaded to Claude project | Zero friction in sessions, version history in git |
| Resume strategy | One Tier 1 base (base-tier1-resume.docx), copy and edit per application | General resumes optimized for no one; specific resumes win |
| Resume tiers to build now | Tier 1 only | Tier 2 waits for public container boundary research artifact; Tier 3 waits for prerequisites in career plan |
| Claude Code node | NightForge only | Only node where code is written; Cerberus is headless infra, Tairn config requires manual operator execution |
| Claude Code default directory | ~/Github/ | Scoped to repos, not home directory |
| Personal site framework | Astro | Static output, zero JS by default, Content Collections for research writeups, markdown-first, no framework overhead |
| Personal site repo | azraelsec.dev (new, clean history) | Old personal-site repo archived |
| Personal site remotes | GitHub primary, Gitea mirror | GitHub is public portfolio face; Gitea is local backup. Consistent with veil/nightforge pattern. |
| Brand primary color | #8B6FEF | Committed purple-violet, distinctive in security space, maps to Azrael gatekeeper concept |
| Brand accent | #E6B450 | Muted amber, contrast without clash |
| Brand background | #0D0F1A | Near-black, not pure black |
| Brand primary text | #E6EAF0 | Slightly warm white |
| Brand secondary text | #8B93A6 | Muted blue-gray |
| Brand heading font | Inter | Clean, professional, widely supported |
| Brand monospace font | JetBrains Mono | Code and terminal content |
| Brand tagline | "Below the abstraction" | Systems-depth direction, earned |
| Brand site/repo name | azraelsec.dev | Locked |
| SVG generation | Claude AI (this interface), not Claude Code | Claude Code for integration only -- SVG drafting here is faster and produces equivalent quality |
| No em-dash rule | No em-dashes in any public-facing copy | Flagged during Session 22, fixed Session 23 |
| No "real engagement/real conditions" rule | Never use this framing in public-facing copy | Reads as marketing copy; infrastructure details carry the signal without narration |
| HackSpaceCon in copy | Never reference in public-facing copy until talk is accepted | Premature, reads as aspiration not signal |
| MCP config source of truth | ~/.claude.json | Claude Code reads this. ~/.config/Claude/claude_desktop_config.json mirrors it. |
| git MCP | Dropped | Redundant -- Bash tool + github MCP covers all local git operations |
| Model routing | Haiku: mechanical/commit tasks. Sonnet: default. Opus: deliberate deep reasoning only. | Pro plan context budget -- Opus burns 5x faster |
| Claude Code vs direct edit | Claude Code for multi-file pattern work. Direct Neovim for single-file mechanical changes. | Token efficiency, speed |
| Claude Code vs terminal ops | Infra ops (fastfetch, Mythic, Netdata config) are direct terminal work -- planned in chat, executed by Darrius. Claude Code adds nothing for single-node config changes. | Correct tool for the task |
| Cowork on NightForge | Functional via community port (claude-desktop-appimage). Use for knowledge work, not site builds. | Official Linux support not yet available |
| Cowork (official) | Deferred -- Someday | AUR reverse-engineered port is fragile. Not a Q1 priority. Will revisit when stable. |
| Session naming convention | Generated at session close by Claude | Eliminates mental overhead |
| Session close handoff workflow | Claude AI outputs changed sections only → Claude Code applies surgical edits to existing file, saves as new version, commits, pushes | Eliminates full document regeneration token cost. Claude Code filesystem MCP + bash handles mechanical file ops. Haiku-tier task. |
| Session opening protocol | Backlog review against schedule is standard practice | Established S28. Prevents priority drift and surfaces scheduling conflicts before work starts. |
| ECC installation approach | Option B -- manual clone, read hooks, port to shell scripts | Local-first, understand before automate, no CLAUDE_PLUGIN_ROOT dependency |
| Eyadkelleh/awesome-claude-skills-security | Dropped | Unaudited agents, payload lists have marginal research value, no institutional backing |
| Session manager | Agent not skill | Behavioral system -- actively manages state, not passive context |
| E-shaped specialization | Confirmed target structure | Three depth areas locked Session 26. Broad foundation + depth beats T-shape for high-performing teams. |
| E-shape depth areas | Red team infrastructure and operations / Threat intelligence and adversary tradecraft / Vulnerability research | Target profile, not current state. Each depth compounds with the others. Operational orientation is deliberate hedge against agentic AI shift. |
| E-shape horizon tracks | Quantum computing offensive security / Bio-engineering offensive security | Long-term parallel investments, not active depth areas. Named and intentional but not competing for current focus. |
| Networking Hands-On track | Foreground Block 1, not background | Load-bearing for CRTA, container research, and attack infra depth track |
| Claude AI account preferences | Set in Settings → Profile | Persists across all conversations, not just Azrael project |
| Noah Grayson partnership structure | Deferred until post-call | Vision alignment and role fit must be established before structure decision |
| .claude/ public visibility | Hooks, commands, settings tracked publicly in nightforge repo. Runtime data, credentials, session state gitignored. | hooks/commands/settings are deliberate operator choices worth showing. Runtime data excluded. azraelsec.dev writeup delivers the narrative. |
| research-writer WebFetch | Intentional | Research content requires inline reference fetching -- CVE advisories, papers, prior work |
| vuln-analyst WebFetch + WebSearch | Intentional | CVE lookups and reference material during analysis sessions. Outbound calls accepted for known research sources |
| Research-log MCP | No custom server -- filesystem MCP + Obsidian CLI covers the need | Filesystem MCP extended to azrael-vault; Obsidian CLI slash commands serve as structured logging interface |
| RAG | Someday/Deferred | Trigger condition: vault research content exceeds 100 interconnected notes or rg searches regularly miss relevant material. Reevaluate Q3/Q4. |
| Obsidian Sync | Deferred until quality laptop acquired | Cross-device need not present until then; vault lives on NightForge only |
| Router agent pattern | Mandatory entry point for agentic/multi-step tasks only | Single-step mechanical tasks bypass router |
| Router-escalation triggers | Veil topology changes, multi-node coordination, novel exploit chain development, architectural container boundary reasoning, tasks that cannot map to a single existing agent | Opus only on confirmed high-consequence scope -- never on ambiguity alone |
| Risk tier -- Low (Haiku) | Read-only, markdown/doc writes, template-based note generation, reversible tasks with no infra impact | Fast, token efficient, appropriate for mechanical tasks |
| Risk tier -- Medium (Sonnet) | Code/script writes, CRTA note enrichment, routine container boundary research, infra reads with reasoning, git ops beyond commit messages | Default execution tier |
| Risk tier -- High (Opus) | Veil topology changes, multi-node coordination, novel exploit chains, architectural research reasoning | Deliberate tool, not a default |
| Risk tier -- Manual approval | ~/.ssh, /etc/wireguard, WireGuard keys, vault credentials, destructive ops, unknown outbound calls | Hook enforcement, not router decision |
| Write tier by file type | Markdown/docs = Low (Haiku), code/scripts = Medium (Sonnet), infra config = escalation | Accurate risk mapping, token efficient |
| Container research escalation | Sonnet default + explicit escalation for architectural/novel findings | Token efficient; Opus reserved for genuinely novel reasoning |
| CRTA note split | Creation (Haiku) vs enrichment (Sonnet) | Creation is mechanical, enrichment requires reasoning |
| Obsidian CLI binary | obsidian at /usr/sbin/obsidian | Not obsidian-cli -- confirmed Session 27a |
| Obsidian CLI write mechanism | obsidian CLI for all six vault commands | Wikilink-aware, template support, richer than filesystem MCP for vault ops |
| Slash commands location | ~/.claude/commands/ (global) | Available across all repos; project commands override if name conflicts |
| Site launch strategy | Launch when first container boundary finding is published | Site announces Azrael Security with research signal attached. No soft launch with empty research page. CWL certification breadth is portfolio context, not a gate condition. |
| skill-06 version | v1.5 | Updated S32: skill-09 added as unconditional read alongside skill-06 |
| Tailscale | Removed from Cerberus S27b | Conflicts with local-first OPSEC philosophy. iptables-nft tables were polluting nftables ruleset. |
| nftables access model | Internal services WireGuard-only (10.0.0.0/24). SSH LAN-only (192.168.1.0/24). DNS/Caddy/Pi-hole LAN-only. WireGuard port 51820 any. Port 22 (Cowrie) any, rate-limited. | Updated S28. Cowrie added as external-facing with rate limit. |
| Cowrie access model | External exposure, rate-limited (10 new conn/min, burst 5) | Internal-only produced zero signal. Rate limit keeps log volume manageable and reduces abuse risk. nftables default-drop policy provides backstop. |
| Starship on Cerberus + Tairn | Dropped permanently | Headless nodes, never visually inhabited. Brand color polish has zero signal value on nodes accessed only via SSH. |
| NightForge dotfiles management | Stow migration deferred to Infrastructure Audit session | All dotfiles currently copies, not symlinks. Stow restructure requires full directory layout decision affecting all dotfiles simultaneously. |
| tmux | Confirmed multiplexer on NightForge | Not Zellij. ZELLIJ-WORKFLOWS.md in nightforge docs is stale -- correct during Infrastructure Audit session. |
| llmfit | Installed on NightForge, built from source | ~/Repos/3rd-party/llmfit, symlinked to /usr/local/bin/llmfit. GTX 1650 4GB VRAM detected. Relevant for local LLM model selection decisions. |
| Claude Code ecosystem audit | Completed S28 | 13 items reviewed. Decisions locked per item. See Near-Term for trigger conditions. |
| Piebald-AI/claude-code-system-prompts | Reference repo -- bookmark, no install | Updated within minutes of each Claude Code release. Read before S28 hooks session. |
| gstack | Extract selectively, do not install wholesale | Completed S29. /investigate and /review extracted. gstack cloned to ~/Repos/3rd-party/gstack/ read-only. |
| Claude Code branch enforcement | `review/*` prefix required for all Claude Code pushes | Operator reviews before merge to main; force push blocked on all branches including `review/*` |
| nightforge repo visibility | GitHub primary for portfolio signal | Config, hooks, and commands are deliberate operator choices worth showing; runtime data gitignored |
| gstack installation | Not installed wholesale — selective extraction only | Product dev workflow doesn't map to offensive security research context; `/investigate` and `/review` patterns extracted into Azrael-native commands |
| `.claude/commands/` location | Symlinked from `~/Github/nightforge/.claude/commands/` | Repo is source of truth; symlink wires live Claude Code to tracked commands without duplication |
| Infrastructure OS philosophy | Specialization over consolidation | Each node runs OS best suited to its role |
| Redirector architecture | Dedicated VM (Hermes), not Cerberus proxy | Correct engagement separation — redirector is disposable, C2 is not |
| Redirector OS | Alpine Linux | Minimal, disposable, fits redirector role |
| SSH key usage | homelab-id_ed25519 for all internal lab SSH; id_ed25519 for GitHub only; paradigm key deleted after migration session | Consistent, auditable key usage across lab |
| SSH algorithm | ed25519 for all lab keys | Curve25519, transparent parameters, no weak parameter choices |
| Redirector TLS model | TLS terminates at Hermes, plain HTTP over WireGuard to Tairn:80 | WireGuard encrypts internal leg; Hermes owns the TLS face; Tairn never touches internet |
| Redirector header masking | nginx-mod-http-headers-more, more_set_headers directive | proxy_hide_header + add_header insufficient — Nginx injects its own Server header after proxy response; headers-more module overwrites it |
| Skills panel upload format | YAML frontmatter (---) required | Bare key-value headers rejected by Claude Skills panel — confirmed S32 |
| Skills source of truth | ~/Github/veil/docs/skills/ | Repackage and re-upload to Skills panel when source files change |
| Claude project files | Handoff doc + troubleshooting.md only | Skills load from Customize panel; GitHub connectors handle repo content |
| GitHub connector behavior | Indexed snapshot with refresh lag | Same-session commits not reflected; handoff doc upload remains required |
| Claude Code hook scope | Pre-tool hooks only fire inside Claude Code sessions | Manual shell git commands bypass hooks by design — correct behavior |
| Security research MCP server | Side project candidate | CVE lookup + nuclei wrapper — gap in current MCP ecosystem, publishable |

### MCP Servers (NightForge, ~/.claude.json)
| Server | Package | Status | Scope |
|---|---|---|---|
| github | @modelcontextprotocol/server-github | Connected | All projects |
| fetch | uvx mcp-server-fetch | Connected | All projects |
| filesystem | @modelcontextprotocol/server-filesystem | Connected | ~/Github, ~/Documents/azrael-ops, ~/Documents/azrael-vault |
| sequential-thinking | @modelcontextprotocol/server-sequential-thinking | Connected | All projects |
| context7 | @upstash/context7-mcp | Connected | azraelsec.dev only |

### Claude Code Agents (NightForge, ~/.claude/agents/)
| Agent | Model | Purpose |
|---|---|---|
| router | sonnet | Entry point for all agentic/multi-step tasks. Parses intent, assesses risk tier, dispatches to specialist agents. |
| router-escalation | opus | High-consequence reasoning. Veil topology, multi-node coordination, novel exploit chains, architectural research. |
| site-builder | sonnet | Astro page builds, brand token enforcement |
| security-reviewer | sonnet | Read-only OPSEC and code review |
| research-writer | sonnet | Security research content, practitioner tone. WebFetch enabled -- intentional. |
| infra-auditor | sonnet | Read-only Veil config review |
| commit-scribe | haiku | Commit message generation |
| vuln-analyst | sonnet | Container boundary research analysis, read-only. WebFetch + WebSearch enabled -- intentional. |

### Claude Code Plugins (NightForge, ~/.claude/plugins/)
| Plugin | Source | Purpose |
|---|---|---|
| claude-hud | jarrodwatts/claude-hud | Real-time statusline: context, tools, agents, todos, usage. Config at ~/.claude/plugins/claude-hud/config.json |
| variant-analysis | trailofbits/skills | 5-step vuln class hunting with rg/Semgrep/CodeQL |
| static-analysis | trailofbits/skills | CodeQL + Semgrep + SARIF parsing toolkit |
| differential-review | trailofbits/skills | 7-phase security-focused diff/PR review |
| audit-context-building | trailofbits/skills | Deep architectural context mapping before audit |
| sharp-edges | trailofbits/skills | Error-prone API and dangerous config detection |
| semgrep-rule-creator | trailofbits/skills | Write and refine custom Semgrep rules |

### Claude Code Skills (NightForge, ~/.claude/skills/)
| Skill | Source | Purpose |
|---|---|---|
| strategic-compact | ECC (affaan-m) | Monitors context window, suggests compaction before limit |

### Cerberus Service Inventory (updated Session 27b)
**Podman rootless Quadlets (~/.config/containers/systemd/):**
- caddy.container -- TLS reverse proxy, local CA, .lan virtual hosts (glances.lan + noc.lan added S27b, netdata.lan removed)
- gitea.container -- private Git server (git.lan, port 3000)
- glances.container -- system monitor (glances.lan, port 61208) -- replaced netdata S27b
- homepage.container -- NOC dashboard (dash.lan, port 8282)
- nightforge-cowrie.container -- Cowrie SSH honeypot (port 22 maps here, real SSH on 2121)
- searxng.container -- private search engine (search.lan, port 8888)
- vaultwarden.container -- password manager (vault.lan, port 8081)

**System Quadlets (/etc/containers/systemd/):**
- pihole.container -- DNS sinkhole (pihole.lan, port 8083), network=host

**Systemd user services:**
- nightforge-shield.service -- nftables-based connection tracking and rate limiting

**NOC scripts (~/scripts/noc/):**
- wg-status.sh -- WireGuard peer health, outputs ~/noc-status/wg.json
- cowrie-status.sh -- Cowrie activity (24h), outputs ~/noc-status/cowrie.json
- suricata-status.sh -- Suricata high/critical alerts (24h), outputs ~/noc-status/suricata.json
- nftables-status.sh -- drop counter values, outputs ~/noc-status/nftables.json
- All run via cron every 5 minutes. Suricata uses /tmp/eve-recent.json (tail -c 52428800 workaround). Logrotate installed and verified S27b.

### Operator Rules

| Rule | Detail |
|---|---|
| Credentials | Never share tokens, keys, or passwords in chat |
| Tool selection | Use `rg` on NightForge, `grep` on Cerberus |
| Tairn changes | Always go in `configuration.nix`, then execute `nixos-rebuild switch` |
| Operational self-sufficiency | On any infrastructure issue, attempt full diagnosis independently before engaging Claude. Use skill-02 debug workflow as the framework. After resolution, identify which diagnostic steps required prompting — that delta is the current capability gap. Goal is to narrow it each session. |
| azrael-ops naming | Session number convention is canonical — azrael-handoff-q1-sNNN.md |
| Caddy reload | `systemctl --user restart caddy.service` (not HUP) |
| Pi-hole DNS reload | `sudo podman exec pihole pihole reloaddns` |
| Pi-hole DNS records | `/etc/pihole/hosts/local.lan` |
| nftables reload | `sudo nft -f /etc/nftables.conf` |
| Cerberus interface recovery | `sudo ip link set enp6s21f0u2c2 up` then restart nftables and WireGuard |
| Claude Code launch directory | `~/Github/` (not home directory) |
| fd not find on NightForge | find is aliased to fd |

### Practitioner Principles
- **Interrogate the why** -- push back on every recommendation before accepting it. The instinct to question assumptions is the same one that finds vulnerabilities in code and specs.
- **Systems thinking** -- every decision is an architecture decision. Understand how components interact before touching any of them. This is what separates depth from surface-level execution.

---

## SECTION 1 -- WHO DARRIUS IS

**Darrius Grate (ForeverLX)** -- offensive security engineer and vulnerability researcher. Building toward Red Team Infrastructure and Vulnerability Research roles. Target conference: HackSpaceCon (May 2026). Philosophy: quality over ease, local-first, depth over breadth, manual understanding before automation, OPSEC-conscious at all times.

**E-shaped specialization (target profile, not current state):**
- Depth area 1: Red team infrastructure and operations
- Depth area 2: Threat intelligence and adversary tradecraft
- Depth area 3: Vulnerability research
- Horizon tracks (long-term, not competing for current focus): quantum computing offensive security, bio-engineering offensive security

**Azrael Security** -- professional offensive security brand. All work -- infrastructure, research, tooling -- is portfolio signal. Every research effort should produce multiple outputs (GitHub, talk, YouTube, research brief). Tagline: "Below the abstraction."

**Noah Grayson** -- planned Azrael Security partner. Offensive security practitioner, 10+ pentest engagements over 6+ months. Pursuing a vision of modern offensive security that bridges CTEM, CPT/PTaaS, Human-Led Agent-Assisted Pentesting, CRQ, ASM, APM, Purple Teaming, and Security Posture Management. Aligned with Sprocket Security philosophy. Structure deferred until post-call vision alignment check. Three questions for the call: what is Noah's vision for Azrael Security, where does he see himself in the work, what is his timeline and risk tolerance.

### Active Capability Gaps

Named gaps requiring active attention -- not deferred to quarterly review.

| Gap | Description | Action |
|---|---|---|
| Veil operational fluency | Can build and configure Veil infrastructure with Claude's guidance but cannot diagnose and resolve service failures independently. Debug workflows, Quadlet lifecycle, nftables troubleshooting, and Pi-hole administration need to be internalized, not just referenced. | On next infra issue, attempt diagnosis independently before asking Claude. Track which steps required prompting -- that is the gap map. Skill-02 is the reference standard. |

---

## SECTION 2 -- INFRASTRUCTURE STATE

**Veil mesh:** Operational. Three nodes connected via WireGuard hub-and-spoke. Hermes redirector VM added to mesh S30.

**Cerberus:** All Podman Quadlets running. Pi-hole, SearXNG, Vaultwarden, Gitea, Homepage, Caddy, Glances, Cowrie all operational. nightforge-shield.service active. Tailscale removed S27b. nftables hardened S27b, updated S28 -- port 22 now open externally with rate limiting (10 new conn/min, burst 5). Cowrie reachable from internet as of S28. noc.lan fully operational -- all four JSON endpoints verified (wg, cowrie, suricata, nftables). Caddyfile now tracked in veil repo under edge-node/configs/caddy/Caddyfile. Ethernet interface (enp6s21f0u2c2) went DOWN during S27b -- recovered manually, documented in troubleshooting.md.

**NightForge:** Operational. Arch Linux, Niri WM, Zsh + Starship, Neovim, tmux. Hostname: NightForge. AppArmor removed — 0 audit errors confirmed post-reboot. Claude Code installed. claude-hud installed and configured. ccsm installed. llmfit installed — built from source at ~/Repos/3rd-party/llmfit, symlinked to /usr/local/bin/llmfit. GTX 1650 4GB VRAM detected. Starship config updated with Azrael brand colors. All MCP servers connected. 8 agents configured. azrael-vault at ~/Documents/azrael-vault/. Dotfiles: all files are copies, not symlinks — stow migration deferred to Infrastructure Audit session. obsidian-status.sh rescued and committed to nightforge repo S28. Claude Code hooks modularized into ~/.claude/hooks/pre-tool/ (5 scripts) S29. ~/.claude/commands/ symlinked to ~/Github/nightforge/.claude/commands/ S29. gstack cloned to ~/Repos/3rd-party/gstack/ S29. wireshark-cli installed S34. ForeverLX added to wireshark group (requires logout/login to activate). Paradigm user deleted S34 — account and home directory fully removed. caido added to ~/Repos/3rd-party/. dradis-ce removed from ~/Repos/3rd-party/.

**Tairn:** Mythic C2 operational. Poseidon agent installed. Apollo agent installed. WireGuard-only access confirmed. AllowedIPs updated to 10.0.0.0/24 on Cerberus peer (S30). HTTP C2 profile running on *:80. Poseidon payload compiled S31 with callback_host https://10.0.0.5, callback_port 443, post_uri /data. End-to-end callback chain confirmed S31: NightForge → Hermes:443 → Tairn:80 → Mythic.

**Hermes:** Alpine Linux 3.23.3 VM provisioned on NightForge (libvirt, default network). Static IP 192.168.122.200. WireGuard peer 10.0.0.5, full mesh connectivity confirmed S30. Nginx installed S31. Self-signed cert at /etc/nginx/ssl/redirector.crt + redirector.key (CN: cdn.cloudflare.com). Redirector config at /etc/nginx/http.d/redirector.conf — TLS terminates at Hermes, /data proxied to Tairn:80, all other URIs 301 to cloudflare.com. Server header masking via nginx-mod-http-headers-more (NetDNA-cache/2.2). DNS set to 1.1.1.1 (public) — Pi-hole on Cerberus not resolving external names for Hermes, needs investigation. Poseidon callback through redirector confirmed S31. SSH key: homelab-id_ed25519. Note: Nginx not yet configured to start on boot.

---

## SECTION 3 -- ACTIVE OPERATIONS

**Mythic C2 (Tairn):**
- Status: Operational. HTTP listener active on *:80.
- Poseidon agent: installed. Payload compiled S31, callback chain confirmed through Hermes redirector.
- Apollo agent: installed.
- Redirector: Hermes (10.0.0.5) Nginx running. TLS terminates at Hermes, proxies /data to Tairn:80. Header masking active.
- Open items: Nginx not set to start on boot on Hermes. Pi-hole not resolving external names for Hermes (DNS gap). poseidon.bin stored at ~/lab/poseidon.bin on NightForge.

**Container Boundary Research:**
- Status: Pre-research. Research question locked S27c.
- **Falsifiable research question (locked):** In rootless Podman deployments on RHEL-family Linux, does nvidia-container-runtime execute OCI hooks with privileges exceeding the container's user namespace mapping, and can a compromised agentic workload supply attacker-controlled input to that hook execution to achieve code execution in the host's namespace?
- Threat model: compromised agentic workload in GPU-enabled multi-tenant environment (OpenClaw/NemoClaw deployments)
- Prior work identified: CVE-2025-23266 (NVIDIAScape) -- direct hit. Research is additive: rootless-specific behavior + full input surface mapping beyond env vars
- Prior work paragraph: not yet written -- after reading schedule completes
- Reading schedule: Wind-down 10-10:45pm Mon-Fri + Saturday deep research 8-9:30am, both recurring on Azrael Security calendar
- CLAUDE.md added to security-research repo Session 27a

---

## SECTION 4 -- REPOSITORY STATE

| Repo | Remote | Status | Notes |
|---|---|---|---|
| veil | GitHub + Gitea | Active | Caddyfile added to edge-node/configs/caddy/ S28. nftables.conf updated S28 -- Cowrie port 22 rate-limited rule added. skill-06 v1.3 committed S27b. troubleshooting.md updated S27b. |
| nightforge | GitHub + Gitea | Active | S29: hooks modularized into .claude/hooks/pre-tool/ (5 scripts), .gitignore restructured to track .claude/ selectively, review/ branch enforcement locked, /investigate and /review slash commands added, existing review.md renamed to write-first-review.md, ~/.claude/commands/ symlinked to repo. S34: stale scripts removed (recon-pipeline.sh, security-hardening.sh, host-maintenance.sh), tools-inventory.md overhauled. Commits e00210e + b08d94d. |
| security-research | GitHub | Active | CLAUDE.md added S27a |
| azraelsec.dev | GitHub + Gitea | Active | / /infrastructure /projects complete. /research /about copy locked, build gated on first published research finding (Near-Term, not dedicated session slot) |
| azrael-ops | Gitea (private) | Active | claude-hud config backed up S27b |

---

## SECTION 5 -- BACKLOG

### Next Session
- **Stow migration** — full dotfiles layout decision, restructure, symlink all dotfiles. operator-terminal symlinks absorbed into this pass. Do not pre-script — audit, decide layout, execute in session.

### NightForge Infrastructure Audit + Improvement Session (Near-Term, no date)
Open-ended scope — do not pre-script. Audit, refine, restructure, improve in the session:
- NightForge nftables hardening — drop logging, SSH rate limiting, WireGuard port logging. Separate infra session after stow.
- README update — dotfiles section stale, changelog stale. Do after stow migration settles.
- Dotfiles reference repos deep-dive — breadonpenguins-dots, caelestia-dots-*, maxhu08-dotfiles, maxhu08-wallpapers — extract anything useful for NightForge, then decide keep or remove.
- ~/Repos/3rd-party/ audit — 16+ repos present, some undocumented (Rubeus, RustHound-CE, recon-ng, gef, pwndbg confirmed present S33)
- Any other gaps, areas for improvement, and clarifying questions surfaced in session

### Session S29 — Claude Code Site Build (Near-Term, gated)
- **Gate condition:** first container boundary finding published
- Subheader sizing fix (text-sm to text-base on descriptions, text-xs to text-sm on links, text-lg to text-xl on names — both infrastructure.astro and projects.astro, do not touch status badges)
- Install ui-ux-pro-max-skill at session start — review folder structure issue before installing
- Install microsoft/playwright-cli skill at session start
- Build /research page
- Build /about page (copy cert images from security-research/assets/certificates/ first)

### Obsidian CLI Integration Session (Near-Term, schedule after NightForge Audit)
- Vault: ~/Documents/azrael-vault/, binary: obsidian at /usr/sbin/obsidian
- Install kepano/obsidian-skills at session start — clone into azrael-vault/.claude/skills/
- Templater plugin confirmed enabled
- ATTACKmd confirmed at 30-Resources/ATTACK/
- Commands to build: /attack, /linux, /ad, /cloud, /finding, /intel
- All six use obsidian CLI exclusively (wikilink-aware, template support)
- Slash commands location: ~/.claude/commands/ (global)
- Session steps: templates in Templater-Obsidian/ → bash helpers → slash command definitions → test each → backup to azrael-ops

### azraelsec.dev Claude Code Workflow Writeup (Near-Term)
- Gate is open — hooks session completed S29
- Do not publish until reviewed against brand-communication-standard (skill-05)
- Scope: document the full Claude Code hooks + commands workflow as a public-facing portfolio piece

### Near-Term (Next Few Sessions)
- **Cowrie log review — NOC script** — write a lightweight Cowrie log parser as a programming exercise. C or Python. Learning vehicle for file I/O and log parsing. Gate: after Python offensive scripting track is underway.
- **NOC dashboard redesign** — current NOC dashboard is not fully operational. Scope a redesign session separately from the log script work.
- **Regenerate all passwords** — 21+ random characters, store in local Vaultwarden vault.
- **Review 40+ saved LinkedIn jobs** — apply Phase 3 filter from career plan. Floating near-term, no hard deadline.
- **Alignerr application** — blocked on phone verification (carrier bill unpaid). Keep until resolved.
- **Read gadievron/raptor** — read CLAUDE.md and .claude/ structure before next container boundary research session. Clone first: cd ~/Repos/3rd-party && git clone https://github.com/gadievron/raptor
- **Read transilienceai/communitytools** — skill-writing methodology before CRTA lab work. Clone first.
- **Read spaceraccoon/from-day-zero-to-zero-day** — before next container boundary research session. Clone first.
- **suricata-tail** — C implementation of EVE JSON tail parser. Learning vehicle for file I/O, lseek. Gate: after Python offensive scripting track completes or runs dry.
- **security-research CLAUDE.md** — write during first container boundary research session, not before.
- **Orange-Cyberdefense/GOAD** — needed when AD lab work starts (CRTA).

### Someday / Deferred
- **RAG for azrael-vault** — trigger condition: vault research content exceeds 100 interconnected notes. Reevaluate Q3/Q4.
- **Custom research-log MCP server** — reevaluate Q3/Q4 alongside RAG trigger condition.
- **Security research MCP server** — CVE lookup + nuclei + exploit-db wrapper for Claude Code. Gap confirmed S32. Gate: after first container boundary finding published.
- **Obsidian Sync** — deferred until quality laptop acquired.
- **Second Chromebook integration** — iPXE network boot from Cerberus. Name TBD.
- **Feyra** — blocked, requires hardware upgrade or LLM fine-tuning study.
- **PortSwigger intermediate** — deferred to Q2 per career plan.
- **TP-Link router / network segmentation** — deferred, revisit when Chromebooks ready.
- **Four additional Chromebooks** — operator nodes and/or agentic workflow nodes.
- **Sashiko** — file for Phase 5 kernel research track.
- **Chopper + Rippy** — file for Phase 2-3 RE track.
- **Claude Dispatch** — revisit when stable official Linux support ships.
- **Clipboard manager** — deferred until written personally in Python/C/Go.
- **Cowork (official)** — revisit when stable official Linux support ships.
- **vulncheck-oss/0day.today.archive** — Q2 item, pair with explorar.dev CVE root cause track.
- **ECC testing-handbook-skills** — deferred to Q2+ when fuzzing becomes part of research.
- **ToB plugins** — fix-review (gate: first finding published), yara-authoring (threat hunting track), dwarf-expert (RE track Q2+), constant-time (crypto primitives track).
- **Astro-style TUI install script** — NightForge dotfiles automation. Not Veil deployment.

### Personal Site Build Status
| Page | Status | Notes |
|---|---|---|
| `/` | Complete | Home page committed, em-dashes fixed |
| `/infrastructure` | Complete | Committed Session 23 |
| `/projects` | Complete | Committed Session 23 |
| `/research` | Copy locked | Build S29 (gated on first published finding) |
| `/about` | Copy locked | Build S29 (gated on first published finding) |

### Side Projects and Ideas Backlog
| Project | Priority | Notes |
|---|---|---|
| Adversary Emulation Platform ("Phantom" or "Wraith") | High | Do not scope until container research published |
| Yonyx-inspired offensive security decision tree tool | High | Do not scope until container research published |
| Network reconciliation tool (Go/Rust) | Medium | Learning vehicle, stays in backlog |
| Security research agent framework | Medium | |
| Prompt Improver | Medium | Claude Code agent/skill — scoped to Claude AI/Code workflow |
| LLM queue sidebar | Medium | Claude Code agent/skill or standalone — scoped to Claude AI/Code workflow |
| Security research MCP server | High | CVE lookup + nuclei + exploit-db wrapper. Gap confirmed S32. Gate: after first container boundary finding published. |

---

## SECTION 6 -- COURSES AND CERTIFICATIONS

### In Progress
| Course | Status | Notes |
|---|---|---|
| CRTA (Certified Red Team Analyst) | Active -- Q1 Friday Block 1 | Lab work on Tairn, every technique reproduced end-to-end before moving on. Operational mindset: treat each technique as a simulated operation. |
| CRT-ID (Certified Red Team Infrastructure Developer) | Started | Lab work on Tairn |

### Not Yet Started
MCRTA, BTF, PTF, CPIA, CRT-COI

### Completed Certifications
AD-RTS, CAPT, COSJ -- images in `security-research/assets/certificates/`

### Q1 Learning Track (March 20 -- May 31)
**Foreground (simultaneous):**
- Python offensive scripting -- Tue/Thu Block 1. Write from scratch, no Claude generation.
- Networking hands-on (Wireshark/tcpdump on Veil) -- Mon/Wed Block 1. Foreground, load-bearing.
- CWL/CRTA lab reproduction + writeup -- Fri Block 1. Operational mindset: sniper discipline, not race to the flag.

**Background (sequential):**
- March 20 -- April 19: OST2 Arch1001 -- x86 Assembly Foundations
- April 20 -- May 9: OST2 Dbg1012 -- Debugger Internals
- May 10 -- May 31: Nightmare + pwn.college -- Modern Exploit Dev

**Curated platform sequence (Q2+):**
OST2 exploitation learning path, pwn.college, ret2.systems or challenges.re, explorar.dev (CVE root cause), Protohackers (Go, 1 challenge/week background throughout)

---

## SECTION 7 -- SESSION LOG

### Session 34 — 2026-03-28 — NightForge Audit: Paradigm Migration + Tooling Cleanup

Paradigm user fully audited across all major directories (Labs, engage, Repos, Tools, vaults) and deleted — zero content loss confirmed via Claude Code parallel inventory passes and vault diffs. Sgaeyl (142M, 2758 files) and CRTO-Journey-Vault (76M extracted) both audited: one Templater template migrated, everything else confirmed redundant or empty. Caido git repo migrated to ~/Repos/3rd-party/caido/; dradis-ce removed. Three stale offsec-workstation scripts removed from nightforge and committed. tools-inventory.md fully overhauled with 12 categories and current toolset. wireshark-cli installed, ForeverLX added to wireshark group. Remaining audit items deferred: stow migration (next session), nftables hardening, README update, dotfiles reference repos deep-dive.

### Session 33 — 2026-03-28 — Hermes Hardening + Backlog Refinement

Hermes redirector fully persistent: Nginx and WireGuard (wg-quick.wg0 symlink pattern) both start on boot, confirmed surviving reboot. Pi-hole DNS fixed for all WireGuard peers — Cerberus nftables was missing UDP+TCP/53 rules for 10.0.0.0/24, added and committed to veil repo. nftables.conf and bookmarks.yaml both added to veil repo as newly tracked configs. Homepage updated with Alpine Linux and NixOS Wiki bookmarks using CDN icon names. troubleshooting.md updated with three entries: Alpine wg-quick boot persistence, WireGuard AllowedIPs hub-and-spoke scope, Nginx Server header masking. Full Section 5 backlog audit completed: 17+ items closed, dropped, or absorbed — duplicates removed, gate conditions tightened, Cowrie log review scoped as programming exercise, NOC dashboard redesign added as separate item, gsd-2 evaluated and dropped, public announcement dropped, azrael-ops naming convention locked to session number, operational self-sufficiency rule added to Section 0. Programming language sequence decided: C → Go → Rust (Phase 4-5 only). Drop Session 29 entry to maintain three-entry limit.

### Session 32 — 2026-03-27 — Skills Migration + Workflow Infrastructure

All nine skill files packaged with YAML frontmatter and uploaded to Claude Customize > Skills panel. skill-09 (verification-and-accuracy-standard) written, committed, and pushed to veil. skill-06 patched to v1.5 with skill-09 unconditional read added. Source files updated with YAML frontmatter, committed to veil. Claude Code push-to-main hook patched to block bare git push, committed to nightforge. GitHub connector confirmed as indexed snapshot — same-session commits not reflected, handoff upload remains required. Project audit completed: ForeverLX/ForeverLX connector removed, backup resume kept. MCP ecosystem audited — no immediate security-specific wins, security research MCP server added to ideas backlog. Stale docs/99-workstation-audit.md flagged for cleanup.

### Session 31 — 2026-03-27 — Mythic Redirector Chain

Mythic HTTPS listener + Nginx redirector on Hermes fully operational. Self-signed cert generated on Hermes (CN: cdn.cloudflare.com), Nginx configured with URI allowlist (/data proxied to Tairn:80, all other URIs 301 to cloudflare.com), Server header masking via nginx-mod-http-headers-more (NetDNA-cache/2.2). Poseidon payload compiled with callback_host https://10.0.0.5, port 443, post_uri /data. End-to-end callback chain confirmed: NightForge → Hermes:443 → Tairn:80 → Mythic active callback. Open items: Nginx not set to start on boot on Hermes; Pi-hole not resolving external names for Hermes (DNS workaround: 1.1.1.1). Session close procedure patched — SESSION PROCEDURE block added to Section 0, skill-06 Step 0 re-read enforced.

---

*Schema version 1.0. To update: tell Claude "session close" or "wrap up." Claude asks five questions, outputs changed sections only. Claude Code applies edits, saves as azrael-handoff-q1-sNNN.md, commits, and pushes. Commit: `git add azrael-handoff-q1-s030.md && git commit -m "docs: session handoff YYYY-MM-DD SNNN"`*
