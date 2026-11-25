# Python for AI

You do not need to become a Python developer. You need to become fluent in about
fifteen percent of the language, because that fifteen percent is what every AI tutorial,
SDK and code sample assumes you already know.

This chapter is that fifteen percent. It is deliberately not a Python course — there are
excellent ones and this isn't trying to be another. It is the specific subset that stops
you being stuck, chosen by working backwards from what you will actually meet in Books II
and III.

If you already write PowerShell or build Power Platform solutions, you are much closer
to this than you think. You've been doing the hard part — modelling a process, handling
the cases where it goes wrong — for years. What follows is mostly new syntax for ideas
you already hold.

## Why Python, honestly

Not because it's elegant. Not because it's fast — it isn't, particularly.

Python won AI for a boring reason: the libraries are there. Every model provider ships a
Python SDK first. Every paper's reference implementation is Python. Every framework you
will meet in Book III — LangGraph, CrewAI, the MCP SDK — is Python-first. Choosing
anything else means doing translation work forever for no benefit.

There are good C# and TypeScript SDKs, and if your organization is a .NET shop you will
likely ship production services in C#. Learn Python anyway. It's the language the
documentation is written in, and being unable to read the documentation is a much worse
problem than being unable to write the service.

## What you already know, renamed

| You know | Python calls it | Looks like |
|---|---|---|
| `$variable` | variable | `name = "Sridhar"` |
| Array / collection | list | `teams = ["IT", "HR"]` |
| Hashtable / `@{}` | dictionary | `person = {"name": "Ada", "dept": "IT"}` |
| Function / cmdlet | function | `def greet(name):` |
| `ForEach-Object` | `for` loop | `for t in teams:` |
| Module / import | import | `import json` |
| `try/catch` | `try/except` | `except ValueError:` |

The conceptual distance is small. The friction is syntactic, and syntax stops being
friction after about a week of actually typing it.

Two differences that catch PowerShell people specifically:

**Indentation is syntax.** Python has no braces. The indentation *is* the block
structure, and getting it wrong is a syntax error rather than a style complaint. Use an
editor that inserts four spaces for tab and this stops mattering within a day.

**Everything is case-sensitive.** `Name` and `name` are different variables. PowerShell
forgives this; Python does not.

## The four shapes of data

Almost every bug you hit in your first month will be a confusion between these.

```python
employees = ["Bruno", "Willie", "Ada"]        # list    — ordered, changeable
person    = {"name": "Bruno", "dept": "IT"}   # dict    — key/value lookup
location  = ("Toronto", "ON")                 # tuple   — ordered, fixed
dept_ids  = {"IT", "HR", "IT"}                # set     — unique, unordered
```

**Lists** are sequences. You index them from zero, loop over them, append to them. If
you'd use an array, use a list.

**Dictionaries** are the important one. A dict maps keys to values, and it's the shape
that JSON becomes when Python reads it. Every API response you ever handle will arrive
as a dict, nested inside other dicts, with lists in between. Being fluent with dicts is
most of what "being able to work with APIs" actually means.

```python
response = {
    "choices": [
        {"message": {"role": "assistant", "content": "Hello."}}
    ],
    "usage": {"prompt_tokens": 12, "completion_tokens": 3}
}

reply  = response["choices"][0]["message"]["content"]   # "Hello."
tokens = response["usage"]["prompt_tokens"]             # 12
```

Read that second-to-last line slowly, because you will write it a thousand times. Go
into `choices`, take the first item, go into `message`, take `content`. That shape — a
list of choices, each with a message — is what nearly every chat completion API returns.

**Tuples** are lists that can't be changed after creation. You'll meet them as return
values more than you'll create them.

**Sets** hold unique items and discard duplicates. `dept_ids` above contains two
entries, not three. Useful for deduplication, rare otherwise.

### The safer way to read a dict

```python
person["email"]              # KeyError if 'email' is missing — crashes
person.get("email")          # None if missing — keeps going
person.get("email", "n/a")   # "n/a" if missing — keeps going, with a default
```

Use `.get()` when reading anything that came from outside your program. API responses
change shape between versions, optional fields are genuinely optional, and a crash in
production at 4pm because a field was absent is an avoidable kind of embarrassment.

## Functions

```python
def build_prompt(role, task, tone="professional"):
    """Assemble a prompt from its parts."""
    return f"You are a {role}. {task} Respond in a {tone} tone."

prompt = build_prompt("recruiter", "Summarize this CV.")
```

