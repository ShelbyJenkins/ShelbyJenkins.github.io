---
title: 'LLMs, non-programmatic computing, and logit bias.'
# Optional description: If provided the description field will be used for previews. If not provided, the first 3-4 lines from the post will be used as a description.
description: 'Exploring the syntax we use to describe LLMs and a discussion on how to use LLMs to make business logic decisions in software with logit bias.'

# Published date is required and in the format of ISO-8601: `yyyy-mm-dd`. For more info see https://docs.astro.build/en/guides/content-collections/#working-with-dates-in-the-frontmatter
pubDate: 2023-08-22
# Optionally specify an update date. If not provided, one will be generated from the git history. Only if the post has been changed since the day published.
modifiedDate: 2023-08-22
heroImage: './hero.webp'
# Optional
heroAlt: 'A poorly made hero using some python libraries in chatGPT.'
---

<h6>

Yes, I did use ChatGPT's code interpreter to make the hero image using some python libraries as a joke. [See the code.](https://chat.openai.com/share/e9aa1240-2a84-4e10-be25-a6e4ce7a9b31)

</h6>

<hr>
Oh, what Promethean days we live in! With the creation of LLMs we have entered in a new era of computing… do we call it an era of non-deterministic computing? Maybe non-programmatic computing? Lets ask ChatGPT.

> Language models like GPT (from OpenAI) are often misunderstood to be non-deterministic, but they’re actually deterministic in their operation. Let’s break this down:
>
> <h2>Deterministic vs. Non-deterministic:</h2>
>
> - Deterministic means that for the same input, you will always get the same output.
> - Non-deterministic means that the same input can produce different outputs on different runs, even under the exact same conditions.
>   GPT and similar large language models (LLMs) are deterministic. If you give the model the same input and the model’s weights are unchanged, it will produce the same output every time. The non-deterministic perception might come from the vastness of the model and the wide range of possible outputs it can generate based on slight variations in input, but the computation itself is deterministic.\*

\*you can turn up the spiciness to introduce some randomness though.

> <h2> Programmatic vs. Non-programmatic computing:</h2>
>
> Programmatic computing typically refers to executing explicit instructions written by a programmer.
> Non-programmatic might refer to computing models that learn from data and make decisions based on that data, rather than following a specific set of instructions.
>
> In this sense, GPT and other LLMs operate in a non-programmatic manner when producing outputs because they’re generating responses based on patterns learned from data rather than following an explicit set of programmed instructions.
>
> However, it’s essential to understand that while the model’s response generation might seem non-programmatic, the underlying mechanisms (like matrix multiplications, activations, etc.) are all deterministic and based on well-defined algorithms. The “non-programmatic” nature refers more to the model’s capability to generalize from its training data and produce outputs for a wide range of inputs, rather than the computational steps it undergoes.

So, if you’re trying to deflate the hype and magic around LLMs, one could say that while any sufficiently advanced technology is indistinguishable from magic, it ain’t LLMs; they’re just plain old computing — math and numbers and lots of IF statements. And also some vast datasets it bounces tokens through like a ball dropping through the pins of a plinko machine.

But ChatGPT wants you to know:

> While this metaphor is illustrative, it can be misleading. The path a ball takes through a Plinko machine’s pins is governed by physics and is, in theory, deterministic. However, it evokes a sense of randomness because, in practice, the exact conditions are nearly impossible to replicate consistently.

So, by the same logic, in theory LLMs are deterministic, but in practice they are not. When you use an LLM, it’s outputs don’t behave programmatically or deterministically. So for those of us concerned with systems in practice, it represents a fundamentally a new era of computing. Yes, it makes mistakes. Yes, it can be slow. Yes, it’s annoyingly every where, all the time, all at once. But it’s exciting, it’s new, and here’s why.

I can ask an LLM a yes or no question like this:

```yaml
role: system
  content: |
  A user has made a query. The query may be related to a list of topics.
  The list of available topics and descriptions will be included like:
  "topic name: description"
  If this query is related to one of these topics, return '1'.
  Otherwise, return '0'.
role: user
    content: <user input>
```

And I can expect it will reliably work 99% of the time. In case you don’t understand why this is impressive let me try to quickly explain. Ten years ago, writing an if/else statement to handle this problem would of taken PhD. level computer science and a dictionary, thesaurus, and maybe an encyclopedia worth of text to solve. Or maybe not. But it wouldn’t of been solvable by merely asking the first generation commercial LLM and banging out 15m of code to handle the logistics of proxying the requests and responses.

This is the era of computing we are in: complex tasks can now be solved by feeding enough relevant data to a model, and then prompting it correctly. This is a departure from the old way of manually writing as many thousands of conditional statements required to get the job done.

<hr>

So how can you use this yourself? It’s actually rather simple, and works with the GPT-3.5 or GPT-4. GPT-4 is obviously better for accuracy, but speed wise they’re both fast enough that 3.5 does not have an advantage outside of cost.

