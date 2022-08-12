# Custom device tree overlays for Beagle Bone (Black) running nerves

 The recent release of the  [delux](https://hex.pm/packages/delux)  library raised my interest in device tree overlays again. delux only works with LEDs known to the  `led`  subsystem in Linux. Like many others I was driving LEDs manually using  [Circuits.GPIO](https://github.com/elixir-circuits/circuits_gpio). However device tree overlays allow you to logically “convert” GPIOs into LEDs — at least from the kernel point of view — not physically of course. This article shows you how to create those overlays and how to integrate them into your custom nerves Beagle Bone system.
 
## Device tree overlay to “convert” GPIOs into LEDs

On many SBC platforms (including the Beagle Bone family) the physically available external pins are muxed. I.e. they can be configured to perform different functions (e.g. GPIO, I2C, PMW, ADC …).

This configuration can be changed during runtime (to a limited extend) using tools like  `/usr/bin/config-pin`  — e.g. configuring a GPIO for input or output or configure pull-up or pull-down resistors).

Device tree overlays are used by the Linux kernel to configure hardware incl. GPIO pins. In addition to properties like input/output use or pull up/down resistors you can also change by which kernel sub system a pin is handled. E.g. the following device tree overlay configures and converts four pins from GPIO use to LED. I.e. from the kernel point of view they are not GPIOs anymore, but LEDs accessible under  `/sys/class/leds/`:

```dts
/dts-v1/;
/plugin/;

#include <dt-bindings/gpio/gpio.h>
#include <dt-bindings/pinctrl/am33xx.h>

&{/chosen} {
	overlays {
		MYFW_GPIO_LEDs.mycompany.com-overlays = __TIMESTAMP__;
	};
};

&ocp {
	P8_08_pinmux { status = "disabled"; };	/* P8_08: GPIO67, gpio2, 3 */
	P8_10_pinmux { status = "disabled"; };	/* P8_10: GPIO68, gpio2, 4 */
	P8_12_pinmux { status = "disabled"; };	/* P8_12: GPIO44, gpio1, 12 */
	P8_14_pinmux { status = "disabled"; };	/* P8_14: GPIO26, gpio0, 26 */ 
};

&am33xx_pinmux {
	bb_gpio_led_pins: pinmux_bb_gpio_led_pins {
		pinctrl-single,pins = <
			AM33XX_PADCONF(AM335X_PIN_GPMC_OEN_REN, PIN_OUTPUT_PULLDOWN, MUX_MODE7)	/* P8_08: Red LED */
			AM33XX_PADCONF(AM335X_PIN_GPMC_WEN, PIN_OUTPUT_PULLDOWN, MUX_MODE7)		/* P8_10: Green LED */
			AM33XX_PADCONF(AM335X_PIN_GPMC_AD12, PIN_OUTPUT_PULLDOWN, MUX_MODE7)	/* P8_12: Blue LED */
			AM33XX_PADCONF(AM335X_PIN_GPMC_AD10, PIN_OUTPUT_PULLDOWN, MUX_MODE7)	/* P8_14: Yellow LED */
		>;
	};
};

&{/} {
	leds {
		pinctrl-names = "default";
		pinctrl-0 = <&bb_gpio_led_pins>;

		compatible = "gpio-leds";

		myfw_red_led {
			label = "myfw:red:indicator";
			gpios = <&gpio2 03 GPIO_ACTIVE_HIGH>;
			default-state = "off";
		};
		myfw_green_led {
			label = "myfw:green:indicator";
			gpios = <&gpio2 04 GPIO_ACTIVE_HIGH>;
			default-state = "off";
		};
		myfw_blue_led {
			label = "myfw:blue:indicator";
			gpios = <&gpio1 12 GPIO_ACTIVE_HIGH>;
			/* Blink LED at 4 Hz (125 ms on, off) */
			linux,default-trigger = "timer";
			default-state = "on";
			led-pattern = <125 125>;
		};
		myfw_yellow_led {
			label = "myfw:yellow:indicator";
			gpios = <&gpio0 26 GPIO_ACTIVE_HIGH>;
			default-state = "off";
		};

	};
};
```

