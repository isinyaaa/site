+++
title = "GSoC '22 final report"
date = "2022-09-10"
description = "DRM AMDGPU Unit tests"

[taxonomies]
tags = ["kernel", "linux", "graphics", "unit tests"]
+++

Like all things, Google Summer of Code, too, comes to an end.

Now let's go over what had to be done, what is done, and what's left.

<!-- more -->

## What had to be done

Well, this is rather easy for me to talk about, I'll be on X.org's Developers
Conference soon, and full of motivations behind the work I've done.

{% callout() %}

Not just me, though! Me, Maíra and Magali (who might be familiar names to you
already) will be there as well, and unfortunately Tales didn't manage to get a
visa due to bureaucracy layers no one dares to understand.

{% end %}

Looking retrospectively, the project's motivation boils down to Siqueira
wanting to resolve an internal tension between AMD engineers and the weird code
they have to manage, unit tests being like a safety valve. As I've talked about
previously, GPU code can be quite _intense_, the DML module being a
particularly fun example.

No tests, just procedurally generated stuff, and there is so much to be done,
really. Initially we proposed lots of tests, some docs, some refactoring, in my
case specifically I wanted to make the
[`drivers/gpu/drm/amd/display/dc/dml/dcn20/display_rq_dlg_calc_20.c`](https://github.com/torvalds/linux/blob/master/drivers/gpu/drm/amd/display/dc/dml/dcn20/display_rq_dlg_calc_20.c)
(breathes **heavily**) look pretty.

So...

## What was actually done

### On the Kernel

Though we all had AMD code enhancement as a primary motivator, I actually did
some unit tests, and at the end I was caught up in trying to understand what we
were testing.

Wasn't my smartest move (what external contributor actually knows what an RQ
DLG is? Difficult answers to find, really) but I learnt so much, apart from the
fact it was really satisfying to document something so convoluted and full of
internal "slang" (I mean product (code)names or acronyms). But I soon found out
I'd be a terrible detective, as most of the things we had to test were simply
unattainable for someone who didn't have internal docs or lots of experience
with real GPUs: most things in the DML submodule refer back to themselves, to
GPU internals, or even to AMD "internals".

In the end there were some interesting results from the tests I was actually
able to conclude, as follows:

