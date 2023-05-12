# Spacemesh app for Ledger Wallet

## Overview

This app adds support for the Spacemesh native token to Ledger hardware wallets.

Current Features:
- Pubkey queries
- Parse, display and sign all Spacemesh CLI generated transaction formats
- Blind sign arbitrary transactions (Enabled via settings)

## Prerequisites

### For building the app
* [Install Docker](https://docs.docker.com/get-docker/)
* For Linux hosts, install the Ledger Nano [udev rules](https://github.com/LedgerHQ/udev-rules)

#### Get the [Ledger App Builder](https://github.com/LedgerHQ/ledger-app-builder) Docker image

Note: These instructions are for building on Linux. They may differ slightly for other platforms.

1. Pull the latest "full" image
```
> docker pull ghcr.io/ledgerhq/ledger-app-builder/ledger-app-builder:latest
```
  * You may try to [build the container image yourself](https://github.com/LedgerHQ/ledger-app-builder#build-the-container-image) but we don't recommend doing so. We recommend using the latest image as described above.

2. Run the image
```
> docker run --rm -ti  -v "$(realpath .):/app" --privileged ghcr.io/ledgerhq/ledger-app-builder/ledger-app-builder:latest
```
  * The `--privileged` flag is required to allow the Docker container to access your Ledger device via USB
  * If permissions errors are encountered, ensure that your user is in the `docker` group and that the session has been restarted

3. Set `BOLOS_SDK` to [the correct SDK name for your device](https://github.com/LedgerHQ/ledger-app-builder#compile-your-app-in-the-container), then run `make` inside the Docker container. E.g., for Nano S:
```
bash-5.1# BOLOS_SDK=$NANOS_SDK make
```

### For working with the device
#### Install Python3 PIP
Ubuntu Linux:
```
sudo apt install pip3
```
MacOS
```
brew install python3
```
#### Install ledgerblue python module
```
pip3 install ledgerblue
```
#### Locally clone SDK repos corresponding to those included in the Docker image
* Setup environment
```bash
cat >>"${HOME}"/.profile <<EOF
LEDGER_SDK_BASE_PATH= # set to a writable path
: "${LEDGER_SDK_BASE_PATH:?}"
export NANOS_SDK="${LEDGER_SDK_BASE_PATH}"/nanos-secure-sdk
export NANOX_SDK="${LEDGER_SDK_BASE_PATH}"/nanox-secure-sdk
export NANOSP_SDK="${LEDGER_SDK_BASE_PATH}"/nanosplus-secure-sdk
EOF
```
* Update the current session's environment
```bash
. "${HOME}"/.profile # Or close and reopen your terminal
```
* Clone the repositories
```bash
# Nano S SDK
git clone --branch 2.1.0 --depth 1 https://github.com/LedgerHQ/nanos-secure-sdk.git "${NANOS_SDK}"

# Nano X SDK
git clone --branch 2.0.2-2 --depth 1 https://github.com/LedgerHQ/nanox-secure-sdk.git "${NANOX_SDK}"

# Nano S+ SDK
git clone --branch 1.0.3 --depth 1 https://github.com/LedgerHQ/nanosplus-secure-sdk.git "${NANOSP_SDK}"
```

### For running the test suite
* [Rust](https://rustup.rs/)
* Solana [system dependencies](https://github.com/solana-labs/solana/#1-install-rustc-cargo-and-rustfmt)

## Build
It is highly recommended that you read and understand the [Ledger App Builder](https://developers.ledger.com/docs/nano-app/build/)
build process before proceeding.  A convenience wrapper script (`./docker-make`) has been provided for simplicity

`docker-make` manages the current target SDK for you, automatically setting `BOLOS_SDK` to the
correct path when the Docker container is launched. A `TARGET_SDK` must be specified when building
from clean and clean must be run _before_ switching
#### `TARGET_SDK`
|Moniker|Device|
|:-----:|:-----|
|s|Nano S|
|x|Nano X|
|sp|Nano S+|

```bash
./docker-make <TARGET_SDK>
```

### Clean
```bash
./docker-make clean
```

## Working with the device
Requires that the `BOLOS_SDK` envvar [be set](https://developers.ledger.com/docs/nano-app/load/).
This can be achieved by first [building](#build) for the desired target device.
### Load
```bash
make load-only
```

### Delete
```bash
make delete
```

## Test
### Unit
Run C tests:
```bash
make -C libsol
```
### Integration
First enable `blind-signing` in the App settings
```bash
cargo run --manifest-path tests/Cargo.toml
```
