

[HamTec-IC7100-Photo-Console.zip](https://github.com/user-attachments/files/33265456/HamTec-IC7100-Photo-Console.zip)

[HamTec-IC7100-GitHub-Ready.zip](https://github.com/user-attachments/files/33265461/HamTec-IC7100-GitHub-Ready.zip)

# HamTec IC-7100 Photo Console

A Raspberry Pi Linux control panel for the **Icom IC-7100**, using a real IC-705 photograph as an interactive radio chassis.

**Version 0.3.1 · Raspberry Pi 4 · Debian 13 ARM64**

The LCD is overlaid with live controls, the photographed VFO knob tunes the IC-7100, and the interface scales with the browser window. The surrounding page is black.

![Actual IC-705 photograph used by the interface](web/assets/ic705.png)

*Source photograph, not a screenshot of the running application. Credit: Batchman99 / VORON SPb, CC BY-SA 4.0.*

## Features

- Live USB CI-V frequency and mode control.
- Draggable VFO dial, mouse-wheel tuning and keyboard controls.
- FIL1 / FIL2 / FIL3 selection and readback.
- Direct MHz entry and receive band presets.
- Live received-audio spectrum and waterfall.
- Window fitting, size slider and resize handle.
- Photographic radio view and flat screen view.
- Clickable photographed MENU, M.SCOPE, QUICK and EXIT buttons.
- Automatic discovery of a uniquely identified IC-7100 CI-V interface.
- No demo mode or simulated radio readings.

## Requirements

- Raspberry Pi 4, a suitable Pi power supply and a monitor or touchscreen.
- Debian 13 ARM64 with a desktop and Chromium. Other distributions are unverified.
- IC-7100, powered normally, with its original control head connected.
- USB-A to mini-B **data** cable from the Pi to the radio main unit.
- Internet access for installing dependencies.

## Install

Download the repository ZIP from GitHub using **Code → Download ZIP**, then extract it. Open a terminal in the extracted project folder containing `server.py` and run:

```bash
sudo apt update
sudo apt install -y python3-venv libportaudio2
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
bash start-panel.sh
```

Open **http://127.0.0.1:7100** in Chromium on the Pi. Keep the server terminal open. Stop it with **Ctrl+C**.

For later launches, open a terminal in the same project folder and run:

```bash
bash start-panel.sh
```

## Connect the IC-7100

The default radio settings are **19200 baud** and **CI-V address 88h**. Match these in the radio's Set menu or override them:

```bash
bash start-panel.sh --baud 19200 --address 88
```

The address is hexadecimal. The application prefers a uniquely matching IC-7100 A interface in `/dev/serial/by-id`. If detection fails or several radios are attached, identify the devices:

```bash
ls -l /dev/serial/by-id/
```

Then supply the actual CI-V interface:

```bash
bash start-panel.sh \
  --port /dev/serial/by-id/YOUR_ICOM_CIV_DEVICE \
  --baud 19200 \
  --address 88
```

Select the CI-V interface rather than the second data/GPS interface. Close competing applications using that port. Restart the server after reconnecting USB.

If serial access is denied:

```bash
sudo usermod -aG dialout "$USER"
```

Log out and back in, or reboot. Run the application as your normal user.

## Live audio waterfall

On the radio select **ACC/USB Output Select: AF**, then adjust the USB AF level. Automatic input selection requires a uniquely matching Icom/USB audio capture device.

If automatic selection fails:

```bash
.venv/bin/python -m sounddevice
```

Find the input-device index, stop the server and restart with that number. Here `3` is only an example:

```bash
bash start-panel.sh --audio-device 3
```

The capture input must support mono 48 kHz. Audio-device indices can change after reconnecting USB.

**This is an AF waterfall, not a wideband RF scope.** The display processes received audio up to 3 kHz, not the IC-7100's 12 kHz IF output. The existing USB connection does not provide the IC-705's native wideband RF spectrum. No generated signals are substituted when audio is missing.

## Controls

| Control | Action |
| --- | --- |
| VFO knob | Drag clockwise/counter-clockwise or scroll to tune |
| Frequency display | Enter MHz; scroll to tune |
| Mode box | Select radio mode |
| FIL box | Select FIL1, FIL2 or FIL3 |
| MENU / BAND | Select a receive band preset |
| + / − or up/down arrows | Tune by the selected step |
| QUICK photographed button | Cycle tuning step |
| HOLD | Freeze waterfall history; radio polling continues |
| CENT/FIX | Clear AF waterfall history |
| SPAN / REF | Adjust AF span and display reference |
| EXPD/SET or M.SCOPE | Expand the scope |
| SIZE / resize handle | Resize the panel |
| FIT | Restore automatic fitting |
| SCREEN VIEW / RADIO VIEW | Switch between flat LCD and photographic chassis |
| EXIT photographed button | Close open dialogs |
| MENU → SET | View connection and audio errors |
| MENU → FULL SCREEN | Toggle fullscreen |

Filter preset widths are configured on the IC-7100 for the current mode. This application selects presets; it does not edit or assume their bandwidths. The large VFO dial appears in radio view; screen view retains tuning buttons and keyboard controls. Other photographed controls are decorative.

## Kiosk display

With the server running, use another terminal on the Pi desktop:

```bash
chromium --kiosk http://127.0.0.1:7100
```

Some installations name the browser `chromium-browser`. **Alt+F4** closes the kiosk window. Automatic startup is not configured.

## Troubleshooting

| Problem | Check |
| --- | --- |
| Browser cannot connect | Keep the server running and open the URL on the Pi itself |
| Port already in use | Stop the previous server or use `--http-port 7101` |
| Radio not detected | Supply the correct serial path with `--port` |
| Permission denied | Join `dialout`, then log out/in |
| No CI-V reply | Check interface, baud, address, power and competing apps |
| Dial animates but frequency does not change | Read the connection errors in MENU → SET |
| Filter rejected | Check preset support for the radio's current mode |
| No audio waterfall | Check AF output, audio level and input-device selection |
| Panel too large | Click FIT or reduce SIZE |
| Old UI appears | Refresh using Ctrl+Shift+R |

When using `--http-port 7101`, open **http://127.0.0.1:7101**. The server is bound to loopback and is not configured for remote network access.

## Status and limits

The author reports that the application runs on their Raspberry Pi 4 and IC-7100. Individual feature operation and other configurations have not been independently verified. No new runtime tests were performed for this public packaging update.

Not implemented: live S-meter measurement, AGC/preamp/attenuator/NR control, arbitrary filter-bandwidth editing, VFO A/B selection, wideband RF input, speaker playback, transmit/PTT and automatic startup. Unavailable values remain blank or marked unavailable.

## Licenses

Original application code and documentation: **MIT**, see [LICENSE](LICENSE).

IC-705 photograph and adaptations of that photograph: **CC BY-SA 4.0**, see [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md) and [photo attribution](web/assets/ATTRIBUTION.md). The photograph is not covered by the application's MIT license.

This is independent software, not official Icom software.

## References

- [IC-7100 product page and manuals](https://www.icomjapan.com/lineup/products/IC-7100EUR/)
- [IC-705 photograph source](https://commons.wikimedia.org/wiki/File:ICOM_IC-705_nobg.png)

CI-V commands 04 and 06 carry mode and filter selection: 01=FIL1, 02=FIL2, 03=FIL3.
