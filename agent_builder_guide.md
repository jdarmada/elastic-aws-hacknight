# ⚽ World Cup 2026 Predictor - Elastic Agent Builder Guide

After running the notebook to ingest live 2026 match data, this guide walks you through building a conversational AI agent on top of it in **Elastic Agent Builder** - no extra code required.

You'll build three custom ES|QL tools and wire them into a custom agent with a football analyst persona. The agent will use real 2026 results to compare teams and predict upcoming matchups.

---

## Prerequisites

- Elastic Serverless project running
- `wc2026_matches` index populated (run the notebook first)
- Agent Builder enabled (on by default on Serverless)

---

## Navigate to Agent Builder


Look for **Agents** in the main navigation.


---

## Quick Setup via Dev Tools Console (fast path)

If you just want everything created in one go, paste the API commands below into Kibana's **Dev Tools Console** instead of clicking through the UI.

Open the **hamburger menu (top left) → Management → Dev Tools**, paste each block, and press the **play button**. Dev Tools uses your current Kibana session, so there's no endpoint to find and no auth headers to set.

Run them in order: the three tools first, then the agent. Once all four have run, open **Kibana → Agents** and the **World Cup 2026 Predictor** is ready to chat.

> Prefer to understand each tool field-by-field, or tweak things in the UI? Skip this section and follow the manual **Step 1 - Step 5** walkthrough below instead. The two paths produce the same result.

### Tool 1: `get_team_form`

```
POST kbn://api/agent_builder/tools
{
  "id": "get_team_form",
  "type": "esql",
  "description": "Returns a team's 2026 World Cup results so far - wins, losses, draws, goals scored and conceded, and each match result with opponent, date, and stage. Use this when the user asks about a team's current form, recent results, or how they have performed in the tournament.",
  "configuration": {
    "query": "FROM wc2026_matches | WHERE status == \"played\" AND (team1 == ?team_name OR team2 == ?team_name) | EVAL goals_scored = CASE(team1 == ?team_name, score_ft1, score_ft2), goals_conceded = CASE(team1 == ?team_name, score_ft2, score_ft1), result = CASE(winner == ?team_name, \"win\", winner == \"draw\", \"draw\", \"loss\") | KEEP date, team1, team2, score_ft1, score_ft2, group, stage, result, goals_scored, goals_conceded, winner, stadium | SORT date ASC",
    "params": {
      "team_name": {
        "type": "string",
        "description": "The team name exactly as it appears in the data, e.g. France"
      }
    }
  }
}
```

### Tool 2: `get_team_stats_2026`

```
POST kbn://api/agent_builder/tools
{
  "id": "get_team_stats_2026",
  "type": "esql",
  "description": "Returns aggregated 2026 tournament statistics for a team: total matches played, wins, draws, losses, goals scored, and goals conceded. Use this when comparing two teams statistically or assessing overall tournament performance.",
  "configuration": {
    "query": "FROM wc2026_matches | WHERE status == \"played\" AND (team1 == ?team_name OR team2 == ?team_name) | EVAL goals_scored = CASE(team1 == ?team_name, score_ft1, score_ft2), goals_conceded = CASE(team1 == ?team_name, score_ft2, score_ft1), is_win = CASE(winner == ?team_name, 1, 0), is_draw = CASE(winner == \"draw\", 1, 0), is_loss = CASE(winner != ?team_name AND winner != \"draw\", 1, 0) | STATS matches_played = COUNT(*), wins = SUM(is_win), draws = SUM(is_draw), losses = SUM(is_loss), total_scored = SUM(goals_scored), total_conceded = SUM(goals_conceded)",
    "params": {
      "team_name": {
        "type": "string",
        "description": "The team name, e.g. Germany"
      }
    }
  }
}
```

### Tool 3: `get_upcoming_fixtures`

```
POST kbn://api/agent_builder/tools
{
  "id": "get_upcoming_fixtures",
  "type": "esql",
  "description": "Returns upcoming scheduled matches for a team that have not yet been played. Use this when the user asks when a team plays next, who their next opponent is, or to identify a real upcoming matchup to predict.",
  "configuration": {
    "query": "FROM wc2026_matches | WHERE status == \"upcoming\" AND (team1 == ?team_name OR team2 == ?team_name) | KEEP date, round, group, team1, team2, stadium | SORT date ASC",
    "params": {
      "team_name": {
        "type": "string",
        "description": "The team name, e.g. England"
      }
    }
  }
}
```

### Agent: `wc2026_predictor`

