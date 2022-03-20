# Logic

## An Axiom System for the Propositional Calculus

Denote some language form set

$$
\begin{align}
Form_L
\end{align}
$$

Definition

$$
\begin{align}
PC(Axiom) = PropositionalCalculus = G(Axiom = Axiom_1 \cup Axiom_2 \cup Axiom3, MP),\\
Axiom_1 = \{h_2(a, h_2(b, a)) \vert a, b \in Form_L\},\\
Axiom_2 = \{h_2(h_2(a, h_2(b, c)), h_2(h_2(a, b), h_2(a, c))) \vert a, b, c \in Form_L\},\\
Axiom_3 = \{ h_2(h_2(h_1(a), h_1(b)), h_2(h_2(h_1(a), b), a)) \vert a, b \in Form_L \},\\
MP(x, h_2(x, y)) = y,\quad x, y \in PC.
\end{align}
$$

Definition

$$
\begin{align}
h_{\land}(a, b) =& h_1(h_2(a, h_1(b))), \\
h_{\lor}(a, b) =& h_2(h_1(a), b), \\
h_{\leftrightarrow}(a, b) =& h_{\land}(h_2(a, b), h_2(b, a)).
\end{align}
$$

Q1:

$$
\begin{align}
\{h_2(a, a) \vert a \in Form_L\} \subset PC.
\end{align}
$$

Define:

$$
\begin{align}
Axiom_{Q1} = \{h_2(a, a) \vert a \in Form_L\}.
\end{align}
$$

Proof:

$$
\begin{align}
&\forall a \in Form_L, h_2(a, a) \in Form_L,\\
&MP(Axiom_1(a, a), MP(Axiom_1(a, h_2(a, a)), Axiom_2(a, h(a, a), a))) \\
=& MP(Axiom_1(a, a), MP(Axiom_1(a, h_2(a, a)), h_2(Axiom_1(a, h_2(a, a)), h_2(Axiom_1(a, a), h_2(a, a))))) \\
=& MP(Axiom_1(a, a), h_2(Axiom_1(a, a), h_2(a, a)))\\
=&h_2(a, a) \in PC.
\end{align}
$$

Q2:

$$
\begin{align}
\{Target = h_2(h_2(h_1(a), a), a) \vert a \in Form_L\} \subset PC
\end{align}
$$

Proof:

$$
\begin{align}
&\forall a \in Form_L, h_1(a) \in Form_L, h_2(h_1(a), h_1(a)) \in PC\\
&MP(h_2(h_1(a), h_1(a)), Axiom_3(a, a))\\
=&MP(h_2(h_1(a), h_1(a)), h_2(h_2(h_1(a), h_1(a)), Target)) \\
=& Target
\end{align}
$$

Q3:

$$
\begin{align}
h_2(a, c) \in PC(Axiom \cup \{h_2(a, b), h_2(b, c)\}).
\end{align}
$$

Define

$$
\begin{align}
Q3(h_2(a, b), h_2(b, c)) = h_2(a, c), \forall a, b, c \in PC(Axiom)
\end{align}
$$

Proof:

$$
\begin{align}
&\forall a, b, c \in Form_L,\\
&MP(h_2(a, b), MP(Axiom_1(a, h_2(b, c)), Axiom_2(a, b, c)))\\
=& MP(h_2(a, b), MP(
    h_2(a, h_2(b, c)),
    h_2(h_2(a, b), h_2(a, c))
))\\
=& MP(h_2(a, b), h_2(
    h_2(a, b), h_2(a, c)
)) \\
=& h_2(a, c)
\end{align}
$$

Q4:

$$
\begin{align}
h_2(b, h_2(a, c)) \in PC(Axiom \cup\{h_2(a, h_2(b,c))\})
\end{align}
$$

Proof.

$$
\begin{align}
&X = Axiom_1(a, b) = h_2(b, h_2(a, b))\\
&Y = MP(h_2(a, h_2(b, c)), Axiom_2(a, b, c)) = h_2(h_2(a, b), h_2(a, c))\\
&Q3(X, Y) = h_2(b, h_2(a, c))
\end{align}
$$

