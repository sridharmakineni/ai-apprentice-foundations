# Inside the Model

Chapter 1 said a language model predicts the next chunk of text. True, and not yet
useful. This chapter turns that sentence into the four things you can actually control:
**tokens**, the **context window**, **temperature**, and **embeddings**.

These are not academic concepts. They are, in order: what you are billed for, what
constrains your design, what makes output reproducible or not, and what makes search
work. Every cost estimate, latency problem and architecture decision in the rest of this
series traces back to one of them.

## Tokens: the unit of everything

A model doesn't read characters and doesn't read words. It reads **tokens** — fragments
produced by splitting text on statistical boundaries learned during training.

Microsoft's own documentation uses a clean example: `hamburger` splits into `ham`, `bur`
and `ger` — three tokens. The short, common word `pear` is a single token.

The pattern: **common words are one token; unusual words fragment.** A rough English
rule of thumb is about three-quarters of a word per token, so 1,000 tokens is roughly
750 words. But the rule breaks exactly where you'd least like it to:

| Text | Behaviour |
|---|---|
| Ordinary English prose | ~0.75 words per token — the rule holds |
| Names, surnames, product codes | Fragment badly — several tokens each |
| Code, JSON, XML | Punctuation-heavy, token count climbs fast |
| Non-English text | Often far more tokens for the same meaning |
| Long numbers, GUIDs | Expensive — digits tokenize poorly |

This has a direct consequence for the enterprise work you'll do: a prompt full of
employee IDs, SKUs and GUIDs costs substantially more than a prompt of the same
character length in plain prose. If you are estimating cost, count tokens on
representative real data. Never extrapolate from a sentence of English.

**Tokens are the billing unit and the limit unit.** You pay per token in and per token
out. You are rate-limited in tokens per minute. Every constraint you hit is denominated
in tokens.

## The context window

The **context window** is the maximum number of tokens the model can consider in a
single request — your prompt plus the response it generates, counted together.

Here is where almost everyone forms the wrong mental model, so it's worth stating
bluntly:

> **The context window is not memory. It is the size of what you resend every time.**

Language models are **stateless**. The model does not remember your last message. When a
chat application appears to recall what you said five turns ago, it is because the
application stored those turns and sent the entire conversation again with the new
message appended. Nothing on the model's side persisted.

```{=typst}
#let turn-bar(n, history, new-bits) = {
  let unit = 0.62cm
  grid(
    columns: (1.5cm, auto),
    align: (right + horizon, left + horizon),
    text(size: 8.5pt, font: ("Segoe UI", "Calibri"), fill: rgb("#5A6269"))[Turn #n],
    {
      box(inset: (left: 6pt))[
        #stack(
          dir: ltr,
          ..range(history).map(_ => box(
            width: unit, height: 0.42cm,
            fill: rgb("#E4E7EA"), stroke: 0.5pt + rgb("#CDD2D7"),
          )),
          ..range(new-bits).map(_ => box(
            width: unit, height: 0.42cm,
            fill: rgb("#F0B323"), stroke: 0.5pt + rgb("#D49C14"),
          )),
        )
      ]
    },
  )
}

#block(width: 100%, inset: (y: 9pt))[
  #turn-bar(1, 0, 1)
  #v(3pt)
  #turn-bar(2, 1, 1)
  #v(3pt)
  #turn-bar(3, 2, 1)
  #v(3pt)
  #turn-bar(4, 3, 1)
  #v(3pt)
  #grid(
    columns: (1.5cm, auto),
    align: (right + horizon, left + horizon),
    text(size: 8.5pt, font: ("Segoe UI", "Calibri"), fill: rgb("#5A6269"))[Turn 12],
    box(inset: (left: 6pt))[
      #stack(
        dir: ltr,
        ..range(11).map(_ => box(width: 0.62cm, height: 0.42cm,
          fill: rgb("#E4E7EA"), stroke: 0.5pt + rgb("#CDD2D7"))),
        box(width: 0.62cm, height: 0.42cm,
          fill: rgb("#F0B323"), stroke: 0.5pt + rgb("#D49C14")),
      )
    ],
  )
  #v(9pt)
  #box(inset: (left: 1.5cm + 6pt))[
    #grid(
      columns: (auto, auto),
      column-gutter: 14pt,
      [
        #box(width: 0.42cm, height: 0.3cm, fill: rgb("#F0B323"),
             stroke: 0.5pt + rgb("#D49C14"))
        #h(4pt)
        #text(size: 8pt, font: ("Segoe UI", "Calibri"), fill: rgb("#5A6269"))[what you just typed]
      ],
      [
        #box(width: 0.42cm, height: 0.3cm, fill: rgb("#E4E7EA"),
             stroke: 0.5pt + rgb("#CDD2D7"))
        #h(4pt)
        #text(size: 8pt, font: ("Segoe UI", "Calibri"), fill: rgb("#5A6269"))[resent, and paid for, again]
      ],
    )
  ]
]
```

