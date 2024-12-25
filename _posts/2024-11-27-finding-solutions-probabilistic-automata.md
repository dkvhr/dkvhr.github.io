---
title: "Finding Solutions for Probabilistic Automata"
date: 2024-11-27 00:00:00 -0300
categories: [linear systems, formal languages]
tags: [linear systems, formal languages]
math: true
mermaid: true
# image: //
#   path: /commons/devices-mockup.png
#   width: 800
#   height: 500
#   alt: Responsive rendering of Chirpy theme on multiple devices.
---

In Formal Languages, we encounter the concept of automata. They essentially describe what is accepted by a regular language. There are deterministic finite automata (DFA) and non-deterministic finite automata (NFA). A common problem in creating automata is minimization. Looking at a DFA, how can we obtain its smallest possible representation, i.e., with the smallest number of states?

## Deterministic Finite Automata

They are defined as a 5-tuple ($\Sigma$, $Q$, ${q_0}$, $F$, $\delta$) where:

$\Sigma$ is the alphabet that describes the language\
$Q$ is the set of states of the automaton\
${q_i}$ is the initial state of the automaton\
$F$ is the set of final states\
$\delta$ is the transition function, which essentially describes what each state will read and which state it will transition to (or itself). Transition functions will be quite important here in this work.

### Example of a DFA

Let's see an example of an automaton described as follows:

$\Sigma$ = {${0, 1}$}\
$Q$ = {${q_0}, {q_1}, {q_2}, {q_3}, {q_4}, {q_5}$}\
${q_i}$ = ${q_0}$\
$F$ = {${q_1}, {q_2}, {q_4}$}\
$\delta$, the transition function, is defined as follows:

| $\delta$ | 0  | 1  |
|----------|----|----|
| q0       | q3 | q1 |
| q1       | q2 | q5 |
| q2       | q2 | q5 |
| q3       | q0 | q4 |
| q4       | q2 | q5 |
| q5       | q5 | q5 |

This means, for example, that from ${q_0}$ we can read a `0` and go to state ${q_3}`, or from ${q_0}$ we can read a `1` and go to state ${q_1}`.

We can build an automaton based on this:

![Automaton_1](/assets/img/automata1.png)

To verify if the constructed automaton accepts a word, we must read the word (traversing the automaton) and end in a final state.

In the automaton depicted, we can see it accepts the word `'010'`, as it ends in the final state ${q_2}$.

![Automaton1_010](/assets/img/automata1_painted.png)

## DFA Minimization

From an automaton, is it possible to generate a smaller automaton that recognizes the same language?

Let’s examine the automaton from the previous example:

![Automaton_1](/assets/img/automata1.png)

This automaton defines which language will be recognized. What we want is to create a smaller automaton that recognizes exactly the same language. In other words, we want:

$L(A) = L(B)$

Where $L(A)$ is the language recognized by DFA $A$ and $L(B)$ is the language recognized by DFA $B$.

### Correctness

Besides preserving the language (i.e., $L(A) = L(B)$), we want to ensure that only equivalent states are "merged" into a single state. Essentially, we want only indistinguishable states with respect to the accepted language to be "merged."

We then need $L(A) \subseteq L(M)$, where $L(M)$ represents the words accepted by the minimized automaton $M$.

Thus, any word accepted by automaton $A$ must also be accepted by the minimized automaton $M$.

