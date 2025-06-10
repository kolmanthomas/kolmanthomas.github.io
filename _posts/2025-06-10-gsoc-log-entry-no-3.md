---
title: GSoC Log Entry No. 3
subtitle: 'Classdef, part 1: Status and updates on missing features'
layout: default
date: 2025-06-10
keywords: GSoC, Chebfun, GNU Octave 
published: true
---

This is the first part of a two-part series on the status of classdef in GNU Octave.
The motivation for writing this post is that the <a href='https://wiki.octave.org/Classdef' target='_blank'>wiki section on classdef</a> is a bit outdated and is worth being refreshed.
I originally thought I could (1) cover everything in one post, and (2) discuss some classdef internals, but it turns out that testing classdef features and reported bugs took a lot of time.
I am working concurrently on saving/loading classdef objects, which is a highly requested feature; that will be a blog post of its own.

The first part of this series focuses on classdef features.
The first section covers class attributes, the second section covers classdef property attributes, and the third section covers classdef method attributes.
The wiki lists the status of classdef property and method attributes, but not class attributes for some reason.

**Wiki Status:** Status of the feature reported on the wiki page.

**Current Status:** Status of the features as tested by me.  This one is a bit of a judgment call on my part.

Either "Full Support" if it's flawless or has minor bugs, "Partial Support" if there are some major bugs, "Not Supported" if it doesn't work or there is major missing functionality, or "Unknown" if I can't figure out the behaviour of the property. 

All the testing is done on the dev branch (version 11) of GNU Octave, and the MATLAB testing is done on MATLAB online R2025a.



# Classdef class attributes

## Abstract

**Wiki Status: N/A**

**Current status: Full Support** 

Used to define a class that cannot be instantiated directly, but can be subclassed into a concrete class.
This works fine on Octave.
The attribute applied to a classdef doesn't force the subclasses to implement any methods or properties on their own; for that, you need to use the Abstract property and method attributes.

## AllowedSubclasses

**Wiki Status: N/A**

**Current status: Not Supported** 

This attribute is used to specify the names of which class(es) can have this class as a superclass.
Take for example the following three classdefs:
```
classdef (AllowedSubClasses = ?Subclass1) Superclass
end

classdef Subclass1 < Superclass
end

classdef Subclass2 < Superclass
end
```
In theory, trying to instantiate Subclass2 should throw an error.
In practice on Octave, it does not.

TODO: Test if this works in MATLAB.

## ConstructOnLoad

**Wiki Status: N/A**

**Current Status: Not Supported**

The point of this class attribute is that when a classdef object is loaded from a saved classdef object file, then instead of constructing the object behind the scenes by setting the properties, it will call a (no argument) constructor upon loading.
MATLAB documents exactly how <a href='https://www.mathworks.com/help/matlab/matlab_oop/understanding-the-save-and-load-process.html' target='_blank'>default</a> saving and loading of objects works.
This can be overrided by implementing custom 'saveobj' and 'loadobj' methods, but those are not supported in Octave yet.

## HandleCompatible

**Current Status: Full Support**

A logical attribute that toggles whether the class can be a superclass for a handle class or not.
Seems to work as intended.

## Hidden

**Wiki Status: N/A**

**Current Status: Unknown**
 
This attribute is used to hide the class (somewhat); according to MATLAB <a href='https://www.mathworks.com/help/matlab/matlab_oop/class-attributes.html' target='_blank'>documentation</a>, the only two cases in MATLAB are to hide the class from the 'help' and the 'superclasses' function, which the latter is not even implemented in Octave.
Octave's help function does not print superclasses anyways, so this attribute would not seem to do anything.

## Sealed

**Wiki Status: N/A**

**Current Status: Full Support**

Used to declare that the class cannot be subclassed.
Seems to work exactly as intended.
















# Classdef property attributes 

## AbortSet

**Wiki Status: Not Supported**

**Current Status: Not Supported**

