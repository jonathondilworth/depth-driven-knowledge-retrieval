# Depth Driven Knowledge Retrieval

This repository serves for any continuation of work (post-submission) based on the [https://github.com/jonathondilworth/uom-thesis](https://github.com/jonathondilworth/uom-thesis) repo.

**Note:** Any corrections and continuations of exploratory research *(in-part due to my own curiosity, and perhaps culminating in a short-paper)* will be pushed here, as to not affect the integrity of timestamps in [uom-thesis](https://github.com/jonathondilworth/uom-thesis) post-submission (besides a [one-liner modification in the README.md](https://github.com/jonathondilworth/uom-thesis/pull/2/commits/c5a7979cb29db1e8fde0f9baa4f5ca03e314171f)).

## Aims & Objectives

1. Reproduce experimental results in full, documenting any corrections to existing code and/or datasets.

2. Re-evaluate any conclusions and concluding re-marks, provide additional reflectionary remarks.

3. Produce a better methodology for testing downstream BioMedical QA.

4. Expand exploratory methodology, ensuring alignment between heterogeneous curvature of mixed models and underlying ontology data. Specifically, ensure that the heterogeneous curvature does in fact match the structure of the ontology data (after/post-processing). Additonally, consider including reference to a proof or including a lemma that shows quotienting by equivalance over a quasi--or partial--order does retain a polyhierarchy; and does not neccesarily collapse the inferred view to a pure hierarchy. A proof demonstrating this would further motivate the use of heterogeneous curvature *(at least, that is my intuition)*.

5. Perform additional experiments to build upon 4. This will likely include performing experiments regarding the differing initialisation schemes as outlined in §3.3.1: isotropic initialisation, tuning and learning appropriate weightings with backprop from downstream QA (though, will require some differentialable proxy function, as we can't backprop through decoder-only generation, obviously).

6. Any further exploratory investigation that *may, potentially* lead to some novel findings that *could, possibly* form the basis of some publishable work *(take a 'see how it goes' attitude in this respect)*.

7. Then, for my own interest(s): build upon the existing deployment pipeline to allow for automated infrastructure provisioning and deployment to a local proxmox cluster with PCIe passthrough *(this will require looking into the use of Terraform & Ansible for local VM/LXC deployments with Proxmox)*.

## Additional Notes

This repo is, of course, a work in progress. As I won't have **as much** time to work on independent research, progress may be *a little* slower than usual. However, I will try and push updates as frequently as possible.

### License

MIT License