# Quick Rustup Tutorial
## Overview
**Rustup** is a manager for installations of the Rust programming langauge.  Rustup is formally a *toolchain multiplexer* that manages and installs the Rust toolchain, namely the compiler `rustc` and package manager `cargo`.

An overview of upcoming concepts:
- **toolchain** - the tools that together transform Rust code into a final product.  Individual tools are called **components**.  The most basic required component is the compiler `rustc`.  Others are optional, like the linter `clippy`.
- **channel** - variants of the `rustc` compiler (stable, beta, and nightly) and specific version of that variant.
- **target** - the platform for which the code will be built for e.g. x86, amd64, RISC-V, etc.
- **toolchain specification** - a specific instance of a toolchain.  A basic specificaton includes the channel (variant and version of `rustc`) and host platform (machine the toolchain will run on).  We can order `rustup` to install/set-up a specific toolchain given a toolchain specification.
- **profile** - a specific grouping of components.  Rustup can manage profiles for easier toolchain management.
- **proxy** - user-tools (CLI commands) to manage/manipulate specific components of the toolchain.  For example, the command `rustc` can be used to invoke and pass arguments to the Rust compiler.  They are called *proxies* because they are wrappers around the component e.g. `rustc` is a CLI wrapper around the actual compiler.

## Installing Toolchains
Rustup manages Rust **toolchains**: single installations of the Rust compiler plus any other tools.  The official toolchains are those provided through the official channels tracked by Rustup, but we can specify alternative/custom toolchains, too.
### Toolchain Specification
Toolchains are named by their channel, version, and host-platform (specified as a *target-triple*); together they make up the **toolchain specification**.  For example:

``` stable-1.16.0-x86_64-linux-gnu ```

- The *channel* is `stable`
- The *version* is `1.16.0`
- The *host-platform* is `x86_64-linux-gnu`

A **channel** refers to different versions of the Rust programming language, implemented in variants of the `rustc` compiler.

The channels are:
- **stable** - the latest official release of Rust.
	- Uses major-minor versioning e.g. `1.45.2`
- **beta** - the next official release of Rust, made available so developers can look for problems.
- **nightly** - Rust with the latest experimental features added, updated every night (hence "nightly").
	- Versions specify night of release e.g. `nightly-2020-07-27`

We pass a toolchain specification as an argument to Rustup.  However, we can omit any part of the toolchain specification and Rustup will infer:
- The default channel is *stable*.  
- The default version is the *latest*.
- The default host-platform is the same as the host-machine.

We will see this in action in the coming sections.
### Show Installed Toolchains
To show all installed toolchains on your machine:
```Shell
$ rustup show
```

An example output:
``` Shell
Default host: x86_64-unknown-linux-gnu
rustup home:  /home/theodore/.rustup

installed toolchains
--------------------

stable-x86_64-unknown-linux-gnu (default)

active toolchain
----------------

stable-x86_64-unknown-linux-gnu (default)
```
### Installing a Toolchain
We can install a toolchain by providing the full toolchain specification:
``` Shell
$ rustup toolchain install stable-1.16.0-x86_64-linux-gnu
```

However, as explained previously, we can omit any part of the toolchain specification and Rustup will infer:
- The default channel is *stable*.  
- The default version is the *latest*.
- The default host-platform is the same as the host-machine.

For example, to install the latest *nightly* channel toolchain:
``` Shell
$ rustup toolchain install nightly
```

Or to install a specific version of nightly (remembering the version scheme):
``` Shell
$ rustup toolchain install nightly-2020-07-27
```

A nightly channel installation might fail if your system is missing a component (such as the non-default component `clippy`).  To force Rustup to install with missing components anyway:
``` Shell
$ rustup toolchain install nightly --force
```

It might be easier to have Rustup install a toolchain with a different profile.  The profile `minimal` should always be installable:
``` Shell
$ rustup toolchain install nightly --profile=minimal
```

### Run on a Specific Toolchain
To run Rustup on a specific toolchain, for example, to check the compiler version:
``` Shell
$ rustup run nightly rustc --version
```

To set the global default channel to a different one than `stable`:
``` Shell
$ rustup default nightly
```
Any invocation of `cargo` or `rustc` will now use the `nightly` variant.

To set the compiler version for *only* a package, from within the package root:
``` shell
$ rustup override set nightly
```

We can also specify this in the `rust-toolchain.toml` file, explained further in the [[RustupTutorial#Overrides]] section.

## Cross-Compilation
A Rust project is compiled to a machine of a particular architecture, called the **target platform**.  To build an executable for the target platform, `rustc` compiles using the variant of the Standard Library for that target platform.  

By default, the target platform is the same as the host platform - only the Standard Library of the host platform is initially installed.  

To see the available target platforms the current compiler can build for:
``` Rust
$ rustc --print target-list
```

To install the Standard Library of another target platform:
``` Shell
$ rustup add target arm-linux-androideabi
```

To build a package with that target platform:
``` Shell
$ cargo build --target=arm-linus-androideabi
```

We can also define a *custom target* by providing a target file, formatted in `.JSON`.
## Overrides
At the package root (alongside `Cargo.toml`) we can include a **Override File** `rust-toolchain.toml` (or `rust-toolchain` without an extension in older versions of Rust) that specifies arguments or parameters of Rustup for this particular package.

An example `rust-toolchain.toml`:
``` toml
[toolchain]
channel = "nightly-2020-07-10"
components = [ "rustfmt", "rustc-dev" ]
targets = [ "wasm32-unknown-unknown", "thumbv2-none-eabi" ]
profile = "minimal"
```

## Maintenance

### Updating
The latest releases of all things Rust are documented at [Rust Forge](https://forge.rust-lang.org/).

To check for available updates:
``` Shell
$ rustup check
```

To update the Rust Programming language itself i.e. the compiler:
``` Shell
$ rustup update
$ rustup update --nightly
```
Without specification, `rustup update` will update the channel set to `default` to the latest version (by default, this is `stable`).

Rustup by default auto-updates.  To expliticly update Rustup:
``` Shell
$ rustup self-update
```
To turn-off/on the auto-update feature, set the `auto-self-update` variable to `disable`/`enable`, or `check-only`.

### Configuration
Rustup behaves depending on certain **environment variables**.  Some variables need to be set before installation of Rustup.  Others can be set in the Shell.

Environment variables can also be set for individual packages/workspace by defining them in the appropriate `.cargo/config.toml`.

The Rustup configuration file is `$RUST_HOME/settings.toml` (which is by default `$HOME/.rustup/settings.toml`.  This should not be manually modified by hand.