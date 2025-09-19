# Depth Driven Knowledge Retrieval

This repository serves for any continuation of work (post-submission) based on the [https://github.com/jonathondilworth/uom-thesis](https://github.com/jonathondilworth/uom-thesis) repo.

**Note:** Any corrections and continuations of exploratory research *(in part due to my own curiosity, and perhaps culminating in a short paper)* will be pushed here, as to not affect the integrity of timestamps in [uom-thesis](https://github.com/jonathondilworth/uom-thesis) post-submission (besides a [one-liner modification in the README.md](https://github.com/jonathondilworth/uom-thesis/pull/2/commits/c5a7979cb29db1e8fde0f9baa4f5ca03e314171f)).

## Background

This work extends research on hierarchical ontology embeddings (HiT, OnT) for biomedical knowledge retrieval, specifically addressing SNOMED CT.

## Aims & Objectives

1. Reproduce experimental results in full, documenting any corrections to existing code and/or datasets.

2. Re-evaluate any conclusions and concluding remarks, provide additional reflective remarks.

3. Produce a better methodology for testing downstream BioMedical QA. Test this and demonstrate transferability between retrieval gains and downstream QA tasks.

4. Expand exploratory methodology, ensuring alignment between the heterogeneous curvature of mixed models and underlying ontology data. Specifically, ensure that the heterogeneous curvature does in fact match the structure of the ontology data (after/post-processing). Additionally, consider including a reference to a proof or a lemma that demonstrates how quotienting by equivalence over a quasi-or partial-order retains a polyhierarchy and does not necessarily collapse the inferred view to a pure hierarchy. A proof demonstrating this would further motivate the use of heterogeneous curvature *(at least, that is my intuition)*.

5. Perform additional experiments to build upon 4. This will likely include performing experiments regarding the differing initialisation schemes as outlined in §3.3.1: isotropic initialisation, tuning and learning appropriate weightings with backprop from downstream QA (though, it will require some differentiable proxy function, as we can't backprop through decoder-only generation, obviously).

6. Explore extensions that could form the basis for publication, particularly in venues focused on biomedical NLP or knowledge representation *(see how this goes...)*.

7. Then, for my own interest(s): build upon the existing deployment pipeline to allow for automated infrastructure provisioning and deployment to a local Proxmox cluster with PCIe passthrough *(this will require looking into the use of Terraform & Ansible for local VM/LXC deployments with Proxmox)*.

## Roadmap

- **Initial Phase (2-4 weeks):** Reproducing experimental results, dataset reconstruction, and issuing any corrections (if necessary).
- **Extending Existing Work (1-2 month/s):** Improved methodology and experimental implementation, demonstrating transferability in retrieval gains for downstream biomedical QA.
- **Exploratory research (ongoing):** Proof/lemma for inclusion in writing/paper showing that a transitive reduction of the quasi-ordering discussed in aims & objectives (4) does not collapse to pure hierarchy *(this was something I was initially concerned about)*, then move to better understand which model/s and curvature/s might best suit the task, conduct experiments and report results. Develop a framework to allow for easy adoption of this approach.

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

## License

MIT License