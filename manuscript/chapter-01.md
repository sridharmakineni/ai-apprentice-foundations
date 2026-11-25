# What AI Actually Is

There is a meeting that happens in every organization now. A vendor is presenting. The
slide says *AI-powered*. Someone from finance asks what that means, and the answer takes
ninety seconds and contains no information.

Afterwards your director turns to you, because you are the technical person in the room,
and asks the question they didn't want to ask in front of the vendor: *is that actually
AI?*

This chapter exists so you can answer that. Not with a definition recited from a
training course, but with the kind of answer that changes what your organization buys.

## The word means too much

"Artificial intelligence" has described, at various points over seventy years: a program
that plays checkers, a system that diagnoses infections from a decision tree, a spam
filter, a chess engine, a recommendation algorithm, and a model that writes a competent
cover letter. These things have almost nothing in common mechanically.

That breadth is not a problem to be solved by finding a better definition. It is a
permanent feature of the term, and the practical response is to stop treating "is it
AI?" as the interesting question. It isn't. The interesting questions are narrower:

- What is this system actually doing?
- Did a person write its rules, or did it derive them from data?
- What happens when it's wrong, and who finds out?

Most of this chapter is about being able to ask those three questions precisely.

## The nesting

Four terms get used interchangeably and shouldn't be. They nest, and the nesting is
literal rather than decorative.

```{=typst}
#let ring(label, note, depth) = {
  let shades = (rgb("#26292B"), rgb("#3D4348"), rgb("#5A6269"), rgb("#8A6410"), rgb("#F0B323"))
  let fills  = (rgb("#F4F5F6"), rgb("#EAECEE"), rgb("#E0E3E6"), rgb("#FCF3DE"), rgb("#F8E4B0"))
  block(
    width: 100% - depth * 1.5cm,
    inset: (x: 12pt, y: 9pt),
    fill: fills.at(depth),
    stroke: (left: 3pt + shades.at(depth)),
    radius: 2pt,
  )[
    #set par(justify: false, leading: 0.5em)
    #text(size: 9.5pt, weight: "bold", font: ("Segoe UI", "Calibri"),
          fill: rgb("#111827"))[#label]
    #h(0.5em)
    #text(size: 8.5pt, fill: rgb("#5A6269"), font: ("Segoe UI", "Calibri"))[#note]
    #v(0.45em)
  ]
}

#block(width: 100%, inset: (y: 8pt))[
  #ring("Artificial Intelligence", "any system doing things that need intelligence", 0)
  #pad(left: 1.5cm, ring("Machine Learning", "learns its rules from data", 1))
  #pad(left: 3.0cm, ring("Deep Learning", "neural networks, many layers", 2))
  #pad(left: 4.5cm, ring("Generative AI", "produces content, not labels", 3))
  #pad(left: 6.0cm, ring("Large Language Models", "over text", 4))
]
```

**Artificial intelligence** is the widest ring: any system performing tasks that
normally need human intelligence. A hand-coded chess engine from 1985 qualifies. So does
a decision tree an expert wrote out by hand.

**Machine learning** is the subset where the system derives its own rules from data
rather than having them written. Nobody tells a spam classifier that the word "invoice"
in a subject line from an unknown sender is suspicious. It infers that from a few
million examples.

**Deep learning** is machine learning using neural networks with many layers. The depth
is the point: each layer builds features from the previous layer's output, so early
layers learn simple patterns and later ones learn combinations of them. This is why deep
learning handles messy, unstructured input — images, audio, language — where older
techniques needed a human to engineer the features first.

**Generative AI** is deep learning that produces new content rather than a label or a
number. The distinction from everything above it is the shape of the output, not the
size of the model.

**Large language models** are generative models over text. This is what you mean when
you say "AI" in 2026, and it is four rings deep in a structure most people treat as one.

Memorize this. You will use it in meetings constantly, usually to tell someone that the
thing they are worried about and the thing in front of them are at different levels.

## The distinction that matters most at work

Here is the one that will come up this month.

> **Automation follows rules a person wrote. Machine learning derives the rules from
> data.**

A Power Automate flow that routes expense claims over $5,000 to a director is
automation. It is useful, it is reliable, it is doing real work — and there is no
intelligence in it anywhere. Somebody chose $5,000 and typed it into a condition.

A system that examines two years of expense claims, learns which ones were queried by
finance, and flags new claims resembling those is machine learning. Nobody picked the
threshold. There may not *be* a threshold — the learned rule might be a combination of
amount, vendor, submitter, day of month and category that no human would have written.

This distinction has consequences you will care about:

