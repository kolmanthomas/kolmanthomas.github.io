---
title: GSoC Log Entry No. 7
subtitle: Summary of Progress
layout: default
date: 2025-08-31
keywords: GSoC, Chebfun, GNU Octave 
published: true 
---

This is the final report for my Google Summer of Code 2025 project: [Porting Chebfun to GNU Octave](https://summerofcode.withgoogle.com/programs/2025/projects/6xbfUWnM).

## Introduction

Google Summer of Code is a program, run by Google, that organizes and funds projects related to open-source software.
This year, I participated in the program as a mentee under the GNU Octave organization.
Under the supervision of my mentors Colin B. Macdonald (Associate Professor, Department of Mathematics, University of British Columbia) and Andreas Bertsatos (Researcher, Science and Technology in Archaeological and Cultural Research Center, The Cyprus Institute), I worked on porting Chebfun, a numerical/symbolic hybrid computing library, from MATLAB to Octave.

## What I've set out to do

[My first blog post also talks about the goals of this project](https://kolmanthomas.github.io/blog/2025/05/13/gsoc-log-entry-no-1-the-beginnings-of-my-quest-to-port-chebfun/)

I realize, though, that my proposal has some jargon that might be hard to decode if you're not familiar with either of MATLAB, Chebfun or GNU Octave before, so here's a quick glossary of terms before I try to describe it again:

[**MATLAB**](https://www.mathworks.com/) -- A numerics-focused computing environment and language, mostly used in academia and in traditional engineering domains (read: not pure software engineering). Not [libre software](https://en.wikipedia.org/wiki/Free_software), licenses cost a lot of money and are quite restrictive in terms of deploying code.

[**GNU Octave**](https://octave.org/) -- A libre alternative to MATLAB licensed under [GPLv3](https://www.gnu.org/licenses/gpl-3.0.en.html), mostly compatible with MATLAB code. What this means is that most MATLAB code should be able to be run in Octave. Unfortunately, this is not the case for all MATLAB code.

[**Chebfun**](www.chebfun.org) -- A (very large) MATLAB library that implements a completely novel way of doing numerical computations with functions. It does this by representing functions as piecewise Chebyshev polynomial interpolants, and it overloads many MATLAB operators to work on these function-like objects. Initially created by researchers at the University of Oxford, it has been under active development for over 20 years.

[**Classdef**](https://www.mathworks.com/help/matlab/ref/classdef.html) -- MATLAB's object-oriented programming system, introduced in 2008. GNU Octave supports a fairly basic subset of it, but feature parity nor stability was ever achieved.

The scope of the original project was to port Chebfun from MATLAB to GNU Octave.
This project required two different strategies to be employed:

* Fixing bugs and implementing missing features in GNU Octave, mostly to due with the classdef system.
* Fixing code in Chebfun proper, applying workarounds whenever possible, and getting it ready to be shipped as an Octave package.

## Why is it important to port Chebfun?

It's a justified question.
To start, Chebfun is a permissively licensed ([BSD license](https://en.wikipedia.org/wiki/BSD_licenses)) library that was developed by academics to experiment with new numerical algorithms and a novel interface for doing mathematical analysis with computers.
Chebfun is a [wellspring of scientific research](https://www.chebfun.org/publications/) with a [dedicated book to go along with it](https://www.chebfun.org/ATAP/).
My personal motivation is that it is a shame to see such cool software like that require a proprietary platform to run.

On a more practical note, it is hard to invigorate interest in a software project that exists on a platform that less and less new mathematicians/computer scientists/engineers are learning.
Even though Octave has extremely similar syntax to MATLAB, the odds of something picking it up on a whim to try Chebfun are much higher than if they had to invest in a restrictive MATLAB license.

### Could you have not ported it to another language, like Python or even a compiled language like C/C++, and had those same benefits?

Short answer is it's possible, but:

* Chebfun's interface is pretty much designed on the principle of overloading MATLAB operations on discrete objects (matrices, vectors) to work on continuous objects (functions). The API would have to be redesigned for the new language.
* In a similar vein, the MATLAB/Octave numerics-focused syntax is quite pleasant for experimenting with this sort of thing.
* It would have been a very large project (a solid year or two of full-time work).
* It would be divorced from upstream improvements in the sense that I would have to manually add those same improvements to the port every time something was added/changed.
* The port had the dual benefit of improving GNU Octave internals itself, particularly the classdef-related bits.
* It's already been sort of done by [ApproxFun](https://juliaapproximation.github.io/ApproxFun.jl/stable/), which is a Julia library that implements some of the same functionality, but with some differences.


## What I've got done

### Loading and saving of classdef objects

I didn't really expect to work on this issue at all when I first started the project.
It started when I wrote my [midterm checklist](https://kolmanthomas.github.io/blog/2025/05/18/gsoc-log-entry-no-2/), which had the line

> - [ ] Fix *at least one* classdef issue in Octave proper.
>
> Ideally, the classdef bug to be patched will be one that plagues the Chebfun codebase somehow.
> I have not decided on which bug I will dedicate my time to, or how many bugs I could realistically fix; that is something I will learn with experience.
> The dream would be if Chebfun would immediately start working after a few core Octave patches applied, but since this is very unclear if it is even feasible, I cannot rely on this being the path forward for the port.

In one of our weekly video meetings, Andreas mentioned that loading and saving of classdef objects as a possible issue to work on, mostly because it was a missing classdef feature that couldn't be worked around easily.

<details>
<summary>
Explanation of saving and loading in Octave
</summary>
<br>
<style>
  body {
    margin-left: 50px; /* Adjust the value as needed */
  }
</style>
<div markdown="1" >
> To give you a bit of background before I explain, if you create some workspace variables, etc.
> you can save these variables by calling something along the lines of `save -FORMAT FILENAME`.
>
> The options for FORMAT are:
* Native Octave formats, either text-based or binary
* MAT files, either corresponding to v5 or v7 MAT files
>
> However, before the summer started, classdef objects could not be loaded and saved.
</div>
</details>

Initially, I admit I was a bit skeptical to work on this issue, because Chebfun doesn't really intimately rely on saving and loading of objects.
It would be nice to have, but it wasn't a reason for Chebfun to not work.
However, while working on the loading and saving of classdef objects, it was actually the perfect bug for me to work on, because I got up to speed very quickly with (1) all classdef features, and (2) Octave internals, because saving of loading of classdefs touches all the classdef internals very deeply.
So in the months of June and part of July, I spent a long time poking around at Octave load/save internals.

The result was introductory support of loading and saving of classdef objects. 
Some of the kinks still need to be ironed out.
You can see the discussion on [bug #45833](https://savannah.gnu.org/bugs/index.php?45833) about how complex getting full saving/loading support is.
A lot of the complexity comes from MATLAB's poorly documented descriptions of how saving and loading works at a fine-grained level.
Another source of complexity was that the native Octave file formats were designed to save simpler types like numeric arrays and structs, not complex objects.
So lots of reverse-engineering and testing edge cases was needed.


### Getting Chebfun working

Of course, the main thrust of the project.
We've been measuring progress of Chebfun by counting the number of tests passed.

Basically, in the `tests` folder you have the following subfolders:
```
adchebfun       chebfun3t       chebtech2       misc            spinpref
ballfun         chebfun3v       classicfun      operatorBlock   spinpref2
ballfunv        chebgui         deltafun        singfun         spinpref3
bndfun          chebmatrix      diskfun         spherefun       spinprefsphere
cheb            chebop          diskfunv        spherefunv      spinscheme
chebfun         chebop2         domain          spinop          treeVar
chebfun2        chebpref        fun             spinop2         trigspec
chebfun2v       chebtech        functionalBlock spinop3         trigtech
chebfun3        chebtech1       linop           spinopsphere    unbndfun
```

Essentially, we've starting from basically 0 tests passing to:

```
Total number of tests:
1104
Number of successful tests:
576
Number of failed tests:
50
Number of crashed tests:
478
```

A failed test is one that executes without error, but produces an incorrect result.
A crashed test is one that produces an interpreter error.

Qualitatively, 1D Chebfun is pretty good. There's some annoying bugs, but generally it works well in most cases and fails fast if it doesn't. You can run most of the [Chebfun Guide](https://www.chebfun.org/docs/guide/) without encountering errors, which feels pretty good.

Unfortunately, differential equation solvers are a bit of a ways away. Hopefully they will be working soon, but there's some underlying Octave interpreter bugs that are causing all the tests to fail.

Overall, is this a success? I would say so! My opinion is that getting Chebfun working without an immediate crash was a large hurdle, and now that it's semi-fixed, a lot of the remaining bugs will be easier to work around. There's quite a lot of remaining crashed tests, but many of them have the same underlying cause, and generally only one individual test out of the usual 4 - 20 tests in a file will crash.

## Patches for GNU Octave

[Bug #67048: findobj with 'tag'=[]](https://savannah.gnu.org/bugs/?67048). This was the initial patch I submitted before I was even accepted into GSoC. It fixs a very small issue with the `findobj` function. It was the first patch I submitted to Octave, before I was formally accepted into GSoC.

[Bug #45833: support load/save of classdef objects](https://savannah.gnu.org/bugs/index.php?45833). I've already discussed at length in an earlier section. This discussion (along with [the discussion on Discourse about loading and saving](https://octave.discourse.group/t/mat-file-read-write-compatibility-for-strings-datetime-table-etc/6550/44)) also contains the genesis of MAT file support for loading and saving classdef objects.

[Bug #67311: Very bad slowdown of load()](https://savannah.gnu.org/bugs/?67311): 
Does it really count if I was the reason for this bug? 
I introduced it while working on load/save of classdef objects. 
In a nutshell, the problem was that every time a variable was being loaded from a save file to the workspace, Octave would check the type of the variable. 
In order to support classdef objects, I added a call to `which` to determine if the variable type was a classdef constructor. 
The offending line was
```
octave::__get_help_system__ ().which (typ, class_type);
```
*which* (ha!) is a function that searches the Octave path with the given name.
This was a terrible idea, because `which` is a heavyweight user-facing function that allows you to search for anything in the Octave path. 
What would happen is, you'd have a save file with about ~100 or so variables to load that would take around 30 seconds to load. 
That's a catastrophic performance hit, since it took less than 0.1 seconds to load beforehand.

The fix was quite simple, actually, I had to use the internal `cdef_manager` utility to find if the variable was a classdef, rather than using `which`.
It was a bug that was fixed within three hours of being reported, so a crisis was successfully averted!

[GNU Octave - Bugs: bug #67362, Subsref should resolve to property... [Savannah]](https://savannah.gnu.org/bugs/index.php?67362). I talk about this bug in [one of my Chebfun status reports](https://kolmanthomas.github.io/blog/2025/07/22/gsoc-log-entry-no-5/). I pushed a fix myself, because the prospect of manually renaming a variable in hundreds of places sounded horrendous.

[GNU Octave - Patches: patch #10537, Support for InferiorClasses... [Savannah]](https://savannah.gnu.org/patch/index.php?10537). Final patch I submitted before the end of the GSoC period. You can read more about it in [my Chebfun status report](https://kolmanthomas.github.io/blog/2025/08/07/gsoc-log-entry-no-8/).


## Chebfun Patches

If you want to check the latest progress on Chebfun, [check out this repository](https://github.com/cbm755/chebfun). 

The following is a list of commits that were applied by myself to Chebfun during the GSoC period.
The nature of this work is that many commits were either rolled back once they were fixed in core Octave, or they were superceded by better workarounds.
Colin also commited many changes during this period: [you can look at all the commits during the summer of 2025 on this branch](https://github.com/kolmanthomas/chebfun/tree/gsoc_2025).


```git
* c9e452aea (HEAD -> gsoc_2025, origin/gsoc_2025, octave_dev) Rewrote README and added gsoc.md for final status report of GSoC 2025
* 4faac7d31 (origin/edit_octave_notes, edit_octave_notes) Updated octave_notes.md
* 5797dad8e Changed octave_notes to use Markdown
* 6f1cdac70 Revert "octave: TEMPORARY: remove the domain class and replace with domain.m"
* 04dbfed5c Turned vectorize off by default in chebop.
* dc33cc34b Disable tests that crash that Octave interpreter.
* d83eda024 Renamed filename resetPointValues to clearPointValues in order to match function name.
* 073c5da2f Changes to seedRNG.m by changing 'verLessThan' calls to 'compatible_verLessThan'.
* 2be39a0b8 Changes to plot.m by changing 'verLessThan' calls to 'compatible_verLessThan'.
* f663c4f23 Replaced 'fields' method with the 'fieldnames' method.
* fde15d655 Declare fun class as Abstract.
* 0988a0c7b Workaround for isequal precendence call.
* f1a6b203e Revert "octave: use static methods instead of class-related functions"
* 9ecedcaca Ignore this test for now, it crashes the interpreter.
* bea491edb Ignore these tests for now.
* 8a8531469 Changed verLessThan to compatible_verLessThan
* c6b1b0fc5 Added a new function to handle 'verLessThan('matlab', version)' calls.
* e3dc50141 Fix unintended behaviour making f a complex-valued chebfun.
* ca29a7908 Revert "octave: temporary workaround for f.funs{j}"
* 70b9c3fbe Revert "octave: TEMPORARY dumb hack: always return [-1, 1] for domain"
* cf7052d4f Added comma delimiter for cell array of anon functions.
* 3ffe9891b Revert "octave: workaround subsref and property fighting over "domain""
* 0d2c86de2 Revert "Workaround for subsref issue with domain"
* b6a9da520 Revert "Temp fix for 2*chebfun() to work"
* 52f428b50 Add method to check if Octave is being used
* 3e6c8d3bd Temp fix for 2*chebfun() to work
* 7b1460582 Workaround for subsref issue with domain
* 839da2ed2 Update octave_notes.txt
* cb59de187 resetPointValue: rename function to match filename
```

## Bug Reports for GNU Octave

These are bugs I've reported that I didn't submit a patch for. Some of them were fixed by others, some are still open at the time of writing.

[Bug #67197: Class with abstract properties can be instantiated, but should not be able to be](https://savannah.gnu.org/bugs/?67197).
An incompatibility between MATLAB and Octave regarding abstract classes.
In short, MATLAB behaviour is the following: if even a single method or a property is labelled abstract, then the entire class is abstract.
Octave doesn't have the same semantics, which is a bug.
Easy workaround though, you can just manually label the entire class as abstract.

[Bug #67248: AddressSanitizer catches 'container-overflow' in svd.cc when running tests](https://savannah.gnu.org/bugs/?67248). 
To be honest, I had heard about C++ code sanitizers before this summer, but I never bothered to use one. 
What a huge mistake! 
Sanitizers are fantastic tools, they catch a legitimately huge class of memory errors. 
If you compile with Clang, it's dead simple to use, just add `-fsanitize=address` to your compiler flags.
This is just one of them that I encountered before someone else did, but using a sanitizer saved me an incredible amount of time. 
I *highly* recommend using sanitizers for pretty much every debug build you'll ever do in a C/C++ codebase. 

[Bug #67258: Bad memory access when trying to integrate 1/sqrt(x) in 'quadv.m'](https://savannah.gnu.org/bugs/?67258). The one bug that got away. It was automatically fixed when I updated the project dependencies, and now I'll never know what caused it. Oh well.

[Bug #67361: Classdef property and method should not have the same name](https://savannah.gnu.org/bugs/?67361). An incompatibility with MATLAB, and a source of many headaches in the Octave internals.

[Bug #67348: Cell array construction of anonymous functions fails without comma delimiting](https://savannah.gnu.org/bugs/?67348). Probably the tiniest bug I could have reported. Essentially, MATLAB allows you to initialize a vector by writing 

```
[ 1 2 3 4 ]
```
instead of putting a delimiter of some kind between the values, like a comma, which is used in other languages. 
It's supposed to mimic writing a vector on a blackboard.
But this feature got extended in two different ways.
First, to cell arrays, which is a container that lets you mix different datatypes together, like so
```
{ 1 2.2 3 4 + 2i }
```
which looks horrible on the eyes, but it's valid MATLAB/Octave.
The second way this feature was extended was by means of allowing this lack of delimiters between non-numeric datatypes. 
So in MATLAB, you could write
```
{ @(x) cos(x) @(x) sin(x) }
```
(In MATLAB, @(x) ... is the syntax for an anonymous/lambda function).
Octave mostly supports this feature, but fails in the above anonymous function case.
The workaround is hilariously simple: just put a comma!
Still, it's always a good thing to document the discrepancies between MATLAB and Octave.

[Bug #67412: verLessThan('matlab', ...) should emit a warning rather than throw an error](https://savannah.gnu.org/bugs/index.php?67412). A proposal that fell flat on it's face. Let me give some context to explain the rational behind this proposal.
In MATLAB code, you often see version checks like this
```
verLessThan('matlab', VER)
```
(As an aside, after R2023 it's [no longer recommended](https://www.mathworks.com/help/matlab/ref/verlessthan.html) to check the MATLAB version this way). 
The pseudocode for the function behaviour looks something like this:
```
if toolbox exists
    if version_of_toolbox < VER
        return true
    else
        return false
else
    throw error
```
Straightforward, if you accept that `MATLAB` counts as an acceptible input for toolbox.
Well, Octave doesn't, because of course not, it cannot acknowledge the existence of MATLAB.
I'm joking a bit.
But, strictly speaking, if you accept that MATLAB is an acceptible first argument, you have to assign it a version. 
My thought is that MATLAB's version on Octave is 0.0.0, because Octave is supposed to be feature compatible with the latest MATLAB, and every time it is not, it's an incompatibility bug.
Either Octave supports the feature that's being distinguished, or it doesn't, but checking *on Octave* whether MATLAB is less than some version is devoid of meaning.
The real motivation for this proposal is that porting MATLAB code would be ever so slightly easier if Octave didn't throw an error on this function call.

[Bug #67413: Stacktrace from error should show class name along with function name](https://savannah.gnu.org/bugs/index.php?67413). Essentially, a case that errors should contain more information than simply the name of the function where the error occured. Still a work in progress at the time of writing, but there's some progress being made.

## What I learned

Quite a bit.

On the programming side, I learned how to read (two!) giant codebases, one in C++ and one in MATLAB.

When I was contributing to Octave, I learned the following 
* How to use Mercurial (version control system), how to structure my patches for submission, how to follow coding style guideline.
* How to play around with compilation options and flags, how to get the fastest possible build times, how to debug build errors and missing/invalid dependencies.
* How to use LLDB to make sense of otherwise-opaque crashes, how to decipher complicated C++ compilation errors.
* How to navigate a codebase that heavily uses object-oriented design and inheritance.
* How to use every tool at my disposal to read code, including Doxygen, grep, clangd, memory sanitizers, and of course, good old `std::cout` statements.

When I was contributing to Chebfun, I learned the following:
* How to really use Git, and by that I mean how to leverage every feature of Git in order to get a sane commit history.
* Almost every nook and cranny of MATLAB's classdef system (not a joke).
* How to read numerical analysis-heavy code, and how to piece together the mathematical ideas behind the code. 
* Heavily related to the previous point, how to read academic papers implementing algorithms.
* How to struggle through hundreds of lines of code checking input arguments (no static typing in MATLAB/Octave!).
* How to document language discrepancies between MATLAB and Octave, how to create minimum working examples to demonstrate bugs, and how to brainstorm workarounds in a collaborative capacity.


On the more human side, I learned how to work effectively with others.
I learned how to present my ideas and findings clearly, how to debate for my positions while being collaborative and open to feedback, and how to consolidate my knowledge into something sharable.
Lastly, I learned how to jump into a new project and get up to speed quickly.
I fully believe I can launch into new projects with a high degree of autonomy and confidence at this point.

## Next Steps

I will co-maintain the Octave port of Chebfun with Colin.
We will continue to develop the capabilities of Chebfun further, and I'll continuously update the blog on the progress.
Another possible avenue of approach may be to try to upstream our changes to the original Chebfun project; however, we have not decided if this is feasible yet.

Getting CI/CD working for Chebfun is a high priority task.
Right now, most of the testing is done manually, and producing tarballs for releases is also done manually.
This means setting up a Docker container with the latest Octave build, which does not exist as of now.

In core Octave, the rudimentary capabilities and the myriad of bugs regarding classdef arrays are a big issue; see [this issue for a particularly bad case](https://github.com/cbm755/chebfun/issues/13). 
This is tough becuase these sorts of issues cannot be worked around in Octave code.
Fixing classdef arrays is a difficult undertaking, but one I hope to contribute to.


### Loading and saving

Even after all the work put into loading and saving, it is unclear if the native Octave file formats should continue to be used. 
The three options going forward are:

* Rely completely on the MAT file format for loading and saving, which is undocumented, subject to continual revisions, and parsers for it are not widely distributed. 
The pros and cons of using MAT files as a suitable archival format is even [discussed by the US government](https://www.loc.gov/preservation/digital/formats/fdd/fdd000440.shtml).
* Remain with using the native Octave text- and binary-based formats. 
They would need to be extended in some ways to support classdef objects, but it's not completely impractical.
* Devise a new file format for loading and saving variables in general that is also suitable for classdef objects.

## Acknowledgements

I would like to thank my mentors, Colin B. Macdonald and Andreas Bertsatos, for their patient guidance and support throughout this project.
They were kind enough to hear out my ideas, let me work in the direction that I thought was best, and they were always available to answer my questions.
Their enthusiasm to see this project succeed was infectious, and I am very honored they entrusted me with this project.
I learned a lot from both of them, and I am very grateful for their mentorship.

I would also like the thank the GNU Octave maintainers for their support, their willingness to hear out my ideas and review my patches, and of course for administering GSoC. 
In particular, I would like to thank John W. Eaton and Markus Mützel for hosting office hours and answering my questions, but there were many others who helped me along the way as well.

I would also like to thank Google for running the GSoC program and making this project possible.
