This page is primarily for those interested in vaultwarden development, or who have a specific reason for wanting to build their own binary.

Typical users should either [[deploy via Docker|Which-container-image-to-use]], [[extract the pre-built binaries|Pre-built binaries]] from the Alpine-based Docker images, or look for a [[third-party package|Third-party-packages]].

## Dependencies
- `Rust stable` (strongly recommended to use [rustup](https://rustup.rs/))
- On a Debian based distro some general packages to make sure building should go fine install the following: `build-essential`, `git`
- `OpenSSL` (should be available in path, see [openssl crate docs](https://docs.rs/openssl/latest/openssl/#automatic))
  On a Debian based distro, you need to install `pkg-config` and `libssl-dev`
- For the SQlite3 backend on a Debian based distro you need to install `libsqlite3-dev`
- For the MySQL backend on a Debian based distro you need to install `libmariadb-dev-compat` and `libmariadb-dev`
- For the PostgreSQL on a Debian based distro you need to install `libpq-dev` and `pkg-config`
- `NodeJS` (only when compiling the web-vault, install through your system's package manager, use the [prebuilt binaries](https://nodejs.org/en/download/)) or [nodesource binary distribution](https://github.com/nodesource/distributions) *Note: Building the web-vault currently requires NodeJS v16 and NPM v8.11*

## Run/Compile
### All backends
```sh
# Compile with all backends and run
cargo run --features sqlite,mysql,postgresql --release
# or just compile with all backends (binary located in target/release/vaultwarden)
cargo build --features sqlite,mysql,postgresql --release
```

### SQlite backend
```sh
# Compile with sqlite backend and run
cargo run --features sqlite --release
# or just compile with sqlite (binary located in target/release/vaultwarden)
cargo build --features sqlite --release
```
### MySQL backend
```sh
# Compile with mysql backend and run
cargo run --features mysql --release
# or just compile with mysql (binary located in target/release/vaultwarden)
cargo build --features mysql --release
```
### PostgreSQL backend
```sh
# Compile with postgresql backend and run
cargo run --features postgresql --release
# or just compile with postgresql (binary located in target/release/vaultwarden)
cargo build --features postgresql --release
```

When run, the server is accessible in [http://localhost:8000](http://localhost:8000).


### Install the web-vault
A compiled version of the web vault can be downloaded from [dani-garcia/bw_web_builds](https://github.com/dani-garcia/bw_web_builds/releases).

*Note: building the Vault needs ~1.5GB of RAM. On systems like a RaspberryPI with 1GB or less, please [enable swapping](https://www.tecmint.com/create-a-linux-swap-file/) or build it on a more powerful machine and copy the directory from there. This much memory is only needed for building it, running vaultwarden with vault needs only about 10MB of RAM.*

If you prefer to compile it manually, follow these steps:

#### New (easy way):

- Clone the git repository at [dani-garcia/bw_web_builds](https://github.com/dani-garcia/bw_web_builds):
```sh
# clone the repository
git clone https://github.com/dani-garcia/bw_web_builds.git bw_web_builds
cd bw_web_builds

# Use docker as the build environment (safest way and uses the correct build versions)
# This will build the web-vault, and extract the files to a docker_build directory.
make docker-extract

# Using the host provided npm and node instead.
make full
```

#### Old (very manual way):

- Clone the git repository at [bitwarden/clients](https://github.com/bitwarden/clients) and checkout the latest release tag (e.g. v2022.6.0):
```sh
# clone the repository
git clone https://github.com/bitwarden/clients.git web-vault
cd web-vault
# switch to the latest tag
git -c advice.detachedHead=false checkout web-v2022.6.0
# Or use the commit hash for that version
git -c advice.detachedHead=false checkout bb5f9311a776b94a33bcf0a7bff44cd87a2fcc92
```

- Patch all the images from [resources](https://github.com/dani-garcia/bw_web_builds/tree/master/resources) according to the instructions in the [apply_patches script](https://github.com/dani-garcia/bw_web_builds/blob/master/scripts/apply_patches.sh)

- Download the patch file from [dani-garcia/bw_web_builds](https://github.com/dani-garcia/bw_web_builds/tree/master/patches) and copy it to the `web-vault` folder.
To choose the version to use, assuming the web vault is version `vXXXX.Y.Z`:
  - If there is a patch with version `vXXXX.Y.Z`, use that one
  - Otherwise, pick the one with the largest version that is still smaller than `vXXXX.Y.Z`
- Apply the patch
```sh
# In the 'web-vault' directory
git apply vXXXX.Y.Z.patch
```

- Then, build the Vault:

```sh
npm ci
# Read the note below (we do use this for our docker builds).
# npm audit fix

# Change to the web-vault directory
cd apps/web
# Build the web-vault
npm run dist:oss:selfhost
```

*Note: You might be asked to run ```npm audit fix``` to fix vulnerability. This will automatically try to upgrade packages to newer version, which might not be compatible and break web-vault functionality``` Use it at your own risk, if you know what you are doing. We do use this on our own releases btw!*

Finally copy the contents of the `build` folder into the destination folder:
- If you run with `cargo run --release`, it's `vaultwarden/web-vault`.
- If you run the compiled binary directly, it's next to the binary, in `vaultwarden/target/release/web-vault`.

## Configuration
The available configuration options are documented in the default `.env.template` file, and they can be modified by uncommenting the desired options in that file or by setting their respective environment variables. See the Configuration section of this wiki for the main configuration options available.
If you want to use this file you need to copy it, and name it `.env` and adjust the settings in that specific file.

Note: the environment variables override the values set in the `.env` file.

## More information for deployment
- [Configuring your reverse proxy](https://github.com/dani-garcia/vaultwarden/wiki/Proxy-examples)
- [Setting up Autostart via systemd](https://github.com/dani-garcia/vaultwarden/wiki/Setup-as-a-systemd-service)

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
Refer to [using the MariaDB (MySQL) Backend](https://github.com/dani-garcia/vaultwarden/wiki/Using-the-MariaDB-%28MySQL%29-Backend) if you want to migrate from SQLite.

## How to migrate from SQLite backend to PostgreSQL backend (for developers)
Refer to [using the PostgreSQL backend](https://github.com/dani-garcia/vaultwarden/wiki/Using-the-PostgreSQL-Backend) if you want to migrate from SQLite.