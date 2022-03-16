# Group

## Definition

### Definition of function

\begin{align}
Function(A, B) =& \{f \in P(A \times B) \vert \phi_1(f), \phi_2(f) \}, \\
\phi_1(f) = & (\forall a \in A \exists b \in B ((a, b) \in f) ), \\
\phi_2(f) =& (\forall a \in A\forall b_1, b_2 \in B ((a, b_1) \in f \land (a, b_2) \in f \to b_1 = b_2)).
\end{align}

### Definition of semigroup on set $S$

$$
\begin{align}
SemiGroup(S) =&  \{f \in Function(S\times S, S) \vert \phi(f)\},\\
\phi(f) =& \forall a, b, c, m, n, x, y \in f
(\{(a, b, m), (m, c, x), (b, c, n), (a, n, y)\}\subset f \to x = y). \\
p.s.& f(f(a, b) ,c) = f(a, f(b, c)).
\end{align}
$$

### Definition of left-monoid on set $S$ and left-identity

$$
\begin{align}
LeftIdentity(f) = \{e_L \in S \vert \forall a \in S((e_L, a, a)\in f)\}.
\end{align}
$$

$$
\begin{align}
LeftMonoid(S) =& \{f \in SemiGroup(S) \vert LeftIdentity(f) \ne \varnothing\}.
\end{align}
$$

### Definition of right-monoid on set $S$ and right-identity

$$
\begin{align}
RightIdentity(f) = \{e_R \in S \vert \forall a \in S((a, e_R, a)\in f)\}.
\end{align}
$$

$$
\begin{align}
RightMonoid(S) =& \{f \in SemiGroup(S) \vert RightIdentity(f) \ne \varnothing\}.
\end{align}
$$

### Definition of left-group on set $S$ and left-inverse

$$
\begin{align}
LeftInverse(f) =& \{g_L \in P(S \times S) \vert \phi_1(f, g_L), \phi_2(f, g_L) \}, \\
\phi_1(f, g_L) = & \forall a \in S \exists b \in S ((a, b) \in g_L) \\
\phi_2(f, g_L) =& \forall a,b \in S ((a, b) \in g_L \to f(b, a) \in LeftIdentity(f)).
\end{align}
$$

$$
\begin{align}
LeftGroup(S) =& \{f \in LeftMonoid(S) \vert LeftInverse(f) \ne \varnothing\}.
\end{align}
$$

### Definition of right-group on set $S$ and right-inverse

$$
\begin{align}
RightInverse(f) =& \{g_R \in P(S \times S) \vert \phi_1(f, g_R), \phi_2(f, g_R) \}, \\
\phi_1(f, g_R) =& \forall a \in S \exists b \in S ((a, b) \in g_R), \\
\phi_2(f, g_R) =& \forall a, b \in S ((a, b) \in g_R \to f(a, b) \in RightIdentity(f)).
\end{align}
$$

$$
\begin{align}
RightGroup(S) =& \{f \in RightMonoid(S) \vert RightInverse(f) \ne \varnothing\}.
\end{align}
$$

### Definition of group on set $S$

$$
Group(S) = LeftGroup(S) \cap RightGroup(S)
$$

---

## Proposition

### Proposition1

$$
\begin{align}
\forall f \in Group(S) \\
\forall e_{L1}, e_{L2} \in LeftIdentity(f) \\
\forall e_{R1}, e_{R2} \in RightIdentity(f)\\
(e_{L1} = e_{L2} = e_{R1} = e_{R2})
\end{align}
$$

Proof.
$$
\begin{align}
(7) \Rightarrow& (e_{L1}, e_{R1}, e_L) \in f \\
(9) \Rightarrow& (e_{L1}, e_{R1}, e_R) \in f \\
(3) \Rightarrow& e_{L1} = e_{R1} \\
Similarity \Rightarrow& e_{L2} = e_{R1} \land e_{L1} = e_{R2} \\
(25), (26) \Rightarrow& e_{L1} = e_{L2} = e_{R1} = e_{R2}
\end{align}
$$

P.S. Now, we can use $e$ safely.

### Proposition2

$$
\begin{align}
\forall f\in Group(f) \\
\forall g_L \in LeftInverse(f)
\forall (a, b_{L1}), (a, b_{L2}) \in g_L (b_{L1} = b_{L2})\\
\forall g_R \in RightInverse(f)
\forall (a, b_{R1}), (a, b_{R2}) \in g_L (b_{R1} = b_{R2})\\
(b_{L1} = b_{L2} = b_{R1} = b_{R2})
\end{align}
$$

