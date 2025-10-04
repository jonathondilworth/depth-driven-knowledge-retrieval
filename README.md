### Depth Driven Knowledge Retrieval with Hyperbolic Bi-Encoders for Biomedical Question Answering in SNOMED CT

This repository serves for any continuation of work (post-submission) based on the [https://github.com/jonathondilworth/uom-thesis](https://github.com/jonathondilworth/uom-thesis) repo. Any corrections and continuation of research efforts will be pushed here, as to not affect the integrity of timestamps in [uom-thesis](https://github.com/jonathondilworth/uom-thesis).

## Background

This work extends research on effective knowledge retrieval using transformer-based ontology embeddings, i.e. HiT and OnT, applied to biomedical question answering, with a specific focus on SNOMED CT.

## Progress

Progress is currently logged under [progress.md](./progress.md).

**Currently Working On:** Modifications to `ELNormalizeData.py` and `transferID2text.py` for hard negatives in OnT training *(currently testing changes; will subsequently train and evaluate a new OnT model after training data has been produced)*. I might note that whilst OnT seems to underfit on large ontologies such as SNOMED CT, training HiT over 20 epochs may *(or may not)* be excessive. As such, in the meantime *(whilst I wait for these processes to complete)*, I'm validating that the HiT models are being appropriately trained *(note that for our retrieval tasks, we use mixed training samples, for mixed-hop prediction)*; I just want to double check that build pipeline uses the most appropriate settings/parameters when training the HiT model for use in evaluation.

## Corrections

A list of issued corrections are provided under [corrections.md](./corrections/corrections.md), whereas corrected `.tex` files are located in the [thesis corrections](./corrections/thesis/) directory.

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
    * *(In Progress - Taking a little longer than expected to get this working for large ontologies, random negatives are likely insufficient, see notes in [progress.md](./progress.md))*
* Reproducing experimental results in full (from start to finish, including model re-training)
* Re-write sections and sub-sections: discussion of results, conclusions.
* Editorial changes to thesis
* Extending the existing work: demonstrate performance transferability to downstream QA, consider any additional potential for exploratory research
* Re-release work
* Prepare short-form paper

### Model Re-Training

*(Moved to [progress.md](./progress.md)).*

### Additional TODOs

* Add a `MAX_JVM_RAM` variable to `.env` that allows the build pipeline to assign $>8$ GB of RAM when running scripts like `ELNormalizeData.py` and `DeepOnto` related scripts.

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

## Preliminary Reflective Thoughts

Perhaps the motivation behind exploring mixed model spaces was, *in-part*, misplaced. The Poincaré ball model (hyperbolic space) should be preferable for modelling DAGs, and the complexities introduced through existential restriction (role chains/role inclusion) are captured through the EL normalisation procedure during dataset construction, the associated learning objectives during training, and through role embedding (or complex verbalisation) at inference. However, the conducted experiments did not explicitly use role embeddings, and by simply leveraging class labels, rather than complex verbalisations, the measured knowledge retrieval performance is unlikely to have been optimal.

With that said, since the knowledge retrieval task is evaluated over the inferred view, using model combinations that allow for a mixture of *(approximate)* geometries may be a potential alternative to explicit role embeddings, or when user queries (for embedding and then retrieval) are unlikely to resemble complex concept verbalisations.

*I'll have a bit more of a think about this once I have some additional results.*

## License

MIT License