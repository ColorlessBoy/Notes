# Set Theory

## Axiom of Set Theory

### Extensionality

$$
\begin{align}
\forall a \forall b (\forall x(x \in a \leftrightarrow x\in b) \to a = b).
\end{align}
$$

### Empty Set

$$
\begin{align}
\exists a \lnot (\exists x (x \in a))
\end{align}
$$

### Pairing

$$
\begin{align}
\forall a \forall b \exists c \forall x((x = a \land x = b) \leftrightarrow x \in c).
\end{align}
$$

### Power Set

$$
\begin{align}
\forall a \exists b \forall x(x \in b \leftrightarrow (\forall y (y \in x \rightarrow y \in a)))
\end{align}
$$

### Union

$$
\begin{align}
\forall a \exists b \forall x (x \in b \leftrightarrow \exists y(y \in a \land x \in y))
\end{align}
$$

Denote

$$
\begin{align}
b = \cup a
\end{align}
$$

### Infinity Set

$$
\begin{align}
\exists a (\varnothing \in a \land (\forall x (x \in a \to \cup\{x, \{x\}\} \in a)))
\end{align}
$$

### Separation

$$
\begin{align}
\forall p [\forall a \exists b \forall x(x \in b \leftrightarrow x \in a \land \phi(x, p))]
\end{align}
$$

### Replacement

$$
\begin{align}
\forall p
    [\forall m \forall n_1 \forall n_2
    (\phi(m, n_1, p) = \phi(m, n_2, p) \rightarrow n_1 = n_2)]\\
    \rightarrow \forall a \exists b \forall y (y \in b \leftrightarrow \exists x (x \in a \land \phi(x, y, p))).
\end{align}
$$

## Problem

### P1

$$
\begin{align}
\forall a \forall b \forall c \forall d(\phi_1(a, b, c, d) \leftarrow \phi_2(a, b, c, d)), \\
\phi_1(a, b, c, d) = \{\{a\}, \{a, b\}\} = \{\{c\}, \{c, d\}\},\\
\phi_2(a, b, c, d) = \leftrightarrow a=c \land b=d.
\end{align}
$$

Proof.

$$
\begin{align}
\phi_1 \Rightarrow& (a = c \rightarrow \{a\} = \{c\})\\
\phi_1 \Rightarrow& (b = d \rightarrow \{b\} = \{d\})\\
\phi_1 \Rightarrow& (a = c \land b=d \rightarrow \{a, b\} = \{c, d\})\\
\Rightarrow& (\{a\} = \{c\} \land \{b\} = \{d\} \land \{a, b\} = \{c, d\} \rightarrow \{\{a\}, \{a, b\}\} = \{\{c\}, \{c, d\}\})\\
\Rightarrow& (a=c \land b = d \rightarrow\{\{a\}, \{a, b\}\} = \{\{c\}, \{c, d\}\})
\end{align}
$$

---

## Ordinal Number

### Forall Symbols

$$
\begin{align}
    \forall_X x \phi = \forall x (x \in X \land \phi).
\end{align}
$$

### Function

$$
\begin{align}
    Function(X, Y) := \{f \in P(X \times Y) \vert \phi_1(f) \land \phi_2(f)\},\\
    \phi_1(f) = \exists_X x \exists_Y y ((x, y) \in f),\\
    \phi_2(f) = \forall_X x \forall_Y y_1 \forall_Y y_2 (
        (x, y_1) \in f \land (x, y_2) \in f \to y_1 = y_2
    ).
\end{align}
$$

### Surjection

$$
\begin{align}
    Surjection(X, Y) := \{
            f \in Function(X, Y) \vert \phi(f)
        \},
    \phi(f) = \forall_Y y \exist_X x ((x, y) \in f).
\end{align}
$$

### Injection

$$
\begin{align}
    Injection(X, Y) := \{
            f \in Function(X, Y) \vert \phi(f)
        \},\\
    \phi(f) = \forall_X x_1 \forall_X x_2 (
        f(x_1) = f(x_2) \to x_1 = x_2
    ).
\end{align}
$$

### Inverse $f^{-1}$

$$
\begin{align}
    \{f^{-1}\} = Inverse(f) = \{g \in P(Y \times X) \vert \forall_X x \forall_Y y((x, y) \in f \to (y, x) \in g)\}.
\end{align}
$$

### Bijection

$$
\begin{align}
    Bijection(X, Y) := Injection(X, Y) \cap Surjection(X, Y).
\end{align}
$$

### Partial Ordering

$$
\begin{align}
    PartialOrdering(S) := \{
        r \in P(S \times S) \vert \phi_1(r) \land \phi_2(r)
        \},\\
    \phi_1(r) = \forall_S a ((a, a) \notin r),\\
    \phi_2(r) = \forall_S a \forall_S b \forall_S c
    ((a, b) \in r \land (b, c) \in r \rightarrow (a, c) \in r).
\end{align}
$$

### Linear Ordering

$$
\begin{align}
    LinearOrder(S) := \{r \in PartialOrdering(S) \vert \phi(r)\},\\
    \phi(r)=\forall_S a \forall_S b \{
            (a, b) \in r \lor (b, a) \in r \lor a = b
        \}.
