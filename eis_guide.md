# 🤖 Using the Elastic Inference Service (EIS)

The **Elastic Inference Service (EIS)** lets you use AI models - LLMs for chat and embedding models for semantic and vector search - **without deploying a model, managing infrastructure, or bringing your own API keys**. The models are hosted by Elastic on **AWS Bedrock** (and other backends) and are billed per token as part of your Serverless project.

For this hack night, **EIS is the recommended way to add AI to your project** - it's the fastest path and requires zero setup.

---

## Why use EIS tonight

- **No API keys to manage** - you don't need your own AWS Bedrock, OpenAI, or Anthropic account
- **No model deployment** - embedding models are ready the moment your project spins up
- **Already powering the starter project** - the World Cup Predictor agent's LLM runs on EIS
- **Bedrock under the hood** - Elastic Managed LLMs run on AWS Bedrock, so "use EIS" *is* "use Bedrock", minus the setup

---

## 1. LLM / chat - powers Agent Builder (zero setup)

On Elastic Cloud Serverless, **Agent Builder, the AI Assistant, and Search Playground all use Elastic Managed LLMs running on EIS by default**. There is nothing to configure - the LLM is already wired up.

- The **World Cup Predictor agent** you build in [starter_project.md](starter_project.md) already runs on this. You never add a connector or a key.
- To **see or switch the model**, use the model selector in the Agent Builder chat interface.
- To **set a default model** (Elastic 9.4+), search for **Model management** in Kibana's global search bar and pick from the **Default model** dropdown.


---

## 2. Semantic & vector search (zero setup)

Perfect for the **Soccer RAG Chatbot**, **semantic search**, and **Similar Player Finder** ideas in the [open challenge](open_challenge.md).

The simplest path is a `semantic_text` field. You don't even need to choose a model - on a current Serverless project, `semantic_text` automatically uses the deployment's default embedding model (a Jina embedding model served by EIS):

```
PUT wc_notes
{
  "mappings": {
    "properties": {
      "summary": { "type": "semantic_text" }
    }
  }
}
```

Index documents normally - Elasticsearch calls EIS to embed them at index time:

```
POST wc_notes/_doc
{
  "summary": "Brazil edged Morocco 1-0 in a tense Group C clash decided by a late header."
}
```

Then search with a natural-language `semantic` query - no keywords, no key, no model deployment:

```
GET wc_notes/_search
{
  "query": {
    "semantic": {
      "field": "summary",
      "query": "which teams won close low-scoring games"
    }
  }
}
```

**Want a specific model, or raw vectors for your own kNN search?** Create a `text_embedding` endpoint backed by EIS (`service: "elastic"`):

```
PUT _inference/text_embedding/wc-embeddings
{
  "service": "elastic",
  "service_settings": {
    "model_id": "<embedding-model-id>"
  }
}
```

> Check the [EIS docs](https://www.elastic.co/docs/explore-analyze/elastic-inference/eis) for the current list of available embedding model IDs.

Then point a `semantic_text` field at it with `"inference_id": "wc-embeddings"`, or generate vectors to store in a `dense_vector` field for [kNN search](https://www.elastic.co/docs/solutions/search/vector/knn).

---

## 3. Calling EIS from Python / the API

The inference API works from the same Python client the notebook uses. For example, to generate embeddings for a batch of text:

```python
resp = es.inference.inference(
    inference_id='wc-embeddings',
    task_type='text_embedding',
    input=['Lionel Messi', 'Kylian Mbappe', 'Erling Haaland'],
)
print(resp['text_embedding'][0]['embedding'][:5])
```

For chat/LLM responses, the easiest path is Agent Builder or Search Playground (both already on EIS). To drive chat completion programmatically, see the [chat completion API](https://www.elastic.co/docs/api/doc/elasticsearch-serverless/operation/operation-inference-chat-completion-unified).

---

## Reference

- [Elastic Inference Service (EIS)](https://www.elastic.co/docs/explore-analyze/elastic-inference/eis)
- [Default inference endpoints](https://www.elastic.co/docs/explore-analyze/elastic-inference/inference-api)
- [Model configuration in Agent Builder](https://www.elastic.co/docs/explore-analyze/ai-features/agent-builder/models)
- [Semantic search with `semantic_text`](https://www.elastic.co/docs/solutions/search/semantic-search/semantic-search-semantic-text)
- [Create an inference endpoint (API)](https://www.elastic.co/docs/api/doc/elasticsearch-serverless/operation/operation-inference-put)
- [Build AI agents with EIS (Elasticsearch Labs)](https://www.elastic.co/search-labs/blog/build-ai-agents-elastic-inference-service)