See the <a href='https://www.mathworks.com/help/matlab/matlab_oop/set-events-when-value-does-not-change.html' target='_blank'>documentation</a> for a thorough description of this attribute.
Here's my attempt at explaining it: 
Normally, when you set the property of a classdef object, it calls the 'set' method (if one is defined), sets the attribute if not, and triggers a 'PreSet' and a 'PostSet' event.
This happens even if the property is set to the same value as it already has.
AbortSet is a way to disable calling the 'set' method every time the property value and the new value are the same.
It does this by calling the 'isequal' method between the new value and the current value of the property.
It also only works for handle classes, and not value classes.
I'm not quite sure if this is a performance optimization feature or not, but in my view, it doesn't make sense from a performance perspective, since you pay for a comparison operation every time with this feature enabled, for the chance that you avoid some unnecessary 'set' method calls.

Since events aren't supported yet in Octave, the only thing that this attribute would potentially support is skipping the 'set' method call under the specified condition.

Here's an example classdef to illustrate what should happen:

```
classdef AbortSetClass < handle
    properties (AbortSet)
        a
    end

    methods
        function ret = get.a(obj)
            disp("Accessing property 'a'");
            ret = obj.a;
        end

        function obj = set.a(obj, value)
            disp("Setting property 'a'");
            obj.a = value;
        end
    end
end
```
If you run the following sequence of commands:
```
obj = AbortSetClass();
obj.a = 1;
obj.a = 1; 
```
Then the second time you set the property `a` to 1, it should not call the `set` method, and thus not print "Setting property 'a'".
In Octave, it does however, so it doesn't seem to support this attribute.



## Abstract

**Wiki Status: Full Support**

**Current Status: Not Supported**

Abstract properties are properties that must be defined in subclasses of the classdef, in order for the subclass to be able to be instantiated.

```
classdef AbstractPropertiesClass 
    properties (Abstract) % Initialization is not allowed
        a 
    end
end
```

```
> absPropClass = AbstractPropertiesClass 
```

This runs without issue, but should throw an error because `a` is abstract and cannot be defined.
Also, the property `a` can be freely set and changed, which is clearly not the intended behaviour.
MATLAB's behaviour is to throw an error upon trying to instantiate:

```
Abstract classes cannot be instantiated. Class 'AbstractPropertiesClass' defines abstract methods and/or properties.
```
According to MATLAB <a href='https://www.mathworks.com/help/matlab/matlab_oop/abstract-classes-and-interfaces.html' target='_blank'>documentation on abstract classes</a>, the declaration of any abstract properties or abstract methods renders the whole class abstract, so MATLAB is consistent with itself in its behaviour.

I have reported this as a bug <a href='https://savannah.gnu.org/bugs/?67197' target='_blank'>#67197</a>.

## Access 

**Wiki Status: Partial Support**

**Current Status: Support**

This seems to work fully.

```
classdef VisibleParent
    properties (Access = public)
        FirstProp = 1
    end

    properties (Access = private)
        SecondProp = 2
    end

    properties (Access = protected)
        ThirdProp = 3
    end
end

classdef VisibleChild < VisibleParent
    methods
        function obj = VisibleChild()
            disp(obj.FirstProp)
            disp(obj.SecondProp)
            disp(obj.ThirdProp)
        end
    end
end
```
As expected, instantiating `VisibleChild` displays the value of `FirstProp` and `ThirdProp`, but throws an error when trying to access `SecondProp`. Relatedly, instantiating a 'VisibleParent' object and trying to query what 'SecondProp' and 'ThirdProp' are throws an error.

Bug <a href='https://savannah.gnu.org/bugs/index.php?60007' target='_blank'>#60007</a> is related, but no longer seems to be an issue. 
I've reported this as fixed.

## Constant

**Wiki Status: Full Support**

**Current Status: Partial Support**

Allows for constant (non-modifiable) properties in classdefs.

This works for setting literal values.
If you try to set a constant property to be a result of arithmetic operations on other constant properties, it will not run in Octave (it will in MATLAB).
This is documented in <a href='https://savannah.gnu.org/bugs/index.php?57557' target='_blank'>#57557</a>.

