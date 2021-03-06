---
layout: post
title: A Charming Proposal of the Functional Stack
date: 2018-04-19 13:17 -0700
categories: programming charm
---
_**A Lisp Philosophy**: [All code is data and all data is code.](http://wiki.c2.com/?DataAndCodeAreTheSameThing)_

From this, we can deduce a few things. Most code interacts with data through subroutines of some sort, taking in arguments or a global state, then subsequently returning a value or modifying a state. These arguments, values, and states are the data manipulated in the program. Data is traditionally restricted to values such as numbers or strings, but using the definition mentioned above; data can be extended to include code itself as well.

What are the effects of extending the definition of data like this? Consider this little program, written in [Reverse Polish Notation](https://en.wikipedia.org/wiki/Reverse_Polish_notation):
```
1 2 +
```
Let's break this down. The function `+` describes a function that adds two numbers together, then it's applied to the two arguments `1` and `2`. With our new definition of data, though, there is no reason that these two arguments have to be values. Consider the following example, now written in [Charm](https://github.com/aearnus/charm), the subject of this article.
```
[ 1 2 + ] i 3 +
```
What's going on here? Well, the "program" `1 2 +` was listed out; then, as a whole, it was passed to a function `i`. This function, `i`, [interprets a list at the top of the Charm stack](https://aearnus.github.io/charm/#i-id). Finally, the result of `i` and the number `3` are passed to `+` and the computation concludes.

Wait... a function that _interprets and executes a list_? It sounds awful -- that is, until you remember the aforementioned code philosophy.

This function `i`, and many other higher level functions like it, beget the true power of Charm. Take the humble if/then block, for example. In a C like language, it may look something like this:
{% highlight c %}
if (condition) {
    do something;
} else {
    do something else;
}
{% endhighlight %}
Compare that with the equivalent code in Charm....
```
[ condition ] [ do something ] [ do something else ] ifthen
```
Right off the bat, it's much more obvious what Charm's if/then construct does behind the scenes. It takes three programs (denoted with square brackets), then uses the first one to generate a condition, the second one to run if the condition is true, or the third one to run if the condition is false. The C like notation, on the other hand, is wholly ambiguous. The programmer of that language does not know if the if/then block is allowed to evaluate as an expression, or if it has to be a statement; if the inner blocks have access to global scope; or even if the parser has special rules for special cases (I'm looking at you, [bracket-less if statements...](https://stackoverflow.com/questions/97506/formatting-of-if-statements)).

If/then blocks may be a simple example, but it evinces a larger problem: the problem of unnecessary complexity and syntax riddled with gotcha's. Case statements, for loops, class syntax; all of these use complex syntactic rules and keywords galore. Keywords and syntax aren't the solution to ease of use. (Try telling that to C++!)

## Charm is unambiguous

Charm prides itself on having minimal syntax, so that every program can be clearly read and understood by the programmer. The entirety of Charm's syntax can be put on the back of a notecard -- or in a tiny text box on the screen:
```
program = [{ typesignature definition } | { function space }] eoi;
space = ' ';
function = name | '[ ' name ' ]' | '" ' name ' "';
definition = name ' := ' [{ function space }];
typesignatureunit = { name space } '-> ' { name space } [{ '| ' typesignatureunit }];
typesignature = name ' :: ' typesignatureunit;
```
Charm is entirely made up of space delimited tokens to avoid ambiguity and speed up the parser. There are only four special syntactic constructs, which are listed [here](https://github.com/aearnus/charm/blob/master/Readme.md#basic-syntax-and-implementation).

## Charm is expressive

The distinct lack of syntactic cruft allows for the code to "speak for itself", without the line noise and distraction of complicated syntax. This means that the expressiveness per line of Charm is incredibly high. Here's an example of a highly expressive line that doesn't get clouded by complex syntax:
```
[ 1 2 3 4 5 ] [ i 2 * q ] map
```
Of course, there are two programs here being pushed onto the stack and pushed to the function `map`. The first program is homeomorphic to a list of the first five natural numbers, and the second program essentially unboxes a function provided by the map function, multiplies the number by 2, and boxes the function back up to send back to map. The overall output of that program is then, of course, `[ 2 4 6 8 10 ]`. In a traditional language (such as Ruby, in this example), this would be equivalent to:
{% highlight ruby %}
[1, 2, 3, 4, 5].map{|n| n * 2 }
{% endhighlight %}
Now, let me put it out there: I love Ruby. But, there is a significant amount of line noise, everywhere from the pipes in the block argument to the commas in the list. Every little thing contributes to the fact that Ruby is a "read-only" language. Even compare that to an implementation in C++:
{% highlight c++ %}
#include <algorithm>

...

std::vector<int> v = {1, 2, 3, 4, 5};
std::for_each(v.begin(), v.end(), [](int n){ return (n * 2) });
{% endhighlight %}
... and I won't even comment.

## Charm is meta

At its core, a program is simply a list of functions to run in order. Thus, all code that runs in Charm is equivalent to a mere list. This was hinted at earlier through the use of the word "program" instead of the use of the word "list", but needed to be cleared up before moving onto this point. Because Charm works in this way, metaprogramming is as easy as writing a list. Take the small program here, for example
```
1 1 +
```
As you can probably tell, it simply adds 1 and 1 together. But, with a little bit of metaprogramming magic...
```
[ 1 1 + ] 2 repeat [ + ] 2 repeat concat i
```
To figure out what this little block does, we have to break it down step by step. First, let's look at the `[ 1 1 + ] 2 repeat`. [The glossary says that `repeat`](https://aearnus.github.io/charm/#repeat-id) repeats a list a certain number of times. So, that little piece turns into `[ 1 1 + 1 1 + ]`

Next, we'll look at `[ + ] 2 repeat`. Using the same logic as above, this turns into `[ + + ]`.

Penultimately, we'll check out `concat`. [According to the glossary](https://aearnus.github.io/charm/#concat-id), `concat` concatenates two lists or strings. Thus, it turns our `[ 1 1 + 1 1 + ]` and our `[ + + ]` into a `[ 1 1 + 1 1 + + + ]`.

Finally, last (but not least!), we have the function `i`. `i`nterpret [runs the top of the stack as a program](https://aearnus.github.io/charm/#i-id). Thus, our `[ 1 1 + 1 1 + + + ]` is executed to yield the final result of `4`; which we got through run-time metaprogramming. Our program _wrote_ the program that gets run to produce our output. Charm allows you to build up a program as you run your program, and that's a fantastic idea both in theory and in practice.

Though briefly touched upon above, it is worth mentioning that all of Charm's many list manipulation tools can be used to manipulate programs themselves. An example of this in action is the function [`stepthrough`](https://aearnus.github.io/charm/#stepthrough-id), which is an interactive debugger for Charm... written in Charm. It modifies the program it is fed in real-time in order to show the execution order and stack state after every function. [Here's a video of it](https://asciinema.org/a/w0cWh9eJfJ05ZcR6wfXOm7MuG) (excuse the error at the end -- the REPL doesn't know how to handle the end of input, and asciinema requires you to press `C-D` to end recording).

## Charm is safe

If you are anything like me, when you hear "stack-based", the first images that come to your mind are traumatic recollections of smashed stacks and mismatched variadic arguments. Charm, though, uses a unique run-time (planned parse-time) type system to ensure no functions or programs do anything they aren't intended to do. This run-time type system gives Charm the safety of a strongly typed interpreted language -- something that isn't often seen in stack-based languages. A type signature, in very general terms, looks like this:
```
<function name> :: <types> -> <types> | <alternate type signature> | ....
```
If you can't visualize that, perhaps perhaps a concrete example would help. This is the type signature for `map`, ripped straight out of the Charm prelude:
```
map :: list list -> list
```
This means that `map` can pop two lists (a program and a list to map the program over) off the stack and it can push one list on the stack (the mapped list). Any other behavior, such as popping an `int` instead of a list, causes the interpreter to halt and announce the type mismatch, in the form of
```
Charm (Stack 0)$ 10 [ i 1 + q ] map
[RUNTIME ERROR]: Type signature check for function `map` failed.
The function has type signature `list list -> list ` but it popped types `int list `
```

... and you would never even dream of getting that in any other language in Charm's class.

## Charm is easily extendable

I will admit that not every problem is suitable to be solved using Charm's abstractions. This kind of concatenative, functional, stack-based programming is very good for some applications (DSLs, list processing, recursive algorithms); but not suitable for many others. Plain and simple, Charm is not always the right tool for the job. Admitting that, though, is one step in the process towards making a truly good tool. Thus, there is a large focus on making Charm easily extendable through a C/C++ FFI interface. Since code speaks louder than words, here's an example of a simple Charm application that implements the C++ FFI:

In a C++ file `lib.cpp`, I've placed the code
{% highlight C++ %}
#include <charm/CharmFFI.hpp>
#include <iostream>

extern "C"
MutateFFI charmFFIHelloWorld(Runner* r) {
    CharmFunction f1 = r->getCurrentStack()->pop();
    std::cout << "Hello from C++!" << std::endl;
    std::cout << "This is what I saw on the top of the stack: " << charmFunctionToString(f1) << std::endl;
}
{% endhighlight %}

Notice the return type of our FFI function, `MutateFFI`. It's defined as `typedef void (*MutateFFI)(Runner*)` and used to aid in the process of writing functions that interact directly with Charm. Then, it's compiled as a shared library and linked with `libcharmffi`. Once we've compiled that shared library, it can be loaded _directly into_ Charm through the code
```
" hello " " ./lib.so " " charmFFIHelloWorld " ffi
```
and called by saying `hello`! This whole example (and along with a Makefile) is available [here on GitHub](https://github.com/Aearnus/charm/tree/master/test/ffi).

With this easily extendable interface, Charm can fit into a niche that's not quite general purpose, but moreso as a higher level _interface_ to lower level functions. Though it isn't explored in this blog post, Charm can also serve as an extremely powerful DSL through its C++ extensions capability.

## Final thoughts

You may very well be surprised at the elegance and ease of understanding that comes once you begin to use Charm. That feeling, though, can't be written into a blog post. You'll have to try it yourself. :)

## Try Charm!

Check out the [TryItOnline page for Charm](https://tio.run/#charm)! I'd also implore you to visit [the Charm GitHub page](https://github.com/Aearnus/charm) to download it and try it out yourself. Finally, we have a page on the [Esolangs Wiki](https://esolangs.org/wiki/Charm)!