Three things worth noticing:

**`f"..."` is an f-string** — anything in braces gets evaluated and inserted. It's the
cleanest way to build strings and you'll use it constantly.

**`tone` has a default**, so callers can omit it. Defaults are how Python functions stay
usable as they grow parameters.

**The triple-quoted string** is a docstring. Write them. In six weeks you will not
remember what `build_prompt` was for, and neither will the coding assistant you ask
about it.

## Classes, briefly

You will *read* far more classes than you write. Most AI SDKs hand you an object and
expect you to call methods on it:

```python
from openai import AzureOpenAI

client = AzureOpenAI(azure_endpoint=endpoint, api_key=key, api_version="2024-10-21")
response = client.chat.completions.create(model=deployment, messages=messages)
```

`AzureOpenAI` is a class. `client` is an instance of it. `chat.completions.create` is a
method reached by walking attributes. You don't need to be able to write that class. You
need to recognise the pattern — construct a client once, call methods on it many times —
because every SDK in this book follows it.

## JSON is the common tongue

JSON — JavaScript Object Notation — is how systems exchange structured data, and it is
the format of essentially every AI API. Python's `json` module converts between the two
worlds:

```python
import json

text = '{"name": "Bruno", "skills": ["Python", "SQL"]}'

data = json.loads(text)        # JSON string  -> Python dict
back = json.dumps(data)        # Python dict  -> JSON string
pretty = json.dumps(data, indent=2)   # ...formatted for humans
```

`loads` reads, `dumps` writes — the `s` means "string". The mapping is almost
one-to-one: JSON objects become dicts, arrays become lists, `true` becomes `True`,
`null` becomes `None`.

That last row is a genuine trap. JSON says `null`, Python says `None`, and the error
message when you confuse them is unhelpful.

## Calling an API

Here is the whole shape of it, with nothing hidden:

```python
import os
import requests

response = requests.get(
    "https://api.example.com/employees",
    headers={"Authorization": f"Bearer {os.environ['API_TOKEN']}"},
    params={"department": "IT"},
    timeout=30,
)
response.raise_for_status()
employees = response.json()

for person in employees:
    print(person.get("name", "unknown"))
```

Four things that matter more than they look:

**`os.environ`** reads the token from the environment, not from the file. A credential
written into source code gets committed, gets pushed, and gets found. This is not a
style preference — it is the single most common way small projects leak secrets, and
your repo is public.

**`timeout=30`** stops a hung request blocking your program forever. Omit it and you
will eventually have a process that never returns and no idea why.

**`raise_for_status()`** turns an HTTP error into a Python exception. Without it, a 401
response sails past silently and you debug the wrong thing for twenty minutes.

**`.json()`** parses the response body into dicts and lists — the shapes from earlier.

## Virtual environments, and why your machine will break without them

Every Python project needs its own isolated set of libraries. Skip this and you will
eventually have two projects needing incompatible versions of the same package, and
fixing one breaks the other.

```bash
python -m venv .venv              # create the environment
.venv\Scripts\activate            # activate it (Windows)
pip install requests openai       # install into it
pip freeze > requirements.txt     # record exactly what you installed
```

`requirements.txt` is what makes your work reproducible — on CI, on a colleague's
machine, on the server. Commit it. Never commit `.venv`.

## Errors are data, not accidents

```python
try:
    response = client.chat.completions.create(model=deployment, messages=messages)
except RateLimitError:
    time.sleep(20)
    # retry
except APIError as err:
    log.error("Model call failed: %s", err)
    return "Something went wrong. Reference: %s" % correlation_id
```

This matters more in AI work than in ordinary scripting, for a reason worth internalising
early: **you are calling a remote, metered, occasionally slow service that can refuse
you.** Rate limits, timeouts, content filters and transient failures are normal operating
conditions, not exceptional ones. Code that assumes the happy path works beautifully in
a notebook and falls over the first week it meets real users.

Note also what the user sees — a generic message and a correlation ID, not the raw
exception. The detail goes in your logs where you can find it; the stack trace does not
go on someone's screen.

## Comprehensions, because you can't read code without them

A list comprehension builds a list in one line. Python programmers use them constantly,
which means every code sample you read will contain them.

```python
names = [person["name"] for person in employees]
```

