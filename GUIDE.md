# DSpico Guide
This guide aims to document step by step how to setup your DSpico.

## Prerequisites
1. Ensure you have a Linux or WSL (Windows Subsystem for Linux) environment set up.
2. Install [BlocksDS](https://blocksds.skylyrac.net/docs/setup/)
3. Install [.NET 9.0](https://learn.microsoft.com/en-us/dotnet/core/install/linux-ubuntu-install?tabs=dotnet9&pivots=os-linux-ubuntu-2404) for your environment (note: this link points to the instructions for Ubuntu, but links for most OS'es are available on the same page)
4. Install a few prerequesite packages: `sudo apt install cmake gcc-arm-none-eabi build-essential git`
5. Set an environment variable for dlditool: `export DLDITOOL=/opt/wonderful/thirdparty/blocksds/core/tools/dlditool/dlditool`

## 1. Getting and assembling your DSpico
Refer to the [DSpico hardware repository](https://github.com/LNH-team/dspico-hardware) for all information about ordering and assembling a DSpico.

## 2. Compiling the DSpico DLDI
1. Clone the [DSpico DLDI repository](https://github.com/LNH-team/dspico-dldi)
2. Run `make`

You should now have `DSpico.dldi`.

## 3. Compiling the DSpico Bootloader
1. Clone the [DSpico Bootloader repository](https://github.com/LNH-team/dspico-bootloader)
2. Initialize the submodules: `git submodule update --init`
3. Run `make`

You should now have `BOOTLOADER.nds`.

4. DLDI patch the bootloader: `$DLDITOOL /path/to/DSpico.dldi BOOTLOADER.nds`
5. Clone the [DSRomEncryptor repository](https://github.com/Gericom/DSRomEncryptor)
6. Compile DSRomEncryptor by running: `dotnet build`

The compiled executable can be found at `DSRomEncryptor/DSRomEncryptor/bin/Debug/net9.0/`, in the project's folder.

7. Ensure files containing the NTR and TWL blowfish tables are present in the same folder as the DSRomEncryptor executable (see also the DSRomEncryptor readme)
8. Run `DSRomEncryptor /path/to/BOOTLOADER.nds default.nds`

You should now have `default.nds`.

## 4. Optional: Compiling Wrfuxxed
Follow these steps if you want to use the Wrfuxxed exploit to boot the DSpico on unmodified DSi and 3DS systems.
1. Clone the [Wrfuxxed repository](https://github.com/LNH-team/dspico-wrfuxxed)
2. Run `make`
3. DLDI patch the exploit: `$DLDITOOL /path/to/DSpico.dldi uartBufv060.bin`

## 5. Compiling the DSpico Firmware
1. Clone the [DSpico Firmware repository](https://github.com/LNH-team/dspico-firmware)
2. Initialize the submodules:
    ```bash
    git submodule update --init
    cd pico-sdk
    git submodule update --init
    cd ..
    ```
    Note that you shouldn't use `--recursive` because it draws in a lot of unnecessary submodules inside the pico-sdk.
3. Copy `default.nds` from step 3 to the `roms` folder of the firmware repository
4. Optional if you want to use Wrfuxxed:
    1. Copy the WRFU Tester v0.60 rom to `roms/dsimode.nds` (SHA-1: `2d65fb7a0c62a4f08954b98c95f42b804fccfd26`)
    2. Copy `uartBufv060.bin` to the `data` folder
    3. Uncomment (remove the `#`) the `DSPICO_ENABLE_WRFUXXED` option in `CMakeLists.txt`
5. Run `./compile.sh`. If you get an error saying that there are missing permissions,  run `chmod +x compile.sh` and retry.

The `build` folder should now contain `DSpico.uf2`.

## 6. Flashing the DSpico
1. Boot the DSpico in BOOTSEL mode.
    - If the rp2040 was not flashed before, it should automatically enter BOOTSEL mode when it is connected to a PC via USB.
    - If the rp2040 already has the DSpico Firmware, eject the micro SD card and it should automatically enter BOOTSEL mode when it is connected to a PC via USB.
    - Otherwise, hold the BOOTSEL button while connecting to a PC via USB to enter BOOTSEL mode.
2. Copy `DSpico.uf2` to the USB drive that appeared when connecting the DSpico.
3. Disconnect the DSpico from the PC.

## 7. Compiling Pico Loader
1. Clone the [Pico Loader repository](https://github.com/LNH-team/pico-loader)
2. Initialize the submodules: `git submodule update --init`
3. Run `make`

You should now have the following files:
- `picoLoader7.bin`
- `picoLoader9_DSPICO.bin`
- `data/aplist.bin`
- `data/savelist.bin`
- `data/patchlist.bin`

## 8. Compiling Pico Launcher
1. Clone the [Pico Launcher repository](https://github.com/LNH-team/pico-launcher)
2. Initialize the submodules: `git submodule update --init`
3. Run `make`

You should now have `LAUNCHER.nds`.

## 9. Prepare the micro SD card
1. If necessary, format your micro SD card using these steps: https://dsi.cfw.guide/sd-card-setup.html.
> [!WARNING]
> Do NOT use the built-in formatting tool of Windows!
2. Copy the `_pico` folder from the Pico Launcher repository to the root of the micro SD card.
3. Copy `LAUNCHER.nds` to `/_picoboot.nds`.
4. Copy `picoLoader7.bin` to `/_pico/picoLoader7.bin`.
5. Copy `picoLoader9_DSPICO.bin` to `/_pico/picoLoader9.bin`.
6. Copy `aplist.bin` to `/_pico/aplist.bin`.
7. Copy `savelist.bin` to `/_pico/savelist.bin`.
8. Copy `patchlist.bin` to `/_pico/patchlist.bin`.
9. Copy any DS roms to your micro SD card. It is recommended to make a folder to put them in.

The final directory structure will look like this:
```
.
├── _pico
│   ├── themes
│   │   ├── material
│   │   │   └── theme.json
│   │   └── raspberry
│   │       ├── bannerListCell.bin
│   │       ├── bannerListCellPltt.bin
│   │       ├── bannerListCellSelected.bin
│   │       ├── bannerListCellSelectedPltt.bin
│   │       ├── bottombg.bin
│   │       ├── gridcell.bin
│   │       ├── gridcellPltt.bin
│   │       ├── gridcellSelected.bin
│   │       ├── gridcellSelectedPltt.bin
│   │       ├── scrim.bin
│   │       ├── scrimPltt.bin
│   │       ├── theme.json
│   │       └── topbg.bin
│   ├── aplist.bin
│   ├── savelist.bin
│   ├── patchlist.bin
│   ├── picoLoader7.bin
│   └── picoLoader9.bin
└── _picoboot.nds
```
Note: If you want to play DSiWare on the DSpico, additional files are required. See the Pico Loader readme for more information.

## 10. Test your DSpico
1. Insert the micro SD card in your DSpico.
2. Insert the DSpico in your DS, DSi or 3DS device.
3. Turn on the device.
4. The DSpico should appear on the menu, of if you are using Wrfuxxed on a DSi, Pico Launcher should start automatically after going through the exploit.
5. If you're not using Wrfuxxed on a DSi, launch the DSpico from the menu.
6. Pico Launcher should start and shows the contents of your micro SD card.
7. Navigate to a DS rom and press A to start it.

## Troubleshouting

### The DSpico is not detected by my DS, DSi or 3DS system or hangs at the DS(i) splash screen
- Check that the bootloader was properly encrypted with the correct blowfish keys.
- Check that the firmware was compiled in `RelWithDebInfo` mode.

### Wrfuxxed doesn't work, WRFU Tester stays on screen
- Ensure the correct version of WRFU Tester is being used (v0.60).
- Ensure that the `DSPICO_ENABLE_WRFUXXED` option is enabled in the firmware `CMakeLists.txt`.

### I get "*Error: Failed to mount SD card.*", or a blue screen after the Wrfuxxed exploit
- Mounting your micro SD card failed. Check if there's anything wrong with it, try reformatting (see Step 9) or try a different micro SD card. In case of a blue screen, make sure the exploit has been patched with the proper DLDI driver.

### I get "*ERROR: Failed to open Pico Loader file.*", or a red screen after the Wrfuxxed exploit
- `/_pico/picoLoader9.bin` or `/_pico/picoLoader7.bin` could not be opened. Check that you copied all necessary files to the micro SD card (see Step 9).
