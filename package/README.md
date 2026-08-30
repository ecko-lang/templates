# {name}

A package written in Ecko: text helpers for building URLs and previews.

## Before you publish

Two things in `ecko.json` are placeholders:

- `module` is `github.com/you/{name}`. Replace `you` with your own owner. The
  last segment has to stay `{name}`, because that is what becomes the directory
  the package vendors into and the name people `import`.
- `description` and `LICENSE` describe the example code. Replace both.

## Install

    ecko get github.com/you/{name}@v0.1.0

## Usage

    import {name}

    {name}.slugify("Hello, World!")        # "hello-world"
    {name}.excerpt("a long sentence", 10)  # "a long..."

`ecko example.ecko` runs the package straight from this directory, with no
install step.

## API

- `slugify(text)` reduces `text` to a lowercase, hyphen-separated slug.
- `excerpt(text, limit)` keeps whole words from `text` up to `limit`
  characters, then adds an ellipsis if it had to cut. The ellipsis is not
  counted against `limit`.

Every export carries a `##` comment. `ecko doc main.ecko` prints
`*Undocumented.*` for any that does not, so add one whenever you add an export.

## Testing

    ecko test

Tests live in `tests/`, which keeps them out of the published archive. A test
file at the package root would ship inside the zip.

## License

MIT
