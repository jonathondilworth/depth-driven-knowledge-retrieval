### Progress

This file outlines the current progress in terms of the number of corrections issues (which can be found under [corrections.md](./corrections/corrections.md)) and provides recent updates on what is currently being worked on. Some results regarding training outcomes for HiT and OnT models *(in an attempt to overcome issues associated with ontology size and existing model training procedures, etc)* are also provided.

**Number of noted corrections (25/09/2025):** 9

**Currently working on:** Re-training HiT-FULL and OnT-FULL on H200 GPU architecture; increased batch size from $32$ to $64$ and $96$, changed epochs from $1$ to $2$; and decreased the learning rate from $1e-5$ to $5e-6$. Spending some time thoughtfully reflecting upon methodological decisions and re-evaluating.

HiT Models Trained: 1
OnT Models Trained: 4

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

**Update (4/10/25):** Implemented changes to `ELNormalizeData.py` and `transferID2text.py` in an attempt to simulate hard negatives for verbalised concepts *(rather than randomly sampling 10 negatives)*. However, the implemented graph algorithm for building the EL normalised hierarchy *(which I suspect is fairly huge)* is either incredibly inefficient or has entered into an infinate loop, as it's been running in excess of 36 hours. Either way, I suspect that combining `nf1` with `nf1_org` has potentially introduced a cycle or contains a large number of redundant edges; so I'll need to revisit that and re-write it *(TODO: first simply try constructing the hierarchy over NF1, don't also include `nf1_org` in the edge set)*. However, in the meantime I have trained an OnT model across 20 epochs, and the performance typically doesn't improve for SNOMED CT (FULL) w.r.t. learning NF1 mappings *(intuitively, one might suspect overfitting when training for such a pre-longed period of time; which actually may occur during HiT training \w 20 epochs; though, in actual fact, the model is likely underfitting, see the following sentences)*; which would seem to indicate that *(or, at least.. I suspect)* the contrastive training objective requires more difficult examples *(hard negatives, or some mixture of random and hard negatives)* to better discriminate between concepts within any given local neighbourhood, rather than simply learning agaisnt *global random* negatives *(easy negatives)*. Additionally, I need to check whether the provided negatives account for only direct subsumptions, or whether we consider the transitively closed graph (or, the inferred view) during OnT training *(the fact that we require a hierarchical pre-order, which implies a partial order, for embedding suggests that only direct subsumptions-the covering relation-are valid, I need to check the paper again, actually; then reconsider how best to solve these problems, ideally in as short a time as possible!)*.

**Note:** OnT is, *of course*, trained on verbalised atomic and complex concepts; so the hierarchical pre-order is an assumed property deriving from the *(hyperbolic)* contrastive training objectives *(as well as the logical loss, etc)*, rather than having any kind of grounding to the ontologies graph structure; which raises the question: *will sampling negatives, such as cousins and siblings (as hard negatives), have a benefitial effect on improving the training procedure?* We might suppose (and hope) that by taking this approach (constructing associated NF1 graph patterns and sampling 'hard negatives' in their verbalised form), might help the model recognise subtle differences between lexically similar siblings and cousins; it's worth a try, at least!

**Next TODO:** *(currently in progress)* Review issues, as noted above; then update the modifications to `ELNormalizeData.py` to construct an EL normalised hierarchy for `nf1` **only** and re-run the implemented algorithm for hard negative sampling; see if this improves performance during model training.

## Model Results and Model Re-training

**Newest/Latest HiT Model (FULL)**

*Experiment Name: HiT-all-MiniLM-L12-v2-hit_dataset-mixed*

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

**OnT Model (FULL) #1**

*Experiment Name: OnTr-all-MiniLM-L12-v2-H200-OnT-F-96-LR-5e-6--epochs-2*

*Parameters:*

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

*(01/10/25) Update:* I've tried training multiple OnT models by modifying the batch size, applying accumulated gradients (for an effective batch size of 512), lowering the LR and training over multiple epochs, switching to cosine annealing; regardless of these changes, the NF1 performance still struggles to exceed an MMR value of 0.2 on the evaluation set when trained on SNOMED~CT FULL. Batch composition may be an issue, so I need to look into that. I also need to review `ELNormalizeData.py` and understand whether we're simply using mixed/random negatives, or applying weighted hard negatives (similarly to HiT; if not, this would likely be advantageous). Failing that, we might apply curriculum learning *(train NF1, then modify parameters and train NF2/NF3 and NF4)*; and adopt a weighting scheme that can penalise incorrectly classified siblings or cousins within the local neighbourhood of candidate $\leftrightarrow$ target pairs *(if it is the case that negatives are uniformly weighted and are sampled at random during training). Beyond that, the only other issue I can think of is excessive verbalisation length (since a lot of the verbalisations for SNOMED CT concepts are actually quite long), which may be adding noise or contributing to a complex loss landscape that is difficult to navigate; perhaps the use of cyclic LR *may be advantageous?* Beyond that, I'd probably just be guessing at what might improve NF1 performance. *Let's revisit this after we've tried some of the aforementioned changes.*

**OnT Model (FULL) #2**

*Experiment Name: OnTr-all-MiniLM-L12-v2-H200-OnT-FULL-64-LR-1e-5-cosine-annealing-2-epochs-grad-acc-8*

See wandb results *(TODO: place results here and add a description)*.

**OnT Model (FULL) #3**

*Experiment Name: OnTr-all-MiniLM-L12-v2-H200-OnT-FULL-64-LR-1e-5-cosine-annealing-2-epochs-grad-acc-8-downweighted-logical-loss*

See wandb results *(TODO: place results here and add a description)*.

**OnT Model (FULL) #4**

*Experiment Name: OnTr-all-MiniLM-L12-v2-H200-OnT-FULL-64-LR-5e-6-cosine-annealing-20-epochs-grad-acc-8-downweighted-logical-loss-075-downweighted-centripetal-loss-085*

*Parameters:*

* Batch Size: 64
* Learning Rate: 5e-6
* Annealing strategy: cosine
* Epochs: 20

```
Batches: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 23835/23835 [02:10<00:00, 182.79it/s]
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:14<00:00, 186.90it/s]
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
computing metrics.......██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▉| 2765/2766 [04:55<00:00,  9.38it/s]
H1: 13, H10: 27268, H100: 59048
MRR: 0.10234491224308012, MR: 404.929632852316
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:14<00:00, 190.44it/s]
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
computing metrics.......██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▉| 2765/2766 [04:55<00:00,  9.37it/s]
H1: 13, H10: 29312, H100: 65302
MRR: 0.1095210187023194, MR: 378.42785305052377
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:14<00:00, 189.09it/s]
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.38it/s]
computing metrics.......██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▉| 2765/2766 [04:54<00:00,  9.37it/s]
H1: 13, H10: 32016, H100: 68854
MRR: 0.1180585318677859, MR: 367.5121874046535
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:14<00:00, 194.74it/s]
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
computing metrics.......██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▉| 2765/2766 [04:55<00:00,  9.37it/s]
H1: 13, H10: 35770, H100: 70514
MRR: 0.12845306203306153, MR: 367.1989422892206
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:14<00:00, 188.56it/s]
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
computing metrics.......██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▉| 2765/2766 [04:54<00:00,  9.37it/s]
H1: 13, H10: 40058, H100: 71157
MRR: 0.1399344270118707, MR: 375.05829839648334
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:14<00:00, 184.83it/s]
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
computing metrics.......██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▉| 2765/2766 [04:55<00:00,  9.37it/s]
H1: 13, H10: 44240, H100: 71305
MRR: 0.15188102587975863, MR: 389.92166612048413
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:14<00:00, 189.95it/s]
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
computing metrics.......██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▉| 2765/2766 [04:55<00:00,  9.37it/s]
H1: 13, H10: 46949, H100: 71142
MRR: 0.16282277180908203, MR: 411.44315369577254
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:14<00:00, 188.38it/s]
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
computing metrics.......██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▉| 2765/2766 [04:55<00:00,  9.37it/s]
H1: 13, H10: 48046, H100: 70759
MRR: 0.17166972210730008, MR: 439.6269761450058
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:14<00:00, 186.26it/s]
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
computing metrics.......██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▉| 2765/2766 [04:55<00:00,  9.37it/s]
H1: 13, H10: 48174, H100: 70150
MRR: 0.17707516417856864, MR: 474.85236120371104
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:14<00:00, 188.61it/s]
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
computing metrics.......██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▉| 2765/2766 [04:55<00:00,  9.37it/s]
H1: 13, H10: 47621, H100: 69396
MRR: 0.17860439042224602, MR: 517.7282722927238
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:14<00:00, 191.68it/s]
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
computing metrics.......██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▉| 2765/2766 [04:55<00:00,  9.37it/s]
H1: 13, H10: 46476, H100: 68391
MRR: 0.1757804676445304, MR: 569.0928322014171
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:14<00:00, 191.40it/s]
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
computing metrics.......██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▉| 2765/2766 [04:55<00:00,  9.37it/s]
H1: 13, H10: 44754, H100: 67195
MRR: 0.16985106306056105, MR: 629.8414789870385
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:14<00:00, 187.55it/s]
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
computing metrics.......██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▉| 2765/2766 [04:55<00:00,  9.37it/s]
H1: 13, H10: 42627, H100: 65865
MRR: 0.16161128402229366, MR: 701.2100731131276
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:14<00:00, 188.83it/s]
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
computing metrics.......██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▉| 2765/2766 [04:55<00:00,  9.37it/s]
H1: 13, H10: 40052, H100: 64214
MRR: 0.15154608164659747, MR: 784.5272055416813
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:14<00:00, 187.51it/s]
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
computing metrics.......██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▉| 2765/2766 [04:55<00:00,  9.37it/s]
H1: 18, H10: 37483, H100: 62489
MRR: 0.1417079827622413, MR: 881.2074627371657
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:14<00:00, 188.88it/s]
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
computing metrics.......██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▉| 2765/2766 [04:55<00:00,  9.37it/s]
H1: 254, H10: 34897, H100: 60436
MRR: 0.13326258605838115, MR: 992.8403489541546
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:14<00:00, 192.09it/s]
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
computing metrics.......██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▉| 2765/2766 [04:55<00:00,  9.37it/s]
H1: 1649, H10: 32516, H100: 58181
MRR: 0.1344812682335521, MR: 1121.0758252065134
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:14<00:00, 187.88it/s]
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
computing metrics.......██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▉| 2765/2766 [04:55<00:00,  9.37it/s]
H1: 3665, H10: 30353, H100: 55699
MRR: 0.1444493588186359, MR: 1267.9140270981886
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:14<00:00, 188.47it/s]
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
computing metrics.......██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▉| 2765/2766 [04:55<00:00,  9.37it/s]
H1: 4625, H10: 28534, H100: 53191
MRR: 0.15039850644644231, MR: 1435.2703716678154
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:14<00:00, 191.05it/s]
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
computing metrics.......██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████▉| 2765/2766 [04:55<00:00,  9.37it/s]
H1: 4318, H10: 26943, H100: 50516
MRR: 0.14791232307688218, MR: 1625.2770501621596
INFO:hierarchy_transformers.evaluation.ont_eval:Eval results: {'axiom_kind': 'nf1', 'centri_weight': 0.9, 'H@1': np.float64(0.0001469042749144), 'H@10': np.float64(0.5381329596691263), 'H@100': np.float64(0.7841976201507463), 'MRR': np.float64(0.17860439042224602), 'MR': np.float64(517.7282722927238), 'median': np.float64(8.0), 'AUC': np.float64(0.999325681870819)}
{'eval_axiom_kind': 'nf1', 'eval_centri_weight': 0.9, 'eval_H@1': np.float64(0.0001469042749144), 'eval_H@10': np.float64(0.5381329596691263), 'eval_H@100': np.float64(0.7841976201507463), 'eval_MRR': np.float64(0.17860439042224602), 'eval_MR': np.float64(517.7282722927238), 'eval_median': np.float64(8.0), 'eval_AUC': np.float64(0.999325681870819), 'eval_runtime': 6406.9, 'eval_samples_per_second': 0.0, 'eval_steps_per_second': 0.0, 'epoch': 20.0}
100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 49720/49720 [69:14:26<00:00,  1.83s/it]INFO:sentence_transformers.trainer:Saving model checkpoint to experiments/OnTr-all-MiniLM-L12-v2-H200-OnT-FULL-64-LR-5e-6-cosine-annealing-20-epochs-grad-acc-8-downweighted-logical-loss-075-downweighted-centripetal-loss-085/checkpoint-49720
INFO:sentence_transformers.SentenceTransformer:Save model to experiments/OnTr-all-MiniLM-L12-v2-H200-OnT-FULL-64-LR-5e-6-cosine-annealing-20-epochs-grad-acc-8-downweighted-logical-loss-075-downweighted-centripetal-loss-085/checkpoint-49720
{'train_runtime': 249270.2901, 'train_samples_per_second': 102.088, 'train_steps_per_second': 0.199, 'train_loss': 0.3647588308166326, 'epoch': 20.0}
100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 49720/49720 [69:14:29<00:00,  5.01s/it]
0.9
Batches: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 23835/23835 [02:13<00:00, 178.11it/s]
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 4039/4039 [00:22<00:00, 180.43it/s]
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 4039/4039 [07:10<00:00,  9.37it/s]
computing metrics.......
H1: 16, H10: 54079, H100: 79937
MRR: 0.13861776705790535, MR: 4793.696158161489
INFO:hierarchy_transformers.evaluation.ont_eval:Eval results nf1: {'axiom_kind': 'nf1', 'centri_weight': 0.9, 'H@1': np.float64(0.00012380547065423452), 'H@10': np.float64(0.4184547529693968), 'H@100': np.float64(0.6185398692304717), 'MRR': np.float64(0.13861776705790535), 'MR': np.float64(4793.696158161489), 'median': np.float64(22.0), 'AUC': np.float64(0.9937179351195513)}
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1682/1682 [00:09<00:00, 172.49it/s]
Evaluating nf2: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1682/1682 [02:59<00:00,  9.37it/s]
computing metrics.......
H1: 16772, H10: 31094, H100: 43871
MRR: 0.383828173300188, MR: 451.98961063098227
INFO:hierarchy_transformers.evaluation.ont_eval:Eval results nf2: {'axiom_kind': 'nf2', 'centri_weight': 0.9, 'H@1': np.float64(0.3117182417990893), 'H@10': np.float64(0.5779016819998142), 'H@100': np.float64(0.8153703187436112), 'MRR': np.float64(0.383828173300188), 'MR': np.float64(451.98961063098227), 'median': np.float64(7.0), 'AUC': np.float64(0.9994127146440184)}
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1849/1849 [00:10<00:00, 178.33it/s]
Evaluating nf3: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1849/1849 [05:55<00:00,  5.20it/s]
computing metrics.......
H1: 15679, H10: 25072, H100: 41698
MRR: 0.3106133001097696, MR: 1844.0187648978074
INFO:hierarchy_transformers.evaluation.ont_eval:Eval results nf3: {'axiom_kind': 'nf3', 'centri_weight': 0.9, 'H@1': np.float64(0.26505840785758966), 'H@10': np.float64(0.4238500160600477), 'H@100': np.float64(0.704917755650601), 'MRR': np.float64(0.3106133001097696), 'MR': np.float64(1844.0187648978074), 'median': np.float64(18.0), 'AUC': np.float64(0.9975898283655649)}
Batches: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 551/551 [00:02<00:00, 184.48it/s]
Evaluating nf4: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 551/551 [00:58<00:00,  9.37it/s]
computing metrics.......
H1: 15939, H10: 17122, H100: 17555
MRR: 0.927333665362395, MR: 4.015095624538902
INFO:hierarchy_transformers.evaluation.ont_eval:Eval results nf4: {'axiom_kind': 'nf4', 'centri_weight': 0.9, 'H@1': np.float64(0.9045457125021281), 'H@10': np.float64(0.9716815163725101), 'H@100': np.float64(0.9962544690993701), 'MRR': np.float64(0.927333665362395), 'MR': np.float64(4.015095624538902), 'median': np.float64(1.0), 'AUC': np.float64(0.9999962448575239)}
INFO:hierarchy_transformers.evaluation.ont_eval:Combined eval results: {'axiom_kind': 'combined', 'centri_weight': 0.9, 'H@1': 0.18631020653236546, 'H@10': 0.490223775470144, 'H@100': 0.7045848183700647, 'MRR': np.float64(0.2820494547980054), 'MR': np.float64(2898.1602838954022), 'median': np.float64(11.0), 'AUC': np.float64(0.9962029086590458)}
INFO:sentence_transformers.SentenceTransformer:Save model to experiments/OnTr-all-MiniLM-L12-v2-H200-OnT-FULL-64-LR-5e-6-cosine-annealing-20-epochs-grad-acc-8-downweighted-logical-loss-075-downweighted-centripetal-loss-085/final
```