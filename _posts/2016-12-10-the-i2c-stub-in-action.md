---
layout:     post
title:      Using the I2C-Stub to Emulate a Device
date:       2016-12-10
summary:    See how the i2c-stub module can be used in emulating a device without having the hardware.
tags:       [outreachy,linux_kernel,iio]
---

The <span class="bg-dark-gray white">i2c-stub</span> module is a fake I2C/SMBus driver. This post demonstrates how to use this module to emulate a device.

## Pre-requisites

  * a copy of the kernel source
  * have <span class="bg-dark-gray white">i2c-tools</span> installed[^1];
    we're going to utilize **i2cset** from there.
  * specify a *target* chip driver, in this demo we'll use <span class="bg-dark-gray white">
   drivers/iio/light/al3320a</span>
  * any documentation[^2] about the *target* chip driver, this could be the [source
    code](http://lxr.free-electrons.com/source/drivers/iio/light/al3320a.c), a datasheet
    or any other material where you can find the *addresses* needed by *i2c-stub* and
    *i2cset*.
  * knowledge about configuring and installing a kernel from source

## Setting it up

Compile and install your copy of the kernel source. Make sure to configure the
i2c-stub and the *target* chip driver as a *module*.

`make menuconfig`

`Device Drivers -> I2C support -> I2C/SMBus Test Stub`

Next, we need to find the addresses required by the i2c-stub module and i2cset.
The i2c-stub needs the *I2C slave address*. On our *target* driver, it can be
found on the header of the [source code](http://lxr.free-electrons.com/source/drivers/iio/light/al3320a.c)
which is **0x1C**. For i2cset, we need the addresses of registers supported by
the driver for read and write. We'll be using these addresses (also taken from
the source code):

`#define AL3320A_REG_CONFIG_RANGE	0x07`

This is where the *scale* can be read or written from. I'll explain the meaning
of *scale* in a future post.

`#define AL3320A_REG_DATA_LOW		0x22`

Data from the device can be read from here. This register in particular
corresponds to the *low byte* of the device's output.

## Usage

From the i2c-stub [documentation](https://github.com/torvalds/linux/blob/master/Documentation/i2c/i2c-stub), usage is as follows:

> load the i2c-stub module.

`$ modprobe i2c-stub chip_addr=0x1c`

Use the *slave address* noted in the previous section to specify the **chip_addr**
module parameter required in loading i2c-stub

> use i2cset to pre-load some data on the addresses found earlier.

`$ i2cset 4 0x1C 0x07 0x04 b`

`$ i2cset 4 0x1C 0x22 0x64 w`

i2cset is used to set the I2C registers. The first parameter passed refers to
the number of the I2C bus associated with i2c-stub. This number can be noted
by executing the **i2cdetect -l** command. The second parameter is the slave
address while the third parameter correspond to the register address noted
previously.

The last two parameters refer to the value to write and the write size. The [man page](https://linux.die.net/man/8/i2cset) explains all these in detail if you want to know more.


> load the target chip driver module.

This is accomplished by executing the following command:

`$ echo al3320a 0x1c > /sys/class/i2c-adapter/i2c-4/new_device`

The name of the *target* driver and its *slave address* were used to instantiate[^3]
the device.


> observe its behavior in the kernel log.

You may use **dmesg** for this.

## See it in Action

Here's a full terminal session for demonstration:

{% highlight shell %}
$ whoami
root
$ modprobe i2c-stub chip_addr=0x1c
$ dmesg | tail
[10055.535293] i2c-stub: Virtual chip at 0x1c
$ i2cdetect -l
i2c-3	i2c       	DPDDC-D                         	I2C adapter
i2c-1	i2c       	i915 gmbus dpb                  	I2C adapter
i2c-4	smbus     	SMBus stub driver               	SMBus adapter
i2c-2	i2c       	i915 gmbus dpd                  	I2C adapter
i2c-0	i2c       	i915 gmbus dpc                  	I2C adapter
$ i2cset 4 0x1C 0x07 0x04 b
WARNING! This program can confuse your I2C bus, cause data loss and worse!
I will write to device file /dev/i2c-4, chip address 0x1c, data address
0x07, data 0x04, mode byte.
Continue? [Y/n] Y
$ i2cset 4 0x1C 0x22 0x64 w
WARNING! This program can confuse your I2C bus, cause data loss and worse!
I will write to device file /dev/i2c-4, chip address 0x1c, data address
0x22, data 0x64, mode word.
Continue? [Y/n] Y
$ dmesg | tail
[10055.535293] i2c-stub: Virtual chip at 0x1c
[10269.063197] i2c i2c-4: smbus byte data - addr 0x1c, wrote 0x04 at 0x07.
[10291.972838] i2c i2c-4: smbus word data - addr 0x1c, wrote 0x0064 at 0x22.
$ echo al3320a 0x1c > /sys/class/i2c-adapter/i2c-4/new_device
$ dmesg | tail
[10055.535293] i2c-stub: Virtual chip at 0x1c
[10269.063197] i2c i2c-4: smbus byte data - addr 0x1c, wrote 0x04 at 0x07.
[10291.972838] i2c i2c-4: smbus word data - addr 0x1c, wrote 0x0064 at 0x22.
[10383.427685] i2c i2c-4: new_device: Instantiated device al3320a at 0x1c
[10383.437720] i2c i2c-4: smbus byte data - addr 0x1c, wrote 0x01 at 0x00.
[10383.437720] i2c i2c-4: smbus byte data - addr 0x1c, wrote 0x04 at 0x07.
[10383.437721] i2c i2c-4: smbus byte data - addr 0x1c, wrote 0x04 at 0x09.
[10383.437722] i2c i2c-4: smbus byte data - addr 0x1c, wrote 0x00 at 0x06.
$ ls /sys/bus/iio/devices/iio:device0/
dev		    in_illuminance_scale	    name   subsystem
in_illuminance_raw  in_illuminance_scale_available  power  uevent
$ cat /sys/bus/iio/devices/iio:device0/name
al3320a
$ cat /sys/bus/iio/devices/iio:device0/in_illuminance_scale_available
0.512 0.128 0.032 0.01
$ cat /sys/bus/iio/devices/iio:device0/in_illuminance_scale
0.032000
$ echo 0.512 > /sys/bus/iio/devices/iio:device0/in_illuminance_scale
$ cat /sys/bus/iio/devices/iio:device0/in_illuminance_scale
0.512000
$ cat /sys/bus/iio/devices/iio:device0/in_illuminance_raw
100
$ modprobe -r i2c-stub
{% endhighlight %}

## Wrapping Up

The *sysfs attributes* associated with *al3320a* can now be seen after
instantiating the device. It's as if you had plugged the *actual* hardware in
there! Amazing, isn't it? This leads to an interesting use-case for i2c-stub
which is for *testing* the code without the actual device.

In the previous section, I tried reading and writing on the exposed attributes
and they work as expected. The *value* I specified for the data[^4] register reads
correctly and the scale can be written and read without issue.

That's it! Thanks for reading!

---

[^1]: Can be obtained via the [lm-sensors site](http://www.lm-sensors.org/wiki/I2CTools) or through this [cloned repository](https://github.com/groeck/i2c-tools).
[^2]: We need to know the *chip address* of the *target* as well as *register* addresses you need to read/write from.
[^3]: [Instantiating from user-space](https://www.kernel.org/doc/Documentation/i2c/instantiating-devices)
[^4]: 0x64 - a hex value is 100 in decimal notation.
