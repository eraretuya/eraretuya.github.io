---
layout:     post
title:      Outreachy Wrap-Up
date:       2017-03-17
summary:    Thoughts about the Outreachy experience.
tags:
  - outreachy
  - linux_kernel
  - iio
---

My [Outreachy](https://wiki.gnome.org/Outreachy) internship ended last week,
March 7 (6 in the US). Three months have gone by fast, part of me still can't believe I finally
[upstreamed](https://marc.info/?l=linux-iio&m=148872784529828&w=2) my first driver.

Since last update, I worked on supporting both *I2C* and *SPI* by factoring out common
code and using *regmap* API to handle relevant calls of the I2C/SPI subsystem. I also
got the opportunity to learn about *ACPI* and the *Device Tree* which are necessary in
enabling enumeration of the sensor especially when using SPI protocol[^1]. The last few
weeks I worked on writing support for *triggered buffer* where I get to explore
*interrupt* handling, *IIO trigger* and *buffer*. I submitted the patchset this week
and will be working on revisions from here on.

I'm happy and content of the work I've done. I learned a lot out of this experience
and I'm very grateful of this opportunity to contribute to the Linux Kernel. I can't
imagine what it's like if I haven't taken ["20 seconds of courage"](https://www.nerdfitness.com/blog/the-20-second-challenge/) to join the [outreachy-kernel mailing list](https://groups.google.com/forum/#!forum/outreachy-kernel).

The internship journey was fun and nerve-wracking at the same time. Things were
not always smooth sailing:

  * Wiring mishap leading to -EPROTO error.
  * Internet connectivity issues (has always been stable, why does it have to occur
    during the internship???) -- this lead to moving on into another ISP provider.
  * Got sick on one of the weekends. It was stomach pain that made me uncomfortable for
    a few days. Doctor's hunch it could be *Hyperacidity*. Took the prescription and got
    better. Have to avoid coffee and spicy foods during this time which is sad.
  * I cannot get some stuff to work. These are instances where Daniel (one of my mentors)
    would connect to my computer to help troubleshoot things.

These incidents diversified the experience. On the upside, working on the patches led
to exploring other parts of the kernel. I didn't expect I'd get a chance to submit
patches about device tree or regmap. I don't think it won't be the last though
since I plan on continuing the project on my spare time. So yes, there will be more
*adxl345* related-posts in the future.

I would like to thank the organizations that made this program possible, the Linux Kernel
for accepting me as an intern, my mentors Daniel and Alison for the guidance, and lastly
Jonathan (IIO maintainer) and other people who comment on my patches for their insightful
code reviews.

---

[^1]: I2C can be [instantiated from user-space](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/Documentation/i2c/instantiating-devices?id=HEAD) without having to rely on ACPI or Device Tree.
