# Distributed Semantic Search Architecture

A search engine built from first principles in Python. The system crawls articles from the web, builds an inverted index over them, ranks results by TF-IDF relevance, and serves queries through a desktop interface.

Every core data structure is implemented by hand rather than taken from a library, including the trie, binary search tree, graph, topic tree, stack, queue, and Levenshtein distance for fuzzy matching.

---

## Pipeline

```
Crawler  ->  Indexer  ->  Query Processor  ->  TF-IDF Ranker  ->  Interface
```

| Stage | Module | What it does |
|---|---|---|
| Crawl | `crawler3.py` | Fetches and parses article pages, writes normalized records to `articles.json` |
| Index | `indexer.py` | Builds the inverted index mapping terms to article IDs, plus per article term counts and a topic tree |
| Process query | `query_processor.py` | Tokenizes the query, removes stop words using a trie, emits clean terms |
| Rank | `tfidf.py` | Scores candidate articles by term frequency and inverse document frequency |
| Serve | `search_gui.py`, `main.py` | Desktop interface with search history and article navigation |

---

## How it works

**Indexing.** The indexer loads crawled articles and builds several structures over them at once: an inverted index from term to the set of articles containing it, per article term frequency counters, a binary search tree for article lookup by identifier, an undirected graph linking related articles, and a topic tree grouping articles by subject. This lets the query path answer term lookups, related article traversal, and topic browsing without rescanning the corpus.

**Query processing.** Incoming queries are tokenized to lowercase alphabetic terms, then filtered against a stop word trie. A trie is used rather than a set because it gives prefix aware lookup that the fuzzy matching path also relies on. Remaining terms are passed to the ranker as a queue.

**Ranking.** TF-IDF is computed over the inverted index. Term frequency comes from the per article counters built at index time, and inverse document frequency from the number of articles containing each term. Articles are scored against all query terms and returned in descending order.

**Fuzzy matching.** Levenshtein distance in `data_structures.py` handles misspelled queries by finding near matches among indexed terms.

---

## Repository layout

```
crawler3.py           Web crawler
indexer.py            Inverted index construction and corpus structures
query_processor.py    Tokenization and trie based stop word removal
tfidf.py              TF-IDF relevance scoring
data_structures.py    Trie, BST, Graph, TopicTree, Stack, Queue, Levenshtein
search_gui.py         Desktop search interface
navigation.py         Result navigation
history_manager.py    Search history persistence
article_viewers.py    Article display
web_searcher.py       Live web search fallback
main.py               Entry point
articles.json         Crawled corpus
debug_pipeline.py     Pipeline inspection utility
test_fuzzy.py         Fuzzy matching tests
```

---

## Running it

Requires Python 3.10 or newer.

```bash
git clone https://github.com/shahmeerabdul/dss-architecture.git
cd dss-architecture
python -m venv .venv
source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python main.py
```

To rebuild the corpus from scratch before searching:

```bash
python crawler3.py
```

Running the tests:

```bash
pytest test_fuzzy.py
```

---

## Built with

Python, BeautifulSoup and Requests for crawling, CustomTkinter for the interface, pandas for tabular handling, pytest for tests. No search or indexing library is used; the retrieval stack is written from scratch.

---

## Team

Four person project, started December 2025.

| Contributor | Responsibility |
|---|---|
| Shahmeer Abdullah | Search engine and system integration: query processor, TF-IDF ranking, wiring the pipeline stages together |
| Khuzaim | Search engine and system integration |
| Ahsan | Indexer and interface |
| Abeer | Crawler and interface |

---

## Status and known limitations

- The crawler works from a fixed seed configuration and would need reworking to run against arbitrary sources.
- Ranking is TF-IDF only. Semantic ranking through embeddings is the natural next step.
- The corpus is held in memory, which caps the practical index size.
- The interface is a desktop application. A web frontend was scoped but not built.

---

## License

See `LICENSE`.

