# Garmin Running Coach

An AI-powered running training coach that connects Cursor's AI agent to your Garmin ecosystem via MCP (Model Context Protocol). The agent can analyze your training data, health metrics, and recovery status, then create and push structured workouts directly to your Garmin device.

### Why Cursor?

This project started as a [ChatGPT Custom GPT pulling data from Strava](https://github.com/jorgenjanssonlee/ChatGPT-Running-coach-from-Strava-data). It worked, but had limitations that eventually pushed me to Cursor:

- **Better data** - Strava doesn't expose the health and wellness metrics Garmin collects (sleep, HRV, stress, body battery, training readiness). Cursor + MCP pulls directly from Garmin Connect (150 tools) and gets the full picture.
- **Direct Garmin interaction** - ChatGPT Custom GPTs couldn't talk to Garmin at all. Creating workouts meant copy-pasting from the chat into Connect's workout builder. Now the AI creates and schedules workouts on the device.
- **Model choice** - Cursor supports multiple LLM providers (Claude, GPT-4, Gemini, etc.), not just ChatGPT.
- **Persistent coaching rules** - Cursor Rules give the AI consistent coaching behavior across conversations, similar to GPT Instructions but with more flexibility.
- **Cost consolidation** - I already had a Cursor subscription for work, so this solution comes at no extra cost. ChatGPT requires a paid plan for Custom GPTs, and Claude requires a paid plan for Projects.
- **Familiarity** - I already spend a lot of time in Cursor for work, so using the same tool for coaching keeps everything in one place.

**Trade-off:** Cursor is a [developer-oriented IDE](https://cursor.com). If you're not comfortable with a code editor and terminal commands, the [ChatGPT + Strava approach](https://github.com/jorgenjanssonlee/ChatGPT-Running-coach-from-Strava-data) is simpler to get running.

### Credits & Acknowledgements

This project would not be possible without:

- **[Taxuspt/garmin_mcp](https://github.com/Taxuspt/garmin_mcp)** -- The Garmin Connect MCP server that makes this entire project work. Exposes 150 tools plus 5 workout-template resources across 17 modules. Without this, there is no AI coaching.
- **[cyberjunky/python-garminconnect](https://github.com/cyberjunky/python-garminconnect)** -- The Python library that `garmin_mcp` is built on. Provides the underlying Garmin Connect API client.
- **[AI-Powered Triathlon Coaching](https://dzone.com/articles/ai-powered-triathlon-coaching-claude-garmin)** (DZone, 2025) -- The article that inspired this project.

## How It Works

A single MCP server ([Taxuspt/garmin_mcp](https://github.com/Taxuspt/garmin_mcp)) bridges Cursor to Garmin Connect, providing 150 tools covering activities, health metrics, workouts, training performance, gear, nutrition, and more.

Cursor Rules provide persistent coaching instructions so the AI agent behaves as an experienced running coach across every conversation.

## Project Structure

```
garmin-training-coach/
├── .cursor/
│   ├── mcp.json                       # MCP server configuration
│   └── rules/
│       ├── running-coach.mdc          # Core coaching persona and analysis methodology
│       ├── athlete-profile.mdc        # Your goals, zones, preferences — gitignored;
│       │                              #   copy from docs/ on first run, then edit this
│       ├── training-plans.mdc         # Plan output format and workout workflow
│       ├── workout-handling.mdc       # Workout upload/schedule rules, reuse, notes, construction
│       └── injury-prevention.mdc      # Health monitoring and red flags
├── docs/
│   └── athlete-profile.template.md    # Shipped profile template (upstream-owned);
│                                      #   copy this to .cursor/rules/athlete-profile.mdc on first run
├── .gitignore
├── LICENSE
├── README.md                          # Overview and installation
├── USAGE.md                           # Day-to-day usage guide
```

**Why the split?** `athlete-profile.mdc` holds your personal data (goals, weight, HR, races, injury history). Keeping it gitignored means that data stays local — never enters git history, even if you fork this repo. `docs/athlete-profile.template.md` is the upstream-owned template you copy from on first run; when the template evolves, you can pull upstream and merge the changes into your live file at your leisure. Full first-run and merge workflows are in [USAGE.md](USAGE.md).

## Prerequisites

- **OS:** macOS or Windows (both are supported)
- [Cursor IDE](https://cursor.com) (free plan works, Pro recommended for regular use)
- A Garmin Connect account with a synced Garmin device
- Python 3.12+ and [uv](https://docs.astral.sh/uv/) (see installation steps below)

## Installation

> **Security note:** No credentials are stored in any project file. Authentication uses pre-saved tokens in `~/.garminconnect` (macOS) or `%USERPROFILE%\.garminconnect` (Windows). Neither your password nor tokens are shared with the AI or any third party.

macOS

### 1. Install Python 3.12 and uv (if not already installed)

```bash
brew install python@3.12 uv
```

These install alongside any existing Python versions without replacing them. `uv` is a fast Python package manager used to run the MCP server.

### 2. Clone or create the project directory

Either clone this repo:

```bash
cd ~/Documents
git clone https://github.com/jorgenjanssonlee/garmin-training-coach.git
```

Or create the directory manually and copy `.cursor/` plus `docs/athlete-profile.template.md` from this repo into it (you still need the template for first-run profile setup — see [USAGE.md](USAGE.md)):

```bash
mkdir -p ~/Documents/garmin-training-coach/.cursor/rules
mkdir -p ~/Documents/garmin-training-coach/docs
```

### 3. Authenticate with Garmin Connect

Run the one-time authentication tool:

```bash
uvx --python 3.12 --from git+https://github.com/Taxuspt/garmin_mcp garmin-mcp-auth
```

You'll be prompted for your Garmin email, password, and MFA code (if enabled). OAuth tokens are saved to `~/.garminconnect`.

To verify tokens later: `uvx --python 3.12 --from git+https://github.com/Taxuspt/garmin_mcp garmin-mcp-auth --verify`

To force re-auth when tokens expire: `uvx --python 3.12 --from git+https://github.com/Taxuspt/garmin_mcp garmin-mcp-auth --force-reauth`

### 4. Verify MCP server in Cursor

1. Open the `garmin-training-coach` folder as a workspace (File > Open Folder)
2. Open **Customize** in the sidebar → **MCPs**
3. `garmin` should appear. Toggle it **on**. Connected (green, if shown) = running.
4. If it fails to connect, verify tokens with the full `uvx` line in **Garmin MCP: upstream updates & authentication** below. For logs: Output panel → **MCP Logs** (Cmd+Shift+U on Mac).

Windows

### 1. Install Python 3.12 and uv (if not already installed)

**Option A — winget:**

```powershell
winget install Python.Python.3.12
winget install astral-sh.uv
```

**Option B — Python.org + pip:**

1. Download Python 3.12 from [python.org](https://www.python.org/downloads/)
2. Install with "Add Python to PATH" checked
3. Open PowerShell and run: `pip install uv`

`uv` is a fast Python package manager used to run the MCP server.

### 2. Clone or create the project directory

Either clone this repo:

```powershell
cd $env:USERPROFILE\Documents
git clone https://github.com/jorgenjanssonlee/garmin-training-coach.git
```

Or create the directory manually and copy `.cursor/` plus `docs/athlete-profile.template.md` from this repo (you still need the template for first-run profile setup — see [USAGE.md](USAGE.md)):

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\Documents\garmin-training-coach\.cursor\rules"
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\Documents\garmin-training-coach\docs"
```

### 3. Authenticate with Garmin Connect

Run the one-time authentication tool in PowerShell:

```powershell
uvx --python 3.12 --from git+https://github.com/Taxuspt/garmin_mcp garmin-mcp-auth
```

You'll be prompted for your Garmin email, password, and MFA code (if enabled). OAuth tokens are saved to `%USERPROFILE%\.garminconnect`.

To verify tokens later: `uvx --python 3.12 --from git+https://github.com/Taxuspt/garmin_mcp garmin-mcp-auth --verify`

To force re-auth when tokens expire: `uvx --python 3.12 --from git+https://github.com/Taxuspt/garmin_mcp garmin-mcp-auth --force-reauth`

### 4. Verify MCP server in Cursor

1. Open the `garmin-training-coach` folder as a workspace (File > Open Folder)
2. Open **Customize** in the sidebar → **MCPs**
3. `garmin` should appear. Toggle it **on**. Connected (green, if shown) = running.
4. If it fails to connect, verify tokens with the full `uvx` line in **Garmin MCP: upstream updates & authentication** below. For logs: Output panel → **MCP Logs** (Ctrl+Shift+U on Windows).

## Garmin MCP: upstream updates & authentication

Cursor runs the server via `uvx --from git+https://github.com/Taxuspt/garmin_mcp` (see `.cursor/mcp.json`). `uv` **caches** that Git install, so you are not guaranteed the latest upstream `main` until you run an explicit refresh.

### Refresh the MCP package from Git

In a terminal (any directory; macOS or Windows):

```bash
uvx --refresh --python 3.12 --from git+https://github.com/Taxuspt/garmin_mcp garmin-mcp --help
```

(`garmin-mcp-auth --help` works too—it is the **same** package.) If the cache still looks stale, use `--reinstall` or `-U` instead of `--refresh`.

### Make Cursor use the new build

After refreshing: open **Customize → MCPs**, toggle **garmin** off then on. Or run **Developer: Reload Window** from the Command Palette. That restarts the MCP process so it picks up the refreshed install. If status or tool count doesn't update, restart Cursor.

### Authentication changes & re-login

Upstream `garmin_mcp` / `python-garminconnect` have **switched to a new Garmin login path** in May 2026 (see e.g. [garmin_mcp#77](https://github.com/Taxuspt/garmin_mcp/pull/77)). Older saved tokens may fail `--verify`. Re-authenticate interactively (MFA in the terminal when prompted):

```bash
uvx --python 3.12 --from git+https://github.com/Taxuspt/garmin_mcp garmin-mcp-auth --force-reauth
```

Then:

```bash
uvx --python 3.12 --from git+https://github.com/Taxuspt/garmin_mcp garmin-mcp-auth --verify
```

Toggle **garmin** off/on under **Customize → MCPs** again.

## Next Steps

Installation complete. See **[USAGE.md](USAGE.md)** for:

- **First-run setup:** copy `docs/athlete-profile.template.md` → `.cursor/rules/athlete-profile.mdc`, then run the onboarding flow (including time zone confirmation)
- **Merging upstream template updates** into your live profile after `git pull`
- Example prompts and how to talk to the coach
- Customizing the coaching rules

## Available Data/Tools

See **[Available MCP Tools (Taxuspt/garmin_mcp — tool coverage)](https://github.com/Taxuspt/garmin_mcp/blob/main/README.md#tool-coverage)** for a complete list of tools and examples.

## Troubleshooting

| Problem                                        | Solution                                                                                                                                                                                                                |
| ---------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| MCP server disconnected / red under **Customize → MCPs** | `uvx --python 3.12 --from git+https://github.com/Taxuspt/garmin_mcp garmin-mcp-auth --verify` — if invalid, `--force-reauth`, then toggle **garmin** off/on under **Customize → MCPs** (see **Garmin MCP: upstream updates & authentication**). Check **MCP Logs** in the Output panel. |
| `uvx` command not found                        | macOS: `brew install uv`. Windows: `winget install astral-sh.uv` or `pip install uv`. Restart Cursor after installing.                                                                                                  |
| MCP tools not appearing in agent chat          | Make sure you opened this folder as the Cursor workspace. MCP config is project-level. Confirm `garmin` is toggled on under **Customize → MCPs**.                                                                      |
| Want latest upstream `garmin_mcp` after merges | `uvx --refresh --python 3.12 --from git+https://github.com/Taxuspt/garmin_mcp garmin-mcp --help`, then toggle **garmin** off/on under **Customize → MCPs**                                                             |
| Tokens expired                                 | Same as disconnected MCP: `uvx … garmin-mcp-auth --force-reauth`                                                                                                                                                       |
| Garmin MFA required                            | Run `uvx … garmin-mcp-auth` (or `--force-reauth`) in an interactive terminal                                                                                                                                            |
