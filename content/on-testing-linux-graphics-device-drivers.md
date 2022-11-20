+++
title = "On testing Linux graphics drivers"
date = "2022-06-11"
description = "I sincerely hope you're not reading this out of necessity"

[taxonomies]
tags = ["kernel", "starters", "linux", "graphics", "testing"]
+++

As you can probably imagine, the Linux graphics stack comprises many layers of
abstractions, from the pretty little button from which you open your
Proton-able AAA Steam title to the actual bytecode that runs on whatever
graphics card you have installed.

<!-- more -->

These many abstractions are what allow us to have our glorious moments of being
a hero (or maybe a villain, whatever you're up to...) without even noticing
what's happening, and -- most importantly -- that allow game devs to make such
complex games without having to worry about an awful lot of details. They're
really a marvel of engineering (!), but not without their costs. Someone
still's gotta put in their hard work into making those ABIs foolproof and the
result of their usage _unbreakable_ (so to speak).

If you have any experience with development, you might already know the answer
to debugging such complex scenarios: simplify and test every bit of behavior
that might show up. In a simpler context, you might have an API that provides
various kinds of mathematical functions, and to free it from bugs you might
design what are called **unit tests**, which are simply function-scoped tests
that aim for covering possible use cases, like the example below.

{% callout() %}

```python
def sum(x, y):
  return x + y
```

```python
def test_sum():
  assert sum(1, 1) == 2, "Failed to compute"
```

{% end %}

In the first piece of code above, we have a very simple summation function,
then we have an `assert`ion, which will scream at the developer if it fails.
That's a very simple way of testing stuff, roughly we're looking at specific
results we expect for a given function. Another interesting thing we might do
is test edge cases, for example, what happens if this sum overflows? This kind
of testing can smooth out undefined behavior that we might not have accounted
for, and could break other functions that expect sensible results when calling
our simple summation.

So, how exactly does this work in the context of graphics? What should we even
test?

As the reader might have suspected, the stuff we test for game functionality
differs totally from what we should test in the kernel, so it's worth looking
at the different _abstraction layers_ we have on our beloved Tux machine 🐧.

## A bird's eye view of the Linux graphics stack

Though in most cases we start from the bottom of the stack and build our way
towards the top, here I think it makes more sense for us to build it upside
down, as that's what we're used to interacting with.

### The X server and TTYs

Even though Linux is generally regarded as a developer/hacker OS, most modern
distributions don't require you to ever leave a graphic environment. Even if
you love using the "terminal" on your distro, that's simply a _terminal
emulator_, which simulates using a TTY, which used to be an endpoint for
interaction with a central computer that held all the resources for users. As
Linux, and specifically the X server, were created during an era where
computation was very much migrating from that model of "distributed access",
you can still peek into a TTY if you want to! On most distribution, pressing
`CTRL+ALT+F1` (or `+F2`, `+F3` and so on) will take you to an old-school
text-only display (of course one of these TTYs will also contain the display
server session you began with).

But then, it seems logical that the graphic session we're used to is one layer
above the OS itself, which only provides those TTYs and, quite likely, the
ABI needed to "talk" to the OS, which is indeed the case (currently) :).

In an X session for example, the X server (or your compositor) will render its
windows through this ABI, and not directly through the hardware. This is
actually a recent development, as before the current graphics stack was
implemented (it's called **DRI**, which is short for _Direct Rendering
Interface_) the X server used to access hardware directly.

But enough history techno-babble! How does our pretty game, then, render its
stuff using the X server? The short answer is: it doesn't!

Actually anything that wants to render "non-X" elements will have to use a
graphics API like **OpenGL**, and that's where Mesa comes in.

### Mesa and graphics APIs

First, notice that we're already down a level: the X server shows us
cute windows and deals with user input in that interface, wonderful! But then I
open Minecraft, and we're already asking for 3D objects which the X server can't
possibly handle effectively, so Mesa was introduced to provide a
second route for applications to send their complex commands directly to the
kernel, without the need to go through the display sessions' (_bloated and
slow_) rendering mechanisms, which are quite often single-handedly optimized to
show us application interfaces (the infamous GUIs).