\end{align}
$$

### \{Max, Min, Upper, Lower, Greatest, Least, Sup, Inf\} of $X \in P(S)$

$$
\begin{align}
    Max(r, X) := \{
            a \in X \vert \forall_X b ((a, b) \notin r)
        \}.
\end{align}
$$

$$
\begin{align}
    Min(r, X) := \{
            a \in X \vert \forall_X b ((b, a) \notin r)
        \}.
\end{align}
$$

$$
\begin{align}
    Upper(r, X) := \{
            a \in S \vert \forall_X b ((b, a) \in r \lor a = b)
        \}.
\end{align}
$$

$$
\begin{align}
    Lower(r, X) := \{
            a \in S \vert \forall_X b ((a, b) \in r \lor a = b).
        \}.
\end{align}
$$

$$
\begin{align}
    Greatest(r, X) := Uppser(r, X) \cap X.
\end{align}
$$

$$
\begin{align}
    Least(r, X) := Lower(r, X) \cap X.
\end{align}
$$

$$
\begin{align}
    Sup(r, X) := Least(r, Upper(r, X)).
\end{align}
$$

$$
\begin{align}
    Inf(r, X) := Greatest(r, Lower(r, X)).
\end{align}
$$

### OrderPreserving

$$
\begin{align}
    OrderPreserving(S_1, S_2) := \{(r_1, r_2, f) \in \Phi \vert \phi(r_1, r_2, f)\},\\
    \Phi = P(PartialOrder(S_1) \times PartialOrder(S_2) \times Function(S_1, S_2)),\\
    \phi(r_1, r_2, f) = \forall_{S_1} a \forall_{S_2} b ((a, b) \in r_1 \rightarrow (f(a), f(b)) \in r_2).
\end{align}
$$

### Increasing

$$
\begin{align}
    Increasing(S_1, S_2) := OrderPreserving(S_1, S_2) \cap \Phi,\\
    \Phi = P(LinearOrder(S_1) \times LinearOrder(S_2) \times Function(S_1, S_2)).
\end{align}
$$

### Isomorphism

$$
\begin{align}
    Isomorphism(S_1, S_2) := \{(r_1, r_2, f) \in \Phi \vert \phi(r_1, r_2, f)\},\\
    \Phi = OrderPreserving(S_1, S_2) \cap \\
    P(PartialOrder(S_1) \times PartialOrder(S_2) \times Injection(S_1, S_2)),\\
    \phi(r_1, r_2, f) = (r_2, r_1, f^{-1}) \in OrderPreserving(S_2, S_1).
\end{align}
$$

### Automorphism

$$
\begin{align}
    Automorphism(S) = Isomorphism(S, S).
\end{align}
$$

### WellOrdering

$$
\begin{align}
    WellOrdering(S) = \{
            r \in LinearOrder(S) \vert
            \forall_{P(S)} X (X = \varnothing \lor Least(r, X) \ne \varnothing)
        \}.
\end{align}
$$

### Lemma1

$$
\begin{align}
    \forall r_1, r_2 \in WellOrdering(S)
    \forall f \in Function(S, S)\\ (
        (r_1, r_2, f) \in OrderPreserving(S, S) \to
        \forall_S a ((a, f(a)) \in r_1 \lor a = f(a))
    ).
\end{align}
$$

Proof.
$$
\begin{align}
    W :=& \{
            a \in S \vert (f(a), a) \in r_1
        \} \in P(S).\\
    (51) \Rightarrow W =& \varnothing \lor Least(r_1, W) \ne \varnothing\\
    (38) \Rightarrow W=& \varnothing \lor \exist_W a \forall_W b ((a, b) \in r_1 \lor a = b)\\
    (54) \Rightarrow W=& \varnothing \lor \exist_W a \forall_W b ((a, b) \in r_1 \lor a = b) \land (f(a), a) \in r_1\\
    =& \varnothing \lor \exist_W a \forall_W b ((a, b) \in r_1 \land (f(a), a) \in r_1 \lor a = b \land (f(a), a) \in r_1)\\
    =& \varnothing \lor \exist_W a ((a, f(a)) \in r_1 \land (f(a), a) \in r_1 \lor a = f(a) \land (f(a), a) \in r_1)\\
    =& \varnothing \lor \exist_W a ((a, a) \in r_1 \lor (a, a) \in r_1)\\
    (29) W =& \varnothing.
\end{align}
$$

### Identity

$$
\begin{align}
    Identity(S) = \{
            f \in Function(S, S) \vert \forall_S a ((a, a) \in f)
        \}.
\end{align}
$$

### Corollary1

$$
\begin{align}
    \forall r_1, r_2 \in WellOrdering(S) \forall (r_1, r_2, f) \in Automorphism(S) (f = Identity(S)).
\end{align}
$$

Proof.