- [drm/amd/display: Introduce Kunit tests to display_rq_dlg_calc_20](https://lore.kernel.org/all/20220811004010.61299-4-tales.aparecida@gmail.com/)

   This was a single patch I've done on the unit tests. Of course there are
   lots more functions to be tested, but that comes in the next section :)

   We collectively discussed some alternatives to deal with the fact the
   functions being tested were all static, as will be discussed by Tales in his
   presentation at the Linux Plumbers Conference.

   [LPC: How to introduce KUnit to physical device drivers?](https://lpc.events/event/16/contributions/1310/)

   There was also some documentation produced regarding the functions and
   structs involved in these tests, but they were not sent yet as they're
   smaller changes and could be grouped in a patchset addressing this
   specifically.

### On other projects

#### KWorkflow

KW (for the intimate) is a much needed and very interesting project, whom
I tried my best to contribute to: I spent about a month and a half at the
beginning of GSoC pushing it, to the point where I simply had no will to make
my commit messages pretty or to respond maintainers.

Lucky me the owner of the project is also my GSoC mentor, and he completely
understood where I was at and that I'd not be able to accomplish the (optional)
goals I had set for KW in my proposal.

I really think this situation helped me understand better what is it that we're
doing when we contribute to free software, and that was the lesson I took.

Anyhow, there were many contributions during this period, even though I didn't
finish many of them:

- [#614: Small code cleanup](https://github.com/kworkflow/kworkflow/pull/614)
- [#615: Small build fixes](https://github.com/kworkflow/kworkflow/pull/615)
- [#616: Separate build config from base kw config](https://github.com/kworkflow/kworkflow/pull/616)
- [#617: Speedup build](https://github.com/kworkflow/kworkflow/pull/617)
- [#619: Update CI](https://github.com/kworkflow/kworkflow/pull/619)
- [#620: Add rsync dependency](https://github.com/kworkflow/kworkflow/pull/620)
- [#631: Update KW setup](https://github.com/kworkflow/kworkflow/pull/631)

#### IGT GPU Tools

Some things took a lot of our attention apart from the basic motivations of the
project. As I was previously already working on IGT GPU Tools, I figured it'd
be interesting to finish related work in that project. What good is making
tests people aren't going to use (or maintain?).

IGT is widely used by the kernel graphics people. It mainly tests GPU stuff
using userspace APIs, but also does some other interesting things, and we
figured it'd be very cool to be able to run unit tests there: easy to integrate
with the pre-existing CI, not too much headache to maintain as the KTAP specs
get more stable, as well as not requiring so much effort from engineers that
are already so used to it.

- [patchset: Add support for KUnit tests](https://lists.freedesktop.org/archives/igt-dev/2022-August/044928.html)

  Whilst it was not so difficult from the purely technical point of view, this
  might be what helped me the most towards acquiring some intimacy with the C
  language. I felt it deep when I came back to the same code after a month or
  two and saw so much to improve on it, then stepped out again as I waited for
  v1 to be reviewed (which didn't happen) and came back a while ago to write
  the v2 linked above.

  There were several challenges involved, specially for me: I started coding C
  because of the kernel, all I knew were C-like languages, and they were all
  masking some deep truth about computers to me: pretty error handling, easy to
  use strings, etc.

  I admit I wasn't as ready for C as I thought, now that I understand it
  better, I can see that clearly. In the end this "quasi-traumatic" experience
  (and I mean the IGT patchset specifically) taught me a lot, and I'm very
  thankful for that opportunity (thanks, André!).

  Apart from the language itself, we also had to decide what would be supported
  in a KTAP parser, and that wasn't easy.

### Blog posts

Well, for me personally this was the best part of this project, I really
enjoy writing these things, I also enjoy receiving feedback -- specially if
it's unasked :).

If you know my blog you know I like to go the difficult route: to talk about
the objective engineering experience might teach you a lot, but where's the fun
in that? Even now I'm trying to bring some subjective matters to this.

Anyhow, I published two blog posts which you can check out here:

- [Real contributions with real money](https://crosscat.me/real-contributions-with-real-money/)
- [An introduction to the Linux graphics stack](https://crosscat.me/an-introduction-to-the-linux-graphics-stack/)

And I actually made lots of contributions to the Flusp site as well, whose
repository was used to review these posts, which I hope will be hosted there as
long as they're hosted on my own website.

- [#95: add reports to site bar](https://gitlab.com/flusp/flusp.gitlab.io/-/merge_requests/95)
- [#100: config: add english feed button](https://gitlab.com/flusp/flusp.gitlab.io/-/merge_requests/100)
- [#101: clean and update Jekyll dependencies in Gemfile](https://gitlab.com/flusp/flusp.gitlab.io/-/merge_requests/101)
- [#102 (blog post review): Add graphics stack blog post](https://gitlab.com/flusp/flusp.gitlab.io/-/merge_requests/102)
- [#103 (blog post review): add Isabella's GSoC blog post](https://gitlab.com/flusp/flusp.gitlab.io/-/merge_requests/103)
- [#107: \_pages/tutorials: add 'kernel' category for posts](https://gitlab.com/flusp/flusp.gitlab.io/-/merge_requests/107)
- [#117: Add last updated date and fix feed](https://gitlab.com/flusp/flusp.gitlab.io/-/merge_requests/117)

## What still needs to be done

As I pointed out previously, GPU code can be very interesting! So much so, that
I purposefully didn't attempt to rush what was left of my proposal at the end
of GSoC. It might have been really satisfactory to end GSoC with everything
accomplished, but if I learnt anything from undergrad it is that you should NOT
rush what's important to you.

From what I've talked with my peers, there are two ways of seeing this project:

1. GSoC is like a deal I made with Google and X.org to accomplish something
   until a due date and get money for it.

    This PoV is okay, but does it teach you anything new? I might as well have
    done some freelancing in that time period, would have got the money, and
    then I'd be very comfortable.

    But I believe there's so much more to this experience, and that's something
    Siqueira told us time and again, which brings me to a second, more
    wholesome way of seeing things:

2. GSoC is a sort of first commitment to a community.

    I know this doesn't sound so clear as the first way of looking at things,
    but follow me on this:

    For the community, timing might be important, but it's definitely not as
    important as doing solid, good work, and keep pushing it even if it's not
    as quickly as you'd like.

    I got really burnt out from the first two years at Uni, started working,
    went to live by myself, all that young adult jazz. Trying to always keep up
    with everything was a real challenge, and though I didn't always give my
    best, well I really tried.

    At first, I was really sad, almost spiraling out of everything
    software-related, but now I see things more clearly, and I'm trying to find
    a rhythm that I can work with, in which I can deliver what I want, and,
    most importantly, what I committed to.

    Going back to Siqueira, what he told us is that (translating loosely):

    > You got to make a dent in the community so that someone notices you.

    And now I see it more clearly.

    I've heard so many stories of people who engage on things like GSoC or
    Outreachy just to put it on their CV, then quit, but this is so much more
    important to me.

    I've already started making a career in free software, but the project I
    work on at the moment doesn't allow me to interact so directly with some
    upstream or a community. I definitely want to improve in doing I enjoy and
    believe in, and if it takes some learning that's only part of the journey
    to becoming a reliable member of some free software community.

So, in concrete terms, what is there to finish?

1. First and foremost, the ongoing patchset for IGT needs lots of attention, as
   pointed by Michał Winiarski in this thread for example:

    [Kernel lore archives: \[PATCH v5 9/9\] drm: selftest: convert drm_mm selftest to KUnit](https://lore.kernel.org/all/20220708203052.236290-10-maira.canal@usp.br/)

2. Secondly, of course, finish the original proposal of testing the
   `dcn20/display_rq_dlg_calc_20.c` file.

Might not sound like a lot, but those are really important things, and I'm sure
they'll keep me busy for some time.

I was also pinged by some people to review their patches, and I want to get
back to them soon.

I've really been thinking a lot about giving back to the people that helped me
get here, they were all awesome and I hope I can sow these same seeds and help
fellow students become contributors as well. Just last week me and Maíra
decided to try to get some people for a project on (fixing) coverage reports
for the tests we wrote, but I might as well find another project for myself --
probably related to XR, if you're wondering, but I should write about that in
the future :)

That's it, dear reader, thank you a lot for reading this through, see you on
the next one!
