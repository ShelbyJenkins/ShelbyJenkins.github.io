---
title: 'GenAI: Retrieval is all you need'
# Optional
description: 'Cataloging the SotA options available for retrieval pipelines in Gen AI. With an attached spreadsheet.'

# Published date is required and in the format of ISO-8601: `yyyy-mm-dd`. For more info see https://docs.astro.build/en/guides/content-collections/#working-with-dates-in-the-frontmatter
publishedDate: 2024-03-17
# Optionally specify an update date. If not provided, one will be generated from the git history. Only if the post has been changed since the day published.
# updatedDate:
heroImage: './hero.webp'
# Optional
---

This will be an evergreen post that will be continually updated. Currently I'm in the process of researching search libraries as there isn't an easy comparison tool for them like there is [for vector databases.](https://superlinked.com/vector-db-comparison/)

<h1>The sheet</h1>

<iframe style="height: 377px; width: 100%;" src="https://docs.google.com/spreadsheets/d/e/2PACX-1vRoAuiO_BLEP8oMB4flfwsa2ZWHJsZYsThZ5fcI-Ewg33ZsS_vCLfhBqk1s5MO1RieityzM3mVB8Grj/pubhtml?widget=true&amp;"></iframe>

➡️ [Open sheet in full window](https://docs.google.com/spreadsheets/d/1vPshk1R0O9vb7RjXT-KJ9-YEPsg1rrcXwriQHEgkISM)

<h1>A brief introduction to retrieval</h1>

The savvy may recognize that retrieval is a core technology for many, probably most, projects and businesses - not just those with an `.ai` TLD. Search tech like semantic search has been evolving for some time, but has recently been given a hype injection as a required competency for generative AI. With cool names like 'retrieval augmented generation' (RAG) and 'context enhanced querying' (CEQ) it's easy to see why.

But better search isn't just 'picks and shovels' sold to the AI goldrush out of the VC's general stores; all value is created from data arbitrage and therefore being able to sift and pan the correct data out of endless streams of data can create real value. The old metaphor of 'a needle in a haystack' can be turned-of-a-phrase to 'nuggets of gold in the river'.

<h2>Cataloging retrieval options</h2>

There is a tendency for the 'RAG AI' stack to necessarily use semantic similarity via vector dbs for retrieval. I will make no comment of how we got here, but I suspect the dev rel teams for vector db providers are doing fantastic work. However, by no means are vector DBs the only, or necessarily the best, option. As always, the best option will depend entirely on your use case.

<h3>Traditional databases and search providers</h3>

Traditional SQL and no-SQL databases offer text search capability ranging from primitive keyword matching to near SotA search techniques. There are also extensions that add capabilities for semantic search like the [pgvector](https://github.com/pgvector/pgvector) for Postgres.

Beyond just databases, there are also tools specifically for search and retrieval with industry staples like Elasticsearch and Clickhouse as well as new comers like Algolia and Miliesearch.

These are a good starting point for building a RAG implementation as you likely already have experience with these, and so there is little overhead to fitting these into your PoC or MVP. They might also be used in conjecture with more advanced or specialized search techniques.

<h3>Stand alone search libraries</h3>

Open source libraries with novel retrieval techniques are also an option. They range from simple libraries that have basic APIs for accessing algorithms like [Spotify's ANNOY](https://github.com/spotify/annoy) to more full featured libraries with advanced interfaces like [usearch](https://github.com/unum-cloud/usearch).

Advantages for this technique are the simplicity of implementing in-process software; A library can be installed with a single terminal command, and implemented with a few lines of code in your application. This is attractive when the alternative is stand-alone databases or services that require _Yet Another Docker Container And API_ for you to manage.

Of course, implementing a stand alone library is not always _that simple_. You need to integrate with your existing document database, create embeddings, build an index in memory, and manage the CI/CD pipeline for this if you choose to use it in production. Additionally, since these libraries tend not to be 'products', they might not be under active development. They're as good as they'll get, and no further improvements will be made.

<h3>Vector Databases</h3>

Vector databases range from bare-bones open source software created by a single person like [sqlite-vss](https://github.com/asg017/sqlite-vss) to cloud only services with Unicorn valuations like Pinecone to those somewhere in between like Qdrant or LanceDB.

These solutions aim to be scalable to enterprise levels, whether through a managed cloud service or with a self-hosted architeture. Some even have in-process implementations which can be consumed as a stand alone packages in popular languages. The strengths of vector DBs are that if you _need_ semantic search there are turn key options for every scale and architecture.

<h4>My experience with Vector DBs</h4>

I used Pinecone for my first RAG application, and it was extremely easy to get started. With just an API key and a python library, I was getting relatively good results. My application was a chat bot on documentation for a dev tooling company's Slack and public Discord. In general it worked, but there were issues with retrieving technical documentation - semantic search struggles with domain novel technical keywords and strings. Think of a user asking `What does this error mean? Error: 1112345ab - bad lift header`. At best, semantic search will return documentation related to errors. Some vector DBs have text, keyword, or hybrid search which could solve this by retrieving documents with any of those strings. However, if you _only_ use a vector DB you will always be at the mercy of your vector DB to implement the latest and greatest techniques that solves your needs.

I will speculate that Google's decay in search quality over the past decade is at least in part related to their implementation of semantic search. In e-commerce we can also see the weakness in semantic search.

> If you're young, you may not understand how search used to work. If you want an illumination, compare the old-school search results from Ebay to the 'modern' results from Amazon search.

For this reason, I believe that semantic search, and therefore vector DBs, are only a _part_ of retrieval strategy. My reason for creating this post is to assemble and catalog the various options to implement retrieval.

<h1>Resources</h1>

- [SuperLinked's vector database comparison tool](https://superlinked.com/vector-db-comparison/)
- [Prashanth Rao's excellent series on vector databases](https://thedataquarry.com/posts/)
- [Awesome Semantic Search](https://github.com/currentslab/awesome-vector-search)
