# Usage Guide

Once installation is complete and the MCP server shows green in Cursor settings, you're ready to start.

## Chat Mode

Always use **Agent mode** (not Ask mode) when talking to the coach. Agent mode allows the AI to call MCP tools to fetch your Garmin data and edit your `.mdc` configuration files. Ask mode is read-only and cannot call any tools.

## Tool Approval

The first time the AI calls a Garmin tool, Cursor will show an approval prompt asking you to confirm. This is a security feature — it prevents the AI from making unauthorized external calls.

You have three options:
- **Run** — approve this single call. You'll see the prompt again next time this tool is used.
- **Allowlist MCP Tool** — permanently approve this specific tool. Recommended for tools the coach uses frequently (activity data, health metrics, etc.).
- **Skip** — decline the call.

Since the coach calls Garmin tools frequently, it's easiest to click **Allowlist MCP Tool** each time a new tool comes up. After a few sessions most tools will be allowlisted and you won't see the prompt anymore.

## Getting Started — Athlete Profile Setup

Before using the coach, set up your athlete profile. The easiest way is to let the AI do it for you. Open a new Agent chat in Cursor and paste this prompt:

> "Run the onboarding flow: read my current athlete-profile.mdc, pull what you can from my Garmin data (resting HR, max HR, VO2max, recent training patterns, typical weekly structure, **time zone from `get_userprofile_settings`**), confirm or correct the time zone with me, and walk me through filling in the rest — goals, race targets, preferred interval style, training availability, injury history. Update the file when we're done."

The AI will fetch your Garmin data, ask you questions about things it can't determine automatically, and write everything to `athlete-profile.mdc`. You can re-run this any time your goals or circumstances change.

Alternatively, open `.cursor/rules/athlete-profile.mdc` and fill in the placeholder values manually.

## Talking to the Coach

Open a new Agent chat in Cursor (within this project workspace) and interact naturally. The coaching rules are loaded automatically from the `.mdc` files.

### Analyze recent training

> "Look at my running activities from the last two weeks and tell me how my training is going."

### Check recovery status

> "Check my sleep, stress, body battery, HRV, and training readiness. Am I recovered enough for a hard workout today?"

### Get a workout recommendation

> "I have 45 minutes to run tomorrow morning. Create an appropriate workout based on my recent training load and push it to my Garmin."

### Build a weekly plan

> "Plan my running for next week. I'm training for a 10K race in 6 weeks and can run 5 days per week."

### Investigate a specific run

> "Analyze my last long run. How was my heart rate, cadence, and pace distribution?"

### Check race readiness

> "What are my current race predictions? How has my VO2max been trending?"

### Override preferences on the fly

You can override any stored preference directly in chat:

> "Focus on building VO2max for the next month"

> "I want to add hill sprints for the next 5 weeks"

> "Switch to a 6-week peak phase with only 2 weeks taper"

The AI will apply the change immediately and ask whether you want to update your default preferences in the `.mdc` files or keep it as a temporary change.

## Customization and fine-tuning

The coaching instructions are split across four Cursor Rules files (`.mdc`) in `.cursor/rules/`. The `.cursor` directory is hidden in macOS Finder but fully visible inside Cursor's file explorer -- browse to it there to edit the files.

| File                    | Purpose                                                         | Edit?                 |
| ----------------------- | --------------------------------------------------------------- | --------------------- |
| `running-coach.mdc`     | Core coaching persona, analysis methodology, tool reference     | Rarely                |
| `athlete-profile.mdc`   | **Your goals, zones, training preferences, injury history**     | **Yes -- start here** |
| `training-plans.mdc`    | Weekly plan output format, workout creation workflow             | Optional              |
| `workout-handling.mdc`  | Workout upload confirmation, reuse, step notes, construction     | Optional              |
| `injury-prevention.mdc` | Health monitoring thresholds, red flags, response protocol    | Optional              |

**To personalise the coach, edit `athlete-profile.mdc`** (or run the onboarding flow above, or ask the chat to update them for you). Key sections:

- Race calendar (primary A-race + secondary B/C races)
- Training availability and time constraints
- Heart rate zones and LTHR (auto-populated from Garmin)
- Interval preferences (Norwegian singles, short HIIT, etc.)
- Warm-up and cool-down durations
- Training philosophy (80/20, progressive overload, etc.)
- Injury history

The other three files contain general coaching methodology that works for any runner. You can edit them manually or ask the AI to update them for you in chat. Examples:

**athlete-profile.mdc:**

> "Update my race calendar — my A-race is now the Sydney Marathon on Sep 20, goal sub-3:45"

> "Add a note to my injury history that I had a calf strain in January"

**training-plans.mdc:**

> "I prefer my weekly plan output to include a column for RPE targets"

> "Change the default taper to 3 weeks for marathon distance"

**running-coach.mdc:**

> "Be more detailed in activity analysis — always include cadence and GCT breakdown"

> "When analysing intervals, always compare to the previous time I did the same session"

**injury-prevention.mdc:**

> "I'm sensitive to high training load — lower the acute-to-chronic ratio warning to 1.2"

> "Add shin splints to the red flags — I have a history of tibial stress"
