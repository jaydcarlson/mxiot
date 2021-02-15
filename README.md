# mxiot
mxiot is a low-cost hardware prototyping platform to help you explore switching small IoT projects from bare-metal to a secure-boot-capable WiFi/BT-connected Embedded Linux system capable of running rudimentary C/C++, as well as applications written in almost any modern language or application framework (like Qt, Rust, Ruby, Python, Node.js, or .NET Core).

The heart of mxiot is the i.MX6ULZ, a 900 MHz Cortex-A7 microprocessor available [direct from NXP for $2.68](https://www.nxp.com/part/MCIMX6Z0DVM09AB#/) (even in single quantities). These are drop-in compatible with the i.MX6UL and i.MX6ULL.

![Alt text](/docs/pinout.svg?raw=true "mxiot pinout")

### Features
The i.MX6ULZ is stripped down to the basics, but there's plenty there for most always-on IoT projects. The board exposes:
 - 3 UARTs (one with hardware flow-control signals, and one dedicated to the console)
 - Two I2C peripherals (one shared with one of the UARTs)
 - SPI with 2 chip-select signals
 - I2S with separate BCLK/WS signals for TX and RX paths.
 - 4x PWM outputs
 - Two USB ports

In addition to the DRAM and SD Card socket necessary for booting, mxiot also sports a small complement of on-board peripherals:
 - WiFi and Bluetooth connectivity
 - 4x 12-bit analog inputs, care of the Texas Instruments TLA2024
 - APA102 addressable RGB LED
 - QSPI Flash
 - Pushbutton

WiFi and BLE are handled by whatever 44-pin standard SDIO-interfaced WiFi/BT *module du jour* you'd like to populate; the board supports all physically-compatible Ampak modules, as well as extremely low-cost alternatives, like the RTL8723BS modules regularly found for [around $2 from AliExpress](https://www.aliexpress.com/wholesale?catId=0&initiative_id=SB_20210214115323&SearchText=rtl8723bs), Taobao, or other distributors in Asia.

The design is open-source, and more importantly, the BOM has wide availability and doesn't lean on ICs with limited documentation or support. This means you can prototype your project around the mxiot and then copy the guts into your own design to optimize for size, power consumption, cost, or expanded functionality.

### What if I don't have Altium?
You can [download a ZIP of the repo](https://github.com/jaydcarlson/mxiot/archive/master.zip) and upload this file directly to [Altium's online viewer](https://www.altium.com/viewer/) to interact with the PCB, schematic, BOM, and 3D model of the design without having to download any software. The repo also contains a schematic and gerbers for offline viewing.

### Supports 4-layer and 6-layer stack-ups
For the sake of compactness, mxiot tightly arranges parts onto a dense 6-layer PCB measuring only 2.3 x 0.8 inches (58 x 20mm). Having said that, I've done many lower-cost 4-layer designs with mostly single-sided placements around the same guts of this board without any routing issues.

### Security
I'm a hardware guy, not a cybersecurity consultant, so I know that it's really easy to roll your eyes when people start talking about secure-boot and key storage. But seriously, folks, if you are making devices that you're going to sell to people that will connect to the internet, you have an ethical obligation to think through the security implications. You should be prototyping around hardware that has the underlying capabilities that allow you to turn on security measures during the design cycle.

The i.MX6's OTP key storage, TrustZone, and secure-boot capabilities let you establish a chain of trust from the boot ROM, to U-Boot, to the kernel, and an encrypted rootfs, which will allow you to store private data (like keys!) without having to worry as much about physical device security. You can also transport firmware updates over unencrypted channels and not have to worry about your image being reverse-engineered or modified.

### What about...?
 - *...the Raspberry Pi Zero?* While it has plenty of DRAM, the older ARM11 processor is incompatible with newer application frameworks. There's also no pathway to using the processor in a custom design, and no secure boot capability.
 - *...the Allwinner V3s or F1C100s SIPs?* These are fine for C/C++ applications or very simple Python scripts, but the V3s lacks sufficient RAM to run large, modern application frameworks, and the F1C200s is an older ARM9 cores that have dwindling software support and no security features.
 - *...the Microchip SAMA5D27 or SAM9X60 SIPs?* These both cost twice as much as the combined cost of the i.MX6ULZ and 512MB DDR3, while offering inferior performance as well as the software compatibility issues mentioned above.
 - *...the OSD32MP15x or OSD335x modules?* Unless you're literally assembling no more than a handful of units for the entirety of the project's lifecycle, these are way too expensive to justify the time savings when compared to an iMX6UL/ULL/ULZ design.

### What if I need an LCD, CSI, Ethernet, CAN, or industrial temp grades?
Grab the mxiot schematics as a starting point, but solder down a full-featured i.MX6ULL. These come in tons of different SKUs that mix and match peripherals, operating frequencies, and temperature grades. All of these are 100% electrically and mechanically compatible (in fact, mxiot started out as a 528 MHz i.MX6UL dev board from 5 years ago!).

# Hardware Status
 - [x] i.MX6ULZ
 - [x] DRAM stress tests
 - [x] RTL8723BS WiFi
 - [x] TLA2024 ADC
 - [x] USB port
 - [x] JTAG port
 
Not tested yet:
 - [ ] RTL8723BS BT
 - [ ] APA102 RGB LED
 - [ ] Pushbutton
 - [ ] QSPI Flash 

# Booting Linux
While you could use mxiot as a (very bad) single-board computer, it's designed for embedded tasks. Most of the sensors and peripherals you'll use with this board will require kernel and DTS modifications; I recommend setting up a project-specific Buildroot tree you can work out of to build a U-Boot, Linux kernel, and rootfs with the specific packages you need.

I don't have published patches yet, but you can largely use an unmodified i.MX6ULZ EVK port to get mxiot to boot. 

## Buildroot
Buildroot doesn't have an i.MX6ULZ defconfig, but we can start with an i.MX6ULL one and go from there
```
mkdir mxiot && cd mxiot
git clone https://git.busybox.net/buildroot/
cd buildroot
make imx6ullevk_defconfig
make menuconfig
```
Change **Kernel > In-tree Device Tree Source file names** to imx6ulz-14x14-evk

Go to **Bootloaders**. Change **Build system** to **Kconfig** and **Board defconfig** to **mx6ulz_14x14_evk**

## U-Boot
The big change we have to make is to switch SDHC2 to SDHC1 everywhere, since mxiot boots from the first MMC device (which is required to use SD/MMC Manufacture Mode), while the i.MX6UL, ULL, and ULZ EVKs all boot from SDHC2.

If you're in Buildroot, you can access the U-Boot menuconfig with `make uboot-menuconfig`. Make sure to enable the FSL USDHC MMC driver (`CONFIG_FSL_USDHC=y`) — I've noticed some defconfigs don't activate this driver by default. Also, change **Environment > mmc device number** (`CONFIG_SYS_MMC_ENV_DEV`) to 0.

You'll have to hack at `u-boot/include/configs/mx6ullevk.h` to switch the default SDHC2 boot device to SDHC1. These two changes should do the trick:
```
//#define CONFIG_SYS_FSL_ESDHC_ADDR   USDHC2_BASE_ADDR
#define CONFIG_SYS_FSL_ESDHC_ADDR     USDHC1_BASE_ADDR

...

//#define CONFIG_MMCROOT              "/dev/mmcblk1p2"
#define CONFIG_MMCROOT                "/dev/mmcblk0p2"
```
## Linux
There are lots of changes to make here depending on what you want your board to do. If you just want to get to a login and poke around with userspace tools, the default kernel and device tree should work.

Otherwise, here are some notes to get the onboard hardware working:

### RTL8723BS WiFi / Bluetooth module
To get WiFi working, assuming you've soldered down an RTL8723BS module, choose `CONFIG_RTL8723BS=m`

Bluetooth requires activating three-wire HCI support in the Linux kernel (`BT_HCIUART_3WIRE`) and using the appropriate [firmware and hciattach software available from lwfinger](https://github.com/lwfinger/rtl8723bs_bt).

### Analog Inputs
The 4 analog inputs are provided by an external I2C ADC, the TLA2024, which is on I2C2 at address 0x48. To bring this sensor up, edit your DTS (by default, `linux/arch/arm/boot/dts/imx6ulz-14x14-evk.dts`) to include it:
```
&i2c2 {
    ...
	tla2024: adc0@48 {
		compatible = "ti,tla2024";
		reg = <0x48>;

		v0@0 {
			single-channel = <0>;
		};

		v1@1 {
			single-channel = <1>;
		};

		v2@2 {
			single-channel = <2>;
		};

		v3@3 {
			single-channel = <3>;
		};
	};
};
```
And apply [this patch](https://patchwork.kernel.org/project/linux-iio/patch/1553608583-29006-1-git-send-email-ibtsam.haq.0x01@gmail.com/) and activate the new driver.

Other than the two I2C buses, if you want to use any of the I/O pins for their peripheral functions, or get audio working, you'll have to enable the appropriate peripherals in your DTS file and/or enable specific drivers in the Linux kernel.