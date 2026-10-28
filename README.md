# Gremsy Firmware Debug Tools

A cross-platform desktop application for STM32 firmware flashing, serial debugging, interactive VT100 shell execution, real-time data visualization, and remote SFTP/UART file management.

---

## Table of Contents

- [Interface Overview](#interface-overview)
- [System Requirements](#system-requirements)
- [Quick Start Workflows](#quick-start-workflows)
  - [STM32 Firmware Flashing](#stm32-firmware-flashing)
  - [Serial and Interactive Shell](#serial-and-interactive-shell)
  - [Real-Time Data Plotter](#real-time-data-plotter)
  - [Remote SSH and File Transfer](#remote-ssh-and-file-transfer)
- [Feature Details](#feature-details)
  - [Hardware Pin Control](#hardware-pin-control)
  - [Log Keyword Highlighting](#log-keyword-highlighting)
  - [Real-Time Plotter Layouts](#real-time-plotter-layouts)
  - [Remote File Management](#remote-file-management)
  - [Chip Security Tools](#chip-security-tools)
- [Menu Reference](#menu-reference)
- [Keyboard Shortcuts](#keyboard-shortcuts)
- [Troubleshooting](#troubleshooting)

---

## Interface Overview

```
+-------------------------------------------------------------------------------+
|  Menu: Tools | View | Preferences | Help                                      |
+------------------------------------+------------------------------------------+
|  LEFT PANEL (Config & Actions)     |  RIGHT PANEL (Terminals & Visualization) |
|  - 1. Connection: Serial / SSH     |  - Action Toolbar: FTP, Find, Save, Rec  |
|  - 2. Pin Control: DTR/RTS Pins    |  - Tabs: [System Log] [UART] [SSH]       |
|  - 3. Firmware Upgrade: .hex Flash |  - Visual Panel: Terminal + Plotter      |
|  - 4. Target Device Info: Chip ID  |  - Bottom: Send Command Row & Presets    |
+------------------------------------+------------------------------------------+
```

- **Left Panel**: Configure COM port / SSH, map hardware control pins (DTR/RTS), select `.hex` firmware, and inspect target MCU identity.
- **Right Panel**: Real-time System Log, interactive VT100 UART/SSH terminals, and split-screen real-time data plotter.
- **Top Toolbar**: Toggle Sidebar, open Remote File Transfer (`FTP`), Find (`Ctrl+F`), Save Log (`Ctrl+Shift+S`), Record (`Ctrl+R`), Clear (`Ctrl+L`), and Plotter (`Ctrl+P`).

---

## System Requirements

| Operating System | Requirements & Packages | Quick Launch |
| --- | --- | --- |
| **Linux** (Ubuntu/Debian) | Python >= 3.8, `python3-tk`, `tkdnd`, `zenity` | `./run.sh` |
| **Linux** (Fedora) | Python >= 3.8, `python3-tkinter`, `tkdnd`, `zenity` | `./run.sh` |
| **Linux** (Arch) | Python >= 3.8, `tk`, `tkdnd`, `zenity` | `./run.sh` |
| **Windows** (10/11) | Python >= 3.8 | `run.bat` |

> [!NOTE] On Linux, running `./run.sh` automatically installs system packages, creates a virtual environment, installs Python dependencies, adds your user to the `dialout` group, and creates a desktop shortcut.

---

## Quick Start Workflows

### STM32 Firmware Flashing

1. **Select Port**: Under **1. Connection Settings**, choose your serial port and baudrate (default: `115200`). Click **Scan** (`Ctrl+S`) if port is not listed.
2. **Verify Hardware Pins**: In **2. Hardware Pin Control**, ensure pin mappings match your hardware (Default: `DTR -> Reset`, `RTS -> Boot0`, Active Low). If using an inverting transistor circuit, check **Active High**.
3. **Select Firmware**: Drag and drop your `.hex` file into **3. Firmware Upgrade** or click **Browse...** (`Ctrl+B`).
4. **Flash**: Click **Start Flashing** (`Ctrl+D`). The tool enters bootloader mode, erases memory, flashes pages, verifies integrity, and resets MCU back to run mode.

### Serial and Interactive Shell

1. In **1. Connection Settings**, click **Open Port** (`Ctrl+O`).
2. Switch to the **UART Terminal** tab.
3. Select preferred mode:
   - **Log Mode**: Chronological streaming with millisecond timestamps and color tags.
   - **Interactive Shell Mode**: Full VT100 ANSI terminal emulation with cursor navigation and color rendering (ideal for FreeRTOS CLI, U-Boot, NuttX).
   - **HEX / BIN Mode**: Raw byte stream inspection.
4. Type in the bottom input bar, choose line ending (`\r\n` or `\n`) and format (ASCII/HEX/BIN), then press **Enter**.

### Real-Time Data Plotter

1. Format firmware UART output as comma-separated numeric values:
   ```c
   // Firmware C example:
   printf("%.2f,%.2f,%.2f\r\n", roll, pitch, yaw);
   ```
2. Open serial port and click **Plot** (`Ctrl+P`) on the toolbar.
3. The plotter automatically detects column count after receiving 5 valid lines and streams multi-channel curves in real time.

### Remote SSH and File Transfer

1. In **1. Connection Settings**, expand **Remote SSH**, input Host IP, User, and Password/Key, then click **Connect SSH**.
2. Switch to **SSH Terminal** tab for remote command-line access.
3. Click **FTP** on the toolbar to open the remote file manager:
   - **Via SSH (SFTP)**: Full bidirectional file navigation, directory download/upload, and deletion.
   - **Via UART**: Serial console file transfer for embedded Linux devices via Base64 chunked streaming with MD5 verification.

---

## Feature Details

### Hardware Pin Control

Controls hardware lines via FTDI/CP2102 serial handshake lines (DTR and RTS):
- **Pin Mapping**: Assign Reset/Boot to `DTR`, `RTS`, or `None`.
- **Active High**: Check this box if your board uses inverting NPN/MOSFET reset transistors.
- **Action Buttons**:
  - **Boot Mode** (`Ctrl+1`): Drives BOOT0 active and issues a reset pulse to enter bootloader.
  - **Run Mode** (`Ctrl+2`): Releases BOOT0 and issues a reset pulse for normal firmware execution.
  - **Reset Pulse** (`Ctrl+3`): Sends an active 100ms pulse on NRST.
  - **Halt MCU** (`Ctrl+4`): Holds NRST asserted to freeze the MCU.

> [!NOTE] While **Halt MCU** is active, flashing and chip erase operations are blocked until the MCU is released.

### Log Keyword Highlighting

Open via **Tools > Configure Log Keywords...**:
- Add custom keywords and assign foreground/background highlight colors.
- Real-time dynamic tagging across System Log and UART Log.
- Keywords and color schemes persist automatically in application configuration.

### Real-Time Plotter Layouts

Toggle with **Plot** (`Ctrl+P`) to reveal the bottom analysis panel:
- **5 Layout Modes**:
  - `Grid`: 2D multi-plot matrix.
  - `List`: Vertically stacked subplots.
  - `Overlay`: All channels on a single shared axes.
  - `Distribution`: Histogram probability density analysis.
  - `Spectrum`: FFT frequency spectrum (Linear, Log, or PSD in dB/Hz).
- **Controls**: Pause/Resume streaming, sample window buffer (500 to 100,000 samples), per-channel visibility checkboxes, and custom channel color picker.

### Remote File Management

Click **FTP** on toolbar:
- Dual-pane file manager supporting SFTP (over SSH) and UART Chunked Streaming (over Serial).
- Real-time progress bar with speed calculation and MD5 checksum verification.
- Supports drag-and-drop file transfers, directory recursive uploads, and breadcrumb path navigation.

### Chip Security Tools

Accessible under **Tools** menu:
- **Disable Write Protection (WRP)**: Unlocks write-protected flash sectors and reboots target MCU.
- **Disable Readout Protection (RDP)**: Clears STM32 chip readout protection for debugging access.

> [!WARNING] Disabling Readout Protection (RDP) triggers a hardware mass erase! All flash memory will be wiped immediately.

---

## Menu Reference

| Menu | Item | Description |
| --- | --- | --- |
| **Tools** | New Window (`Ctrl+N`) | Open an independent application window |
| | Configure Log Keywords... | Open custom keyword highlight manager |
| | Reset UART Config | Restore baudrate to 115200 and 8N1 framing |
| | Reset to Defaults | Reset all saved settings to factory state |
| | Disable Write Protection (WRP) | Remove sector write protection |
| | Disable Readout Protection (RDP) | Unlock protected MCU (triggers mass erase) |
| **View** | Log Font Size | Adjust terminal font size (8 pt to 20 pt) |
| | Max Log Lines | Set terminal buffer capacity (1,000 to Unlimited) |
| | Update Interval | UI refresh rate (Realtime 10ms to 500ms) |
| | UI Scaling | High-DPI scaling factor (Auto, 100% to 200%) |
| **Preferences** | Configure Shortcuts... | Custom keybinding assignment dialog |
| | Toggle Dark/Light Mode | Switch between Ubuntu Dark and Light themes |
| | Show/Hide System Ports | Toggle display of `/dev/ttyS*` ports |
| | UART Display Mode | Default mode: ASCII Log, Shell, HEX, BIN |
| | UART Transmit Format | Default format: ASCII, HEX, BIN |
| **Help** | User Guide | Open in-app documentation viewer |
| | Check for Updates... | Check and apply Git updates with auto-restart |
| | About | Version, system info, and dependencies |

---

## Keyboard Shortcuts

| Shortcut | Action | Shortcut | Action |
| --- | --- | --- | --- |
| `Ctrl+O` | Open / Close Serial Port | `Ctrl+1` | Force MCU into Boot Mode |
| `Ctrl+S` | Scan Serial Ports | `Ctrl+2` | Force MCU into Run Mode |
| `Ctrl+B` | Browse Firmware (.hex) | `Ctrl+3` | Send Reset Pulse (100ms) |
| `Ctrl+D` | Start Flashing | `Ctrl+4` | Toggle Halt MCU |
| `Ctrl+E` | Erase Chip | `Ctrl+P` | Toggle Real-time Plotter |
| `Ctrl+Q` | Cancel Active Operation | `Ctrl+T` | Open Command Presets |
| `Ctrl+F` | Toggle Search Bar | `Ctrl+R` | Toggle Disk Recording |
| `Ctrl+L` | Clear Terminal Buffer | `Ctrl+Shift+S` | Save Terminal Log to File |

> [!TIP] All shortcuts can be customized via **Preferences > Configure Shortcuts...**.

---

## Troubleshooting

- **Permission Denied on Linux (`/dev/ttyUSB*` or `/dev/ttyACM*`)**:
  ```bash
  sudo usermod -aG dialout $USER
  ```
  Log out and log back in, or run `./run.sh` (which handles permissions automatically).

- **STM32 Bootloader Connection Fails (NACK / Timeout)**:
  - Check [Hardware Pin Control](#hardware-pin-control): Verify DTR (NRST) and RTS (BOOT0) wiring.
  - Transistor inversion: Toggle **Active High** in Hardware Pin Control.
  - Manual Bootloader entry: Hold `BOOT0` button, press and release `RESET`, then release `BOOT0`. Click **Start Flashing**.
  - If target is locked, use [Chip Security Tools](#chip-security-tools) (**Tools > Disable Readout Protection (RDP)**).

- **Missing GUI Packages on Linux**:
  - Ubuntu/Debian: `sudo apt-get install -y python3-tk tkdnd zenity`
  - Fedora: `sudo dnf install -y python3-tkinter tkdnd zenity`
  - Arch Linux: `sudo pacman -S --noconfirm tk tkdnd zenity`

- **SSH / SFTP Connection Fails**:
  - Ensure target has `sshd` running on port 22 (`systemctl status sshd`).
  - Verify IP address and SSH private key permissions (`chmod 600 key.pem`).

---

## License

Copyright © Gremsy JSC. All rights reserved.