| | Automation | Machine learning |
|---|---|---|
| **Why did it do that?** | Read the rule | Sometimes genuinely hard to say |
| **Changing behaviour** | Edit the rule | Retrain, or adjust data |
| **When it's wrong** | The rule was wrong | The data, the model or the task was wrong |
| **Auditability** | Trivial | A discipline of its own |
| **Needs data?** | No | Yes, and a lot |

Neither is better. A great deal of what gets sold as AI is automation, and much of it is
automation that *should* be automation — if you can write the rule, writing the rule is
cheaper, faster, auditable and doesn't drift. The failure is not using automation. The
failure is paying for a model you didn't need, or using a rule where the real pattern is
too complex to write down.

If you already build Power Platform solutions, you have been shipping the left-hand
column for years. That is not a lesser skill. It means you already know something most
people entering AI don't: that the hard part of automating a process is understanding
the process.

## What changed

Machine learning has been in production since the 1990s. Your bank has used it on fraud
for decades. So why did everything feel different from late 2022?

Two shifts.

**The output changed shape.** Classical ML mostly produced decisions: a category, a
score, a number. Fraud or not. Churn risk 0.73. Those are valuable but narrow, and each
model did exactly one thing. Generative models produce *content* — language, code,
images. The output space is open-ended.

**The interface changed.** Previous ML needed a data scientist to frame the problem,
engineer features, train a model and deploy it, per task. Generative models take
instructions in English. The barrier between having an idea and testing it collapsed
from months to minutes.

That second shift is why a business analyst can now do things that required a specialist
team in 2019, and it's the reason this book can exist at all.

## How a language model actually works

You can build a great deal without this. But you cannot debug, cost or trust what you
build without it, so here it is honestly.

A language model predicts the next chunk of text, given everything before it. That's it.
That is the whole mechanism.

Text is first broken into **tokens** — pieces roughly three-quarters the length of an
average English word. "Understanding" might be two tokens; "the" is one; an unusual
surname might be four. The model reads tokens and emits a probability distribution over
every possible next token. It picks one, appends it, and repeats.

Everything that looks like reasoning emerges from that loop, scaled to a vast number of
parameters trained on a vast amount of text. When a model produces a correct, structured
argument, it is not retrieving an argument. It is producing the sequence of tokens that
a correct argument would most plausibly consist of.

This is not a reason to be dismissive — the results are genuinely remarkable, and
"merely predicting tokens" undersells what emerges at scale. But it explains the
failures precisely, and the failures are what you are paid to anticipate.

## Why it makes things up

A model asked about a benefit policy that does not exist will often produce one. Fluent,
plausible, specific, and wrong. This is **hallucination**, and the single most useful
thing to understand about it is that it is not a defect.

The model is optimizing for plausible continuation. It is not, at any point, consulting
a store of verified facts and reporting what it finds. Fluency and factuality are
different objectives, and the model was trained on the first one.

So when the model invents a policy, it is working exactly as designed. There is no bug
to fix, no patch coming, no version that eliminates it. There are only techniques for
*grounding* the model — giving it the real documents at the moment of asking, and
requiring it to cite them. That is what retrieval-augmented generation is for, and it is
Chapter 8 of this series.

Say this plainly to stakeholders. The instinct is to soften it, and softening it sets up
the much worse conversation that happens six months later, after someone has acted on a
confident invention.

## What it's good at, and what it isn't

**Good at:**

- Transforming text that already exists — summarizing, reformatting, changing register
- Producing first drafts a human will edit
- Extracting structure from unstructured input
- Explaining things in another vocabulary
- Writing code of the kind that has been written many times before

**Bad at:**

- Exact arithmetic over structured data
- Anything requiring a guarantee
- Knowing what it doesn't know
- Reliable recall of specific facts without being given them
- Reasoning about events after its training cutoff

Notice the shape. Generative models are strong where the input is messy and the output
tolerates variation, and weak where the input is clean and the output must be exact.
That is almost precisely the inverse of a database, which is why the two are
complementary and why most good AI systems are a model *plus* deterministic code rather
than a model alone.

The practical rule, which will save you from most bad architectures:

> Use the model to decide **what** to compute. Use code to compute it.

A payroll assistant should not calculate a deduction. It should work out which
calculation the employee is asking about, call the function that performs it correctly,
and explain the result.

## Agents, and why the word is slippery

An **agent** is a system that can reason about a goal, use tools to act, observe what
happened, and decide what to do next. The defining feature is the loop, not the model.

