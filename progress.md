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

*(7/10/25)* Reconsidering the approach. See git history for prior entries.