You can do this as a workaround:
```
classdef ConstantClass
    properties (Constant)
        a = 5
        b = 10
        c
    end

    methods
        function out = get.c(obj)
            out = obj.a + obj.b; % Or any other expression involving constant values
        end
    end
end
```
which is valid in Octave.

I would personally downgrade the status to only partially supported if it cannot handle any more complexity than definining literal values.


## Dependent

**Wiki Status: Partial Support**

**Current Status: Partial Support**

A Dependent property is something that cannot store data directly, but instead computes its value inside of the associated 'get' method.

```
classdef Dependent
    properties 
        a 
        b 
    end

    properties (Dependent)
        c 
    end

    methods
        function out = get.c(obj)
            out = obj.a + obj.b; 
        end
    end
end
```
Like we've seen with the Constant property attribute, you can do the same thing without the Dependent property attribute.
It's just that with the Dependent property attribute, not defining a 'get' method should throw an error; it however does not throw an error in Octave.
Octave blocks you from setting the value of `c` directly, though, not by an error but by simply ignoring the value.

## GetAccess

**Wiki Status: Partial Support**

**Current Status: Full Support**

It's like the Access property attribute, but only for getting the property value, not setting it.
Seems to work totally fine: here's an example:
```
classdef GetAccessClass
    properties (GetAccess = private)
        a    
    end

    methods
        function print_vals(obj)
            disp(obj.a);
        end
    end
end
```
`a` is not able to be queried directly, but the value is visible from a call to `print_vals`.
Setting the property attribute to 'protected' works similarly, but allows the value to be queried from subclasses.
I can't find any bugs related to this, so I'm upgrading the status to Full Support.

## GetObservable

**Wiki Status: Not Supported**

**Current Status: Not Supported**

This requires the events and listeners functionality, which is not implemented in Octave.
More on this down below.

## Hidden

**Wiki Status: Full Support**

**Current Status: Full Support**

The Hidden property attribute hides properties from the 'properties' function. 

Here's an example classdef with a hidden property:
```
classdef HiddenPropertiesAndMethods
    properties
        a = 1
        b = 2
    end

    properties (Hidden)
        c 
    end

    methods
        function ret = get.c(obj)
            ret = obj.b + obj.a;
        end

        function obj = set.c(obj, value)
            obj.c = value;
        end
    end
end
```
Calling `properties(HiddenPropertiesAndMethods)` in Octave will return the properties `a`, `b`, but not `c` as expected.

## NonCopyable

**Wiki Status: Not Supported**

**Current Status: Not Supported**

The superclass "matlab.mixin.Copyable" is not implemented in Octave, so this attribute is not supported. 
See the MATLAB <a href='https://www.mathworks.com/help/matlab/matlab_oop/custom-copy-behavior.html' target='_blank'>documentation</a> for more information on this attribute.

## SetAccess

**Wiki Status: Partial Support**

**Current Status: Full Support**

Like the GetAccess property attribute, but for setting the property value instead of getting it.
Similar story as with GetAccess: it works better than its reported status, cannot find any bugs related to it.

## SetObservable

**Wiki Status: Not Supported**

**Current Status: Not Supported**

Like getObservable, this requires the events and listeners functionality.

## Transient

**Wiki Status: Not Supported**

**Current Status: Not Supported**

This attribute is supposed to mark properties that shouldn't be saved when a classdef object is saved to a file.
Since saving and loading classdef objects is not implemented yet, this attribute is not supported.







# Classdef Method Attributes

## Abstract

**Current Status: Partial Support**

<a href='https://savannah.gnu.org/bugs/?51377' target='_blank'>#51377</a>

