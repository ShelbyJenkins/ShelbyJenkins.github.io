---
title: 'Retrieval is all you need (Part 1): A brief intro to text retrieval in RAG'
# Optional
description: 'Cataloging the SotA technologies available and permutations of stacks for retrieval pipelines in Gen AI. Part 1 is a brief intro to RAG, and an overview of retrieval techniques and technologies.'

# Published date is required and in the format of ISO-8601: `yyyy-mm-dd`. For more info see https://docs.astro.build/en/guides/content-collections/#working-with-dates-in-the-frontmatter
pubDate: 2024-03-17
# Optionally specify an update date. If not provided, one will be generated from the git history. Only if the post has been changed since the day published.
# updatedDate:
heroImage: './hero.webp'
# Optional
---

This is a two part blog series. [Part 2](/blog/retrieval-is-all-you-need-2) will be continuously updated.

- [Intro](#retrieval-augmented-generation-and-llms-as-arbitrage-engines)
- [Retrieval Technology](#retrieval-technology)
  - [Keyword/Character Pattern Matching](#keywordcharacter-pattern-matching)
  - [Full-Text Search (FTS)](#full-text-search-fts)
  - [Semantic search](#semantic-search)
  - [LLMs for retrieval](#using-llms-for-retrieval)
- [Spreadsheet of open-source libraries](#spreadsheet-of-open-source-libraries)

<details open>

<summary>

# Retrieval augmented generation and LLMs as arbitrage engines

</summary>

<blockquote class="reddit-embed-bq" data-embed-height="2"  ><a href="https://www.reddit.com/r/programming/comments/10x38hn/comment/j7rhkn7/">Comment</a><br> by<a href="https://www.reddit.com/user/2bias_4ever/">u/2bias_4ever</a> from discussion<a href="https://www.reddit.com/r/programming/comments/10x38hn/microsoft_announces_new_bing_chatgpt_within_a/"><no value=""></no></a><br> in<a href="https://www.reddit.com/r/programming/">programming</a></blockquote><script async="" src="https://embed.reddit.com/widgets.js" charset="UTF-8"></script>

I'm at risk of anthropomorphizing a piece of software when I say, "An LLM is making a series of decisions on how to respond to user input." Sure, it isn't _actually_ deciding anything, and is really just predicting the next most likely token, but for the sake of this conversation lets agree it's a distinction without difference; An LLM takes training input and input context, and returns useful outputs. That it can reliably _choose_ the most useful information to generate is _the reason LLMs are useful at all_.

In some ways, LLMs are no different than traditional retrieval methods like search engines or a SQL queries. Sure, LLMs can retrieve the information you need AND create a bespoke response on how to use that information to perform the task at hand, but really you just needed to know how to do XYZ mundane thing and you didn't feel like fighting your way to the documentation page and then actually reading the friendly manual.

It's the same when using an LLM in business logic. In a business workflow with thousands of potential input variants you could [just write a switch statement with a thousand cases](https://github.com/fachinformatiker/undertale/blob/master/scripts/SCR_TEXT.gml) like the creator of the beloved indie game Undertale famously did. In the end, the difference between an LLM and a billion manually written if statements is just a matter of will. Now with LLMs, we don't have to handle every possible input, and can instead ask the LLM to choose the correct action in a workflow. That is, based on this input, the LLM _retrieves_ the correct action to perform. It's retrieval all the way down.

![Always has been informational arbitrage.](./retrieval-meme.jpg)(class:'medium')

LLMs infamously sound confident, even when confidently incorrect. So how can you be confident in using an LLM when it's training data doesn't encompass the state of every particle in the universe at this exact current instance? Available commercial models are generalist trained on limited public data from, perpetually it seems, two years ago! A retrieval layer to add additional context solves this problem by ensuring the LLM has the correct and up to date context it needs to be confidently correct. This is known as 'retrieval augmented generation' (RAG) or 'context enhanced querying' (CEQ).

The savvy may recognize that retrieval is a core competency for many, probably most, projects and businesses - not just those with an `.ai` TLD. Search tech like semantic search has been evolving for some time, but has recently been given a hype injection for it's use with generative AI. But better search isn't just 'picks and shovels' sold to the AI goldrush out of the VC's general stores; being able to sift and pan the correct data out of endless streams of data creates value. The old metaphor of 'a needle in a haystack' can be turned-of-a-phrase to 'nuggets of gold in the river'. This post will catalog available SotA tools for retrieval, with a focus for use with LLMs.

</details>
<hr>

# Retrieval Technology

## Keyword/Character Pattern Matching

Keyword/character pattern matching is the oldest and simplest form of text search. It essentially involves scanning text for specific keywords, much like using the "ctrl+f" function in a web browser to find words on a page. This method is included here to contrast with more advanced techniques.

However, keyword matching still has some relevance in modern retrieval. This is particularly true when querying for sequences of characters that don't exist in the natural language(s) that advanced search techniques are optimized for. A common example is searching for an uncommon and verbose error message. In the past, Google was more reliant on keyword search and never returned empty results. However, Google's modern search implementation may now return empty results in some cases, even if the exact error message exists in a project's public repository. So there is a still a need for consideration of Keyword search here.

<hr>

## Full-Text Search (FTS)

Full-Text Search (FTS) is a more advanced approach to text retrieval. It's widely used as it offers significant advantages over simple keyword matching, particularly in terms of search performance, scalability, and the ability to handle large amounts of text data effectively.

<h3>"Backend Features" and Optimizations (Not Visible to Users)</h3>

Scalability and efficiency:

- FTS is generally faster than simple keyword search, especially for large datasets, due to magic like inverted indexes, preprocessing optimizations, scalability techniques, and advanced query optimization.
- Scalability and Distributed Processing: FTS engines employ techniques like sharding and distributed processing to handle large volumes of text data efficiently and maintain fast search performance.
- Caching and Query Optimization: FTS engines incorporate caching mechanisms and advanced query optimization techniques to improve the efficiency and speed of search queries.

Improved accuracy and relevancy:

- FTS engines utilize TF-IDF (Term Frequency-Inverse Document Frequency) weighting to assign higher weights to more important terms within documents and across the corpus, enhancing search result relevance.
- Stop word removal removes common stop words (e.g., "the," "and," "is") from search queries to improve search efficiency and focus on more meaningful terms.
- Tokenization breaks down the text into individual words or tokens, which forms the basis for further processing and indexing.
- Linguistic processing techniques, such as stemming and lemmatization, are applied to the tokens to reduce words to their base or dictionary form. This helps match variations of the same word and improves search recall.
- Synonym expansion, as part of linguistic processing, allows matching of related words or phrases, further enhancing search results.
- Handling special characters and punctuation is also a part of tokenization and linguistic processing, ensuring that the search engine can effectively deal with various types of text data.

<h3>"Frontend Features" (User-Facing)</h3>

- Boolean Operators: FTS allows users to combine search terms using Boolean operators like AND, OR, and NOT to create more precise and complex queries.
- Phrase Search: Users can search for exact phrases by enclosing search terms in quotes, ensuring that the FTS engine looks for the specific sequence of words. (This a form of keyword search that Google infamously no longer respects!)
- Wildcard searches (e.g., "appl\*" to match "apple" or "application")
- Proximity searches (e.g., using the asterisk (\*) symbol to represent one or more words that can appear between the specified search terms, such as "apple \* cider" to find pages where these words are near each other)
- Spellings Correction and Suggestions: FTS engines often incorporate spelling correction and suggestion features to handle misspelled search queries and provide alternative search terms.
- Faceted search allows users to explore and refine search results by selecting multiple predefined categories (facets) associated with the data, providing a dynamic and interactive way to narrow down the result set.
- Filtering is a general term for narrowing down search results based on specific criteria, such as keywords, date ranges, or attributes, allowing users to restrict the results to a subset that matches the selected criteria. (This is also an important concept for semantic search.)

<h3>Full-Text Search (FTS) Open-source Libraries</h3>

Established libraries:

- Apache Solr (or Apache Lucene)
- Elasticsearch (or OpenSearch)
- Manticore

Upcoming libraries:

- Meilisearch
- Quickwit (or Tantivy)
- Typesense

<hr>

## Semantic search

Semantic search focuses on understanding the intent and contextual meaning behind a user's query. It considers the semantic relationships between words and concepts. A simple example is searching for "fish recipes" using semantic search; this will return documents with cooking instructions for not just fish but also specific types of fish like halibut, whereas keyword or full-text search would only return documents containing the exact word "fish."

Concepts:

- The two most common types of algorithms used in semantic search are KNN and ANN. They both find the most similar or relevant items to a given query, but ANN is the more common of the two.
  - KNN: exact k-nearest neighbor search works by measuring the similarity or distance between the query and the items in a dataset. It then retrieves the top K nearest neighbors (most similar items) based on the calculated similarity scores. KNN is a simple yet effective approach for finding semantically related items, but it can be computationally expensive for large datasets.
  - ANN: approximate k-nearest neighbor search efficiently finds the _approximate_ nearest neighbors in high-dimensional spaces. Unlike KNN, which performs an exhaustive search to find the exact nearest neighbors, ANN algorithms aim to find approximate neighbors quickly by trading off some accuracy for speed. ANN is particularly useful when dealing with large-scale datasets or real-time search requirements.
- Scoring: search results receive relevance scores based on their proximity in the vector space. This can be useful for ensuring relevance of returned documentation.

Embeddings:

- Semantic search embeddings, which are dense vector representations that capture the semantic meaning and relationships of the text.
- Embedding Dimensionality: Dimensionality refers to the count of vectors in an embedding and is an important metric in semantic search.
  - Different semantic search engines and vector databases have varying limits and performance characteristics based on embedding dimensionality.
  - Higher dimensionality allows for more information to be conveyed by the embedding, but there is often a sweet spot between size and performance.
  - Some content may perform better with smaller dimensionality, while others may benefit from higher dimensionality.
  - The higher the dimensionality of embeddings, the higher the computational cost of querying. Therefore, embedding dimensionality should be carefully considered to balance information richness and query performance.
- Embedding Generation: Embeddings need to be generated for both the content and the queries in semantic search. Embeddings can be created on the device using open-source libraries and models or through paid APIs.

Features of vectorDBs and libraries:

- Larger-than-memory datasets: Working with datasets larger than the device's memory is common in semantic search. Some semantic search algorithms are designed to work only in-memory, which limits their scalability. However, most modern semantic search systems handle indexing on disk using techniques like Inverted File (IVF) and diskANN to overcome memory limitations. Additionally, vector databases, which are specialized databases optimized for storing and searching high-dimensional vectors, solve this problem as a core part of their functionality.
- Filters: Filtering is crucial in semantic search, as it allows users to narrow down and refine search results based on specific criteria, similar to SQL modifiers. Vector databases often provide robust filtering capabilities, enabling users to combine semantic similarity search with traditional filtering options. However, filtering support may be more limited in standalone semantic search libraries, requiring additional implementation efforts to achieve the desired filtering functionality.
- Hybrid search: hybrid search can mean different things for different tools, but it typically refers to some combination of semantic search techniques with traditional full-text search methods. This is a valuable feature and worth researching the implementation of before choosing a semantic search tool.

<h3>Semantic Search Open-source Libraries</h3>

Semantic search libraries:

- Arroy
- Cuvs
- Usearch
- Voyager

Vector databases:

- Chroma
- LanceDB
- Milvus
- Qdrant
- Weaviate

Traditional databases with semantic search extensions:

- Postgres: pgvector
- Sqlite: sqlite-vss

<hr>

## Using LLMs for retrieval

I've used LLMs in retrieval in two ways:

- Query expansion for semantic search. Beyond just generating synonyms and related words to a query, you can also ask the LLM to describe what they think a response _might_ look like, and then use that response to perform the search. The goal is to take what may be a short query from a user and then expand the queries context with inputs from the LLM.
- Using an LLM as a relevance filter. We ask an LLM if a document is likely to contain context relevant to the user's query. Essentially, we're using the LLM as a boolean classifier. This can be accomplished using [logit bias](/blog/llms-non-programmatic-computing-and-logit-bias). To reduce token consumption this can be performed on the titles of the document or other metadata which represents an abbreviation of documents complete context.

<hr>

# Spreadsheet of open-source libraries

➡️ [Open sheet in full window](https://docs.google.com/spreadsheets/d/1vPshk1R0O9vb7RjXT-KJ9-YEPsg1rrcXwriQHEgkISM)

<iframe style="height: 650px; width: 100%;" src="https://docs.google.com/spreadsheets/d/e/2PACX-1vRoAuiO_BLEP8oMB4flfwsa2ZWHJsZYsThZ5fcI-Ewg33ZsS_vCLfhBqk1s5MO1RieityzM3mVB8Grj/pubhtml?widget=true&amp;"></iframe>

Continue to ➡️ [Part 2](/blog/retrieval-is-all-you-need-2)
