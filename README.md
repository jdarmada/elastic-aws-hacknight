# Elastic and AWS Hack Night (World Cup Edition)

Welcome to the **Elastic and AWS Hack Night**! Tonight you'll build a project using **Elasticsearch** and **public soccer datasets** to create search experiences, AI applications, or anything else you can imagine.

Whether you're exploring semantic search, building a RAG chatbot, or experimenting with vector search, this is your chance to showcase what's possible with Elasticsearch.

## Judging Criteria and Presentations

Projects will be evaluated on the following:

| Criteria | Description |
|----------|-------------|
| **Use of Elasticsearch** | Demonstrates meaningful use of Elasticsearch features such as search, aggregations, vector search, and Agent Builder. |
| **Use of AWS Bedrock** | Use of a managed LLM through AWS Bedrock. If you're using the built-in chat within Agent Builder, it already uses Bedrock. |
| **Creativity** | Presents a unique idea, novel user experience, or interesting technical implementation. |
| **Usefulness** | Solves a real problem or provides valuable insights from the data. |

At the end, you'll have the chance to present what you built, no matter how complete your project is. Don't be shy! It's in the spirit of the event to show off your ideas even if it's not done. 

Some presentation guidelines:
- **2 mins max**
- Quickly mention what the project does, but more importantly, show the Elasticsearch portion from the queries you used, the custom tools and agents you built within Agent Builder.

## Prizes

The **top three projects** will each win a pair of **Meta Ray-Ban Smart Glasses**.

Good luck, have fun, and happy hacking!

## What you can build

You have the choice on where to start:
1. Head to [starter_project.md](https://github.com/jdarmada/elastic-aws-hacknight/blob/main/starter_project.md) and follow the steps to build the World Cup Predictor agent. Extend this project by adding more data, queries, features, nuance etc.
2. A completely new project that uses Elasticsearch and soccer data in some capacity. Head over to [open_challenge.md](https://github.com/jdarmada/elastic-aws-hacknight/blob/main/open_challenge.md) for examples of what you can build and an example of how to ingest data.

Either direction you follow you must use a serverless Elastic deployment: [Elastic Cloud Serverless free-trial](https://www.elastic.co/cloud/cloud-trial-overview)


## Public Soccer Datasets

These are publicly available soccer datasets for player performance analysis, match data, and event-level analytics. You are not restricted to these, feel free to use any dataset you find.

| Dataset | Description | Data Type |
|---------|-------------|-----------|
| **[FIFA World Cup 2026 Player Performance Dataset](https://www.kaggle.com/datasets/rauffauzanrambe/fifa-world-cup-2026-player-performance-dataset)** | Simulated/player performance dataset for the FIFA World Cup 2026. Includes player statistics, match performance metrics, team information, and tournament-related data suitable for machine learning and analytics. | Player & Match Statistics |
| **[openfootball/worldcup.json](https://github.com/openfootball/worldcup.json)** | Open-source JSON dataset containing historical FIFA World Cup tournaments, including teams, fixtures, match results, venues, and tournament structure in an easy-to-use format. | Historical Match Results |
| **[StatsBomb Open Data](https://github.com/statsbomb/open-data)** | One of the most comprehensive free football analytics datasets available. Provides detailed event-level data (passes, shots, dribbles, pressures, tackles, etc.), lineups, matches, competitions, and 360° data for selected competitions. Widely used in football analytics research and visualization. :contentReference[oaicite:0]{index=0} | Event-Level Match Data |


## Resources

Handy documentation and references for building tonight.

### Getting started
- [Elasticsearch quickstart](https://www.elastic.co/docs/solutions/search/get-started) - your first index and query
- [Connecting to Elasticsearch](https://www.elastic.co/docs/reference/elasticsearch/clients) - endpoints, API keys, and client setup

### Agent Builder
- [Agent Builder overview](https://www.elastic.co/docs/explore-analyze/ai-features/elastic-agent-builder)
- [Building custom tools](https://www.elastic.co/docs/explore-analyze/ai-features/agent-builder/tools/custom-tools)
- [Building custom agents](https://www.elastic.co/docs/explore-analyze/ai-features/agent-builder/custom-agents)
- [Expose agents over MCP](https://www.elastic.co/docs/explore-analyze/ai-features/agent-builder/mcp-server) - connect to Claude Desktop or your own app

### Search & querying
- [ES|QL reference](https://www.elastic.co/docs/explore-analyze/query-filter/languages/esql) - the query language the starter tools use
- [Query DSL](https://www.elastic.co/docs/explore-analyze/query-filter/languages/querydsl) - full-text, filters, and boolean queries
- [Aggregations](https://www.elastic.co/docs/explore-analyze/query-filter/aggregations) - stats, terms, and metrics for dashboards
- [Search relevance & autocomplete](https://www.elastic.co/docs/solutions/search/full-text) - for player/team search experiences

### Vector & semantic search (great for RAG and "similar player" ideas)
- [Semantic search with `semantic_text`](https://www.elastic.co/docs/solutions/search/semantic-search/semantic-search-semantic-text) - the fastest path to semantic search
- [kNN / dense vector search](https://www.elastic.co/docs/solutions/search/vector/knn)
- [Bringing your own embeddings](https://www.elastic.co/docs/reference/elasticsearch/mapping-reference/dense-vector)

### Ingesting data
- [Python Elasticsearch client](https://www.elastic.co/docs/reference/elasticsearch/clients/python) - what the notebook uses
- [Bulk API](https://www.elastic.co/docs/api/doc/elasticsearch/operation/operation-bulk) - efficient batch indexing
- [Upload a file in Kibana](https://www.elastic.co/docs/manage-data/ingest/upload-data-files) - no-code CSV/JSON ingest

### AWS Bedrock
- [Amazon Bedrock documentation](https://docs.aws.amazon.com/bedrock/)



