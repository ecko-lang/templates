# {name}

An AI agent written in Ecko: a tool the model can call, a typed answer, and a
contract on the output.

## Run it

    cd {name}
    ecko main.ecko

That works with no API key. `ai` falls back to mock mode, which is
deterministic and schema-valid, so the program runs and the tests pass offline.
To use a real provider:

    ECKO_AI_API_KEY=sk-... ecko main.ecko

The code does not change between the two.

## What to look at

- `search_notes` is marked `@tool`, and `research` calls it with
  `ai "..." using [search_notes]`. Both halves matter: the attribute describes
  the tool, the `using` list is what puts it in front of the model.
- `triage` uses `ai[Urgency]`, so the return type is the schema and nothing
  outside the enum can come back.
- `answer` carries `@requires` and `@ensures`. A violated `@ensures` retries
  the call rather than returning something wrong.

Note the order: `export` comes before an attribute, as in
`export @tool("...") fn search_notes(query)`. The other way round is a parse
error.

## Test it

    ecko test

`ecko test` forces mock mode, so the cases stay offline and deterministic even
when `ECKO_AI_API_KEY` is set.