Read it right to left: *for each person in employees, take their name, collect into a
list.* The long form is four lines:

```python
names = []
for person in employees:
    names.append(person["name"])
```

Identical result. Add a condition to filter:

```python
it_staff = [p["name"] for p in employees if p["dept"] == "IT"]
```

And dictionaries work the same way:

```python
by_name = {p["name"]: p for p in employees}      # name -> whole record
lookup  = by_name["Ada"]["dept"]                  # instant lookup, no looping
```

That second pattern — turning a list into a dict keyed by something — is worth
remembering. Searching a list means scanning every item; a dict lookup is instant. When
you're matching thousands of records, it's the difference between a script that takes a
minute and one that takes an hour.

Don't write clever ones. A comprehension spanning three lines with two conditions is
harder to read than the loop it replaced. Use them for the simple cases, which is most
cases.

## Type hints: you'll read them, not write them

Modern Python code is annotated with types:

```python
def search_employees(query: str, limit: int = 10) -> list[dict]:
    ...
```

This says `query` is a string, `limit` is an integer defaulting to 10, and the function
returns a list of dicts. Python does **not** enforce any of this at runtime — it is
documentation that editors and type checkers can read.

It matters to you for one reason: every AI SDK is annotated this way, and the
annotations are the fastest way to understand an unfamiliar function. When you hover
over a method and your editor tells you it returns `ChatCompletion`, you've learned more
in a second than the docs would have told you in a minute.

Write them in your own code when it's easy. Don't agonize over them.

## Reading files

You'll do a lot of this — documents in, text out.

```python
from pathlib import Path

text = Path("policy.txt").read_text(encoding="utf-8")

with open("employees.json", encoding="utf-8") as f:
    employees = json.load(f)        # note: load, not loads — this reads a file
```

**Always specify `encoding="utf-8"`.** On Windows, Python historically defaulted to a
regional encoding, and the result is a `UnicodeDecodeError` on the first document
containing an accented name, em-dash or curly quote — which, in a Canadian organization,
is approximately the first document.

The `with` block closes the file automatically, even if something fails partway. Use it.

Note the pairing that catches everyone exactly once: `json.load()` takes a **file**,
`json.loads()` takes a **string**. Same for `dump` and `dumps`.

## Errors you will actually see

| Error | What it means | Usual cause |
|---|---|---|
| `KeyError: 'email'` | Dict has no such key | API response shape differs from expectation — use `.get()` |
| `TypeError: 'NoneType' object is not subscriptable` | You indexed into `None` | Something returned nothing and you didn't check |
| `IndentationError` | Block structure is wrong | Mixed tabs and spaces |
| `ModuleNotFoundError` | Library isn't installed | Wrong virtual environment, or you forgot `pip install` |
| `JSONDecodeError` | Text isn't valid JSON | The model returned prose around the JSON — see chapter 4 |
| `UnicodeDecodeError` | Wrong file encoding | Missing `encoding="utf-8"` |

The second row deserves attention, because it's the most common and least obvious. When
a function returns `None` and you treat it as a dict, the error appears at the point you
*used* the value — not where it went wrong. Debugging means tracing backwards to find
what returned nothing, which is why checking return values early saves time later.

The last row will become familiar in chapter 4, and it's not really a Python problem:
it's what happens when a model wraps its JSON in a helpful sentence.

## Build this

Build an **Employee Directory Search**: load a list of people from a JSON file, search
by name, filter by department, export the results.

It is deliberately unglamorous and there is no AI in it. That's the point — it exercises
dicts, lists, functions, file handling and JSON, which are precisely the things that
will otherwise block you in Book II when you're trying to understand why your chatbot
can't read its own configuration.

Your version should:

- Load `employees.json` into a list of dicts
- Search by partial name, case-insensitively
- List unique departments (a set earns its keep here)
- Export filtered results back to JSON
- Fail gracefully when the file is missing or malformed

That last requirement is the one that teaches you something.

## A worked example, end to end

Throughout this series the examples use the same two colleagues, so you are never
decoding an unfamiliar scenario and an unfamiliar concept at the same time.

**Bruno** works in IT Operations. He files the support tickets you'll classify in
chapter 4, and he types in a hurry — his tickets are terse, ambiguous and realistic.

**Willie** works in HR. She asks the policy questions the assistant in Book II has to
answer correctly, and she will notice immediately if it invents something.

