# SparseFold: English Speaker Script

Paper: Yunzhong Ren, Xin You, and Hailong Yang. *SparseFold: Index-Based Block Sparse Attention for Training-Free AlphaFold 3 Acceleration*. IEEE CLUSTER 2026, pp. 425–436. DOI: 10.1109/CLUSTER68337.2026.00046.

## Speaking plan

| Presenter | Slides | Section | Duration | Cumulative window |
|---|---|---|---|---|
| Speaker 1 | 1–3 | Background | 2:30 | 0:00–2:30 |
| Speaker 2 | 4–7 | Method | 3:30 | 2:30–6:00 |
| Speaker 3 | 8–11 | Results and conclusions | 3:00 | 6:00–9:00 |
| All speakers | 12 | Questions and discussion | 1:00 | 9:00–10:00 |
| As needed | 13–15 | Backup material | Optional | During discussion only |

Read only the spoken paragraphs. Timing includes short pauses and handoffs. Rehearse to the section boundaries. Slides advance manually.

## Rehearsal notes

- Finish Speaker 1 at 2:30, Speaker 2 at 6:00, and Speaker 3 at 9:00.
- Keep the two handoff sentences in the script.
- Read IBSA as “I-B-S-A,” AMX as “A-M-X,” and lDDT as “L-D-D-T.”
- Say “three point five nine times faster,” not “three hundred and fifty-nine percent faster.”
- All measurements come from the supplied paper. Diagram block patterns are illustrative. Proposed limitations and future tests are our interpretation.

## Slide 1 — SparseFold

**Speaker 1 · Background · 0:00–0:20 · 0:20**

Hello everyone. Today we will present SparseFold, a training-free approach to accelerating AlphaFold 3 on CPUs. I will introduce the background, our second speaker will explain the method, and our third speaker will discuss the experiments and conclusions. We will leave the final minute for questions.

Source: Ren, You & Yang · IEEE CLUSTER 2026 · pp. 425–436

## Slide 2 — Accurate structures. Expensive inference.

**Speaker 1 · Background · 0:20–1:20 · 1:00**

AlphaFold 3 predicts the structures of proteins and their interactions with molecules such as DNA, RNA, and ligands. This makes it useful for studying biological mechanisms and molecular interactions. But accurate predictions can still be expensive to produce. The workflow begins with preprocessing, including sequence alignments and template searches. During inference, an input embedder creates molecular representations. The recycling stage repeatedly refines them through the Pairformer. A diffusion module then generates atomic coordinates, and a confidence module evaluates the predictions. SparseFold focuses on the inference stage, especially the expensive attention inside the Pairformer. It does not accelerate the sequence search or template search. The motivation is practical: how can researchers using CPU servers obtain predictions faster while keeping the structures reliable?

Source: Paper §II-A, Fig. 2 · pp. 426–427

## Slide 3 — The bottleneck is cubic

**Speaker 1 · Background · 1:20–2:30 · 1:10**

The main bottleneck is triangle self-attention. Standard attention over N tokens has quadratic cost. In the Pairformer, however, the model maintains an N by N matrix of pair features. It performs attention over N tokens for each of N rows or columns. That adds another factor of N, giving cubic complexity. Doubling the sequence length can therefore make this operation much more expensive. The paper's profiling shows that recycling takes at least ninety percent of inference time, and the Pairformer takes roughly eighty percent of recycling time. Triangle self-attention becomes increasingly dominant as inputs grow. The opportunity is that many attention blocks contribute very little. The difficulty is that useful interactions can be distant and can change during refinement, so a fixed local window is not enough. The authors therefore aim to identify important blocks cheaply and compute attention selectively. Our second speaker will now explain how they do this.

Source: Paper §II-B–III, Fig. 3 · p. 427

## Slide 4 — A small map guides the expensive computation

**Speaker 2 · Method · 2:30–3:05 · 0:35**

Thank you. The central method is Index-based Block Sparse Attention, or IBSA. Think of it as using a small map to guide a much larger computation. First, it summarizes query and key blocks. Second, it uses these summaries to identify promising attention blocks. Third, a sparse FlashAttention kernel evaluates only the selected blocks at full resolution. The model uses its existing pretrained parameters, so there is no additional training. The matrices here are schematic illustrations of this process.

Source: Paper §IV-A, Fig. 4, Algorithms 1–3 · pp. 427–429

## Slide 5 — Keep the strongest. Sample the uncertain.

**Speaker 2 · Method · 3:05–4:05 · 1:00**

