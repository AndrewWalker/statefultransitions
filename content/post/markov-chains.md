+++
Categories = ["math"]
Description = ""
Tags = ["math"]
date = "2015-05-22T09:47:11+10:00"
title = "Markov chains and Directed Graphs"

+++

One of the simplest possible probabilistic concepts is that of a Finite Markov
chain.  The basic principle of a Markov chain is that it describes how
something (a state) changes over time.  In effect, if you start in a state,
after some time, a transition occurs, and state updates.  If this sounds
familiar, it's not surprising, Markov chains share many common features with
directed graphs. 

My interest lies more in their applications to statistical physics (which is a
fancy term for describing study of the motion of lots of little bits of stuff).
For my application, Markov chains have been helpful as a tool for developing
initution into more complex systems.

So what are the similarities between Markov chains and Graphs?

Markov Chains                                 | Graph Theory
-------------                                 | ------------
Markov Chain                                  | Directed Graph with edges labelled with transition probability
State                                         | Vertex
Transition                                    | Edge
$T(i, j) = 0$                                 | No edge from $(i, j)$
$T(i, j) > 0$                                 | Edge from $(i, j)$, with weight $T(i, j)$
Accessible $i \rightarrow j$                  | Reachable
Communicating Classes $(i \leftrightarrow j)$ | Strongly Connected Components
Permutation of a MC                           | Isomorphism
Transpose $T'$                                | Flip edges in graph
Absorbing $T(i,i) = 1$                        | There exists an edge $(s, s)$, and the out degree of $s = 1$



