---
title: 'Retrieval is all you need (Part 2): RAG building blocks and stacks'
# Optional
description: 'Cataloging the SotA technologies available and permutations of stacks for retrieval pipelines in Gen AI. Part 2 is a continuously updated collection of technologies and workflows for RAG.'

# Published date is required and in the format of ISO-8601: `yyyy-mm-dd`. For more info see https://docs.astro.build/en/guides/content-collections/#working-with-dates-in-the-frontmatter
pubDate: 2024-03-24
# Optionally specify an update date. If not provided, one will be generated from the git history. Only if the post has been changed since the day published.
# updatedDate:
heroImage: './hero.webp'
# Optional
---

This post will be a **continually updated** overview of RAG implementations. If you have any questions about the technology discussed, see [part 1](/blog/retrieval-is-all-you-need-1).

- [Building blocks](#building-blocks)
  - [Traditional databases](#traditionial-databases)
  - [Vector databases](#vector-databases)
  - [Full text search databases](#full-text-search-databases)
  - [Stand alone search libraries](#stand-alone-search-libraries)
  - [Embeddings](#embeddings)
- [Stacks](#stacks)
  - [Where to run](#where-to-run)
  - [All-In-One](#all-in-one)
  - [Document database + retrieval index](#document-database--retrieval-index)

<!-- See https://help.miro.com/hc/en-us/articles/360016335640-How-to-embed-Miro-boards -->
<iframe width="748" height="428"  src="https://miro.com/app/embed/uXjVKe2md8I=/?pres=1&frameId=3458764582902485384&embedId=563512251893&autoplay=yep" scrolling="no" allow="fullscreen; clipboard-read; clipboard-write" allowfullscreen></iframe>

# Building blocks

Many tutorials of the 'RAG stack' use semantic search via vector dbs for retrieval. Congrats to the #devrel teams for vector db providers for the fantastic work in this! Indeed, this is a very quick way to get started with decent results. However, by no means are vector DBs the only option. As always, the best option(s) will depend entirely on your use case.

<iframe style="height: 377px; width: 100%;" src="https://docs.google.com/spreadsheets/d/e/2PACX-1vRoAuiO_BLEP8oMB4flfwsa2ZWHJsZYsThZ5fcI-Ewg33ZsS_vCLfhBqk1s5MO1RieityzM3mVB8Grj/pubhtml?widget=true&amp;"></iframe>

➡️ [Open sheet in full window](https://docs.google.com/spreadsheets/d/1vPshk1R0O9vb7RjXT-KJ9-YEPsg1rrcXwriQHEgkISM)

## Traditional databases

Traditional databases now have advanced retrieval and search functionality, either natively or via extensions. This may be attractive if you're already using these databases. However, since this functionality is not a core feature, the feature may not be as performant as tools that specialize in that feature. For example [sqlite-vss](https://github.com/asg017/sqlite-vss) is not going to perform as well as a dedicated vector database.

## Vector databases

Vector databases range from bare-bones open source software created by a single person to cloud only services with Unicorn valuations like Pinecone to those somewhere in between like Qdrant or LanceDB.

These solutions aim to be scalable to enterprise levels, whether through a managed cloud service or with a self-hosted architecture. Some even have in-process implementations which can be consumed as a stand alone packages in popular languages. The strengths of vector DBs are that if you _need_ semantic search there are turn key options for every scale and architecture.

## Full text search databases

Traditional FTS is may be a good starting point for building a RAG implementation if you already have experience with them. They also excel at FTS whereas the vector databases that implement FTS may not be as feature rich when it comes to FTS. For a very sophisticated RAG workflow combining a FTS provider with a vectorDB may make sense.

## Stand alone search libraries

This category consists of open-source libraries with novel search techniques or algorithms. They range from simple libraries that have basic APIs for accessing algorithms like [Spotify's ANNOY](https://github.com/spotify/annoy) to more full featured libraries with advanced interfaces like [usearch](https://github.com/unum-cloud/usearch).

These libraries don't handle your dataset, and require more work to implement. However, they may provide SoTA search functionality.

## Embeddings

WIP

# Stacks

If you're building greenfield, this is probably a harder decision. Sometimes being boxed into choice is a blessing. So if you're starting from scratch, what do you do? You want something quick to build and iterate on, but also scalable, but also without vendor lock. Analysis paralysis!

## Where to run

I do all of my development in a devcontainer. It's fantastic, and it makes maintaining and deploying an app from a repo simple. I will assume this document is for people building with a container as a deployment target.

That being said, I prefer databases or retrieval services that support an in-process / embedded model. For example LanceDB and Qdrant can both be instantiated from the Python client which makes the initial setup and deployment easier. Other's may require more setup work to run in a container or may even require a separate docker container which means multi-container deployments.

## All-in-one

Just use a single database for your documents and use it's retrieval features for everything. There are two options here:

- An in-process database or running the database in your apps container
- A cloud based database via API

<iframe width="768" height="432" src="https://miro.com/app/embed/uXjVKe2md8I=/?pres=1&frameId=3458764583918028234&embedId=648071622447&autoplay=yep" frameborder="0" scrolling="no" allow="fullscreen; clipboard-read; clipboard-write" allowfullscreen></iframe>

Pros:

- Easiest way to start. Even easier if you're using a cloud service.
- Results should be decent.
- In theory, you can scale this:
  - If you are using a cloud provider, they can handle it.
  - If your self-hosting a service that supports sharding, replication, or other advanced scaling features.

Cons:

- If the retrieval results do not meet your needs, you'll have to migrate.
- Scaling may be a challenge; either you have to engineer scaling yourself, or pay for it.
- If you need to migrate, you need to re-implement both storage _and_ retrieval.

## Document database + retrieval index(s)

In this case you have you have a database strictly for storing of the original documents, and then a secondary service for retrieval. The retrieval index can be a vector or FTS database or a standalone library that generates a local disk or in-memory index to retrieve from. Either of the two services can run in the cloud or _backup_ to a cloud service.

<iframe width="768" height="432" src="https://miro.com/app/embed/uXjVKe2md8I=/?pres=1&frameId=3458764583920033154&embedId=727820748597&autoplay=yep" frameborder="0" scrolling="no" allow="fullscreen; clipboard-read; clipboard-write" allowfullscreen></iframe>

Pros:

- A traditional database is extremely well supported for both local use or cloud use and can easily be migrated or scaled.
- By using a base document database for storage, you're able to use any retrieval method. This is useful for testing different retrieval methods, and even adding additional retrieval techniques from multiple providers.

Cons:

- More complex. Congrats, you've now adopted two different databases that require love and care.

This is a WIP and being continuously updated.

<h1>Resources</h1>

- [SuperLinked's vector database comparison tool](https://superlinked.com/vector-db-comparison/)
- [Prashanth Rao's excellent series on vector databases](https://thedataquarry.com/posts/)
- [Awesome Semantic Search](https://github.com/currentslab/awesome-vector-search)
