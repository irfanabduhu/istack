# Rune

A collection of skill plugins for AI coding agents.

## Plugins

| Plugin | Description |
|--------|-------------|
| [fablesmith](fablesmith/) | Original fiction in any author's authentic voice |
| [readweave](readweave/) | Long-form content distilled into presentations and reading guides |
| [smolsmith](smolsmith/) | The markdown-only edition of fablesmith |
| [smolweave](smolweave/) | The markdown-only edition of readweave |

## Installation

### Claude Code

```bash
# Add marketplace (one-time)
claude plugin marketplace add irfanabduhu/rune

# Install individual plugins
claude plugin install fablesmith@irfanabduhu
claude plugin install readweave@irfanabduhu
claude plugin install smolsmith@irfanabduhu
claude plugin install smolweave@irfanabduhu
```

### OpenCode / Pi / Codex CLI

These agents share the `.agents/skills/` convention. Clone the repo and symlink the plugins you want:

```bash
git clone https://github.com/irfanabduhu/rune.git ~/.rune

# Symlink into your project
mkdir -p .agents/skills
ln -s ~/.rune/fablesmith .agents/skills/fablesmith
ln -s ~/.rune/readweave .agents/skills/readweave
ln -s ~/.rune/smolsmith .agents/skills/smolsmith
ln -s ~/.rune/smolweave .agents/skills/smolweave

# Or install globally
mkdir -p ~/.agents/skills
ln -s ~/.rune/fablesmith ~/.agents/skills/fablesmith
ln -s ~/.rune/readweave ~/.agents/skills/readweave
ln -s ~/.rune/smolsmith ~/.agents/skills/smolsmith
ln -s ~/.rune/smolweave ~/.agents/skills/smolweave
```

Each plugin contains a `SKILL.md` with YAML frontmatter — the standard format these agents expect.

### Cursor

Copy the `SKILL.md` from each plugin into `.cursor/rules/`:

```bash
cp fablesmith/SKILL.md .cursor/rules/fablesmith.md
cp readweave/SKILL.md .cursor/rules/readweave.md
cp smolsmith/SKILL.md .cursor/rules/smolsmith.md
cp smolweave/SKILL.md .cursor/rules/smolweave.md
```

### Windsurf

Copy into `.windsurf/rules/`:

```bash
cp fablesmith/SKILL.md .windsurf/rules/fablesmith.md
cp readweave/SKILL.md .windsurf/rules/readweave.md
cp smolsmith/SKILL.md .windsurf/rules/smolsmith.md
cp smolweave/SKILL.md .windsurf/rules/smolweave.md
```

### Cline

Copy into `.clinerules/`:

```bash
cp fablesmith/SKILL.md .clinerules/fablesmith.md
cp readweave/SKILL.md .clinerules/readweave.md
cp smolsmith/SKILL.md .clinerules/smolsmith.md
cp smolweave/SKILL.md .clinerules/smolweave.md
```

### GitHub Copilot

Copy into `.github/instructions/`:

```bash
mkdir -p .github/instructions
cp fablesmith/SKILL.md .github/instructions/fablesmith.instructions.md
cp readweave/SKILL.md .github/instructions/readweave.instructions.md
cp smolsmith/SKILL.md .github/instructions/smolsmith.instructions.md
cp smolweave/SKILL.md .github/instructions/smolweave.instructions.md
```

### Amazon Q Developer

Copy into `.amazonq/rules/`:

```bash
mkdir -p .amazonq/rules
cp fablesmith/SKILL.md .amazonq/rules/fablesmith.md
cp readweave/SKILL.md .amazonq/rules/readweave.md
cp smolsmith/SKILL.md .amazonq/rules/smolsmith.md
cp smolweave/SKILL.md .amazonq/rules/smolweave.md
```

### Aider

Load as read-only context:

```bash
aider --read fablesmith/SKILL.md --read readweave/SKILL.md --read smolsmith/SKILL.md --read smolweave/SKILL.md
```

Or add to `.aider.conf.yml`:

```yaml
read:
  - fablesmith/SKILL.md
  - readweave/SKILL.md
  - smolsmith/SKILL.md
  - smolweave/SKILL.md
```

> **Note:** Only Claude Code, OpenCode, Pi, and Codex CLI support invocable slash commands. The other agents will use the skill files as passive context/instructions.
