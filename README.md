# clap-dyn-autocomplete

Dynamic, runtime-driven shell completions for CLIs built with [clap](https://github.com/clap-rs/clap).

Unlike `clap_complete`, which can only generate completions for values known ahead of time, `clap-dyn-autocomplete` supports generating completion candidates at runtime — useful for things like package names, file contents, or any other dynamic data source.

## Features

- Runtime-generated completion candidates
- Shell-agnostic Rust logic with minimal shell adapter scripts
- Supports Zsh, Fish, and PowerShell

## Usage

Add to your `Cargo.toml`:

```toml
[dependencies]
clap-dyn-autocomplete = "0.1"
```

Implement `CustomCompleterFactory` and `CustomCompleter` traits to provide dynamic completions for your arguments, then add a `complete` subcommand to your CLI that calls `Complete::println_to_stub_script`.

Generate the shell adapter script:

```bash
my-cli complete --emit-stub zsh > ~/.zfunc/_my-cli
```

## Credits

I stole this from Microsoft. Original code from [OpenVMM](https://github.com/microsoft/openvmm/tree/main/support/clap_dyn_complete), licensed under MIT.
