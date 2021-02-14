# mxiot
mxiot is a low-cost hardware prototyping platform to help you explore switching small IoT projects from bare-metal to a secure-boot-capable WiFi/BT-connected Embedded Linux system capable of running rudimentary C/C++, as well as applications written in almost any modern language or application framework (like Qt, Rust, Ruby, Python, Node.js, or .NET Core).

The heart of mxiot is the i.MX6ULZ, a 900 MHz Cortex-A7 microprocessor available [direct from NXP for $2.68](https://www.nxp.com/part/MCIMX6Z0DVM09AB#/) (even in single quantities). These are drop-in compatible with the i.MX6UL and i.MX6ULL.

![Alt text](/pinout.svg?raw=true "mxiot pinout")

### On-Board Peripherals
The i.MX6ULZ is stripped down to the basics, but there's plenty there for most always-on IoT projects. The board exposes:
 - 3 UARTs (one with hardware flow-control signals, and one dedicated to the console)
 - Two I2C peripherals (one shared with one of the UARTs)
 - SPI with 2 chip-select signals
 - I2S with separate BCLK/WS signals for TX and RX paths.
 - 4x PWM outputs
 - 4x 12-bit analog inputs

WiFi and BLE are handled by whatever 44-pin standard SDIO-interfaced WiFi/BT *module du jour* you'd like to populate; the board supports all physically-compatible Ampak modules, as well as extremely low-cost alternatives, like the RTL8723BS modules regularly found for [around $2 from AliExpress](https://www.aliexpress.com/wholesale?catId=0&initiative_id=SB_20210214115323&SearchText=rtl8723bs), Taobao, or other distributors in Asia.

The design is open-source, and more importantly, the BOM has wide availability and doesn't lean on ICs with limited documentation or support. This means you can prototype your project around the mxiot and then copy the guts into your own design to optimize for size, power consumption, cost, or expanded functionality.

### Supports 4-layer and 6-layer stack-ups
For the sake of compactness, mxiot tightly arranges parts onto a dense 6-layer PCB measuring only 2.3 x 0.8 inches (58 x 20mm). Having said that, I've done many lower-cost 4-layer designs with mostly single-sided placements around the same guts of this board without any routing issues.

### Security
I'm a hardware guy, not a cybersecurity consultant. It's really easy to roll your eyes when people start talking about secure-boot and key storage, but seriously, folks, if you are making devices that you're going to sell to people that will connect to the internet, you have an ethical obligation to think through the security implications. You should be prototyping around hardware that has the underlying capabilities that allow you to turn on security measures during the design cycle.

The i.MX6's OTP key storage, TrustZone, and secure-boot capabilities let you establish a chain of trust from the boot ROM, to U-Boot, to the kernel, and an encrypted rootfs, which will allow you to safely store private data (like keys!) without having to worry about physical device security (you can, indeed, continue to boot from MicroSD cards). You can also transport firmware updates over unencrypted channels and not have to worry about your image being reverse-engineered or modified.

### What about...?
 - *...the Raspberry Pi Zero?* While it has plenty of DRAM, the older ARM11 processor is incompatible with newer application frameworks. There's also no pathway to using the processor in a custom design, and no secure boot capability.
 - *...the Allwinner V3s or F1C100s SIPs?* These are fine for C/C++ applications or very simple Python scripts, but the V3s lacks sufficient RAM to run large, modern application frameworks, and the F1C200s is an older ARM9 cores that have dwindling software support and no security features.
 - *...the Microchip SAMA5D27 or SAM9X60 SIPs?* These both cost twice as much as the combined cost of the i.MX6ULZ and 512MB DDR3, while offering inferior performance as well as the software compatibility issues mentioned above.
 - *...the OSD32MP15x or OSD335x modules?* Unless you're literally assembling no more than a handful of units for the entirety of the project's lifecycle, these are way too expensive to justify the time savings when compared to an iMX6ULZ design.
 