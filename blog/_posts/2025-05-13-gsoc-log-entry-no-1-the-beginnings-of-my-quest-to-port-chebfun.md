---
title: GSoC Log Entry No 1&#58 The Beginnings of My Quest to Port Chebfun 
subtitle: My project with GNU Octave.
layout: default
date: 2025-05-13
keywords: GSoC, Chebfun, GNU Octave 
published: true
---

I am overjoyed to announce that I have been selected for Google Summer of Code's 2025 program with GNU Octave.
I am being mentored by Colin Macdonald (maintainer of the Symbolic package) and Andreas Bertsatos (maintainer of the Statistics package).

You can read the offical project announcement <a href='https://summerofcode.withgoogle.com/programs/2025/projects/6xbfUWnM' target='_blank'>here</a>, but I have pasted the project summary below, since I am fond of it:

> I propose to work on the following project: porting the Chebfun package to GNU Octave. Chebfun is a numerical computation package for MATLAB that provides a framework for representing functions in a symbolic manner using ”Chebyshev technology”, a term coined by the package authors to describe the use of Chebyshev polynomials to represent functions up to machine precision. GNU Octave is supposed to be mostly compatible with MATLAB, so Chebfun should ideally work without change on the former, but there are some slight differences and unimplemented features in Octave that make this difficult. Primarily, the main issue is that Octave’s support for object-oriented programming (classdef) is not as feature complete and as stable as MATLAB’s, which is a major problem since Chebfun is written in classdef. The goal of this project is to port Chebfun to GNU Octave, and to make it work as a dropin replacement for the MATLAB version. Since fixing classdef is not a feasible/recommended goal for a GSoC project, I will instead focus primarily on making the necessary changes to the Chebfun codebase to make it work in Octave. Along the way, I will compile new information about the status of classdef in Octave, and I will share my findings with the Octave developers, and perhaps aid in some changes to the Octave codebase itself.

In a conversation with my mentors, it was discussed that I could play a more active role in actually fixing classdef-related issues in Octave itself.
This will involve getting familiar with the Octave codebase, which is written in C++, and making changes to the compiler to patch Octave's classdef implementation.

I will be posting regularly on this blog about my thoughts and progress on either Chebfun, GNU Octave internals, or the theory behind Chebyshev polynomials.
The first post that is likely to follow will be a preliminary compilation of (up-to-date) classdef-related issues in GNU Octave.

I sincerely wish that these diary entries will be useful to future GSoC students, people interested in Chebfun, compiler design, numerical analysis or really or any reason. 

