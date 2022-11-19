+++
title = "SMT solvers for fun and CTFs"
date=2022-11-04
description = "Satisfiability modulo theories, solvers and how Conda uses them."
showFullContent = false
readingTime = false
+++

# Introduction
It is no surprise even to an amateur that computer systems are built on 
steadfast grounds of mathematical theory. Indulging in computers over the 
course of many years leads many into finding very niche fields where 
computers and mathematics intersect directly. 

In this post, we turn our attention to one of such systems known as 
Satisfiability Modulo Theories (SMT), which prove satisfiability of first-order logical equations. We'll then visit Conda to see how they use a SMT solver for a neatly practical cause.

# Mathematical logic
While peeking into the history of computers, we find that mathematical 
logic first entered into computing through lambda calculus and combinatory logic. 

In modern times, mathematical logic has evolved into a solid tool used 
for discussing semantics of programming languages. Incidentally, if you want to peek into the action, there's no better window than Ralf Jung's 
brilliant [PhD thesis](https://research.ralfj.de/thesis.html) on Rust's semantics. Although interesting, it is a PhD thesis, so we our attention 
to more digestible examples.

If you've trudged your way through the terrible education of the country I live in, you've probably encountered something called [Linear Programming](https://en.wikipedia.org/wiki/Linear_programming). It poses optimization problems by presenting equations constrained to specific conditions and asks us to deliver the most optimal solution under those specific conditions. A classic problem consists of a linear function $f(x, y) = ax + by$ 
and a set of $n$ constraints $a_nx + b_ny \leq c_n$. 

What they don't teach you in school, is that those $n$ constraints can be 
modeled as first-order logic equations as well. For example, consider a 
system of three linear equations:

$$
\begin{align*}
-3x + 2y - 5z &= -14  \\\
2x - 3y + 4z  &= 10  \\\
x + y + z &= 4  
\end{align*}
$$

Since it is absolutely necessary that we satisfy all three of these 
constraints to find values of $x$, $y$ and $z$. A 'winning' condition for
us would be when a solution to such a system exists, and can be 
calculated. If the winning condition is denoted with $W$, then we can 
express $W$ as:

$$
(-3x + 2y - 5z = -14) \wedge (2x - 3y + 4z  = 10) \wedge (x + y + z = 4)  
$$

The $\wedge$ symbol is the usual conjunction (AND condition).

Now that we've modeled this as a logic problem, we can finally talk about Satisfiability Modulo Theories.

## Satisfiability Modulo Theories
Abbreviated as SMT, the instance of such a system is a formula in 
first-order logic (such as the one above), and SMT determines whether if 
this system is indeed solvable. 

SMT systems can include more complex data formats such as integers or data 
structures like lists, strings and arrays.

Now let's talk about SAT systems, of which SMT systems are a generalization.

# SAT and Z3 
Boolean **SAT**isfiability problem is a specialization of SMT that only 
concerns itself with satisfiability of a boolean formula.

SAT systems try to find a solution to a system of first-order logic equations. If a solution exists, then the system is evaluated to a SATisfiable status. Otherwise, it is UNSATisfiable.

A condition on SAT systems is that they only operate on first order logical systems in [Conjunctive Normal Form](https://en.wikipedia.org/wiki/Conjunctive_normal_form). Do you see how that applies to our example above? We'll bubble this example through the entire section.

If you dig further into SAT, you're bound to discover that there are 
automated solvers for it. SAT problem is NP-complete and automated solvers have been able to massively contribute to complex systems such as microprocessors or scheduling problems, which involve hundreds and thousands of variables and constraints.

A very popular automated theorem solver of this sort is [Z3 by Microsoft](https://github.com/Z3Prover/z3). Let's see how we can use Z3 to solve our problem that we discussed in the first section of this post. I like to use Z3 through Python. The code is pretty self-explanatory coupled with the comments.

```python
# Import the Z3 library 
from z3 import *

# Initialize the Solver 
s = Solver()

# Describe our variables in the system
x, y, z = Ints('x y z')

# Add all three equations into this system
# These equations are assumed to be conjunctive
s.add(-3*x + 2*y - 5*z == -14)
s.add(2*x - 3*y + 4*z == 10)
s.add(x + y + z == 4)

# Check if a solution exists
print(s.check())

# If it does, describe it
print(s.model())
```

We then execute this script to find the following. 
```bash
$ python sat_equation.py
sat
[z = -4, y = -2, x = 10]
```
The `check` method returns `sat`, which means our system is indeed solvable. We can also see that it calculated the solution for us too. Neat!

You can try playing around different permutations of our script, to crash into various `unsat` results too. But how do we ever use this in real life? 

## A toy example with Z3
The example that follows models a pretty real-life situation. 

> **Problem**: You went to a fancy restaurant with your friends and you can pool together a sum of $1505. There are six dishes on the menu. You must decide with your friends, the quantity of each dish you can order so that you can eat well and still afford the entire course.

This is a very neat problem to model in your brain. Does it remind you
of linear programming? We can indeed solve this in Z3, and we will!

Assume that the six dishes are boringly named $A, B, C, D, E$ and $F$ and have prices as described below. We label the quantity of each dish 
by the corresponding lower case letter. 
Thus, we can express the given problem as a series of linear equations. As the quantity of dishes is non-negative, it makes sense that $a \geq 0,\ b \geq 0,\ c \geq 0,\ d \geq 0,\ e \geq 0 \text{and} \ f \geq 0 $.

Finally, we express our financial constraint as following. Notice that 
quantity of each dish is multiplied by its price.

$$a\*215 + b\*275 + c\*335 + d\*355 + e\*420 + f\*580 = 1505$$

You can further model these equations in CNF, like we did in the example we saw earlier. Cool! Let's turn to Z3 and Python.

```python

# Import the Z3 library
from z3 import *


# Initialize the solver
s = Solver()

# Set up the variables (Dish quantities)
a, b, c, d, e, f = Ints('a b c d e f')

# Add constraints
# Quantity of dishes must be non-zero 
s.add(a >= 0)
s.add(b >= 0)
s.add(c >= 0)
s.add(d >= 0)
s.add(e >= 0)
s.add(f >= 0)

# Add our financial constraint 
s.add(a*215 + b*275 + c*335 + d*355 + e*420 + f*580 == 1505)
```

It is important to understand that such problems can have multiple 
solutions, and we need to enumerate all of them. So, we write some more 
code which does exactly that. It is fine if you skip over this snippet.

```python
# Enumerate all possible solutions:
results=[]

while True:
		# If we found a solution, add it to the system 
		# and check for another
    if s.check() == sat:
        m = s.model()
        print(m)
        results.append(m)
        block = []
        for d in m:
            c=d()
            block.append(c != m[d])
        s.add(Or(block))
    else:
        print("Total solutions: ", len(results))
        break
```

Running this code,

```bash
$ python sat_equation.py
[d = 0, b = 0, a = 7, f = 0, c = 0, e = 0]
[c = 0, d = 2, e = 0, f = 1, b = 0, a = 1]
Total solutions:  2
```

Turns out there are two ways for us to order our dishes! We can either:

- Order 7 of dishes $A$.
- OR order 1 dish of $A$, 2 of $D$ and 1 of $F$.

Very neat Z3! This satisfies our curiosity for the more applicative aspect of such solvers.

## CTF experiences
In computer applications, you'll often find anti-tamper or 
anti-cheating mechanisms. These mechanisms often place a lot of restrictions on what the user can and cannot do. 

A very toy-ish example finds its place in CTFs, where a very large number of constraints are given inside a binary, and we have to figure out a particular value for a variable that satisfies these constraints. For instance,

```c
(arg1 + 0x14) ^ 0x2b) == *(arg1 + 7)) &&
(*(arg1 + 0x15) - *(arg1 + 3) == -0x14)) &&
(*(arg1 + 2) >> 6 == '\0')) &&
((*(arg1 + 0xd) == 't' && 
((*(arg1 + 0xb) & 0x3fffffffU) == 0x5f)))) &&

// .... HUNDREDS of such checks ....
```

Notice how the `arg1` variable is constrained by all of these 
equations consisting of byte-level operations. To solve such a system, we use Z3 again in the hopes of finding a solution. The solution is a value 
of `arg1` that satisfies all these checks. Imagine finding it by hand!

Apart from this classic scenario, there have been challenges implementing custom cryptography. Such as [this one](https://www.hackthebox.com/blog/memory-acceleration-ca-ctf-2022-crypto-writeup), which require you to use Z3 to solve them. 


# How Conda uses SMT solvers
Recently, I unexpectedly came across a **very** interesting application for SAT solving systems. Apparently, the Conda project uses them to improve their performance!


Dependency management is a very interesting problem in computer science. If you have ever read a config file that describes dependencies, such as `package.json` for Javascript projects, or `requirements.txt` for Python projects or `Cargo.toml` for Rust projects, you know what dependency constraints look like! They're often something like this:

```
docopt == 0.6.1             # Version Matching. Must be version 0.6.1
keyring >= 4.1.1            # Minimum version 4.1.1
coverage != 3.5             # Version Exclusion. Anything except version 3.5
Mopidy-Dirble ~= 1.1        # Compatible release. Same as >= 1.1, == 1.*
```

Conda isolated the performance issue to that specific dependency management module of theirs. They modeled dependency management as a SAT problem in first-order CNF logic, then used their in-house SAT solver to figure out valid dependency versions quickly during runtime and improve performance! How cool!

You can read Conda's blog post on this: [Here](https://www.anaconda.com/blog/understanding-and-improving-condas-performance)



# Exercise to the reader
I once played a very nice CTF. There was a reversing challenge in there, 
which I couldn't solve at that time for some very frustrating reason. A part of that challenge involved huge number of constraints in the binary. 

The system wouldn't return `sat`, even though it was supposed to. 
I've snipped the challenge quite a bit.
You're given a solution code for the challenge.
The solution code is incomplete. 
Complete it. 
Does it return `sat`? 
What's the solution?

[Solve me!](https://gist.github.com/Crystalsage/1536740eb0608458627569f2092f52f6)



$$
\blacksquare
\blacksquare
\blacksquare
$$
