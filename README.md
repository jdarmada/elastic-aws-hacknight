# Elastic and AWS Hack Night (World Cup Edition)

Welcome to the Elastic and AWS hack night! Tonight you'll be building a project that uses Elasticsearch and public soccer data. 

Projects will be judged based on:
- Use of Elasticsearch features
- Creativity
- Usefulness

Top 3 projects will win Meta Ray-Ban glasses.

You have the choice on where to start:
1. Follow the starter project and add complexity.
2. A completely new project that uses Elasticsearch and soccer data in some capacity.

Either direction you follow you must use a serverless Elastic deployment:
Sign-up for a free trial here: [Elastic Serverless free-trial](https://www.elastic.co/cloud/cloud-trial-overview)


### What you'll build - Hack Night Starter Project

Predict 2026 World Cup matches using live data from the tournament happening right now. You'll build a **World Cup Predictor agent** that you can ask things like:

> *"Predict the upcoming Brazil vs Scotland match"*
> *"Compare France and Argentina based on their 2026 results so far"*
> *"Who are the top scorers in the tournament?"*
> *"When does England play next and what's their form?"*

The agent pulls live stats from Elasticsearch, reasons over them, and gives you a structured prediction with scoreline, key factors, and confidence level. The project has two parts that build on each other:

- **Part 1 - Notebook:** fetch live 2026 match data from a public API and ingest it into Elasticsearch
- **Part 2 - Agent Builder:** build a conversational AI agent on top of that data inside Kibana — no extra code, LLM included

Run the notebook first to populate the index, then move to Agent Builder.

---

## Teams in the Dataset

All 48 qualified nations from the 2026 World Cup - including `Brazil`, `Germany`, `France`, `Argentina`, `Spain`, `England`, `USA`, `Mexico`, `Canada`, `Morocco`, `Japan`, `Portugal`, and more.

The index includes both completed results and upcoming fixtures, so the agent can answer questions about form, schedules, and predictions.

---

## Prerequisites

- **Elastic Serverless** project (sign up at [elastic.co](https://www.elastic.co))

---

## Part 1 - Notebook

`world_cup_predictor.ipynb`

Run this first - it fetches live 2026 data and ingests it. All queries and analysis happen in Agent Builder (Part 2). The notebook's only job is to populate the index.

### Option A - Google Colab (recommended, no install needed)

The fastest way to get started. Just a browser - no Python install required.

1. Go to [colab.research.google.com](https://colab.research.google.com)
2. Click **File → Upload notebook** and upload `world_cup_predictor.ipynb`
3. Fill in your credentials in the Section 1 cell (see below)
4. Click **Runtime → Run all** to run every cell top to bottom

That's it. Colab handles all the dependencies automatically when the first cell runs `pip install`.

---

### Option B - Run locally

If you'd prefer to run on your own machine:

**Requirements:** Python 3.9+ with pip installed. Check with `python --version` in a terminal.

```bash
# Install Jupyter and dependencies
pip install jupyter elasticsearch requests

# Launch Jupyter and open the notebook
jupyter notebook world_cup_predictor.ipynb
```

This opens a browser tab. Run cells one at a time with **Shift + Enter**, or run all at once via **Cell → Run All**.

---

### Filling in your credentials

Whichever option you use, fill in these two values in the Section 1 cell before running:

```python
ELASTIC_ENDPOINT = "https://your-project.es.region.aws.elastic.cloud"
ELASTIC_API_KEY  = "your-elastic-api-key"
```

> **Finding your credentials:** Elastic Cloud Console → Your Project → Connection Details

### What the notebook does

**Section 1 - Connect** to Elastic Serverless.

**Section 2 - Fetch live data** from [openfootball/worldcup.json](https://github.com/openfootball/worldcup.json) - a public domain GitHub repo updated daily with real 2026 results. No API key required. Both completed results and upcoming fixtures are fetched and printed so you can see what's in the data.

**Section 3 - Create index** `wc2026_matches` with explicit mappings, including a nested `goals` field capturing scorer name, minute, team, and goal type.

**Section 4 - Enrich and ingest** all matches. Computed fields added at ingest: `status` (played/upcoming), `winner`, `total_goals`, `team1_win`, `team2_win`, `stage`.

**Section 5 - Verify** with quick sense-check queries confirming the data landed correctly. This is the last step in the notebook - building the agent happens in Part 2.

**To refresh with latest results:** re-run cells 2-4. The index is dropped and recreated each time so there are no duplicates. Takes about 30 seconds.

### Index schema - `wc2026_matches`

| Field | Type | Notes |
|---|---|---|
| `date` | date | Match date |
| `round` | keyword | e.g. `Matchday 1`, `Quarter-finals` |
| `group` | keyword | e.g. `Group A`, `Knockout` |
| `stage` | keyword | `group`, `round_of_32`, `round_of_16`, `quarter`, `semi`, `final` |
| `status` | keyword | `played` or `upcoming` |
| `team1` | keyword | |
| `team2` | keyword | |
| `score_ft1` | integer | Team 1 full-time goals (played only) |
| `score_ft2` | integer | Team 2 full-time goals (played only) |
| `total_goals` | integer | Computed at ingest |
| `winner` | keyword | Team name or `draw` (played only) |
| `team1_win` | boolean | |
| `team2_win` | boolean | |
| `goals` | nested | `scorer`, `minute`, `team`, `type` (`goal`/`penalty`/`own_goal`) |
| `stadium` | keyword | Host city/venue |

---

## Part 2 - Agent Builder

Everything for Part 2 lives in **`agent_builder_guide.md`**. It gives you two paths to the same result:

- **Fast path - Dev Tools Console:** the guide's *Quick Setup* section has the five API commands (four tools + one agent) ready to paste into Kibana's **Dev Tools Console** (hamburger menu → Management → Dev Tools). Paste each block, press play, and you're done - no Kibana endpoint to find, no auth headers, Dev Tools uses your current session.
- **Manual walkthrough:** the rest of the guide builds the same tools and agent field-by-field through the Agent Builder UI, if you want to understand each piece or tweak it.

Either way, once the four tools and the agent exist, open **Kibana → Agents** and the **World Cup 2026 Predictor** is ready to chat.

### Navigate to Agent Builder

In your Elastic Serverless project, open **Kibana** from the project dashboard.

Look for **Agents** in the main navigation. If it's not visible:

> **Management → Advanced Settings** → search for Agent Builder → enable it.

---

### The Four Tools

| Tool ID | What it does |
|---|---|
| `get_team_form` | Full match-by-match results for a team in 2026 - scores, opponents, stage, outcome |
| `get_team_stats_2026` | Aggregated stats: wins, draws, losses, goals scored/conceded |
| `get_upcoming_fixtures` | Scheduled matches not yet played - lets the agent find real upcoming matchups |
| `get_top_scorers` | Tournament goal scorers ranked by total goals |

See `agent_builder_guide.md` for the exact ES|QL query and parameter config for each tool.

---

### The Custom Agent

| Field | Value |
|---|---|
| **Agent ID** | `wc2026_predictor` |
| **Display name** | World Cup 2026 Predictor |
| **Tools** | All four above |

**Custom instructions summary:** The agent is instructed to always check upcoming fixtures first to confirm a match is actually scheduled, then pull team stats and form, then produce a structured prediction with scoreline, key factors, and confidence level. Full instructions in `agent_builder_guide.md`.

---

### Step 5 - Chat With Your Agent

Try these - all grounded in live 2026 data:

```
When do France and Argentina play and what's their form so far?
```
```
Compare Brazil and Morocco based on their 2026 results
```
```
Who are the top scorers in the tournament?
```
```
Predict the Germany vs Ecuador match
```
```
How has England performed in the group stage?
```

Watch the **thinking trace** - you'll see the agent calling tools in sequence before forming its answer.

---