Finally, we also want $L(M) \subseteq L(A)$. That is, any word $w$ accepted by $M$ must also be accepted by $A`. Since $M` is created from $A$, preserving the transitions and only merging equivalent states, $M$ cannot accept any word that $A` does not accept.

Combining everything:

If $L(A) \subseteq L(M)$ and $L(M) \subseteq L(A)$, then $L(A) = L(M)$.

### Minimization Examples

When researching DFA minimization, we can find several algorithms. Looking at [this website](https://www.geeksforgeeks.org/minimization-of-dfa/), we see a demonstration of minimization using the automaton shown earlier:

![Automaton_1](/assets/img/automata1.png)

Using the algorithm mentioned there, we find the following corresponding minimized automaton:

![automaton_1_min](/assets/img/minimized_automata1.png)

The major advantage is eliminating redundant states, which improves efficiency in terms of time complexity and space utilization.

## Transition Matrix Representation

For future purposes, it is interesting to pick the transition function we defined previously and turn it into something we can work with and do math.

For that, we will turn the transition function and turn it into a matrix!

Consider the following dictionary for the automata we are using as an example:

```python
transitions = {
    0: {'a': 3, 'b': 1},
    1: {'a': 2, 'b': 5},
    2: {'a': 2, 'b': 5},
    3: {'a': 0, 'b': 4},
    4: {'a': 2, 'b': 5},
    5: {'a': 5, 'b': 5},
}
```

It says that, starting from a state `0`, if we read an `'a'` we will go to state `3` and if we read a `'b'`, we will go to state `1`. If we start from state `1` and read an `'a'`, we will be going to state `2` and if we read a `'b'` we will be going to state `5`. And so on.

Let's get this transition function (defined as a dicitonary on the code) and turn it into a matrix of matrixes. Each matrix inside this bigger matrix will represent the transitions of each state.

Let's see, for example, how to represent the transitions of state `0`:

$$\begin{bmatrix}
0 & 0\\\
0 & 1\\\
0 & 0\\\
1 & 0\\\
0 & 0\\\
0 & 0\\
\end{bmatrix}$$

This will be the first matrix within the larger matrix and indicates the state `0`. The first row indicates the transitions to the state `0`, the second indicates the transitions to the state `1`, and so on. The first column indicates a reading of `'a'` (or the first letter of the alphabet that was defined) and the second a reading of `'b'`. This means that, when looking at the first row and first column of the first matrix, we are seeing if the state `0` makes transitions to the state `0` from the reading of an `'a'`. Since the value found was `0`, it does not make a transition under these conditions.

Let's go to the second row of the first matrix:

$$\begin{bmatrix}
    \begin{bmatrix}
        0 & 0\\\
        0 & 1\\\
        0 & 0\\\
        1 & 0\\\
        0 & 0\\\
        0 & 0\\
    \end{bmatrix}

    \begin{bmatrix}
        0 & 0\\\
        0 & 0\\\
        1 & 0\\\
        0 & 0\\\
        0 & 0\\\
        0 & 1\\
    \end{bmatrix}

    \begin{bmatrix}
        0 & 0\\\
        0 & 0\\\
        1 & 0\\\
        0 & 0\\\
        0 & 0\\\
        0 & 1\\
    \end{bmatrix}

    \begin{bmatrix}
        1 & 0\\\
        0 & 0\\\
        0 & 0\\\
        0 & 0\\\
        0 & 1\\\
        0 & 0\\
    \end{bmatrix}

    \begin{bmatrix}
        0 & 0\\\
        0 & 0\\\
        1 & 0\\\
        0 & 0\\\
        0 & 0\\\
        0 & 1\\
    \end{bmatrix}

    \begin{bmatrix}
        0 & 0\\\
        0 & 0\\\
        0 & 0\\\
        0 & 0\\\
        0 & 0\\\
        1 & 1\\
    \end{bmatrix}

\end{bmatrix}$$

## QR Decomposition

My question was whether it would be possible to minimize a DFA using QR decomposition. This is part of the reason why we made a transition matrix that way in the previous section.

Performing QR decomposition of the previous transition matrix, we obtain the following matrices:

$Q$ Matrix = 

$$\begin{bmatrix}
    \begin{bmatrix}
        0 & 0\\\
        0 & 1\\\
        0 & 0\\\
        -1 & 0\\\
        0 & 0\\\
        0 & 0\\
    \end{bmatrix}

    \begin{bmatrix}
        0 & 0\\\
        0 & 0\\\
        -1 & 0\\\
        0 & 0\\\
        0 & 0\\\
        0 & -1\\
    \end{bmatrix}

    \begin{bmatrix}
        0 & 0\\\
        0 & 0\\\
        -1 & 0\\\
        0 & 0\\\
        0 & 0\\\
        0 & -1\\
    \end{bmatrix}

    \begin{bmatrix}
        1 & 0\\\
        0 & 0\\\
        0 & 0\\\
        0 & 0\\\
        0 & -1\\\
        0 & 0\\
    \end{bmatrix}

    \begin{bmatrix}
        0 & 0\\\
        0 & 0\\\
        -1 & 0\\\
        0 & 0\\\
        0 & 0\\\
        0 & -1\\
    \end{bmatrix}

    \begin{bmatrix}
        0 & 0\\\
        0 & 1\\\
        0 & 0\\\
        0 & 0\\\
        0 & 0\\\
        -1 & 0\\
    \end{bmatrix}
\end{bmatrix}$$

$R$ Matrix = 

$$\begin{bmatrix}
    \begin{bmatrix}
        -1 & 0\\\
        0 & 1\\
    \end{bmatrix}

    \begin{bmatrix}
        -1 & 0\\\
        0 & -1\\
    \end{bmatrix}

    \begin{bmatrix}
        -1 & 0\\\
        0 & -1\\
    \end{bmatrix}

    \begin{bmatrix}
        1 & 0\\\
        0 & -1\\
    \end{bmatrix}

    \begin{bmatrix}
        -1 & 0\\\
        0 & -1\\
    \end{bmatrix}

    \begin{bmatrix}
        -1 & -1\\\
        0 & 0\\
    \end{bmatrix}
\end{bmatrix}$$

The intuition from here is to look at the equivalent states in the orthogonal matrix Q, since the states will be linearly independent and the process will possibly eliminate some redundancies between the states (causing a minimization)

From there, we can see that in the Q matrix states 1, 2 and 4 are equivalent, while the others remain completely different. Then we put them together and form the new automata:

![automata_1_qr](/assets/img/automata1_minimized_qr.png)

In this we reduced 6 states to 4 states.

However, if we look at the transition matrix by itself, we also see the equivalences between the states. In fact, the QR decomposition is not doing anything very useful for us, as we are just continuing with a 1-step minimization. So I decided to change the problem.

## Changing the problem

From now on we will see probabilistic automata.

The difference between it and the previous one is that, now, we will have a _probability_ of arriving at any other state from a state we are looking at.

![prob_automata1](/assets/img/prob_automata1.png)

Looking at this automaton, we have a 1/2 probability of reading an `'a'` and going to state $q_1$ from $q_0$ and a 1/2 probability of reading a `'b'` and staying in $q_0$.

### Larger example

![prob_automata2](/assets/img/prob_automata2.png)

From this example, we will create a transition matrix as follows:

$$\begin{pmatrix}
    0 & 1/2 & 1/2\\\
    2/3 & 1/3 & 0\\\
    0 & 1/3 & 2/3\\
\end{pmatrix}$$

On closer inspection, this looks very much like a linear dynamic system (and it is!). It reminds me a lot of the PageRank problem.

So how do we find a solution to this system?

We know that we have to start reading the automaton from the initial state (which in this case is $q_0$). So a good guess would be:

$$\begin{pmatrix}
    1\\\
    0\\\
    0\\
\end{pmatrix}$$

From there, we can apply one of the many methods we know to solve systems of this type.

You can use this function in Julia:


```julia
function dynamic_system(A, X0, n)
    Xn = A^n * X0
    return Xn