```{=typst}
#let stage(label) = block(
  width: 3.1cm, height: 1.5cm,
  fill: rgb("#F4F5F6"), stroke: 0.8pt + rgb("#C9CFD4"), radius: 3pt,
  inset: 7pt,
)[
  #set align(center + horizon)
  #set par(justify: false, leading: 0.45em)
  #text(size: 9pt, weight: "bold", font: ("Segoe UI", "Calibri"), fill: rgb("#26292B"))[#label]
]

#let arrow = {
  set align(horizon)
  text(size: 15pt, fill: rgb("#F0B323"), weight: "bold")[→]
}

#block(width: 100%, inset: (y: 10pt))[
  #set align(center)
  #grid(
    columns: (auto, 0.55cm, auto, 0.55cm, auto, 0.55cm, auto),
    align: horizon,
    stage("Observe"), arrow,
    stage("Reason"), arrow,
    stage("Act\nvia a tool"), arrow,
    stage("Observe\nthe result"),
  )
  #v(2pt)
  // the loop back, drawn as a bracket under the row
  #box(width: 15.3cm, height: 0.95cm)[
    #place(top + left, line(start: (0.35cm, 0cm), end: (0.35cm, 0.6cm),
                            stroke: 1pt + rgb("#C9CFD4")))
    #place(top + left, line(start: (0.35cm, 0.6cm), end: (14.95cm, 0.6cm),
                            stroke: 1pt + rgb("#C9CFD4")))
    #place(top + left, line(start: (14.95cm, 0.6cm), end: (14.95cm, 0cm),
                            stroke: 1pt + rgb("#C9CFD4")))
    #place(top + center, dy: 0.62cm)[
      #box(fill: white, inset: (x: 6pt))[
        #text(size: 8.5pt, font: ("Segoe UI", "Calibri"), fill: rgb("#5A6269"))[
          goal not met yet — go round again
        ]
      ]
    ]
  ]
]
```

A chatbot answers and stops. An agent answers, notices it needs the current project
status, calls an API, reads the response, finds the sponsor's email address, drafts a
summary, and sends it.

The word is slippery because it is applied to everything from a single function call to
genuinely autonomous multi-step systems, and the risk profile across that range is
enormous. The question worth asking is never "is it an agent?" but:

- **What can it actually do?** Which tools, with which permissions?
- **What happens when it's wrong?** Draft an email, or send one?
- **Where is the human?** Approving each action, or reviewing afterwards, or nowhere?

An agent that can only read is a convenience. An agent that can write to production
systems is an entirely different proposition, and the distinction deserves more
attention than it usually gets. Book III of this series is about building these
properly.

## Training, inference, and the question everyone gets wrong

Two phases, constantly conflated, and the conflation produces badly wrong decisions.

**Training** builds the model. It happens once, costs an enormous amount, and ends with
a fixed set of weights.

**Inference** is using the finished model. It happens every time anyone sends a prompt.
It is cheap individually and expensive in aggregate.

Which brings us to the objection you will hear within a week of proposing anything:
*"everything we type gets used to train it."*

Half right, and the half that's wrong matters. **Inference does not update the model.**
Sending a prompt does not teach the model anything; weights are frozen. Your data cannot
"leak into other people's answers" through the mechanism people imagine.

But whether your prompts are *retained, logged, reviewed by humans or used in some
future training run* is a completely separate question — one of contract, configuration
and vendor policy, not of how transformers work. It varies enormously between a
consumer chat product and an enterprise deployment under your own tenancy.

So the colleague's instinct is right and their reasoning is wrong, which is the most
awkward kind of disagreement to resolve. The answer is not "don't worry about it." It is
"that specific fear is unfounded, and here is the real question we need answered before
we approve this tool" — then go and read the data handling terms for the specific
deployment you are proposing, under your organization's own classification rules.

## Where the Microsoft pieces fit

If you work in a Microsoft shop, you have by now been shown four or five products with
overlapping names and no clear explanation of how they relate. Here is the map, because
every later chapter assumes it.

**Microsoft 365 Copilot** is a finished product. It sits inside Word, Outlook, Teams and
Excel, grounded in your organization's own tenant content through Microsoft Graph. You
configure it; you don't build it. Its value and its limits both come from that grounding
— it knows about your documents and nothing about your line-of-business database.

**Copilot Studio** is the low-code layer for building your own conversational agents. If
your background is Power Platform, this is the nearest thing to familiar ground, and it
is a legitimate production choice for a well-scoped internal assistant. Its ceiling is
real but higher than most developers assume.

**Azure AI Foundry** is the developer platform: model catalogue, deployments,
evaluation, agent hosting, content safety. This is where you go when you are building
rather than configuring, and it is the subject of Book III.