Proof.
$$
\begin{align}
(11) \Rightarrow& f(b_{R1}, a), f(b_{R2}, a) \in LeftIdentity(f) \\
(22) \Rightarrow& f(b_{R1}, a) = f(b_{R2}, a) = e \\
(9) \Rightarrow& \forall g_R \in RightInverse(f) \forall b_{R1} \in g_R(b) \\
& (f(f(b_{L1}, a), b_{R1}) = f(f(b_{L2}, a), b_{R1}) = f(e, b_{R1}) = b_{R1}) \\
(6) \Rightarrow& f(b_{L1}, f(a, b_{R1})) = f(b_{L2}, f(a, b_{R1})) = b_{R1} \\
(9)(15) \Rightarrow& b_{L1} = f(b_{L1}, f(a, b_{R1})), b_{L2} = f(b_{L2}, f(a, b_{R1})) \\
(36) \Rightarrow& b_{L1} = b_{L2}=b_{R1} \\
Similarity \Rightarrow& b_{L1} = b_{L2} = b_{R2}
\end{align}
$$

P.S. Now we can use function $g(a)$ safely.

### Proposition3

$$
\begin{align}
\forall f \in Group(S) \forall a \in S (g(g(a)) = a)
\end{align}
$$

Proof.
$$
\begin{align}
Definition \Rightarrow& f(g(g(a)), g(a)) = e = f(a, g(a))\\
\to& f(f(g(g(a)), g(a)), a) = f(f(a, g(a)), a)\\
\to& f(g(g(a)), f(g(a), a)) = f(a, f(g(a), a))\\
\to& f(g(g(a)), e) = f(a, e)\\
\to& g(g(a)) = a
\end{align}
$$

### Proposition4

$$
\begin{align}
g(f(a, b)) = f(g(b), g(a))
\end{align}
$$

Proof.
$$
\begin{align}
A =& g(f(a, b)) \\
B =& f(g(b), g(a))\\
e =& f(A, f(a, b))\\
B=& f(e, B)\\
=& f(f(A, f(a, b)), B)\\
=& f(A, f(f(a, b), B))\\
=& f(A, f(a, f(b, B)))\\
=& f(A, f(a, f(f(b, g(b)), g(a))))\\
=& f(A, f(a, f(e, g(a))))\\
=& f(A, f(a, g(a)))\\
=& f(A, e) = A
\end{align}
$$

### Proposition5

$$
\begin{align}
\forall f \in LeftGroup(S)\forall c \in S ((c, c, c) \in f \to c = e).
\end{align}
$$

Proof.
$$
\begin{align}
(Condition) \Rightarrow& LeftInverse(f) \ne \varnothing, (13), (6), (7)\\
(13) \Rightarrow& \forall g_L \in LeftInverse(f) \forall b \in g_L(c) (f(c, b) \in LeftIdentity) \\
(Condition) \Rightarrow& \forall g_L \in LeftInverse(f) \forall b \in g_L(c) (f(f(c, c), b) \in LeftIdentity) \\
(6) \Rightarrow& \forall g_L \in LeftInverse(f) \forall b \in g_L(c) (f(c, f(c, b)) \in LeftIdentity)\\
(60)(7) \Rightarrow& \forall g_L \in LeftInverse(f) \forall b \in g_L(c) (c \in LeftIdentity)
\end{align}
$$

### Proposition $LeftGroup(S) = RightGroup(S)$

Proof.
$$
\begin{align}
(13)\Rightarrow&\forall f \in LeftGroup(S) \forall g_L \in LeftInverse(f)
\forall a, b \in S\forall e_L \in LeftIdentity(f) \\
&(a, b) \in g_L \to (b, a, e_L) \in f \\
(6)\Rightarrow& f(f(a, b), f(a, b))
=f(a, f(b, f(a, b)))
= f(a, f(f(b, a), b))
= f(a, f(e_L, b))
= f(a, b)\\
(66)(58) \Rightarrow& f(a, b) = e_L\\
(65)(67) \Rightarrow& (a, b) \in g_L \to (a, b, e_L) \in f\\
\Rightarrow& g_L \in RightInverse(f)\\
\Rightarrow& RightInverse(f) \ne \varnothing \\
(69)\Rightarrow& \forall e_L \in LeftIdentity(f) \forall a \in S((e_L, a, a) \in f \land \phi) \\
& \phi = \exists b \in S ((b, a, e_L) \in f \land (a, b, e_L) \in f)\\
& f(a, e_L) = f(a, f(b, a)) = f(f(a, b), a) = f(e_L, a) = a\\
\Rightarrow& e_L \in RightInverse(f)\\
\Rightarrow& RightInverse(f) \ne \varnothing
\end{align}
$$

### Proposition6

$$
\begin{align}
Group(S) = \{f \in SemiGroup(S) \vert \phi_1(f), \phi_2(f)\},\\
\phi_1(f) = (\{x \in S \vert \forall a, b \in S, (a, x, b) \in f\} \ne \varnothing),\\
\phi_1(f) = (\{y \in S \vert \forall a, b \in S, (y, a, b) \in f\} \ne \varnothing).
\end{align}
$$

Proof.
$$
\begin{align}
(Condition) \Rightarrow& \forall a, b \in S, \exists y_a, x_a,\\
    &f(y_a, a) = a, f(a, x_a) = b \\
