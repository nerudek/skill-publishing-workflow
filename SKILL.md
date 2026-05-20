---
name: publishing-protocol
description: Shared publishing methodology for Vox and Nexus — how to add, check, and publish skills. Both agents use the same process. Only Vox publishes; Nexus pushes to Vox.
version: 1.0.0
author: nerudek
compatible-with: hermes-agent
---

# Publishing Protocol — Vox + Nexus Shared Methodology

## Problem

Publishing AI skills and articles across GitHub, Dev.to, and ClawHub requires coordination between multiple agents on different machines. Without a shared protocol, agents publish duplicates, skip quality checks, forget cover images, and push content with hardcoded paths or Polish text. This skill defines the exact process: who creates, who vets, who publishes, and how skills sync between agents via Tailscale rsync.

## Who Does What

| Role | Agent | Actions |
|------|-------|---------|
| Creates/edits skills | Nexus, Vox | Write SKILL.md, test, validate |
| Pushes to Vox | Nexus | Sync via `skills-sync.sh` |
| Vets skills | Vox | Run pre-publish checklist |
| Publishes to GitHub | Vox | `gh repo create`, push |
| Publishes to Dev.to | Vox | API post |

**Rule:** Only Vox publishes. Nexus creates and pushes to Vox for publishing.

## Adding a New Skill

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
What problem does this solve? (2-3 sentences, like a research paper abstract)

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
# From Nexus (MacBook):
./skills-sync.sh push
```

### 4. Vox Pre-Publish Checklist

```bash
# 1. Anti-duplicate check
ls ~/.hermes/skills/ | grep -i "<skill-name>"
gh repo list nerudek --limit 50 | grep -i "<skill-name>"

# 2. Skill vetter quick scan
grep -r "api_key\|token\|password\|secret" ~/.hermes/skills/<name>/SKILL.md
grep -r "/Users/" ~/.hermes/skills/<name>/SKILL.md  # Only ~/.hermes allowed

# 3. Arena SEO: send to 2 AIs for FAQ questions
# 4. Cover image: 1280x640 PNG
# 5. Quality gate: test the skill commands
```

### 5. Publish (Vox only)

```bash
gh repo create nerudek/skill-<name> --public --description "EN benefit-first description"
cp ~/.hermes/skills/<name>/SKILL.md /tmp/publish/SKILL.md
cd /tmp/publish && git init && git add . && git commit -m "skill-<name> v1.0.0"
git remote add origin https://github.com/nerudek/skill-<name>.git
git push -u origin main
```

## Syncing Skills Between Agents

```bash
# From Mac Mini (Vox):
~/.hermes/scripts/bridge/skills-sync.sh push   # Send to Nexus
~/.hermes/scripts/bridge/skills-sync.sh pull   # Get from Nexus
~/.hermes/scripts/bridge/skills-sync.sh check  # Compare

# From Nexus (MacBook):
./skills-sync.sh push   # Send to Vox for publishing
./skills-sync.sh pull   # Get latest from Vox
./skills-sync.sh check  # Check for differences
```

## Skill Directory Structure

```
~/.hermes/skills/<skill-name>/
  SKILL.md          # Main skill file (REQUIRED)
  README.md         # GitHub README (optional, auto-generated from SKILL.md)
  references/       # Supporting docs
  scripts/          # Executable scripts
  templates/        # File templates
```

## Common Pitfalls

1. Creating a skill that already exists — always search first
2. Polish text in public content — all published material must be English
3. Hardcoded `~` paths — use `~/.hermes` or generic placeholders
4. Missing FAQ — 15 questions minimum for SEO
5. No cover image before Dev.to publication
6. Vox publishes without Nexus review — sync first, publish after

## Quality Gate (mandatory before every publish)

1. Code/commands tested on sample data
2. No credentials exposed
3. English language verified
4. YAML frontmatter valid
5. FAQ questions from 2+ AI models (Arena protocol)
6. Cover image generated (1280x640)
7. Anti-duplicate check passed

## FAQ

**Q: Who decides if a skill is ready to publish?**
Vox does the final check. Nexus creates, Vox publishes.

**Q: What if Nexus creates a skill that already exists?**
Vox rejects with "DUPLICATE: similar skill exists at <path>". Nexus merges changes into existing skill instead.

**Q: How are skills versioned?**
Start at 1.0.0. Increment MAJOR for breaking changes, MINOR for new features, PATCH for fixes.

**Q: Can skills be in Polish?**
No. All published content must be English. Internal skills (like system-bridge) can have Polish docs.

**Q: Where are skills stored?**
`~/.hermes/skills/` on both machines. Synced via Tailscale rsync.

**Q: What is the Arena SEO protocol?**
Before publishing any article/skill, send content to 2-3 AI models. Ask each: "What questions would users searching for this have?" Compile best 10-15 into FAQ section. Boosts SEO dramatically.

**Q: How often should skills be synced?**
After any skill is created or modified. Minimum once per session.

**Q: What happens if sync conflicts?**
Vox's version wins (Vox is publisher). Nexus should pull before editing.

**Q: Can Vox create skills too?**
Yes. Vox creates, vets, and publishes directly.

**Q: How to handle skill deprecation?**
Add `status: deprecated` to frontmatter. Keep in repo for historical reference.

**Q: What about openclaw/hermes compatible skills?**
Mark `compatible-with: [hermes-agent, openclaw]` if skill works with both.

**Q: How to test a skill before publishing?**
Vox loads the skill with `skill_view()` and runs its commands. Nexus tests locally before pushing.

**Q: Can we publish to ClawHub?**
`npx clawhub publish` — but GitHub is the primary target. ClawHub is secondary.

**Q: What about Dev.to articles for skills?**
Skills with broad appeal get a Dev.to article. Skills with niche use stay GitHub-only.

**Q: How to handle merge conflicts in skill files?**
Always accept the most recent version. Use `skills-sync.sh check` to identify differences before sync.

---

If this saved you time: [PayPal.me/nerudek](https://www.paypal.me/nerudek)
GitHub: [github.com/nerudek](https://github.com/nerudek)