### Header & includes

```dts
/dts-v1/;
/plugin/;

#include <dt-bindings/gpio/gpio.h>
#include <dt-bindings/pinctrl/am33xx.h>
```

The first line of the overlay tells the device tree overlay compiler  `dtc`  which version of the overlay syntax we’re using. The second line informs dtc that this is not a complete device tree but just an add-on/overlay. Lastly we’ll include some defintions/macros from the Linux kernel which whill simplify the overlay later.

### Visibility in sysfs

```dts
&{/chosen} {
	overlays {
		MYFW_GPIO_LEDs.mycompany.com-overlays = __TIMESTAMP__;
	};
};
```

This part tell Linux to create a entry called  `MYFW_GPIO_LEDs.mycompany.com-overlays`  under  `/sys/firmware/devicetree/base/chosen/overlays`  if this overlay is loaded/active. The content of the file will be the timestamp of the preprocessor run (`__TIMSTAMP__`  is actually a C preprocessor macro):

```elixir
iex(1)> cmd "ls -alh /sys/firmware/devicetree/base/chosen/overlays"
-r--r--r--    1 root     root           9 Aug 11 09:20 name
-r--r--r--    1 root     root          25 Aug 11 09:20 BB-BONE-eMMC1-01-00A0.bb.org-overlays
-r--r--r--    1 root     root          25 Aug 11 09:20 MYFW_GPIO_LEDs.mycompany.com-overlays
-r--r--r--    1 root     root          25 Aug 11 09:20 BB-HDMI-TDA998x-00A0.bb.org-overlays
drwxr-xr-x    3 root     root           0 Aug 11 09:20 ..
drwxr-xr-x    2 root     root           0 Aug 11 09:20 .
0       
iex(2)> cat "/sys/firmware/devicetree/base/chosen/overlays/MYFW_GPIO_LEDs.mycompany.com-overlays"
Wed Aug 10 13:18:16 2022
```

### Disable pin-muxing for On-Chip-Peripherals

```dts
&ocp {
	P8_08_pinmux { status = "disabled"; };	/* P8_08: GPIO67, gpio2, 3 */
	P8_10_pinmux { status = "disabled"; };	/* P8_10: GPIO68, gpio2, 4 */
	P8_12_pinmux { status = "disabled"; };	/* P8_12: GPIO44, gpio1, 12 */
	P8_14_pinmux { status = "disabled"; };	/* P8_14: GPIO26, gpio0, 26 */ 
};
```

This part disables the pin muxing for four GPIO pins.

### GPIO pin names

Please note that physical GPIO pins on the pin headers are known by different names by different subsystems. E.g. pin #8 on pin header 8 (`P8_08`) is known as  `GPIO 67`  to Linux. It’s also the fourth pin (zero index based) on the third GPIO chip (`gpio2, 3`). If you have the GPIO number (e.g.  `67`) it’s easy to get the chip/pin number as each chip controls 32 pins. As  `67`  equals  `2 * 32 + 3`  GPIO 67 equals  `gpio2, 3`  — that is the fourth pin on the third chip. You can also check the assignments in  [two PDFs](https://github.com/derekmolloy/boneDeviceTree/tree/master/docs)  on  [Derek Molloys site](http://derekmolloy.ie/beaglebone/beaglebone-gpio-programming-on-arm-embedded-linux/). The column  `GPIO No.`  is the Linux GPIO Number (e.g.  `67`). The column  `Head_pin`  is the physical pin (e.g.  `P8_08`, pin header 8, pin 8). The column  `Mode7`  provides the chip number and pin offset (e.g.  `gpio2[3]`).

### Configuring GPIOs

```dts
&am33xx_pinmux {
	bb_gpio_led_pins: pinmux_bb_gpio_led_pins {
		pinctrl-single,pins = <
			AM33XX_PADCONF(AM335X_PIN_GPMC_OEN_REN, PIN_OUTPUT_PULLDOWN, MUX_MODE7)	/* P8_08: Red LED */
			AM33XX_PADCONF(AM335X_PIN_GPMC_WEN, PIN_OUTPUT_PULLDOWN, MUX_MODE7)		/* P8_10: Green LED */
			AM33XX_PADCONF(AM335X_PIN_GPMC_AD12, PIN_OUTPUT_PULLDOWN, MUX_MODE7)	/* P8_12: Blue LED */
			AM33XX_PADCONF(AM335X_PIN_GPMC_AD10, PIN_OUTPUT_PULLDOWN, MUX_MODE7)	/* P8_14: Yellow LED */
		>;
	};
};
```

