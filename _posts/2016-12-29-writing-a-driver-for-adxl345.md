---
layout:     post
title:      Writing a Driver for the ADXL345 Accelerometer
date:       2016-12-29
summary:    An introduction to the sensor and driver writing process.
tags:
  - outreachy
  - linux_kernel
  - iio
---

The goal of my outreachy project is to write a driver for a sensor using the
Industrial I/O interface. The sensor chosen is [ADXL345](http://www.analog.com/media/en/technical-documentation/data-sheets/ADXL345.pdf) from Analog Devices.
This sensor is a 3-axis digital accelerometer with high resolution measurement
of +/-(2g/4g/8g and 16g).

This sensor is no stranger to me. I used this before on a school project. It's
amusing that I'm writing a driver for it now. Come to think of it, this sensor
choice from that school project is overkill where we could've gone with a much
simpler sensor for tilt determination.

## Overview of features

The adxl345 is feature-filled:

  * User-selectable
    * Measurement ranges of 2g, 4g, 8g and 16g as mentioned before
    * Bandwidth
    * Resolution: fixed 10-bit resolution or full resolution where resolution
      increases with range, up to 13-bit resolution at +/- 16g
  * Sensing functions: activity/inactivity, single/double taps in any direction,
    free-fall detection
  * Power modes: low power, auto sleep and standby mode
  * 32-level FIFO buffer
  * Self-Test
  * I2C and SPI (3-and 4-wire) digital interfaces
  * Interrupts

## Planned Features for the Minimal Driver

As of this writing, I'm still waiting for the sensor to arrive. I have written
*device probing* and support for *read raw* which I will write about on a separate
post. I am told that these are enough for the initial support submission. Let's
see if I could include the *scale* as well.

## Writing Process

I set up a development branch under my clone of the [IIO tree](https://git.kernel.org/cgit/linux/kernel/git/jic23/iio.git/), created the source
file under <span class="bg-dark-gray white">accel/</span> and modified the *Kconfig* and *Makefile* so that I could build
the driver in-tree without hiccups. Here's what it looks like:

{% highlight shell %}
$ git clone git://git.kernel.org/pub/scm/linux/kernel/git/jic23/iio.git
$ git checkout -b adxl345_development
$ touch drivers/iio/accel/adxl345.c
$ vim drivers/iio/accel/Makefile
$ vim drivers/iio/accel/Kconfig
{% endhighlight %}

I just follow the kernel [patch philosophy](https://kernelnewbies.org/PatchPhilosophy) where you commit only one specific
change at a time so that it's easier to review. Here's what my git log looks
like so far with the first pre-patch being the Kconfig+Makefile modification.

{% highlight shell %}
46ba4d5 iio: accel: adxl345: Expose available scales to sysfs
e38d33b iio: accel: adxl345: implement read_raw
35a3a17 iio: accel: adxl345: Declare acceleration channels
eecb517 iio: accel: adxl345: Add initial code for probe()
ce994c5 iio: accel: adxl345: Add i2c client structure and allow initialization
9a1aa84 iio: accel: adxl345: Write initial source
fb83aae iio: accel: adxl345: Add support for building adxl345
f8611c0 Staging: iio: impedance-analyzer: ad5933: fix wrong comments
7c271ee iio: adc: spmi-vadc: Changes to support different scaling
ba71704 iio: adc: spmi-vadc: Update function for generic voltage conversion
{% endhighlight %}

Here are some sources I'll be referring to during the writing process:

  * [drivers/iio/dummy/iio_simple_dummy.c](https://git.kernel.org/cgit/linux/kernel/git/jic23/iio.git/tree/drivers/iio/dummy/iio_simple_dummy.c?h=testing)
  * [Existing drivers in dirvers/iio/accel/](https://git.kernel.org/cgit/linux/kernel/git/jic23/iio.git/tree/drivers/iio/accel?h=testing)
  * [Industrial I/O driver developer's guide](https://dbaluta.github.io/)

Thanks for reading!