(80) \Rightarrow& f(f(y_a, a), x_a) = f(a, x_a) = b \\
(6),(81) \Rightarrow& f(y_a, f(a, x_a)) = b \\
(80) \Rightarrow& f(y_a, b) = b \\
(83)(7) \Rightarrow& y_a \in LeftIdentity(f), LeftIdentity(f) \ne \varnothing \\
(78) \Rightarrow& \forall c \in S \exists d \in S ((d, c, y_a) \in f) \\
(85) \Rightarrow& LeftInverse(f) \ne \varnothing \\
(84)(86)(14)(63.5) \Rightarrow& \{f \in SemiGroup(S) \vert \phi_1(f), \phi_2(f)\} = LeftGroup(S) = Group(S)
\end{align}
$$

### Definition of equivalence relation

$$
\begin{align}
EqRelation(S) =& \{r \in P(S \times S) \vert \phi_1(r), \phi_2(r), \phi_3(r)\},\\
\phi_1(r) =& \forall a \in S((a, a) \in R),\\
\phi_2(r) =& \forall a, b \in S((a, b) \in R \to (b, a) \in R),\\
\phi_3(r) =& \forall a, b, c \in S((a, b) \in R \lang (b, c) \in R \to (a, c) \in R).
\end{align}
$$

Also define
$$
\begin{align}
\forall r \in Relation(S) \forall a \in S(r(a) = \{b \in S \vert (a, b) \in r\}).
\end{align}
$$

### Proposition7

$$
\begin{align}
\forall r \in EqRelation(S) \forall a, b \in S ((a, b) \in r \leftrightarrow r(a) = r(b)).
\end{align}
$$

Proof.
$$
\begin{align}
(89) \Rightarrow& b \in r(b)\\
r(b) = r(a), Axiom Extension \Rightarrow& b \in r(a)\\
(92) \Rightarrow& (a, b) \in r\\
(92) \Rightarrow& \forall c \in r(a), (a, c) \in r\\
(90) \Rightarrow& (c, a) \in r\\
(98)(96)(91) \Rightarrow& (c, b) \in r, c \in r(b) \\
(99) \Rightarrow& r(a) \subset r(b)\\
Similarty \Rightarrow& r(b) \subset r(a) \\
(100, 101) \Rightarrow& r(a) = r(b)
\end{align}
$$

### Definition of congruence relation

$$
\begin{align}
\forall f \in Monoid(S)\\
Congruence(f) = \{r \in EqRelation \vert \phi(r)\}, \\
\phi(r) = \forall a, b, c, d \in S((a_1, a_2) \in r \land (b_1, b_2) \in r \to (f(a_1, b_1), f(a_2, b_2))\in r).
\end{align}
$$

### Theorem1

Denote

$$
\begin{align}
r(S) = \cup_{a \in S} \{r(a)\} = \{r(a), r(b), r(c), \ldots\}.
\end{align}
$$

$$
\begin{align}
\forall f \in Monoid(S)
\forall r \in Congruence(f) \\
\{f_r \in P(r(S) \times r(S) \times r(S)) \vert \forall a, b \in S((r(a), r(b), r(f(a, b)) \in f_r)\}
\subset Monoid(r(S)).
\end{align}
$$

Proof.
$$
\begin{align}
(93) \Rightarrow& \forall a_1, a_2 \in S (r(a_1) = r(a_2) \rightarrow (a_1, a_2) \in r)\\
(93) \Rightarrow& \forall b_1, b_2 \in S (r(b_1) = r(b_2) \rightarrow (b_1, b_2) \in r)\\
(109)(110)(105) \Rightarrow& (f(a_1, b_1), f(a_2, b_2)) \in r \\
(93) \Rightarrow& r(f(a_1, b_1)) = r(f(a_2, b_2)) \\
(3)\Rightarrow& f_r \in Function(r(S) \times r(S), r(S)) \\

(108) \Rightarrow& f_r(f_r(r(a), r(b)), r(c)) = f_r(r(f(a, b)), r(c)) = r(f(f(a, b), c)) \\
(108) \Rightarrow& f_r(r(a), f_r(r(b), r(c))) = f_r(r(a), r(f(b, c))) = r(f(a, f(b, c))) \\
(6) \Rightarrow& f_r(f_r(r(a), r(b)), r(c)) = f_r(r(a), f_r(r(b), r(c)))\\
(113)(116) \Rightarrow& f_r \in SemiGroup(S) \\
(7) \Rightarrow& \exists e \in S \forall a \in S (f_r(r(e), r(a)) = r(f(e, a)) = r(a))\\
\Rightarrow& r(e) \in LeftIdentity(f_r) \\
(117)(118)(7) \Rightarrow& f_r \in LeftMonoid(S) \\
(14) \Rightarrow& (f(g_L(a), a) = e) \\
\Rightarrow& f_r(r(g_L(a)), r(a)) = r(f(g_L(a), a)) = r(e) \in LeftIdentity(f_r)\\
\Rightarrow& r(g_L(a)) \in LeftInverse(r(a)) \\
\Rightarrow& f_r \in LeftGroup(S)
\end{align}
$$