This part configures the four pins as GPIOs (`MUX_MODE7`) and activates pull-down resistors (`PIN_OUTPUT_PULLDOWN`). Please note that this again uses a different pin name! GPIO  `67`  equals  `AM335X_PIN_GPMC_OEN_REN`.  `AMX335X`  is the prefix for the Beagle Bone proccessor.  `PIN_GPMC_OEN_REN`  is the uppercase identifier from column  `Mode0`  in the PDFs mentioned above.

### Converting GPIOs to LEDs

```dts
&{/} {
	leds {
		pinctrl-names = "default";
		pinctrl-0 = <&bb_gpio_led_pins>;

		compatible = "gpio-leds";

		myfw_red_led {
			label = "myfw:red:indicator";
			gpios = <&gpio2 03 GPIO_ACTIVE_HIGH>;
			default-state = "off";
		};
		myfw_green_led {
			label = "myfw:green:indicator";
			gpios = <&gpio2 04 GPIO_ACTIVE_HIGH>;
			default-state = "off";
		};
		myfw_blue_led {
			label = "myfw:blue:indicator";
			gpios = <&gpio1 12 GPIO_ACTIVE_HIGH>;
			/* Blink LED at 4 Hz (125 ms on, off) */
			linux,default-trigger = "timer";
			default-state = "on";
			led-pattern = <125 125>;
		};
		myfw_yellow_led {
			label = "myfw:yellow:indicator";
			gpios = <&gpio0 26 GPIO_ACTIVE_HIGH>;
			default-state = "off";
		};

	};
};
```

This part finally allows us to “convert” GPIOs to LEDs. I.e. The Linux  `led`  subsystem will use GPIO pins to switch LEDs.

```plain
&{/} {
	leds {
		pinctrl-names = "default";
		pinctrl-0 = <&bb_gpio_led_pins>;

		compatible = "gpio-leds";
---8<-----SNIP-----8<-----SNIP-----8<-----SNIP--
```

Here we’ll referencing the  `led`  branch root of the device tree (`&{/}`). So we’re informing the kernel about LEDs we have … or will have … We’ll also specify to use control the Beagle Bone GPIO pins (`<&bb_gpio_led_pins>`). The  `compatible = "gpio-leds";`  line tell the kernel that these “LEDs” will be controlled by GPIOs. Placing this under the  `led`  subsystem together with the  `compatible`  settings leads to the  `led`  subsystem taking care of those “LEDs” by controlling them via GPIO. The only thing missing now is actually defining LEDs and the GPIOs they can be controlld with:

### Defining GPIO based LEDs

```plain
		myfw_red_led {
			label = "myfw:red:indicator";
			gpios = <&gpio2 03 GPIO_ACTIVE_HIGH>;
			default-state = "off";
		};
		myfw_green_led {
			label = "myfw:green:indicator";
			gpios = <&gpio2 04 GPIO_ACTIVE_HIGH>;
			default-state = "off";
		};
		myfw_blue_led {
			label = "myfw:blue:indicator";
			gpios = <&gpio1 12 GPIO_ACTIVE_HIGH>;
			/* Blink LED at 4 Hz (125 ms on, off) */
			linux,default-trigger = "timer";
			default-state = "on";
			led-pattern = <125 125>;
		};
		myfw_yellow_led {
			label = "myfw:yellow:indicator";
			gpios = <&gpio0 26 GPIO_ACTIVE_HIGH>;
			default-state = "off";
		};
```

This finally defines the actual LEDs. Each LED has a logical label which needs to be unique within the device tree (branch) — e.g.  `myfw_red_led`.

