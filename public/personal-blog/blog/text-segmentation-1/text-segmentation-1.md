---
title: 'Text Segmentation for RAG and the Shaky Foundation of AI'
description: "Text segmentation and it's common shortcoming in RAG, LLMs, and AI."
pubDate: 2024-06-30
heroImage: './hero.webp'
---

RAG stands for "Retrieval Augmented Generation" and the later two words is what you do when you paste the transcript of your companies all-hands into ChatGPT and ask if your department was mentioned. General LLMs are only useful for generalized knowledge, so to do anything interesting you either need to train a model, or do some sort of RAG. The 'R' in RAG is doing most of the work though, or rather I should say it requires the _most_ work to get right. Retrieving the correct content, reliably, is hard. Remember how loved Google was twenty years ago when they had search right when no one else did? Now every AI tool requires their own, internal, version of Good Google.

LLMs (and semantic search) are very, very, very good at putting an Instagram 'beauty filter' on their outputs, and objectively evaluating a RAG pipeline is hard. So if it looks like it's working, it's left alone. But it's _not working._ Specifically, preparing content for RAG is not a solved problem.

Preparing documents for RAG is dependent on your use case, but has two constraints: how will it be stored in the retrieval system, and how will it be used in an LLM. Vector databases (semantic search) creates embeddings of from documents. Those embeddings have a maximum length they can be created from. Similarly, LLMs have a maximum length an input can be. Both of these maximums are given in their 'token' count. [OpenAI has a fun tool](https://platform.openai.com/tokenizer?view=bpe) for playing with the text/token relationship.

_So, just for hard limits of the models, preparing documents for RAG requires splitting text into smaller 'chunks'. This is called 'chunking', and is the topic of this post._

# The Problem With Naive Text Splitting

There are other reasons to split text as well. An extremely good one is as a matter of efficiency; it's wasteful to jam a full document into an LLM when you only need a sentence from it. Someone is paying for those tokens! But an even better reason than that: garbage in, garbage out. There are people who hype the LLMs with massive, 200k token count limits as the solution to all problems. Have they never closed down a ChatGPT chat and opened a new one because it got hung up on something it said five messages ago? LLMs are really good at ignoring irrelevant details, but it's still a dice roll. And if enough irrelevant details build up, eventually it will cause mistakes. Jamming a full help article into an LLM acting as a customer support bot might work often enough, but it's not ideal. The ideal approach would be to use only context that is actually relevant. _This requires 'topic based chunking' and is something I'm currently working on. This will be covered in part two._

# The Problem With Chunking

Imagine you have a sentence `"one, two, three, four, five, six, seven, eight, nine"`, and you want to split it into chunks no more than four words long.

Your text chunking tool will probably produce something like `"one, two, three, four"`, `"five, six, seven, eight"`, `"nine"`. The final chunk is now 'orphaned' from any surrounding context that would make it useful. If you asked your RAG implementation `What did seven eat?`, that final chunk that answers the question would not be retrievable.

> As long as the the total token count of the incoming text is not evenly divisible by they max token count, the final chunk will be smaller than the others. In some cases it will be so small it will be "orphaned" and rendered useless.

This is pervasive problem because most people don't even know it is a problem. Notable implementations with this issue are [LlamaIndex](https://github.com/run-llama/llama_index/blob/b17edc7c6800ba8c4f4c86e23ad57f2e3a0c3714/llama-index-core/llama_index/core/node_parser/text/token.py), and [Langchain](https://github.com/langchain-ai/langchain/blob/29aa9d67506ac07b92d37d58c684ce3c6dc290cd/libs/text-splitters/langchain_text_splitters/character.py#L58). Here is [a great visualizer of chunking](https://chunkviz.up.railway.app/) that shows the issue visually.

# The Solution Is Obvious

Text chunkers should attempt to 'balance' the chunks they produce so all are near the same size: `"one, two, three"`, `"four, five, six"`, `"seven, eight, nine"`.

![Why didn't I think of that?](./scorpio.jpg)(class:'small')

<p class="caption">My Goodness Why Didn't I Think of That</p>

I have a some Rust projects. I have [llm_client](https://github.com/ShelbyJenkins/llm_client) which is an interface for llama.cpp, mistral.rs, openai, and anthropic with a long term goal to create the easiest way to embed a local LLM into Rust projects for decision making. Rust doesn't have the depth of tooling as other ecosystems, so I ended up rolling a lot of my own tools I needed for llm_client. Around the point where I created the 3rd or 4th implementation of something that didn't yet exist in Rust, I spun off the [llm_utils](https://github.com/ShelbyJenkins/llm_utils) crate. No 'chains', 'agentic AI', or x buzzword - just high quality tools.

The text chunking implementation discussed here is part of the llm_utils crate. If you're curious how it works, [check out some of the tests.](https://github.com/ShelbyJenkins/llm_utils/blob/main/src/text_utils/chunking/mod.rs) I do not claim to have invented this methodology, but I have not come across any implementations except mine. I'm sure they exist. Please send them to me because I'm curious how others have done it. As for why it's not the defacto standard? Because it's a deceptively difficult problem - especially to do it efficiently! My original implementation in Python last year was 100x slower than Langchain's `RecursiveTextSplitter`!

# One-Shot Linear Chunking

The standard chunking implementation is to attempt to build chunks from 'splits' where a split is some division of the incoming text. First the largest and most likely semantically relevant information breakpoint is used: paragraphs. What often happens is that there actually isn't any paragraphs in the text, or maybe a paragraph is too large to be used. So then we try the next 'separator' to split the text on and retry until we find a separator works.

> 'semantic' separator priority: paragraphs -> sentences -> words -> characters (graphemes)

The unbalanced chunk implementation for building chunks is simple: add a split to a chunk. If the chunk is below some token count minimum, add another split. If it's above the minimum, and below the maximum, the chunk is complete. Do this with until you run out of splits. The final chunk ignores any token count minimum, and you often end up with orphaned chunks. If for some reason the attempt fails, re-split on the next separator and retry.

This is simple and fast, but produces unbalanced chunks.

# Depth First Search (DFS)

My initial balanced chunking implementation uses DFS to find a combination of splits where all chunks were within the same size window.

Starting at the first split we build a chunk until the chunk size is above the minimum. We save any split that put the chunk within the valid size range as a `valid_end_split` because the range of splits between the end split and the starting split could be used to build a valid chunk.. We then continue building the chunk and adding any additional valid end splits to a list of good end splits for the starting split. When we exceed the maximum token length for a chunk, we break.

For each valid end splits we found, we recursively call the function with the end as the new start. Eventually each "depth" exploration attempt runs out of splits to create a chunk with, at which point it tries the other end splits in the function that called it. The successful exit condition is when an attempt runs out of splits while generating a valid chunk. Meaning, we successfully found a combination of splits where each is in a chunk within the given valid ranges. This is fast and is optimized by using memoisation so that once we've seen an end split, we knows it's a bad path and can skip it.

Here I ran into an issue fundamental to token based text splitting. (As an aside, the _actual_ first issue I ran into was the lack of rule based or ML implementations of sentence splitting in Rust. See note at bottom.)

# Token Counting Difficulties

In principle, implementing this should be simple; we're just arranging text until the numbers are green. The difficulty comes because we aren't counting characters in text, _we're counting tokens_ and token counts for bits of text is dynamic relative to the text surrounding it.

The problem is that was that I was pre-counting the token count of splits, so I didn't have to do it within the chunk building process. I was then using these split token counts to increment the token counts of chunks. It was fast, but it turns out inaccurate! When we combine splits, the token counts are not always exactly equal to the sum of the token counts of each split: The sentence `one two three ` is a fixed number of tokens, but if we build a chunk of `one two three one two three ` it's not always the case the token count is simply double of `one two three `. The word `one` is a single token, but so is ` one`, `one `, `one.`, and other common permutations of `one`! Pre-calculating token counts was not an option. So then what about actually building the chunk as a string inline within the chunking process, and counting the tokens of a string each time a split is added? Obviously, it works. But tokenization is not cheap. Especially doing it thousands and thousands of times. It is **slow**: Large texts like Romeo and Juliet were seeing run times of ~60s while other chunking libraries were benching at 300ms for the same task.

The solution here was to use the actual token count of a split to _estimate_ how it's addition will increase the token count of the chunk. A synthetic token count! Each separator is given a ratio to adjust it's token count when adding to a chunk. For example, the token count of a sentence is only 99% of it's length when added to a chunk. 1% doesn't sound like much, but it adds up when working with chunks of dozens of sentences. An estimated token count allows us to call the tokenizer minimally: when a split is created and when a chunk is finalized.

# In Place Splitting

We were fast, but there was still a performance issue; the time complexity scales drastically as the count of splits goes up, and since the DFS method uses a single separator for each attempt it's very common to fail the larger splits and end up on tiny words or even individual characters for splits. This can result in a massive number of combinations to try, and was never going to be viable. I really wanted to be _near_ as fast as a traditional chunking implementation as technically possible.

This lead to an additional chunk building method, 'linear chunking + in place splitting'. Chunks are built as normal except when we reach a chunk that exceeds the maximum size, we split the split (on the next separator) that pushed it over the limit, and then use the smaller splits created to continue building the chunk to within the minimum and maximum token count. Because it splits all the way down to single characters this can get very granular to the point where chunks are _all_ the exact same token count.

In practice we end up with most chunks being made up with large splits, and only near the end is additional splitting required. The granularity does require greater accuracy and therefore actually calling the tokenizer. The optimization here is to run an iteration of the builder using token count estimates, and once we find a good chunk we feed that chunk back to the function using the actual token count to correct any inaccuracies.

# Faster With Parallelization

At this point performance was substantially improved and near what I thought was reasonable for my skill and time budget. But hey, `Rust` and `Fearless Concurrency` [right](https://doc.rust-lang.org/book/ch16-00-concurrency.html)? Why _not_ try rayon.

```rust
Separator::get_all().par_iter().find_map_any(|separator| {
            let config = Arc::new(ChunkerConfig::new(...)?);
            if config.initial_separator.group() == SeparatorGroup::Semantic {
                let chunks: Option<Vec<Chunk>> = DfsTextChunker::run(&config);
                if let Some(chunks) = chunks {
                    match chunks {
                        Ok(chunk) => {
                        println!(
                            "Successfully Split with:
                            DfsTextChunker on separator: {:#?}",
                            separator,
                            chunking_start_time.elapsed()
                        );
                        return Some(ChunkerResult::new(incoming_text, &config, chunking_start_time, chunk));
                        },
                        Err(e) => {
                            eprintln!("Error: {:#?}", e);
                        }
                    }
                }
            }
            let chunks = LinearChunker::run(&config)?;
            match chunks {
                Ok(chunks) => {
                    println!(
                        "Successfully Split with:
                        LinearChunker on separator: {:#?}",
                        separator,
                        chunking_start_time.elapsed()
                    );
                    Some(ChunkerResult::new(incoming_text, &config, chunking_start_time, chunks))
                }
                Err(e) => {
                    eprintln!("Error: {:#?}", e);
                    None
                }
            }
        })
```

It turned out to be really _easy_. We create a new thread for each separator. We create a config that generates the initial splits. This fails if the split count is less than the required chunks, but it's ok because another separator will succeed. Then if the separator is in the semantic group (paragraphs, sentences) we attempt to use the `DfsTextChunker`. This can fail, and that ok because we then try the `LinearChunker` which should always work. Because we do this in parallel, one thread (or separator attempt in this case) will finish first and we return those chunks.

# Speed Benchmarks

By this point I had spent a lot of time optimizing. The major Rust chunking crate, [text-splitter](https://github.com/benbrandt/text-splitter) does the traditional one-shot linear method, and so is obviously going to be significantly faster. It's also much more polished in general. I did [write some tests](https://github.com/ShelbyJenkins/llm_utils/blob/5f123a77394cefc104ed20fb545f5940e6da2ab9/src/text_utils/chunking/external_text_chunker.rs#L65) to compare the speeds, but there is an [ongoing issue with text-splitter causing extreme slow downs when using Tiktoken](https://github.com/benbrandt/text-splitter/issues/223) and so it would be unfair to compare them. Going off text-splitter's posted benchmarks my implementation is between 2x-10x slower. That sounds worse than it is because most chunking is typically not done on massive texts (and time complexities) like Romeo and Juliet. Smaller and more common texts are processed in reasonable times.

```
Speed comparison for content: The ACM A. M. Turing A...
content token_count: 284
absolute_length_max: 128
duration: 25.915969ms

Speed comparison for content: Doom (stylized as DOOM...
content token_count: 1244
absolute_length_max: 256
duration: 71.252411ms

Speed comparison for content: Who Controls OpenAI?...
content token_count: 4828
absolute_length_max: 512
duration: 153.630351ms

Speed comparison for content: The Call of the Wild...
content token_count: 39823
absolute_length_max: 1024
duration: 1.173179426s

Speed comparison for content: MEDITATIONS By Marcus ...
content token_count: 78479
absolute_length_max: 2048
duration: 2.317496042s
```

# Chunk Balance Comparison

```
The ACM A. M. Turing A...
content token_count: 284
absolute_length_max: 128

Balanced result smallest_token_size: 92
Unbalanced result smallest_token_size: 45

Doom (stylized as DOOM...
content token_count: 1244
absolute_length_max: 256

Balanced result smallest_token_size: 249
Unbalanced result smallest_token_size: 135

Who Controls OpenAI?...
content token_count: 4828
absolute_length_max: 512

Balanced result smallest_token_size: 389
Unbalanced result smallest_token_size: 152

The Call of the Wild...
content token_count: 39823
absolute_length_max: 1024

Balanced result smallest_token_size: 766
Unbalanced result smallest_token_size: 607

MEDITATIONS By Marcus ...
content token_count: 78479
absolute_length_max: 2048

Balanced result smallest_token_size: 1536
Unbalanced result smallest_token_size: 774

```

The smallest token sizes of the unbalanced chunks from text-splitter tell the story here! This project was way more work than I initially assumed it would be, but I'm happy with how much I learned.

With this working I can go back to working on topic extraction and topic based chunking for my [llm_client](https://github.com/ShelbyJenkins/llm_client) crate.

# A Note on Rust Sentence Segmentation

Rust lacks the ML ecosystem of Python. There is, for example, no equivalent of the [SpaCy sentence splitter](https://spacy.io/api/sentencizer). Rust does have the great Unicode segmentation crate, but it's splits based on limited rules. For example abbreviations are seen as sentence boundaries. Not ideal!

I implemented a [rule based sentence splitter](https://github.com/ShelbyJenkins/llm_utils/blob/main/src/text_utils/splitting/rule_based.rs) in the process of implementing the chunker. It's adapted from [another rust project](https://github.com/indicium-ag/readability-text-cleanup-rs/blob/master/src/katana.rs) with some changes. It's not at all perfect, but it works often enough! I perform a check to ensure the indices are correctly aligned, and if it fails the split attempt then fails over to splitting on unicode sentence boundaries.