**Azure OpenAI** is the model endpoint itself, now reached through Foundry. When you
write code that calls a model, this is usually what you're calling.

**AI Builder** is the Power Platform's own in-house ML — form processing, prediction,
object detection — aimed at makers rather than engineers.

The useful way to hold these is as a ladder of control versus effort:

| Layer | You supply | Control | Effort |
|---|---|---|---|
| M365 Copilot | Configuration | Low | Hours |
| AI Builder | Training examples | Low–medium | Days |
| Copilot Studio | Topics, actions, knowledge | Medium | Days–weeks |
| Foundry / Azure OpenAI | Code, architecture, evaluation | High | Weeks |

Most organizations overshoot. They reach for the bottom row because it sounds serious,
then spend four months rebuilding something the third row would have done in two weeks.
The discipline worth developing early is reaching for the *highest* row that actually
satisfies the requirement, and being able to explain why the rows above it don't.

You will learn the bottom row in this series, because that is where the engineering
careers are. But knowing when not to use it is the part that makes you useful.

## Five questions for the next vendor meeting

Everything above compresses into this. When someone says AI-powered:

1. **Is it learning from data, or following rules someone wrote?** Both are fine. Only
   one justifies the price difference.
2. **Where do the answers come from?** A model's training data, or your documents
   retrieved at query time? Only the second can be verified.
3. **What happens when it's wrong?** Not *if*. Ask what the failure looks like and who
   notices.
4. **What can it do on its own?** Read, or write? To which systems?
5. **Where does our data go, under what terms, and for how long?**

Any vendor who can answer these five clearly is worth continuing with. Any vendor who
deflects to capability demos has told you something useful.

## Where this leaves you

You now have the vocabulary that the rest of this book assumes. More usefully, you have
a way of interrogating claims that doesn't depend on knowing the implementation — which
is exactly the position you're in during most real conversations.

The next chapter puts that vocabulary to work in Python, because from here on the way to
understand these systems is to build small ones badly and watch what happens.

## Build this: the "What is AI?" deck

**Ten slides, for a room of colleagues in your own organization.** IT staff, business
analysts, a couple of managers. Not stupid, not technical, and several of them have been
told by a vendor this year that something is "AI-powered" and privately suspect it isn't.

Your job is not to impress them. It is to leave them able to ask better questions.

### Why a deck and not an essay

Because you cannot hide in slides. An essay lets you write four paragraphs that sound
informed around a concept you only half-grasp. Ten slides force you to compress each idea
to its load-bearing sentence, and compression is where you find out what you actually
know.

You will also do exactly this at work within six months.

### The slides

| # | Slide | The test it sets |
|---|---|---|
| 1 | Title + the question the deck answers | Can you state the point in one line? |
| 2 | The nesting — AI → ML → DL → GenAI | Can you draw it from memory? |
| 3 | Automation vs machine learning, with a real example from your workplace | Do you see the distinction in the wild? |
| 4 | What generative AI actually does | Can you say it without saying "it understands"? |
| 5 | Three things it does well | Grounded in your own job, not generic |
| 6 | Three things it does badly | Harder and more valuable than slide 5 |
| 7 | Hallucination — what and why | Explain the cause, not just the symptom |
| 8 | Agent vs chatbot | The loop, not the model size |
| 9 | Five questions to ask any AI vendor | Your actual contribution |
| 10 | Where this is going in our organization | An opinion, held honestly |

### Rules

**No slide may contain the phrase "AI understands."** It doesn't, and the shortcut will
cost you clarity later when you are debugging why it got something wrong.

**Slide 6 must be specific.** "It makes mistakes" is not an answer. "It cannot be trusted
to compute payroll deductions, because token prediction is not arithmetic" is.

**Slide 9 is yours.** Don't copy the list from this chapter. Write the five questions you
would actually ask, having read it.

Every claim should survive someone asking *how do you know that?*

### If you want to push further

Present it to one actual human being. The questions they ask will show you which slides
are weak far faster than re-reading them will.

---

### Chapter review

1. Place AI, ML, deep learning and generative AI in their nesting, and give an example
   of something that is AI but not ML.
2. A system flags unusual purchase orders. What single question tells you whether it's
   automation or machine learning?
3. Explain to a non-technical manager why hallucination cannot be patched out.
4. Why should a language model not calculate a payroll deduction? What should it do
   instead?
5. A colleague says hosted LLMs train on company prompts. Where are they right and where
   are they wrong?
6. What distinguishes an agent from a chatbot, and which follow-up question matters more
   than the label?
