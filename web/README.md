# {name}

A web app written in Ecko.

## Run it

    cd {name}
    ecko dev main.ecko

Then open http://localhost:8080. `ecko dev` restarts it whenever you save. Set
`PORT` to serve somewhere else.

Run it from this directory: the static route resolves `static/` against the
working directory.

## Layout

- `app.ecko` holds the routes and handlers. It binds no port, which is what
  lets the tests call the router directly.
- `main.ecko` reads `PORT` and starts the server.
- `static/` is served under `/static`.
- `tests/{name}_test.ecko` runs offline, with no socket.

## Add a route

Add it to the list in `routes()` and give it a handler:

    web.post("/things", fn(req) http.json({ created: true })),

Middleware goes in a second list: `web.router([...], [require_auth])`. Each one
is a `fn(req, next)` that either calls `next(req)` or returns its own response.

## Test it

    ecko test
