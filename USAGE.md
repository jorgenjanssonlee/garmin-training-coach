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

## Refresh MCP or re-authenticate

Upstream **[Taxuspt/garmin_mcp](https://github.com/Taxuspt/garmin_mcp)** updates often (new tools, auth changes). **`uv` caches** the Git install used by Cursor — refresh it from a terminal, then **toggle the `garmin` MCP off and on** in **Cursor > Settings > Cursor Settings > Tools & MCP** (or **Developer: Reload Window**) so Cursor runs the new build.

If **`uvx --python 3.12 --from git+https://github.com/Taxuspt/garmin_mcp garmin-mcp-auth --verify`** fails after an upstream auth change, run **`--force-reauth`** with the same **`uvx --from git+…`** pattern, then **`--verify`** again. Do **not** use **`uv run garmin-mcp-auth`** from this repo alone—it expects a local `garmin_mcp` Python project.

See **README → [Garmin MCP: upstream updates & authentication](README.md)** (same heading in the repo root README) for full commands and troubleshooting.

## Reduce MCP context size (optional)

The Garmin MCP registers **150 tools** by default. Every one of those adds to the tool inventory the AI has to carry in context each turn, even when it never calls them. If you don't use Garmin's nutrition logging, gamification badges, or women's-health features, you can shrink the surface to just what this coach actually touches.

Two env vars in the `garmin` MCP block of `.cursor/mcp.json` control it:

- **`GARMIN_ENABLED_TOOLS`** — comma-separated **allowlist**. If set, only these tools are registered.
- **`GARMIN_DISABLED_TOOLS`** — comma-separated **denylist**. Ignored if an allowlist is set.

Tool names are case-insensitive. Names that match no tool are skipped with a warning on the MCP server's stderr — that makes typos easy to spot.

### Recommended: denylist the tools you don't use

Simplest starting point — turn off the ~27 tools a running coach doesn't need. Add an `env` block to `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "garmin": {
      "command": "uvx",
      "args": [
        "--python", "3.12",
        "--from", "git+https://github.com/Taxuspt/garmin_mcp",
        "garmin-mcp"
      ],
      "env": {
        "GARMIN_DISABLED_TOOLS": "get_nutrition_daily_food_log,get_nutrition_summary_between_dates,get_nutrition_daily_meals,get_nutrition_daily_settings,set_nutrition_daily_settings,search_foods,get_custom_foods,get_custom_food_serving_units,create_custom_food,update_custom_food,delete_custom_food,log_custom_food,log_food,delete_food_log,upsert_and_log,get_earned_badges,get_adhoc_challenges,get_available_badge_challenges,get_badge_challenges,get_non_completed_badge_challenges,get_inprogress_virtual_challenges,get_pregnancy_summary,get_menstrual_data_for_date,get_menstrual_calendar_data,add_body_composition,set_blood_pressure,add_hydration_data"
      }
    }
  }
}
```

That removes:

- **All 15 nutrition tools** — food logs, meals, custom foods.
- **6 gamification tools** — earned badges, adhoc / badge / virtual challenges. `get_goals`, `get_race_predictions`, and `get_personal_record` live in the same upstream module but are **kept** — they're running-coaching-critical.
- **3 women's health tools** — pregnancy and menstrual data (remove from the list if applicable to you).
- **3 manual health-data writes** — body composition, blood pressure, hydration entries.

Leaves ~123 tools registered. Add or remove entries to match your setup — e.g. keep `add_body_composition` if you use it, drop `get_courses` / `download_course_gpx` / `upload_course` / `delete_course` if you don't build GPS routes, drop the 3 gear tools (`get_gear`, `get_gear_stats`, `get_gear_activities`) if you don't rotate shoes.

### Alternative: allowlist just the tools the coach uses

For maximum context savings, `GARMIN_ENABLED_TOOLS` gives you an explicit allowlist — the coach's `.mdc` files reference roughly 50 tools. Building the list from scratch is tedious, so a good workflow is: start with the denylist above, run the coach for a few weeks, note any tool the AI mentions or tries to call that isn't in your working set, then flip to `GARMIN_ENABLED_TOOLS` once the list is stable.

### After changing the env block

Toggle the `garmin` MCP off and on in **Cursor > Settings > Cursor Settings > Tools & MCP** (or run **Developer: Reload Window**) so Cursor restarts the server with the new environment. The tool count in the MCP panel updates immediately — that's how you confirm the filter took effect. If you see `unknown filter names` warnings on the MCP server's stderr, they list tool names that didn't match anything (usually a typo — the tool got renamed upstream, or you added an extra space).

## Getting Started — Athlete Profile Setup

### First run: copy the template

The shipped profile lives at **`docs/athlete-profile.template.md`**. On first use, copy it into place:

```sh
cp docs/athlete-profile.template.md .cursor/rules/athlete-profile.mdc
```

The live file at `.cursor/rules/athlete-profile.mdc` is **gitignored** — your personal data (weight, HR, races, injury history, name) stays local and never enters git history, even if you fork or push this repo somewhere. Cursor only loads the live copy; the template in `docs/` is never applied as a rule.

### Fill it in

The easiest way is to let the AI do it for you. Open a new Agent chat in Cursor and paste this prompt:

> "Run the onboarding flow: read my current athlete-profile.mdc, pull what you can from my Garmin data (resting HR, max HR, VO2max, recent training patterns, typical weekly structure, **time zone from `get_userprofile_settings`**), confirm or correct the time zone with me, and walk me through filling in the rest — goals, race targets, preferred interval style, training availability, injury history. Update the file when we're done."

The AI will fetch your Garmin data, ask you questions about things it can't determine automatically, and write everything to `athlete-profile.mdc`. You can re-run this any time your goals or circumstances change.

Alternatively, open `.cursor/rules/athlete-profile.mdc` and fill in the placeholder values manually.

### Merging upstream template updates

When this repo evolves the template (new fields, refined guidance, tighter zone tables), your live file **won't auto-update** — it's a local copy, independent from git. The template carries a `template_version:` stamp in its frontmatter and a changelog block near the top so you can tell when upstream has moved.

**Recommended workflow after `git pull`:**

1. Check whether the template changed since your last merge:

   ```sh
   git log -1 --oneline -- docs/athlete-profile.template.md
   ```

2. If it did, open `docs/athlete-profile.template.md` and read the changelog block near the top — it lists what changed in each version.

3. Copy the structural additions into your live file. Two ways:

   - **Manual diff:** open both side by side (e.g. `code --diff docs/athlete-profile.template.md .cursor/rules/athlete-profile.mdc`) and copy new sections/fields across while keeping your personal values intact.
   - **AI-assisted (usually easier):** in a fresh Agent chat, paste:

     > "Read `docs/athlete-profile.template.md` and my live `.cursor/rules/athlete-profile.mdc`. Update the live file with any new upstream fields, sections, or refined guidance from the template — but don't change any of my personal values. Show me the diff before writing."

4. Once you're happy with the merge, update the `template_version:` line in your live file's frontmatter to match the template's version. That marks your merge point for next time.

**Edge cases:**

- **Field renamed upstream** — the AI can usually spot this from context, but eyeball the diff before writing.
- **Field removed upstream** — decide whether the personal value belongs somewhere else (e.g. under "Other Notes") or can be dropped.
- **Section reorganised** — the AI will preserve your values under the new structure; verify no personal values were dropped.

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

| File                    | Purpose                                                      | Edit?                 |
| ----------------------- | ------------------------------------------------------------ | --------------------- |
| `running-coach.mdc`     | Core coaching persona, analysis methodology, tool reference  | Rarely                |
| `athlete-profile.mdc`   | **Your goals, zones, training preferences, injury history**  | **Yes -- start here** |
| `training-plans.mdc`    | Weekly plan output format, workout creation workflow         | Optional              |
| `workout-handling.mdc`  | Workout upload confirmation, reuse, step notes, construction | Optional              |
| `injury-prevention.mdc` | Health monitoring thresholds, red flags, response protocol   | Optional              |

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
