# ecko templates

Five starting points for an Ecko project. Each one is a small, complete,
running program: a manifest, source split into logic and entry point, a README,
and a test suite that passes.

They exist because every shape here has a correct form that is otherwise
discoverable only by reading the guides, and the errors arrive late when you
get it wrong.

Nothing here is required to use Ecko. A single `.ecko` file is still a complete
program. These are for when you would rather edit something that works than
start from an empty file.

`ecko scaffold` is the command that writes them, and you can copy one by hand
instead. All five need ecko 0.17.0 or newer, which is the release `ecko scaffold`
itself ships in.

## The five templates

| Template | What it is | Then run |
|---|---|---|
| `agent` | An AI agent: a `@tool` the model can call, `ai[T]` output, and contracts | `ecko main.ecko` |
| `web` | A web app: routes, path parameters, static files, and tests | `ecko dev main.ecko` |
| `sse` | A streaming app: server-sent events over a channel, with a client | `ecko dev main.ecko` |
| `cli` | A command-line tool: options, a parsed spec, and tests | `ecko main.ecko --help` |
| `package` | An importable package: exports, doc comments, example, and tests | `ecko test` |

Those two columns are the `about` and `next` strings from `templates.json`, so
they are the same words `ecko scaffold --list` prints.

## Copying one by hand

```bash
git clone https://github.com/ecko-lang/templates
cp -r templates/web my-site && cd my-site

# the two files that carry the project's name
mv tests/template_test.ecko tests/my-site_test.ecko
sed -i 's/{name}/my-site/g' ecko.json README.md

ecko test
ecko dev main.ecko
```

That is exactly what `ecko scaffold` does for you.

## Using `ecko scaffold`

```bash
ecko scaffold web my-site
cd my-site
ecko dev main.ecko
```

With nothing after it, `ecko scaffold` lists what is available. The first run
on a machine downloads this repo and caches it. Every run after that is
offline.

| Flag | Effect |
|---|---|
| `--list` | Print the templates and exit |
| `--update` | Refetch the templates, replace the cache, report what changed |
| `--name <n>` | Use `n` as the project name instead of the directory's |
| `--force` | Overwrite files that are already there |
| `--ref <ref>` | Take the templates from a branch or tag other than `main` |

`ECKO_TEMPLATES_REPO` points at another repo or at a local directory, which is
how you use your own set:

```bash
ECKO_TEMPLATES_REPO=~/my-templates ecko scaffold internal-service svc
```

## What each template gives you

### `agent`

An AI agent: a tool the model can call, a typed answer, and a contract on the
output. It runs with **no API key**. `ai` falls back to mock mode, which is
deterministic and schema-valid, so the program runs and the tests pass offline.
Set `ECKO_AI_API_KEY` to point the same code at a real provider. The code does not
change between the two.

Worth reading in `app.ecko`: `search_notes` is marked `@tool`, which is what
lets `ai` decide to call it; `triage` uses `ai[Urgency]`, so the return type is
the schema; `answer` carries `@requires` and `@ensures`, and a violated
`@ensures` retries the call rather than returning something wrong.

### `web`

An HTTP server: routes, path parameters, static files. It serves on 8080, or
`PORT` if you set it, and `ecko dev` restarts it on save. Routes live in
`routes()` in `app.ecko`. Add one with `web.get` or `web.post` and a handler.
Middleware goes in a second list, `web.router([...], [require_auth])`, where
each one is a `fn(req, next)`.

### `sse`

A streaming endpoint plus a browser client that renders the events. Open
<http://localhost:8080> and watch ticks arrive.

The handler returns a map with a `stream:` channel instead of a `body:`, and
the server writes each value it receives as a chunk until the channel closes.
`feed` is an `async fn`, and calling one spawns a task, which is why the
handler can return immediately while the body fills in behind it.

### `cli`

A command-line tool with options and generated usage text:

```bash
ecko main.ecko --help
ecko main.ecko -n Ada -c 2 --loud
```

`ecko build main.ecko -o my-tool` compiles it into a single self-contained
executable with no runtime to install.

### `package`

An importable package in the shape the package guide documents: a `module` path
whose last segment matches the directory, `export` markers, a `##` comment on
every export, an `example.ecko`, and tests under `tests/`.

Replace two things in `ecko.json` before publishing. The `you` in
`"module": "github.com/you/my-pkg"` is yours to fill in, and the `description`
describes the example code, as does the LICENSE beside it.

## How a template is put together

Every template splits **what it does** from **starting it**:

```
my-site/
  ecko.json              the manifest, named after your directory
  README.md              how to run it
  app.ecko               exported, documented functions. Binds nothing.
  main.ecko              reads PORT or argv, then starts
  static/style.css       served under /static
  tests/my-site_test.ecko
```