```
POST kbn://api/agent_builder/agents
{
  "id": "wc2026_predictor",
  "name": "World Cup 2026 Predictor",
  "description": "Ask me anything about the 2026 World Cup - form, stats, upcoming matches, and predictions based on real tournament data.",
  "labels": ["worldcup", "hacknight"],
  "avatar_color": "#16C47F",
  "avatar_symbol": "⚽",
  "configuration": {
    "instructions": "You are a football analyst specialising in the 2026 FIFA World Cup, currently underway in the USA, Mexico, and Canada.\n\nYou have access to live tournament data: completed results, upcoming fixtures, and team form.\n\nWhen a user asks you to predict a match:\n1. Call get_upcoming_fixtures for each team to confirm they are actually scheduled to play\n2. Call get_team_stats_2026 for both teams to get their tournament numbers\n3. Call get_team_form for both teams to see their actual match-by-match results\n4. Produce a prediction structured as:\n   - Current form summary for each team (results so far, goals scored/conceded)\n   - Statistical edge: which team has the data advantage and why\n   - Key factors: 2-3 things that will decide this match\n   - Predicted scoreline and winner\n   - Confidence level: Low / Medium / High with one sentence of reasoning\n\nAlways ground your predictions in the 2026 data - this is a live tournament, not historical analysis.\n\nKeep your tone punchy and engaging. \n\nOnly answer questions about the 2026 World Cup. Politely decline anything off-topic.",
    "tools": [
      {
        "tool_ids": [
          "get_team_form",
          "get_team_stats_2026",
          "get_upcoming_fixtures"
        ]
      }
    ]
  }
}
```

---

## Manual Walkthrough (UI, step by step)

The remaining steps build the exact same three tools and agent through the Agent Builder UI - useful if you want to understand each field or tweak things. If you already ran the Dev Tools commands above, you can skip straight to **Step 5 - Chat With Your Agent**.

---

## Step 1 - Create Tool: `get_team_form`

This is the core tool - it returns a team's 2026 results so far: wins, goals scored, goals conceded, and their actual match-by-match results.

Go to **Manage components** (bottom left sidebar) → **Tools** → **New tool**.

| Field | Value |
|---|---|
| **Tool ID** | `get_team_form` |
| **Type** | ES\|QL |
| **Description** | Returns a team's 2026 World Cup results so far - wins, losses, draws, goals scored and conceded, and each match result. Use this whenever the user asks about a team's current form, recent results, or how they've performed in the tournament. |

**Query:**

```esql
FROM wc2026_matches
| WHERE status == "played"
  AND (team1 == ?team_name OR team2 == ?team_name)
| EVAL
    goals_scored   = CASE(team1 == ?team_name, score_ft1, score_ft2),
    goals_conceded = CASE(team1 == ?team_name, score_ft2, score_ft1),
    result         = CASE(
        winner == ?team_name, "win",
        winner == "draw",    "draw",
                              "loss"
    )
| KEEP date, team1, team2, score_ft1, score_ft2, group, stage, result, goals_scored, goals_conceded, winner, stadium
| SORT date ASC
```

**Add one parameter:**

| Name | Type | Description |
|---|---|---|
| `team_name` | keyword | The team name exactly as it appears in the data, e.g. France |

Save the tool.

---

## Step 2 - Create Tool: `get_team_stats_2026`

Aggregates a team's 2026 performance into headline numbers for easy comparison.

| Field | Value |
|---|---|
| **Tool ID** | `get_team_stats_2026` |
| **Type** | ES\|QL |
| **Description** | Returns aggregated 2026 tournament statistics for a team: total matches played, wins, draws, losses, goals scored, goals conceded, and goal difference. Use this when comparing two teams statistically or assessing their overall tournament performance. |

**Query:**

```esql
FROM wc2026_matches
| WHERE status == "played"
  AND (team1 == ?team_name OR team2 == ?team_name)
| EVAL
    goals_scored   = CASE(team1 == ?team_name, score_ft1, score_ft2),
    goals_conceded = CASE(team1 == ?team_name, score_ft2, score_ft1),
    is_win  = CASE(winner == ?team_name, 1, 0),
    is_draw = CASE(winner == "draw", 1, 0),
    is_loss = CASE(winner != ?team_name AND winner != "draw", 1, 0)
| STATS
    matches_played  = COUNT(*),
    wins            = SUM(is_win),
    draws           = SUM(is_draw),
    losses          = SUM(is_loss),
    total_scored    = SUM(goals_scored),
    total_conceded  = SUM(goals_conceded)
```

**Add one parameter:**

| Name | Type | Description |
|---|---|---|
| `team_name` | keyword | The team name, e.g. Germany |

Save the tool.

---

## Step 3 - Create Tool: `get_upcoming_fixtures`

Shows what matches are still to come - essential for identifying who is actually playing whom next.

