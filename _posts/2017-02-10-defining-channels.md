---
layout:     post
title:      Defining IIO Channels
date:       2017-02-10
summary:    Introducing the IIO device channels and the sysfs interface.
tags:
  - outreachy
  - linux_kernel
  - iio
---

## IIO Channels

Data channels for each sensor may vary depending on the sensor's functionality
and point of measurement. In the case of **ADXL345**, an accelerometer, 3 channels
represent *acceleration* on the X, Y and Z axes. A light
sensor on the other hand, can have a channel that represents measurement in the
*visible* spectrum and another for the *infrared* spectrum.

The struct **iio_chan_spec** is used to define channels. In ADXL345, its channels are
defined like this:
{% highlight c %}
From 3def3a4f5ab058b6db77d3e80160bae798469017 Mon Sep 17 00:00:00 2001
From: Eva Rachel Retuya <eraretuya@gmail.com>
Date: Tue, 20 Dec 2016 18:35:18 +0800
Subject: [PATCH 1/3] iio: accel: adxl345: Declare acceleration channels

Declare channels of type IIO_ACCEL and fill-in members in probe
corresponding to this declaration.

Signed-off-by: Eva Rachel Retuya <eraretuya@gmail.com>
---
 drivers/iio/accel/adxl345.c | 18 ++++++++++++++++++
 1 file changed, 18 insertions(+)

diff --git a/drivers/iio/accel/adxl345.c b/drivers/iio/accel/adxl345.c
index e22904a..6999372 100644
--- a/drivers/iio/accel/adxl345.c
+++ b/drivers/iio/accel/adxl345.c
@@ -18,6 +18,9 @@
 #include <linux/iio/iio.h>
 
 #define ADXL345_REG_DEVID		0x00
+#define ADXL345_REG_DATAX0		0x32
+#define ADXL345_REG_DATAY0		0x34
+#define ADXL345_REG_DATAZ0		0x36
 
 #define ADXL345_DEVID			0xE5
 
@@ -25,6 +28,19 @@ struct adxl345_data {
 	struct i2c_client *client;
 };
 
+#define ADXL345_CHANNEL(reg, axis) {					\
+	.type = IIO_ACCEL,						\
+	.modified = 1,							\
+	.channel2 = IIO_MOD_##axis,					\
+	.address = reg,							\
+}
+
+static const struct iio_chan_spec adxl345_channels[] = {
+	ADXL345_CHANNEL(ADXL345_REG_DATAX0, X),
+	ADXL345_CHANNEL(ADXL345_REG_DATAY0, Y),
+	ADXL345_CHANNEL(ADXL345_REG_DATAZ0, Z),
+};
+
 static const struct iio_info adxl345_info = {
 	.driver_module	= THIS_MODULE,
 };
@@ -59,6 +75,8 @@ static int adxl345_probe(struct i2c_client *client,
 	indio_dev->name = id->name;
 	indio_dev->info = &adxl345_info;
 	indio_dev->modes = INDIO_DIRECT_MODE;
+	indio_dev->channels = adxl345_channels;
+	indio_dev->num_channels = ARRAY_SIZE(adxl345_channels);
 
 	return devm_iio_device_register(&client->dev, indio_dev);
 }
-- 
2.7.4
{% endhighlight %}

## Channel sysfs Attributes

Exposition of chip information and configuration parameters are accessible via *sysfs*
files. These files are hereto referred as *attributes* and for the IIO subsystem
is found under **/sys/bus/iio/iio:deviceX/** directory.

For an example of what these attributes were, you can check out this file on the
kernel sources:
[Documentation/ABI/testing/syfs-bus-iio](http://lxr.free-electrons.com/source/Documentation/ABI/testing/sysfs-bus-iio)

For the ADXL345 IIO driver [basic support](https://marc.info/?l=linux-iio&m=148584711012686&w=2)
 submission, the following attributes are present:
{% highlight shell %}
~ # ls /sys/bus/iio/devices/iio\:device0/
deferred_probe  in_accel_scale  in_accel_y_raw  name            subsystem
dev             in_accel_x_raw  in_accel_z_raw  power           uevent
{% endhighlight %}

The attributes follow this generic form:

`{direction}_{type}_{index}_{modifier}_{info}`

The [industrialio-core.c](http://lxr.free-electrons.com/source/drivers/iio/industrialio-core.c#L49)
file contain further details for examination:

{% highlight c %}
static const char * const iio_direction[] = {
	[0] = "in",
	[1] = "out",
};

static const char * const iio_chan_type_name_spec[] = {
	[IIO_VOLTAGE] = "voltage",
	[IIO_CURRENT] = "current",
	[IIO_POWER] = "power",
	[...]
	[IIO_COUNT] = "count",
	[IIO_INDEX] = "index",
	[IIO_GRAVITY]  = "gravity",
};

static const char * const iio_modifier_names[] = {
	[IIO_MOD_X] = "x",
	[IIO_MOD_Y] = "y",
	[IIO_MOD_Z] = "z",
	[...]
	[IIO_MOD_Q] = "q",
	[IIO_MOD_CO2] = "co2",
	[IIO_MOD_VOC] = "voc",
};

/* relies on pairs of these shared then separate */
static const char * const iio_chan_info_postfix[] = {
	[IIO_CHAN_INFO_RAW] = "raw",
	[IIO_CHAN_INFO_PROCESSED] = "input",
	[IIO_CHAN_INFO_SCALE] = "scale",
	[...]
	[IIO_CHAN_INFO_DEBOUNCE_TIME] = "debounce_time",
	[IIO_CHAN_INFO_CALIBEMISSIVITY] = "calibemissivity",
	[IIO_CHAN_INFO_OVERSAMPLING_RATIO] = "oversampling_ratio",
};
{% endhighlight %}

