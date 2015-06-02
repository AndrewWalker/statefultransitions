+++
Categories = ["math"]
Description = "Solving the classic game of snakes and ladders with absorbing Markov chains"
Tags = []
date = "2015-06-02T21:23:37+10:00"
title = "Snakes and Ladders"

+++

One of the simplest non-trivial examples of something interesting to do with
Markov chains (that's pretty easy to visualise and understand) is working out
how many moves (on average) it takes to complete a game of snakes and ladders?
This selection of an example is somewhat intentional because it's the same
example as the wikipedia entry for [absorbing Markov
chains](http://en.wikipedia.org/wiki/Absorbing_Markov_chain), and because it's
just hard enough to actually justify solving the problem properly.

A full description of absorbing Markov chains is available on wikipedia, which
is a near verbatim transcription from Kemeny and Snell's book (also available
online). In short, a (Discrete Time, Finite State) Markov chain is a system
that describes the evolution of probability of going from one state to another
in an instantaneous transition. Transitions between states are modelled using a
stochastic matrix (a matrix where the rows must sum to one).

The wikipedia article on [snakes and
ladders](http://en.wikipedia.org/wiki/Snakes_and_Ladders) quotes the solution
as 39.6, for the Milton Bradley version of Chutes and Ladders. That's a useful
baseline for comparison, because the board layout is easy to find online. It's
also worth noting that the page for [absorbing Markov
chains](http://en.wikipedia.org/wiki/Absorbing_Markov_chain) quotes the
solution as 39.2.

Each of the states in the chain represents a location on the board, and we'll
have one extra state (state 0) that represents the initial state off the board.
That means we can conveniently number those state 0 - 100 (inclusive), where 0
is the initial state off the board, and 100 is the terminal state.

Ignoring (initially) the chutes and ladders, a transition from a state results
in you advancing by the number of squares on a six sided dice.  From any state
$x$, you can reach states $x+1, x+2, ..., x+6$, unless such a transition would
take you off the board (beyond state 100), where the transition is invalid
(which means you stay where you are). The probability of each of these
transitions is 1/6, except for the edge case where the move is invalid - in
those cases a small adjustment is needed to keep the player in that states.


```python
def snakes_and_ladders_matrix():
    n = 101
    A = np.zeros((n, n))
    for i in xrange(6):
        A += np.diag(np.ones(n-1-i),i+1)

    # work out the cells that are reachable for every initial cell 
    b = np.sum(A, axis=1)

    # update the transition matrix to have a self-edge for every missing edge 
    A += np.diag(6 - b)

    # normalize
    A /= 6.
    
    return A

A = snakes_and_ladders_matrix()
```

We can verify that the resulting matrix is a valid (right) stochastic matrix
trivially - and it's always a good idea to enforce constraints like this when
working with Markov Chains.

```python
np.testing.assert_allclose( 1, np.sum(A, axis=1) )
```

So for the game with no snakes or ladders, this is enough to show how many
moves it take to finish the game.  The equation for mean number of steps can be
written in terms of the fundamental matrix $N$ and the identity matrix $I$:

$$
N = (I-Q)^{-1}
$$

The equation is then

$$
t = N \mathbf{1}
$$

Where $\mathbf{1}$ is the column vector of ones. While it's possible to solve
the equation like this, but only just, it's not an effective way to way to use
a computer to solve the equation.  Solutions to linear equations on a computer
are solved in the form: 

$$
A x = b
$$

But our unknown is on the wrong side of the equation. Somewhat conveniently,
multiplying through by $I-Q$  gives:

$$
(I-Q) t = \mathbf{1}
$$

This means we can solve the equation:

```python
Q = A[:-1,:-1]
I = np.identity(Q.shape[0])
o = np.ones((Q.shape[0],1))
result = np.linalg.solve(I-Q, o)
print 'mean turns to complete (without snakes and ladders)', result[0, 0]
```

Which gives 33.3, which is close enough to give confidence that we're in the
ballpark when we add the snakes and ladders.


```python
ladders = np.array([
    [1, 38],  [4, 14],  [9, 31],  [21,42],
    [28,84],  [36,44],  [51,67],  [71,91],
    [80,100]
]) 

snakes = np.array([
    [98, 78],  [95, 75],  [93, 73],  [87, 24],
    [64, 60],  [62, 19],  [56, 53],  [49, 11],
    [48, 26],  [16, 6],
]) 

def snakes_and_ladders_matrix():
    n = 101
    A = np.zeros((n, n))
    for i in xrange(6):
        A += np.diag(np.ones(n-1-i),i+1)

    # work out the cells that are reachable for every initial cell 
    b = np.sum(A, axis=1)

    # update the transition matrix to have a self-edge for every missing edge 
    A += np.diag(6 - b)

    edges = np.vstack([ladders, snakes])

    for src, dst in edges:
        b = A[:, src].copy()
        A[:, dst] += b
        A[:, src] = 0
    
    A /= 6.
    return A


A = snakes_and_ladders_matrix()
assert_allclose( 1, np.sum(A, axis=1) )
```

And running the code gives the solution

```python
Q = A[:-1,:-1]
I = np.identity(Q.shape[0])
o = np.ones((Q.shape[0],1))
result = np.linalg.solve(I-Q, o)
print 'mean turns to complete', result[0, 0]
```

Gives 39.598
