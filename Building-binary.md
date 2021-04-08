This page is primarily for those interested in bitwarden_rs development, or who have a specific reason for wanting to build their own binary.

Typical users should either [[deploy via Docker|Which-container-image-to-use]], [[extract the pre-built binaries|Pre-built binaries]] from the Alpine-based Docker images, or look for a [[third-party package|Third-party-packages]].

## Dependencies
- `Rust nightly` (strongly recommended to use [rustup](https://rustup.rs/))
- On a Debian based distro some general packages to make sure building should go fine install the following: `build-essential`, `git`
- `OpenSSL` (should be available in path, install through your system's package manager or use the [prebuilt binaries](https://wiki.openssl.org/index.php/Binaries))  
  On a Debian based distro, you need to install `pkg-config` and `libssl-dev`
- For the SQlite3 backend on a Debian based distro you need to install `libsqlite3-dev`
- For the MySQL backend on a Debian based distro you need to install `libmariadb-dev-compat` and `libmariadb-dev`
- For the PostgreSQL on a Debian based distro you need to install `libpq-dev` and `pkg-config`
- `NodeJS` (only when compiling the web-vault, install through your system's package manager, use the [prebuilt binaries](https://nodejs.org/en/download/)) or [nodesource binary distribution](https://github.com/nodesource/distributions)
*Note: web-vault currently uses a package base (e.g. node-sass <v4.12) which requires NodeJS v11*

## Run/Compile
### All backends
```sh
# Compile with all backends and run
cargo run --features sqlite,mysql,postgresql --release
# or just compile with all backends (binary located in target/release/bitwarden_rs)
cargo build --features sqlite,mysql,postgresql --release
```

### SQlite backend
```sh
# Compile with sqlite backend and run
cargo run --features sqlite --release
# or just compile with sqlite (binary located in target/release/bitwarden_rs)
cargo build --features sqlite --release
```
### MySQL backend
```sh
# Compile with mysql backend and run
cargo run --features mysql --release
# or just compile with mysql (binary located in target/release/bitwarden_rs)
cargo build --features mysql --release
```
### PostgreSQL backend
```sh
# Compile with postgresql backend and run
cargo run --features postgresql --release
# or just compile with postgresql (binary located in target/release/bitwarden_rs)
cargo build --features postgresql --release
```

When run, the server is accessible in [http://localhost:8000](http://localhost:8000).

~*Note: A previous [issue](https://github.com/rust-lang/rust/issues/62896) meant that compilation could fail with a segfault due to an incompatibility between the Rust compiler and LLVM. As a work around an older version of the compiler could be used, e.g. ```cargo +nightly-2019-08-27 build --features yourbackend --release```*~

### Install the web-vault
A compiled version of the web vault can be downloaded from [dani-garcia/bw_web_builds](https://github.com/dani-garcia/bw_web_builds/releases).

If you prefer to compile it manually, follow these steps:

*Note: building the Vault needs ~1.5GB of RAM. On systems like a RaspberryPI with 1GB or less, please [enable swapping](https://www.tecmint.com/create-a-linux-swap-file/) or build it on a more powerful machine and copy the directory from there. This much memory is only needed for building it, running bitwarden_rs with vault needs only about 10MB of RAM.*

- Clone the git repository at [bitwarden/web](https://github.com/bitwarden/web) and checkout the latest release tag (e.g. v2.1.1):
```sh
# clone the repository
git clone --recurse-submodules https://github.com/bitwarden/web.git web-vault
cd web-vault
# switch to the latest tag
git checkout "$(git tag --sort=v:refname | tail -n1)"
```

- Download the patch file from [dani-garcia/bw_web_builds](https://github.com/dani-garcia/bw_web_builds/tree/master/patches) and copy it to the `web-vault` folder.
To choose the version to use, assuming the web vault is version `vX.Y.Z`:
  - If there is a patch with version `vX.Y.Z`, use that one
  - Otherwise, pick the one with the largest version that is still smaller than `vX.Y.Z`
- Apply the patch
```sh
# In the 'web-vault' directory
git apply vX.Y.Z.patch
```

- Then, build the Vault:

```sh
npm install
# Read the note below (we do use this for our docker builds).
# npm audit fix
npm run dist
```
*Note: You might be asked to run ```npm audit fix``` to fix vulnerability. This will automatically try to upgrade packages to newer version, which might not be compatible and break web-vault functionality``` Use it at your own risk, if you know what you are doing. We do use this on our own releases btw!*

Finally copy the contents of the `build` folder into the destination folder:
- If you run with `cargo run --release`, it's `bitwarden_rs/web-vault`.
- If you run the compiled binary directly, it's next to the binary, in `bitwarden_rs/target/release/web-vault`.

## Configuration
The available configuration options are documented in the default `.env` file, and they can be modified by uncommenting the desired options in that file or by setting their respective environment variables. See the Configuration section of this wiki for the main configuration options available.

Note: the environment variables override the values set in the `.env` file.

## More information for deployment
- [Configuring your reverse proxy](https://github.com/dani-garcia/bitwarden_rs/wiki/Proxy-examples)
- [Setting up Autostart via systemd](https://github.com/dani-garcia/bitwarden_rs/wiki/Setup-as-a-systemd-service)

## How to recreate database schemas for the sqlite backend (for developers)
Install diesel-cli with cargo:
```sh
cargo install diesel_cli --no-default-features --features sqlite-bundled
```

Make sure that the correct path to the database is in the `.env` file.

If you want to modify the schemas, create a new migration with:
```
diesel migration generate <name>
```

Modify the *.sql files, making sure that any changes are reverted in the down.sql file.

Apply the migrations and save the generated schemas as follows:
```sh
diesel migration redo

# This step should be done automatically when using diesel-cli > 1.3.0
# diesel print-schema > src/db/sqlite/schema.rs
```

## How to migrate from SQLite backend to MySQL backend (for developers)
Refer to [using the MySQL backend](https://github.com/dani-garcia/bitwarden_rs/wiki/Using-the-MySQL-Backend) if you want to migrate from SQLite.

## How to migrate from SQLite backend to PostgreSQL backend (for developers)
Refer to [using the PostgreSQL backend](https://github.com/dani-garcia/bitwarden_rs/wiki/Using-the-PostgreSQL-Backend) if you want to migrate from SQLite.