Let us look more closely at block selection. IBSA averages the tokens inside each query and key block to obtain summary vectors. Their dot products, followed by softmax, give a coarse attention map. For each query block, the algorithm ranks these scores and retains up to S candidates. Cumulative probability thresholds divide the candidates into a static set and a dynamic pool, while low-scoring candidates are pruned. The static set contains the strongest blocks and is always included. The dynamic set is chosen by weighted random sampling, so candidates with higher scores are more likely to participate. This is different from uniformly sampling arbitrary positions. The purpose is to preserve dominant interactions while still giving other potentially useful dependencies a chance. The final selected indices tell the attention kernel exactly which key and value blocks to visit.

Source: Paper §IV-A2, Algorithms 1–2 · pp. 428–429; schematic values are illustrative

## Slide 6 — Reuse the map. Resample the route.

**Speaker 2 · Method · 4:05–5:05 · 1:00**

Finding the indices also costs time. In the paper, this overhead reaches up to sixteen percent of end-to-end runtime at the largest tested input. The authors observe that summary attention maps remain similar across recycling iterations. They exploit this temporal stability through index reuse. In the first recycling iteration, each of the forty-eight Pairformer layers computes and stores its own index map and weight map. In iterations two through eleven, each layer reuses the maps produced by that same layer. Crucially, it does not freeze the final sampled sequence. Weighted random sampling runs again each time. The diagram shows static blocks staying the same while a dynamic block can change. This reduces repeated index computation and avoids permanently excluding every block outside the static set. It is reuse across recycling iterations, not one shared map for all layers.

Source: Paper §IV-B, Fig. 6 · pp. 429–430

## Slide 7 — Make sparsity fast on real CPUs

**Speaker 2 · Method · 5:05–6:00 · 0:55**

The algorithm also needs an efficient implementation. SparseFold uses thirty-two by thirty-two attention tiles and an AMX-optimized CPU backend. Work is distributed over sixty-four physical cores. Most computation uses BF16, with FP32 accumulation for the attention output. The kernel shares query, key, and value projections, and fuses attention with surrounding operations to reduce memory traffic. There is an important qualification to the complexity claim. With a bounded number of selected blocks, the selected attention work becomes approximately quadratic across all pair rows. However, building the summary map still costs time, which is why reuse matters. We should not interpret this as a proof that every part of AlphaFold 3 becomes quadratic. The overall approach combines selection, reuse, and CPU engineering. Our third speaker will now evaluate the evidence.

Source: Paper §IV-A3–C · pp. 428–430; complexity qualification from §IV-B

## Slide 8 — What was actually measured?

**Speaker 3 · Results & conclusions · 6:00–6:35 · 0:35**

The main benchmark uses five molecular systems, ranging from five hundred and twelve to two thousand and forty-eight padded tokens. All CPU methods run on the same sixty-four physical cores. Timing averages five warm inference runs and excludes preprocessing and compilation. There are three CPU comparisons: SF-torch, the dense native backend, and an Intel-optimized xfold fork. JAX AlphaFold 3 is an accuracy reference, not a CPU timing baseline. Structural quality is measured against true structures using TM-score and lDDT.

Source: Paper §V-A–C, Table II · pp. 430–432

## Slide 9 — The speedup depends on the baseline

**Speaker 3 · Results & conclusions · 6:35–7:30 · 0:55**

The headline result is up to three point five nine times faster inference, but the baseline matters. Across the five inputs, SparseFold is one point nine six to three point five nine times faster than SF-torch. Against SF-native, which already has dense C++ and AMX optimizations, the speedup is one point three five to two point eight six. That comparison is especially useful because it isolates the benefit of enabling IBSA on the native backend. Against the optimized xfold implementation, the improvement is smaller: one point two two to one point five seven times. Gains generally become stronger for larger inputs. Index reuse provides an additional speed ratio of roughly one point zero six to one point one six compared with no reuse. These are warm inference measurements on the reported server, not universal speed guarantees.

Source: Paper §V-C, Fig. 8b,d · pp. 432–433; ranges are reported, not digitized

## Slide 10 — Speed is useful only if quality survives

**Speaker 3 · Results & conclusions · 7:30–8:20 · 0:50**

The accuracy results are encouraging, but they need a careful reading. In the five-target benchmark, SparseFold's TM-scores remain close to the JAX and SF-torch results. Its reported lDDT values range from zero point eight zero eight to zero point nine nine two, supporting preservation of local geometry as well as global topology. Reusing indices causes no significant score difference in the reported ablation. The authors also test one hundred and forty-one CAMEO monomer targets. Their TM-score distribution largely overlaps with JAX AlphaFold 3, although there is some accuracy loss in the lower-scoring region. Finally, the sensitivity analysis shows that excessive pruning can damage predictions, while dynamic sampling helps complex cases. So the evidence supports a useful operating point, not lossless sparsity under every setting.

