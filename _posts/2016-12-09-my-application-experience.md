---
layout:     post
title:      Applying to the FOSS Outreachy Internship Program
date:       2016-12-09
summary:    Read on to find out about my application experience as well as thoughts and reflections on that matter.
tags:
  - outreachy
  - linux_kernel
  - iio
---

Hello there!

I just started an [internship](https://wiki.gnome.org/Outreachy/2016/DecemberMarch#Linux_kernel) under the Linux Kernel organization as part of the [Outreachy](https://wiki.gnome.org/Outreachy/) program.

## Beginning My FOSS Journey

I completed my Bachelor's degree in Computer Engineering last July of 2014. Unlike
my peers who started their career right away after graduation, I took a
*sabbatical* to think over things I wanted to pursue as well as better manage
my health.

During that span of time, I learned new skills on my own and also became aware about
the [FOSS](https://en.wikibooks.org/wiki/FOSS_A_General_Introduction/Introduction)
movement. I got excited of the fact that there is a lot to learn (not
limited to technical) with examining projects open to the public and interacting
with the community. I also discovered the Outreachy program which helped me got
started contributing to the organization of my choice.

## My Application Experience

I made a shortlist of organizations where my current skills are applicable and
then checked out the available projects. I started contributing patches to the
*Linux Kernel* during the Outreachy [round 11](https://kernelnewbies.org/OutreachyRound11)
and finally succeeded on round 13. Yes, I worked through *3* applications. I did
not let failure deter me and used my previous observations and learnings (from
mistakes) to make it through.

### Round 11

The project that caught my attention was all about writing a driver for a chosen
sensor using the [Industrial I/O interface](https://www.kernel.org/doc/htmldocs/iio/index.html).

The application process was highly competitive. I started a bit late, went through
the [first patch tutorial](https://kernelnewbies.org/Outreachyfirstpatch) as well
as overcame initial *barriers* which consist of reaching out to mentors and
submitting my first patch to a [mailing list](https://groups.google.com/forum/#!forum/outreachy-kernel)
. The experience on the former is akin to introducing yourself to a stranger you
just met -- I was *scared* and nervous considering the fact that these people are
veterans in the industry. Posting to the mailing list on the other hand, was
also scary and *intimidating* at first since I'm getting my patch seen by many
eyes. It was all good after that. The mentors were nice and approachable.

At this stage, I know *nothing* about the kernel. The codebase is *huge* and it's
easy to get lost. I can read the syntax, have a few assumptions of *what* it does
but I have little to no idea *why* it's doing that. You have to start somewhere
and thankfully new contributors can settle first under the Staging
<span class="bg-dark-gray white">drivers/staging/</span> subsystem.

I acquainted myself with the [Kernel workflow](https://kernelnewbies.org/PatchPhilosophy#head-32f427292f3e08feae8bbee205b9c0f0cf2c51ec). Did the pre-requisite staging cleanups which are
*checkpatch.pl-identified* issues (Hint: [coding-style fixes](https://github.com/torvalds/linux/blob/master/Documentation/CodingStyle)) and familiarizing with
the available projects. You may be wondering why waste time fixing coding-style
issues when you can go ahead and move on and do a more meaningful change. Well,
you can do whatever you want but the main takeaway on this is you get to read and
recognize code while doing the intended change/s.

### Round 12

Before the start of this round, I read through the [Linux Device Drivers 3rd Ed.](https://lwn.net/Kernel/LDD3/) book, *ooh-ing* and *ahh-ing* at the recognition of some code snippets I
encountered the previous round. On this [round](https://kernelnewbies.org/OutreachyRound12),
I was more informed on whatever change/s I'm trying to make. It's also the round
where I branched-off to more complex changes than the previous coding-style issues.

My strategy for this round is to get myself familiar on as many projects as I can.
I managed to solve the entirety of [IIO tasks](https://kernelnewbies.org/IIO_tasks),
I learned how to use [coccinelle](http://coccinelle.lip6.fr/) to match code
patterns and perform transformations as well as submit a [patch](https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/?id=b429a773c193ee7cb752144e590181a1b8cc8fb5)
[^1] about [workqueues](https://www.kernel.org/doc/Documentation/workqueue.txt).

Obviously the strategy didn't work, diving into different projects is time-consuming
in retrospect and in the process majority of the
[small-tasks](https://wiki.gnome.org/Outreachy#Make_a_Small_Contribution)
got claimed before I got the chance to tackle them. Despite the outcome, I'd
argue that what I did is time well-spent because learning something new is *thrilling*.

### Round 13

What was different on this round is that, I focused on a *single* project -- my
first choice which is still the IIO project. I tackled the small-tasks and
communicated more with the mentors. Later on, I got myself more involved[^2]
by working on patches suggested[^3] to me via the code-reviews. This is a huge
payoff, I learned more about the IIO in the process and it resulted to me
getting picked to become the intern for this round of internships.

## Summary

I think what has worked for me in succeeding are the following:

  * Start early.
  * Learn as much as you can about the project.
  * Exercise patience and self-reliance. Some things will not go your way, try
    your best to overcome it on your own and if still stuck, do not hesitate to
    ask for help.
  * Claim small-tasks ASAP.
  * Help or provide assistance for others.
  * Do not be afraid to go the extra mile when it comes to contributions.
  * Communicate, communicate and communicate. Have to reiterate that.
  * Document what you do as you do it. This is really helpful when you need to
    look back for reference.
  * Have fun.

Thanks for reading and take care!

---

[^1]: This is my first patch outside <span class="bg-dark-gray white">drivers/staging/</span> and posting to [LKML](https://en.wikipedia.org/wiki/Linux_kernel_mailing_list)! The change looks simple, but the behind-the-scenes analysis and decision-making took a bit of effort and is complicated than it looks.
[^2]: I learned about regulators by working through this [issue](https://marc.info/?l=linux-iio&m=147583702532337&w=2)
[^3]: I understood how to properly [declare](https://marc.info/?l=linux-iio&m=147594643726702&w=2) these [defines](https://git.kernel.org/cgit/linux/kernel/git/jic23/iio.git/commit/?h=testing&id=579d542534c96c9109b63e79a2c6d22e2bca21cc) that will aid in readability instead of wondering, *what is that value for? Where did that come from?*