Each bar is one request. The amber block is the only new thing; everything grey is
conversation you already sent, being sent again because the model has no memory of it.
By turn twelve you are paying to retransmit eleven earlier turns to get one answer.

Three consequences follow, and they explain most of the surprises people hit:

**Cost grows with conversation length.** Turn 20 resends turns 1 through 19. A long
conversation isn't 20 cheap calls; it's 20 calls of steadily increasing size. Chat
applications get more expensive per message the longer the session runs.

**Latency grows too.** More input tokens means more to process before the first word of
output appears.

**Things fall out of the window.** When a conversation exceeds the limit, something must
go — usually the oldest turns. The model then genuinely does not know what was discussed
earlier, and will say so confidently or, worse, invent it. "The chatbot forgot" is not a
bug report; it's a design consequence someone didn't plan for.

This is also the honest answer to *"why can't we just put all 50,000 PDFs in the
prompt?"* Even if they fit, you would pay to resend them on every single question. That
economic reality — far more than any technical limit — is why retrieval exists, and why
Book II is largely about it.

### What a large context window actually buys you

Models now advertise context windows of 128,000 tokens and beyond — roughly a
300-page book in a single request. It is tempting to read that as "the limit problem is
solved." It isn't, for three reasons.

**You pay for all of it.** A 128K-token prompt costs what 128K tokens cost, on every
single call. Capacity is not permission.

**Attention isn't uniform.** Models reliably attend to the beginning and end of a long
context and are measurably weaker in the middle. Burying the one relevant paragraph
halfway through 100,000 tokens of filler is a good way to have it ignored. Retrieving
the right 2,000 tokens beats dumping 100,000 and hoping.

**Latency scales with input.** A very large prompt takes noticeably longer before the
first token appears, which users feel directly.

The practical reading: a large context window is a useful ceiling, not a strategy. It
means you rarely have to fight the limit — not that you should fill it.

## Temperature: the determinism dial

When the model produces each token, it computes a probability distribution over every
possible next token. **Temperature** controls how that distribution is sampled.

| Temperature | Behaviour | Use for |
|---|---|---|
| `0` | Near-deterministic — consistently picks the highest-probability token | Classification, extraction, routing, anything tested |
| `0.3–0.7` | Moderate variation | Drafting, summarizing, explanation |
| `1.0+` | High variation, more surprising | Brainstorming, creative generation |

The default in most SDKs is around `0.7`, which is a reasonable choice for chat and a
poor one for everything else you'll build at work.

**Set temperature to 0 for any task with a right answer.** Classifying a support email,
extracting an invoice total, deciding which function to call — these are not creative
tasks, and variation in them is pure downside. You also cannot meaningfully test a
system that returns something different each run.

One honest caveat: temperature `0` gives you *near*-deterministic, not guaranteed
identical, output. Floating-point behaviour, model updates and infrastructure changes
can all shift results. Don't build anything that depends on byte-identical responses
across time.

There is a second dial, **top_p**, which limits sampling to the most probable tokens
whose cumulative probability reaches a threshold. The practical advice is simple: adjust
one or the other, not both. Changing temperature is the convention; leave `top_p` at its
default until you have a specific reason.

## Embeddings: meaning as geometry

Everything above concerns generation. Embeddings solve a different problem: **how do you
find text that means the same thing, when it uses different words?**

