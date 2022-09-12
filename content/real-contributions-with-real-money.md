+++
title = "Real contributions with real money"
date = 2022-06-13
updated = 2022-06-25
description = "I'd like 10 pounds of `fReE sOFtWare` please"

[taxonomies]
tags = ["kernel", "starters", "linux", "graphics", "testing"]
+++

Recently I've heard from a friend that his professor simply doesn't believe
that free software should be profitable, so I'm making this blog post.

<!-- more -->

Of course, I don't _just_ want to rub it in his face, I'm also here to talk
about my Google Summer of Code project :). Let's hop into it!

{% callout() %}

**DISCLAIMER:**

For those who don't know, at the moment of writing this I also work with open
source at Red Hat, and of course it is a profitable business otherwise I'd be
unemployed. Notice the context here is very different though as Red Hat not
only provides source code for their projects but also many other products
related to systems maintenance and services, such as customer assistance and
maintenance of customer's product's infrastructure.

It should be also noted that this blog is **in no way what-so-ever related to
Red Hat or any of its subsidiaries, and conveys my personal opinion only.**

Now we can proceed :).

{% end %}

For thy poor readers who don't already know about this,
[Google Summer of Code](https://summerofcode.withgoogle.com/) (GSoC for the
intimate) is an annual initiative for encouraging people (mainly students) to
contribute to open source software projects all around, and there are many
organizations that take part on it, submitting their mentors' ideas as possible
projects, then parsing through to-be-contributors proposals.

Let me explain this in more detail, so you can catch that carp next year:

