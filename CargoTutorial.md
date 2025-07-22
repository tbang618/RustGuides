# Cargo Tutorial
## Rust Compiler 
The Rust compiler itself is **rustc**.  Simplistically, your Rust source code is compiled into a **crate**.  If the crate is used by another program, it is a *library crate*.  If the crate itself is executable, then it is a *binary crate*.

``` Diagram
Rust Source Code -> rustc -> crate
```

This is fine for small projects, but for larger or more complex projects, simply using *rustc* is tedious.  Besides keeping tack of dependencies, we would need to know in what order to invoke the compiler, and any arguments, depending on the compilation target.  Anything that needs to be changed can become a hassle.
## Cargo Overview
We can group the source files of a project together with a **manifest** that describes the project: this abstraction is called **package**.

**Cargo** is Rust's package manager.  We can abstract the tedium of building projects to Cargo.

What Cargo can do:
- Correctly invoke rustc (or another tool)
- Resolve dependencies (fetch and build necessary library crates)
- Set up project managment tools

## Cargo Usage
### Naming Conventions
By convention, Rust uses **snake case** for crates and files.
### Create Binary Crate Package
Cargo creates a package by:
- Creating `src/` : the directory that contains our source files.
	- By default, a single source file `main.rs` is already created.
- A manifest file `cargo.toml` with basic attributes filled in.
- Creating a git repository for this project.

``` Shell
$ cargo new hello_world --bin
```

Arguments:
- `--bin` indicates our project produces a binary crate (pass `--lib` for a library crate).
- `--vcs none` if we don't want a git repository initialized.

To build our project:

``` Shell
$ cargo build
```

Outputs:
- The binary executable is in `/target/debug/`
- The package dependencies info is in the newly created `cargo.lock` file.

To compile and run in one-go:

``` Shell
$ cargo run
```

The previous command builds our project in debugging mode (tests modules are run and the crate is not optimized).  To produce an optimized, final crate:

``` Shell
$ cargo build --release
```

The output binary crate is located in `target/release/`.
### Dependencies
Packages/crates maintained by the official Rust community are found at [crates.io](crates.io) which is Rust's official *package registry*.  Cargo by default will resolve package dependencies by looking there.

We can explicitly list dependencies in the manifest file under the `[dependencies]` heading.  For example:

``` toml
[package]
name = "hello_world"
version = "0.1.0"
edition = "2021"

[dependencies]
time = "0.1.12"
regex = "0.1.41"
```

Our package, in order to build, relies on the library crate `time` (version 0.1.12) and the library crate `regex` (version 0.1.41).   When building our project: 
1. Cargo will automatically get the packages for `time` and `regex` and *their* dependencies.
2. Build all these library packages.
3. and then use the resulting library crates to build our binary crate.

### Package Layout
Cargo organizes files in a package to a specific layout:

```
.
├── .cargo/
├── Cargo.lock
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── main.rs
│   └── bin/
│       ├── named-executable.rs
│       ├── another-executable.rs
│       └── multi-file-executable/
│           ├── main.rs
│           └── some_module.rs
├── benches/
│   ├── large-input.rs
│   └── multi-file-bench/
│       ├── main.rs
│       └── bench_module.rs
├── examples/
│   ├── simple.rs
│   └── multi-file-example/
│       ├── main.rs
│       └── ex_module.rs
└── tests/
    ├── some-integration-tests.rs
    └── multi-file-test/
        ├── main.rs
        └── test_module.rs
```

Explanation:

- `.cargo` is where local configuration files for Cargo are stored (optional, see Configuration section).
- `Cargo.toml` and `Cargo.lock` are stored in the root of your package (_package root_).
- Source code goes in the `src` directory.
- The default library file is `src/lib.rs`.
- The default executable file is `src/main.rs`.
    - Other executables can be placed in `src/bin/`.
- **Benchmarks** go in the `benches` directory.
- **Examples** go in the `examples` directory.
- **Integration tests** go in the `tests` directory.