Q5:

$$
\begin{align}
h_2(h_2(h_1(a), h_1(b)), h_2(b, a)) \in PC(Axiom)
\end{align}
$$

Proof.

$$
\begin{align}
&t0 := h_2(h_1(a), h_1(b))\\
&t1 := Axiom_1(b, h_1(a)) = h_2(b, h_2(h_1(a), b))\\
&t2 := Axiom_3(a, b) = h_2(t0, h_2(h_2(h_1(a), b), a))\\
&t3 := Q4(t2) = h_2(h_2(h_1(a), b), h_2(t0, a)) \\
&t4 := Q3(t1, t3) = h_2(b, h_2(t0, a))\\
&Q4(t4) = h_1(A, h_2(b, a))
\end{align}
$$

Q6:

$$
\begin{align}
\forall a \in Form_L, a \in PC(Axiom \cup b) \Rightarrow h_2(b, a) \in PC(Axiom).
\end{align}
$$

Denote
$$
Q6(a\in PC(Axiom \cup b)) = h_2(b, a) \in PC(Axiom).
$$

Proof.

If $a \in PC(Axiom)$, then

$$
\begin{align}
&MP(a, Axiom_1(a, b)) \\
=& MP(a, h_2(a, h_2(b, a))) \\
=& h_2(b, a) \in PC(Axiom).
\end{align}
$$

If $a \notin PC(Axiom)$, then

$$
\begin{align}
a \in \cup_{n \in N}\{f_n(b)\}, \quad f_0(b) = b.
\end{align}
$$

Step0:

$$
f_0(b) = b, a = b, Q_1(a) = h(a, a) = h(b, a) \in PC(Axiom).
$$

Stepk: assume $\forall c_{k-1} \in \cup_{k-1}\{f_{k-1}(b)\}, h_2(b, c_{k-1}) \in PC(Axiom)$, we need proof $\forall c_k \in \cup_{k}\{f_k(b)\}$.

$$
\begin{align}
&Q3(h_2(b, c_{k-1}), Axiom_1(c_{k-1}, c_k))\\
=&Q3(h_2(b, c_{k-1}), h_2(c_{k-1}, h_2(c_{k-1}, c_k)))\\
=&h_2(b, h_2(c_{k-1}, c_k)) \in PC(Axiom)\\
&MP(h_2(b, h_2(c_{k-1}, c_k)), Axiom_2(b, c_{k-1}, c_k))\\
=& h_2(h_2(b, c_{k-1}), h_2(b, c_{k}))\\
&MP(h_2(b, c_{k-1}), h_2(h_2(b, c_{k-1}), h_2(b, c_{k})))\\
=& h_2(b, c_k) \in PC(Axiom)
\end{align}
$$

Q7:

$$
\begin{align}
\forall a, b, c \in Form_L, h_2(a, c) \in PC(Axiom \cup \{h_2(a, h_2(b, c)), b\})
\end{align}
$$

Proof:

$$
\begin{align}
&MP(b, Q4(h_2(a, h_2(b, c))))\\
=& MP(b, h_2(b, h_2(a, c)))\\
=& h2(a, c)
\end{align}
$$

Q8:

$$
\begin{align}
\forall a \in Form_L, h_2(h_1(h_1(a)), a) \in PC(Axiom).
\end{align}
$$

Proof.

$$
\begin{align}
&t = Q7(Q1(h_1(a)), Axiom_3(a, h_1(a)))\\
=&Q7(h_2(h_1(a), h_1(a)), h_2(h_2(h_1(a), h_1(h_1(a))), h_2(h_2(h_1(a), h_1(a)), a)))\\
=& h_2(h_2(h_1(a), h_1(h_1(a))), a)\\
&Q3(Axiom_1(h_1(a), h_1(h_1(a))), t)\\
=&Q3(h_2(h_1(h_1(a)), h_2(h_1(h_1(a)), h_1(a))),  t) \\
=& h_2(h_1(h_1(a)), a)
\end{align}
$$
