# Ledger Application Builder

These container images contain all dependencies to compile an application for Ledger devices

The image is stored in the following directory:

- `legacy` contains all needed tools to compile `C` and `Rust` applications. This image is quite heavy, but based on Ubuntu 22.04, so it is a good pick for tools using the `glibc`, such as `SonarQube` or `CodeQL`.

## Using Ledger image

To use or build these container images, first install Docker on you computer.

The images corresponding to the previous Dockerfiles are built and pushed on [ghcr.io](ghcr.io) every time the SDK is updated.
They can be pulled via these commands:

```bash
# pull the legacy image, built from `legacy/Dockerfile`
$ docker pull ghcr.io/karlsen-network/ledger-app-builder/ledger-app-builder-legacy:latest
```

## Compile your app in the container

The `BOLOS_SDK` variable is used to specify the target SDK, allowing to compile the application for each Ledger device.

In the source folder of your application, you can compile with the following commands:

* For Nano S
```bash
$ sudo docker run --rm -ti -v "$(realpath .):/app" --user root ghcr.io/karlsen-network/ledger-app-builder/ledger-app-builder-legacy:latest
bash$ BOLOS_SDK=$NANOS_SDK make
```

* For Nano S+
```bash
$ sudo docker run --rm -ti -v "$(realpath .):/app" --user root ghcr.io/karlsen-network/ledger-app-builder/ledger-app-builder-legacy:latest
bash$ BOLOS_SDK=$NANOSP_SDK make
```

* For Stax
```bash
$ sudo docker run --rm -ti -v "$(realpath .):/app" --user root ghcr.io/karlsen-network/ledger-app-builder/ledger-app-builder-legacy:latest
bash$ BOLOS_SDK=$STAX_SDK make
```

### Code static analysis

The Docker images include the [Clang Static Analyzer](https://clang-analyzer.llvm.org/), which can be invoked with:

```bash
$ sudo docker run --rm -ti -v "$(realpath .):/app" --user root ghcr.io/karlsen-network/ledger-app-builder/ledger-app-builder-legacy:latest
bash$ BOLOS_SDK=$NANOS_SDK make scan-build
```

## Load the app on a physical device

:warning: Only Nano S, Nano S+ and Stax devices allow application side-loading. This section will not work with a Nano X.

To load the app from the container, you will need additional docker arguments in order to allow Docker to access your USB port.
Your physical device must be connected, unlocked and the screen showing the dashboard (not inside an application). Same as for compilation, `BOLOS_SDK` variable is used to specify the target device. Use the following docker command to load the app (here for Nano S device) :

```bash
$ sudo docker run --rm -ti  -v "$(realpath .):/app" --privileged -v "/dev/bus/usb:/dev/bus/usb" --user root ghcr.io/karlsen-network/ledger-app-builder/ledger-app-builder-legacy:latest
bash$ BOLOS_SDK=$NANOS_SDK make load
```

## Build the container image

If the provided images does not suit your needs or you want to tinker with them, you can build these images yourself.


### Standard Build

Containers can be built using `Docker`:

```bash
$ (cd full && sudo docker build -t ledger-app-builder-legacy:latest .)
```

### App Scanner

Image can embed the [Coverity Scan](https://scan.coverity.com/) build tool. It is an excellent static analysis tool, and it can be very useful to find bugs in Nano apps.

The build tool must be downloaded before building the image. Archive can be downloaded from <https://scan.coverity.com/download>. Download is available to everyone, but it requires to create an account. After having registered, download Coverity Build Tool 2021.12 for Linux64 and place the downloaded archive in the `coverity` directory.

Then, build container from the `coverity/` directory with:

```bash
$ (cd full && sudo docker build -t ledger-app-scanner:latest .)
```

