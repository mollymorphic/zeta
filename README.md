The Zeta Programming Language
=============================

This is an implementation of a Virtual Machine (VM) for the zeta programming
language I'm working on in my spare time. Zeta draws inspiration from LISP,
Smalltalk, ML, Python, JavaScript and C. The language is currently at the early
prototype stage, meaning the syntax and semantics are likely to change a lot
in the coming months.

At the moment, this platform and language is mostly of interest to tinkerers
and those with a special interest in compilers or language design. It's still
much too early to talk about adoption and long term goals. I'll be happy if
this project can help me and others learn more about language and compiler
design.

## Quickstart

Requirements:

- A C compiler, GCC recommended (clang untested)

- GNU make

- A POSIX compliant OS (Linux or Mac OS)

To built the zeta VM, go to the source directory and run `make`

Tests can then by running the `make test` command

A read-eval-print loop (shell) can be started by running the `./zeta` binary

## Language Design

Planned features of the zeta programming language include:

- Dynamic typing

- Garbage collection

- Separate integer and floating-point types (both 64-bit)

- Dynamically extensible objects with prototypal inheritance, as in JavaScript

- No distinction between statements and expression, everything is an expression, as in LISP

- A user-extensible grammar, giving programmers the ability to define new syntactic constructs

- Operator overloading, to allow defining new types that behave like native types

- Functional constructs such as map and foreach

- A module system

- A very easy to use canvas library to render simple 2D graphics and make simple UIs

- The ability to suspend and resume running programs

## Zeta Core Language Syntax

The syntax of the Zeta programming language is not finalized. The language is
designed to be easy to parse (no backtracking or far away lookup), relatively
concise, easy to read and familiar-seeming to most experienced programmers.
In Zeta, every syntactic construct is an expression which has a value (although
that value may have no specific meaning in some cases).

The Zeta grammar will be extensible. The Zeta core language itself (without
extensions), is going to be kept intentionally simple and minimalistic.
Features such as regular expressions, switch statements, pattern matching and
template strings will be implemented as grammar extension in libraries, and
not part of the core language. The advantage here is that the core VM will not
need to know anything about things such as regular expressions, and multiple
competing regular expression packages can be implemented for Zeta.

Here is an example snippet of what Zeta code might look like:

```
// Load/import the standard IO module
// Modules are simple objects with properties
io = import("io")

io.println("This is an example Zeta script");

// Fibonacci function
fib = fun (n) if n < 1 then n else fib(n-1) + fib(n-1)

/* Compute the meaning of life and print out the answer */
io.println(fib(42))

foo = fun (n)
{
    io.println("It's also possible to execute expressions in sequence");
    io.println("inside blocks with curly braces.");

    // Since we have parenthesized expressions, we could almost pretend
    // This is JavaScript code, except for the lack of semicolons
    if (n < 1) then
    {
        io.println("n is less than 1")
    }
    else
    {
        io.println("n is greater than or equal to 1")
    }

    // Local variables are declared by directly assigning to them
    x = 7 + 1

    /*
    The variable "unit" refers to this script or code unit, the scope of
    this unit is an object. To assign to the global scope, we need to do
    the following:
    */
    unit.y = 3

    // Global variables are resolved as you would expect, here z is 3.
    z = y

    // This creates a local y which is equal to 4, but the global y remains 3
    y = y + 1

    // We can also create anonymous closures
    bar = fun () x

    // This function returns the closure bar, the last expression we
    // have evaluated
}

// This is an object literal
obj = { x:3, y:5 }

// When declaring a method, the "this" argument is simply the first
// function argument, and you can give it the name you want, avoiding all
// of the JavaScript "this" issues
obj.method = fun (this, x) this.x = x

// Make the fib and foo functions available to other modules.
// If anyone imports this module, they will be getting a reference
// To the "exports" object we are writing to here:
exports.fib = fib
exports.foo = foo
```

Everything is still in flux. Your comments on the syntax and above
example are welcome.

## Zeta VM

I've chosen to implement the VM core in pure C (not C++), for the following reasons:

- To make low-level details explicit (for instance, the layout of hosted objects in memory)

- To avoid hidden sources of overhead

- To maximize portability. GCC is available on almost every platform in existence.

An important goal of the Zeta VM is that it should be easy to build on both
Mac and Linux with minimal dependencies. It shouldn't require any extra dependencies
to run out of the box. You may need to install libraries to use graphical capabilities
and such, but the core VM will built with only a C compiler installed on your
machine. Zeta must always be easy to install and get started with.

The zeta implementation will be largely [self-hosted](https://en.wikipedia.org/wiki/Self-hosting).
The core VM will implement an interpreter in C, but the garbage collector and zeta JIT
compiler will be written in the zeta language itself.

A custom x86 assembler and backend will be built for the JIT compiler, as was done
for the [Higgs](https://github.com/higgsjs/Higgs) VM. This will allow self-modifying
code to be generated by the JIT. The x86 backend will be implemented in the zeta language as well.

## Plan of Action

I plan to bootstrap Zeta in several stages. The first step is to write a
parser for the Zeta core language in C. This is already largely done. The core
parser will have limited capabilities and will not feature an extensible
grammar, as it will only serve to bootstrap the system.

The second step of the project is to implement an interpreter for the Zeta
core language. This interpreter will also have limited language support. The
point of the interpreter is to eventually interpret the code for the Zeta
JIT compiler, which will itself be written in Zeta. The interpreter will not
be fast, and it will not be running user code, at least not in later
implementations of Zeta. As of today, the interpreter is very incomplete.

In order to have an extensible grammar, we will implement a Zeta parser in
Zeta itself. This parser will run on the Zeta core interpreter. Implementing
the parser in Zeta will allow us to make use of objects, garbage collection,
closures, etc. It will also make it much easier for Zeta code to interface
with the parser in order to extend the grammar. I intend to implement the
self-hosted, extensible Zeta parser soon after the interpreter is ready, so
that we can showcase the power of the extensible Zeta grammar.

The Zeta JIT compiler will be written in Zeta and initially interpreted.
However, the goal is for the JIT compiler to be able to compile itself to
native x86 machine code. It will also be able to JIT compile the self-hosted
Zeta parser. Once these components are compiled to native code, the system
should run at an acceptable speed. Note that the first version of
the Zeta JIT compiler is likely to be quite far from the performance-level
of modern JavaScript JIT. I do believe, however, that we can very easily beat
the performance of the stock CPython and Ruby implementations, even with a
naive JIT.

You might be thinking that the startup of the Zeta VM will be dog slow if we
use an interpreted JIT compiler to compile itself, the Zeta runtime, and our
self-hosted Zeta parser, all of this before any user code can be run. You would
be right, this bootstrap phase is likely to take up to a few minutes. This is
where the ability to serialize an initialized heap to disk will come in handy.
Because all of the Zeta data structures are self-hosted, we will be able to
save the initialized heap to disk after the bootstrap/initialization is
complete. Then, restarting the VM from this saved point should only take
milliseconds.

