## Rough Proof *(Attempt)*

**Note: very much a work in progress here!**

**Strategy:** Define a pure hierarchy and a polyhierarchy. Then, Inferred $\sqsubseteq$ -> Quasi-order (reflexive + transitive) -> Quotient Poset (satisfies antisymmetry, i.e. collapses equiv to single nodes, forming a partial-order) -> Transitive Reduction (removes transitive edges, simplifying the graph) -> Hasse Diagram -> SNOMED CT Example showing that the transitive reduction of the quotient poset retains a poly-hierarchical structure for the given example -> theorem -> cases for which this holds and does not hold (a proof by example/counterexample; though, I need to think through how to show this..).

**Poly-hierarchy Preservation Under Transitive Reduction**

*Definitions*

Let $(\mathcal{O}, \sqsubseteq)$ be an OWL ontology defined purely in terms of the `subClassOf` relation, with an inferred $\sqsubseteq$ relation.

The inferred $\sqsubseteq$ relation forms a quasi-order, satisfying the reflexive property $\forall C \in N_C : C \sqsubseteq C$ and the transitive property $C \sqsubseteq D \land D \sqsubseteq E \imples C \sqsubseteq E$. Note that a quasi-order does not satisfy antisymmetry.

Equivalence is defined $C \equiv D$ iff $C \sqsubseteq D \land D \sqsubseteq C$.

The quotient $N_C/\equiv$ with order $[C] \leq [D] \iff C \sqsubseteq D$ forms a partial order, which is antisymmetric by construction, yielding a DAG.

Two elements, $a,b$, are incomparable, denoted $a \parallel b$ iff $a \not\leq b \land b \not\leq a$.

*Example*

Consider the following verified SNOMED CT concepts:

$$
X = \mathrm{Viral pneumonia}, 
P = \mathrm{Pneumonia}, 
Q = \mathrm{Viral disease}, 
Z = \mathrm{Disease}
$$

Where $X \sqsubseteq P$, $X \sqsubseteq Q$, such that $P \parallel Q$ and $Q \parallel P$, where both $P$ and $Q$ have a common parent, i.e. $P \sqsubseteq Z$ and $Q \sqsubseteq Z$, shown below.

```
    Z↺
   / \
  P↺  Q↺
   \ /
    X↺
```

After quotienting, we have a *partial-order* $[X] < [P]$ and $[X] < [Q]$ with $[P] \parallel [Q]$.

```
    Z
   / \
  P   Q
   \ /
    X       *(note: edges are directed upwards from X < P < Z and X < Q < Z)*
```

*Lemma: Incomparable Parents Preservation.*

Let $(\mathcal{O}, \leq)$ be an OWL ontology with an inferred $\sqsubseteq$ relation, with a **partial order** (after quotienting). If $X$ has parents $P$ and $Q$, where $P \parallel Q$, then both edges $\langle X,P \rangle$ and $\langle X,Q \rangle$ are preserved in the transitive reduction.

*Proof*

Suppose an edge $\langle X,P \rangle$ is removed through transitive reduction, then $\exists [W] : [X] < [W] < [P]$.

```
    Z
  /  \
 P    Q
 |   /
 W<-X 
 
 *(I really need this visualisation, hah >.< .. I really don't know if this is a worthwhile proof, still it's a good exercise!)*
```

But then either:

1. $[W] < [Q]$ implies $[Q] and [P]$ comparable via $[W]$, *contradiction*.
    * *(contradiction, since there is no way to arrive at both $P$ and $Q$ through $W$, i.e. contradicts $[P] \parallel [Q]$)*.
2. $[Q] < [W]$ implies $[Q] < [W] < [P]$, *contradiction*.
    * *(contradiction, for the same reason, i.e. contradicts $[P] \parallel [Q]$)*.
3. $[W] \parallel [Q]$, then $[X] \rightarrow [Q]$ is not transitive through $[W]$.

$\therefore$ Both edges survive transitive reduction. $\square$


--

*Previous attempts:*

Suppose some concept $X$ has multiple parents $\geq 2$, denoted $P$ and $Q$, where $P$ and $Q$ share a common parent $Z$. If edges $\langle P,Q \rangle$, $\langle Q,P \rangle$ do not exist, then two distinct paths from $X \rightarrow Z$ exist.

```
    Z
   / \
  P   Q
   \ /
    X
```

Now suppose that $P$ and $Q$ are distant ancestors with an arbitrary path length, denoted $\star$, between $X \rightarrow P$ and $X \rightarrow Q$. If $P$ and $Q$ share a common parent $Z$, then at least two paths from $X \rightarrow Z$ exist with a *potentially* variable path length.

```
    Z
   / \
  P   Q
  |   |
  *   *
   \ /
    X
```



**References:**

[1] R. Shearer and I. Horrocks, ‘Exploiting Partial Information in Taxonomy Construction’, in The Semantic Web - ISWC 2009, Springer, Berlin, Heidelberg, 2009, pp. 569–584. doi: 10.1007/978-3-642-04930-9_36.

[2] W3C OWL Working Group, ‘OWL 2 web ontology language document overview (second edition)’. Dec. 2012. [Online]. Available: https://www.w3.org/TR/owl2-overview/

    "Technically, this means that the subclass relationship between classes is transitive. Besides this, it is also reflexive, meaning that every class is its own subclass – this is intuitive as well since clearly, every person is a person etc."
    - OWL2 Primer, [2] W3C OWL Working Group, ‘OWL 2 web ontology language document overview (second edition)’. Dec. 2012. [Online]. Available: https://www.w3.org/TR/owl2-overview/





### Drafts

Consider the set $S = {X, P, Q, Z}$ with the inferred $\sqsubseteq$ relation, $(S, \sqsubseteq)$ such that $X \sqsubseteq P \sqcap Q$, $P \sqcap Q \sqsubseteq Z$ which forms a quasi-order over S.

```
    Z↺
   / \
  P↺  Q↺
   \ /
    X↺
```

*(I need a sufficiently complex example that is still relatively simple... It's actually pretty tough to bring this all together into a single example!)*


### Additional Notes

Additionally, it might be noted that this is not going to be as convinsing (for a good enough motivating factor for mixed model space) for a polyhierarchy that is retained within the same semantic branch, i.e. a DAG contained within the disease branch/fragment of SNOMED CT, for instance. I suspect that the embedding space would still likely capture the structure fairly well, *or at least that is my intuition, I may need to think about this some more...*. What might be better (at some point, we might move on to this later, see how it goes) is if we can show polyhierarchy through role inclusion; a role chain/role inclusion does connect concepts across top-level branches... *I think.* At which point, graph patterns would become substancially more complicated; and OnT does account for this with role embeddings as rotations... Let's think about this later, once I have this initial proof sketched out.