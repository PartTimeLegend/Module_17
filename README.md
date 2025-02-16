# Module 17
Welcome to Module 17!

This project provides a standalone modem board that transforms any 9600 baud capable transceiver into an [M17](https://m17project.org/) compatible radio. It is based on a STM32F4 microcontroller and is designed to run the [OpenRTX](https://openrtx.org/) firmware. Audio and PTT connectivity is given through a "Kenwood 2 Pin" compatible connector (2.5 and 3.5mm audio jacks).

<img src="https://github.com/M17-Project/Module_17/blob/main/Module17_0.1e.jpg?raw=true" width="500">

Several boards of various revisions have been built. While the changes between r0.1a and r0.1b were quite drastic, r0.1c and d are only minor revisions, mainly for parts availability. The pinout, baseband filters and components are considered stable and will (probably) not change in future revisions, ensuring software compatibility. Revision 0.1d boards can be ordered partially or even fully assembled at [JLCPCB](https://www.jlcpcb.com) with the manufacturing data in this repository. The volume knob and digital potentiometers are only available through Mouser/Digikey, but JLCPCB can acquire them through their global sourcing program.

The board was designed with the [Waveshare 18179](https://www.waveshare.com/1.3inch-oled-module-c.htm) in mind and shares the footprint and pinout for said display.

Basic OpenRTX support is already given, but the UI needs an overhaul for Module 17. 

## Flashing
### Linux
You need `dfu-util` 0.11 to start the process. `dfu-util` installation instructions are [here](https://dfu-util.sourceforge.net/build.html). It is recommended to use the most recent version - built from source.

To flash, hold the "Escape" (upper left) button down while plugging in USB (or pressing the reset switch) to boot into DFU mode.

For your convenience, a pre-built binary is available [here](https://files.openrtx.org/nightly/) and [here](https://openrtx.schinken-radio.de/nightly/). The name of the file is `openrtx_mod17_wrap`.
You can then follow standard [OpenRTX](https://github.com/OpenRTX/OpenRTX)
flashing instructions for the "mod17" target if you have dfu-util
installed:<br>
`dfu-util -d 0483:df11 -a 0 -D openrtx_mod17_wrap -s 0x08000000`

Once flashing is complete, reset the board to boot into the newly flashed application. If there are any errors while flashing, make sure there's a valid udev rule available for the Module17. Download the [udev](https://github.com/OpenRTX/OpenRTX/blob/master/99-openrtx.rules) file and then run the following commands:<br>
`sudo cp 99-openrtx.rules /etc/udev/rules.d`<br>
`sudo udevadm control --reload-rules`.

### Windows
Be sure that you have WinUSB installed for your DFU device. You can use [Zadig](https://zadig.akeo.ie/). To enter the DFU mode, hold down the upper left button (below the display) while plugging the USB-C cable in. The button can now be released. Nothing should appear on the display at this moment. After the Module17 is connected and in DFU mode, select it from the list in Zadig, then install the driver.

- Download the [dfu-util-0.11-binaries.tar.xz dfu-util binary for Windows](https://dfu-util.sourceforge.net/releases/) and unzip.
- Download the `openrtx_mod17_wrap` file, it's available [here](https://files.openrtx.org/nightly/) and [here](https://openrtx.schinken-radio.de/nightly/).
- Navigate to `{extraction_location}\dfu-util-0.11-binaries\win64` (or `win32`) and copy the wrap file there. Example path: `C:\Users\SP5WWP\Desktop\dfu-util-0.11-binaries\win64`.
- Run the command prompt (`cmd`) as administrator and change the working directory with `cd {extraction_location}\dfu-util-0.11-binaries\win64`.
- Run the following command: `dfu-util -d 0483:df11 -a 0 -D openrtx_mod17_wrap -s 0x08000000`
- After the flashing is complete, close the command prompt and reset the device.

### Building the firmware yourself
Building instructions are available [at the OpenRTX project's website](https://openrtx.org/#/compiling).<br>

## Usage (rev 0.1d/e)
### Power supply
The modem can be supplied with:
- a 6..15V source via the DC plug (upper right hand side) or pin 9 of the DE9 connector (upper left hand side),
- 5V through the USB-C connector (in the middle of the upper side)
Both inputs can be used at the same time.

### DE-9 connector
The DE-9 connector at the top of the board can be used to connect Module17 to a 9600baud radio. The pinout is shown in the table below.
| Pin      |             Function             |             Direction            | `CT-167` cable wire color |
|----------|:--------------------------------:|----------------------------------|---------------------------|
| 1        |  unused (floating)               |  --                              |                           |
| 2        |  baseband output (towards radio) |  output                          |  brown                    |
| 3        |  CAT-RX                          |  input                           |                           |
| 4        |  CAT-TX                          |  output                          |                           |
| 5        |  radio PTT                       |  output, open-drain, low-active  |  red                      |
| 6        |  baseband input (from radio)     |  input                           |  orange                   |
| 7        |  unused (floating)               |  --                              |                           |
| 8        |  ground                          |  --                              |  black, thick             |
| 9        |  12V input                       | input (supply)                   |                           |

![CT-167 Wiring](./hardware/wiring/CT-167/CT-167.png)

Additionally, there is a 2.54mm pin header just next to the DE-9 connector that can be used for convenient access to the baseband, PTT and CAT signals. Pin 9 doesn't have to be used, it only provides an alternative way for supplying the board.  

### Kenwood mic-speaker connector
On the left hand side of the board, there is a Kenwood-type connector. The pinout is standard and most microphone-speakers should be compatible with Module17.

### Volume knob with a power switch
The volume knob is at the bottom of the board. As the name suggests, it is used for audio volume setting. It also acts as the power switch for the module.

**Note** - the power switch is on the 12V line, so it is not possible to turn the device off while it is powered with 5V (USB).<br>
**Warning** - due to leakage current reaching one of the transistors in the power control section, the `Shutdown` menu entry doesn't work as intended.<br>
Workaround: solder a 10k resistor between STM32's pin 50 or D7's pin 1 (`GPIO_POWER` net) and ground.

### Transmission/reception
At idle, the device will look for valid M17 signal in the baseband. If there is a valid signal detected carrying voice data, it will be decoded and sent to the speaker at the Kenwood  connector. There is also an additional, unpopulated, 2-pin speaker connector in the lower left corner of the modem. It can be used to connect an external >=8ohms speaker.
Transmission is triggered by the PTT key of the mic-speaker. A valid baseband along with a PTT signal is then sent to the radio.

## Revisions
### Revision 0.1a
This is the initial prototyping revision and not yet designed with "ease of use and enclosing" in mind. Its main purpose is getting OpenRTX up and running on the hardware and providing a proof of concept.

#### Errata of Revision 0.1a
* Do not install R23 (1k5 pull-up on USB D+) as the STM32F4 has an integrated pull-up
* Replace C3 and C6 with 18pF, otherwise Y1 starts too slow during USB DFU
* Short BOOT0 on pin 94 to PB8 on pin 95 to be able to use SW4 in software
* The 2.5mm mono jack used for the Kenwood 2-pin speaker mic has the "sleeve" connection where stereo plugs have the "ring". This means that the speaker mic has no GND connection. To fix this, add a bodge wire to the outer shell of the mono jack to GND. Next revisions will use a stereo jack with shorted ring and sleeve connections.

### Revision 0.1b
Fixed some bugs:
* Changed audio amp from LM386 to LM4861MX
* Mic amplifier changed from LMV341 to MAX9814 (with an AGC)
* Fixed Kenwood connector
* Fixed baseband processing at U6/U7
* C3/C6 value change from 22 to 18p
* Changed micro from STM32F407VETx to -405RGTx
* Added TPS5431 DC/DC converter IC
* Added MAX3232 and `CAT_TX`/`CAT_RX` at DE9 connector

### Revision 0.1c
* Reduced audio amplifier gain (R3 value change 510k->22k)
* Fixed `RADIO_RX` baseband DC bias

### Revision 0.1d
* Changed supply voltage for U6 and U7 (baseband filter Op-Amps) to 5V and replaced MCP602 with MCP6002 as the former ICs are not rail2rail capable on their input
* Fixed `CAT_TX` and `CAT_RX` solder jumpers
* Fixed volume knob direction
* Replaced PSU with in-stock parts
* Replaced MAX3232 with simple 5V/3.3V level shifters as most, if not all, radios use either 5V or 3.3V TTL instead of RS232
* Replaced STM32F405RGT6 with GD32F405RGT6 - these ICs are pin and code compatible

### Revision 0.1e
* Replaced digital potentiometers with multiturn ones to improve parts availability
* Moved the TX potentiometer from the op-amp's output to between STM32 and the first op-amp
* Added power-on button
* Fixed misc BOM errors and removed a few unnecessary components from the schematic

#### Errata of Revision 0.1e
~~R31 has too high value. Add a 2.2k resistor across it (in parallel) to increase the sound volume coming from the speaker. Alternatively, replace R31 with a 1.8k or 2k resistor.~~ Resolved.

### Future revisions
Future revisions will feature at least additional power supply options and will be designed with an enclosure in mind.

## Acknowledgments
This project is inspired by David Rowe's [SM1000](https://www.rowetel.com/wordpress/?p=3125) Codec2 smart mic and shares some of its circuitry. The analog baseband filtering is inspired by the [MMDVM_RPT_Hat](https://github.com/mathisschmieder/MMDVM_RPT_Hat) and is a simplified version of F0DEI's original filter design that is used with great success at many MMDVM repeater sites.

## License
Module 17 is licensed under the [TAPR Open Hardware License](https://tapr.org/the-tapr-open-hardware-license/)
