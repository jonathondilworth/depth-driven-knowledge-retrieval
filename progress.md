### Progress

This file outlines the current progress in terms of (i) the number of thesis corrections issued (which can be found under [corrections.md](./corrections/corrections.md)), (ii) recent updates on work currently being undertaken with brief notes on the thought process, and (iii) previously explored techniques *(a small progress log)*. Results regarding training outcomes for HiT (SFT and domain-tuned) and OnT models have been moved to [models.md](./research-notes/models.md) within the [research notes](./research-notes/) directory.

## Corrections

**Number of noted corrections (25/09/2025):** 9

*(see [corrections.md](./corrections/corrections.md))*

## Current Work \w Notes

**Currently working on:** Re-training HiT-FULL and OnT-FULL on H200 GPU architecture; increased batch size from $32$ to $64$ and $96$, changed epochs from $1$ to $2$; and decreased the learning rate from $1e-5$ to $5e-6$. Spending some time thoughtfully reflecting upon methodological decisions and re-evaluating.

HiT Models Trained: 1
OnT Models Trained: 5

Still struggling to break through MRR >= 0.2 for NF1. Currently looking into:

* Batch composition of NF1 -> NF4 with accumulated gradients for an effective batch size of 256 -> 512:
    * Increase relative NF1 samples per batch, decrease NF2/NF3, substancially decrease NF4 representation.
    * Review `ELNormalizeData.py`; allow for sampling of hard negatives relative to the child *(check what the current policy is)*.
    * Also review the data produced via `ELNormalizeData.py`, if many samples have generic root concepts as positive parents, downweight these examples, since really, we need discriminative examples; otherwise we're essentially diluting the training procedure and *dampening* the model.
* Curiculum Learning (Training NF1 first, then NF2/NF3/NF4):
    * Apply varying parameters for centripetal/clustering weight and margin during NF1 vs NF2/NF3/NF4.
* Other potential changes:
    * We might simulate hard negatives (or further reinforce the penalty; by having increased the effective batch size) we might measure $d_\kappa$ between the child and the set of negatives, upweight negatives as a function of geodesic distance (closer -> harder penalty).
    * Potentially limiting the verbalisation length (some verbalisations are fairly huge for SNOMED CT, this may be a contributing factor?).
    * Upweighting NF1 loss, downweighting NF2/NF3/NF4 loss.
    * Try increasing warm-up time.
    * Try increasing number of epochs: $1 \rightarrow 2 \rightarrow 4 \rightarrow 16$.
* If nothing else is working, then perhaps try applying cyclic learning rates that anneal over several epochs *(though, I'm pretty much guessing at this point; perhaps the loss landscape is particularly complex? Though, it may just be the case that the model needs to better disambiguate between easy and hard examples.)*.

**Work in Progress:** ~~Review issues, as noted above; then~~ Update the modifications to `ELNormalizeData.py` to construct an EL normalised hierarchy for `nf1` **only** and re-run the implemented algorithm for hard negative sampling; see if this improves performance during model training.

### Research Log

*(Log entries are sorted descending by date)*

**Note (5/10/25):** Changes to track progress during `ELNormalizeData.py` for selecting *hard negatives* have been implemented. The estimated time for dataset construction is ~48 hours. The approach I've implemented to select hard negatives is too inefficient. *TODO: rework this; let it run for now, and see if the technique can improve NF1 performance (we ought to check that it may be helpful before optimising it; 'pre-mature optimisation being the root of all evil' and whatnot).* Have also implemented entry points in `Makefile` for SFT'ing SBERT and evaluating HiT on custom datasets aligned with [Language Models as Hierarchy Encoders](https://arxiv.org/pdf/2401.11374) *(for convenience and reproducibility)*. Additional model results will be reported shortly.

**Note (4/10/25):** OnT is, *of course*, trained on verbalised atomic and complex concepts; so the hierarchical pre-order is an assumed property deriving from the *(hyperbolic)* contrastive training objectives *(as well as the logical loss, etc)*, rather than having any kind of grounding to the ontologies graph structure; which raises the question: *will sampling negatives, such as cousins and siblings (as hard negatives), have a benefitial effect on improving the training procedure?* We might suppose (and hope) that by taking this approach (constructing associated NF1 graph patterns and sampling 'hard negatives' in their verbalised form), might help the model recognise subtle differences between lexically similar siblings and cousins; it's worth a try, at least!

**Update (4/10/25):** Implemented changes to `ELNormalizeData.py` and `transferID2text.py` in an attempt to simulate hard negatives for verbalised concepts *(rather than randomly sampling 10 negatives)*. However, the implemented graph algorithm for building the EL normalised hierarchy *(which I suspect is fairly huge)* is either incredibly inefficient or has entered into an infinate loop, as it's been running in excess of 36 hours. Either way, I suspect that combining `nf1` with `nf1_org` has potentially introduced a cycle or contains a large number of redundant edges; so I'll need to revisit that and re-write it *(TODO: first simply try constructing the hierarchy over NF1, don't also include `nf1_org` in the edge set)*. However, in the meantime I have trained an OnT model across 20 epochs, and the performance typically doesn't improve for SNOMED CT (FULL) w.r.t. learning NF1 mappings *(intuitively, one might suspect overfitting when training for such a pre-longed period of time; which actually may occur during HiT training \w 20 epochs; though, in actual fact, the model is likely underfitting, see the following sentences)*; which would seem to indicate that *(or, at least.. I suspect)* the contrastive training objective requires more difficult examples *(hard negatives, or some mixture of random and hard negatives)* to better discriminate between concepts within any given local neighbourhood, rather than simply learning agaisnt *global random* negatives *(easy negatives)*. Additionally, I need to check whether the provided negatives account for only direct subsumptions, or whether we consider the transitively closed graph (or, the inferred view) during OnT training *(the fact that we require a hierarchical pre-order, which implies a partial order, for embedding suggests that only direct subsumptions-the covering relation-are valid, I need to check the paper again, actually; then reconsider how best to solve these problems, ideally in as short a time as possible!)*.

**(Update 22.09.25)** Note that editorial changes to the original thesis are likely required. Details on product manifolds should likely be folded into the preliminaries section, and we can probably collapse the encoder-only paragraphs into a single paragraph *(and possibly trim the introductory materials on Ontology)*.