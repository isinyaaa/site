+++
title = "On Contributing to the Kernel (pt. 1)"
date = "2021-10-04T20:00:00-03:00"
author = "Isabella Basso"
authorTwitter = "isinyaaa"
cover = ""
tags = ["kernel", "starters", "linux"]
keywords = ["", ""]
description = "You gotta start somewhere, right?"
showFullContent = false
+++

So, for starters, the Linux Kernel is the most beautiful thing ever ~~if you don't mind a little mess, I mean...~~ **no, really**, it's beautiful and I'll prove it to you! All you need to do is clone the "main" tree:

```bash
git clone https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
```

Well, actually, this isn't _really_ the main thing but it's _one of them_, together with the one you probably know best (if you're a rolling release Linux user, that is) [Greg Kroah-Hartman's tree](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/) and that's only half the story, as each Linux distribution likes to put their own flavor of commits on top of that.

> A useful tip if you want to 'check out' both trees is to append remotes to your cloned repo, as it doesn't make sense to clone 2GB repos that share 90% of their code 'just because'… If you don't know how to do that it goes on the lines of:
>
> ```bash
> git remote add X https://link.to.x/git/repo.git
> git fetch X
> git checkout X/master
> ```

Once you have all that, I'd appreciate it if you could take the time and just peer at the massive `git log` of this thing (I personally like using the Git Graph extension on VScode/VScodium):

![kernel_graph.png](/kernel-contrib/kernel_graph.png)

I mean, we have almost 1.1mi commits and some 10k developers working on it, all from different backgrounds and places, a couple hundred from competing companies even, and they all come together here to code for this project.

I'm not gonna lie and tell you they all love it, of course a lot of them only do this for the money, but even if you can't find the beauty in this huge effort of human collaboration, there's the sheer power of it to be admired… And if you think it's probably the fruit of a lawsuit in the first place one can only really wonder how important of a role licenses have played so far… But enough talk!

## Contributions

I think it'd be pretty cool to have my name in there, I mean, it **is** a huge collaborative project, so your contributions probably aren't going to stand out a lot, but I prefer thinking about my code potentially helping a couple hundred devices (maybe even thousands? who knows!), or just thinking that you'll forever have your name in the list of contributors of the **largest open-source project ever** 😲. So, with that said, how can you have your little, humble, commit in there?

### What do they look like?

Well, each commit for the Kernel is called a _patch_, and you can pretty much think of it as a minimal piece of code that has a very well-aimed goal, such as a fix for a processor X or to add compatibility to a weird phone Y, or even to add some tests for some part of the code that maybe forgotten by mankind.

And why should this be _minimal_? I mean, people **do** comment on kernel code, and they will also add some verbose names and whatnot… But their **diffs** must be kept to a minimum! No one would dare "fix random whitespace in file X", not in a kernel patch.

The reasoning here is simple: we want to minimise the amount of effort that it'd take a maintainer to read through the changes we're proposing. The less unnecessary stuff, the better!

Then you can argue that the examples I gave don't seem minimal at all! How can one add compatibility to a whole new piece of hardware with a single commit? Well, I lied… You probably can't do 'a patch' for this. We're actually talking about a _patch series_, think of it like your branch of commits, and then each commit is a patch, very minimal, very focused, but we're thinking big, and our series is all about that final goal!

Another very important principle is that commits have to be _descriptive_, and that's a real pain to learn, I swear I'm trying, but it never seems enough. I don't want to sound dumb in the internet, you probably don't want it either, so being thorough without being verbose ends up being a real challenge! You have to write _really well_ and always keep in mind that those who're reading may not have taken a look at what you're editing, like, never (true story)!

