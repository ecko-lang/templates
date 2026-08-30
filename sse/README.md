# {name}

A server-sent-events app written in Ecko: a streaming endpoint and a browser
client that renders what it sends.

## Run it

    cd {name}
    ecko dev main.ecko

Then open http://localhost:8080 and watch the ticks arrive: five of them, a
second apart, then `done`. Set `PORT` to serve somewhere else.

Run it from this directory: both the static route and the client page resolve
against the working directory.

## How the stream works

A handler returns a map with a `stream:` channel instead of a `body:`. The
server writes each value it receives as a chunk and ends the response when the
channel closes:

    { status: 200, headers: { "content-type": "text/event-stream" }, stream: ch }

`feed` is an `async fn`, and calling one spawns a task. That is why `events`
can return its response straight away while the body fills in behind it.

The client page is served from a route rather than a static mount, because
`std.web` has no directory index: mounting `static/` at `/` returns 404 for `/`.

## Test it

    ecko test

The tests drain the channel directly, so they need no socket and no waiting.
