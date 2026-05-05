# Local Build

Builds ZMK firmware for this Corne config using upstream Zephyr SDK, no Nordic tooling.

## One-time setup

```fish
# System packages (Arch)
sudo pacman -S --needed gperf ccache

# Workspace + venv
mkdir -p ~/personal/zmk-workspace
cd ~/personal/zmk-workspace
python3 -m venv .venv
source .venv/bin/activate.fish
pip install --upgrade pip west

# Fetch ZMK + Zephyr sources (~1-2 GB)
west init -m https://github.com/zmkfirmware/zmk --mr main --mf app/west.yml
west update
west zephyr-export
west packages pip --install

# Zephyr SDK (ARM toolchain only, ~150 MB)
west sdk install -t arm-zephyr-eabi
```

## Per-session setup

Every new shell, before running any `west` command:

```fish
cd ~/personal/zmk-workspace
source .venv/bin/activate.fish
```

## Build

```fish
# Left half
west build -s zmk/app -d build/left -b nice_nano/nrf52840/zmk -- \
  -DSHIELD=corne_left \
  -DZMK_CONFIG=/home/djunho/personal/Corne/config

# Right half
west build -s zmk/app -d build/right -b nice_nano/nrf52840/zmk -- \
  -DSHIELD=corne_right \
  -DZMK_CONFIG=/home/djunho/personal/Corne/config

# Settings reset (only when re-pairing is needed)
west build -s zmk/app -d build/reset -b nice_nano/nrf52840/zmk -- \
  -DSHIELD=settings_reset \
  -DZMK_CONFIG=/home/djunho/personal/Corne/config
```

Output: `build/<side>/zephyr/zmk.uf2`.

## Iterate

Edit `config/corne.keymap`, then rerun the same `west build` — incremental rebuild takes seconds.

For a clean rebuild, add `-p always` or delete the build dir.

## Flash

1. Double-tap reset on the controller.
2. It mounts as USB mass storage.
3. Drag `build/<side>/zephyr/zmk.uf2` onto it.

---

## ZMK Studio

Studio is enabled by default on the right half (`CONFIG_ZMK_STUDIO=y` is set in `corne_right.conf`; the snippet is wired into CI via `build.yaml`). The right half is the BLE/USB central in this config — it's the side that connects to the host. For local builds, the snippet still has to be passed on the command line:

```fish
west build -p -s zmk/app -d build/right -b nice_nano/nrf52840/zmk \
  -S studio-rpc-usb-uart \
  -- -DSHIELD=corne_right \
     -DZMK_CONFIG=/home/djunho/personal/Corne/config
```

The left half and settings-reset builds are unchanged — Studio only runs on the central (right).

**Using it:** flash the right half, open https://zmk.studio, plug the USB cable into the **right** half, and press `lower + top-right key` to unlock — that combo is bound to `&studio_unlock` on the lower layer.

**If the build fails** with a linker "region overflow" error, flash is tight; disable unused features in `corne.conf` (RGB, display, etc. are already off).
