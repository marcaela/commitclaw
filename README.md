# claw 🦞

**Research experiment**: an autonomous AI developer powered by [OpenClaw](https://github.com/openclaw/openclaw) running entirely free in GitHub Actions using [OpenRouter](https://openrouter.ai) free models.

## What This Does

A GitHub Actions workflow runs **once per day** (or on manual trigger) and:

1. Installs [OpenClaw](https://docs.openclaw.ai) — an open-source personal AI assistant (Node.js/TypeScript)
2. Discovers the best available **free** model on OpenRouter via `openclaw models scan`
3. Runs the OpenClaw agent in headless mode (`openclaw agent --local`)
4. The agent examines the repository, makes a small incremental improvement, and commits + pushes

Over time, the repository evolves through small daily changes — entirely autonomously and at zero cost.

## Setup

### 1. Get an OpenRouter API Key (Free)

1. Sign up at [openrouter.ai](https://openrouter.ai) (free)
2. Create an API key at [openrouter.ai/keys](https://openrouter.ai/keys)
3. Free models cost $0 — no credit card needed

### 2. Create a GitHub Personal Access Token

1. Go to [GitHub Settings → Developer Settings → Personal Access Tokens → Fine-grained tokens](https://github.com/settings/tokens?type=beta)
2. Create a new token with:
   - **Repository access**: select this repository
   - **Permissions**: Contents (Read and write), Metadata (Read-only)
3. Copy the token

### 3. Add Repository Secrets

In your repository, go to **Settings → Secrets and variables → Actions** and add:

| Secret Name         | Value                              |
| ------------------- | ---------------------------------- |
| `OPENROUTER_API_KEY` | Your OpenRouter API key            |
| `GH_PAT`            | Your GitHub Personal Access Token  |

### 4. Enable the Workflow

The workflow is at [`.github/workflows/openclaw-daily.yml`](.github/workflows/openclaw-daily.yml). It runs automatically on schedule, or you can trigger it manually from the **Actions** tab.

## How It Works

```
┌─────────────────────────────────────────────────────┐
│                  GitHub Actions                      │
│                                                      │
│  schedule (daily 9 AM UTC) or manual trigger         │
│       │                                              │
│       ▼                                              │
│  ┌─────────────────────────────────────────────┐     │
│  │ 1. Install OpenClaw (npm install -g)        │     │
│  │ 2. Scan OpenRouter free model catalog       │     │
│  │ 3. Set best free model as primary           │     │
│  │ 4. Run: openclaw agent --local              │     │
│  │    └─ Agent reads repo, plans improvement   │     │
│  │    └─ Makes changes, git commit, git push   │     │
│  └─────────────────────────────────────────────┘     │
│                      │                               │
│                      ▼                               │
│              Repository updated                      │
└─────────────────────────────────────────────────────┘
                       │
                       ▼
              OpenRouter (free models)
              e.g., Gemini, Llama, etc.
```

### Key Components

| Component | Role |
| --- | --- |
| **[OpenClaw](https://github.com/openclaw/openclaw)** | Open-source AI assistant framework with tool use, skills, and multi-provider model support |
| **[OpenRouter](https://openrouter.ai)** | Model router with free-tier models ($0 cost) |
| **`openclaw models scan`** | Discovers best available free models from OpenRouter's catalog |
| **`openclaw agent --local`** | Runs a single agent turn in headless mode (no gateway needed) |
| **GitHub Skill** | Built-in OpenClaw skill that teaches the agent to use `gh` CLI |
| **Daily Developer Skill** | Custom workspace skill ([`skills/daily-developer/SKILL.md`](skills/daily-developer/SKILL.md)) guiding incremental improvements |

### Manual Trigger with Custom Task

From the **Actions** tab, click **Run workflow** and optionally provide a custom task:

```
Create a Python utility module with a function that generates Fibonacci numbers
```

## Configuration

### Changing the Schedule

Edit the cron expression in [`.github/workflows/openclaw-daily.yml`](.github/workflows/openclaw-daily.yml):

```yaml
schedule:
  - cron: "0 9 * * *"   # Daily at 9 AM UTC
  # - cron: "0 */12 * * *"  # Every 12 hours
  # - cron: "0 9 * * 1-5"   # Weekdays only
```

### Using a Specific Free Model

If you want to pin a specific model instead of auto-scanning, edit the `openclaw.json` config in the workflow:

```json
{
  "agents": {
    "defaults": {
      "model": {
        "primary": "openrouter/google/gemini-2.5-flash-preview:free"
      }
    }
  }
}
```

### Adjusting Agent Behavior

Modify the custom skill in [`skills/daily-developer/SKILL.md`](skills/daily-developer/SKILL.md) to change what kind of improvements the agent makes.

## Research Context

This experiment explores:

- **Autonomous AI development**: Can an AI agent make meaningful incremental improvements to a codebase over time?
- **Zero-cost AI infrastructure**: Running entirely on free-tier services (GitHub Actions + OpenRouter free models)
- **Agent skill composition**: Using OpenClaw's skill system to guide autonomous behavior

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

## References

- [OpenClaw](https://github.com/openclaw/openclaw) — Open-source AI assistant
- [OpenClaw Docs](https://docs.openclaw.ai) — Full documentation
- [OpenClaw Models](https://docs.openclaw.ai/concepts/models) — Model configuration
- [OpenClaw Skills](https://docs.openclaw.ai/tools/skills) — Skill system
- [OpenRouter](https://openrouter.ai) — Model router with free models
- [OpenRouter Free Models](https://openrouter.ai/models?pricing=free) — Available free models