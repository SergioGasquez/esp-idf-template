# Rust on ESP-IDF "Hello, World" template

A "Hello, world!" template, to use with [cargo-generate](https://github.com/cargo-generate/cargo-generate), of a Rust binary crate for the ESP-IDF framework.

This is the crate you get when running `cargo new`, but augmented with extra configuration so that it does build for the ESP32[XX] with ESP-IDF and (by default) with STD support.

Or if you rather
* ... want to mix Rust and C/C++ in a traditional ESP-IDF `idf.py` CMake project - [follow these instructions](README-cmake.md)
* ... want to mix Rust and C/C++ with PlatformIO - [follow these instructions](README-pio.md)

To generate a project using this template:
```sh
cargo generate https://github.com/esp-rs/esp-idf-template cargo
```
After running the command, there will be a few prompts:
- `Project Name`: Name of the crate.
- `Which MCU to target?`: SoC model.
- `STD support`: When `true`, adds support for [Rust Standard Library](https://doc.rust-lang.org/std/). Otherwise, we will use [Rust Core Library](https://doc.rust-lang.org/core/index.html).
- `ESP-IDF Version`: ESP-IDF branch/tag to use. Possible choices:
  - [`v.4.4`](https://github.com/espressif/esp-idf/tree/release/v4.4): Stable
  - [`v.4.3.2`](https://github.com/espressif/esp-idf/tree/v4.3.2): Previous stable
  - [`mainline`](https://github.com/espressif/esp-idf/tree/master): **Unstable**
- `Dev Containers support?`: Adds support for:
    -  [VS Code Dev Containers](https://code.visualstudio.com/docs/remote/containers#_quick-start-open-an-existing-folder-in-a-container)
    -  [GitHub Codespaces](https://docs.github.com/en/codespaces/developing-in-codespaces/creating-a-codespace)
    -  [Gitpod](https://www.gitpod.io)
  Dev Containers also have integration with [Wokwi simulator](https://wokwi.com/) and allow flashing from the container using [web flash](https://github.com/bjoernQ/esp-web-flash-server).

![CI](https://github.com/esp-rs/esp-idf-template/actions/workflows/ci.yml/badge.svg)

## Prerequisites

### Install Rustup

If you don't have `rustup` installed yet, follow the instructions on the [rustup.rs site](https://rustup.rs)

### Install Rust & Clang - for Xtensa MCUs (ESP32, ESP32-S2 and ESP32-S3)

- Install the [Rust Espressif compiler toolchain and the Espressif LLVM Clang toolchain](https://github.com/esp-rs/rust-build)
- This is necessary, because support for the Xtensa architecture (ESP32 / ESP32-S2 / ESP32-S3) is not upstreamed in LLVM yet
- Make sure that you DON'T have a system Clang installed as well, because even if you have the Espressif one first on your `$PATH`, Bindgen will still pick the system one
  - A workaround that does not require uninstalling the system Clang is to do `export LIBCLANG_PATH=<path to the Espressif Clang lib directory>` prior to continuing the build process

### Install Rust & Clang - for RiscV32 MCUs (ESP32-C3)

- You **can** target the ESP32-C3 with the Espressif toolchains just fine, but this MCU is also supported by the stock compilers
- So alternatively - just use the stock nightly Rust compiler, and a recent, stock Clang (as in Clang 11+)

To install the nightly Rust compiler toolchain:
```sh
rustup install nightly
rustup component add rust-src --toolchain nightly
```

Installing a recent Clang compiler is OS-specific. The [Clang Getting Started page](https://clang.llvm.org/get_started.html) contains useful guidelines.

### Install Python3

You need a Python 3.7 or later installed on your machine. Install it from the package distro of your OS, or download and install [from the official Python site](https://www.python.org/downloads/).

### Install Cargo Sub-Commands

```sh
cargo install cargo-generate
cargo install ldproxy
cargo install espflash
cargo install cargo-espflash
```
> **Note**
> If you are running macOS or Linux then libuv must also be installed for `espflash` and `cargo-espflash`; this is available via most popular package managers. If you are running Windows you can ignore this step.
> ```
> # macOS
> brew install libuv
> # Debian/Ubuntu/etc.
> apt-get install libuv-dev
> # Fedora
> dnf install systemd-devel
> ```
> Also, the `espflash` and `cargo-espflash` commands shown below, assume that version `2.0` or
> greater.

## Generate the project

```sh
cargo generate --vcs none --git https://github.com/esp-rs/esp-idf-template cargo
cd <your-project-name>
```

## Build and Flash
`cargo-espflash` allows to build the project and flash it to your device:
```sh
cargo espflash flash /dev/ttyUSB0
```
- Replace `dev/ttyUSB0` above with the USB port where you've connected the board. If you do not
specify any USB port, `cargo-espflash` will print a list of the recognized USB ports for you to select
the desired port.
- You can include the `--monitor` argument to the `cargo-espflash` command to open a serial monitor after flashing the device.
- For more details on [`cargo-espflash` usage see the README](https://github.com/esp-rs/espflash/tree/main/cargo-espflash#usage)

## Build

To build using the default ESP-IDF native builder just use:
```sh
cargo build
```

- Replace `<your-project-name>` with the name of the generated project

To build using the PlatformIO builder, use:
```sh
cargo build --features pio
```

## Flash

In the root of the generated project:

```sh
espflash flash /dev/ttyUSB0 target/[xtensa-esp32-espidf|xtensa-esp32s2-espidf|xtensa-esp32s3-espidf|riscv32imc-esp-espidf]/debug/<your-project-name>
```

- Replace `dev/ttyUSB0` above with the USB port where you've connected the board. If you do not
specify any USB port, `espflash` will print a list of the recognized USB ports for you to select
the desired port.
- Replace `<your-project-name>` with the name of the generated project
- You can include the `--monitor` argument to the `espflash` command to open a serial monitor after flashing the device.
- For more details on [`espflash` usage see the README](https://github.com/esp-rs/espflash/tree/main/espflash#usage)

## Monitor
Both `espflash` and `cargo-espflash` allows monitoring a serial port withouth flashing the device:
```sh
espflash monitor /dev/ttyUSB0
```
Or
```sh
cargo espflash monitor /dev/ttyUSB0
```


- Replace `dev/ttyUSB0` above with the USB port where you've connected the board. If you do not
specify any USB port, `cargo-espflash`/`espflash` will print a list of the recognized USB ports for you to select
the desired port.

The monitor should output more or less the following:
```
Opening /dev/tty.usbserial-0001 with speed 115200
Resetting device... done
ets Jun  8 2016 00:22:57

rst:0x1 (POWERON_RESET),boot:0x13 (SPI_FAST_FLASH_BOOT)
configsip: 0, SPIWP:0xee
clk_drv:0x00,q_drv:0x00,d_drv:0x00,cs0_drv:0x00,hd_drv:0x00,wp_drv:0x00
mode:DIO, clock div:2
load:0x3fff0048,len:12
ho 0 tail 12 room 4
load:0x3fff0054,len:4800
load:0x40078000,len:17448
load:0x4007c428,len:4840
entry 0x4007c6a0
I (178) cpu_start: Pro cpu up.
I (178) cpu_start: Starting app cpu, entry point is 0x4008115c
I (0) cpu_start: App cpu up.
I (193) cpu_start: Pro cpu start user code
I (193) cpu_start: cpu freq: 160000000
I (193) cpu_start: Application information:
I (197) cpu_start: Project name:     esp-idf
I (202) cpu_start: App version:      f08dcd7
I (207) cpu_start: Compile time:     Oct 23 2021 14:48:03
I (213) cpu_start: ELF file SHA256:  0000000000000000...
I (219) cpu_start: ESP-IDF:          4.3.0
I (224) heap_init: Initializing. RAM available for dynamic allocation:
I (231) heap_init: At 3FFAE6E0 len 00001920 (6 KiB): DRAM
I (237) heap_init: At 3FFB3498 len 0002CB68 (178 KiB): DRAM
I (243) heap_init: At 3FFE0440 len 00003AE0 (14 KiB): D/IRAM
I (250) heap_init: At 3FFE4350 len 0001BCB0 (111 KiB): D/IRAM
I (256) heap_init: At 4008C538 len 00013AC8 (78 KiB): IRAM
I (263) spi_flash: detected chip: generic
I (267) spi_flash: flash io: dio
I (272) cpu_start: Starting scheduler on PRO CPU.
I (0) cpu_start: Starting scheduler on APP CPU.
Hello, world!
```
