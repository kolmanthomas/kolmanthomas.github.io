---
title: GSoC Log Entry No. 6
subtitle: Chebfun Status Report No. 3
layout: default
date: 2025-08-07
keywords: GSoC, Chebfun, GNU Octave 
published: true 
---

Chebfun is definitely making some progress! 
Many of the tests are now passing -- over 50% of them in the Chebfun folder, and over 90% in some of the other folders.
I'm at the point where I can run many of the examples from the <a href="www.chebfun.org/docs/guide/" target="_blank">Chebfun Guide</a> and from the book *Approximation Theory and Approximation Practice*. 

The pencils-down date of the GSoC period is August 25th.
I do plan on continuing to work on the port (for the love of the game) and maintaining it as an Octave package, but I do wish to have a final product of some sort at the finish line, so hopefully 1D Chebfun exists by then!


Here are two separate issues that I found were notable enough to pop up a few times in the codebase.

## Inferior classes

If you want an official understanding of what this attribute is supposed to accomplish, you can see the <a href="https://www.mathworks.com/help/matlab/matlab_oop/class-precedence.html" target="_blank">documentation on class precedence</a>.

Octave does not support this attribute yet.
Here's my take on explaining why this attribute is useful and how it works: Define an ordinary class `Bar` like so

```
classdef Bar
    properties
        val = 5
    end

    methods

        function c = minus(a, b)
            disp("Bar.minus called!")
            c = a.val - b.val;
        end

    end
end
```
and a class `Baz` that declares `Bar` inferior to it

```
classdef (InferiorClasses = {?Bar}) Baz
    properties
        val = 3
    end

    methods

        function c = minus(a, b)
            disp("Baz.minus called!")
            c = b.val - a.val;
        end

    end
end
```

Well, when you use 
```
octave> bar = Bar;
octave> baz = Baz;
octave> minus(baz, bar)
Baz.minus called!
ans = 2
```
naturally, this last line will call `Baz.minus`, because `Baz` is the first argument. 
It's essentially the same as writing `baz.minus(bar)`.
With the `InferiorClasses` attribute, you should expect
```
octave> minus(bar, baz)
Baz.minus called!
ans = -2
```

With the `InferiorClasses` attribute, you should expect `Baz.minus called!` and `ans = -2`, but since this attribute is not supported, you get
```
octave> minus(bar, baz)
Bar.minus called!
ans = 2
```

One last thing, InferiorClasses doesn't override anything related to subsref, so if you explicitly wanted to call `Bar.minus`, you can still write 
```
octave> bar.minus(baz)
ans =  2
```

Chebfun uses this sort of method resolution ordering in several places; here's an example of a classdef header
```
classdef (InferiorClasses = {?chebtech2, ?chebtech1}) singfun < onefun 
%SINGFUN   Class for functions with singular endpoint behavior.
```
To give a bit of context, `singfun`, `chebtech1` and `chebtech2` are supposed to be somewhat interchangeable classes that implement the actual polynomial interpolation aspect of Chebfun, and you should be able to do basic operations like addition and subtraction interchangeably.
Since `singfun` overrides operations like `plus` and `minus`, and so does `chebtech1` and `chebtech2`, the `InferiorClasses` attribute is used so that each class does not need to implement all of the combinatorial possibilities of operations between them.

Here's a peculiar part of Chebfun: The `chebop` class has the following header

```
classdef (InferiorClasses = {?double}) chebop
```

Except, from the documentation given 

> The following MATLAB classes are always inferior to classes defined using the classdef syntax and cannot be used in this list.
>
> double, single, int64, uint64, int32, uint32, int16, uint16, int8, uint8, char, string, logical, cell, struct, and function_handle.

So it seems as if declaring 'double' to be an inferior class is a bit redundant, but not prohibited.


## Abstract

I've talked about this over at <a href="https://savannah.gnu.org/bugs/?67197" target="_blank"></a>, but I'll give a brief desription of the issue here.

```
classdef (Abstract) Foo
```
then Foo is not able to be instantiated; a subclass of Foo must be created that is not also labelled abstract. 

> A class is abstract when it declares:
>
>   The Abstract class attribute
>
>   An abstract method
>
>   An abstract property

So what this means is that

```
classdef Foo

    properties (Abstract)
        a
    end

    methods (Abstract)

        function retval = exampleMethod(this)
            disp("This is an abstract method!");
        end

    end
end
```

the first line should be interpreted as `classdef (Abstract) Foo`, even though there's no explicit declaration of the class as abstract itelf.
But in Octave, this isn't true; you can instantiate `Foo` directly.

In Chebfun, this problem exists with several classes, such as `fun`, `classicfun`, `chebtech`, `onefun`, and some others. These are classes that are not the leafs of an inheritance chain, and so they should not be able to be instantiated directly. Not a bug that causes an incompatibility, but nonetheless not something that should be allowed to be done by the end user.







