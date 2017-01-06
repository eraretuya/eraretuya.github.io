---
layout:     post
title:      I2C and Device Probing
date:       2017-01-06
summary:    Writing the initial code of the ADXL345 accelerometer.
tags:
  - outreachy
  - linux_kernel
  - iio
---
My approach in writing the driver is to pull small bits of change one at a time
so that I don't get overwhelmed and to make sure I understand every bit of code
I put in.

Where do we start?

## A Bit of Documentation

The first thing you'll notice when you read any driver source code is the
multi-line comment describing what the driver this source code is written for
including *author* information, *license* and any other information relevant to the
driver. Furthermore, at the end of the source file, the same information is
documented[^1] by the use of **MODULE_\*** macros found in the header [linux/module.h](http://lxr.free-electrons.com/source/include/linux/module.h).
With this header included, we also add the headers for [i2c](http://lxr.free-electrons.com/source/include/linux/i2c.h) and [iio](http://lxr.free-electrons.com/source/include/linux/iio/iio.h) since this
is an IIO driver with support for the I2C interface. The commit about this
change looks like this:

{% highlight c %}
commit ae64be9abdb311ed4f1571f24be7d2a9192589b1
Author: Eva Rachel Retuya <eraretuya@gmail.com>
Date:   Wed Dec 14 17:17:04 2016 +0800

    iio: accel: adxl345: Write initial source
    
    Write initial source for adxl345: header, includes and module
    information.
    
    Signed-off-by: Eva Rachel Retuya <eraretuya@gmail.com>

diff --git a/drivers/iio/accel/adxl345.c b/drivers/iio/accel/adxl345.c
new file mode 100644
index 0000000..cdbd530
--- /dev/null
+++ b/drivers/iio/accel/adxl345.c
@@ -0,0 +1,22 @@
+/*
+ * ADXL345 3-Axis Digital Accelerometer
+ *
+ * Copyright (c) 2017 Eva Rachel Retuya <eraretuya@gmail.com>
+ *
+ * This file is subject to the terms and conditions of version 2 of
+ * the GNU General Public License. See the file COPYING in the main
+ * directory of this archive for more details.
+ *
+ * IIO driver for ADXL345
+ * 7-bit I2C slave address: 0x1D (ALT ADDRESS pin tied to VDDIO) or
+ * 0x53 (ALT ADDRESS pin grounded)
+ */
+
+#include <linux/i2c.h>
+#include <linux/module.h>
+
+#include <linux/iio/iio.h>
+
+MODULE_AUTHOR("Eva Rachel Retuya <eraretuya@gmail.com>");
+MODULE_DESCRIPTION("ADXL345 3-Axis Digital Accelerometer driver");
+MODULE_LICENSE("GPL v2");
{% endhighlight %}

## I2C

ADXL345 supports two communication protocols, the [I2C](http://lxr.free-electrons.com/source/Documentation/i2c/summary) and [SPI](http://lxr.free-electrons.com/source/Documentation/spi/spi-summary). I'll deal with
I2C first and then provide support for SPI later on. Looking up the kernel
documentation found in [Documentation/i2c/writing-clients](http://lxr.free-electrons.com/source/Documentation/i2c/writing-clients) we arrive with this
commit:

{% highlight c %}
commit 638041f126f6d92efab792025eb8c78d0944f13f
Author: Eva Rachel Retuya <eraretuya@gmail.com>
Date:   Thu Dec 15 16:23:22 2016 +0800

    iio: accel: adxl345: Add i2c client structure and allow initialization
    
    Populate struct i2c_device_id and i2c_driver. Make use of the macro
    'module_i2c_driver' to initialize the driver.
    
    Signed-off-by: Eva Rachel Retuya <eraretuya@gmail.com>

diff --git a/drivers/iio/accel/adxl345.c b/drivers/iio/accel/adxl345.c
index cdbd530..ec721fb 100644
--- a/drivers/iio/accel/adxl345.c
+++ b/drivers/iio/accel/adxl345.c
@@ -17,6 +17,30 @@
 
 #include <linux/iio/iio.h>
 
+static int adxl345_probe(struct i2c_client *client,
+			 const struct i2c_device_id *id)
+{
+
+	return 0;
+}
+
+static const struct i2c_device_id adxl345_i2c_id[] = {
+	{ "adxl345", 0 },
+	{ }
+};
+
+MODULE_DEVICE_TABLE(i2c, adxl345_i2c_id);
+
+static struct i2c_driver adxl345_driver = {
+	.driver = {
+		.name	= "adxl345",
+	},
+	.probe		= adxl345_probe,
+	.id_table	= adxl345_i2c_id,
+};
+
+module_i2c_driver(adxl345_driver);
+
 MODULE_AUTHOR("Eva Rachel Retuya <eraretuya@gmail.com>");
 MODULE_DESCRIPTION("ADXL345 3-Axis Digital Accelerometer driver");
 MODULE_LICENSE("GPL v2");
{% endhighlight %}

## Device Probing

Now that we have the necessary structures written and initialized, let's write
that *probe* function which is currently missing. According to the IIO developer's
guide[^2], we need to allocate an **iio_dev** structure using **devm_iio_device_alloc**,
fill-in the **indio_dev** members[^3] and finally call the **devm_iio_device_register**.

Another thing that we need to do is to verify the device by reading through the
*DEVID* register to confirm the *device ID*. This is useful if you are to support
other devices on the same driver code. The device ID lets you differentiate
and identify them. Here's the commit with all those presented:

{% highlight c %}
commit 621c51516e496c62f4bb71788ce279024a751dd0
Author: Eva Rachel Retuya <eraretuya@gmail.com>
Date:   Thu Dec 15 19:51:15 2016 +0800

    iio: accel: adxl345: Add initial code for probe()
    
    Declare associated structures required for allocating iio_dev. In probe,
    verify the device ID and allocate the iio_dev and fill-in the members as
    well as allow for device registration.
    
    Signed-off-by: Eva Rachel Retuya <eraretuya@gmail.com>

diff --git a/drivers/iio/accel/adxl345.c b/drivers/iio/accel/adxl345.c
index ec721fb..3120290 100644
--- a/drivers/iio/accel/adxl345.c
+++ b/drivers/iio/accel/adxl345.c
@@ -17,11 +17,50 @@
 
 #include <linux/iio/iio.h>
 
+#define ADXL345_REG_DEVID		0x00
+
+#define ADXL345_DEVID			0xE5
+
+struct adxl345_data {
+	struct i2c_client *client;
+};
+
+static const struct iio_info adxl345_info = {
+	.driver_module	= THIS_MODULE,
+};
+
 static int adxl345_probe(struct i2c_client *client,
 			 const struct i2c_device_id *id)
 {
+	struct adxl345_data *data;
+	struct iio_dev *indio_dev;
+	int ret;
+
+	ret = i2c_smbus_read_byte_data(client, ADXL345_REG_DEVID);
+	if (ret < 0) {
+		dev_err(&client->dev, "Error reading device ID: %d\n", ret);
+		return ret;
+	}
+
+	if (ret != ADXL345_DEVID) {
+		dev_err(&client->dev, "Invalid device ID: %d\n", ret);
+		return -ENODEV;
+	}
+
+	indio_dev = devm_iio_device_alloc(&client->dev, sizeof(*data));
+	if (!indio_dev)
+		return -ENOMEM;
+
+	data = iio_priv(indio_dev);
+	i2c_set_clientdata(client, indio_dev);
+	data->client = client;
+
+	indio_dev->dev.parent = &client->dev;
+	indio_dev->name = id->name;
+	indio_dev->info = &adxl345_info;
+	indio_dev->modes = INDIO_DIRECT_MODE;
 
-	return 0;
+	return devm_iio_device_register(&client->dev, indio_dev);
 }
 
 static const struct i2c_device_id adxl345_i2c_id[] = {
{% endhighlight %}

## Hey, Where's the Remove Function?

The commit on the previous section initially includes a *remove* function. This 
led to my computer hanging when the module is unloaded. If you have been
checking the links to the documentation I included, you'll notice that I'm using
a different version of **iio_device_alloc** and **iio_device_register** where they are 
prefixed by **devm_\***. These are what we call the resource-managed[^4] functions
and it allows us to free the memory allocated automatically when the device is
detached from the system or when the driver for the device is unloaded. This
means that, in most cases you don't have to provide a remove function.

Now, the question is *when* to provide the remove function? This is a question I
actually asked my mentors. Thankfully, [Alison](https://outreachyiio.blogspot.com/) explained it to me in detail. If
there is a *special* shutdown or undoing that needs to be done then you have to
provide the remove function and make use of the regular **iio_device_register/
*_unregister** instead of the resource-managed one.

## What's Next?

I will talk about raw measurements and scale on the next post. Stay tuned and
thanks for reading!

---

[^1]: [Licensing and Module Documentation](http://tldp.org/LDP/lkmpg/2.6/html/x279.html)
[^2]: [IIO Driver Developer's Guide](https://dbaluta.github.io/iiosubsys.html#iiodevice), as hosted on [Daniel's](https://plus.google.com/+DanielBaluta) (my other mentor) github pages.
[^3]: [drivers/staging/iio/Documentation/device.txt](http://lxr.free-electrons.com/source/drivers/staging/iio/Documentation/device.txt)
[^4]: [Documentation/driver-model/devres.txt](http://lxr.free-electrons.com/source/Documentation/driver-model/devres.txt)
