# Local Build

Builds ZMK firmware for this Corne repository using upstream Zephyr SDK.

## One-time setup

```fish
# System packages (Arch)
sudo pacman -S --needed gperf ccache

# Workspace + venv
mkdir -p ~/personal/zmk-workspace
cd ~/personal/zmk-workspace
python3 -m venv .venv
source .venv/bin/activate.fish
pip install --upgrade pip west protobuf==7.36.1 grpcio-tools

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
set -x ZEPHYR_TOOLCHAIN_VARIANT zephyr
set -x ZEPHYR_SDK_INSTALL_DIR /home/djunho/zephyr-sdk-0.17.0
```

*(If using bash, use `export ZEPHYR_TOOLCHAIN_VARIANT=zephyr` and `export ZEPHYR_SDK_INSTALL_DIR=/home/djunho/zephyr-sdk-0.17.0`)*

---

## Build: Corne V3 Choc (PandaKB PCB)

Builds firmware for the PandaKB PCB Corne V3 (with 128x32 OLED display and 27 RGB LEDs per half).

```fish
# Left half (Peripheral)
west build -s zmk/app -d build/pcb_left -b nice_nano/nrf52840/zmk -- \
  -DSHIELD=corne_left \
  -DZMK_CONFIG=/home/djunho/personal/Corne/config

# Right half (Central with ZMK Studio)
west build -s zmk/app -d build/pcb_right -b nice_nano/nrf52840/zmk \
  -S studio-rpc-usb-uart -- \
  -DSHIELD=corne_right \
  -DZMK_CONFIG=/home/djunho/personal/Corne/config
```

---

## Build: Corne Handwired

Builds firmware for the original handwired Corne board.

```fish
# Left half (Peripheral)
west build -s zmk/app -d build/hw_left -b nice_nano/nrf52840/zmk -- \
  -DSHIELD=corne_hw_left \
  -DZMK_CONFIG=/home/djunho/personal/Corne/config

# Right half (Central with ZMK Studio)
west build -s zmk/app -d build/hw_right -b nice_nano/nrf52840/zmk \
  -S studio-rpc-usb-uart -- \
  -DSHIELD=corne_hw_right \
  -DZMK_CONFIG=/home/djunho/personal/Corne/config
```

---

## Settings Reset

When re-pairing halves or resetting Bluetooth bonding keys:

```fish
west build -s zmk/app -d build/reset -b nice_nano/nrf52840/zmk -- \
  -DSHIELD=settings_reset \
  -DZMK_CONFIG=/home/djunho/personal/Corne/config
```

Output files: `build/<target>/zephyr/zmk.uf2`.

---

## Iterate

Edit keymap or configuration files, then rerun the same `west build` — incremental rebuilds take only seconds.

For a clean rebuild from scratch, add `-p always` or remove the build directory.

---

## Flash

1. Double-tap the reset button on the controller (nice!nano / SuperMini).
2. The controller mounts as a USB mass storage drive.
3. Drag and drop the corresponding `.uf2` file onto the drive.

---

## ZMK Studio

Studio is enabled by default on the right half (`CONFIG_ZMK_STUDIO=y`). The right half acts as the BLE/USB Central in this configuration — it is the half that connects directly to the host computer.

**How to use:**
1. Connect the **right** half to your computer via USB.
2. Open https://zmk.studio in a Chromium-based browser.
3. Press `&studio_unlock` (on Layer 3 / adjust layer, top-left or top-right key) to unlock live editing.

---

## Visual Keymap Diagram (keymap-drawer)

To regenerate the visual SVG diagram for `config/corne.keymap` locally without pushing to GitHub:

```bash
uvx --from keymap-drawer keymap -c keymap_drawer.config.yaml parse -z config/corne.keymap > img/corne.yaml && \
uvx --from keymap-drawer keymap -c keymap_drawer.config.yaml draw img/corne.yaml -o img/corne.svg
```

To preview the generated SVG:

```bash
xdg-open img/corne.svg
```