Source: Paper §V-B,D–E, Figs. 7, 9–12 · pp. 431–434

## Slide 11 — Spend computation where it matters

**Speaker 3 · Results & conclusions · 8:20–9:00 · 0:40**

To conclude, SparseFold combines three ideas: choose useful attention blocks, reuse stable candidate maps, and execute the sparse work efficiently on CPUs. The result is faster inference with broadly preserved structural quality in the reported experiments. There are still limits. Performance was measured on one CPU platform; the custom benchmark is small; and the larger CAMEO test contains monomers. Preprocessing is outside the speedup, and more aggressive sparsity can reduce accuracy. Our takeaway is that model-specific sparsity and hardware-aware execution can work together effectively. Thank you. We are happy to take questions.

Source: Paper §V–VII · pp. 430–435; limitations are our interpretation of the evaluation scope

## Slide 12 — Questions & discussion

**All speakers · Discussion · 9:00–10:00 · 1:00**

Invite one or two short questions. Speaker 1 answers questions on AlphaFold 3 and the motivation. Speaker 2 answers questions on selection, complexity, index reuse, and implementation. Speaker 3 answers questions on evaluation, baselines, quality, and limitations. Use the backup slides only if relevant.

Source: Ren, You & Yang · SparseFold · IEEE CLUSTER 2026 · DOI: 10.1109/CLUSTER68337.2026.00046

## Slide 13 — Backup: what does quadratic mean here?

**Speaker 2 · Backup · Optional backup**

Suggested answer: The quadratic statement applies most directly to the selected full-resolution attention work when the block budget is fixed. Each pair row visits a bounded number of key blocks for every query block. Across N pair rows, that gives quadratic work. However, the pooled score map still compares all summary query and key blocks. With a fixed block size, that initial map construction can retain a cubic term across the effective batch dimension, although its constant is much smaller. Reuse reduces how often this map is rebuilt. We therefore distinguish the selected attention computation from the total asymptotic cost of the complete pipeline.

Source: Paper §IV-A–B · pp. 427–430; derivation clarifies the scope of the complexity claim

## Slide 14 — Backup: the default operating point

**Speaker 2 · Backup · Optional backup**

Suggested answer: S limits the candidate list to sixteen blocks. The cumulative-score thresholds separate strong static blocks from a remaining dynamic pool. R controls the dynamic sampling budget, but the actual number of samples depends on the remaining probability mass and candidate count. Therefore, R equal to six does not mean six additional blocks outside the S candidates. The default thresholds are zero point nine and zero point nine five, and the pruning threshold is zero point zero two. The sensitivity experiments show that stronger pruning improves speed but can sharply degrade accuracy.

Source: Paper Algorithms 1–2, §V-B,D · pp. 429, 431, 433–434

## Slide 15 — Backup: claims, evidence, and open questions

**Speaker 3 · Backup · Optional backup**

Suggested answer on scope: This is CPU inference acceleration. The paper uses GPUs only to obtain JAX accuracy references. Suggested answer on training-free: The method changes the attention computation without optimizing new model weights; it still uses selection hyperparameters. Suggested answer on reliability: The results support broadly preserved quality at the default settings, but do not prove identical outputs or universal accuracy. Suggested answer on future work: I would test more heterogeneous complexes, additional hardware platforms, and several random seeds, and report more interface- or ligand-specific metrics. These are our suggested extensions, not experiments reported in the paper.

Source: Paper §V, VII · pp. 430–435; critical discussion by the presentation group

## Additional Q&A preparation

**Q: Does training-free mean parameter-free?**

A: No. No model weights are retrained, but the sparse attention algorithm still uses block sizes, candidate budgets, sampling parameters, and pruning thresholds.

**Q: Why compare predicted structures with true structures?**

A: The paper uses true structures to avoid treating baseline prediction errors as ground truth. Regions without experimentally resolved coordinates can also be excluded from the structural comparison.

**Q: Does the 3.59× result include MSA search?**

A: No. It measures warm forward inference on preprocessed inputs. Data preprocessing and initial compilation are excluded.

**Q: Why is xfold sometimes faster for attention but slower overall?**

A: Its hand-tuned kernels have low overhead for short sequences. SparseFold also optimizes surrounding operations and the broader model, so its end-to-end timing can still be better.

**Q: Are the random results reproducible?**

A: The paper reports deterministic structural scores for fixed seeds and hyperparameters. Dynamic sampling still occurs; fixing the seed makes the pseudorandom sequence reproducible. This does not establish robustness across many seeds.

