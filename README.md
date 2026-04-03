# Garmin Running Coach

An AI-powered running training coach that connects Cursor's AI agent to your Garmin ecosystem via MCP (Model Context Protocol). The agent can analyze your training data, health metrics, and recovery status, then create and push structured workouts directly to your Garmin device.

### Background

This project replaces an earlier approach using [ChatGPT Custom GPTs with Strava data](https://github.com/jorgenjanssonlee/ChatGPT-Running-coach-from-Strava-data). Compared to that setup, this offers:

- **Richer data** -- pulls directly from Garmin Connect (96+ data tools) instead of Strava's more limited API, including health metrics (sleep, HRV, stress, body battery, training readiness) that Strava doesn't expose
- **Workout creation** -- generates structured workouts and uploads them directly to your Garmin device
- **Model choice** -- Cursor supports multiple LLM providers (Claude, GPT-4, Gemini, etc.), not just ChatGPT
- **Persistent coaching rules** -- Cursor Rules give the AI consistent coaching behavior across conversations, similar to GPT Instructions but with more flexibility

**Trade-off:** This requires [Cursor IDE](https://cursor.com), which is a developer-oriented tool. If you're not comfortable with a code editor and terminal commands, the [ChatGPT + Strava approach](https://github.com/jorgenjanssonlee/ChatGPT-Running-coach-from-Strava-data) may be more accessible.

### Why Cursor?

This project started as a [ChatGPT Custom GPT pulling data from Strava](https://github.com/jorgenjanssonlee/ChatGPT-Running-coach-from-Strava-data). It worked, but had limitations that eventually pushed me to Cursor:

- **Cost consolidation** -- I already had a Cursor subscription for work, so this solution comes at no extra cost. ChatGPT requires a paid plan for Custom GPTs, and Claude requires a paid plan for Projects. AI subscription costs add up quickly, and consolidating into one tool makes sense.
- **Better data** -- Strava doesn't expose the health and wellness metrics that Garmin collects (sleep, HRV, stress, body battery, training readiness, SpO2). With Cursor + MCP, the AI pulls directly from Garmin Connect and gets the full picture.
- **Direct Garmin interaction** -- ChatGPT Custom GPTs couldn't interact with Garmin at all. Creating workouts meant copy-pasting from the chat into Garmin Connect's workout builder, which got tedious even after streamlining the format. Now the AI creates and schedules workouts directly on the device.
- **Familiarity** -- I already spend a lot of time in Cursor for work, so using the same tool for coaching keeps everything in one place.

That said, Cursor is a developer tool. If you're not technical, the ChatGPT + Strava setup is simpler to get running.

### Credits & Acknowledgements

This project would not be possible without:

- **[Taxuspt/garmin_mcp](https://github.com/Taxuspt/garmin_mcp)** -- The Garmin Connect MCP server that makes this entire project work. Exposes 95+ tools covering ~88% of the Garmin Connect API via MCP. Without this, there is no AI coaching.
- **[cyberjunky/python-garminconnect](https://github.com/cyberjunky/python-garminconnect)** -- The Python library that `garmin_mcp` is built on. Provides the underlying Garmin Connect API client.
- **[matin/garth](https://github.com/matin/garth)** -- Garmin SSO authentication library. Handles OAuth token management so credentials never need to be stored in project files.
- **[AI-Powered Triathlon Coaching](https://dzone.com/articles/ai-powered-triathlon-coaching-claude-garmin)** (DZone, 2025) -- The article that inspired this project.

## How It Works

A single MCP server ([Taxuspt/garmin_mcp](https://github.com/Taxuspt/garmin_mcp)) bridges Cursor to Garmin Connect, providing 96+ tools covering activities, health metrics, workouts, training performance, gear, nutrition, and more.

Cursor Rules provide persistent coaching instructions so the AI agent behaves as an experienced running coach across every conversation.

## Project Structure

```
garmin-training-coach/
├── .cursor/
│   ├── mcp.json                  # MCP server configuration
│   └── rules/
│       ├── running-coach.mdc     # Core coaching persona and analysis methodology
│       ├── athlete-profile.mdc   # Your goals, zones, preferences (edit this one)
│       ├── training-plans.mdc    # Plan output format and workout workflow
│       ├── workout-handling.mdc  # Workout upload/schedule rules, reuse, notes, construction
│       └── injury-prevention.mdc # Health monitoring and red flags
├── .gitignore
├── README.md                     # Overview and installation
├── USAGE.md                      # Day-to-day usage guide
```

## Prerequisites

- **OS:** macOS or Windows (both are supported)
- [Cursor IDE](https://cursor.com) (free plan works, Pro recommended for regular use)
- A Garmin Connect account with a synced Garmin device
- Python 3.12+ and [uv](https://docs.astral.sh/uv/) (see installation steps below)

## Installation

> **Security note:** No credentials are stored in any project file. Authentication uses pre-saved tokens in `~/.garminconnect` (macOS) or `%USERPROFILE%\.garminconnect` (Windows). Neither your password nor tokens are shared with the AI or any third party.

<details>
<summary>macOS</summary>

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

Or create the directory manually and copy the `.cursor/` config files from this repo into it:

```bash
mkdir -p ~/Documents/garmin-training-coach/.cursor/rules
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
2. Go to **Cursor > Settings > Cursor Settings > Tools & MCP**
3. `garmin` should appear. Toggle to **enabled**. Green = running.
4. If red, verify tokens with `garmin-mcp-auth --verify`

</details>

<details>
<summary>Windows</summary>

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

Or create the directory manually and copy the `.cursor/` config files from this repo:

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\Documents\garmin-training-coach\.cursor\rules"
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
2. Go to **File > Preferences > Cursor Settings > Tools & MCP** (or open Settings and search for "MCP")
3. `garmin` should appear. Toggle to **enabled**. Green = running.
4. If red, verify tokens with `garmin-mcp-auth --verify`

</details>

## Next Steps

Installation complete. See **[USAGE.md](USAGE.md)** for:
- Setting up your athlete profile (onboarding flow, including time zone confirmation)
- Example prompts and how to talk to the coach
- Customizing the coaching rules

## Available Data (96+ tools)

| Category | Tools | Examples |
|---|---|---|
| Activity Management | 14 | List activities, detailed stats, splits, HR zones, weather |
| Health & Wellness | 31 | Sleep, HR, HRV, stress, body battery, steps, SpO2, respiration, training readiness |
| Training & Performance | 9 | VO2max, fitness age, race predictions, training status, endurance/hill scores |
| Workouts | 8 | Create, upload, schedule, delete structured workouts |
| Weight & Body Composition | 5 | Weight tracking, body fat, muscle mass |
| Gear Management | 5 | Track shoes, equipment stats |
| Nutrition | 8 | Food logs, meals, custom foods |
| Devices | 7 | Connected devices, settings, alarms |
| Challenges & Badges | 10 | Active challenges, earned badges |
| User Profile | 3 | Profile settings, unit system |

## Troubleshooting

| Problem | Solution |
|---|---|
| MCP server shows red in Cursor settings | Run `garmin-mcp-auth --verify` to check token validity. Re-auth with `garmin-mcp-auth --force-reauth` if needed |
| `uvx` command not found | macOS: `brew install uv`. Windows: `winget install astral-sh.uv` or `pip install uv`. Restart Cursor after installing. |
| MCP tools not appearing in agent chat | Make sure you opened this folder as the Cursor workspace. MCP config is project-level. |
| Tokens expired | Run `garmin-mcp-auth --force-reauth` |
| Garmin MFA required | Run `garmin-mcp-auth` interactively in terminal -- you'll be prompted for the MFA code |

