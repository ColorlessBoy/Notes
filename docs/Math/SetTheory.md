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