$$
\begin{align}
    (53) \Rightarrow& \forall (r_1, r_2, f) \in Automorphism(S) \phi.\\
    \phi :=& \forall_S a ((a, f(a)) \in r_1 \lor a = f(a))
    \land
    \forall_S b ((f^{-1}(b), b) \in r_2 \lor b = f(b)). \\
    =& \forall_S a\forall_S b((a, f(a))) \in r_1 \land b = f(b))
    \lor \forall_S a\forall_S b ((f^{-1}(b), b) \in r_2 \lor a = f(a)) \\
    &\lor \forall_S a\forall_S b((a, f(a)) \in r_1 \land (b, f^{-1}(b)) \in r_2)
    \lor \forall_S a \forall_S b(a = f(a) \land b = f(b)).\\
    =& \forall_S a((a, f(a))) \in r_1 \land a = f(a))\\
    &\lor \forall_S a\forall_S b((a, f(a)) \in r_1 \land (b, f^{-1}(b)) \in r_2)\\
    &\lor \forall_S a \forall_S b(a = f(a) \land b = f(b)).\\
    =&\forall_S a ((f^{-1}(a), f(f^{-1}(a))) \in r_1 \land (f(a), f^{-1}(f(a))) \in r_2)
    \lor \forall_S a (a = f(a)).\\
    =& \forall_S a ((a, f(a)) \in r_2 \land (f(a), a) \in r_2)
    \lor \forall_S a (a = f(a))\\
    =& \forall_S a(a = f(a)).
\end{align}
$$

### Corollary2

$$
\begin{align}
    \forall r_1 \in WellOrdering(S_1) \forall r_2 \in WellOrdering(S_2)\\
    \forall(r_1, r_2, f_1), (r_1, r_2, f_2) \in Isomorphism(S_1, S_2)(f_1 = f_2).
\end{align}
$$

Proof

$$
\begin{align}
    (63) \Rightarrow \forall_{S_1} a(f^{-1}_2 \circ f_1 (a) = a) \Rightarrow \forall_{S_1} a (f_1(a) = f_2(a)).
\end{align}
$$

### Initial Segment

$$
\begin{align}
    \forall a \in S \forall r \in WellOrdering(S), InitialSegment(r, a) := \{b \in S \vert (b,a) \in r\} \in P(S).
\end{align}
$$

### Lemma2

$$
\begin{align}
\forall_S a \forall r_0 \in WellOrdering(S)
(
    Isomorphism(S, \Phi_1) \cap \Phi_2 = \varnothing
),\\
\Phi_1 = InitialSegment(r_0, a),\\
\Phi_2 = \{r_0\} \times PartialOrdering(\Phi_1) \times Function(S, \Phi_1).
\end{align}
$$

Proof

$$
\begin{align}
    (53) \Rightarrow& \forall_S a ((a, f(a)) \in r_0 \lor a = f(a))\\
    \Rightarrow& \exist_\Phi b (b = f(a))\\
    \Rightarrow& \exist_S ((f(a) , a) \in r_0). (Controdiction.)
\end{align}
$$

### Theorem1

$$
\begin{align}
    \forall r_1 \in WellOrdering(S_1) \forall r_2 \in WellOrdering(S_2) (
        \phi_1 \lor \phi_2 \lor \phi_3
    ),\\
    \Phi = (\{r_1\} \times \{r_2\} \times Injection(S_1, S_2)),\\
    \phi_1 = Isomorphism(S_1, S_2) \cap_{valid} \Phi \ne \varnothing,\\
    \phi_2 = \exist a \in S_1 (Isomorphism(InitalSegment(S_1, a), S_2) \cap_{valid} \Phi \ne \varnothing,\\
    \phi_3 = \exist b \in S_2 (Isomorphism(S_1, InitalSegment(S_2, b)) \cap_{valid} \Phi \ne \varnothing,\\
\end{align}
$$

Proof.

$$
\begin{align}
    r :=& \{(a, b) \in S_1 \times S_2 \vert \\
    & Isomorphism(InitialSegment(S_1, a), InitalSegment(S_2, b)) \cap_{valid} \Phi \ne \varnothing\}.\\
    (75) \Rightarrow& r \in Function(S_1, S_2) \lor r = \varnothing.\\
    (78) \Rightarrow& \forall a_1, a_2 \in S_1 \forall b \in S_2 ((a_1, b) \in r \land (a_2, b) \in r \to a_1 = a_2)\\
    \Rightarrow& r \in Injection(S_1, S_2) \lor r = \varnothing.\\
\end{align}
$$

---

## Script Talks

### SubSet

$$
\begin{align}
    h_{subset}(a, b) := h_{forall}(x, h_{imply}(h_{belong}(x, a), h_{belong}(x, b))).
\end{align}
$$

### EmptySet Axiom

$$
\begin{align}
    Axiom_{EmptySet}(a, b) = h_{exist}(a, h_{forall}(b, h_{not}(h_{belong}(b, a)))).
\end{align}
$$

需要证明 $h_{subset}(\varnothing, \varnothing)$ 可导出，或者说为True，即

$$
\begin{align}
    h_{imply}(h_{forall}(b, h_{not}(h_{belong}(b, a))), h_{subset}(a, a)).
\end{align}
$$
