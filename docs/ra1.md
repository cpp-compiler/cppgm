## Reading Assignment A (core)

### Overview

After reading this assignment description, read the 355 pages numbered 32 through 386 of [N3485](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3485.pdf). This includes clauses 3 (Basic Concepts) through to 15 (Exception handling) inclusive.

After this assignment you will have a complete exhaustive knowledge of the C++ core language, not including the standard library. Such knowledge will be assumed for the remainder of the course.

### Prerequisities

You should complete Programming Assignment 6 before starting this assignment. PA6 is necessary to get you used to the mapping between nonterminal names, such as a `class-specifier`, and code examples of these nonterminals, such as `struct { ... }`. You will need to know this mapping for efficient comprehension of the text.

### Discussion

We are getting into the hardest part of the course. One of the most useful things about C++, compared to other modern industrial languages, is how much the compiler can do during static analysis (aka at compile-time, during translation), however implementing all these features is quite difficult.

In the next programming assignments you will be implementing a large part of the semantic analysis phase (translation phases 7 and 8), including but not limited to symbol table management, name lookup, template instantiation, evaluation of constant expressions, overload resolution, and so on. This also necessitates building up the foundation type system handling, including complex compound types such as user-defined classes, including inheritance.

It will be necessary to have complete knowledge of all the C++ _language features_ (not including the standard library) in order to proceed.

If you already possess an expert knowledge of C++ than you may be able to get away with making a single pass of the assigned 355 pages. For everyone else, you will undoubtably need to make two passes.

In either case we suggest making only forward passes of the documentation. That is, do not jump around looking for definitions as you read. If there is terminology you do not understand on your first pass, leave it or make a note about it, and press on. You should be able to tie up forward references on your second pass.

If there is a requirement in N3485 that you do not understand please post to forum.cppgm.org with tag `raa`. Make sure to include the section number and quote the text you are unsure about.

As you are reading try to imagine the different ways in which each requirement could be implemented in a C++ compiler. If you can't do this in all cases it is ok, but make sure you at least understand the _required_ behaviour of the compiler, even if you are not sure how to _implement_ that behaviour yet.

### Architecture

It's important to have an overview of the translation process before you start reading the assigned material. The C++ spec is designed so that a C++ compiler can almost(*) make a single forward parse of a translation unit. As it parses entities are declared and defined. The C++ compiler acts on these declarations and definitions to add them to an "internal database", which you will represent in memory using normal object-oriented techniques. As you encounter usages of these entities during parsing you will look them up in this _partially completed_ database to resolve relationships, and then record those relationships. The important thing to understand is that (in general) at entity use, a needed declaration or definition is required to be above the use in the translation unit - this is specifically to support this single pass architecture.

> Side Note: (*) We say "almost" because there is an exception to do with certain definitions within a class specifier - specifically inline function bodies, default parameters and initializers. So that they can forward reference class members and use their enclosing class as a complete type, these items must be delimited and then defered to be parsed after the class specifier has completed, just like they were defined after the class specifier in the translation unit - but you will delve deeper into this in the appropriate assignment.

### Reading Schedule

We've delimited the material for you into 9 blocks of approximately 40 pages each. For a single pass if you schedule 2 hours per day to read one block, than this will take 9 days and a total of 18 hours. If you need to make 2 passes it will take double that of course.

    Block 1: 51 pages
       3 Basic Concepts
       4 Standard Conversions

    Block 2: 50 pages
       5 Expressions
       6 Statements

    Block 3: 39 pages
       7 Declarations

    Block 4: 32 pages
       8 Declarators

    Block 5: 38 pages
       9 Classes
       10 Derived Classes
       11 Member Access Control

    Block 6: 31 pages
       12 Special Member Functions

    Block 7: 30 pages
       13 Overloading

    Block 8: 42 pages
       14.0..14.6 Templates Part I

    Block 9: 42 pages
       14.7..14.8 Templates Part II
       15 Exceptions

After completing this assignment you do not have to do anything, you are expected to complete it before commencing Programming Assignment 7\. RAA is self-assesed and will be due on the same day as PA7.
