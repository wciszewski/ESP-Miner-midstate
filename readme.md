# About This Fork

This fork allows to pass an externally computed midstate to the Bitaxe Max. The midstate is the internal state of SHA-256 after updating it with the first 512 bits of the block header.

Below is the original readme of ESP-Miner with additional complete instructions for building and flashing the firmware for Bitaxe Max.

[![](https://dcbadge.vercel.app/api/server/3E8ca2dkcC)](https://discord.gg/osmu)

![GitHub Downloads (all assets, all releases)](https://img.shields.io/github/downloads/bitaxeorg/esp-miner/total)
![GitHub commit activity](https://img.shields.io/github/commit-activity/t/bitaxeorg/esp-miner)
![GitHub contributors](https://img.shields.io/github/contributors/bitaxeorg/esp-miner)
![Alt](https://repobeats.axiom.co/api/embed/70889479b1e002c18a184b05bc5cbf2ed3718579.svg "Repobeats analytics image")

# ESP-Miner
esp-miner is open source ESP32 firmware for the [Bitaxe](https://github.com/bitaxeorg/bitaxe)

If you are looking for premade images to load on your Bitaxe, check out the [latest release](https://github.com/bitaxeorg/ESP-Miner/releases/latest) page. Maybe you want [instructions](https://github.com/bitaxeorg/ESP-Miner/blob/master/flashing.md) for loading factory images.

# Bitaxetool
We also have a command line python tool for flashing Bitaxe and updating the config called Bitaxetool 

**Bitaxetool Requires Python3.4 or later and pip**

Install bitaxetool from pip. pip is included with Python 3.4 but if you need to install it check <https://pip.pypa.io/en/stable/installation/>

```
pip install --upgrade bitaxetool
```
The bitaxetool includes all necessary library for flashing the binaries to the Bitaxe Hardware.

- Flash a "factory" image to a Bitaxe to reset to factory settings. Make sure to choose an image built for your hardware version (401) in this case:

```
bitaxetool --firmware ./esp-miner-factory-401-v2.4.2.bin
```
- Flash just the NVS config to a bitaxe:

```
bitaxetool --config ./config-401.cvs
```
- Flash both a factory image _and_ a config to your Bitaxe: note the settings in the config file will overwrite the config already baked into the factory image:

```
bitaxetool --config ./config-401.cvs --firmware ./esp-miner-factory-401-v2.4.2.bin
```

## AxeOS API
The esp-miner UI is called AxeOS and provides an API to expose actions and information.

For more details take a look at [`main/http_server/openapi.yaml`](./main/http_server/openapi.yaml).

Things that can be done are:
  
  - Get System Info
  - Get System Statistics
  - Update Swarm
  - System Restart Action
  - Update System Settings Action
  - System Options
  - Update OTA Firmware
  - Update OTA WWW
  - Logs over WebSocket

Some API examples in curl:
  ```bash
  # Get system information
  curl http://YOUR-BITAXE-IP/api/system/info
  ```
  ```bash
  # Get system statistics
  curl http://YOUR-BITAXE-IP/api/system/statistics
  ```
  ```bash
  # System restart action
  curl -X POST http://YOUR-BITAXE-IP/api/system/restart
  ```

## Administration

The firmware hosts a small web server on port 80 for administrative purposes. Once the Bitaxe device is connected to the local network, the admin web front end may be accessed via a web browser connected to the same network at `http://<IP>`, replacing `IP` with the LAN IP address of the Bitaxe device, or `http://bitaxe`, provided your network supports mDNS configuration.

### Recovery

In the event that the admin web front end is inaccessible, for example because of an unsuccessful firmware update (`www.bin`), a recovery page can be accessed at `http://<IP>/recovery`.

### Unlock Settings

In order to unlock the Input fields for ASIC Frequency and ASIC Core Voltage you need to append `?oc` to the end of the settings tab URL in your browser. Be aware that without additional cooling overclocking can overheat and/or damage your Bitaxe.

## Development

### Prerequisites

- Install the ESP-IDF toolchain from https://docs.espressif.com/projects/esp-idf/en/stable/esp32/get-started/
- Install nodejs/npm from https://nodejs.org/en/download
- (Optional) Install the ESP-IDF extension for VSCode from https://marketplace.visualstudio.com/items?itemName=espressif.esp-idf-extension

### Complete Build and Flash Process for Bitaxe Max

This assumes you have installed bitaxetool and the prerequisites as described above.

Depending on how you installed `npm`, ESP-IDF may not see it because it seems to ignore `.bashrc`. As a workaround, ensure that `npm` is added to `PATH` in `.profile`, for example like this:
```
if [ -d "$HOME/.nvm/versions/node/v22.18.0/bin" ] ; then
    PATH="$HOME/.nvm/versions/node/v22.18.0/bin:$PATH"
fi
```

To compile the firmware, run the following:
```bash
# Only need to do this once in a shell. Change 5.5 to your version of ESP-IDF
. ~/esp/v5.5/esp-idf/export.sh

# Only need to do this once in the project
idf.py set-target esp32s3

# Alternatively you can use the "build" button in VSCode
idf.py build

# This step doesn't seem documented; it was taken from .github/workflows/release-factory.yml
~/esp/v5.5/esp-idf/components/nvs_flash/nvs_partition_generator/nvs_partition_gen.py generate config-102.cvs config.bin 0x6000

# -c to use the binary config generated above (also taken from the github workflow)
./merge_bin.sh -c midstate_override.bin
```

To flash it, connect the Bitaxe Max to your computer via USB and run:
```bash
bitaxetool --firmware midstate_override.bin
```

Below are general building and flashing instructions with some additional information. Note that this fork's modifications are only for Bitaxe Max (with BM1397).

### Building

At the root of the repository, run:
```
idf.py build && ./merge_bin.sh ./esp-miner-merged.bin
```

Note: the merge_bin.sh script is a custom script that merges the bootloader, partition table, and the application binary into a single file.

Note: if using VSCode, you may have to configure the settings.json file to match your esp hardware version. For example, if your bitaxe has something other than an esp32-s3, you will need to change the version in the `.vscode/settings.json` file.

### Flashing

With the bitaxe connected to your computer via USB, run:

```
bitaxetool --config ./config-xxx.cvs --firmware ./esp-miner-merged.bin
```

where xxx is the config file for your hardware version. You can see the list of available config files in the root of the repository.

A custom board version is also possible with `config-custom.cvs`. A custom board needs to be based on an existing `devicemodel` and `asicmodel`.

Note: if you are developing within a dev container, you will need to run the bitaxetool command from outside the container. Otherwise, you will get an error about the device not being found.

## Attributions

The display font is Portfolio 6x8 from https://int10h.org/oldschool-pc-fonts/ by VileR.
