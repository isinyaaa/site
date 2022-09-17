+++
title = "An introduction to the Linux graphics stack"
date = 2022-07-24
updated = 2022-09-17
description = "Alternatively: Yet another introduction to the Linux graphics stack"

[taxonomies]
tags = ["kernel", "starters", "linux", "graphics", "userspace"]
+++

Unless you've run into a terrible problem with your GPU, I bet you never spent
some time actually thinking about how we depend so heavily on graphics to
actually do anything meaningful[^0] on a computer: from small square blinking
letters on a terminal to the utmost insanely complex games we play these days -
in the context of Linux, it all depends on the DRI!

<!-- more -->

This should come as surprise to no one! And as you can probably imagine, the
Linux graphics stack is comprised of many layers of abstractions, going from
the largely abstracted userspace application that you're running (a browser!)
to the bytecode that is actually interpreted by your graphics card.

These many abstractions are what allow us to have our glorious moments of being
a hero (or maybe a villain, whatever you're up to...) without even noticing
what's happening, and -- most importantly -- that allow game devs to make such
complex games without having to worry about an awful lot of details. They're
really a marvel of engineering (!), but not by accident!

All of those abstraction layers come with a history of their own[^1], and it's
kinda amazing that we can even have such a smooth experience with all of those,
community-powered, beautifully thought out, moving pieces, twisting and turning
in a life of their own.

Just have a look at the beast we'll be getting into! Feeling adventurous today,
ain't you? Well, fear not. It's definitely not as bad as it seems, and there's
a lot of logic behind (most) things, so let's go step by step :).

![Linux graphics stack (simplified)](/graphics-stack/linux-graphics-stack.jpeg)

Though in most cases we start from the bottom of the stack and build our way
towards the top, here I think it makes more sense for us to build it upside
down, as that's what we're used to interacting with.

> On a funny thought experiment, maybe driver designers do live upside down,
> who knows...

## Window servers and TTYs

Even though Linux is generally regarded as a developer/hacker OS, most modern
distributions don't require you to ever leave a graphic environment. Even if
you love using the "terminal" on your distro, that's simply a _terminal
emulator_, which "simulates" using a modern TTY[^6].

While TTYs are provided by the kernel itself, we need so-called _display
servers_ to actually render something more complex to the user's screen. It
acts as an intermediate, dealing with multiple applications wanting to draw
things to the screen simultaneously.

{% callout() %}

As Linux was created during an era where computation was very much migrating
from a model of "distributed access" (through time-sharing[^6]), you can still
peek into a TTY if you want to:

> On most distribution, pressing `CTRL+ALT+F1` (or `+F2`, `+F3` and so on) will
> take you to an old-school text-only display (of course one of these TTYs will
> also contain the display server session you began with).

![Classic teleprinter](/graphics-stack/teleprinter.jpeg)

![Modern TTY](/graphics-stack/tty-example.png)

Check out a classic teleprinter in action in
[this video](https://www.youtube.com/watch?v=2XLZ4Z8LpEE) or, if you want to
understand the working details on Linux, check out
[the TTY demystified blog post](https://www.linusakesson.net/programming/tty/index.php).

{% end %}

But then, it seems logical that the graphic session we're used to is one layer
above the OS itself, which only provides those TTYs and, quite likely, the
[Application Binary Interface](https://en.wikipedia.org/wiki/Application_binary_interface)
(i.e. _ABI_) needed to "talk" to the OS, which is indeed the case (currently)
:).

In an X session for example, the X server (or your compositor) will render its
windows through this ABI, and not directly through the hardware. Actually,
the kernel ABI for hardware access through the Direct Rendering Infrastructure
(_DRI_) is a recent development. Before it, the X server would simply access
hardware directly, so all userspace software depended on it, and the slightest
lack of coordination between applications could wreak havoc in the graphics
card 😵!

<p align="center">
  <img align="left" alt="GPU access before DRI" src="/graphics-stack/no-DRI.png" width="50%">
  <img align="right" alt="GPU access with DRI" src="/graphics-stack/DRI-simple.png" width="50%">
</p>

{% callout() %}

The X server has many years of history and lots of "curious" design choices (to
put it lightly). If you're at all interested on understand it better, have a
look at the
[xplain blog](https://magcius.github.io/xplain/article/x-basics.html)!

{% end %}

But enough historic techno-babble! How does our application, then, render its
stuff using a window server? The short answer is: it doesn't!

Actually, anything that wants to render elements that are independent of a
window server will have to use a graphics API like **OpenGL**, and that's where
Mesa comes in!

## Mesa and graphics APIs

First, notice that we're already down a level: the window server shows us
cute windows and deals with user input in that interface, wonderful! But then I
open Minecraft, and we're already asking for 3D objects which the window server
can't possibly handle effectively, so
[Mesa](https://gitlab.freedesktop.org/mesa/mesa) was introduced to provide a
second route for applications to send their complex commands directly to the
kernel, without the need to go through the display sessions' (_bloated and
slow_) rendering mechanisms, which are quite often single-handedly optimized to
show us application interfaces (the infamous GUIs).

{% callout() %}

As a side note, the X server is also OpenGL capable, and you can see this
through its `glx*` API commands, or even some commands (ever heard of
`glxgears`? 😏).

{% end %}

On a birds eye view, our (supposed) game will try to run its 3D routines, which
are written in shader-speak (for example OpenGL's, whose language is called
`GLSL`), which Mesa handles for us, compiling and optimizing it for our
specific GPU, then sending it to be run by the kernel, just like your window
server does, but for anything!

{% callout() %}

An important point to notice here is that, of course, the window server is
still needed for user interaction in our game. It will query and send user
commands to the application, and will also handle windowing and displaying
stuff (including our game, as its window is still managed by the window
server), as well as dealing with many other system-related interactions that
would be a nightmare for game devs to implement.

Another important point is that writing user applications which don't require
using shaders would be a total nightmare without a display manager, as it
provides many useful abstractions for that use case (which already covers >90%
of the uses cases for most people).

{% end %}

We're getting close! Next stop is: the Linux kernel!

## The DRM and KMS subsystems

The Linux kernel is, of course, comprised of many moving parts, including
inside the Direct Rendering Manager (_DRM_) itself, which I cannot possibly
explain in this one blog post (sorry about that!).

The DRM is addressed using `ioctl`s (short for _I/O Control_), which are
`syscall`s (short for _System Call_) used for device specific control, as
providing generalist `syscall`s would be nearly impossible -- it's more
feasible to create one `ioctl(device, function, parameters)` then treat this in
a driver than to create 2000 `mygpu_do_something_syscall(parameters)` for
"obvious" reasons.

{% callout() %}

For the inexperienced reader, it's interesting to notice here that the approach
of one `syscall` per driver-specific command would bloat the general kernel ABI
with too many generally useless functions, as the large majority of them would
be used for one and only one driver.

Just for reference, in my kernel I have 465 `syscall`s as defined in the
system manual. Putting this _vs_ a simple estimate of _drivers in the kernel *
commands only they use_ should give you some perspective on the issue.

> Just use `man syscalls | grep -E '.*\(2.\s+[0-9.]+' | wc -l` if you want to
> query the `syscall`s defined in your kernel.

{% end %}

Those `ioctl`s are then wrapped inside a
[`libdrm` library](https://gitlab.freedesktop.org/mesa/drm) that provides a
more comprehensible interface for the poor Mesa developers, which would
otherwise need to keep checking every `ioctl` they want to use for the
[unusual macro names](https://docs.kernel.org/userspace-api/media/v4l/user-func.html)
provided by the kernel's userspace API.

One very shady aspect of graphics rendering that the DRM deals with is GPU
memory management, and it does this through _two interfaces_, namely:
- **GEM** -- short for _Graphics Execution Manager_
- **TTM** -- short for _Translation Table Maps_

The older of those two is TTM, which was a generalist approach for memory
management, and it provides literally everything anyone could ever hope for.
That being said, TTM is regarded as **too** difficult to use[^2][^3] as it
provides a gigantic API, and very convoluted must-have features that end up
being unyielding. For instance, TTM's **fencing** mechanism -- which is
responsible for coordinating memory access between the GPU and CPU, just like
**semaphores** if you're used to them at all -- has a very odd interface. We
could also talk about TTM's general inefficiencies which have been noted time
and time again, as well as its "wicked ways" of doing things, which abuse the
DMA (short for _Direct Memory Access_) API for one (check out König's talk for
more[^4]).

Just like evolution, an alternative to TTM had to come along, and that was our
friend GEM -- which was conceived by Intel, as a simplified interface for
memory management. It's much easier to use by comparison, but also much more
simplified and thus, only fully attends Intel's specific use case  -- that is,
integrated video cards[^5] --, as it's limited to addressing memory shared by
both GPU and CPU (no discrete video card support at all, really). It also won't
handle any fencing, and simply "wait" for the GPU to finish its thing before
moving on, which is a no-no for those beefy discrete GPUs.

Then, as anyone sane would much rather be dealing with GEM, ~~all software
engineers fired themselves from other companies and went to Intel. I hear
they're currently working hard to deprecate TTM in an integrated graphics
supremacy movement~~. Jokes aside, what actually happened is that DRM drivers
will usually implement the needed memory management (including fencing)
functionality in TTM, but provide GEM-like APIs for those things, so that
everyone ends up happy (except for the people implementing these interfaces, as
they're probably quite depressed).

These memory related aspects are a rabbit hole of their own, and if you'd like
to have a deeper look into this, I recommend these resources:

- [wikipedia.com - DRM](https://en.m.wikipedia.org/wiki/Direct_Rendering_Manager)
- [blogs.igalia.com - Intro to Linux graphics stack](https://blogs.igalia.com/itoral/2014/07/29/a-brief-introduction-to-the-linux-graphics-stack/)
- [studiopxl.com - Linux graphics stack overview](https://studiopixl.com/2017-05-13/linux-graphic-stack-an-overview)

{% callout() %}

It's important to notice, though, that kernel devs are working hard into making
TTM nicer all around, as it's used by too many drivers to simply, paraphrasing
König, "be set on fire" and start a memory interface anew. If you want some
perspective, take a look at Christian König's amazing talk[^4], where he talks
about TTM from the viewpoint of a maintainer.

{% end %}

Notice, however, that the DRM is only responsible for graphics _rendering_ and
display mode-setting (that is, basically, setting resolution and refresh rate)
is done in a separate (but related) subsystem, called **KMS** (short for
_Kernel Mode Setting_).

The KMS logically separates various aspects of image transmission, such as

- **connectors** -- basically outputs on your GPU
- **CRTCs** -- representing a controller that reads the final information
  (known as **scanout buffer** or **framebuffer**) to send to those connectors
- **encoders** -- how that signal should be transmitted through the connector
- **planes** -- which feed the CRTCs framebuffers with data

And you can actually find where this information is available by consulting the
`/sys/class/drm` folder. There you can see subdirectories referring to specific
graphics cards (i.e. `card*`) and also their _render nodes_, which indicate
their capabilities of executing _general programming_ applications, like the
ones in machine learning, which don't depend on having a graphical output.
Inside these subdirectories, then, you can see some files indicating things
like **connection status** or **mode list**.

For a quick overview, check out this image:

![Linux graphics stack](/graphics-stack/linux-graphics-stack-full.png)

I can already hear you say

> OMG Isinya, that was a hell of a lot of information 😵.

But hey, you just read it through (what a nerd)! On a following post I'll talk
about testing the kernel pieces of this puzzle, hope to see you then :).

[^0]: A small disclaimer: of course there are many disabled people which need
  other methods of interaction with a computer that don't rely as heavily on
  graphics, and I do think it's a huge problem that we don't have alternatives
  for those people in a large majority of cases, as most software does not give
  the lightest thought to the vision impaired. I should talk more about this in
  the future but, for now, please don't forget it's an existing issue that
  needs addressing!

[^1]: Which I might even talk about in another post -- excuse me, but that
  requires a little too much research for the time I have currently.

[^6]: A classical TTY (or _teleprinter_ for the acquainted) used to be an
  endpoint for interaction with a central computer that held all the resources
  for users (we call that a _time-sharing_ computer), and was actually not much
  used outside of that context. A terminal emulator would be called a _pseudo_
  TTY (PTY).

[^2]: [lwn.net - GEM v. TTM](https://lwn.net/Articles/283793/)

[^3]: [dri.freedesktop.org - DRM Memory Management](https://dri.freedesktop.org/docs/drm/gpu/drm-mm.html)

[^4]: [youtube.com - the TTM memory manager](https://www.youtube.com/watch?v=MG7_tUNKSt0).

[^5]: The attentive reader might have thought about Intel's discrete video
  cards at this point, and as a matter of fact Intel is actually working with
  TTM to support those products. See:
  [phoronix.com - Linux 5.14 enabling Intel graphics TTM usage for their dGPUs](https://www.phoronix.com/scan.php?page=news_item&px=Linux-5.14-TTM-dGPUs-LMEM)