> As a side note, the X server is also OpenGL capable, and you can see this
> through its `glx*` API commands, or even some commands (`glxgears` is an
> infamous example!)

On a birds eye view, our (supposed) game will try to run its 3D routines, which
are written in shader-speak (for example OpenGL's, whose language is called
`GLSL`), which Mesa handles for us, compiling and optimizing it for our
specific GPU, then sending it to be run by the kernel, just like the X server
does, but for anything!

{% callout() %}

An important point to notice here is that, of course, the X server is still
needed for user interaction in our game. It will query and send user commands
to the application, and will also handle windowing and displaying stuff
(including our game, as its window is still managed by the X server), as well
as dealing with many other system-related interactions that would be a
nightmare for game devs to implement.

Another important point is that writing user applications which don't require
using shaders would be a total nightmare without a display manager, as it
provides many useful abstractions for that use case (which already covers >90%
of the uses cases for most people).

{% end %}

We're getting close! Next stop is: the Linux kernel!

### The DRM and KMS subsystems

The Linux kernel is, of course, comprised of many moving parts, including
inside the **DRM** (short for _Direct Rendering Manager_) itself, which I
cannot possibly explain in this one blog post (sorry about that!).

The DRM is addressed using `ioctl`s (short for _I/O Control_), which are
`syscall`s (short for _System Call_) used for device specific control, as
providing generalist `syscall`s would be nearly impossible -- it's easier to
create one `ioctl(device, function, parameters)` then treat this in a driver
than to create 2000 `mygpu_do_something_syscall(parameters)` for "obvious"
reasons.

{% callout() %}

For the inexperienced reader, it's interesting to notice here that the approach
of one `syscall` per driver-specific command would bloat the general kernel ABI
with too many generally useless functions, as the large majority of them would
be used for one and only one driver.

Just for reference, there are in my kernel 465 `syscall`s as defined in the
system manual. Putting this _vs_ a simple estimate of _drivers in the kernel *
commands only they use_ should give you some perspective on the issue.

> Just use `man syscalls | grep -E '.*\(2.\s+[0-9.]+' | wc -l` if you want to
> query your kernel's defined `syscall`s.

{% end %}