Two recurring users is a small thing that pays off repeatedly. When chapter 4 asks you
to classify *"laptop won't boot after the password reset"*, you already know that's
Bruno, and your attention goes to the classification problem rather than to parsing a
new scenario. The same trick works in your own documentation.

Here they are as data:

```json
[
  {"name": "Bruno",  "dept": "IT Operations", "email": "bruno@example.ca",  "location": "Toronto"},
  {"name": "Willie", "dept": "Human Resources", "email": "willie@example.ca", "location": "Guelph"}
]
```

Now the whole project in one pass:

```python
import json
from pathlib import Path


def load_employees(path: str) -> list[dict]:
    """Read the directory, returning an empty list if anything is wrong."""
    try:
        return json.loads(Path(path).read_text(encoding="utf-8"))
    except FileNotFoundError:
        print(f"No directory file at {path}")
        return []
    except json.JSONDecodeError as err:
        print(f"{path} is not valid JSON: {err}")
        return []


def search(employees: list[dict], query: str) -> list[dict]:
    """Case-insensitive partial match on name."""
    q = query.lower()
    return [p for p in employees if q in p.get("name", "").lower()]


def departments(employees: list[dict]) -> list[str]:
    """Unique department names, alphabetically."""
    return sorted({p["dept"] for p in employees if "dept" in p})


def export(results: list[dict], path: str) -> None:
    Path(path).write_text(json.dumps(results, indent=2), encoding="utf-8")


people = load_employees("employees.json")
print(departments(people))              # ['Human Resources', 'IT Operations']
print(search(people, "bru"))            # Bruno's record
export(search(people, "bru"), "out.json")
```

Forty lines, and every idea from this chapter is in it: dicts and lists, `.get()` with a
default, a comprehension, a set for uniqueness, f-strings, type hints, file handling with
explicit encoding, and error handling that fails usefully instead of crashing.

Read it until each line is obvious. That is genuinely the bar for this chapter — not
writing it from memory, but looking at it and finding nothing mysterious.

A few choices in there are worth defending, because they're habits rather than
decoration.

**`load_employees` returns an empty list on failure rather than crashing or returning
`None`.** The caller can loop over an empty list safely; it would crash looping over
`None`. Returning the same *type* on both the success and failure paths removes an
entire class of downstream bug — the `'NoneType' object is not subscriptable` from the
error table above.

**`search` uses `p.get("name", "")` rather than `p["name"]`.** One record missing a name
field shouldn't take down a search over two thousand records.

**`departments` uses a set comprehension and then sorts.** The set removes duplicates;
the sort makes output stable. Unstable ordering is the kind of thing that produces
mystifying test failures six weeks later.

**The functions each do one thing and return a value.** They don't print, except where
reporting an error. Functions that return data can be tested, reused and composed;
functions that print can only be watched. This matters more as projects grow, and it
costs nothing to start now.

None of this is advanced. It's the difference between code that works on your machine
today and code that still works when someone else runs it on different data.

## Where this leaves you

You can now read almost any AI code sample and know what you're looking at. You won't
understand every line — nobody does, on first read — but you'll recognise the shapes:
construct a client, build a dict of parameters, call a method, pull a value out of a
nested response.

That recognition is the actual goal of this chapter, and it's worth being clear about
why. You are not going to write much Python from scratch in what follows. You are
going to read a great deal of it — documentation, samples, code a model wrote for you —
and the difference between productive and stuck is whether you can look at an
unfamiliar twenty lines and say *I know what that's doing* rather than *I hope that
works*.

The second habit is the dangerous one with AI assistance, incidentally. A coding
assistant will happily hand you working-looking code that calls a deprecated method,
embeds a credential, or silently swallows an error. You need to be able to notice. That
requires exactly the fifteen percent in this chapter, and no more.

Next chapter goes inside the model itself: tokens, context windows, temperature and
embeddings. It's where the vocabulary from chapter 1 turns into numbers you can control,
and where most of the cost and latency decisions you'll ever make are actually set.

---

### Chapter review

1. What is the difference between a list, a dict and a tuple, and which will you use
   most when handling API responses?
2. Why use `person.get("email")` rather than `person["email"]` on data from an API?
3. What does `json.loads` do, and what does the `s` stand for?
4. Give two reasons a credential should never appear in source code.
5. What does `timeout=30` prevent, and what happens without it?
6. Why does error handling matter more when calling a model API than when reading a
   local file?