Microsoft's definition is precise: an embedding is a dense representation of the
semantic meaning of a piece of text — a vector of floating-point numbers, such that the
distance between two embeddings correlates with the semantic similarity of the inputs.

In plain terms: the model converts text into a long list of numbers positioned in a
high-dimensional space, such that things that *mean* similar things end up near each
other. `person` and `human` land close together. `person` and `carburettor` do not.

That single property is what makes modern search work. Keyword search for "time off
policy" misses a document titled "Vacation Entitlement" — zero words in common. Vector
search finds it, because the two phrases sit close together in the space regardless of
sharing no vocabulary.

```{=typst}
#let dot(x, y, label, kind) = {
  let col = if kind == "query" { rgb("#26292B") } else if kind == "near" { rgb("#F0B323") } else { rgb("#AEB6BD") }
  place(top + left, dx: x, dy: y)[
    #box[
      #circle(radius: if kind == "query" { 4.5pt } else { 3.5pt }, fill: col,
              stroke: if kind == "query" { 1.5pt + white } else { none })
    ]
  ]
  place(top + left, dx: x + 0.42cm, dy: y - 0.07cm)[
    #text(size: 7.8pt, font: ("Segoe UI", "Calibri"),
          fill: if kind == "far" { rgb("#8A939C") } else { rgb("#26292B") },
          weight: if kind == "query" { "bold" } else { "regular" })[#label]
  ]
}

#block(width: 100%, inset: (y: 10pt))[
  #box(width: 100%, height: 6.2cm, fill: rgb("#FAFBFC"),
       stroke: 0.6pt + rgb("#E2E6E9"), radius: 2pt)[
    // the close cluster, ringed
    #place(top + left, dx: 1.7cm, dy: 0.9cm)[
      #circle(radius: 1.55cm, fill: rgb("#FDF3DC"), stroke: (dash: "dashed", paint: rgb("#E0B750"), thickness: 0.8pt))
    ]
    #dot(2.9cm, 1.55cm, [*"time off policy"* — the query], "query")
    #dot(2.3cm, 2.3cm, [Vacation Entitlement], "near")
    #dot(3.3cm, 2.9cm, [Annual Leave Guidance], "near")
    #dot(2.45cm, 3.05cm, [Paid Absence FAQ], "near")

    #dot(10.2cm, 1.1cm, [Fleet Carburettor Maintenance], "far")
    #dot(11.4cm, 4.6cm, [Loading Dock Access Hours], "far")
    #dot(8.6cm, 3.4cm, [Q3 Revenue Forecast], "far")

    #place(bottom + left, dx: 0.5cm, dy: -0.35cm)[
      #text(size: 7.5pt, font: ("Segoe UI", "Calibri"), style: "italic", fill: rgb("#8A939C"))[
        Nearby means similar in meaning — not similar in wording.
      ]
    ]
  ]
]
```

Nothing in the shaded cluster shares a word with the query. Keyword search returns none
of them; vector search returns all three, in that order.

### Cosine similarity

The standard measure of closeness is **cosine similarity** — the cosine of the angle
between two vectors. It ranges from -1 to 1, and higher means more similar.

Why the angle rather than straight-line distance? Because a long document and a short
query can be far apart by Euclidean distance purely because of length, while still
pointing in nearly the same direction. The angle captures *what the text is about*
independent of how much of it there is.

You will rarely implement this yourself — the vector database does it — but when you see
a relevance score of `0.87` in Book II, this is what produced it.

### The practical limits

Azure's current embedding models carry specific constraints worth knowing before you
design around them:

- **8,192 tokens** maximum per input. Longer documents must be split first — which is
  why *chunking* is a topic in Book II rather than an afterthought
- **2,048 inputs** maximum per batch request
- **300,000 tokens** aggregate across all inputs in one request. Exceed it and you get
  an HTTP 400, even when every individual input is within its own limit

That last one catches people batching for efficiency: each item is fine, the batch is
not. Split large batches.

Current models such as `text-embedding-3-small` also accept a **`dimensions`** parameter,
letting you request shorter vectors. Shorter vectors mean less storage and faster
search, at some cost to accuracy — a genuine trade-off worth testing rather than
guessing at.