| Field | Value |
|---|---|
| **Tool ID** | `get_upcoming_fixtures` |
| **Type** | ES\|QL |
| **Description** | Returns upcoming scheduled matches for a team that have not yet been played. Use this when the user asks when a team plays next, who their next opponent is, or to identify a real upcoming matchup to predict. |

**Query:**

```esql
FROM wc2026_matches
| WHERE status == "upcoming"
  AND (team1 == ?team_name OR team2 == ?team_name)
| KEEP date, round, group, team1, team2, stadium
| SORT date ASC
```

**Add one parameter:**

| Name | Type | Description |
|---|---|---|
| `team_name` | keyword | The team name, e.g. England |

Save the tool.

---

## Step 4 - Create the Custom Agent

Go to **Manage components → Agents → New agent**.

### Settings tab

| Field | Value |
|---|---|
| **Agent ID** | `wc2026_predictor` |
| **Display name** | World Cup 2026 Predictor |
| **Display description** | Ask me anything about the 2026 World Cup - form, stats, upcoming matches, and predictions based on real tournament data. |
| **Avatar** | Green or football-themed |

**Custom instructions** - paste this exactly:

```
You are a football analyst specialising in the 2026 FIFA World Cup, currently underway in the USA, Mexico, and Canada.

You have access to live tournament data: completed results, upcoming fixtures, and team form.

When a user asks you to predict a match:
1. Call get_upcoming_fixtures for each team to confirm they are actually scheduled to play
2. Call get_team_stats_2026 for both teams to get their tournament numbers
3. Call get_team_form for both teams to see their actual match-by-match results
4. Produce a prediction structured as:
   - Current form summary for each team (results so far, goals scored/conceded)
   - Statistical edge: which team has the data advantage and why
   - Key factors: 2–3 things that will decide this match
   - Predicted scoreline and winner
   - Confidence level: Low / Medium / High with one sentence of reasoning

Always ground your predictions in the 2026 data - this is a live tournament, not historical analysis.

Keep your tone punchy and engaging. You are talking to developers at a hackathon, not writing a press release.

Only answer questions about the 2026 World Cup. Politely decline anything off-topic.
```

### Tools tab

Assign all three tools:
- `get_team_form`
- `get_team_stats_2026`
- `get_upcoming_fixtures`

> **Tip:** Leave built-in Elastic tools unassigned to keep the agent focused.

### Save

Click **Save and chat**.

---

## Step 5 - Chat With Your Agent

Try these prompts - they're all grounded in live 2026 data:

```
When do France and Argentina play and what's their form so far?
```
```
Compare Brazil and Morocco based on their 2026 tournament results
```
```
Predict the Germany vs Ecuador match
```
```
How has England performed in the group stage?
```

Watch the **thinking trace** as the agent calls tools in sequence before responding - that's the context engineering loop in action.

---

## Hack Extensions

### 🟢 Beginner
- **Change the persona** - make the agent sound like a specific pundit, add drama, or make it aggressively optimistic about the host nations
- **Add a `get_group_standings` tool** - query by group to surface who's top, who needs a result, and who's already through
- **Refresh the data** - re-run the notebook after each matchday and ask the agent questions about the new results

### 🟡 Intermediate
- **Add a squad tool** - ingest `worldcup.squads.json` (same GitHub URL base) as a second index; add a tool that lets the agent factor in squad depth when predicting
- **Log predictions** - create a `wc2026_predictions` index and a write tool; instruct the agent to save every prediction it makes, then query which team it's backed most often
- **Build a head-to-head tool** - cross-reference the `wc2026_matches` index with the historical `world_cup_matches` index (from the previous version) to add all-time World Cup history to predictions

### 🔴 Advanced
- **Live polling** - swap the daily openfootball fetch for `worldcup26.ir` (no key for basic endpoints) and poll every few minutes during live matches; add a `get_live_score` tool
- **Expose via MCP** - use Agent Builder's built-in MCP server to connect the predictor to Claude Desktop or a custom app
- **Vector search on match narratives** - generate text descriptions of each match ("Brazil drew 1-1 with Morocco in a tight Group C encounter") and store embeddings; add a semantic similarity tool so the agent can find comparable historical matches

---

## Reference

- [Agent Builder docs](https://www.elastic.co/docs/explore-analyze/ai-features/elastic-agent-builder)
- [Custom tools](https://www.elastic.co/docs/explore-analyze/ai-features/agent-builder/tools/custom-tools)
- [Custom agents](https://www.elastic.co/docs/explore-analyze/ai-features/agent-builder/custom-agents)
- [ES|QL reference](https://www.elastic.co/docs/explore-analyze/query-filter/languages/esql-kibana)
