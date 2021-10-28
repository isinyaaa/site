+++
title = "On Contributing to the Kernel (pt. 2)"
date = "2021-10-07T14:55:33-03:00"
author = "Isabella Basso"
authorTwitter = "isinyaaa" #do not include @
cover = ""
tags = ["kernel", "starters", "linux"]
keywords = ["", ""]
description = "What if I don't wanna do kernel? :("
showFullContent = false
+++

Alright, so you came for part 2 of my very first soap opera! I'm excited to have you here, stranger :)

Today I'll talk about that very time I sent my first patch, which is like, still going on, but hopefully I'll make it to the next release (5.16 if I get the numbers correctly).

Oh and I haven't talked about releases yet! My bad :p but I promise I'll talk about it soon :)

## Context

So, before I was here lecturing you on the kernel, I was just your average Arch user: system broke, fixed system, me happy. But (maybe) with a twist, in that I eventually started enjoying having it broken, and who doesn't! I mean… It's only a problem when you have that assignment for tomorrow afternoon and your machine just won't boot up 😢. But you can learn a lot from an experience like this, so I think it ends up being pretty interesting in the end!

After I started working as a developer, I learned that I wasn't as bad at coding as I once thought, so I started piecing a lot of these experiences together and came to the conclusion that, you know… Python development is okay and all, but after you've done plenty with a high-level language like this, at least for me, it got _pretty boring_ (to put it mildly).

Then I went on a journey looking for **something** that might give me that feeling that you're doing something _cool_ – of course I'm oversimplifying things, but this isn't too far off reality, to be honest. At first I was thinking about something related to game development, in C++ maybe, or even someplace to exercise my algorithmic skills (and please, do have a look at [project Euler](https://projecteuler.net/) if that's your thing!) but again and again I kept having that hollow-ish feeling, because those projects aren't **that** useful (practically speaking, that is), and then it all feels like a hobby, which isn't exactly the best thing when you're really busy. In short I felt like I was **wasting time**. I ended up with one conclusion though: that maybe what I wanted after all was to contribute to something **real**, and by that I mean a _real open source project_, that people use and contribute to and that can **help you** truly.

But then there's a huge question that pops up right in front of you:

## How to start contributing to open-source?

Unfortunately, I don't think I can answer that once and for all… I mean, it's not the same for everybody: you may be interested in data science or quantum computing (both of which I know nothing about!). And the processes for contributing might be radically different as well. But then I can tell you _how_ to look for something to start contributing, and then someone in that field can help you.

### Looking for a project

Just looking up projects on GitHub can get pretty boring pretty soon, so I suggest you look for an organisation that makes projects you look up to (personal examples: [iohk](https://github.com/input-output-hk), rust projects like [this one](https://github.com/gfx-rs), [llvm](https://github.com/llvm), [Khronos Group](https://github.com/KhronosGroup), etc…), and try to find someone that is accessible in there. I, for example, talked with a couple guys from Google and even a girl from Collabora but actually a really nice place to look for those people, again, is in your local community!

Try to speak with your peers or others who might know a project that needs some love, maybe even a project of their own, or they might know someone close by who might be able to help you directly (that's how I met my mentor)!

To be honest the biggest barrier in these cases (apart from taking the first step) is the language barrier. But if you read it up to here I think you are fit for the job :).

### Learning the basics

But then you might run into the same problem I did: you're not sure how you can learn all those cool things, and it seems sooo overwhelming, because usually there is no clear path to learn it all.

Well, for that you're gonna have to face the facts: People build their expertise with time, and if you keep that in mind you might be able to overcome those fears! I have a little mantra of mine as well:

> You just have to learn the basics 😌

Anything you learn at this point can be a building block for contributing. You can ask someone in your field of interest if they can help or maybe just look for some content online.

So, at this stage it's also interesting to take part in events like hackathons, as you can get an overview of the contribution process for a particular project in your field of interest.

I, for example, wasn't so thrilled about the Linux Kernel before I actually went to a hackathon focusing on it, and then I saw there's just so much to do! It gives me that "rush" of solving problems with real hardware, and dealing directly with those things I always found so cool about computers, plus there's an awesome, gigantic, open community, and I think this is super amazing if you're trying to get started :).

## On sending a "PR"

If you aren't used to Git(hub|lab)land speak you might have no idea what I'm talking about, so let me clarify a bit.

There are basically **two** main ways of contributing to projects, whether they're open-source or not:

* There are the mailing lists, which I've talked about in the [previous post](https://isinyaaa.github.io/posts/on-contributing-to-the-kernel/) (check it out if you haven't already!).
* Then there are the *PR*s (that's short for _pull request_). This is the most common way of contributing to open-source nowadays, you basically take the project you want to contribute from some point of interest and then add your code on top, on another version of the project which is basically its copy (we call that a _fork_), and when you're done you can open a PR so that contributors can review your code and possibly accept it.

So, in this post I'll talk a little more about sending PRs. This is pretty straightforward to be honest, specially when compared to the mailing lists. There are a few ways this can happen:

* Say, you might be using a program and then you notice it behaves strange for some inputs, or maybe it crashes often on your system, and you know that's open-source so you can open an _issue_ (and that's another cool thing from the Git* websites), now this might lead you up a discussion where you think you can actually solve the issue, or maybe just complement documentation, which means it's PR time!
* Another thing that might happen is you're working in a project with some friends/colleagues and they prefer that mode of contribution, as it might be preferable to having a lot of people committing changes directly to the main tree.
* Or you can even take a random issue from a project you like and give that a try! That's really cool too!