These attributes are exposed to userspace in the form of *bitmasks*. The **info_mask_\***
members in struct **iio_chan_spec** determine sharing of data across different channels.
A detailed description of what each member is for is found in
the header file [iio.h](http://lxr.free-electrons.com/source/include/linux/iio/iio.h#L205):
{% highlight c %}
/**
 * struct iio_chan_spec - specification of a single channel
 * [...]
 * @info_mask_separate: What information is to be exported that is specific to
 *			this channel.
 * @info_mask_separate_available: What availability information is to be
 *			exported that is specific to this channel.
 * @info_mask_shared_by_type: What information is to be exported that is shared
 *			by all channels of the same type.
 * @info_mask_shared_by_type_available: What availability information is to be
 *			exported that is shared by all channels of the same
 *			type.
 * @info_mask_shared_by_dir: What information is to be exported that is shared
 *			by all channels of the same direction.
 * @info_mask_shared_by_dir_available: What availability information is to be
 *			exported that is shared by all channels of the same
 *			direction.
 * @info_mask_shared_by_all: What information is to be exported that is shared
 *			by all channels.
 * @info_mask_shared_by_all_available: What availability information is to be
 * [...]
 */
{% endhighlight %}

## Reading and Writing on sysfs files

The earlier sections detailed how to create these sysfs files. Now we're going
to add the code that will enable us to *read* and/or *write* from/to these files.

In struct **iio_info** we'll have to provide a **read_raw** function for reading as well
as a **write_raw** function for writing. In ADXL345, I only provided a read_raw
function since the configurable attributes are yet to be implemented.

For the **RAW** attribute:

{% highlight c %}
From 0590939f32cd73de23b2d19bb3fbf1e06ef8c616 Mon Sep 17 00:00:00 2001
From: Eva Rachel Retuya <eraretuya@gmail.com>
Date: Thu, 22 Dec 2016 21:56:24 +0800
Subject: [PATCH 1/2] iio: accel: adxl345: Implement read_raw and
 standby/measurement modes

Enable full-resolution mode as well as measurement mode on device probe.
Return the value read from data register. This value increases with g
range, up to 13-bit resolution at +/- 16 g. On device remove, make sure
to put the sensor in standby mode.

Signed-off-by: Eva Rachel Retuya <eraretuya@gmail.com>
---
 drivers/iio/accel/adxl345.c | 72 ++++++++++++++++++++++++++++++++++++++++++++-
 1 file changed, 71 insertions(+), 1 deletion(-)

diff --git a/drivers/iio/accel/adxl345.c b/drivers/iio/accel/adxl345.c
index 6999372..54ea601 100644
--- a/drivers/iio/accel/adxl345.c
+++ b/drivers/iio/accel/adxl345.c
@@ -18,14 +18,26 @@
 #include <linux/iio/iio.h>
 
 #define ADXL345_REG_DEVID		0x00
+#define ADXL345_REG_POWER_CTL		0x2D
+#define ADXL345_REG_DATA_FORMAT		0x31
 #define ADXL345_REG_DATAX0		0x32
 #define ADXL345_REG_DATAY0		0x34
 #define ADXL345_REG_DATAZ0		0x36
 
+#define ADXL345_POWER_CTL_MEASURE	BIT(3)
+#define ADXL345_POWER_CTL_STANDBY	0x00
+
+#define ADXL345_DATA_FORMAT_FULL_RES	BIT(3)
+#define ADXL345_DATA_FORMAT_2G		0
+#define ADXL345_DATA_FORMAT_4G		1
+#define ADXL345_DATA_FORMAT_8G		2
+#define ADXL345_DATA_FORMAT_16G		3
+
 #define ADXL345_DEVID			0xE5
 
 struct adxl345_data {
 	struct i2c_client *client;
+	u8 data_range;
 };
 
 #define ADXL345_CHANNEL(reg, axis) {					\
@@ -33,6 +45,7 @@ struct adxl345_data {
 	.modified = 1,							\
 	.channel2 = IIO_MOD_##axis,					\
 	.address = reg,							\
+	.info_mask_separate = BIT(IIO_CHAN_INFO_RAW),			\
 }
 
 static const struct iio_chan_spec adxl345_channels[] = {
@@ -41,8 +54,35 @@ static const struct iio_chan_spec adxl345_channels[] = {
 	ADXL345_CHANNEL(ADXL345_REG_DATAZ0, Z),
 };
 
+static int adxl345_read_raw(struct iio_dev *indio_dev,
+			    struct iio_chan_spec const *chan,
+			    int *val, int *val2, long mask)
+{
+	struct adxl345_data *data = iio_priv(indio_dev);
+	int ret;
+
+	switch (mask) {
+	case IIO_CHAN_INFO_RAW:
+		/*
+		 * Data is stored in adjacent registers:
+		 * ADXL345_REG_DATA(X0/Y0/Z0) contain the least significant byte
+		 * and ADXL345_REG_DATA(X0/Y0/Z0) + 1 the most significant byte
+		 */
+		ret = i2c_smbus_read_word_data(data->client, chan->address);
+
+		if (ret < 0)
+			return ret;
+
+		*val = sign_extend32(le16_to_cpu(ret), 12);
+		return IIO_VAL_INT;
+	}
+
+	return -EINVAL;
+}
+
 static const struct iio_info adxl345_info = {
 	.driver_module	= THIS_MODULE,
+	.read_raw	= adxl345_read_raw,
 };
 
 static int adxl345_probe(struct i2c_client *client,
@@ -70,6 +110,15 @@ static int adxl345_probe(struct i2c_client *client,
 	data = iio_priv(indio_dev);
 	i2c_set_clientdata(client, indio_dev);
 	data->client = client;
+	/* Enable full-resolution mode */
+	data->data_range = ADXL345_DATA_FORMAT_FULL_RES;
+
+	ret = i2c_smbus_write_byte_data(data->client, ADXL345_REG_DATA_FORMAT,
+					data->data_range);
+	if (ret < 0) {
+		dev_err(&client->dev, "Failed to set data range: %d\n", ret);
+		return ret;
+	}
 
 	indio_dev->dev.parent = &client->dev;
 	indio_dev->name = id->name;
@@ -78,7 +127,27 @@ static int adxl345_probe(struct i2c_client *client,
 	indio_dev->channels = adxl345_channels;
 	indio_dev->num_channels = ARRAY_SIZE(adxl345_channels);
 
-	return devm_iio_device_register(&client->dev, indio_dev);
+	/* Enable measurement mode */
+	ret = i2c_smbus_write_byte_data(data->client, ADXL345_REG_POWER_CTL,
+					ADXL345_POWER_CTL_MEASURE);
+	if (ret < 0) {
+		dev_err(&client->dev, "Failed to enable measurement mode: %d\n",
+			ret);
+		return ret;
+	}
+
+	return iio_device_register(indio_dev);
+}
+
+static int adxl345_remove(struct i2c_client *client)
+{
+	struct iio_dev *indio_dev = i2c_get_clientdata(client);
+	struct adxl345_data *data = iio_priv(indio_dev);
+
+	iio_device_unregister(indio_dev);
+
+	return i2c_smbus_write_byte_data(data->client, ADXL345_REG_POWER_CTL,
+					 ADXL345_POWER_CTL_STANDBY);
 }
 
 static const struct i2c_device_id adxl345_i2c_id[] = {
@@ -93,6 +162,7 @@ static struct i2c_driver adxl345_driver = {
 		.name	= "adxl345",
 	},
 	.probe		= adxl345_probe,
+	.remove		= adxl345_remove,
 	.id_table	= adxl345_i2c_id,
 };
 
-- 
2.7.4

{% endhighlight %}

For **SCALE**:
{% highlight c %}
From 7fb83f76a43b8ab4ffc917b58d43b13d0536cd55 Mon Sep 17 00:00:00 2001
From: Eva Rachel Retuya <eraretuya@gmail.com>
Date: Mon, 23 Jan 2017 19:01:34 +0800
Subject: [PATCH 2/2] iio: accel: adxl345: Implement scale

Implement scale under full-resolution mode.

Signed-off-by: Eva Rachel Retuya <eraretuya@gmail.com>
---
 drivers/iio/accel/adxl345.c | 15 +++++++++++++++
 1 file changed, 15 insertions(+)

diff --git a/drivers/iio/accel/adxl345.c b/drivers/iio/accel/adxl345.c
index 54ea601..bec7b0f 100644
--- a/drivers/iio/accel/adxl345.c
+++ b/drivers/iio/accel/adxl345.c
@@ -35,6 +35,15 @@
 
 #define ADXL345_DEVID			0xE5
 
+/*
+ * In full-resolution mode, scale factor is maintained at ~4 mg/LSB
+ * in all g ranges.
+ *
+ * At +/- 16g with 13-bit resolution, scale is computed as:
+ * (16 + 16) * 9.81 / (2^13 - 1) = 0.0383
+ */
+static const int adxl345_uscale = 38300;
+
 struct adxl345_data {
 	struct i2c_client *client;
 	u8 data_range;
@@ -46,6 +55,7 @@ struct adxl345_data {
 	.channel2 = IIO_MOD_##axis,					\
 	.address = reg,							\
 	.info_mask_separate = BIT(IIO_CHAN_INFO_RAW),			\
+	.info_mask_shared_by_type = BIT(IIO_CHAN_INFO_SCALE),		\
 }
 
 static const struct iio_chan_spec adxl345_channels[] = {
@@ -75,6 +85,11 @@ static int adxl345_read_raw(struct iio_dev *indio_dev,
 
 		*val = sign_extend32(le16_to_cpu(ret), 12);
 		return IIO_VAL_INT;
+	case IIO_CHAN_INFO_SCALE:
+		*val = 0;
+		*val2 = adxl345_uscale;
+
+		return IIO_VAL_INT_PLUS_MICRO;
 	}
 
 	return -EINVAL;
-- 
2.7.4
{% endhighlight %}

## Raw? Scale? What are these?

Looking back at the channel definition for the ADXL345, the bitmask
**IIO_CHAN_INFO_RAW** was placed under **info_mask_separate** which yielded 3
*separate* sysfs files in the form of **in_accel_[xyz]_raw** as seen above.

We refer back to the IIO sysfs [ABI](http://lxr.free-electrons.com/source/Documentation/ABI/testing/sysfs-bus-iio#L162) which describes this in detail:

{% highlight text %}
What:		/sys/bus/iio/devices/iio:deviceX/in_accel_x_raw
What:		/sys/bus/iio/devices/iio:deviceX/in_accel_y_raw
What:		/sys/bus/iio/devices/iio:deviceX/in_accel_z_raw
KernelVersion:	2.6.35
Contact:	linux-iio@vger.kernel.org
Description:
		Acceleration in direction x, y or z (may be arbitrarily assigned
		but should match other such assignments on device).
		Has all of the equivalent parameters as per voltageY. Units
		after application of scale and offset are m/s^2.
{% endhighlight %}

Whereas a *single* scale file shared across all channels (**IIO_CHAN_INFO_SCALE**)
of the *same type* meant:

{% highlight text %}
What:		/sys/bus/iio/devices/iio:deviceX/in_accel_scale
KernelVersion:	2.6.35
Contact:	linux-iio@vger.kernel.org
Description:
		If known for a device, scale to be applied to <type>Y[_name]_raw
		post addition of <type>[Y][_name]_offset in order to obtain the
		measured value in <type> units as specified in
		<type>[Y][_name]_raw documentation.  If shared across all in
		channels then Y and <x|y|z> are not present and the value is
		called <type>[Y][_name]_scale. The peak modifier means this
		value is applied to <type>Y[_name]_peak_raw values.
{% endhighlight %}

Therefore, the raw attributes represent raw readings from the device. Scale on the
other hand is what you will apply to the raw data resulting to the acceleration
values measured in terms of $$ m/s^2 $$.

The following x, y and z readings:
{% highlight shell %}
~ # cat /sys/bus/iio/devices/iio\:device0/in_accel_x_raw 
8
~ # cat /sys/bus/iio/devices/iio\:device0/in_accel_y_raw 
35
~ # cat /sys/bus/iio/devices/iio\:device0/in_accel_z_raw 
248
~ # cat /sys/bus/iio/devices/iio\:device0/in_accel_scale
0.038300
{% endhighlight %}

Translates to:

$$ X:~8\times0.038300 = 0.3064~m/s^2 $$

$$ Y:~35\times0.038300 = 1.3405~m/s^2 $$

$$ Z:~248\times0.038300 = 9.4984~m/s^2 $$