- The X.Org Foundation published its mentors
  [ideas for projects](https://www.x.org/wiki/SummerOfCodeIdeas/).
- My first mentor, Rodrigo Siqueira, who is also one of X.Org's appointed
  mentors told me about this
- Then I made a proposal[^1] that was very much aligned with his to X.Org in
  GSoC.
- Now we profit!

> By the way, I've already explained how you can find a local community and,
> possibly even mentors on them in a
> [previous blog post](/on-contributting-to-open-source/), so go check it out
> if you're not really sure how to approach these situations.

The most obvious ways Google has to incentivize contributions are, of course,
through money, and also by utilizing their massive reputation to connect people
(that is, contributors and mentors). Now we get a little more philosophical:
why does Google, of all companies in the world, sponsor random people to
contribute to open source?

The implications of this tell us a lot about software engineering, idealistic
thinking, and even capitalism itself!

## Why sponsor something free (as in free beer)?

As you might already know, many of these projects exist not only for their own
sake, but as reliable, auditable, building blocks for larger projects, or even
as tools, making it very easy and cheap (money-wise and also time-wise) to
create whatever you might want. So it happens that by having this sort of, at
first glance, unreasonably undervalued (most times literally free!) tools we
can build much more valuable software and/or media at exponential rates,
because the only thing holding us back is the knowledge to yield them.

Then, as our current financial system much rather values competing alternatives
in an open ~~battle~~ market, it makes a lot of sense for some of this stuff to
be as widely available as possible.

We, of course, can also look at this through idealistic lenses: free (libre)
software objectively makes the world a better place because

- we worry less about security and privacy violations as anyone can audit the
  source code we're using and look for problems;
- we can make whatever changes we like to it (mind your licenses, though);
- but most importantly, **we** can make real social impact by improving such
  code, or adding missing functionality.

That is to say that the real advantage in all of this is that **we** can make
the world a better place for everyone, one commit at a time, and with almost no
barriers to entry!

## The downsides

If you'd never thought about this, of course we have some problems here.

First and foremost, the most obvious issue happens when such software is not
maintained properly (maintainers may lack resources, for example), and as it
might be used by many other projects we might have a broken dependency chain.
This has happened
[recently with Log4j](https://www.ncsc.gov.uk/information/log4j-vulnerability-what-everyone-needs-to-know)
and it's caused a huge turmoil.

Still onto this issue, we're never totally sure if the code has been audited
properly. Sometimes maintainers also aren't as experienced as we'd hope, and
end up making mistakes.

A huge chunk of this problem happens primarily when projects aren't so
interesting and have small user base, or when its maintainers lack appeal (e.g.
they might be blunt on community interactions) and, as a consequence, cannot
build a proper community around it.

This is a problem that large corporations can usually deal with in their
products, as they can just select competent developers and pay them good money
to maintain literally depressing code bases.

Another problem is having _bloated_ or _unruly_ software. Of course this isn't
exclusive to open source (just look at Microsoft Windows for example), but it
can be made worse, as maintainers might have different views on what's
important and no consensus as to which path to follow.

Using the Linux kernel as an example, even with Linus as the head of the
project it still has had a history of too many ABI changes[^2] [^3] and still has many
issues caused simply by leaving code "unattended" (i.e.
[introduction of hypocrite commits](https://portswigger.net/daily-swig/ill-advised-research-on-linux-kernel-lands-computer-scientists-in-hot-water),
or
[easy to solve compilation warnings that were simply overlooked](https://lore.kernel.org/all/20211208012529.372478-1-isabbasso@riseup.net/)).

This might have been a product of overworked maintainers who have no time to
review every piece of every patch, coupled with their trust onto familiar
developers who have no better ways to check if their contributions are up to a
certain threshold than to run some scripts on them or wait for some CI to fail
and warn them.

{% callout() %}

This might not be the full picture of upsides _vs_ downsides, but it's
certainly one which I've been very exposed to recently, so that's my take on
it.

I recommend having a look at
[Rosenzweig's blog post](https://rosenzweig.io/blog/software-freedom-isnt-about-licenses-its-about-power.html)
for a more in depth discussion on issues regarding licensing and open source
software.

{% end %}

## Giving a hand to our beloved projects

With all of this in mind, I personally believe that as starters in this
contribution journey the most valuable thing we can give back to maintainers is
to provide fixes for overlooked problems, or even better, provide ways to help
more experienced developers make better code.

What do I mean with this? Simply put

- If code breaks a lot, make a test for it!
- If it has too many small problems that should not be overlooked, fix them!
- If it misses documentation, understand it and upstream your newfound
  knowledge!

It's literally **that simple**!

What gets to me is that those are usually very simple things, and a new
contributor might want to make contributions as heavy as those they see
maintainers making. But that's just **not realistic** for most newbies.

And of course it's a lot less interesting to send PRs adding a test than it is
to upstream emoji support for Intel graphics drivers, but you often gotta start
small, kiddo.

So, onto my GSoC project, what the hell am I even doing?

## Isinya's GSoC 101

As alluded to above, I'm adding tests to graphics drivers in the kernel.
Specifically to the AMDGPU driver, which is a ginourmous beast of a thing,
being literally the
[largest in the Linux kernel currently](https://www.phoronix.com/scan.php?page=news_item&px=AMDGPU-Closing-4-Million).
Of course that with great powers come great headaches and so it happens that
the AMD driver for its GPUs has many shady pieces of code, and a core
representative of which is its DML submodule that, roughly speaking, is
responsible for providing the absolute best timing values for internal GPU
components, and it achieves this through a series of
[unorthodox](https://github.com/torvalds/linux/blob/master/drivers/gpu/drm/amd/display/dc/dml/dcn20/display_rq_dlg_calc_20.c)
floating point calculations.

> For the matter, I'll specifically be working with this file, wish me luck!

<!--TODO: missing references
For starters, it wasn't even written by human hands. This submodule, my friends,
was procedurally generated! -->

Another thing that makes it an absolute nightmare to deal with is that floating
point routines are simply not welcomed in
[Linux's kernel code](https://stackoverflow.com/questions/13886338/use-of-floating-point-in-the-linux-kernel).
No one likes to even review that.

{% callout() %}

Of course some kernels (like Window's) do have floating point support, and
there's no strong argument to not have it nowadays, as modern CPUs are pretty
well optimized for this -- just look at modern SIMD instruction sets for
example (SSE2 onwards) --, and as a matter of fact, modern GPUs can perform
just the same whether we're using them for integers or floating point numbers!

{% end %}

<!-- TODO: get a better reference for that -->

All of this conspires to a perpetual state of "just let it be"
([perpetuated by the engineers themselves...](https://github.com/torvalds/linux/blob/master/drivers/gpu/drm/amd/display/dc/dml/dcn20/display_rq_dlg_calc_20.c#L56)),
and we end up with bloated, hard-to-read routines, which are unfortunately core
to their most cutting-edge GPUs.

Such is the rabbit hole I got into! But not alone, mind you,
[Maíra Canal](https://mairacanal.github.io/gsoc-22/) and
[Thales Aparecida](https://tales-aparecida.github.io/tales-tips-and-tricks/posts/gsoc-report-1/)
will be joining me for lots of ~~crying in the bathroom~~ I mean **FUN**! Lots
of fun! [Magali Lemes](https://magalilemes.github.io/) will also be joining our
party while doing her final work for undergrad, so stay tuned :).

I also managed to clone my mentor, Rodrigo Siqueira, and now I have three!
Starring: [Melissa Wen](https://melissawen.github.io/) and
[André Almeida](https://andrealmeid.com/)!

I hope to learn a lot, being in such great company, and with them, we should be
able to at least "sweep" some piles of sand from under AMD's GPU driver carpet
(or so I hope).

If you're wondering what exactly we're proposing, you can look at my project
submission[^1], but in short we should be creating tests for some DML files,
which is a task that in itself already encompasses many technical obstacles,
like testing in real hardware, or mocking it, and also ensuring that these
tests are compatible with IGT, which I'll talk about in my next blog post.
We're also concerned with generating coverage reports for the tests,
refactoring these monstrous files, and documenting them better.

Last but not least, GSoC also appreciates community presence, so you should see
more stuff around here, and I've also set up an IRC bouncer to stay up-to-date
24/7 with some communities (ping me [isinyaaa] anytime on #dri-devel if you
wish 😊).

See you later alligator!

[^1]: [My GSoC proposal](https://summerofcode.withgoogle.com/programs/2022/projects/6AoBcunH).
  If you want to see it in full, just ping me and I'll be happy to share :).

[^2]: [Kernel ABI tracker](https://github.com/lvc/kernel-abi-tracker)

[^3]: [ABI tracker preview (outdated)](https://abi-laboratory.pro/?view=timeline&l=linux)
