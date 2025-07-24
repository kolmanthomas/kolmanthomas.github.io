---
title: GSoC Log Entry No. 5
subtitle: Chebfun Status Report No. 1
layout: default
date: 2025-07-22
keywords: GSoC, Chebfun, GNU Octave 
published: true 
---

Here's my first Chebfun report on the port.
Currently, there are two main issues that pose a problem for the port of 1-D Chebfun functionality from MATLAB to Octave.

## Subsref behaviour inside versus outside of a class

The first is that `domain` is a property in the `chebfun` classdef *and* a method in the class folder `@chebfun`. 
A simplified version of the `chebfun` classdef structure looks like this:

```
- @chebfun
    - chebfun.m
    - domain.m
    - subsref.m
    - subsasgn.m
    ...
```
chebfun.m looks roughly like
```
classdef chebfun
    properties
        domain        % (1x(K+1) double) 
        funs          % (Kx1 cell array of FUN objects)
        pointValues   % (1x(K+1) double) 
    end
end
```
where $K$ is some natural number greater than or equal to 1.

domain.m looks roughly like
```
function [A, B] = domain(f, flag)
...
end
```
A little detour: MATLAB does *not* allow a classdef method to have the same name as a property; Octave does. 
Probably a compatibility bug.
However, both allow a method *in the class folder* and a property with the same name.

Outside of the class, an invocation of `f.domain` resolves to function calls by default (no custom subsref) in both MATLAB and Octave. 
Chebfun has an overloaded `subsref` method that is called, which is not a problem, this was intended by the Chebfun developers.

*Inside* the class, however, two things are different.
The first is that the custom class subsref is *not* called by dot, brace or parenthesis indexing.
This is intentional behaviour in order to avoid recursion issues.
This is also true whether the subsref is defined in the classdef or in a separate file in the class folder.
If you want to force the custom subsref or subsasgn behaviour, you have to explicity call `subsref(., .)` or `subsasgn(., ., .)`.
The second is that default subsref behaviour in MATLAB is to resolve to the property value, *not* a method call, which is the opposite of what you'd expect outside of the class.
This behaviour is different in Octave, where the default subsref behaviour is to resolve to the method call both inside and outside the class.
This essentially means that in Octave, you cannot really reach the property value of `domain` by any means.

## Object arrays 

The second is that object arrays are quite broken in Octave.
Specifically, what is causing huge issues is 1x1 object arrays; in MATLAB, they are treated like singular objects, and you are able to reference their properties and subscript further into those properties and so forth.

Let's demonstrate with an example.
Suppose the following commands are run *inside* of a class method (not outside, in order to force default subsref and subsasgn behaviour):
```
--- MATLAB ---
>> f = chebfun('x');
>> h(1) = f;
>> h.pointValues

ans =

    -1
     1

>> h.pointValues(2)

ans =

     1

```
So, all is fine and dandy in MATLAB.
If `h` was an array of greater size than 1x1, then line 4 would not work, because you would be trying to subscript several properties at once (where some of them may be potentially different types and sizes!), but it does work for 1x1 arrays.

In Octave, however, the second line `h(1) = f` causes some quiet issues.
The type of `f`, which is an object of type `chebfun`, is assigned to be the first index position of `h`, which has a type of object array of `chebfun`.
What this means is that while `h` has the same size as `f`, you cannot subscript into property values of `h` at all, so line 4 produces the following error

```
--- Octave ---
octave> h.pointValues(2)
error: can't perform indexing operation on array of chebfun objects
``` 

This is a large problem in Chebfun, because almost every method treats the incoming `chebfun` object as potentially an array of chebfuns, and so unknowingly converts a standard `chebfun` object into a 1x1 array.
The fix for this is relatively simple, if you knew it was a 1x1 array, you would write
```
--- Octave ---
octave> h(1).pointValues(2)
ans = 1
```
and this is of course compatible with MATLAB. 
But this is a very annoying issue that touches pretty much every corner of the Chebfun codebase, so fixing this is very tedious and causes very difficult to track errors in other places in the code.