So a commit message for the kernel must have the main "pieces of code" you edited in the commit title, like `drm/amdgpu, mm` (that'd be a patch for the `mm` folder as well as the `drivers/gpu/drm/amd` folder) followed by the proper edit you made, such as `Fix oops in trace_cachefiles_mark_buried due to NULL object` (feel free to take a look at this commit: [6e9bfdc](https://github.com/torvalds/linux/commit/6e9bfdcf0a3b1c8126878c21adcfc343f89d4a6a)).

Some good practices should be known here: always write your commits imperatively! I mean, I've seen some messages where this isn't so true, but at least the title must be this way, and then for the description you have a little more freedom I guess, just do it responsibly! ⚠️ Don't dare stick some 'I changed X for clarity' ⚠️ (and keep your emojis safe at home, too!)

### Where do they live?

Well, this section isn't so much about where the commits live (I mean, you already know that, right?) but actually, how **you** can find a piece of code to have your chance at contributing! And please do check out [my next blog post](https://isinyaaa.github.io/posts/on-contributing-to-the-kernel-2/) if you're looking for such a thing (not necessarily related to the Linux kernel, by the way!).

If you're at some uni, you probably know someone that knows about a free software student group, or maybe there's such a group on another uni close to where you are. Please, don't feel afraid of contacting them! If you truly want to contribute your interest will show, and people oftentimes want to help!

If you aren't at some uni, I still think going through those student groups is rather safe, but you can try your hand at some alternatives, like the [kernel newbies](https://kernelnewbies.org/) advices (and they even have an IRC channel).

It may seem very difficult at first, but hey, you're reading through this, I think you can manage :D.

## After you've made your patch series

So, you've done it, you've made your own first kernel patch 2000 pro series X, I'm so proud! How do you get it accepted? Easy enough as it may seem, there are no pull requests around here: it's mailing list time and your mom isn't coming to get you 😢. But fear not, it's **really not** as bad as it seems! Of course you might have some trouble with setting up `git-send-email`, or even finding the maintainer for your piece of code, but in the end you're just dealing with other people, who have also been where you are and also made many mistakes…

### Sending e-mails via the terminal

That really seems like the worst part of all: how the hell do you do this? Well, if you haven't noticed yet, `git` was the `systemd` before `systemd` 😟. But jokes aside, setting up `git` to send e-mails is actually pretty easy, and I recommend following through [this site](https://git-send-email.io/)'s tutorial (at least on how to set `git-send-email` up).

### Maintainers

The first thing you should do before actually e-mailing your patch series is looking for the _maintainer_ of the code you messed around with, as they're the person who's going to _yay_ or _nay_ your changes in the end. You can think of a maintainer as the "owner" of a piece of code (it's not actually like this, but it might be easier to picture it this way…). You can use the `scripts/get_maintainer.pl` script to find this all-powerful being you should be mailing your patches to, and it _should_ work most of the time, not always though, for these times you can `git blame` the file you're working on to see who might be able to (a) review your changes, and; (b) help you with submitting it.

### Mailing lists

You should also check out some mailing lists related to the subsystem you contributed such as the `dri-devel` or the `kunit` mailing lists. There is also the general kernel list `linux-kernel@vger.kernel.org`, which you should most likely always send your patches to.

### Sending patches

Now we can finally send our patches! It's usually a good practice to format your patches before sending them by doing:

```bash
git format-patch -[n] --cover-letter -o [output folder]
```

where `[n]` refers to the number of commits you want to include, and then we put them in a folder to keep them all together. The cover letter is a little something special we will use to explain the context behind our patch series. It will be sent as the first e-mail when you mail everything.

Needless to say you actually have to edit the cover letter file for it to make any sense, right? Go on, I'll wait 💅.

Then, to send our e-mail(s), we have two main fields to fill out, the `to` and the `cc`. The `to` field is who you're actually addressing with your patch series (like the maintainers!), and then the `cc` should be for all the lists (and maybe your friends, also!). The command should be a little like this:

```bash
git send-email [output folder] --to="maintainer@mail.com" --cc="linux-kselftest@vger.kernel.org,linux-kernel@vger.kernel.org,kunit-dev@googlegroups.com,~lkcamp/patches@lists.sr.ht"
```

See you on part 2!
