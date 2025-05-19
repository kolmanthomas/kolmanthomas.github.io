---
title: GSoC Log Entry No. 2
subtitle: The idealized checklist for the midterm evaluation. 
layout: default
date: 2025-05-18
keywords: GSoC, Chebfun, GNU Octave 
published: true
---

Week 1 has wrapped up, and I've done enough poking around to get a good sense of the project scope. 
Most of my activity was spent getting familiar with the Chebfun codebase, setting up the blog, and continuing a bit of the mathematical self-study.
My midterm evaluation start date is on July 7th, so I've set out some reasonable goals to achieve by then.
Here they are:

## Checklist for the midterm evaluation

**Note: Status of the checkboxes is frozen after May 18, 2025; I will create a new post, after the midterm evaluation, detailing what was accomplished from this list.**

- [x] Make a patch to the Octave codebase.

See <a href='https://savannah.gnu.org/bugs/?67048' target='_blank'>bug #67048</a>.

- [x] Introduce myself to the community, start a personal blog. 

See <a href='https://octave.discourse.group/t/gsoc-2025-project-discussion-porting-chebfun-to-gnu-octave/6521' target='_blank'>thread</a>.

- [x] Rebase fork of Chebfun to track latest master.

See the <a href='https://github.com/kolmanthomas/chebfun' target='_blank'>fork</a>.

- [ ] Get CI/CD set up on personal GitHub fork. 

This is something I'm immediately working on.
I want to make it as frictionless as possible for multiple people to contribute to the port in the future, and having proper CI/CD set up is a good first step.

- [ ] Write up an updated list of classdef bugs. 

There is an existing list of classdef related issues over at the <a href='https://wiki.octave.org/Classdef' target='_blank'>Octave Wiki</a>.
One of my immediate goals will be to update this list to the best of my knowledge, so I will try to make edits to the Wiki itself if need be.
I will also write a blog post, as mentioned earlier, outlining the various ways in which classdef is buggy or lacking features.
The blog post will contain some information about GNU Octave internals, so if you're interested in compiler practice (instead of theory), it might be worth a read.

- [ ] Compile a list of 'hacks' required to get Chebfun working.

The Git history of the port will be definite enough to keep track of all the workarounds applied, but I want to keep a master list of such in octave_notes.txt (found in the repo) so that future contributors can quickly get up to speed.

- [ ] Write extensive Built-In Self Tests (BISTs) for Chebfun functionality. 

Chebfun already has its own suite of tests that ensure package correctness.
The published Octave BISTs will focus more on code changes to the Chebfun fork, in order to see if the changes modified any of the expected behaviour of Chebfun.

- [ ] Fix *at least one* classdef issue in Octave proper.

Ideally, the classdef bug to be patched will be one that plagues the Chebfun codebase somehow.
I have not decided on which bug I will dedicate my time to, or how many bugs I could realistically fix; that is something I will learn with experience.
The dream would be if Chebfun would immediately start working after a few core Octave patches applied, but since this is very unclear if it is even feasible, I cannot rely on this being the path forward for the port.

I've already submitted a patch to Octave proper using Mercurial and GNU Savannah, so I'm familiar with the workflow.

- [ ] Report any bugs I encounter.

Running Chebfun on Octave has already produced some strange crashes (e.g. illegal hardware instruction exceptions) that shouldn't be happening to any interpreted code.
Also, because I will be writing extensive tests for Chebfun, I might be able to catch a bug in the package itself.
In either case, it will be my duty to isolate the problem as best as I can, and report it to either the Octave or Chebfun teams.

- [ ] Write a blog post about the fundamentals of approximation theory and the purpose of Chebfun. 

This is not strictly necessary for the port, but I believe that Chebfun's power is not necessarily immediately obvious to most users.
To unfamiliar eyes, it gives off the impression of 'niche' academic software, which is not helped by the fact that it only runs on MATLAB.
As (hopefully) a future package maintainer, it will also be imperative to generate some interest in the package, and writing an approachable blog post will be a step towards that.

- [ ] Get 1-D Chebfun working. 

This is the ultimate goal to have before the midterm evaluation. 
The working definition of 'working' is that all the tests in tests/chebfun/ (i.e. the pre-existing tests for 1-D Chebfun) pass, and also that the custom BISTs pass as well.
My naive hope is that getting this finished will be the hardest part of porting Chebfun in general, since I will have learned about most of the issues with porting classdef code to Octave.


