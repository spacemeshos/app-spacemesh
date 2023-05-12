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

### Get the [Ledger App Builder](https://github.com/LedgerHQ/ledger-app-builder) Docker image

Pull the latest "full" image:
```
> docker pull ghcr.io/ledgerhq/ledger-app-builder/ledger-app-builder:latest
```
  * You may try to [build the container image yourself](https://github.com/LedgerHQ/ledger-app-builder#build-the-container-image) but we don't recommend doing so. We recommend using the latest image as described above.

### For working with the device

If you're able to successfully build and load the app using the docker images as described below, you probably don't need to install `ledgerblue` as it's pre-installed in the docker images. If, however, you have issues with the docker build and load process you can try using `ledgerblue` directly.

Follow the instructions in the [blue-loader-python](https://github.com/LedgerHQ/blue-loader-python) repository to install the `ledgerblue` python module. We strongly recommend using a virtualenv as suggested.

### For running the test suite
* [Rust](https://rustup.rs/)
* Solana [system dependencies](https://github.com/solana-labs/solana/#1-install-rustc-cargo-and-rustfmt)

## Build

We recommend building using Ledger's provided ledger-app-builder Docker image. The full steps follow. A convenience wrapper script (`./docker-make`) has been provided for simplicity but it may not work out of the box on all configurations. Instructions on using this script are below.

### Using the Ledger App Builder Docker image

Note: These instructions have been tested on Linux and macOS. They may differ slightly for other platforms. Feel free to open an issue in this repository if you encounter issues on other platforms.

1. Pull the latest "full" image (as described above)

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

## Load

It's possible to load the app directly onto a physical Ledger device using a process called sideloading, using the `ledgerblue` Python module. Note that **sideloading is only supported on Nano S and Blue devices, and not on Nano X**. See the [ledgerblue documentation](https://github.com/LedgerHQ/blue-loader-python#supported-devices) for more information.

The simplest way to load the app is to build it using the Docker image as described above, then run the load command inside the container:
```
bash-5.1# BOLOS_SDK=$NANOS_SDK make load
```
  * If you get a segmentation fault during the load process (fairly common on Linux), try physically disconnecting and reconnecting the Ledger device, then close and reopen the Docker instance.
  * If you get the error `ledgerblue.commException.CommException: Exception : No dongle found` on macOS, try running the full load command natively (i.e., outside of the Docker container) using the latest version of `ledgerblue`. Install `ledgerblue` as outlined above, then copy and paste the load command, which should look something like this:

```
> python3 -m ledgerblue.loadApp --curve ed25519 --appFlags 0xa00  --path "44'/540'" --tlv --targetId 0x33100004 --targetVersion="" --apiLevel 1 --delete --fileName bin/app.hex --appName "Spacemesh" --appVersion "0.1.0" --dataSize $((0x`cat debug/app.map |grep _envram_data | tr -s ' ' | cut -f2 -d' '|cut -f2 -d'x'` - 0x`cat debug/app.map |grep _nvram_data | tr -s ' ' | cut -f2 -d' '|cut -f2 -d'x'`)) `ICONHEX=\`python3 /opt/nanosplus-secure-sdk/icon3.py --hexbitmaponly icons/nanox_app_spacemesh.gif  2>/dev/null\` ; [ ! -z "$ICONHEX" ] && echo "--icon $ICONHEX"`
```
  * Note that the `--targetId` flag and the icon filename will differ depending which Ledger device you're using.

### Using `docker-make`

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
Requires that the `BOLOS_SDK` envvar [be set](https://developers.ledger.com/docs/embedded-app/build-app/).
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
