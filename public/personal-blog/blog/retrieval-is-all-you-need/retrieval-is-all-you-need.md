---
title: 'GenAI: Retrieval is all you need'
# Optional
description: 'Cataloging the SotA options available for retrieval pipelines in Gen AI. With an attached spreadsheet.'

# Published date is required and in the format of ISO-8601: `yyyy-mm-dd`. For more info see https://docs.astro.build/en/guides/content-collections/#working-with-dates-in-the-frontmatter
pubDate: 2024-03-17
# Optionally specify an update date. If not provided, one will be generated from the git history. Only if the post has been changed since the day published.
# updatedDate:
heroImage: './hero.webp'
# Optional
---

This **in progress** post will be continually updated. The gap it aims to fill is a lack in documentation of complete RAG workflows. While databases of [vector databases](https://superlinked.com/vector-db-comparison/) exist, there is a gap for other retrieval techniques; specifically the various semantic search libraries and other advance search methods.

<h1>LLMs as arbitrage engines and a brief introduction to retrieval augmented generation</h1>

<blockquote class="reddit-embed-bq" data-embed-height="260"><a href="https://www.reddit.com/r/programming/comments/10x38hn/comment/j7rhkn7/">Comment</a><br> by<a href="https://www.reddit.com/user/2bias_4ever/">u/2bias_4ever</a> from discussion<a href="https://www.reddit.com/r/programming/comments/10x38hn/microsoft_announces_new_bing_chatgpt_within_a/"><no value=""></no></a><br> in<a href="https://www.reddit.com/r/programming/">programming</a></blockquote><script async="" src="https://embed.reddit.com/widgets.js" charset="UTF-8"></script>

I'm at risk of anthropomorphizing a piece of software when I say, "An LLM is making a series of decisions on how to respond to user input." Sure, it isn't _actually_ deciding anything, and is really just predicting the next most likely token, but for the sake of this conversation lets agree it's a distinction without difference; An LLM takes training input and input context, and returns useful outputs. That it can reliably _choose_ the most useful information to generate is _the reason LLMs are useful at all_.

In some ways, LLMs are no different than traditional retrieval methods like search engines or a SQL queries. Sure, LLMs can retrieve the information you need AND create a bespoke response on how to use that information to perform the task at hand, but really you just needed to know how to do XYZ mundane thing and you didn't feel like fighting your way to the documentation page and then actually reading the friendly manual.

It's the same when using an LLM in business logic. In a business workflow with thousands of potential input variants you could [just write a switch statement with a thousand cases](https://github.com/fachinformatiker/undertale/blob/master/scripts/SCR_TEXT.gml) like the creator of the beloved indie game Undertale famously did. In the end, the difference between an LLM and a billion manually written if statements is just a matter of will. Now with LLMs, we don't have to handle every possible input, and can instead ask the LLM to choose the correct action in a workflow. That is, based on this input, the LLM _retrieves_ the correct action to perform. It's retrieval all the way down.

![Always has been informational arbitrage.](./retrieval-meme.jpg)(class:'medium')

LLMs infamously sound confident, even when confidently incorrect. So how can you be confident in using an LLM when it's training data doesn't encompass the state of every particle in the universe at this exact current instance? Available commercial models are generalist trained on limited public data from, perpetually it seems, two years ago! A retrieval layer to add additional context solves this problem by ensuring the LLM has the correct and up to date context it needs to be confidently correct. This is known as 'retrieval augmented generation' (RAG) or 'context enhanced querying' (CEQ).

The savvy may recognize that retrieval is a core competency for many, probably most, projects and businesses - not just those with an `.ai` TLD. Search tech like semantic search has been evolving for some time, but has recently been given a hype injection for it's use with generative AI. But better search isn't just 'picks and shovels' sold to the AI goldrush out of the VC's general stores; being able to sift and pan the correct data out of endless streams of data creates value. The old metaphor of 'a needle in a haystack' can be turned-of-a-phrase to 'nuggets of gold in the river'. This post will catalog available SotA tools for retrieval, with a focus for use with LLMs.

<!-- See https://help.miro.com/hc/en-us/articles/360016335640-How-to-embed-Miro-boards -->
<iframe width="748" height="428"  src="https://miro.com/app/embed/uXjVKe2md8I=/?pres=1&frameId=3458764582902485384&embedId=563512251893&autoplay=yep" scrolling="no" allow="fullscreen; clipboard-read; clipboard-write" allowfullscreen></iframe>

> Miro embeds seem bugged and aren't showing text inside shapes. Will fix.

<h1>Retrieval technology</h1>

<h3>My experience implementing RAG</h3>

I used Pinecone (a vector database) for my first RAG application, and it was extremely easy to get started. With just an API key and a python library, I was getting relatively good results. My application was a chat bot on documentation for a dev tooling company's Slack and public Discord. In general it worked, but there were issues with retrieving technical documentation - semantic search struggles with domain novel technical keywords and strings. Think of a user asking:

> What does this error mean? Error: 1112345ab - bad lift header

**In this case, semantic search will return documentation related to errors, but not that _specific_ error.** Some vector DBs have text, keyword, or hybrid search which could solve this by retrieving documents with any of those strings. Whether or not you rely on a vector DB for this functionality or use additional tools, semantic search is only a _part_ of full retrieval strategy.

<h2>Traditional text search</h2>
TBD

<h2>Semantic search</h2>
<h3>ANN</h3>
<h3>KNN</h3>

<h2>Using LLMs for retrieval</h2>
tbd

<h1>Retrieval Stacks</h1>

There is a tendency for the 'RAG AI' stack to necessarily use semantic search via vector dbs for retrieval. I will make no comment of how we got here, but I suspect the dev rel teams for vector db providers are doing fantastic work. However, by no means are vector DBs the only, or necessarily the best, option. As always, the best option will depend entirely on your use case.

<iframe style="height: 377px; width: 100%;" src="https://docs.google.com/spreadsheets/d/e/2PACX-1vRoAuiO_BLEP8oMB4flfwsa2ZWHJsZYsThZ5fcI-Ewg33ZsS_vCLfhBqk1s5MO1RieityzM3mVB8Grj/pubhtml?widget=true&amp;"></iframe>

➡️ [Open sheet in full window](https://docs.google.com/spreadsheets/d/1vPshk1R0O9vb7RjXT-KJ9-YEPsg1rrcXwriQHEgkISM)

<h2>Vector Databases</h2>

<!-- <iframe width="768" height="432" src="https://miro.com/app/embed/uXjVKe2md8I=/?pres=1&frameId=3458764582900459388&embedId=384786121699&autoplay=yep" frameborder="0" scrolling="no" allow="fullscreen; clipboard-read; clipboard-write" allowfullscreen></iframe> -->

Vector databases range from bare-bones open source software created by a single person like [sqlite-vss](https://github.com/asg017/sqlite-vss) to cloud only services with Unicorn valuations like Pinecone to those somewhere in between like Qdrant or LanceDB.

These solutions aim to be scalable to enterprise levels, whether through a managed cloud service or with a self-hosted architeture. Some even have in-process implementations which can be consumed as a stand alone packages in popular languages. The strengths of vector DBs are that if you _need_ semantic search there are turn key options for every scale and architecture.

<h2>Traditional databases and search providers</h2>

Traditional SQL and no-SQL databases offer text search capability ranging from primitive keyword matching to near SotA search techniques. There are also extensions that add capabilities for semantic search like the [pgvector](https://github.com/pgvector/pgvector) for Postgres.

Beyond just databases, there are also tools specifically for search and retrieval with industry staples like Elasticsearch and Clickhouse as well as new comers like Algolia and Miliesearch.

These are a good starting point for building a RAG implementation as you likely already have experience with these, and so there is little overhead to fitting these into your PoC or MVP. They might also be used in conjecture with more advanced or specialized search techniques.

<h2>Stand alone search libraries</h2>

Open source libraries with novel retrieval techniques are also an option. They range from simple libraries that have basic APIs for accessing algorithms like [Spotify's ANNOY](https://github.com/spotify/annoy) to more full featured libraries with advanced interfaces like [usearch](https://github.com/unum-cloud/usearch).

Advantages for this technique are the simplicity of implementing in-process software; A library can be installed with a single terminal command, and implemented with a few lines of code in your application. This is attractive when the alternative is stand-alone databases or services that require _Yet Another Docker Container And API_ for you to manage.

Of course, implementing a stand alone library is not always _that simple_. You need to integrate with your existing document database, create embeddings, build an index in memory, and manage the CI/CD pipeline for this if you choose to use it in production. Additionally, since these libraries tend not to be 'products', they might not be under active development. They're as good as they'll get, and no further improvements will be made.

<h1>Resources</h1>

- [SuperLinked's vector database comparison tool](https://superlinked.com/vector-db-comparison/)
- [Prashanth Rao's excellent series on vector databases](https://thedataquarry.com/posts/)
- [Awesome Semantic Search](https://github.com/currentslab/awesome-vector-search)