The  `label`  key will define the name of the LED in the  `sys`  filesystem. The general naming scheme is  `devicename:color:function`  as defined in  [LED handling under Linux](https://docs.kernel.org/leds/leds-class.html)[](https://docs.kernel.org/leds/leds-class.html)  with available colors and functions defined in  `include/dt-bindings/leds/common.h`.

```plain
iex(1)> ls "/sys/class/leds"
beaglebone:green:usr0      beaglebone:green:usr1      beaglebone:green:usr2      
beaglebone:green:usr3      mmc0::                     mmc1::                     
myfw:blue:indicator        myfw:green:indicator       myfw:red:indicator        
myfw:yellow:indicator  
```

The  `gpios`  key defines which GPIO chip and pin offset is used to drive the LED and if the line is on logical high or low when “active”. E.g.  `&gpio2 03`  is the fourth pin at the third GPIO chip — i.e.  `GPIO (2 * 32 + 3) = 67`. See  [LEDs connected to GPIO lines](https://www.kernel.org/doc/Documentation/devicetree/bindings/leds/leds-gpio.txt)  for details.

The  `default-state`  key defines the default behaviour of the LED — e.g. wether it’s on or off.

Please note that you can also define triggers in the device tree. E.g. the blue LED defines  `linux,default-trigger = "timer";`  and  `led-pattern = <125 125>;`  which leads to the blue LED flashing at 4Hz (125ms on, 125ms off) the moment the kernel starts. This can be a pretty good boot indicator. E.g. as long as the the firmare boots the LED is flashing. Once nerves code takes control of the LED (e.g. via delux) the LED is shut off. You can also configure more indication of e.g. disk and network activity (see  [Common leds properties](https://www.kernel.org/doc/Documentation/devicetree/bindings/leds/common.yaml)) or generic complex patterns by using the  `pattern`  trigger (see  [Pattern format for LED pattern trigger](https://www.kernel.org/doc/Documentation/devicetree/bindings/leds/leds-trigger-pattern.txt)).

## Compiling/Installing the Device Tree Overlay (manually)

You  _can_  compile and install the device tree overlay manually. Doing so has the small advantage that it doesn’t require a  [custom nerves system](https://hexdocs.pm/nerves/customizing-systems.html). The basic steps are

-   Run your  `*.dts`  file through the C preprocessor including the  **matching!**  kernel sources to resolve includes and macros.
    
-   Run the result through the device tree compiler  `dtc`.
    
-   Copy the resulting  `*.dtbo`  file to the  `rootfsoverlay/lib/firmware/ directory`  in your nerves project.
    

The following script might be a starting point (untested!):

```plain
#!/bin/sh

LINUX_SRCDIR="UNDEFINED" # Please point to the /matching/ kernel sources here
DTS_FILE="myoverlay.dts"
DTBO_FILE="myoverlay.dtbo"
FIRMWARE_DIR="your_nerves_project/rootfs_overlay/lib/firmware"

cpp -I ${LINUX_SRCDIR}/include -I ${LINUX_SRCDIR}/arch -nostdinc -undef -D__DTS__ -x assembler-with-cpp ${DTS_FILE} | \
dtc -Wno-unit_address_vs_reg -I dts -O dtb -b 0 -o ${DTBO_FILE}
cp ${DTBO_FILE} ${FIRMWARE_DIR}
```

## Compiling/Installing the Device Tree Overlay (nerves)

A recent  `nerves_system_bbb`  [commit](https://github.com/nerves-project/nerves_system_bbb/commit/8923c3f4fb232d55c7790c77737ab8378cc1079f)  makes it super simple to compile/install custom device tree overlays including automatic inclusion of Linux includes and macros. As of today (Aug, 11th, 2022) this is not yet part of the official  `nerves_system_bbb`  release. So you either need to follow  `main`  or update your  `package/extra-dts/extra-dts.mk`  manually.

```plain
#############################################################
#
# extra-dts
#
#############################################################

# Remember to bump the version when anything changes in this
# directory.
EXTRA_DTS_SOURCE =
EXTRA_DTS_VERSION = 0.0.2
EXTRA_DTS_DEPENDENCIES = host-dtc

define EXTRA_DTS_BUILD_CMDS
	cp $(NERVES_DEFCONFIG_DIR)/package/extra-dts/*.dts* $(@D)
        for filename in $(@D)/*.dts; do \
            $(CPP) -I$(@D) -I $(LINUX_SRCDIR)include -I $(LINUX_SRCDIR)arch -nostdinc -undef -D__DTS__ -x assembler-with-cpp $$filename | \
              $(HOST_DIR)/usr/bin/dtc -Wno-unit_address_vs_reg -@ -I dts -O dtb -b 0 -o $${filename%.dts}.dtbo || exit 1; \
        done
endef

define EXTRA_DTS_INSTALL_TARGET_CMDS
        cp $(@D)/*.dtbo $(TARGET_DIR)/lib/firmware
endef


$(eval $(generic-package))

```

One the updated file is in place (or the next version of  `nerves_system_bbb`  is released) you can simply drop your custom  `*.dts`  file into the  `package/extra-dts/`  directory of your custom system. Nerves will take care of the rest (i.e. compiling the file and putting into  `/lib/firmware/`  in the target filesystem).

## Using the Device Tree Overlay

As documented in  [nerves_system_bbb/Device tree overlays](https://github.com/nerves-project/nerves_system_bbb#device-tree-overlays)  there are two ways to load your custom overlay. You can either load the overlay by conconfiguring U-Boot:

```plain
cmd("fw_setenv uboot_overlay_addr7 /lib/firmware/myfw-gpio-leds.dtbo")
```

Or you can add those settings to  `fwup_include/provisioning.conf`:

```plain
[...]
uboot_setenv(uboot-env, "uboot_overlay_addr7", "/lib/firmware/myfw-gpio-leds.dtbo")
[...]
```

My take on these options is that I prefer the second one for overlays which should be enabled by default for all targets. The first approach is something where my use-case would be enabling overlays “dynamically” on a per-device basis.

## DONE!

That’s it! After building/installing the firmware (and reboot if you used the first/dynamic approach above) your custom device tree overlay is ready/loaded and you can control LEDs by simply writing to  `sysfs`  files:

```plain
iex(1)> cd "/sys/class/leds/myfw:red:indicator"
iex(3)> File.write("brightness", "255") # Enable LED
:ok
iex(8)> File.write("brightness", "0") # Disable LED
:ok
```

It’s of course much more convinient to use the delux library:

```plain
iex(1)> Delux.start_link(indicators: %{  
...(1)> port: %{green: "myfw:green:indicator"},
...(1)> read: %{blue: "myfw:blue:indicator"},  
...(1)> write: %{green: "myfw:yellow:indicator"},
...(1)> error: %{red: "myfw:red:indicator"}})
{:ok, #PID<0.1132.0>}
iex(2)> Delux.render(pid, %{error: Delux.Effects.blink(:red,2)})

```

And voilà … the red/error LED is blinking with 2Hz!

## Tips & Tricks

### Which overlay (version) is active?

If you are actively developing your overlay it’s sometimes hard to be sure which version of the overlay is actually active on the device. This is were the string given to chosen section comes into play because you can basically provide any string here. E.g. a version number:

```plain
&{/chosen} {
	overlays {
		MYFW_GPIO_LEDs.mycompany.com-overlays = "Version 0.0.1";
	};
};
```

This can then be easily checked/verified on the device:

```plain
iex(1)> cat "/sys/firmware/devicetree/base/chosen/overlays/MYFW_GPIO_LEDs.mycompany.com-overlays"
Version 0.0.1
```

### Rebuilding the firmware after device tree overlay changes

If you are actively developing your overlay always rebuilding the complete system/firmware takes too long. To force  `buildroot`  to pickup changes to your  `*.dts`  files in  `package/extra-dts`  the execute the following in your  `mix nerves.system.shell`:

```plain
$ make extra-dts-dirclean && make
```

This will force buildroot to clean the  `extra-dts`  build directory and rebuild the system. Based on your setup you might also need to run  `mix compile`  or even  `mix nerves.artifact`  and upload the artifact. Still much faster than rebuilding the whole system.

> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbMzQ1ODU1MzIyXX0=
-->