Those `ioctl`s are then wrapped inside a `libdrm` library that provides a more
comprehensible interface for the poor Mesa developers, which would otherwise
need to keep checking every `ioctl` they want to use for
[weird macro names](https://docs.kernel.org/userspace-api/media/v4l/user-func.html).

One very shady aspect of graphics rendering that the DRM deals with is GPU
memory management, and it does this through _two interfaces_, namely:
- **GEM** -- short for _Graphics Execution Manager_
- **TTM** -- short for _Translation Table Maps_

The older of those two is TTM, which was a generalist approach for memory
management, and it provides literally everything anyone could ever hope for.
That being said,
[it's regarded as **too** difficult to use](https://lwn.net/Articles/283793/),
as it provides a gigantic API, and very convoluted must-have features that end
up being unyielding -- i.e. its **fencing** mechanism -- which is responsible for
coordinating memory access between the GPU and CPU -- just like **semaphores**
if you're used to them at all.

Just like evolution, something better than TTM had to come along, and that was
our friend GEM, Intel's interface for memory management. It's much easier to use
by comparison, but also much more simplified and thus, only attends their
specific use case (that is, integrated video cards), as it's limited to
addressing memory shared by both GPU and CPU (no discrete video card support at
all, really). It also won't handle any fencing, and simply "wait" for the GPU
to finish its thing before moving on, which is a no-no for those beefy discrete
GPUs.

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

- [wikipedia.com/DRM](https://en.m.wikipedia.org/wiki/Direct_Rendering_Manager)
- [blogs.igalia.com/intro-to-linux-graphics-stack](https://blogs.igalia.com/itoral/2014/07/29/a-brief-introduction-to-the-linux-graphics-stack/)
- [studiopxl.com/linux-graphics-stack-overview](https://studiopixl.com/2017-05-13/linux-graphic-stack-an-overview)

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

{% callout() %}

For a quick overview, check out this image:

![Linux graphics stack](/linux-testing/linux-graphics-stack.jpeg)

{% end %}

I can already hear you say

> OMG Isinya, that was a hell of a lot of information 😵.

But fear not my dearest reader, we're already there, you've made it, let's talk
about testing!

## Testing the DRM and KMS

There are a number of tests inside the kernel itself (ok, maybe there are not
so many, I'll give into that), nevertheless, as the kernel links most of its
components statically, except for modules, you're in for trouble parsing the
kernel output on your own. We also might want to test the kernel ABIs in user
space (that is, for example, testing if a command to clear the screen actually
does something from our point of view).

With these goals in mind, Intel introduced **IGT** -- previously short for
_Intel GPU Tools_, now it's just a recursive abbreviation for IGT GPU Tools.
These tools can then be used for performing such tests in a non-chaotic manner.

### Setting up for tests

#### Getting _le kernel_

First, you of course need a kernel to test, so just go get yours. I'll
recommend the AMD upstream branch as that's what I'm using on my project right
now:

```bash
$ git clone https://gitlab.freedesktop.org/agd5f/linux  # amd upstream repo
$ cd linux
$ git checkout --track amd-staging-drm-next  # amd's most up-to-date branch
```

It's important to notice that in order to build this you need some
dependencies. On Arch-based distros you only need `base-devel` and the `bc`
package, then you're all set up!

We can get a generic `.config` for x86, as that's easier:

```bash
$ make -j x86_64_defconfig  # gets default configs for most settings
$ make -j olddefconfig  # now we set the remaining configs to their default values
```

{% callout() %}

A kernel `.config` holds all the information the kernel requires for a build,
like which drivers will be built as modules or will be statically linked (we
call that **builtin**), as well as some settings for CPU frequency scaling
support or number of cores to allow during parallelization, for example.

{% end %}

You can also do `make -j menuconfig` at this step to set any custom configs you
want :). At GSoC we're working with KUnit, which is a unit testing tool for the
kernel, and you can try running some of its tests if you want! (Not yet
supported through IGT as of the date of publishing, but I'm working on it 😉).

Now compile (`make -j$(nproc)`) and install in a VM (which I'll not describe
here, but you can have a look at
[this blog post](https://andrealmeid.com/post/2020-03-10-bootstrap-arch/)).

#### Getting _le IGT_

Now for IGT you should enter your VM and get its repo from
`https://gitlab.freedesktop.org/drm/igt-gpu-tools`.

To run the tests, you probably need some supported hardware, which of course
excludes NVidia graphics cards (for now![^1]) unless you're running basic KMS
tests under the `nouveau` driver.

One important point is that for running any other hardware specific tests on a
VM, most of them will require direct access to the GPU, so you'll either need
to **not** use a VM and run them in the host machine instead _or_ set up PCI
passthrough, which should not be so complex but might leave you with no
graphics display whatsoever -- read a proper tutorial thoroughly if you want to
do it, please. Newer hardware should have support for
[passthrough through OVMF](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF),
which seems a lot simpler to set up, so check that out, maybe.

Then, to run our tests we first need to build IGT. It's important to notice it
has a lot of dependencies, so here is a list of them (Arch-based distro package
names):

```raw
meson ninja alsa-lib libdrm gsl libxmlrpc cmake liboping libxrandr peg pixman
cairo valgrind
```

Notice that here you'll probably also need some `base-devel` packages.

After installing those, you can build IGT by issuing:

```bash
$ meson build && ninja -C build
```

### Running _da tests_

To run tests it's actually quite simple, you can either run a specific one
with

```bash
$ sudo ./build/tests/[test-you-want-to-run]
```

Or you can use their scripts, like `scripts/run-tests.sh`. Passing `-h` will
show you options for it.

As you can see its tests are basically separate by some general ones in the
base `tests` folder, such as those targeting KMS, then there are driver
specific ones, each on their own subfolder.

And that's basically it, you just learned to, somehow, operate IGT, I'm again
very proud of you, dear reader! See you on the next one :).

[^1]: NVidia has recently made a public effort to open source their Linux
  kernel drivers at
  [GitHub/NVidia](https://github.com/NVIDIA/open-gpu-kernel-modules),
  unfortunately, as they were previously using a binary blog of their own
  obscure making, it's not really compatible with DRM/DRI, thus it still needs
  much work before we can actually use (and test) it, at least properly.
