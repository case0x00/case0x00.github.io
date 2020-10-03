---
layout: post
title: busy beaver problem in haskell 
tag: projects
---


## the problem

The busy beaver problem is formally defined as finding the maximum number of 1s that an n-state Turing Machine with a two symbol alphabet can print on a tape before halting. The general problem is non-computable. The inputs to the system are the current non-holt state and the symbol in the current tape cell. the outputs are the symbol to write over the current symbol in the current tape cell, a direction to move, and a state to transition into. The machines score is the number of 1s written to the tape after halting.

The busy beaver function is denoted as $$ \Sigma : N \to N $$ such that $$ \Sigma (n) $$ is the maximum attainable score among all halting binary alphabet n-state TMs. Although $$ \Sigma (n) $$ is an uncomputable function and grows faster than any computable function, there are some small $$ n $$ for which it is possible to obtain their values and prove they are correct.

* $$ \Sigma (1) = 1 $$
* $$ \Sigma (2) = 4 $$
* $$ \Sigma (3) = 6 $$
* $$ \Sigma (4) = 13 $$

For $$ n > 4 $$, $$ \Sigma (n) $$ has not yet been determined. The busy beaver problem is especially interesting because of its growth as $$ n $$ increases. The current 5-state busy beaver champion produces $$ 4098 $$ 1s in 47 million steps. The current 6-state busy beaver champion produces $$ 3.515 e^{18267} $$ 1s in over $$ 7.412 e^{36534} steps. It was proven that for a 10-state busy beaver, $$ \Sigma (10) > 3 \uparrow \uparrow \uparrow 3 = 3 \uparrow \uparrow 3^{3^3} $$ which represents iterated exponentiation. This applies exponentiation iteratively instead of multiplication iteratively (which is what exponentiation is defined as, iteratively multiplying). This value is $$ 3^3 $$ but with $$ 7.6 e^{12} $$ copies of 3. For a 12-state busy beaver, $$ \Sigma (12) > 3 \uparrow \uparrow \uparrow \uparrow 3 = g_1 $$ where $$ g_1 $$ is the starting value in the sequence that defines [Graham's Number](https://en.wikipedia.org/wiki/Graham%27s_number). [this website](http://www.logique.jussieu.fr/~michel/ha.html) contains a nice list of TMs with various permutations of states and symbols.


https://catonmat.net/busy-beaver


## my machine

implementation of the TM in haskell with visualization using perl.