## Where this meets Azure

A few Azure-specific details that cause disproportionate confusion on day one.

**The `model` parameter is your deployment name, not the model name.** In Azure OpenAI
you deploy a model and give that deployment a name of your choosing. API calls reference
*your* name, not Microsoft's. If you deployed `text-embedding-3-small` as
`embeddings-prod`, then `embeddings-prod` is what goes in the request. A 404 here almost
always means someone passed the model name.

**Authenticate with Microsoft Entra ID, not API keys.** Keys get copied into code,
committed and leaked. Entra ID gives you managed identity, proper RBAC and no secret to
rotate. The role you need on the resource is `Cognitive Services OpenAI User`. If you
get a 403, that role is usually the reason.

**Read the error codes literally** — they are more informative than they look:

| Code | Usually means |
|---|---|
| 400 | Request body, token count, or batch size |
| 401 | Not authenticated — token missing, expired, wrong audience |
| 403 | Authenticated, but missing the RBAC role |
| 404 | Wrong endpoint path or a deployment name that doesn't exist |
| 429 | Rate limited — you've exceeded tokens per minute. Back off and retry |

A note on the `429`: it is not an error in the ordinary sense. It is the expected
behaviour of a metered service under load, and any production code you write must handle
it with backoff and retry. Treating rate limits as exceptional is how a system works
perfectly in testing and fails on launch day.

## Streaming, and why responses arrive a word at a time

You've seen chat interfaces type their answer out progressively. That isn't a
presentation flourish — it reflects how generation actually works.

The model produces one token at a time. With a normal request, the API waits until the
entire response is finished and sends it in one piece. With **streaming**, each token is
sent as it's produced.

The total time is the same. What changes is **time to first token** — how long the user
stares at nothing. For a 400-word answer that takes eight seconds to generate, streaming
means something appears after a quarter of a second instead of after eight.

```python
stream = client.chat.completions.create(
    model=deployment, messages=messages, stream=True
)
for chunk in stream:
    if chunk.choices and chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="", flush=True)
```

Use streaming when a human is waiting and reading. Don't use it when your code is going
to parse the result — you'd be reassembling a complete response from fragments for no
benefit, and it complicates error handling considerably.

## Doing the cost arithmetic

Abstract token talk becomes concrete the moment you multiply it out. Work through a
realistic internal assistant:

- 200 staff, each asking 5 questions a working day
- Each question retrieves 4 document chunks of ~500 tokens as context
- System instructions: ~300 tokens
- Average answer: ~400 tokens

Per question:

```text
input  = 300 (system) + 2,000 (4 chunks) + 50 (question)  = 2,350 tokens
output = 400 tokens
```

Per day: 200 staff × 5 questions = 1,000 calls → **2.35M input tokens, 400K output.**
Over a 250-day working year, roughly **590M input and 100M output tokens.**

The exact dollar figure depends on the model and changes often, so look it up rather
than trusting a number printed in a book. The useful part is the *shape* of the answer,
and three things fall out of it immediately:

**Input dominates.** You're sending six times more tokens than you receive. Cost
optimization means attacking the input side — fewer chunks, smaller chunks, tighter
system instructions — not shortening answers.

**Retrieved context is the bulk of it.** Those four chunks are 85% of every request.
Retrieving three instead of four cuts total cost by about a fifth. This is why *how
many chunks to retrieve* is a cost decision as much as a quality one.

**Conversation history would change everything.** This calculation assumes each question
is independent. Make it a multi-turn conversation that resends history, and costs climb
with session length — which is why production assistants summarize or truncate older
turns rather than carrying them forever.

## Choosing a model

The instinct is to reach for the most capable model available. Resist it, and decide
deliberately instead:

| Factor | Favours a smaller model | Favours a frontier model |
|---|---|---|
| Task | Narrow, well-defined, repetitive | Open-ended, multi-step, ambiguous |
| Volume | High — cost multiplies | Low — cost is noise |
| Latency | User is waiting | Batch or background |
| Accuracy ceiling | "Good enough" is defined | Every point of quality matters |

A classifier sorting 10,000 support emails into eight categories should not run on a
frontier model. The task is narrow, the volume is high, and the accuracy difference on
a well-specified classification is often small. A system reasoning about an ambiguous
architecture question should.

