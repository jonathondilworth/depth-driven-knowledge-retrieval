### Depth Driven Knowledge Retrieval with Hyperbolic Bi-Encoders for Biomedical Question Answering in SNOMED CT

This repository serves for any continuation of work (post-submission) based on the [https://github.com/jonathondilworth/uom-thesis](https://github.com/jonathondilworth/uom-thesis) repo. Any corrections and continuation of research efforts will be pushed here, as to not affect the integrity of timestamps in [uom-thesis](https://github.com/jonathondilworth/uom-thesis).

## Background

This work extends research on effective knowledge retrieval using transformer-based ontology embeddings, i.e. HiT and OnT, applied to biomedical question answering, with a specific focus on SNOMED CT.

**Number of noted corrections (25/09/2025):** 9

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

### Model Re-Training

**Latest (New) HiT Model (FULL):**

```
        centri_weight   threshold       precision       recall  f1      accuracy        accuracy_on_negatives
epoch=1.0       1.0     -17.49  0.9171475768089294      0.9067903757095336      0.9119395613670348      0.9840794205665588      0.991808295249939
epoch=2.0       0.9     -19.3   0.9171701073646544      0.9081807732582092      0.9126533269882202      0.984196662902832       0.991798222064972
epoch=3.0       0.9     -20.67  0.9252043962478638      0.92322438955307        0.9242133498191832      0.9862353801727296      0.9925364255905152
epoch=4.0       0.7     -21.76  0.9400038719177246      0.9573206901550292      0.9485833048820496      0.9905654191970824      0.99388986825943
epoch=5.0       0.7     -22.64  0.9401056170463562      0.9621909260749816      0.9510200619697572      0.9909899830818176      0.9938698410987854
epoch=6.0       0.7     -23.08  0.938044011592865       0.9609575271606444      0.9493625164031982      0.9906807541847228      0.9936530590057372
epoch=7.0       0.6     -23.76  0.94743674993515        0.9644322395324708      0.9558589458465576      0.9919024109840392      0.9946494102478028
epoch=8.0       0.7     -24.06  0.9433408975601196      0.9607574343681335      0.9519695043563844      0.9911866188049316      0.99422949552536
epoch=9.0       0.8     -23.65  0.9467164874076844      0.9637661576271056      0.9551652073860168      0.99177485704422        0.9945756793022156
epoch=10.0      0.7     -24.7   0.9469810724258424      0.9609752297401428      0.9539268016815186      0.9915611743927002      0.9946197271347046
epoch=11.0      0.8     -23.82  0.9496983289718628      0.9639611840248108      0.9567765593528748      0.9920822381973268      0.99489426612854
epoch=12.0      0.8     -24.44  0.9459070563316344      0.9616413116455078      0.9537093043327332      0.9915135502815248      0.994500696659088
epoch=13.0      0.8     -24.39  0.9493536949157716      0.9633204340934752      0.956286072731018       0.9919936060905457      0.9948608875274658
epoch=14.0      0.7     -25.42  0.9471782445907592      0.9633558988571168      0.9551985859870912      0.9917847514152528      0.9946275949478148
epoch=15.0      0.7     -25.56  0.949418842792511       0.9654832482337952      0.957383632659912       0.9921860694885254      0.994856297969818
epoch=16.0      0.8     -25.34  0.9488932490348816      0.9648526310920716      0.9568063616752625      0.9920806288719176      0.9948033690452576
epoch=17.0      0.8     -24.98  0.9491025805473328      0.9665317535400392      0.9577378630638124      0.9922454357147216      0.994816780090332
epoch=18.0      0.7     -25.47  0.9512071013450624      0.9629101753234864      0.9570228457450868      0.9921379089355468      0.9950606822967528
epoch=19.0      0.7     -25.68  0.9524450898170472      0.9651692509651184      0.958764910697937       0.9924526810646056      0.9951809644699096
epoch=20.0      0.7     -25.75  0.9506394863128662      0.965316116809845       0.9579216241836548      0.9922903776168824      0.994987726211548
testing 0.7     -25.68  0.9522665739059448      0.9647108912467957      0.9584484100341797      0.99239581823349        0.9951642751693726
```

**Latest (New) OnT Model (FULL):**

* Batch Size: 64
* Learning Rate: 5e-6
* Epochs: 2

*NF1:*

```
H1: 1515, H10: 22885, H100: 49266
MRR: 0.10855280723706726, MR: 2407.127354706022
INFO:hierarchy_transformers.evaluation.ont_eval:Eval results: {'axiom_kind': 'nf1', 'centri_weight': 0.8, 'H@1': np.float64(0.0001469042749144), 'H@10': np.float64(0.5080853852847118), 'H@100': np.float64(0.7676878397161357), 'MRR': np.float64(0.15949874223568358), 'MR': np.float64(798.2874125637056), 'median': np.float64(10.0), 'AUC': np.float64(0.9989579476085229)}
{'eval_axiom_kind': 'nf1', 'eval_centri_weight': 0.8, 'eval_H@1': np.float64(0.0001469042749144), 'eval_H@10': np.float64(0.5080853852847118), 'eval_H@100': np.float64(0.7676878397161357), 'eval_MRR': np.float64(0.15949874223568358), 'eval_MR': np.float64(798.2874125637056), 'eval_median': np.float64(10.0), 'eval_AUC': np.float64(0.9989579476085229), 'eval_runtime': 6422.3004, 'eval_samples_per_second': 0.0, 'eval_steps_per_second': 0.0, 'epoch': 2.0}
```

*NF2:*

```
H1: 18475, H10: 34550, H100: 47120
MRR: 0.4332986059475026, MR: 413.3315862838026
INFO:hierarchy_transformers.evaluation.ont_eval:Eval results nf2: {'axiom_kind': 'nf2', 'centri_weight': 0.8, 'H@1': np.float64(0.343369575318279), 'H@10': np.float64(0.6421336307034662), 'H@100': np.float64(0.8757550413530341), 'MRR': np.float64(0.4332986059475026), 'MR': np.float64(413.3315862838026), 'median': np.float64(5.0), 'AUC': np.float64(0.9994628145414776)}
```

*NF3:*

```
H1: 15599, H10: 22984, H100: 39428
MRR: 0.3009283908001048, MR: 2507.2952005815428
INFO:hierarchy_transformers.evaluation.ont_eval:Eval results nf3: {'axiom_kind': 'nf3', 'centri_weight': 0.8, 'H@1': np.float64(0.26370598279039104), 'H@10': np.float64(0.38855172180616365), 'H@100': np.float64(0.6665426943688402), 'MRR': np.float64(0.3009283908001048), 'MR': np.float64(2507.2952005815428), 'median': np.float64(25.0), 'AUC': np.float64(0.9967203346574399)}
```

*NF4:*

```
H1: 15803, H10: 16654, H100: 17301
MRR: 0.9126695730789114, MR: 14.233130923330117
INFO:hierarchy_transformers.evaluation.ont_eval:Eval results nf4: {'axiom_kind': 'nf4', 'centri_weight': 0.8, 'H@1': np.float64(0.8968276488281028), 'H@10': np.float64(0.9451222972589524), 'H@100': np.float64(0.981839850178764), 'MRR': np.float64(0.9126695730789114), 'MR': np.float64(14.233130923330117), 'median': np.float64(1.0), 'AUC': np.float64(0.9999829002753006)}
```

*COMBINED:*

```
INFO:hierarchy_transformers.evaluation.ont_eval:Combined eval results: {'axiom_kind': 'combined', 'centri_weight': 0.8, 'H@1': 0.19203353168035595, 'H@10': 0.48315718167612215, 'H@100': 0.7016096130308606, 'MRR': np.float64(0.28230296429238527), 'MR': np.float64(3844.0169736811717), 'median': np.float64(12.0), 'AUC': np.float64(0.9949627959787084)}
```

Perhaps a little better? I'm considering up-weighting the NF1 loss relative to NF2 -> NF4. Given that NF1 examples make up the majority of samples, the batch composition should be reasonable without any further changes; and I can't increase the batch size without distributing the load across multiple GPUs (expensive!). LR is already set to anneal linearly; switching to a cyclic LR (with `cycle_momentum=False`) would be simply be a trail and error type approach, but might be worth trying.

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
* The discussion of the HiT/OnT training procedure is inaccurate. HiT/OnT training struggles with $NF1$, not $NF2 \rightarrow NF4$, in fact $NF2 \rightarrow NF4$ performs well, $NF1$ is where the models struggle *(this mistake was likely made by accidently misreading results as reported directly within the console)*.
    * Fix: Correct the `retrieval-notebook.ipynb` and Appendix.F discussion of assumption testing regarding ontology size to reflect the correctly interpreted results *(note that this doesn't change the conslusion, it is simply a mischaracterisation)*.


## Preliminary Reflective Thoughts

Perhaps the motivation behind exploring mixed model spaces was, *in-part*, misplaced. The Poincaré ball model (hyperbolic space) should be preferable for modelling DAGs, and the complexities introduced through existential restriction (role chains/role inclusion) are captured through the EL normalisation procedure during dataset construction, the associated learning objectives during training, and through role embedding (or complex verbalisation) at inference. However, the conducted experiments did not explicitly use role embeddings, and by simply leveraging class labels, rather than complex verbalisations, the measured knowledge retrieval performance is unlikely to have been optimal.

With that said, since the knowledge retrieval task is evaluated over the inferred view, using model combinations that allow for a mixture of *(approximate)* geometries may be a potential alternative to explicit role embeddings, or when user queries (for embedding and then retrieval) are unlikely to resemble complex concept verbalisations.

*I'll have a bit more of a think about this once I have some additional results.*

## License

MIT License