> Something that's really important, though, is to always try and be polite with the people you're dealing in these projects, as they can (and do) often get stressed out from bad interactions, or just want to be as straightforward as possible in their ways in order to be efficient – remember that this may be their job, or it may not! You don't really know what they're going through or if they even have the time to look at your code…
>
> If you want an insider look into this issue check out [Brett Cannon's post/talk on this very subject](https://snarky.ca/the-social-contract-of-open-source/).

### Contribution example

Now, in practice, this might be a little more involved, so I'll try to illustrate a personal example:

---

My mentor co-created a [really cool project](https://github.com/kworkflow/kworkflow) that helps in kernel development workflow (thus "Kworkflow", or "kw" for the intimate), with all sorts of tools. So as I'm getting started on that he suggested that I had a look at his project and tried adopting it in my daily hacking. But then I use a different shell than Linux's "usual" (I use fish btw), and kw isn't really meant to be used that way, so I ended up having some weird problems. Long story short, I opened [an issue](https://github.com/kworkflow/kworkflow/issues/473) about it, and that's **really** simple! Just click on that button in the "issues" tab and voilà!

![new_issue.png](/kernel-contrib/new_issue.png)

Then, shortly after, I tried having my hand at a first contribution to the project!

So, I pinpointed a potential source for the problem with my mentor's help – which isn't documented, unfortunately, but check out [this example](https://github.com/microsoft/vscode-python/issues/15969) or [this one](https://github.com/hrkfdn/ncspot/issues/579) to see how it might go about :). Next thing you do is fix the problem:

1. `git clone` the repo, of course
2. check for some `CONTRIBUTING.md` file, or if there are any guidelines for contribution, that might help you a lot!
3. then you might find out that's a single line fix, or maybe you need a whole new file to support whatever you need! Whatever you find appropriate :) just go on with it
4. now you should probably change your working branch to something more in line with the work you're doing:

    Maybe call it `hotfix-issue-name`, or something like that.

    Just in case you don't know, to change branches you do `git checkout [branch-name]`, if you're creating a new branch you have to use the `-b` flag as well (`git checkout -b [branch-name]`).

5. before you commit anything, try to have a look at the project's `git log` to check out for any patterns – if the project is big there's a good chance they do something, and it may be similar to the Linux kernel's way even!
6. now you can commit your changes!

    `git add [files-I-changed]`

    `git commit`

7. then you fork the project!

    ![fork.png](/kernel-contrib/fork.png)

8. now add your personal remote, as that's where you'll be pushing your commits to:

    `git remote add [remote-name] [link-of-your-fork]`

    PS: the remote name is up to you, just don't use `origin` as that's probably being used by the original remote.

9. then push changes (to your remote of course!!!!!!)

    `git push [remote-name] [branch-name]`

10. now you can open your PR

    unfortunately every Git* website will have its own flavour to this process, but it's pretty straight forward, actually :)

    also, don't PR Linux, your contribution will be politely _ignored_ if you do that!

    ![open-PR-github.png](/kernel-contrib/open-PR-github.png)


Congrats!!!!!!! You've made it :D I'm so proud 😭

Well, you didn't make it just yet, actually! Have a look at my [example's PR](https://github.com/kworkflow/kworkflow/pull/474). That's a lot of corrections they asked… So now you gotta keep up with maintainers' requests! Be sure to check out how to [rebase interactively](https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History) (also [check this out](https://thoughtbot.com/blog/git-interactive-rebase-squash-amend-rewriting-history)) and force pushes (just `git push [remote-name] [branch-name] -f`)!

As per Brett's talk you know that people might be busy (and that includes you!). So noone's expecting you to be there 24/7 replying to requests and updating your PR in real time… Just don't forget about it (and be polite!) :)

This is already pretty huge, so I'm continuing in the next post! See you there :)
