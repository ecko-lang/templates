# {name}

Text helpers for building URLs and previews. A package written in Ecko.

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

## API

- `slugify(text)` reduces `text` to a lowercase, hyphen-separated slug.
- `excerpt(text, limit)` shortens `text` to at most `limit` characters on a
  word boundary, adding an ellipsis when it cut something.

Every export carries a `##` comment. `ecko doc main.ecko` prints
`*Undocumented.*` for any that does not, so add one whenever you add an export.

## Testing

    ecko test

Tests live in `tests/`, which keeps them out of the published archive. A test
file at the package root would ship inside the zip.

## License

MIT