Somewhat similarly to the bug for Abstract properties discussed above (<a href='https://savannah.gnu.org/bugs/?67197' target='_blank'>#67197</a>), instantiating a classdef with an abstract method works:
```
classdef AbstractSuperclass
    methods (Abstract)
        function res = f(obj)
        end
    end
end
```
but trying to call `f` results in the error:
```
error: f: cannot execute abstract method
```
as opposed to MATLAB's different error, which happens at instantiation of the object:
```
Abstract classes cannot be instantiated. Class 'AbstractMethodsClass' defines abstract methods and/or properties.
```

## Access

**Current Status: Full Support**

Works like the Access property attribute, but for methods.
Everything works as expected.

## Hidden

**Current Status: Full Support**

Works like the Hidden property attribute, but for methods.
Seems to work as expected.

## Sealed

**Current Status: Full Support**

This does not work at all in Octave.
Here's an example of the simplest failure:
```
classdef SealedSuperClass
    methods (Sealed)
        function out = increment(self, x)
            out = x + 1;
        end
    end
end
```

```
classdef SealedSubClass < SealedSuperClass
    methods 
        function out = increment(self, x)
            out = x + 2;
        end
    end
end
```
In Octave, calling `increment` on an instance of `SubHiddenProperties` doesn't throw an error as expected.
For instance
```
s = SealedSubClass();
s.increment(5)
```
returns `7`, meaning that it called the subclass method `increment`. 
In MATLAB, however, it throws an error as expected:
```
Method 'increment' in class 'SealedSubClass' conflicts with the sealed method in superclass 'SealedSuperClass'. Overriding a sealed method is not supported.
```
 

## Static

**Wiki Status: Full Support**

**Current Status: Full Support**

Static methods are methods that can be called without instantiating the class (but also, can be called on an instance of the class).

Static methods mostly work fine on Octave.
There are two bugs I could find related to static methods:
The first (bug <a href='https://savannah.gnu.org/bugs/index.php?52582' target='_blank'>#52582</a>) is that if you try to initialize a property with a static method, it will err.
Something like this works in MATLAB, with or without a (Constant) property attribute:
```
classdef StaticMethodsClass
    properties
        a = StaticMethodsClass.increment(5)
    end
    methods (Static)
        function out = increment(x)
            out = x + 1;
        end
    end
end
```
but not in Octave.
Some users report a segfault, I'm getting an illegal instruction error; regardless, the one constant is that it doesn't work.

The second (bug <a href='https://savannah.gnu.org/bugs/index.php?62001' target='_blank'>#62001</a>) can be explained by the following example classdef:
```
classdef StaticMethodsClass
    properties
        statProp = StaticMethodsClass.staticMethod(5);
    end
    methods
        function out = staticMethod(x)
            out = x + 1;
        end
    end
end
```
If you try to use 'feval' to call the static method like this:
```
feval('ClassName.methodName')
```
then you will get an error (again, the error seems to differ between different systems).

I think these are really minor issues with workarounds, so I would say that static methods are fully supported with very minor caveats.


# Classdef features that are not implemented

## Enumerations

Enumerations are a way to associate names with constant values.
The contrast with constants is that enumerations are a type associated with the class, and could theoretically provide stricter type checking.

```
classdef MyEnum
    enumeration
        FirstValue, SecondValue, ThirdValue
    end
end  
```

Octave chokes on this and issues a syntax error.


## Events and Listeners

Events 
```
classdef EventsClass < handle
    events
        StateChange        
    end

    methods
        function triggerEvent(obj)
            notify(obj, 'StateChange');
        end
    end
end
```

The events block in a classdef can be parsed perfectly fine, it's just that nothing happens afterwards with them.

## Property Validation Functions

Property validation functions impose restrictions on the values and types of the properties of a classdef.

```
classdef PropValidFcn 
    properties
        Prop {mustBePositive, mustBeFinite} = 5
    end
end
```
If you were to run this code, 
```
warning: size, class, and validation function specifications are not yet supported for classdef properties; INCORRECT RESULTS ARE POSSIBLE!
```
and if you were to change the value of `Prop` to a negative number, it would do so without throwing an error.

I think this is really key to have for good object-oriented design, since Octave's type system is rather lax otherwise.
It would be very nice to have this implemented.



