# clap-dyn-autocomplete

Dynamic runtime shell completions for CLIs built with [clap](https://github.com/clap-rs/clap).

Unlike `clap_complete`, which can only generate completions for values known ahead of time, `clap-dyn-autocomplete` supports generating completion candidates at runtime — useful for things like package names, file contents, or any other dynamic data source.

## How it works

All completion logic lives in Rust. The shell adapter script calls your binary with a special subcommand, and your binary returns completion candidates line by line. No shell-specific logic needed.

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

Add a `complete` subcommand to your CLI:

```rust
use clap::Parser;
use clap_dyn_autocomplete::{Complete, CustomCompleter, CustomCompleterFactory, RootCtx};

#[derive(Parser)]
enum Cli {
    /// Your normal commands
    List,
    /// Shell completions (used by shell adapter scripts)
    Complete(Complete),
}

struct MyFactory;

impl CustomCompleterFactory for MyFactory {
    type CustomCompleter = MyCompleter;
    async fn build(&self, _ctx: &RootCtx<'_>) -> MyCompleter {
        MyCompleter
    }
}

struct MyCompleter;

impl CustomCompleter for MyCompleter {
    async fn complete(&self, _ctx: &RootCtx<'_>, _path: &[&str], arg_id: &str) -> Vec<String> {
        match arg_id {
            // return dynamic values for specific arguments
            "package" => vec!["firefox".into(), "wget".into()],
            _ => vec![],
        }
    }
}

fn main() {
    let cli = Cli::parse();
    match cli {
        Cli::Complete(c) => {
            tokio::runtime::Runtime::new().unwrap()
                .block_on(c.println_to_stub_script::<Cli>(None, MyFactory));
        }
        Cli::List => println!("listing..."),
    }
}
```

Install the shell adapter script:

```bash
# Zsh
my-cli complete --emit-stub zsh > ~/.zfunc/_my-cli

# Fish  
my-cli complete --emit-stub fish > ~/.config/fish/completions/my-cli.fish
```

## Credits

I stole this from Microsoft. Original code from [OpenVMM](https://github.com/microsoft/openvmm/tree/main/support/clap_dyn_complete), licensed under MIT.
