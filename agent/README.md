# {name}

An AI agent written in Ecko: a tool the model can call, a typed answer, and a
contract on the output.

## Run it

    cd {name}
    ecko main.ecko

That works with no API key. `ai` falls back to mock mode, which is
deterministic and schema-valid, so the program runs and the tests pass offline.
To use a real provider:

    ECKO_API_KEY=sk-... ecko main.ecko

The code does not change between the two.

## What to look at

- `search_notes` is marked `@tool`, which is what lets `ai` decide to call it.
- `triage` uses `ai[Urgency]`, so the return type is the schema and nothing
  outside the enum can come back.
- `answer` carries `@requires` and `@ensures`. A violated `@ensures` retries
  the call rather than returning something wrong.

Note the order: `export` comes before an attribute, as in
`export @tool("...") fn search_notes(query)`. The other way round is a parse
error.

## Test it

    ecko test
