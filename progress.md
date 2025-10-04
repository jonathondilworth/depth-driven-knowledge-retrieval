### Progress

This file outlines the current progress in terms of the number of corrections issues (which can be found under [corrections.md](./corrections/corrections.md)) and provides recent updates on what is currently being worked on. Some results regarding training outcomes for HiT and OnT models *(in an attempt to overcome issues associated with ontology size and existing model training procedures, etc)* are also provided.

#### Links to Model Runs:

**SFT**

*(TODO)*

**HiT**

1. [HiT Model (Standard Training Procedure)](#model-results-and-model-re-training)

**OnT**

1. [OnT Model One (Initial Re-Training Attempt)](#ont-model-full-1-initial-re-training-attempt-w-modified-parameters)
2. [OnT Model Two (Exploratory Run \w Cosine Annealing & Accumulated Gradients)](#ont-model-training-run-2-exploratory-run-w-cosine-annealing-accumulated-gradients)
3. 
[OnT Model Three (Exploratory Run \w Cosine Annealing & Accumulated Gradients with Modified Logical Loss)](#ont-model-training-run-3-exploratory-run-w-cosine-annealing-accumulated-gradients--modified-logical-loss)
4. [OnT Model Four (Re-Training Pre-existing HiT Tuned Model)](#ont-model-training-run-4-re-training-hit-tuned-model)
5. [OnT Model Five (Excessive, Pre-longed Training Run \w 20 epochs; accumulated grads & modified loss weights)](#ont-model-training-run-5-pre-longed-training-run-for-ont-on-standard-plm-base-model-w-training-modified-parameters)

## Corrections

**Number of noted corrections (25/09/2025):** 9

*(see [corrections.md](./corrections/corrections.md))*

## Current Work \w Notes

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

## OnT Model (FULL) #1: Initial Re-training Attempt \w Modified Parameters 

*Experiment Name: OnTr-all-MiniLM-L12-v2-H200-OnT-F-96-LR-5e-6--epochs-2*

* Ontology: SNOMED~CT, Pinned: Sept-25 (FULL ONTOLOGY)

*Parameters:*

* Batch Size: 64
* Learning Rate: 5e-6
* Epochs: 2

*NF1:*

```sh
H1: 1515, H10: 22885, H100: 49266
MRR: 0.10855280723706726, MR: 2407.127354706022
INFO:hierarchy_transformers.evaluation.ont_eval:Eval results: {'axiom_kind': 'nf1', 'centri_weight': 0.8, 'H@1': np.float64(0.0001469042749144), 'H@10': np.float64(0.5080853852847118), 'H@100': np.float64(0.7676878397161357), 'MRR': np.float64(0.15949874223568358), 'MR': np.float64(798.2874125637056), 'median': np.float64(10.0), 'AUC': np.float64(0.9989579476085229)}
{'eval_axiom_kind': 'nf1', 'eval_centri_weight': 0.8, 'eval_H@1': np.float64(0.0001469042749144), 'eval_H@10': np.float64(0.5080853852847118), 'eval_H@100': np.float64(0.7676878397161357), 'eval_MRR': np.float64(0.15949874223568358), 'eval_MR': np.float64(798.2874125637056), 'eval_median': np.float64(10.0), 'eval_AUC': np.float64(0.9989579476085229), 'eval_runtime': 6422.3004, 'eval_samples_per_second': 0.0, 'eval_steps_per_second': 0.0, 'epoch': 2.0}
```

*NF2:*

```sh
H1: 18475, H10: 34550, H100: 47120
MRR: 0.4332986059475026, MR: 413.3315862838026
INFO:hierarchy_transformers.evaluation.ont_eval:Eval results nf2: {'axiom_kind': 'nf2', 'centri_weight': 0.8, 'H@1': np.float64(0.343369575318279), 'H@10': np.float64(0.6421336307034662), 'H@100': np.float64(0.8757550413530341), 'MRR': np.float64(0.4332986059475026), 'MR': np.float64(413.3315862838026), 'median': np.float64(5.0), 'AUC': np.float64(0.9994628145414776)}
```

*NF3:*

```sh
H1: 15599, H10: 22984, H100: 39428
MRR: 0.3009283908001048, MR: 2507.2952005815428
INFO:hierarchy_transformers.evaluation.ont_eval:Eval results nf3: {'axiom_kind': 'nf3', 'centri_weight': 0.8, 'H@1': np.float64(0.26370598279039104), 'H@10': np.float64(0.38855172180616365), 'H@100': np.float64(0.6665426943688402), 'MRR': np.float64(0.3009283908001048), 'MR': np.float64(2507.2952005815428), 'median': np.float64(25.0), 'AUC': np.float64(0.9967203346574399)}
```

*NF4:*

```sh
H1: 15803, H10: 16654, H100: 17301
MRR: 0.9126695730789114, MR: 14.233130923330117
INFO:hierarchy_transformers.evaluation.ont_eval:Eval results nf4: {'axiom_kind': 'nf4', 'centri_weight': 0.8, 'H@1': np.float64(0.8968276488281028), 'H@10': np.float64(0.9451222972589524), 'H@100': np.float64(0.981839850178764), 'MRR': np.float64(0.9126695730789114), 'MR': np.float64(14.233130923330117), 'median': np.float64(1.0), 'AUC': np.float64(0.9999829002753006)}
```

*COMBINED:*

```sh
INFO:hierarchy_transformers.evaluation.ont_eval:Combined eval results: {'axiom_kind': 'combined', 'centri_weight': 0.8, 'H@1': 0.19203353168035595, 'H@10': 0.48315718167612215, 'H@100': 0.7016096130308606, 'MRR': np.float64(0.28230296429238527), 'MR': np.float64(3844.0169736811717), 'median': np.float64(12.0), 'AUC': np.float64(0.9949627959787084)}
```

Perhaps a little better? I'm considering up-weighting the NF1 loss relative to NF2 -> NF4. Given that NF1 examples make up the majority of samples, the batch composition should be reasonable without any further changes; and I can't increase the batch size without distributing the load across multiple GPUs (expensive!). LR is already set to anneal linearly; switching to a cyclic LR (with `cycle_momentum=False`) would be simply be a trail and error type approach, but might be worth trying.

*(01/10/25) Update:* I've tried training multiple OnT models by modifying the batch size, applying accumulated gradients (for an effective batch size of 512), lowering the LR and training over multiple epochs, switching to cosine annealing; regardless of these changes, the NF1 performance still struggles to exceed an MMR value of 0.2 on the evaluation set when trained on SNOMED~CT FULL. Batch composition may be an issue, so I need to look into that. I also need to review `ELNormalizeData.py` and understand whether we're simply using mixed/random negatives, or applying weighted hard negatives (similarly to HiT; if not, this would likely be advantageous). Failing that, we might apply curriculum learning *(train NF1, then modify parameters and train NF2/NF3 and NF4)*; and adopt a weighting scheme that can penalise incorrectly classified siblings or cousins within the local neighbourhood of candidate $\leftrightarrow$ target pairs *(if it is the case that negatives are uniformly weighted and are sampled at random during training). Beyond that, the only other issue I can think of is excessive verbalisation length (since a lot of the verbalisations for SNOMED CT concepts are actually quite long), which may be adding noise or contributing to a complex loss landscape that is difficult to navigate; perhaps the use of cyclic LR *may be advantageous?* Beyond that, I'd probably just be guessing at what might improve NF1 performance. *Let's revisit this after we've tried some of the aforementioned changes.*

--

### OnT Model Training Run (#2): Exploratory Run \w Cosine Annealing, Accumulated Gradients

*Experiment Name: OnTr-all-MiniLM-L12-v2-H200-OnT-FULL-64-LR-1e-5-cosine-annealing-2-epochs-grad-acc-8*

*Parameters:*

* Ontology: SNOMED~CT, Pinned: Sept-2025 (FULL ONTOLOGY)
* Batch Size: 64
* Learning Rate: 1e-5
* Annealing Strategy: cosine
* Epochs: 2
* Accumulated Gradients: Yes
* Accumulated Gradient Value: 8
* Effective Batch Size: 512
* Logical Loss (conj & exist): 1.0
* Clustering Loss Margin: 3.0 $ \times 1.0$
* Centripetal Loss Margin: 1.0 $ \times 0.5$
* Base Model (PLM): all-MiniLM-L12-v2

*config.yaml:*

```yaml
# dataset from local path
dataset_path: "./data/ont_dataset/OnT"
dataset_name: "OnTr-all-MiniLM-L12-v2-H200-OnT-FULL-64-LR-1e-5-cosine-annealing-2-epochs-grad-acc-8-downweighted-logical-loss"

# Base Model:
model_path: "sentence-transformers/all-MiniLM-L12-v2"

# training config
num_train_epochs: 2
train_batch_size: 64
eval_batch_size: 32
learning_rate: 1e-5
role_emd_mode: "sentenceEmbedding"
role_model_mode: "rotation"
existence_loss_kind: "hit"

# original settings:
hit_loss:
  clustering_loss_weight: 1.0
  clustering_loss_margin: 3.0
  centripetal_loss_weight: 1.0
  centripetal_loss_margin: 0.5
logical_loss:
  conj_weight: 1.0
  exist_weight: 1.0
```

### Model Evaluation: 

```sh
2025-10-01 00:58:00 {'cluster_loss': 0.0023, 'centri_loss': 0.0635, 'combined_loss': 0.5658, 'epoch': 2.0}
2025-10-01 00:58:01 {'cluster_loss': 0.0127, 'centri_loss': 0.0514, 'combined_loss': 0.5643, 'epoch': 2.0}
2025-10-01 00:58:08 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 201.90it/s]
2025-10-01 01:00:15 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 01:00:29 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 198.67it/s]
2025-10-01 01:05:24 computing metrics.......
2025-10-01 01:05:24 H1: 13, H10: 26172, H100: 53296
2025-10-01 01:05:24 MRR: 0.09900868526714908, MR: 703.2410473144769
2025-10-01 01:05:27 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 01:05:41 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 197.64it/s]
2025-10-01 01:10:37 computing metrics.......
2025-10-01 01:10:37 H1: 13, H10: 27438, H100: 58982
2025-10-01 01:10:37 MRR: 0.10477294294312899, MR: 664.097273230651
2025-10-01 01:10:40 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 01:10:54 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 200.31it/s]
2025-10-01 01:15:49 computing metrics.......
2025-10-01 01:15:49 H1: 13, H10: 29381, H100: 63900
2025-10-01 01:15:49 MRR: 0.11163158617069292, MR: 649.3759732408213
2025-10-01 01:15:52 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 01:16:06 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 199.24it/s]
2025-10-01 01:21:01 computing metrics.......
2025-10-01 01:21:01 H1: 13, H10: 32104, H100: 66867
2025-10-01 01:21:01 MRR: 0.11976826908087534, MR: 651.6749460409297
2025-10-01 01:21:05 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 01:21:19 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 201.34it/s]
2025-10-01 01:26:14 computing metrics.......
2025-10-01 01:26:14 H1: 13, H10: 35526, H100: 68205
2025-10-01 01:26:14 MRR: 0.12905513563506726, MR: 667.1597414484761
2025-10-01 01:26:17 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 01:26:31 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:14<00:00, 194.21it/s]
2025-10-01 01:31:26 computing metrics.......
2025-10-01 01:31:26 H1: 13, H10: 39272, H100: 68620
2025-10-01 01:31:26 MRR: 0.13869092322654836, MR: 693.9949035516934
2025-10-01 01:31:29 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 01:31:44 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 197.57it/s]
2025-10-01 01:36:39 computing metrics.......
2025-10-01 01:36:39 H1: 13, H10: 42427, H100: 68547
2025-10-01 01:36:39 MRR: 0.14784206577723968, MR: 731.2198026962585
2025-10-01 01:36:42 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 01:36:56 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 202.24it/s]
2025-10-01 01:41:51 computing metrics.......
2025-10-01 01:41:51 H1: 13, H10: 44341, H100: 68191
2025-10-01 01:41:51 MRR: 0.15524770102969904, MR: 778.804888522256
2025-10-01 01:41:55 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 01:42:09 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 199.01it/s]
2025-10-01 01:47:04 computing metrics.......
2025-10-01 01:47:04 H1: 13, H10: 44986, H100: 67575
2025-10-01 01:47:04 MRR: 0.15953632032222326, MR: 836.8814934514594
2025-10-01 01:47:08 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 01:47:22 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 201.67it/s]
2025-10-01 01:52:17 computing metrics.......
2025-10-01 01:52:17 H1: 13, H10: 44439, H100: 66790
2025-10-01 01:52:17 MRR: 0.16008627706097514, MR: 906.1033754082243
2025-10-01 01:52:21 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.38it/s]
2025-10-01 01:52:35 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 198.48it/s]
2025-10-01 01:57:30 computing metrics.......
2025-10-01 01:57:30 H1: 13, H10: 43140, H100: 65832
2025-10-01 01:57:30 MRR: 0.15668527744063895, MR: 987.1457516413727
2025-10-01 01:57:34 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 01:57:48 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 198.44it/s]
2025-10-01 02:02:43 computing metrics.......
2025-10-01 02:02:43 H1: 13, H10: 41074, H100: 64718
2025-10-01 02:02:43 MRR: 0.15027691378964428, MR: 1080.9081396268632
2025-10-01 02:02:48 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 02:03:02 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:14<00:00, 195.78it/s]
2025-10-01 02:07:57 computing metrics.......
2025-10-01 02:07:57 H1: 13, H10: 38527, H100: 63475
2025-10-01 02:07:57 MRR: 0.1414994953214252, MR: 1188.2770840631463
2025-10-01 02:08:02 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 02:08:16 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:14<00:00, 194.07it/s]
2025-10-01 02:13:11 computing metrics.......
2025-10-01 02:13:11 H1: 13, H10: 35822, H100: 62004
2025-10-01 02:13:11 MRR: 0.13210676119766854, MR: 1310.5375001412542
2025-10-01 02:13:17 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 02:13:32 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 198.83it/s]
2025-10-01 02:18:27 computing metrics.......
2025-10-01 02:18:27 H1: 15, H10: 33139, H100: 60284
2025-10-01 02:18:27 MRR: 0.12292964747537227, MR: 1449.1224729639632
2025-10-01 02:18:32 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 02:18:46 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:14<00:00, 195.39it/s]
2025-10-01 02:23:41 computing metrics.......
2025-10-01 02:23:41 H1: 59, H10: 30621, H100: 58367
2025-10-01 02:23:41 MRR: 0.11439266958667456, MR: 1605.4899596578261
2025-10-01 02:23:48 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 02:24:02 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 199.80it/s]
2025-10-01 02:28:57 computing metrics.......
2025-10-01 02:28:57 H1: 385, H10: 28310, H100: 56287
2025-10-01 02:28:57 MRR: 0.1090149429510996, MR: 1781.301187664561
2025-10-01 02:29:04 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 02:29:18 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 199.93it/s]
2025-10-01 02:34:13 computing metrics.......
2025-10-01 02:34:13 H1: 1034, H10: 26360, H100: 54013
2025-10-01 02:34:13 MRR: 0.10993523688513468, MR: 1978.4511995299063
2025-10-01 02:34:21 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 02:34:35 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 205.49it/s]
2025-10-01 02:39:30 computing metrics.......
2025-10-01 02:39:30 H1: 1582, H10: 24757, H100: 51499
2025-10-01 02:39:30 MRR: 0.11586659348144049, MR: 2198.7422847004846
2025-10-01 02:39:38 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 02:39:52 INFO:hierarchy_transformers.evaluation.ont_eval:Eval results: {'axiom_kind': 'nf1', 'centri_weight': 0.9, 'H@1': np.float64(0.0001469042749144), 'H@10': np.float64(0.5021753133016171), 'H@100': np.float64(0.754748963194829), 'MRR': np.float64(0.16008627706097514), 'MR': np.float64(906.1033754082243), 'median': np.float64(10.0), 'AUC': np.float64(0.9988167145217717)}
2025-10-01 02:44:47 computing metrics.......
2025-10-01 02:44:47 H1: 1616, H10: 23371, H100: 48985
2025-10-01 02:44:47 MRR: 0.11920242204996845, MR: 2444.1283717356173
2025-10-01 02:44:55 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 4972/4972 [6:53:32<00:00,  1.80s/it]INFO:sentence_transformers.trainer:Saving model checkpoint to experiments/OnTr-all-MiniLM-L12-v2-H200-OnT-FULL-64-LR-1e-5-cosine-annealing-2-epochs-grad-acc-8/checkpoint-4972
2025-10-01 02:44:55 {'eval_axiom_kind': 'nf1', 'eval_centri_weight': 0.9, 'eval_H@1': np.float64(0.0001469042749144), 'eval_H@10': np.float64(0.5021753133016171), 'eval_H@100': np.float64(0.754748963194829), 'eval_MRR': np.float64(0.16008627706097514), 'eval_MR': np.float64(906.1033754082243), 'eval_median': np.float64(10.0), 'eval_AUC': np.float64(0.9988167145217717), 'eval_runtime': 6414.5426, 'eval_samples_per_second': 0.0, 'eval_steps_per_second': 0.0, 'epoch': 2.0}
2025-10-01 02:44:55 INFO:sentence_transformers.SentenceTransformer:Save model to experiments/OnTr-all-MiniLM-L12-v2-H200-OnT-FULL-64-LR-1e-5-cosine-annealing-2-epochs-grad-acc-8/checkpoint-4972
2025-10-01 02:44:56 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 4972/4972 [6:53:33<00:00,  4.99s/it]
2025-10-01 02:44:56 {'train_runtime': 24814.0114, 'train_samples_per_second': 102.553, 'train_steps_per_second': 0.2, 'train_loss': 0.6678308861349632, 'epoch': 2.0}
2025-10-01 02:44:56 0.9
2025-10-01 02:45:03 Batches: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 23835/23835 [02:01<00:00, 195.44it/s]
2025-10-01 02:47:07 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 4039/4039 [00:20<00:00, 198.81it/s]
2025-10-01 02:47:28 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 4039/4039 [07:10<00:00,  9.38it/s]
2025-10-01 02:54:39 computing metrics.......
2025-10-01 02:54:39 H1: 16, H10: 49932, H100: 77111
2025-10-01 02:54:39 MRR: 0.1259697188158189, MR: 3937.3332843270014
2025-10-01 02:55:07 INFO:hierarchy_transformers.evaluation.ont_eval:Eval results nf1: {'axiom_kind': 'nf1', 'centri_weight': 0.9, 'H@1': np.float64(0.00012380547065423452), 'H@10': np.float64(0.38636592254420243), 'H@100': np.float64(0.5966727279761674), 'MRR': np.float64(0.1259697188158189), 'MR': np.float64(3937.3332843270014), 'median': np.float64(30.0), 'AUC': np.float64(0.9948416264375664)}
2025-10-01 02:55:07 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1682/1682 [00:09<00:00, 181.66it/s]
2025-10-01 02:55:23 Evaluating nf2: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1682/1682 [02:59<00:00,  9.37it/s]
2025-10-01 02:58:22 computing metrics.......
2025-10-01 02:58:22 H1: 18031, H10: 33692, H100: 46339
2025-10-01 02:58:22 MRR: 0.4212456753416027, MR: 546.5821392063934
2025-10-01 02:58:23 INFO:hierarchy_transformers.evaluation.ont_eval:Eval results nf2: {'axiom_kind': 'nf2', 'centri_weight': 0.9, 'H@1': np.float64(0.335117554130657), 'H@10': np.float64(0.626187157327386), 'H@100': np.float64(0.8612396617414738), 'MRR': np.float64(0.4212456753416027), 'MR': np.float64(546.5821392063934), 'median': np.float64(5.0), 'AUC': np.float64(0.9992887470660498)}
2025-10-01 02:58:24 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1849/1849 [00:09<00:00, 192.81it/s]
2025-10-01 02:58:38 Evaluating nf3: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1849/1849 [05:55<00:00,  5.20it/s]
2025-10-01 03:04:34 computing metrics.......
2025-10-01 03:04:34 H1: 15625, H10: 23552, H100: 39712
2025-10-01 03:04:34 MRR: 0.30372501579571387, MR: 2765.7638496779537
2025-10-01 03:04:40 INFO:hierarchy_transformers.evaluation.ont_eval:Eval results nf3: {'axiom_kind': 'nf3', 'centri_weight': 0.9, 'H@1': np.float64(0.2641455209372306), 'H@10': np.float64(0.3981539397832739), 'H@100': np.float64(0.6713438033573952), 'MRR': np.float64(0.30372501579571387), 'MR': np.float64(2765.7638496779537), 'median': np.float64(24.0), 'AUC': np.float64(0.9963816765919677)}
2025-10-01 03:04:41 Batches: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 551/551 [00:03<00:00, 182.73it/s]
2025-10-01 03:04:45 Evaluating nf4: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 551/551 [00:58<00:00,  9.37it/s]
2025-10-01 03:05:44 computing metrics.......
2025-10-01 03:05:44 H1: 15821, H10: 16685, H100: 17106
2025-10-01 03:05:44 MRR: 0.9140518187372619, MR: 27.092333011747346
2025-10-01 03:05:44 INFO:hierarchy_transformers.evaluation.ont_eval:Eval results nf4: {'axiom_kind': 'nf4', 'centri_weight': 0.9, 'H@1': np.float64(0.8978491572555474), 'H@10': np.float64(0.9468815617728846), 'H@100': np.float64(0.9707735088814483), 'MRR': np.float64(0.9140518187372619), 'MR': np.float64(27.092333011747346), 'median': np.float64(1.0), 'AUC': np.float64(0.9999661197881216)}
2025-10-01 03:06:10 INFO:hierarchy_transformers.evaluation.ont_eval:Combined eval results: {'axiom_kind': 'combined', 'centri_weight': 0.9, 'H@1': 0.1904939687622684, 'H@10': 0.4767295064931066, 'H@100': 0.6938348202945184, 'MRR': np.float64(0.28103786990019947), 'MR': np.float64(2703.2059165402943), 'median': np.float64(13.0), 'AUC': np.float64(0.9964586118199495)}
2025-10-01 03:06:10 INFO:sentence_transformers.SentenceTransformer:Save model to experiments/OnTr-all-MiniLM-L12-v2-H200-OnT-FULL-64-LR-1e-5-cosine-annealing-2-epochs-grad-acc-8/final
```

**Wandb Summary:**

```json
{
  "eval/samples_per_second": 0,
  "train_loss": 0.667830886134963,
  "total_flos": 0,
  "eval/H@10": 0.502175313301617,
  "train/centri_loss": 0.0514,
  "eval/axiom_kind": "nf1",
  "eval/H@100": 0.754748963194829,
  "_runtime": 26087,
  "eval/MRR": 0.160086277060975,
  "eval/median": 10,
  "eval/centri_weight": 0.9,
  "train/combined_loss": 0.5643,
  "train/global_step": 4972,
  "_wandb": {
    "runtime": 26087
  },
  "eval/runtime": 6414.5426,
  "_timestamp": 1759286696.57811,
  "eval/MR": 906.103375408224,
  "train/learning_rate": 6.57334917644248e-9,
  "eval/AUC": 0.998816714521772,
  "_step": 39813,
  "train_runtime": 24814.0114,
  "eval/steps_per_second": 0,
  "train/cluster_loss": 0.0127,
  "train/loss": 0.5602,
  "train/epoch": 2,
  "train/grad_norm": 0.869533240795136,
  "train_samples_per_second": 102.553,
  "train_steps_per_second": 0.2,
  "eval/H@1": 0.0001469042749144
}
```

## OnT Model Training Run (#3): Exploratory Run \w Cosine Annealing, Accumulated Gradients & Modified Logical Loss

*Experiment Name: OnTr-all-MiniLM-L12-v2-H200-OnT-FULL-64-LR-1e-5-cosine-annealing-2-epochs-grad-acc-8-downweighted-logical-loss*

* Ontology: SNOMED~CT, Pinned: Sept-2025 (FULL ONTOLOGY)

*Parameters:*

* Batch Size: 64
* Learning Rate: 1e-5
* Annealing Strategy: cosine
* Epochs: 2
* Accumulated Gradients: Yes
* Accumulated Gradient Value: 8
* Effective Batch Size: 512
* Logical Loss (conj & exist): 0.5
* Clustering Loss Margin: 3.0 $ \times 1.0$
* Centripetal Loss Margin: 1.0 $ \times 0.5$
* Base Model (PLM): all-MiniLM-L12-v2

*config.yaml:*

```yaml
# dataset from local path
dataset_path: "./data/ont_dataset/OnT"
dataset_name: "OnTr-all-MiniLM-L12-v2-H200-OnT-FULL-64-LR-1e-5-cosine-annealing-2-epochs-grad-acc-8-downweighted-logical-loss"

# Base Model:
model_path: "sentence-transformers/all-MiniLM-L12-v2"

# training config
num_train_epochs: 2
train_batch_size: 64
eval_batch_size: 32
learning_rate: 1e-5
role_emd_mode: "sentenceEmbedding"
role_model_mode: "rotation"
existence_loss_kind: "hit"

# original settings:
hit_loss:
  clustering_loss_weight: 1.0
  clustering_loss_margin: 3.0
  centripetal_loss_weight: 1.0
  centripetal_loss_margin: 0.5
logical_loss:
  conj_weight: 0.5
  exist_weight: 0.5
```

### Model Evaluation: 

```sh
{'cluster_loss': 0.0013, 'centri_loss': 0.0684, 'combined_loss': 0.2247, 'epoch': 2.0}
{'cluster_loss': 0.0002, 'centri_loss': 0.0569, 'combined_loss': 0.2167, 'epoch': 2.0}
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 198.94it/s]
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 199.44it/s]
computing metrics.......
H1: 13, H10: 26132, H100: 53369
MRR: 0.09867620100561791, MR: 680.9744047551784
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 200.61it/s]
computing metrics.......
H1: 13, H10: 27403, H100: 59544
MRR: 0.10460160608458187, MR: 643.7408156577357
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 203.60it/s]
computing metrics.......
H1: 13, H10: 29418, H100: 64474
MRR: 0.11170554791310418, MR: 630.6903483891381
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 207.87it/s]
computing metrics.......
H1: 13, H10: 32333, H100: 67241
MRR: 0.12018948319446474, MR: 634.139400856565
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 201.80it/s]
computing metrics.......
H1: 13, H10: 36121, H100: 68257
MRR: 0.12991593000498664, MR: 650.1639225701468
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 198.76it/s]
computing metrics.......
H1: 13, H10: 39808, H100: 68572
MRR: 0.13976609711201715, MR: 677.0494615393308
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 204.88it/s]
computing metrics.......
H1: 13, H10: 42674, H100: 68408
MRR: 0.14827845111057533, MR: 713.8686223769112
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 206.28it/s]
computing metrics.......
H1: 13, H10: 43993, H100: 67873
MRR: 0.1541331893791927, MR: 760.5793113579605
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 209.61it/s]
computing metrics.......
H1: 13, H10: 44041, H100: 67125
MRR: 0.1559213807422997, MR: 817.3492592634445
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 202.31it/s]
computing metrics.......
H1: 13, H10: 42895, H100: 66209
MRR: 0.15323543715903418, MR: 884.7796548879572
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:14<00:00, 196.11it/s]
computing metrics.......
H1: 13, H10: 41111, H100: 65095
MRR: 0.14673740106838434, MR: 963.5372402336908
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 203.51it/s]
computing metrics.......
H1: 13, H10: 38784, H100: 63876
MRR: 0.13772934314294263, MR: 1054.4177731571988
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 199.77it/s]
computing metrics.......
H1: 13, H10: 36041, H100: 62387
MRR: 0.12814867803790111, MR: 1158.7030160577672
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 203.47it/s]
computing metrics.......
H1: 13, H10: 33296, H100: 60687
MRR: 0.11895915230649291, MR: 1277.3269863153018
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 205.49it/s]
computing metrics.......
H1: 21, H10: 30696, H100: 58792
MRR: 0.11053544320238354, MR: 1411.6931282700327
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 207.48it/s]
computing metrics.......
H1: 163, H10: 28267, H100: 56732
MRR: 0.10414500211351917, MR: 1563.123546495203
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 205.36it/s]
computing metrics.......
H1: 662, H10: 26272, H100: 54476
MRR: 0.10394877564900706, MR: 1733.2644050941883
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 204.93it/s]
computing metrics.......
H1: 1187, H10: 24563, H100: 51928
MRR: 0.1108205167401445, MR: 1923.922445843174
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 203.46it/s]
computing metrics.......
H1: 1309, H10: 23212, H100: 49345
MRR: 0.11647340041478298, MR: 2136.777553026793
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
INFO:hierarchy_transformers.evaluation.ont_eval:Eval results: {'axiom_kind': 'nf1', 'centri_weight': 0.8, 'H@1': np.float64(0.0001469042749144), 'H@10': np.float64(0.4976777824234685), 'H@100': np.float64(0.7585345733560847), 'MRR': np.float64(0.1559213807422997), 'MR': np.float64(817.3492592634445), 'median': np.float64(11.0), 'AUC': np.float64(0.9989329661267078)}
computing metrics.......
H1: 1311, H10: 22143, H100: 46715
MRR: 0.11760655820000644, MR: 2373.5825997536526
100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 4972/4972 [6:54:10<00:00,  1.83s/it]INFO:sentence_transformers.trainer:Saving model checkpoint to experiments/OnTr-all-MiniLM-L12-v2-H200-OnT-FULL-64-LR-1e-5-cosine-annealing-2-epochs-grad-acc-8-downweighted-logical-loss/checkpoint-4972
{'eval_axiom_kind': 'nf1', 'eval_centri_weight': 0.8, 'eval_H@1': np.float64(0.0001469042749144), 'eval_H@10': np.float64(0.4976777824234685), 'eval_H@100': np.float64(0.7585345733560847), 'eval_MRR': np.float64(0.1559213807422997), 'eval_MR': np.float64(817.3492592634445), 'eval_median': np.float64(11.0), 'eval_AUC': np.float64(0.9989329661267078), 'eval_runtime': 6407.305, 'eval_samples_per_second': 0.0, 'eval_steps_per_second': 0.0, 'epoch': 2.0}
INFO:sentence_transformers.SentenceTransformer:Save model to experiments/OnTr-all-MiniLM-L12-v2-H200-OnT-FULL-64-LR-1e-5-cosine-annealing-2-epochs-grad-acc-8-downweighted-logical-loss/checkpoint-4972
100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 4972/4972 [6:54:11<00:00,  5.00s/it]
{'train_runtime': 24851.9433, 'train_samples_per_second': 102.396, 'train_steps_per_second': 0.2, 'train_loss': 0.2922536910587228, 'epoch': 2.0}
0.8
Batches: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 23835/23835 [02:05<00:00, 189.30it/s]
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 4039/4039 [00:20<00:00, 195.79it/s]
Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 4039/4039 [07:10<00:00,  9.37it/s]
computing metrics.......
H1: 16, H10: 50292, H100: 78490
MRR: 0.12296676851482523, MR: 3859.352365845166
INFO:hierarchy_transformers.evaluation.ont_eval:Eval results nf1: {'axiom_kind': 'nf1', 'centri_weight': 0.8, 'H@1': np.float64(0.00012380547065423452), 'H@10': np.float64(0.3891515456339227), 'H@100': np.float64(0.6073432119781793), 'MRR': np.float64(0.12296676851482523), 'MR': np.float64(3859.352365845166), 'median': np.float64(27.0), 'AUC': np.float64(0.9949438938156662)}
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1682/1682 [00:09<00:00, 186.77it/s]
Evaluating nf2: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1682/1682 [02:59<00:00,  9.37it/s]
computing metrics.......
H1: 20912, H10: 38998, H100: 48538
MRR: 0.4925682632683548, MR: 326.3870272279528
INFO:hierarchy_transformers.evaluation.ont_eval:Eval results nf2: {'axiom_kind': 'nf2', 'centri_weight': 0.8, 'H@1': np.float64(0.38866276368367253), 'H@10': np.float64(0.7248025276461295), 'H@100': np.float64(0.9021094693801691), 'MRR': np.float64(0.4925682632683548), 'MR': np.float64(326.3870272279528), 'median': np.float64(3.0), 'AUC': np.float64(0.999576315707251)}
Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1849/1849 [00:09<00:00, 186.47it/s]
Evaluating nf3: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1849/1849 [05:55<00:00,  5.20it/s]
computing metrics.......
H1: 15626, H10: 24155, H100: 40520
MRR: 0.30627120915396205, MR: 2517.114753266952
INFO:hierarchy_transformers.evaluation.ont_eval:Eval results nf3: {'axiom_kind': 'nf3', 'centri_weight': 0.8, 'H@1': np.float64(0.26416242625057057), 'H@10': np.float64(0.4083478437272835), 'H@100': np.float64(0.6850032965361013), 'MRR': np.float64(0.30627120915396205), 'MR': np.float64(2517.114753266952), 'median': np.float64(21.0), 'AUC': np.float64(0.9967068304457782)}
Batches: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 551/551 [00:03<00:00, 181.01it/s]
Evaluating nf4: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 551/551 [00:58<00:00,  9.37it/s]
computing metrics.......
H1: 15840, H10: 16698, H100: 17171
MRR: 0.915193891186155, MR: 26.72771125361784
INFO:hierarchy_transformers.evaluation.ont_eval:Eval results nf4: {'axiom_kind': 'nf4', 'centri_weight': 0.8, 'H@1': np.float64(0.8989274161511832), 'H@10': np.float64(0.9476193178593724), 'H@100': np.float64(0.9744622893138868), 'MRR': np.float64(0.915193891186155), 'MR': np.float64(26.72771125361784), 'median': np.float64(1.0), 'AUC': np.float64(0.9999667847143772)}
INFO:hierarchy_transformers.evaluation.ont_eval:Combined eval results: {'axiom_kind': 'combined', 'centri_weight': 0.8, 'H@1': 0.2016596488256984, 'H@10': 0.5009083421216717, 'H@100': 0.7109663066655376, 'MRR': np.float64(0.29497154936163117), 'MR': np.float64(2562.181079541518), 'median': np.float64(10.0), 'AUC': np.float64(0.996643318174559)}
INFO:sentence_transformers.SentenceTransformer:Save model to experiments/OnTr-all-MiniLM-L12-v2-H200-OnT-FULL-64-LR-1e-5-cosine-annealing-2-epochs-grad-acc-8-downweighted-logical-loss/final
```

**Wandb Summary:**

```json
{
  "_runtime": 26126,
  "_step": 39813,
  "_timestamp": 1759332376.1671488,
  "_wandb.runtime": 26126,
  "eval/AUC": 0.9989329661267078,
  "eval/H@1": 0.0001469042749144,
  "eval/H@10": 0.4976777824234685,
  "eval/H@100": 0.7585345733560847,
  "eval/MR": 817.3492592634445,
  "eval/MRR": 0.1559213807422997,
  "eval/axiom_kind": "nf1",
  "eval/centri_weight": 0.8,
  "eval/median": 11,
  "eval/runtime": 6407.305,
  "eval/samples_per_second": 0,
  "eval/steps_per_second": 0,
  "total_flos": 0,
  "train/centri_loss": 0.0569,
  "train/cluster_loss": 0.0002,
  "train/combined_loss": 0.2167,
  "train/epoch": 2,
  "train/global_step": 4972,
  "train/grad_norm": 0.7168806195259094,
  "train/learning_rate": 6.573349176442478e-9,
  "train/loss": 0.2218,
  "train_loss": 0.2922536910587228,
  "train_runtime": 24851.9433,
  "train_samples_per_second": 102.396,
  "train_steps_per_second": 0.2
}
```

## OnT Model Training Run (#4): Re-Training HiT Tuned Model

*Experiment Name: OnTr-final-H200-OnT-FULL-HiT-RETRAINED-2-EPOCHS*

* Ontology: SNOMED~CT, Pinned: Sept-2025 (FULL ONTOLOGY)

*Parameters:*

* Batch Size: 64
* Learning Rate: 1e-5
* Annealing Strategy: cosine
* Epochs: 2
* Accumulated Gradients: Yes
* Accumulated Gradient Value: 8
* Effective Batch Size: 512
* Logical Loss (conj & exist): 1.0
* Clustering Loss Margin: 3.0 $ \times 1.0$
* Centripetal Loss Margin: 1.0 $ \times 0.5$
* Base Model: Pre-trained HiT Model

*config.yaml:*

```yaml
# dataset from local path
dataset_path: "./data/ont_dataset/OnT"
dataset_name: "OnTr-final-H200-OnT-FULL-HiT-RETRAINED-2-EPOCHS"

# PLM Base Model (HiT Re-Training):
model_path: "experiments/HiT-all-MiniLM-L12-v2-hit_dataset-mixed/final"

# training config
num_train_epochs: 2
train_batch_size: 64
eval_batch_size: 32
learning_rate: 1e-5
role_emd_mode: "sentenceEmbedding"
role_model_mode: "rotation"
existence_loss_kind: "hit"

# original settings:
hit_loss:
  clustering_loss_weight: 1.0
  clustering_loss_margin: 3.0
  centripetal_loss_weight: 1.0
  centripetal_loss_margin: 0.5
logical_loss:
  conj_weight: 1.0
  exist_weight: 1.0
```

### Model Evaluation: 

```sh
2025-10-01 22:00:16 {'cluster_loss': 0.0, 'centri_loss': 0.0073, 'combined_loss': 0.5073, 'epoch': 2.0}
2025-10-01 22:00:17 {'cluster_loss': 0.0, 'centri_loss': 0.0213, 'combined_loss': 0.5246, 'epoch': 2.0}
2025-10-01 22:00:24 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 200.23it/s]
2025-10-01 22:02:28 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 22:02:41 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 209.37it/s]
2025-10-01 22:07:36 computing metrics.......
2025-10-01 22:07:36 H1: 13, H10: 27737, H100: 60168
2025-10-01 22:07:36 MRR: 0.10316016355184805, MR: 201.71199981919474
2025-10-01 22:07:38 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 22:07:51 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 200.22it/s]
2025-10-01 22:12:46 computing metrics.......
2025-10-01 22:12:46 H1: 13, H10: 28798, H100: 66340
2025-10-01 22:12:46 MRR: 0.10780967706670237, MR: 177.76278349699976
2025-10-01 22:12:48 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.38it/s]
2025-10-01 22:13:02 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 200.58it/s]
2025-10-01 22:17:57 computing metrics.......
2025-10-01 22:17:57 H1: 13, H10: 30665, H100: 71039
2025-10-01 22:17:57 MRR: 0.11417863856790901, MR: 164.8614353677692
2025-10-01 22:17:58 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:54<00:00,  9.38it/s]
2025-10-01 22:18:12 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 202.88it/s]
2025-10-01 22:23:07 computing metrics.......
2025-10-01 22:23:07 H1: 13, H10: 33405, H100: 73344
2025-10-01 22:23:07 MRR: 0.12216669407656956, MR: 158.27403297435956
2025-10-01 22:23:08 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 22:23:22 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 203.45it/s]
2025-10-01 22:28:17 computing metrics.......
2025-10-01 22:28:17 H1: 13, H10: 37112, H100: 74429
2025-10-01 22:28:17 MRR: 0.13195422295436954, MR: 155.60243183076628
2025-10-01 22:28:18 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.38it/s]
2025-10-01 22:28:32 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 205.51it/s]
2025-10-01 22:33:27 computing metrics.......
2025-10-01 22:33:27 H1: 13, H10: 40987, H100: 74962
2025-10-01 22:33:27 MRR: 0.14261452946589964, MR: 155.6123535194874
2025-10-01 22:33:28 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 22:33:42 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 203.58it/s]
2025-10-01 22:38:37 computing metrics.......
2025-10-01 22:38:37 H1: 13, H10: 44107, H100: 75217
2025-10-01 22:38:37 MRR: 0.15387819435729005, MR: 157.666549896602
2025-10-01 22:38:38 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.38it/s]
2025-10-01 22:38:52 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 203.41it/s]
2025-10-01 22:43:47 computing metrics.......
2025-10-01 22:43:47 H1: 13, H10: 45900, H100: 75337
2025-10-01 22:43:47 MRR: 0.16402741770949314, MR: 161.38946583345574
2025-10-01 22:43:48 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.38it/s]
2025-10-01 22:44:02 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 202.30it/s]
2025-10-01 22:48:57 computing metrics.......
2025-10-01 22:48:57 H1: 13, H10: 46664, H100: 75333
2025-10-01 22:48:57 MRR: 0.17126750478394723, MR: 166.63070525352288
2025-10-01 22:48:58 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 22:49:12 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 198.95it/s]
2025-10-01 22:54:07 computing metrics.......
2025-10-01 22:54:07 H1: 13, H10: 46695, H100: 75230
2025-10-01 22:54:07 MRR: 0.17583078562820947, MR: 173.3383883471009
2025-10-01 22:54:08 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 22:54:22 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 203.16it/s]
2025-10-01 22:59:17 computing metrics.......
2025-10-01 22:59:17 H1: 13, H10: 46153, H100: 75095
2025-10-01 22:59:17 MRR: 0.17681633684335854, MR: 181.51068446091782
2025-10-01 22:59:18 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.38it/s]
2025-10-01 22:59:32 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 202.40it/s]
2025-10-01 23:04:27 computing metrics.......
2025-10-01 23:04:27 H1: 13, H10: 45439, H100: 74898
2025-10-01 23:04:27 MRR: 0.1746389717745627, MR: 191.1758331167437
2025-10-01 23:04:29 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 23:04:42 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 200.99it/s]
2025-10-01 23:09:37 computing metrics.......
2025-10-01 23:09:37 H1: 13, H10: 44448, H100: 74627
2025-10-01 23:09:37 MRR: 0.17084648395293703, MR: 202.43613619156318
2025-10-01 23:09:39 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.38it/s]
2025-10-01 23:09:53 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 205.86it/s]
2025-10-01 23:14:48 computing metrics.......
2025-10-01 23:14:48 H1: 14, H10: 43176, H100: 74268
2025-10-01 23:14:48 MRR: 0.16636958593556345, MR: 215.39327404427468
2025-10-01 23:14:49 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 23:15:02 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 200.78it/s]
2025-10-01 23:19:58 computing metrics.......
2025-10-01 23:19:58 H1: 19, H10: 41753, H100: 73834
2025-10-01 23:19:58 MRR: 0.16118680059658252, MR: 230.1759800210186
2025-10-01 23:19:59 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.38it/s]
2025-10-01 23:20:13 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 205.81it/s]
2025-10-01 23:25:08 computing metrics.......
2025-10-01 23:25:08 H1: 174, H10: 40090, H100: 73308
2025-10-01 23:25:08 MRR: 0.15620830566037883, MR: 246.92055868825784
2025-10-01 23:25:09 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 23:25:23 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 206.15it/s]
2025-10-01 23:30:18 computing metrics.......
2025-10-01 23:30:18 H1: 1568, H10: 38430, H100: 72798
2025-10-01 23:30:18 MRR: 0.1587186510948388, MR: 265.80373588871436
2025-10-01 23:30:19 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.38it/s]
2025-10-01 23:30:33 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 203.21it/s]
2025-10-01 23:35:28 computing metrics.......
2025-10-01 23:35:28 H1: 5769, H10: 36714, H100: 72229
2025-10-01 23:35:28 MRR: 0.17764314773965903, MR: 286.9802583255173
2025-10-01 23:35:30 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.37it/s]
2025-10-01 23:35:43 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [00:13<00:00, 201.81it/s]
2025-10-01 23:40:38 computing metrics.......
2025-10-01 23:40:38 H1: 10923, H10: 34966, H100: 71552
2025-10-01 23:40:38 MRR: 0.20252332240601215, MR: 310.7398438294554
2025-10-01 23:40:40 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 2766/2766 [04:55<00:00,  9.38it/s]
2025-10-01 23:40:54 INFO:hierarchy_transformers.evaluation.ont_eval:Eval results: {'axiom_kind': 'nf1', 'centri_weight': 1.9, 'H@1': np.float64(0.15170691467121694), 'H@10': np.float64(0.37639135298837195), 'H@100': np.float64(0.8001084831568599), 'MRR': np.float64(0.2132874196855529), 'MR': np.float64(337.29456567186105), 'median': np.float64(16.0), 'AUC': np.float64(0.9995629469363853)}
2025-10-01 23:45:49 computing metrics.......
2025-10-01 23:45:49 H1: 13425, H10: 33308, H100: 70804
2025-10-01 23:45:49 MRR: 0.2132874196855529, MR: 337.29456567186105
2025-10-01 23:45:50 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 4972/4972 [6:49:36<00:00,  1.78s/it]INFO:sentence_transformers.trainer:Saving model checkpoint to experiments/OnTr-final-H200-OnT-FULL-HiT-RETRAINED-2-EPOCHS/checkpoint-4972
2025-10-01 23:45:50 {'eval_axiom_kind': 'nf1', 'eval_centri_weight': 1.9, 'eval_H@1': np.float64(0.15170691467121694), 'eval_H@10': np.float64(0.37639135298837195), 'eval_H@100': np.float64(0.8001084831568599), 'eval_MRR': np.float64(0.2132874196855529), 'eval_MR': np.float64(337.29456567186105), 'eval_median': np.float64(16.0), 'eval_AUC': np.float64(0.9995629469363853), 'eval_runtime': 6333.2745, 'eval_samples_per_second': 0.0, 'eval_steps_per_second': 0.0, 'epoch': 2.0}
2025-10-01 23:45:50 INFO:sentence_transformers.SentenceTransformer:Save model to experiments/OnTr-final-H200-OnT-FULL-HiT-RETRAINED-2-EPOCHS/checkpoint-4972
2025-10-01 23:45:51 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 4972/4972 [6:49:37<00:00,  4.94s/it]
2025-10-01 23:45:51 {'train_runtime': 24577.9099, 'train_samples_per_second': 103.538, 'train_steps_per_second': 0.202, 'train_loss': 0.5525434498621971, 'epoch': 2.0}
2025-10-01 23:45:51 1.9
2025-10-01 23:45:58 Batches: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 23835/23835 [02:03<00:00, 192.23it/s]
2025-10-01 23:48:04 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 4039/4039 [00:20<00:00, 194.86it/s]
2025-10-01 23:48:25 Evaluating nf1: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 4039/4039 [07:10<00:00,  9.38it/s]
2025-10-01 23:55:36 computing metrics.......
2025-10-01 23:55:36 H1: 13322, H10: 37644, H100: 80155
2025-10-01 23:55:36 MRR: 0.1536122787617792, MR: 3432.6108252408403
2025-10-01 23:56:02 INFO:hierarchy_transformers.evaluation.ont_eval:Eval results nf1: {'axiom_kind': 'nf1', 'centri_weight': 1.9, 'H@1': np.float64(0.10308353000348203), 'H@10': np.float64(0.2912833210817503), 'H@100': np.float64(0.6202267187681356), 'MRR': np.float64(0.1536122787617792), 'MR': np.float64(3432.6108252408403), 'median': np.float64(34.0), 'AUC': np.float64(0.9955024186707613)}
2025-10-01 23:56:02 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1682/1682 [00:09<00:00, 181.29it/s]
2025-10-01 23:56:18 Evaluating nf2: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1682/1682 [02:59<00:00,  9.38it/s]
2025-10-01 23:59:17 computing metrics.......
2025-10-01 23:59:17 H1: 12616, H10: 22284, H100: 40964
2025-10-01 23:59:17 MRR: 0.28456347173981067, MR: 802.0295511569557
2025-10-01 23:59:19 INFO:hierarchy_transformers.evaluation.ont_eval:Eval results nf2: {'axiom_kind': 'nf2', 'centri_weight': 1.9, 'H@1': np.float64(0.2344763497816188), 'H@10': np.float64(0.4141622525787566), 'H@100': np.float64(0.7613418827246539), 'MRR': np.float64(0.28456347173981067), 'MR': np.float64(802.0295511569557), 'median': np.float64(15.0), 'AUC': np.float64(0.9989555094829023)}
2025-10-01 23:59:20 Batches: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1849/1849 [00:09<00:00, 191.35it/s]
2025-10-01 23:59:34 Evaluating nf3: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 1849/1849 [05:55<00:00,  5.20it/s]
2025-10-02 00:05:30 computing metrics.......
2025-10-02 00:05:30 H1: 15646, H10: 25507, H100: 42687
2025-10-02 00:05:30 MRR: 0.31190982422638075, MR: 1557.1651649113317
2025-10-02 00:05:34 INFO:hierarchy_transformers.evaluation.ont_eval:Eval results nf3: {'axiom_kind': 'nf3', 'centri_weight': 1.9, 'H@1': np.float64(0.2645005325173702), 'H@10': np.float64(0.43120382736294016), 'H@100': np.float64(0.7216371105438439), 'MRR': np.float64(0.31190982422638075), 'MR': np.float64(1557.1651649113317), 'median': np.float64(16.0), 'AUC': np.float64(0.9979687240698933)}
2025-10-02 00:05:34 Batches: 100%|██████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 551/551 [00:02<00:00, 187.28it/s]
2025-10-02 00:05:38 Evaluating nf4: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 551/551 [00:58<00:00,  9.37it/s]
2025-10-02 00:06:37 computing metrics.......
2025-10-02 00:06:37 H1: 15130, H10: 16736, H100: 17599
2025-10-02 00:06:37 MRR: 0.8826542842989215, MR: 2.8383746665909992
2025-10-02 00:06:37 INFO:hierarchy_transformers.evaluation.ont_eval:Eval results nf4: {'axiom_kind': 'nf4', 'centri_weight': 1.9, 'H@1': np.float64(0.8586345837353159), 'H@10': np.float64(0.9497758356506442), 'H@100': np.float64(0.99875148969979), 'MRR': np.float64(0.8826542842989215), 'MR': np.float64(2.8383746665909992), 'median': np.float64(1.0), 'AUC': np.float64(0.9999976355738721)}
2025-10-02 00:06:54 INFO:hierarchy_transformers.evaluation.ont_eval:Combined eval results: {'axiom_kind': 'combined', 'centri_weight': 1.9, 'H@1': 0.218286928341044, 'H@10': 0.3932467072598089, 'H@100': 0.6982110278891822, 'MRR': np.float64(0.26621611390777034), 'MR': np.float64(2228.238917071443), 'median': np.float64(19.0), 'AUC': np.float64(0.9970816233001082)}
2025-10-02 00:06:54 INFO:sentence_transformers.SentenceTransformer:Save model to experiments/OnTr-final-H200-OnT-FULL-HiT-RETRAINED-2-EPOCHS/final
```

**Wandb Summary:**

```json
{
  "_runtime": 25841,
  "_step": 39813,
  "_timestamp": 1759362351.180356,
  "_wandb.runtime": 25841,
  "eval/AUC": 0.9995629469363853,
  "eval/H@1": 0.15170691467121694,
  "eval/H@10": 0.3763913529883719,
  "eval/H@100": 0.8001084831568599,
  "eval/MR": 337.29456567186105,
  "eval/MRR": 0.2132874196855529,
  "eval/axiom_kind": "nf1",
  "eval/centri_weight": 1.9,
  "eval/median": 16,
  "eval/runtime": 6333.2745,
  "eval/samples_per_second": 0,
  "eval/steps_per_second": 0,
  "total_flos": 0,
  "train/centri_loss": 0.0213,
  "train/cluster_loss": 0,
  "train/combined_loss": 0.5246,
  "train/epoch": 2,
  "train/global_step": 4972,
  "train/grad_norm": 0.6796685457229614,
  "train/learning_rate": 6.573349176442478e-9,
  "train/loss": 0.5254,
  "train_loss": 0.5525434498621971,
  "train_runtime": 24577.9099,
  "train_samples_per_second": 103.538,
  "train_steps_per_second": 0.202
}
```

## OnT Model Training Run (#5): Pre-longed Training Run for OnT on standard PLM Base Model \w Training Modified Parameters

*Experiment Name: OnTr-all-MiniLM-L12-v2-H200-OnT-FULL-64-LR-5e-6-cosine-annealing-20-epochs-grad-acc-8-downweighted-logical-loss-075-downweighted-centripetal-loss-085*

* Ontology: SNOMED~CT, Pinned: Sept-2025 (FULL ONTOLOGY)

*Parameters:*

* Batch Size: 64
* Learning Rate: 5e-6
* Annealing Strategy: cosine
* Epochs: 20
* Accumulated Gradients: Yes
* Accumulated Gradient Value: 8
* Effective Batch Size: 512
* Logical Loss (conj & exist): 0.75
* Clustering Loss Margin: 3.0 $ \times 1.0$
* Centripetal Loss Margin: 0.85 $ \times 0.5$
* Base Model (PLM): all-MiniLM-L12-v2

*config.yaml:*

```yaml
# dataset from local path
dataset_path: "./data/ont_dataset/OnT"
dataset_name: "H200-OnT-FULL-64-LR-5e-6-cosine-annealing-20-epochs-grad-acc-8-downweighted-logical-loss-075-downweighted-centripetal-loss-085"

# PLM Base model:
model_path: "sentence-transformers/all-MiniLM-L12-v2"

# training config
num_train_epochs: 20
train_batch_size: 64
eval_batch_size: 32
# halfed the LR \w cosine annealing over 20 epochs
learning_rate: 5e-6
role_emd_mode: "sentenceEmbedding"
role_model_mode: "rotation"
existence_loss_kind: "hit"

# downweighted logical loss (NF2->NF4)
# slightly modified centripetal_loss_weight
# TODO: try training a model, simply by upweighting the clustering loss and... 
# possibly, the centripetal loss; as it *might* help with subsumption (NF1)
hit_loss:
  clustering_loss_weight: 1.0
  clustering_loss_margin: 3.0
  centripetal_loss_weight: 0.85
  centripetal_loss_margin: 0.5
logical_loss:
  conj_weight: 0.75
  exist_weight: 0.75
```

**Model Evaluation:**

```sh
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

**Wandb Summary:**

```json
{
  "eval/AUC": 0.999325681870819,
  "eval/H@1": 0.0001469042749144,
  "eval/H@100": 0.7841976201507463,
  "train/combined_loss": 0.3681,
  "eval/runtime": 6406.9,
  "eval/centri_weight": 0.9,
  "eval/axiom_kind": "nf1",
  "_timestamp": 1759585974.7031848,
  "total_flos": 0,
  "train/epoch": 20,
  "train/loss": 0.3448,
  "train_samples_per_second": 102.088,
  "train/learning_rate": 2.2457689041743837e-12,
  "train_runtime": 249270.2901,
  "eval/steps_per_second": 0,
  "eval/MR": 517.7282722927238,
  "train/centri_loss": 0.057,
  "eval/median": 8,
  "eval/MRR": 0.17860439042224602,
  "train/cluster_loss": 0,
  "eval/H@10": 0.5381329596691263,
  "_step": 398137,
  "_runtime": 250565,
  "train_loss": 0.3647588308166326,
  "train/grad_norm": 0.49566471576690674,
  "_wandb": {
    "runtime": 250565
  },
  "train/global_step": 49720,
  "train_steps_per_second": 0.199,
  "eval/samples_per_second": 0
}
```