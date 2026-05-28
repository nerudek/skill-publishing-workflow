# Skill Publishing Workflow

> Multi-agent publishing protocol — anti-duplicate checks, pre-flight checklist, Arena SEO FAQ generation, and cross-machine sync for Vox + Nexus.

**Version:** 1.0.0 &middot; **Author:** nerudek &middot; **Compatible with:** hermes-agent

[![GitHub](https://img.shields.io/badge/GitHub-nerudek-181717?logo=github)](https://github.com/nerudek)
[![PayPal](https://img.shields.io/badge/PayPal-Donate-00457C?logo=paypal)](https://www.paypal.me/nerudek)

---

## Problem

Publishing AI skills and articles across GitHub, Dev.to, and ClawHub requires coordination between multiple agents running on different machines (Mac Mini "Vox" and MacBook "Nexus"). Without a shared protocol, agents publish duplicate skills, skip quality checks, forget cover images, hardcode local paths, and accidentally push Polish text into public repos. There is no single source of truth for who creates, who vets, and who publishes — leading to inconsistent skill repos, broken CI pipelines, and wasted publishing slots.

This workflow defines the exact process: who creates, who vets, who publishes, and how skills sync between agents via Tailscale rsync.

## Solution

A unified **skill-publishing workflow** that enforces a strict division of labor:

| Role | Agent | Actions |
|------|-------|---------|
| Creates / edits skills | Nexus, Vox | Write SKILL.md, test, validate |
| Pushes to Vox | Nexus | Sync via `skills-sync.sh` |
| Vets skills | Vox | Run pre-publish checklist |
| Publishes to GitHub | Vox | `gh repo create`, push |
| Publishes to Dev.to | Vox | API post with cover image |

**Golden rule:** Only Vox publishes. Nexus creates and pushes to Vox for publishing. Vox never publishes un-vetted skills.

## Installation

```bash
# Clone this repository on both Vox and Nexus
git clone https://github.com/nerudek/skill-publishing-workflow.git ~/.hermes/skills/skill-publishing-workflow
```

Ensure the `skills-sync.sh` bridge script is installed and the Tailscale network is active between both machines.

## Quick Start

```bash
# Check which skills are out of sync
~/.hermes/scripts/bridge/skills-sync.sh check

# Push local skills to the other agent
~/.hermes/scripts/bridge/skills-sync.sh push

# Pull remote skills from the other agent
~/.hermes/scripts/bridge/skills-sync.sh pull
```

## Publishing a New Skill (Step by Step)

### 1. Create SKILL.md (any agent)

```markdown
---
name: skill-name
description: One-line benefit-first English description
version: 1.0.0
author: nerudek
compatible-with: hermes-agent
---

# Skill Title

## Problem
What problem does this solve? (2-3 sentences)

## Solution
How does this skill solve it?

## Usage / Quick Start
Minimal working example.

## FAQ
[15 questions from Arena AI protocol]

---

If this saved you time: [PayPal.me/nerudek](https://www.paypal.me/nerudek)
GitHub: [github.com/nerudek](https://github.com/nerudek)
```

### 2. Validate (creating agent)

- [ ] YAML frontmatter complete (name, description, version, author)
- [ ] English only (no Polish in public content)
- [ ] No hardcoded local paths (`~/.hermes` is OK, `~` is NOT)
- [ ] FAQ has 10-15 Q&A pairs
- [ ] PayPal + GitHub footer present
- [ ] Problem section states the problem clearly

### 3. Sync to Vox (if created by Nexus)

```bash
./skills-sync.sh push
```

### 4. Vox Pre-Publish Checklist

```bash
# Anti-duplicate check
ls ~/.hermes/skills/ | grep -i "<skill-name>"
gh repo list nerudek --limit 50 | grep -i "<skill-name>"

# Security scan
grep -r "api_key\|token\|password\|secret" ~/.hermes/skills/<name>/SKILL.md
grep -r "/Users/" ~/.hermes/skills/<name>/SKILL.md
```

### 5. Publish (Vox only)

```bash
gh repo create nerudek/skill-<name> --public --description "EN benefit-first description"
cp ~/.hermes/skills/<name>/SKILL.md /tmp/publish/SKILL.md
cd /tmp/publish && git init && git add . && git commit -m "skill-<name> v1.0.0"
git remote add origin https://github.com/nerudek/skill-<name>.git
git push -u origin main
```

## How It Works

### Skill Directory Structure

```
~/.hermes/skills/<skill-name>/
  SKILL.md          # Main skill file (REQUIRED)
  README.md         # GitHub README (optional, auto-generated from SKILL.md)
  references/       # Supporting docs
  scripts/          # Executable scripts
  templates/        # File templates
```

### Sync Architecture

Skills live in `~/.hermes/skills/` on both machines. The `skills-sync.sh` bridge script uses Tailscale rsync to keep them in sync:

- **Push direction (Nexus -> Vox):** Nexus sends new/updated skills to Vox for publishing
- **Push direction (Vox -> Nexus):** Vox sends published skill repos back to Nexus
- **Conflict resolution:** Vox's version wins (Vox is publisher). Nexus should pull before editing.

### Arena SEO FAQ Protocol

Before publishing any skill or article, send the content to 2-3 different AI models. Ask each: *"What questions would users searching for this have?"* Compile the best 10-15 into the FAQ section. This dramatically boosts search ranking.

## Quality Gate (Mandatory Before Every Publish)

1. Code/commands tested on sample data
2. No credentials exposed
3. English language verified
4. YAML frontmatter valid
5. FAQ questions from 2+ AI models (Arena protocol)
6. Cover image generated (1280x640 PNG)
7. Anti-duplicate check passed
8. Vox review complete

## Common Pitfalls

1. **Duplicate skills** — always search before creating
2. **Polish text in public repos** — all published material must be English
3. **Hardcoded `~` paths** — use `~/.hermes` or generic placeholders
4. **Missing FAQ** — 15 questions minimum for SEO
5. **No cover image** — Dev.to requires one before publication
6. **Vox publishes without Nexus review** — sync first, publish after

## FAQ

**Q: Who decides if a skill is ready to publish?**
Vox does the final check. Nexus creates, Vox publishes.

**Q: What if Nexus creates a skill that already exists?**
Vox rejects with "DUPLICATE: similar skill exists at <path>". Nexus merges changes into the existing skill instead of creating a new one.

**Q: How are skills versioned?**
Start at 1.0.0. Increment MAJOR for breaking changes, MINOR for new features, PATCH for fixes. Follow semver conventions.

**Q: Can skills be in Polish?**
No. All published content must be English. Internal skills (like system-bridge) can have Polish docs for operational notes, but the public SKILL.md must be English-only.

**Q: Where are skills stored?**
`~/.hermes/skills/` on both machines. Synced via Tailscale rsync.

**Q: What is the Arena SEO protocol?**
Before publishing any article or skill, send the content to 2-3 AI models. Ask each: "What questions would users searching for this have?" Compile the best 10-15 into the FAQ section. This boosts SEO dramatically.

**Q: How often should skills be synced?**
After any skill is created or modified. Minimum once per session.

**Q: What happens if sync conflicts?**
Vox's version wins (Vox is publisher). Nexus should pull before making edits.

**Q: Can Vox create skills too?**
Yes. Vox creates, vets, and publishes directly without needing Nexus.

**Q: How to handle skill deprecation?**
Add `status: deprecated` to the YAML frontmatter. Keep the repo for historical reference.

**Q: What about openclaw/hermes compatible skills?**
Mark `compatible-with: [hermes-agent, openclaw]` in frontmatter if the skill works with both platforms.

**Q: How to test a skill before publishing?**
Vox loads the skill with `skill_view()` and runs its commands. Nexus tests locally before pushing.

**Q: Can we publish to ClawHub?**
`npx clawhub publish` — but GitHub is the primary target. ClawHub is secondary.

**Q: What about Dev.to articles for skills?**
Skills with broad appeal get a Dev.to article. Skills with niche use cases stay GitHub-only.

**Q: How to handle merge conflicts in skill files?**
Always accept the most recent version. Use `skills-sync.sh check` to identify differences before syncing.

---

## Configuration

No environment variables or API tokens are required for this workflow itself. Each agent must have:

- **Vox:** GitHub CLI (`gh`) authenticated, Dev.to API key (optional)
- **Nexus:** Tailscale connectivity to Vox, `skills-sync.sh` installed
- **Both:** `~/.hermes/skills/` directory structure in place

## Contributing

This skill is part of the hermes-agent ecosystem. Contributions and forks welcome.

---

If this saved you time: [PayPal.me/nerudek](https://www.paypal.me/nerudek)
GitHub: [github.com/nerudek](https://github.com/nerudek)
