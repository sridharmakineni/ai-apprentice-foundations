# Before You Start {-}

## Who this book is for

You work in IT. You may build Power Platform solutions, administer Microsoft 365, write
PowerShell, analyse business processes, or manage the people who do. You have watched AI
go from a curiosity to something your organization is buying, and you have decided that
understanding it properly is now part of your job.

You are not a data scientist and do not intend to become one. You do not need calculus,
and there is none in here.

What you do have is the thing most people entering AI lack: years of experience with how
work actually happens inside an organization — who approves what, where the data really
lives, which process everyone works around. That knowledge is most of what makes an AI
system useful rather than impressive, and it is why you are better positioned than you
probably think.

## What you will be able to do

By the end of this book you can:

- Tell whether a system is genuinely learning from data or following rules someone wrote,
  and explain the difference to a director
- Read any AI code sample and know what it does
- Estimate what an AI feature will cost to run for a year, from real numbers
- Choose a model and a temperature for a given task, and defend both
- Write a prompt that produces output your code can parse
- **Tell whether a prompt actually works** — which is rarer than it sounds, and the
  single most valuable skill in here

That list is deliberately short. Four chapters cannot make you an AI engineer, and a
book that claims otherwise is selling something.

## How to use it

Each chapter ends with review questions, and each has a project. **The projects are the
point.** The chapters are scaffolding around them.

This is worth being blunt about, because it is where most readers of most technical books
go wrong. Everything in these four chapters is available free, in better-organized form,
on Microsoft Learn. What is not available anywhere is the judgment that comes from having
been wrong about this material in a specific, memorable way — from shipping a prompt that
worked in testing and failed on real tickets, and understanding why.

Read a chapter. Build its project badly. Fix it. The bugs are the lesson.

## What this book does not cover

- **Mathematics.** No linear algebra, no gradient descent, no transformer internals
  beyond what changes a decision you will make.
- **Training models.** You will almost certainly never train one. This book is about
  using and grounding models that already exist.
- **Non-Microsoft stacks.** The concepts are universal; the specifics lean Azure, because
  that is where the readers of this book work.
- **Anything about building agents.** That is Book III, and it assumes everything here.

## A note on accuracy

Technical books rot. This one has specific defences:

Every factual Python claim in chapter 2 is executed against a real interpreter as part of
the build — if a claim stops being true, the build fails. The Azure figures in chapter 3
come from Microsoft's own documentation rather than from memory, and are dated. Where a
claim is contested rather than settled, it says so.

Where a number is likely to move — model pricing, context window sizes, rate limits —
the book tells you to look it up rather than printing a figure that will mislead you in
six months.

## Conventions

Two colleagues appear throughout the examples, so you are never decoding an unfamiliar
scenario and an unfamiliar concept at once.

**Bruno** works in IT Operations. He files the support tickets you will classify, and he
types in a hurry. **Willie** works in HR. She asks the policy questions an assistant has
to answer correctly, and she will notice immediately if it invents something.

Code, output and diagrams appear in boxes. Anything in a tinted panel is a point worth
remembering rather than an aside.