`app.ecko` binds no port and reads no arguments, so the generated test can call
it directly with plain values. That is the whole reason the tests run offline
and need no socket: `web.router(...)` returns an ordinary function you can call
with a request map, and `cli.parse(spec, argv)` is pure.

The `package` template is the one exception. Its exports live in `main.ecko`,
because that is the convention `import` expects.

## How the index works

`templates.json` is the contract between this repo and the CLI:

```json
{
  "name": "web",
  "about": "A web app: routes, path parameters, static files, and tests",
  "next": "ecko dev main.ecko",
  "min_ecko": "0.17.0",
  "files": [
    { "src": "web/ecko.json", "dest": "ecko.json", "subst": true },
    { "src": "web/app.ecko",  "dest": "app.ecko" },
    { "src": "web/tests/template_test.ecko", "dest": "tests/{name}_test.ecko" }
  ]
}
```

- `about` is the one-line description `--list` prints.
- `next` is the command to run from inside the finished project. It is a bare
  command, never a path: the CLI adds the `cd` line.
- `min_ecko` is the oldest release that can use the template. A user on an
  older ecko sees it in `--list` marked as unavailable, rather than getting a
  parse error.
- `src` is a path in this repo; `dest` is a path in the new project.
- `dest` may contain `{name}`, which becomes the project's name. So does the
  body of a file marked `subst`.

## Adding a template

1. Add a directory with the files, following the `app.ecko` / `main.ecko` split.
2. Add an entry to `templates.json`.
3. Set `min_ecko` to the oldest release the template actually works on. If it
   needs syntax that is not released yet, set it to the release that will have
   it.
4. Run the gates locally. These are the same ones CI runs, on a project
   scaffolded out of this checkout rather than on the raw sources:

```bash
name=mytemplate
work=$(mktemp -d)
ECKO_TEMPLATES_REPO="$PWD" ECKO_TEMPLATES_DIR="$work/.cache" \
  ecko scaffold "$name" "$work/demo"
cd "$work/demo"
files=$(find . -name '*.ecko' -not -path './vendor/*' | sort)
ecko fmt --check $files
ecko check --strict $files
ecko test
entry=app.ecko; [ -f "$entry" ] || entry=main.ecko
ecko doc "$entry" | grep Undocumented && echo "an export needs a ## comment"
```

Gate the scaffolded copy, not the template directory. A template's
`ecko.json` holds placeholders - `github.com/you/{name}` - that `ecko check`
rightly refuses as a module path, and only `ecko scaffold` fills them in. It
also exercises `templates.json` itself: a wrong `src` or `dest` fails here.
Run `ecko test` from inside the project, since test discovery is relative to
the working directory and the `sse` template reads `static/index.html` the
same way. The `entry` line matters for `package`, which has no `app.ecko`.

## Rules that are easy to get wrong

**Never mark a `.ecko` file `subst`.** `{` starts string interpolation in Ecko,
so a substituted `.ecko` file stops being a valid program. Only `ecko.json` and
`README.md` are substituted. The CLI rejects an index that asks for otherwise,
and so does CI.

**`export` comes before an attribute.** Write
`export @tool("...") fn search_notes(query)`. Putting `@tool(...)` on the line
above an `export fn` is a parse error, because an attribute has to be followed
directly by `fn` or `async`.

**Every export needs a `##` comment.** `ecko doc` prints `*Undocumented.*`
otherwise, and CI fails on it.

**Unused handler parameters fail `check --strict`.** Prefix them with an
underscore: `fn home(_req)`.

Two more that cost real time when writing these:

- `{` starts interpolation even inside an escaped quote, so `"{\"a\":1}"` does
  not parse. Compare JSON with `json_decode` instead.
- `std.web` has no directory index. Mounting `static/` at `/` returns 404 for
  `/`, so serve a root page from an explicit route.

## Testing

CI runs on every push, on pull requests, and **weekly**. The weekly run is the
one that matters: nobody pushes here between ecko releases, so a push trigger
cannot catch a template going red because a new release changed the formatter.
That is how four stdlib packages drifted after 0.13.0.

It installs the released ecko, validates `templates.json` against the tree
(every `src` exists, every `dest` stays inside the project, no `.ecko` is
substituted, every template declares `min_ecko`), then scaffolds each template
from the checkout with `ecko scaffold` and runs `ecko fmt --check`,
`ecko check --strict`, `ecko test` and the documented-exports check on each
scaffolded project.

## The `cli` template is mirrored in core

`cli/` is byte-identical to `tests/fixtures/templates/cli/` in the core repo,
which uses it to gate the scaffold machinery offline and with no network.
Change one, change both. Nothing enforces this automatically, because CI here
cannot see core.

## License

MIT
