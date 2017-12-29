---
layout:     post
title:      My First Kernel Conference
date:       2017-12-29
summary:    Met a lot of great people and had my first (lightning) talk at the Open Source Summit Europe 2017.
tags:
  - outreachy
  - linux_kernel
  - iio
---

As a former [Outreachy](https://www.outreachy.org/) intern, I was blessed with the opportunity to present my work at the [Open Source Summit Europe 2017](https://events17.linuxfoundation.org/events/open-source-summit-europe) held last _October 23 to 26_. At the conference I was able to meet my mentors, other Outreachy interns and other people passionate about the _Linux Kernel_ and _Open-source_.

## First Time Speaker

This is my _first_ technical conference and I attended as a speaker. I'm a newbie at _public speaking_ and so I had to make preparations for it months before the conference. I participated in a panel discussion entitled __"Outreachy Kernel Internship Report"__ [(slides here)](https://osseu17.sched.com/event/BxJM/panel-discussion-outreachy-kernel-internship-report-moderated-by-julia-lawall-inria), with the entire session lasting about _40 minutes_. Since it's a panel discussion, each participant is allotted _5 minutes_ to present.

This 5-minute talk can be classified as a lightning talk[^1] and it's a good opportunity for first-time speakers like me to hone public speaking skills. The short duration allows you to practice many times!

## Talk Preparation

I have read a lot of articles on the Internet about giving lightning talks. One of the resources that stand out was this: _How To Give a Great Tech Talk_[^2] which covers all formats not only lightning talks.

I also need to consider speech that promotes inclusiveness and so I took advantage of the free offering that is ["Inclusive Speaker Orientation"](https://training.linuxfoundation.org/content/inclusive-speaker-orientation).

If you are curious about the articles, here are some:
* [Giving Lightning Talks](https://www.perl.com/pub/2004/07/30/lightningtalk.html)
* [Giving a good lightning talk](https://software.ac.uk/home/cw11/giving-good-lightning-talk)
* [16 Ways to Prepare for a Lightning Talk](https://www.semrush.com/blog/16-ways-to-prepare-for-a-lightning-talk/)

## Networking Opportunities

The conference granted the opportunity to meet some of the mentors who were instrumental in my involvement in the Kernel community: _Julia Lawall, Daniel Baluta, Rik van Riel and Greg Kroah-Hartman_. Apart from meeting mentors in person, I had the opportunity to be introduced to other people in the conference as well as see in person key people in the community like _Linus Torvalds_.

In as much as I'm _grateful_ of the opportunity, I was not able to make the most out of it. Being a first-time attendee meant that I knew only a handful of people and if it happened I attended a different conference then I absolutely don't know anyone from the crowd! I'm not particularly _shy_ but it got a hold of me at times especially during the __Women in Open Source Lunch sponsored by Adobe__.

![Women in Open Source Lunch](https://farm5.staticflickr.com/4484/37879135031_edea3e976c_b.jpg)
_Women in Open Source Lunch Group Photo_[^3]

There, I was able to mingle with other women attendees. It's a regret I didn't get to introduce myself more to people there, I got tongue-tied and didn't take much initiative to start even a small-talk! With this experience, now I understand why people dread networking gatherings.

I vow to do better on next opportunity.

## Favorite Sessions

Here's the chronological list of my favorite sessions.

### Free and Open Source Software Tools for Making Open Source Hardware - Leon Anavi, Konsulko Group

> Consider the design of the case simultaneously with the design of the PCB

According to _Leon_, open source hardware is a _viable_ business model and there are free and open
source tools available for designing such hardware.

A fully open source _physical_ product consist of an _open source PCB, open source software and open
source case_. The session gave awareness on how open hardware is licensed as well as available FOSS
tools that can be used to design the PCB and the case.

### Fast and Precise Retrieval of Forward and Back Porting Information for Linux Device Drivers - Julia Lawall, Inria 

_Julia_, one of my mentors, presented how [Prequel](http://prequel-pql.gforge.inria.fr/) can be used to
aid in porting of drivers. If you are familiar with _Coccinelle_, you'll be at home with this tool.

__Prequel__ is a patch-like query language for commit history search. To ease driver porting,
_gcc-reduce_ was used to reduce error messages and _prequel_ was used to search commit histories.

> For 86% of these 80 issues, the first commit returned by Prequel gave the needed information

For the full-picture of the process, you may check out the slides[^4].

I'm glad this tool exist. I would normally use any of these commands to do something similar:
{% highlight git %}
git log --grep='search string'
git log --pretty=oneline --abbrev-commit -S "search_query"
{% endhighlight %}

Search the code based on a _keyword_:
{% highlight git %}
git grep 'keyword' /path/to/be/searched/
{% endhighlight %}
See _who_ did the change and _why_:
{% highlight git %}
git blame -L line1,linex file_path.c
{% endhighlight %}

### How Not to be a Good Linux Kernel Maintainer - Bartlomiej Zolnierkiewicz, Samsung Electronics Polska Sp. Z o.o. 

> No starting guide for new maintainers

For budding new maintainers out there, _Bartlomiej_ pointed out several _technical_ and _social_ mistakes
that may happen and provided insight on do's and what NOT to do in those situations.

Also, _Bartlomiej_ notes that the [Documentation/process/](http://elixir.free-electrons.com/linux/latest/source/Documentation/process) is developer-oriented but except for <span class="bg-dark-gray white">Documentation/process/management-style.rst</span>[^5]

### Dirty Clouds Done Dirt Cheap - Matthew Treinish, IBM

> Any guesses what this means

A fun session, _Matthew_ detailed his ~~journey~~ project on building an _OpenStack_ cloud on a _$1500_
budget with the outlook of someone who has <span class="bg-dark-gray white">no prior knowledge</span> of OpenStack. What made this more
interesting is on the reliance of using _release tarballs_ than using packages which resulted to the
the majority of issues encountered.

The project enabled enumeration of areas that need to be improved upon such as that of _logging_ and
_error reporting_.

If you want to learn more about this subject, you may check out this [video](https://www.openstack.org/videos/boston-2017/dirty-clouds-done-dirt-cheap) from another conference and for further details, this [blog post](http://blog.kortar.org/?p=380).

### Why Should We Care About Kernelnewbies! - Vaishali Thakkar, Oracle

> Newbies often looks for more ideas but don't know where to find them

Being a _Kernelnewbie_ myself, I can relate with the difficulties discussed on this session.
_Vaishali_ listed the following obstacles in a Kernelnewbies journey:

#### Setting up an environment

In Outreachy, we have this [First Patch](https://kernelnewbies.org/Outreachyfirstpatch/) tutorial.
Having gone through that, I agree with Vaishali that although it helps in setting up an environment for
Kernel development, it doesn't explain how _kernel development process_[^6] works.

#### First successful contribution

Maintainers pick up the _patches_ based on their _work flow_ and these patches get sent through a _mailing
list_. Due to the <span class="bg-dark-gray white">volume</span>, you may not get a response right away
and worse, you'll have to <span class="bg-dark-gray white">re-send</span> due to the patch getting lost.

#### Find tasks for quality contribution

__TODO__ files are often _outdated_. You may need to post in the mailing list to ask for ideas.
Personally, I got this problem before but got around it by asking and/or observing discussions and
trends.

#### Quality contribution in the kernel

Sending patches about a relevant task may need discussing with the community first.

_Vaishali_ proposed the project _Kernel Bridge_ to help address these concerns. If you are interested in
contributing, you may check out the slides[^7] and the GitHub[^8] page.


Overall, that was a fun week!

I would like to thank the _Linux Foundation_ for providing travel assistance. 

---
[^1]: [What are Lightning Talks?](https://perl.plover.com/lt/osc2003/lightning-talks.html)
[^2]: [Slides](https://www.slideshare.net/PGExperts/ggtt-linux-2013), Watch in YouTube: [part 1](http://www.youtube.com/watch?v=iE9y3gyF8Kw ), [part 2](http://www.youtube.com/watch?v=gcOP4WQfJl4)
[^3]: This photo, ["OpenSourceSummit_Europe_171023_daily_01-57"](https://www.flickr.com/photos/linuxfoundation/37879135031/) is copyright (c) 2017 Linux Foundation and made available under a [Attribution 2.0 Generic](https://creativecommons.org/licenses/by/2.0/)
[^4]: [Fast and Precise Retrieval of Forward and Back Porting Information for Linux Device Drivers](https://schd.ws/hosted_files/osseu17/66/prequel_oss.pdf)
[^5]: [management-style](http://elixir.free-electrons.com/linux/latest/source/Documentation/process/management-style.rst)
[^6]: [How the development process works](http://elixir.free-electrons.com/linux/latest/source/Documentation/process/2.Process.rst)
[^7]: [Why Should We Care About KernelNewbies!](https://schd.ws/hosted_files/osseu17/c1/Open%20Source%20Summit%2C%20Europe%20-%20Kernelnewbies%20talk.pdf)
[^8]: [KernelBridge](https://github.com/nerdyvaishali/kernelbridge)
