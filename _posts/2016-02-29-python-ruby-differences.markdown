---
layout: post
title: Python + Ruby Differences
date:   2016-02-23 08:29:23 -0500
categories:
---

**in progress...**

I'm taking a Data Science course that incorporates Python. I thought I'd write a quick + dirty summary of some key (cool) differences between the two languages. Forgive me if the syntax is slightly off as I'm writing this from memory.

#### Methods
Python and Ruby both use the method declaration `def`, but Python also uses `():` at the end of the declaration. In Ruby, the parens are optional. In Python, the parens and colons are required. Python also requires colons for for loops and if/else statements.

Python does not need the `end` keyword to signify the end of a method block. Python relies on whitespace and indentation to signify the end of a block of code.

In Ruby, methods return the last line of code that is evaluated. In Python, you must be explicit and use `return`.


#### Objects
In Ruby, a collection that uses index is called an Array. Python calls them lists.

In Ruby, a collection of key/value pairs is called a Hash. Python calls them Dictionaries.

#### Classes
Ruby uses the `initialize` method to instatiate new instances of a class. Python uses `__init()__`. 

In Python, you must pass in the keyword `self` as the first argument in a method. In Ruby, you can assign a method to self as in `self.name_of_method`.

#### `&&`, `||` operators
Ruby uses `&&` and `||` for comparisons. Python uses the keywords `and` and `or`.


#### Whitespace and colons 