The first thing you need to wrap your head around is logit bias.

[From the Open API Reference:](https://platform.openai.com/docs/api-reference/chat/create#chat-create-logit_bias)

```json
logit_bias

Modify the likelihood of specified tokens appearing in the completion.

Accepts a json object that maps tokens (specified by their token ID in the GPT tokenizer) to an associated bias value from -100 to 100. You can use this tokenizer tool (which works for both GPT-2 and GPT-3) to convert text to token IDs. Mathematically, the bias is added to the logits generated by the model prior to sampling. The exact effect will vary per model, but values between -1 and 1 should decrease or increase likelihood of selection; values like -100 or 100 should result in a ban or exclusive selection of the relevant token.

As an example, you can pass {“50256”: -100} to prevent the <|endoftext|> token from being generated.
```

A token is simply a collection of characters represented by some numbers.

```py
# Tokens can be whole words
apple = [18040]
# Or numbers
3 = [18]
# Or letters
s = [82]
# Or fragments of words
hel = [2978]
by = [1525]
# Or they can be combined together to form longer words
shelby = [82, 2978, 1525]
```

And logit bias is simply influencing which tokens the LLM returns.

So I can say: under no circumstance return token [19155] — (representing the word “sun”), and if I ask it to describe the sun it will return something like.

> The fiery celestial body at the center of our solar system provides light and warmth, fueling life on Earth with its vast nuclear reactions. It consists primarily of hydrogen and helium, undergoing continuous fusion processes that release immense energy.

I use logit bias for a few things in my RAG question and answer bot.

- I use it to check if incoming questions are relevant to the topics the bot has documents for. Because there is no point in burning tokens for a question a user can ask ChatGPT for free!
- With the Open API minifier project I’m adding to the main service we will be able to have LLMs pick an API endpoint from an index file, and then generate the API call for the bot to make. Logit bias is used twice, first to decide if the incoming request requires an API call, and then second to choose the correct endpoint from the index file.
- I use it to pick out relevant documents returned from the vectorstore. There is a limit to how much context can (and should) be stuffed into a prompt, so filtering out anything not relevant is helpful. Semantic search is good but not perfect, so GPT acting as an extra filter improves the final responses.

```yaml
- role: system
  content: |
  A software developer has asked a question.
  Which of the following documents would be the most useful in answering the question?
  The document titles are numbered using 'title doc_number: [n]' notation.
  Select documents by specifying their doc_number.
  If no relevant documents are found, return simply '0'.
  If multiple documents may be used, use '\n' chars to seperate your selections like:
  n
  n
  n
  ...
- role: user
    content:
```

This is the [prompt template](https://github.com/shelby-as-a/shelby_as_a_service/blob/main/app/prompt_templates/ceq_doc_check.yaml) I use for the doc relevancy check.

```py
doc_counter = 1
content_strs = []
documents_str = ""
for doc in documents:
content_strs.append(f"{doc['title']} doc_number: [{doc_counter}]")
documents_str = " ".join(content_strs)
doc_counter += 1
prompt_message = "Query: " + query + " Documents: " + documents_str

for role in prompt_template:
if role['role'] == 'user':
role['content'] = prompt_message
```

We iterate the returned documents and attach their titles and a number to the prompt like so.

```yaml
role: user
content: Query: Describe the sun. Documents: Title1 doc_number: [1] Title2 doc_number: [2] Title3 doc_number: [3]
```

Finally, we programmatically generate a dict for the logit_bias field where each doc gets a number that is set to a bias of 100. So if we have 3 docs:

```py
logit_bias_weight = 100
# 0-9
logit_bias = {
str(k): logit_bias_weight for k in range(15, 15 + len(documents) + 1)
}
# \n
logit_bias["198"] = logit_bias_weight
# Expected output
{
"15": "100", # Corresponding to no relevant documents
"16": "100", # Corresponding to the first document
"17": "100", # Corresponding to the second document
"18": "100", # Corresponding to the third document
"198": "100" # Representing the newline character
}
```

So with this logit_bias, GPT can only return 0, 1, 2, 3, or ‘\n’.

Which is perfect because not only is it just a few tokens, making it near instant and cheap, it also gives answers that are more accurate than it has the reputation for. It reliably chooses documents that are relevant to the user’s query, and then we’re able to attach those docs to the users query as additional context for the LLM. Win win win, and we just did natural language, non-programmatic computing. Neat. Welcome to the future.

> Using LLMs to generate structured output is an excellent modern example of the old, “I’ll use regex to solve my problem! Now I have two problems.” story.

Would I use this method for something mission critical? Absolutely not. But it’s a taste of what to come, AND I’m having great success using it with my Slack and Discord bots. Check them out if you’re interested in playing with LLMs and semantic search!
