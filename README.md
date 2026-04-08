# commitclaw 🦞

**Note:** this repository is intended for an experimental research only. The code may break some day and there is no guarantee that the code will always work, so do not use this repository as-is in a production environment. The code was also mostly written by AI coding agents (Claude Opus 4.6 and GPT 5.4) with a few manual checks, so bugs may be present.

**Research experiment**: an autonomous AI developer powered by [OpenClaw](https://github.com/openclaw/openclaw) running entirely free in GitHub Actions using [Kilocode](https://kilo.ai) free models (with [OpenRouter](https://openrouter.ai) free as fallback).

## What This Does

A GitHub Actions workflow runs **every 8 hours** (or on manual trigger) and:

1. Installs [OpenClaw](https://docs.openclaw.ai) — an open-source personal AI assistant (Node.js/TypeScript)
2. Uses **[Kilocode](https://kilo.ai)** `kilo-auto/free` as the primary model — a smart-routing free tier managed by Kilo Gateway
3. Falls back to **[OpenRouter](https://openrouter.ai)** free models if Kilocode is unavailable
4. **Picks a target repository** from a configurable list (random selection when multiple are listed)
5. Resolves the **real GitHub identity** of the PAT owner — commits look like normal developer activity
6. Runs the OpenClaw agent in headless mode (`openclaw agent --local`) against the target repo
7. The agent reads its **memory file** (`.openclaw/memory.md`), makes a small incremental improvement, updates memory, and commits + pushes

Over time, the repositories evolve through small incremental changes — entirely autonomously and at zero cost.

## Setup

### 1. Get a Kilocode API Key (Free — Primary Provider)

1. Sign up at [kilo.ai](https://kilo.ai) (free)
2. Go to [app.kilo.ai](https://app.kilo.ai) and navigate to **API Keys**
3. Generate a new API key
4. The `kilo-auto/free` model costs $0 — no credit card needed

### 2. Get an OpenRouter API Key (Free — Fallback Provider)

1. Sign up at [openrouter.ai](https://openrouter.ai) (free)
2. Create an API key at [openrouter.ai/keys](https://openrouter.ai/keys)
3. Free models cost $0 — no credit card needed

### 3. Create a GitHub Personal Access Token

1. Go to [GitHub Settings → Developer Settings → Personal Access Tokens → Fine-grained tokens](https://github.com/settings/tokens?type=beta)
2. Create a new token with:
   - **Repository access**: select **all repositories** you want the agent to work on
   - **Permissions**: Contents (Read and write), Metadata (Read-only)
3. Copy the token

> **Note**: commits will use the **real username and email** of the token owner, so activity looks like normal developer work — not a bot.

### 4. Add Repository Secrets

In **this** repository (where the workflow lives), go to **Settings → Secrets and variables → Actions** and add:

| Type | Name | Value |
| --- | --- | --- |
| **Secret** | `KILOCODE_API_KEY` | Your Kilocode API key (primary provider) |
| **Secret** | `OPENROUTER_API_KEY` | Your OpenRouter API key (fallback provider) |
| **Secret** | `GH_PAT` | Your GitHub Personal Access Token |
| **Variable** | `ALLOWED_REPOS` | Comma-separated `owner/repo` list (see below) |

### 5. Configure Target Repositories

Set the **`ALLOWED_REPOS`** repository **variable** (Settings → Secrets and variables → Actions → Variables):

```
# Single repository
myuser/my-project

# Multiple repositories — one is picked at random each run
myuser/project-a, myuser/project-b, myuser/side-project
```

If `ALLOWED_REPOS` is not set, the workflow falls back to the repository it lives in (`${{ github.repository }}`).

### 6. Enable the Workflow

The workflow is at [`.github/workflows/openclaw-daily.yml`](.github/workflows/openclaw-daily.yml). It runs automatically on schedule, or you can trigger it manually from the **Actions** tab.

## How It Works

```
┌───────────────────────────────────────────────────────────┐
│                     GitHub Actions                         │
│                                                            │
│  schedule (every 8 hours) or manual trigger                │
│       │                                                    │
│       ▼                                                    │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ 1. Resolve PAT owner → real name + email             │  │
│  │ 2. Pick target repo (random from ALLOWED_REPOS)      │  │
│  │ 3. Clone target repo                                 │  │
│  │ 4. Install OpenClaw, configure models                │  │
│  │ 5. Run: openclaw agent --local                       │  │
│  │    ├─ Read .openclaw/memory.md (prior context)       │  │
│  │    ├─ Survey repo, plan improvement                  │  │
│  │    ├─ Make changes, update memory                    │  │
│  │    └─ git commit (real identity), git push           │  │
│  └──────────────────────────────────────────────────────┘  │
│                       │                                    │
│                       ▼                                    │
│            Target repository updated                       │
└───────────────────────────────────────────────────────────┘
                        │
            ┌───────────┴───────────┐
            ▼                       ▼
   Kilocode (primary)      OpenRouter (fallback)
   kilo-auto/free           free models
```

### Key Components

| Component | Role |
| --- | --- |
| **[OpenClaw](https://github.com/openclaw/openclaw)** | Open-source AI assistant framework with tool use, skills, and multi-provider model support |
| **[Kilocode](https://kilo.ai)** | Primary model provider — `kilo-auto/free` smart-routes to the best free model ($0 cost) |
| **[OpenRouter](https://openrouter.ai)** | Fallback model router with free-tier models ($0 cost) |
| **`openclaw agent --local`** | Runs a single agent turn in headless mode (no gateway needed) |
| **GitHub Skill** | Built-in OpenClaw skill that teaches the agent to use `gh` CLI |
| **Daily Developer Skill** | Custom workspace skill ([`skills/daily-developer/SKILL.md`](skills/daily-developer/SKILL.md)) guiding incremental improvements |
| **`.openclaw/memory.md`** | Lightweight memory file committed to each target repo — gives the agent context across runs |

### Multi-Repository Support

| Scenario | Configuration |
| --- | --- |
| **Single repo** | `ALLOWED_REPOS = myuser/my-project` |
| **Multiple repos (random)** | `ALLOWED_REPOS = myuser/repo-a, myuser/repo-b, myuser/repo-c` |
| **Manual override** | Use the `repositories` input when triggering manually |
| **Default (no config)** | Falls back to the repository this workflow lives in |

When multiple repos are listed, each scheduled run randomly selects **one** repository. Over time every listed repo receives roughly equal attention.

### Agent Memory

Since GitHub Actions runners are ephemeral, the agent stores its memory as a compact markdown file (`.openclaw/memory.md`) committed directly to each target repository:

```markdown
# Memory
2026-04-08: Created initial project structure with utils module
2026-04-09: Added fibonacci function to utils
2026-04-10: Wrote unit tests for utils.fibonacci
```

The agent reads this file at the start of each run and appends a one-line entry when done. If the file grows beyond ~80 lines, older entries are summarized to keep it token-friendly.

### Manual Trigger with Custom Task

From the **Actions** tab, click **Run workflow** and optionally provide:

- **Custom task**: e.g., `Create a Python utility module with a fibonacci function`
- **Target repos**: override the target (e.g., `myuser/specific-repo`)

## Configuration

### Changing the Schedule

Edit the cron expression in [`.github/workflows/openclaw-daily.yml`](.github/workflows/openclaw-daily.yml):

```yaml
schedule:
  - cron: "0 */8 * * *"  # Every 8 hours
  # - cron: "0 */12 * * *"  # Every 12 hours
  # - cron: "0 9 * * 1-5"   # Weekdays only
```

### Using a Specific Model

If you want to pin a specific model instead of `kilo-auto/free`, edit the `openclaw.json` config in the workflow:

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "kilocode/anthropic/claude-sonnet-4",
        "fallbacks": ["openrouter/google/gemini-2.5-flash-preview:free"]
      }
    }
  }
}
```

See [Kilocode available models](https://kilo.ai/docs/gateway/models-and-providers) and [OpenRouter free models](https://openrouter.ai/models?pricing=free) for options.

### Adjusting Agent Behavior

Modify the custom skill in [`skills/daily-developer/SKILL.md`](skills/daily-developer/SKILL.md) to change what kind of improvements the agent makes.

## Research Context

This experiment explores:

- **Autonomous AI development**: Can an AI agent make meaningful incremental improvements to a codebase over time?
- **Zero-cost AI infrastructure**: Running entirely on free-tier services (GitHub Actions + Kilocode free models + OpenRouter free fallback)
- **Agent skill composition**: Using OpenClaw's skill system to guide autonomous behavior
- **Stealth integration**: Commits use the real developer identity — the repository history looks organic
- **Persistent memory without infrastructure**: A committed `.openclaw/memory.md` file gives the agent cross-run context with no external storage

## File Structure

```
.
├── .github/
│   └── workflows/
│       └── openclaw-daily.yml    # Scheduled GitHub Actions workflow
├── skills/
│   └── daily-developer/
│       └── SKILL.md              # Custom OpenClaw skill for daily improvements
└── README.md                     # This file
```

Each **target repository** will also gain:

```
<target-repo>/
└── .openclaw/
    └── memory.md                 # Agent memory (created on first run)
```

## References

- [OpenClaw](https://github.com/openclaw/openclaw) — Open-source AI assistant
- [OpenClaw Docs](https://docs.openclaw.ai) — Full documentation
- [OpenClaw Models](https://docs.openclaw.ai/concepts/models) — Model configuration
- [OpenClaw Kilocode Provider](https://docs.openclaw.ai/providers/kilocode) — Kilocode provider setup
- [OpenClaw Skills](https://docs.openclaw.ai/tools/skills) — Skill system
- [Kilocode](https://kilo.ai) — Primary model provider with free-tier smart routing
- [Kilocode Models](https://kilo.ai/docs/gateway/models-and-providers) — Available models and providers
- [OpenRouter](https://openrouter.ai) — Fallback model router with free models
- [OpenRouter Free Models](https://openrouter.ai/models?pricing=free) — Available free models