### Cargo.toml and Cargo.lock
Cargo uses two different files to handle dependencies:
- `Cargo.toml` is where the user tells Cargo what dependencies are needed.
- `Cargo.lock` is used by Cargo internally to track dependencies.

A programmer should never touch `Cargo.lock` themselves; it is maintained and updated automatically by Cargo.  For example, it keeps track of the revision number of dependencies used in the last build.  So if we copy our package to another machine, building will produce the exact same binary crate.  It is better to leave such things free from the taint of human touch.

### Tests
In Rust, tests are special functions (prefaced by the directive `#[test]` used for program verification.

Cargo can run tests via the `cargo test` command; this also checks examples in `examples/`.

By convention:
- Unit and documentation tests should be written with the source files.  
- Integration tests should separately be placed in the `test/` directory.

Specific tests can be run with:

``` Shell
$ cargo test foo
```

where any test with name that contains `foo` will be run.

### Continous Integration
**Continuous integration** or CI is a DevOps practice where code changes (from multiple developers) are regularly (and automatically) integrated into a central repository to be built and tested.

Cargo is able to interact/plug-into several popular CI software:
- GitHub Actions
- GitLab CI
- Source Hut

These software read a special `.yml` file that is part of the package, describing how CI should be performed.  Where and what to call the `.yml` file depends on which CI software tool you're using.

## Workspaces
Cargo can be used to manage multiple packages.  A group of packages that are managed together is called a **workspace**.

In a single package, the build output is placed in 


## Cargo Configuration
Cargo, the program itself, uses a special directory referred as **Cargo Home** to hold its configuration file and to store used crates.  Cargo can be configured for individual packages or globally.
### Cargo Home
By default, Cargo Home is `$HOME/.cargo/` and contains:
- `config.toml` : Cargo's global configuration file.
- `credentials.toml` : private login credentials for custom-defined package registries.
- `.crates.toml` and `.crates2.json` : installed packages information.
- `bin/` : binaries used by Cargo itself plus installed crates.
- `git/`:
	- `git/db/` : Git repositories cloned by Cargo are stored here.  These are Bare Repos.
	- `git/checkous` : Checkous from a repo in `git/db/` are stored here.
- `registry/` : packages/crates used as dependencies.
	- `registry/index` : lists downloaded packages/crates
	- `registry/cache` : the actual packages/crates as `.crate` archives.
	- `registry/src` : where unpacked `.crate` archives are stored.

To clarify, `bin/` contains crates installed by Cargo independently of a package via 
``` Shell
$ Cargo install ..
```

On the other hand, if a package is listed as a dependency in a package's `cargo.toml` manifest, it will be downloaded into `registry/cache/` (known as the **Build Cache**) as a `.crate` archive file and noted in `registry/index`.  When a package is being built, the `.crate` archive is unpacked into `registry/src` and whatever unpacked `.rs` files needed by rustc are used.

Rather than managing Cargo Home by hand, we can use the **Home** crate.

A different Cargo Home may be indicated by the environment variable `$CARGO_HOME`.
### Configuration Files
By default, the global configuration file for Cargo is `$HOME/.cargo/config.toml` or `$CARGO_HOME/config.toml`.

For local configuration, Cargo searches for `.cargo/config.toml` starting at the *package root*, and then successive parent directories (until `/.cargo/config.toml`).  For example, if our project is at `/Projects/foo/bar/baz`, then Cargo searches in the following order:

1. `/projects/foo/bar/baz/.cargo/config.toml`
2. `/projects/foo/bar/.cargo/config.toml`
3. `/projects/foo/.cargo/config.toml`
4. `/projects/.cargo/config.toml`
5. `/.cargo/config.toml`
6. `$CARGO_HOME/config.toml` which defaults to:
    - Windows: `%USERPROFILE%\.cargo\config.toml`
    - Unix: `$HOME/.cargo/config.toml`