end
```

This system converges to the matrix:

$$\begin{pmatrix}
    1/4\\\
    3/8\\\
    3/8\\
\end{pmatrix}$$

This means that after a certain time the probability of being in each of these states is defined by this matrix.

However, we will not always find an automaton that will converge on a solution.

In the following automaton, all transitions will have a probability of `1/2`:

![nao_converge](/assets/img/automato_n_converge.png)

Its transition matrix is:

$P$ = $$\begin{pmatrix}
    0 & 1/2 & 0 & 1/2\\\
    1/2 & 0 & 1/2 & 0\\\
    0 & 1/2 & 0 & 1/2\\\
    1/2 & 0 & 1/2 & 0\\
\end{pmatrix}$$

When trying to see if we can find any convergence, it doesn't work. Starting from the initial guess as in the previous example, in iteration 1000 we have:

$$\begin{pmatrix}
    1/2\\\
    0\\\
    1/2\\\
    0\\
\end{pmatrix}$$

While, on the 1001th iteration we get: 
$$\begin{pmatrix}
    0\\\
    1/2\\\
    0\\\
    1/2\\
\end{pmatrix}$$

_(This behavior keeps happening for more iterations)_

## Power Method and the Perron-Frobenius Theorem

An eigenvalue $\lambda$ and its corresponding eigenvector $v$ of $P$ satisfy the equation:

$P$ $v$ = $\lambda$ $v$

When $\lambda = 1$, we will have:

$P$ $v$ = $v$

Which indicates that the eigenvector $v$ does not change under the action of the matrix $P$. Hence we can say that this is the **steady state of the system**

### Perron-Frobenius theorem

The theorem tells us that if a matrix is ​​primitive, then it has a **stationary** distribution and will converge to that distribution.

Except we also just saw that the steady state of the system happens when $\lambda = 1$. Thus, if the dominant eigenvalue = 1, we can say that the matrix converges and, consequently, has a solution.

Let's test this for the previous examples by applying the power method:

```julia
function power(C)
    n,m=size(C)
    v=randn(n,1)
    for i=1:100
        v=C*v
        v=v/norm(v)
    end
    return v, only(v'*C*v)
end
```
For the matrix
$P$ = $$\begin{pmatrix}
    0 & 1/2 & 0 & 1/2\\\
    1/2 & 0 & 1/2 & 0\\\
    0 & 1/2 & 0 & 1/2\\\
    1/2 & 0 & 1/2 & 0\\
\end{pmatrix}$$, which we know it doesn't converge:

```shell
julia> power(B)
([0.677848522815677; 0.2012992302931254; 0.677848522815677; 0.2012992302931254;;], 0.5458015435925113)
```

The dominant eigenvalue is `0.5458015435925113`, which tells us that the system does not converge to its steady state.

Let's now test for the following automaton:

$P$ = $$\begin{pmatrix}
    0 & 1/2 & 1/2\\\
    2/3 & 1/3 & 0\\\
    0 & 1/3 & 2/3\\
\end{pmatrix}$$

```shell
julia> power(A)
([-0.5773502691896257; -0.5773502691896257; -0.5773502691896257;;], 0.9999999999999998)
```

The dominant eigenvalue is precisely `1`, which gives us confirmation that the system converges to a stationary solution, as we saw previously.

# References
[1] S. C. Coutinho, L. M. Schechter. Autômatos, Linguagens Formais e Computabilidade\
[2] [Geeks for Geeks: Minimization of DFA](https://www.geeksforgeeks.org/minimization-of-dfa/)\
[3] [Lecture 8: Probabilistic Finite Automata](https://santoshv.github.io/2019CS4510/L923_scribed.pdf)\
[4] [Perron-Frobenius Theorem](https://people.math.harvard.edu/~knill/teaching/math19b_2011/handouts/lecture34.pdf)\
[5] [Probabilistic automata and Markov chains](https://www.pouly.fr/data/mpri_prob_automata.pdf)