### Depth Driven Knowledge Retrieval with Hyperbolic Bi-Encoders for Biomedical Question Answering in SNOMED CT

This repository serves for any continuation of work (post-submission) based on the [https://github.com/jonathondilworth/uom-thesis](https://github.com/jonathondilworth/uom-thesis) repo. Any corrections and continuation of research efforts will be pushed here, as to not affect the integrity of timestamps in [uom-thesis](https://github.com/jonathondilworth/uom-thesis).

## Background

This work extends research on effective knowledge retrieval using transformer-based ontology embeddings, i.e. HiT and OnT, applied to biomedical question answering, with a specific focus on SNOMED CT.

**Number of noted corrections (25/09/2025):** 8

**Currently working on:** Re-training HiT-FULL and OnT-FULL on H200 GPU architecture; increased batch size from $32$ to $64$ and $96$, changed epochs from $1$ to $2$; and decreased the learning rate from $1e-5$ to $5e-6$. Spending some time thoughtfully reflecting upon methodological decisions and re-evaluating.

## Aims

1. Reproduce experimental results in full, documenting any corrections to existing code and/or datasets. Note: this should include re-training HiT & OnT models on H200 GPU architecture with 141GB of available VRAM *(in an attempt to overcome issues discussed in Appendix.F, i.e. the effects of ontology size)*.

2. Re-evaluate any conclusions and concluding remarks, provide additional reflective remarks.

3. Produce a better methodology for testing downstream BioMedical QA. Test this and demonstrate transferability between retrieval gains and downstream QA tasks.

4. Expand exploratory methodology, ensuring alignment between the heterogeneous curvature of mixed models and underlying ontology data. Specifically, ensure that the heterogeneous curvature does in fact match the structure of the ontology data (after/post-processing). Additionally, consider including a reference to a proof or a lemma that demonstrates how quotienting by equivalence over a quasi-or partial-order retains a polyhierarchy and does not necessarily collapse the inferred view to a pure hierarchy. A proof demonstrating this would further motivate the use of heterogeneous curvature *(at least, that is my intuition)*.

5. Perform additional experiments to build upon 4. This will likely include performing experiments regarding the differing initialisation schemes as outlined in §3.3.1: isotropic initialisation, tuning and learning appropriate weightings with backprop from downstream QA (though, it will require some differentiable proxy function, as we can't backprop through decoder-only generation, obviously).

6. Explore extensions that could form the basis for publication, particularly in venues focused on biomedical NLP or knowledge representation *(see how this goes...)*.

7. Then, for my own interest(s): build upon the existing deployment pipeline to allow for automated infrastructure provisioning and deployment to a local Proxmox cluster with PCIe passthrough *(this will require looking into the use of Terraform & Ansible for local VM/LXC deployments with Proxmox)*.

## Roadmap

- **Initial Phase (2-4 weeks):** Reproducing experimental results, dataset reconstruction, and issuing any thesis corrections (if necessary). *(Update 22.09.25)* Note that editorial changes to the original thesis are likely required. Details on product manifolds should likely be folded into the preliminaries section, and we can probably collapse the encoder-only paragraphs into a single paragraph *(and possibly trim the introductory materials on Ontology)*.
- **Extending Existing Work (1-2 month/s):** Improved methodology and experimental implementation, demonstrating transferability in retrieval gains for downstream biomedical QA.
- **Exploratory research (ongoing):** A proof for inclusion in writing/paper showing that: $\sqsubseteq$ forms a quasi-order; then, the quotient poset $\rightarrow$ forms a partial order $\rightarrow$ allows for transitive reduction $\rightarrow$ the resulting Hasse diagram *(this is demonstrated through the example in Preliminaries, but a proof that further shows the retainment of poly-hierarchy/DAG ought to strengthen the motivation behind the use of mixed model spaces*); i.e. show that (4) does not collapse to pure hierarchy *(as I was initially concerned about this; however, having worked though multiple examples, this can be shown formally)*, then move to better understand which model/s and curvature/s might best suit the task *(spherical + hierarchical + euclidean)*, conduct experiments and report results. Develop a framework to allow for easy adoption of this approach.

## Checklist

* ~~Dataset reconstruction~~
* Model re-training
* Reproducing experimental results in full (from start to finish, including model re-training)
* Re-write sections and sub-sections: discussion of results, conclusions.
* Editorial changes to thesis
* Extending the existing work: demonstrate performance transferability to downstream QA, consider any additional potential for exploratory research
* Re-release work
* Prepare short-form paper

## Repository Structure

```
.
├── corrections             # documented corrections from uom-thesis
├── data                    # .gitignore'd data dir, use of dvc/git-lfs should be supported
├── deployment              # terraform, ansible and automated deployment scripts
├── docs                    # additional documentation
├── experiments             # code for reproducing experimental results
├── lib                     # self-maintained vendor forks
├── LICENSE                 # MIT License
├── notebooks               # .ipynb notebooks for additional intuition
├── proofs                  # any proofs to support continued writing(s)
├── README.md               # this file
└── src                     # source code to support continued research efforts
 └── depth-driven-knowledge-retrieval
```

## Additional Notes

This repo is, of course, a work in progress. As I won't have **as much** time to work on independent research, progress may be *a little* slower than usual. However, I will try and push updates as frequently as possible.

## Noted Issues and Accompanying Corrections

* Poorly written abstract in an attempt to save word count.
    * Fix: Replace the abstract with the original version, see [abstract.tex](corrections/thesis/abstract.tex).
* Inconsistent (or undefined) notation, e.g. using $Ret_X(q)_k$ when the $top-k$ prefix is undefined.
    * Fix: Add "In instances where only the top-k candidates are selected, the ranked list is written $Ret_X(q)_{1:k}$ for the top-k prefix." under Problem Definition §3.1.
* Introduction of free variables that ought to be bound, e.g. nDCG should use $dist(C^\star, D)$, not $dist(C,D)$, as C is unbound.
    * Fix: Replace $dist(C,D)$ with $dist(C^\star, D)$, under §4.5.
* Documented evaluation metric (nDCG) provides good intuition, but does not fully reflect implementation (one-to-one); not a huge issue as it would technically yield the same results, but principally, should be addressed (the footnote does at least draw attention to this).
* nDCG also defines $dist(C^\star, D) = 0 \iff C^\star = D$; however, this is misleading (and technically inaccurate), since the quotient poset detailed under §3.1. means that any equivalence relations are unaccounted for in this description.
    * Fix: Replace with $dist(C^\star, D) = 0 \iff C^\star \equiv D$; and we might **explicitly** note that this does, in fact, induce a partial-order (a requisite for the subsequent transitive reduction and resulting Hasse Diagram; the way in which it is currently written under §3.1. implies this, though **it really should be made explicit**; and this, likely, belongs within a *larger proof*). The *(Work In Progress)* Proof [is available here.](./proofs/polyhierarchy.md)
* HiT dataset construction **likely** requires the use of a reasoner as part of the build process, i.e. passing `elk` as an argument for `reasoner_type` when loading the ontology prior to instanciating the `HierarchyDatasetConstructor` (see `./scripts/load_taxonomy.py`, line 57-58). The characterisation of "using the stated view" to train the encoders *(as written in the thesis)* is accurate in the case of OnT, since reasoning is an implicit operation during ELNormalisation, i.e $\sqsubseteq_{st}^{\ast} \rightarrow$ gets atomic concepts $\rightarrow$ gets role dependencies $\rightarrow$ applies EL normalisation $\rightarrow$ reflexive transitive closure $\rightarrow$ verbalisation. However, the build scripts as originally shipped set `reasoner_type` to `struct` when building the HiT dataset for training.
    * Fix: Modify the `./scripts/build_hit_data.sh` line 30 to line 31, and add `--infer` (which tells the HiT Dataset Constructor to use `elk` rather than `struct`) and *perhaps* add a footnote to the thesis *(it should likely be documented somewhere, it took me a while to figure this out)*.
* The shipped evaluation dataset appears to contain **some** instances where $\equiv$ classes had not properly been inferred prior to constructing their ancestor sets, resulting in a shallow hierarchy for a minority of cases. However, the build scripts that were shipped do function as intended, and as such the dataset can be accurately re-constructed using the provided scripts.
    * Fix: See the [updated evaluation dataset](./data/evaluation_dataset.json) in [the data directory.](./data).
* The pipeline for OnT-training that has been shipped via `Makefile` (`make ont-data`) fails to apply normalisation to the class labels or role labels (which is required when training on SNOMED~CT due to leakage). Note that the process was undertaken originally during model training, however, it has been failed to be included as part of the full build pipeline *(it will train, it just won't remove high-level semantic branches from verbalisations prior to training)*.
    * Fix: Include [this (`preprocess_ont_dir.py`)](./scripts/preprocess_ont_dir.py) script during the build pipeline after having run `ELNormalizeData.py` with the following parameters `python ./scripts/preprocess_ont_dir.py --base-dir ./data/ont_dataset --strip-parentheses --to-lower-case --collapse-whitespace`.
    * Fix: Modify `./scripts/build_ont_data.sh` to include `conda run -n "$AUTO_ENV_NAME" python ./scripts/preprocess_ont_dir.py --base-dir ./data/ont_dataset --strip-parentheses --to-lower-case --collapse-whitespace` after `ELNormalizeData.py` completes.

## Preliminary Reflective Thoughts

Perhaps the motivation behind exploring mixed model spaces was, *in-part*, misplaced. The Poincaré ball model (hyperbolic space) should be preferable for modelling DAGs, and the complexities introduced through existential restriction (role chains/role inclusion) are captured through the EL normalisation procedure during dataset construction, the associated learning objectives during training, and through role embedding (or complex verbalisation) at inference. However, the conducted experiments did not explicitly use role embeddings, and by simply leveraging class labels, rather than complex verbalisations, the measured knowledge retrieval performance is unlikely to have been optimal.

With that said, since the knowledge retrieval task is evaluated over the inferred view, using model combinations that allow for a mixture of *(approximate)* geometries may be a potential alternative to explicit role embeddings, or when user queries (for embedding and then retrieval) are unlikely to resemble complex concept verbalisations.

*I'll have a bit more of a think about this once I have some additional results.*

## License

MIT License