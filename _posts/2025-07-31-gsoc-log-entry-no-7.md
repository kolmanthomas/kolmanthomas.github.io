---
title: GSoC Log Entry No. 6
subtitle: Chebfun Status Report No. 2
layout: default
date: 2025-07-31
keywords: GSoC, Chebfun, GNU Octave 
published: true 
---

## Chebfun Test Suite

I've previously talked a bit about how Chebfun has a built-in test suite.
The way these tests work is a bit different from Octave's BIST functionality.
They don't use MATLAB's test framework either, it's a homegrown solution, so I'll go through and describe it to the best of my understanding.

The top-level function `chebtest` is the driver for every test.
Running `chebtest` and seeing the output for `testDirNames1` shows you all the directories that contain tests.
```
testDirNames =
{
  [1,1] = adchebfun
  [1,2] = ballfun
  [1,3] = ballfunv
  [1,4] = bndfun
  [1,5] = cheb
  [1,6] = chebfun
  [1,7] = chebfun2
  [1,8] = chebfun2v
  [1,9] = chebfun3
  [1,10] = chebfun3t
  [1,11] = chebfun3v
  [1,12] = chebgui
  [1,13] = chebmatrix
  [1,14] = chebop
  [1,15] = chebop2
  [1,16] = chebpref
  [1,17] = chebtech
  [1,18] = chebtech1
  [1,19] = chebtech2
  [1,20] = classicfun
  [1,21] = deltafun
  [1,22] = diskfun
  [1,23] = diskfunv
  [1,24] = domain
  [1,25] = fun
  [1,26] = functionalBlock
  [1,27] = linop
  [1,28] = misc
  [1,29] = operatorBlock
  [1,30] = singfun
  [1,31] = spherefun
  [1,32] = spherefunv
  [1,33] = spinop
  [1,34] = spinop2
  [1,35] = spinop3
  [1,36] = spinopsphere
  [1,37] = spinpref
  [1,38] = spinpref2
  [1,39] = spinpref3
  [1,40] = spinprefsphere
  [1,41] = spinscheme
  [1,42] = treeVar
  [1,43] = trigspec
  [1,44] = trigtech
  [1,45] = unbndfun
}
```

Each class has it's own set of test functions -- the `chebfun` class has 166 test functions alone, for example -- usually structured to be a minimum of 1 test function per method, but possibly more if the method is complex (`chebfun` construction has 9, for instance). 
Each test function returns a matrix of logical values indicating whether the individual tests passed or failed.
If at least one of the matrix values is false, then the test suite fails.
If the function fails due to an interpreter error, it will report the test as crashed, but continue on to the next test.

To run the tests from only certain directories, you can pass the directory names as arguments to `chebtest`: so `chebtest('chebfun')` will run all the tests in the `chebfun` directory.
For now, we're interested in getting the `chebfun` tests working first, and potentially the others down the line.

## Interpreter panics

Some of the tests in Chebfun cause the interpreter to crash.
Examples are `tests/chebfun/test_constructor_inputs_periodic.m` and `tests/chebfun/test_diag`.
Isolating the cause of these crashes is very tricky, since I am not having much luck with recreating the crashes in a simpler example.
I would like to report these as issues to the bug tracker once I can figure out a MWE to reproduce the crash.