The honest method is to test rather than reason about it: build your evaluation set
(chapter 4 covers this), run both, and look at the numbers. Model choice is one of the
few decisions in this field with a genuinely empirical answer, and it's routinely made
on vibes.

## What this changes about how you design

Pulling the four concepts together into decisions you'll actually make:

**Estimate cost on real data.** Token counts on your actual documents, not on a sample
sentence. Then multiply by expected volume *and* by conversation depth, because of
resending.

**Decide early what stays in the prompt.** Anything you send every turn gets paid for
every turn. System instructions are cheap; whole documents are not. This is the
calculation that makes retrieval the right answer rather than a fashionable one.

**Pin temperature per task, not per application.** Your classifier runs at 0. Your draft
generator runs at 0.5. They are different tasks and deserve different settings even
inside the same product.

**Budget for chunking.** The 8,192-token embedding limit means documents get split, and
*how* you split them determines whether retrieval works. It is the single most
consequential implementation detail in a RAG system, and we'll spend real time on it.

## Where this leaves you

You now understand what you're paying for, what constrains you, what makes output
reproducible, and what makes semantic search possible. That's the full set of levers on
a language model.

## Build this: the token audit

This is the most useful exercise in the book, and the one most likely to change how
people at work treat you.

Pick a real system at your workplace that someone has suggested "adding AI" to. A service
desk. A policy lookup. A document summarizer. Something that exists and has real volume.

Then answer, in writing, the question you will eventually be asked out loud: **what would
this cost to run for a year?**

### Part 1 — Measure, don't estimate

Take real data from that system. Not a sample sentence — real tickets, real policy text,
real email. Put it through a tokenizer.

| Measure | Your number |
|---|---|
| Characters in a typical input | |
| Tokens in that same input | |
| Ratio (tokens per character) | |
| Same ratio for a block of JSON or IDs | |

The gap between those two ratios is the point of the exercise.

### Part 2 — Build the arithmetic

Show your working:

```text
input tokens per request  = system prompt + retrieved context + user question
output tokens per request = typical answer length
requests per day          = users × requests each
annual                    = requests per day × 250 working days
```

Then state which component dominates, and what you would change first if asked to halve
the cost.

Most people discover the answer is "retrieved context," which is the finding that makes
chunking strategy in Book II feel urgent rather than academic.

### Part 3 — Test temperature yourself

Take one real task with a correct answer — classifying ten tickets, extracting a field
from ten documents. Run it three times at temperature 0, then three times at 1.0.

Record how many of the thirty results differ. Then answer in one sentence: **could you
have tested this system at 1.0?**

### Part 4 — The conversation multiplier

Rerun your Part 2 arithmetic assuming a multi-turn conversation that resends full
history, averaging six turns — the diagram earlier in this chapter, with real numbers
attached.

Compare the two annual figures. That gap is the cost of statelessness, and it is the
number most proposals omit entirely.

### Why this one matters disproportionately

Everything else in this book teaches you to build. This teaches you to answer the
question that decides whether anything you build gets approved.

It is also what distinguishes someone who has shipped an AI system from someone who has
followed a tutorial, in an interview. Tutorials never mention cost.

The next chapter is about the one input you control completely: the prompt. Having
understood the machine, we turn to instructing it — and to the harder question of how
you tell whether an instruction is actually any good.

---

### Chapter review

1. Why does a prompt full of employee IDs and GUIDs cost more than the same number of
   characters of English prose?
2. The context window is not memory. What is actually happening when a chatbot appears
   to remember earlier turns, and what does that cost?
3. A colleague asks why you can't simply put all 50,000 company PDFs into the prompt.
   Give the economic answer, not just the technical one.
4. You're building a system that classifies support emails into eight categories. What
   temperature, and why?
5. What does cosine similarity measure, and why is it preferred over straight-line
   distance for comparing documents?
6. Your embedding call returns HTTP 400 although every individual input is under 8,192
   tokens. What is the likely cause?
7. What is the difference between a model name and a deployment name in Azure OpenAI,
   and which error code usually signals confusing them?
