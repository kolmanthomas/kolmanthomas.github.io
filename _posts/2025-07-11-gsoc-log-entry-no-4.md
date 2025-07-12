---
title: GSoC Log Entry No. 4
subtitle: Midterm report
layout: default
date: 2025-07-11
keywords: GSoC, Chebfun, GNU Octave 
published: true 
---

## Midterm evaluation report

- [x] Make a patch to the Octave codebase.

See <a href='https://savannah.gnu.org/bugs/?67048' target='_blank'>bug #67048</a> for my first bug fixing a small issue related to graphics objects in Octave.
See also <a href='https://savannah.gnu.org/bugs/index.php?45833' target='blank'>bug #45833</a>.
I talk about this second patch more down below.

- [x] Introduce myself to the community, start a personal blog. 

See <a href='https://octave.discourse.group/t/gsoc-2025-project-discussion-porting-chebfun-to-gnu-octave/6521' target='_blank'>thread</a>.

- [x] Rebase fork of Chebfun to track latest master.

See the <a href='https://github.com/kolmanthomas/chebfun' target='_blank'>fork</a>.

- [x] Get CI/CD set up on personal GitHub fork. 

Chebfun seems to already have CI/CD set up on GitHub Actions. 
Initially I thought this was broken, but now it runs the integrated test suite after every commit, so it seems to be working well.

- [x] Write up an updated list of classdef bugs. 

See the blog post on <a href='https://kolmanthomas.github.io/blog/2025/06/10/gsoc-log-entry-no-3' target='_blank'>classdef attributes and missing features</a>.

I was going to write a part 2 that covers some of the bugs inherent in classdef.

Lastly, I would like to update the <a href='https://wiki.octave.org/Classdef' target='_blank'>Octave Wiki</a> with the information, but unfortunately, there seems to be some technical difficulties with <a href='https://octave.discourse.group/t/spam-on-the-wiki/6078/34' target='_blank'>validating email addresses</a> that prevents me from making changes.

- [ ] Compile a list of 'hacks' required to get Chebfun working.

I'll update this one shortly, before the midterm evaluation.

- [ ] Write extensive Built-In Self Tests (BISTs) for Chebfun functionality. 

My thoughts on this have evolved a bit.
Chebfun's existing test suite seems to be extensive enough. 
I think that getting maximal test coverage with the existing suite is better than expending the time and effort to write new tests, and the new tests should only target certain large changes to the Chebfun codebase that will diverge from the MATLAB version.
The reasoning is that writing new tests does not really seem to correspond to coverage in the test suite, and I do not have the time to write tests for every bit of functionality that Chebfun can do.

- [x] Fix *at least one* classdef issue in Octave proper.

This was a rather unexpected issue for me to focus on, but I managed to get the ball rolling on implementing saving and loading of classdefs!
See <a href='https://savannah.gnu.org/bugs/index.php?45833' target='blank'>bug #45833</a>, and <a href='https://octave.discourse.group/t/mat-file-read-write-compatibility-for-strings-datetime-table-etc/6550' target='_blank'>this thread</a> for discussion on the matter.

- [x] Report any bugs I encounter.

See <a href='https://savannah.gnu.org/bugs/index.php?67197' target='blank'>bug #67197</a>, <a href='https://savannah.gnu.org/bugs/index.php?67248' target='blank'>bug #67248</a>, and <a href='https://savannah.gnu.org/bugs/index.php?67258' target='_blank'>bug #67258</a>.

To be honest, there were definitely more bugs than this that I've encountered.
A large class of bugs I found were related to classdef issues; the <a href='https://kolmanthomas.github.io/blog/2025/06/10/gsoc-log-entry-no-3' target='_blank'>blog post</a> has a few examples where some of them deviate in functionality from their reported status. 
As I work my way through some of the classdef issues, I will report more of them, but I am particularly interested in the ones that are related to Chebfun.

- [ ] Write a blog post about the fundamentals of approximation theory and the purpose of Chebfun. 

I didn't really get around to doing this.
It's mostly orthogonal to the actual project, so I've put it on the back burner.

- [ ] Get 1-D Chebfun working. 

All of the changes are on <a href='https://github.com/kolmanthomas/chebfun/tree/tk_octave_dev' target='_blank'>this branch</a>, and you can see the large <a href='https://github.com/kolmanthomas/chebfun/commit/21f37940656223eea6ea4de6a727a4a32af214f9' target="_blank">commit</a> here, where most of the changes were made.

Unfortunately, 1-D Chebfun is still a ways away. 
The bugs that are plaguing Chebfun mostly have to do with object arrays and the myriad of ways that they subtly fail in Octave.
There is also a problem with the method 'domain' conflicting with the property 'domain';a large part of the commit deals with this issue by changing the property name to 'm_domain' to avoid conflicts.

All in all, Chebfun seems to have very basic functionality working: constructing a Chebfun (piecewise smooth ones work as well), doing basic arithmetic with them, calling some fairly simple methods like 'domain' and 'size'.


## Thoughts and next steps

There's definitely some wins, some things I feel good about, and some things I am disappointed by.

I am very happy overall with my understanding of Octave internals, and how classdef is implemented under the hood.
Going in, I hadn't really expected to work as deep as I did with Octave internals, but implemented the saving and loading mechanism really helped me understand almost every aspect of how classdef (and variables in general) are stored and manipulated under the hood. 
I had a pretty fantastic time working in this direction, and I'm excited to make more changes to the Octave codebase in the future.
I will also say that the community of core developers have been very kind and helpful, and seem willing to talk about my code in detail.

Becaue I've had the opportunity to work in this direction, I feel ready to tackle some more classdef issues that I think can make a large difference when porting Chebfun.
As for next, I particularly want to target fixing object arrays, since they are really one of the main barriers to getting Chebfun working in Octave.


Chebfun itself is a bit of a mixed bag.
My optimism in the beginning slowed down significantly when I realized that many of the issues were more subtle than I thought, and debugging proved to be a pain.
I'm afraid that I've spent such a long time working in Octave internals, hoping to understand classdef issues under the hood, that I've (1) neglected to really study Chebfun internals as well as I should have, and (2) to just complete the routine boring work of actually implementing workarounds.
I think the next step is just to apply tons of workarounds, and worry less about actually understanding every bug in minute detail.
My short term goal is to have 50% test coverage by July at the very least. 

All in all, I think there's a good deal of work left to do, and I'm excited to get much more productive in the second half of GSoC.
Once again, I must thank my mentors Colin and Andreas for their patience and help so far, I'm thankful for them placing their trust in me, and I am looking forward to continuing to work on this project with them.
I really am appreciating my time in GSoC, I think this has been a fantastic guided introduction to open source contributions, and everyone has been delightful.
