## Rough Proof *(Attempt)*

**Note: very much a work in progress here!**

**Strategy:** Define a pure hierarchy and a polyhierarchy. Then, Inferred $\sqsubseteq$ -> Quasi-order (reflexive + transitive) -> Quotient Poset (satisfies antisymmetry, i.e. collapses equiv to single nodes, forming a partial-order) -> Transitive Reduction (removes transitive edges, simplifying the graph) -> Hasse Diagram -> SNOMED CT Example showing that the transitive reduction of the quotient poset retains a poly-hierarchical structure for the given example -> theorem -> cases for which this holds and does not hold (a proof by example/counterexample; though, I need to think through how to show this..).

--

**Pure Hierarchy:**

**Polyhierarchy:**

**Quasi-order:** The $\sqsubseteq$ relation forms a quasi-order [1], that is, $\sqsubseteq$ i. satisfies reflexivity, and ii. satisfies transitivity; note that under OWL semantics the subClassOf relation is both transitive and reflexive, [2]. This is distinct to a partial-order, as antisymmetry does not hold, i.e. $\neg ((C \sqsubseteq D \land D \sqsubseteq C) \iff (C = D)), \quad \forall C,D \in {N_C | N_C \leftarrow \mathcal{O}}$, or $\neg (C = D) \iff (C \sqsubseteq D \land D \sqsubseteq C)$ *(we might note that my formal logic is a little rusty)*.

**Equivalence**: $C \equiv D$ iff $C \sqsubseteq D \land D \sqsubseteq C$, note that $C$ is not neccesarily equal to $D$, but $C$ can be $\equiv$ to D via a bijective mapping between $C^\mathcal{I} \leftrightarrow D^\mathcal{I}$ *(we omit explicitly defining an interpretation for brevity)*.

**Quotient Poset:** Given the $\sqsubseteq$ relation, as defined over some set $S$, denoted $(S, \sqsubseteq)$, the quotient $S/\equiv$ forms a partial-order $[x] \leq [y]$ iff $x \sqsubseteq y$, which is antisymmetric by construction.

**Transitive Reduction:**

**Hasse Diagram:**

--

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