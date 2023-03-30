# GPT Cache

English | [中文](README-CN.md)

The GPT Cache system is mainly used to cache the question-answer data of users in ChatGPT. This system brings two benefits:

1. Quick response to user requests: compared to large model inference, searching for data in the caching system will have lower latency, enabling faster response to user requests.
2. Reduced service costs: currently, most ChatGPT services are charged based on the number of requests. If user requests hit the cache, it can reduce the number of requests and thus lower service costs.

If the idea 💡 is helpful to you, please feel free to give me a star 🌟, which is helpful to me.

## 🤔 Is Cache necessary?

I believe it is necessary for the following reasons:

- Many question-answer pairs in certain domain services based on ChatGPT have a certain similarity.
- For a user, there is a certain regularity in the series of questions raised using ChatGPT, which is related to their occupation, lifestyle, personality, etc. For example, the likelihood of a programmer using ChatGPT services is largely related to their work.
- If your ChatGPT service targets a large user group, categorizing them can increase the probability of relevant questions being cached, thus reducing service costs.

## 😊 Quickly Start

### alpha test package install

**Note**: You can quickly experience the cache, it is worth noting that maybe this is not very **stable**.

```bash
pip install -i https://test.pypi.org/simple/ gpt-cache==0.0.1
```

If you just want to achieve precise matching cache of requests, that is, two identical requests, you **ONLY** need **TWO** steps to access this cache

1. Cache init

```python
from gpt_cache.core import cache

cache.init()
# If you use the `openai.api_key = xxx` to set the api key, you need use `cache.set_openai_key()` to replace it
cache.set_openai_key()
```
2. Replace the original openai package

```python
from gpt_cache.view import openai

# openai requests DON'T need ANY changes
answer = openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "foo"}
    ],
)
```

Run locally, if you want better results, you can use the example [Sqlite + Faiss + Towhee](example/sf_towhee/sf_manager.py). Among them, Sqlite + Faiss is used for cache data management, and Towhee is used for embedding operations.

In actual production, or in a certain user group, it is necessary to consider the vector search part more, you can get to know [Milvus](https://github.com/milvus-io/milvus)，or [Zilliz Cloud](https://cloud.zilliz.com/), which allows you to quickly experience Milvus vector retrieval.

More Docs：
- [examples](example/example.md)
- [system design](doc/system.md)


## 🥳 Feature

- Support the openai chat completion normal and stream request
- Get top_k similar search results, it can be set when creating the data manager
- Support the cache chain, see: `Cache#next_cache`

```python
bak_cache = Cache()
bak_cache.init()
cache.init(next_cache=bak_cache)
```

- Whether to completely skip the current cache, that is, do not search the cache or save the Chat GPT results, see: `Cache#cache_enable_func`
- In the cache initialization phase, no cache search is performed, but the result returned by the cache chat gpt, see: `cache_skip=True` in `create` request

```python
openai.ChatCompletion.create(
    model="gpt-3.5-turbo",
    messages=mock_messages,
    cache_skip=True,
)
```

- Like Lego bricks, custom assemble all modules, including:
  - pre-embedding function, get feature information in the original request, such as prompt, last message, etc.
  - embedding function, convert feature information into a vector for cache search, choose a model that fits your use case
  - data manager, cache data management, mainly dealing with the search and storage of cache data
  - cache similarity evaluation function, can use the distance of similar search or additional selection model to ensure that the answer is more accurate
  - post-process the cache answer list, first, random or custom combination

## 🤗 All Model

- Pre-embedding
  - get the last message in the request, see: `pre_embedding.py#last_content`
- Embedding
  - [towhee](https://towhee.io/), english model: paraphrase-albert-small-v2, chinese model: uer/albert-base-chinese-cluecorpussmall
  - openai embedding api
  - string, nothing change
- Data Manager
  - scalar store
    - [sqlite](https://sqlite.org/docs.html)
  - vector store
    - [milvus](https://milvus.io/)
  - vector index
    - [faiss](https://faiss.ai/)
- Similarity Evaluation
  - the search distance, see: `simple.py#pair_evaluation`
  - [towhee](https://towhee.io/), roberta_duplicate, precise comparison of problems to problems mode, only support the 512 token
  - string, the cache request and the original request are judged by the exact match of characters
  - np, use the `linalg.norm`
- Post Process
  - choose the most similar
  - choose randomly


## 😆 Contributing

Want to help build GPT Cache? Check out our [contributing documentation](doc/contributing.md).


## 🙏 Thank

Thanks to my colleagues in the company [Zilliz](https://zilliz.com/) for their inspiration and technical support.