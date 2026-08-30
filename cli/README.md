# {name}

A command-line tool written in Ecko.

## Run it

    cd {name}
    ecko main.ecko --help
    ecko main.ecko -n Ada -c 2 --loud

## Layout

- `app.ecko` holds the spec and the logic. It never reads argv and never exits,
  which is what lets the tests call it directly.
- `main.ecko` reads the arguments and prints the result. Change `TOOL` there to
  your own command name.
- `tests/{name}_test.ecko` runs offline. Run it with `ecko test`.

## Ship it

`ecko build main.ecko -o {name}` compiles this into a single self-contained
executable with no runtime to install.
