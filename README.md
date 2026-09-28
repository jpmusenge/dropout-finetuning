# Dataset Size × Dropout During Transformer Fine-Tuning

**Status: in progress.** The experiment pipeline is built and tested. Results from the first pilot (36 runs + 6 control runs) will be added here as they come in. Nothing below is a finding yet.

## The question

Srivastava et al. (2014) showed that when a network is trained **from scratch**, how much dropout helps depends on how much training data there is. On MNIST, dropout did not help at the smallest dataset sizes (at 100 examples it did slightly worse than no dropout), helped most at middle sizes, and helped less again with lots of data.

This project asks what that relationship looks like when you **fine-tune a pretrained model** instead:

> How does training-set size change the effect of dropout on held-out accuracy when fine-tuning pretrained DistilBERT on SST-2, under a fixed training recipe?

A second question: does the pattern hold up when the small datasets get the same number of training steps as a larger one?

Any outcome is acceptable: dropout helps, hurts, does nothing measurable, or changes direction across sizes. Predictions are written down before the main runs (`notes/protocol_v1.md`).

## Design

| | |
|---|---|
| Model | `distilbert/distilbert-base-uncased` with a new 2-class head; all weights trained |
| Data | SST-2 (movie-review sentiment), 67,349 training rows |
| Training sizes | 256, 1,024, 4,096, 16,384 (each 4× the last) |
| Dropout p | 0.0, 0.1, 0.3, set on all three DistilBERT dropout settings (embeddings/feed-forward, attention, classifier head) |
| Repeats | 3 paired repeats per setting → 36 runs |
| Recipe | AdamW, lr 2e-5, weight decay 0.01, batch 16, 3 epochs, linear schedule with 10% warmup, max length 128 |
| Evaluation | Accuracy on 436 held-out examples, using the model at the end of training |

**Controls built into the design**

- **Held-out data never touches training or tuning.** SST-2's labeled validation set is split once into a *dev* half (debugging only) and an *eval* half (final numbers only). I don't carve a dev set from the training data, because SST-2's training set contains phrases cut from the same sentences, which would leak.
- **Nested, class-balanced subsets.** The 256 training rows are inside the 1,024, which are inside the 4,096, and so on, so a larger set only adds data. Row IDs are saved.
- **Paired comparisons.** Within a repeat and size, every dropout rate uses the same training rows and the same seed, so dropout is the only thing that changes. The key result is the *paired difference* from p = 0.
- **Dropout is verified in the actual layers**, not just the config. (DistilBERT silently ignores BERT's `hidden_dropout_prob`, a bug in my first plan that would have produced normal-looking but wrong results.)
- **Step-matched check.** With a fixed 3 epochs, bigger datasets also get more training steps (48 vs. 3,072). A separate 6-run check trains n = 256 for 768 steps to see how much of any effect is really training length (the issue raised by Mosbach et al., 2021). It is plotted separately and never mixed into the main curve.
- **No runs dropped.** Every run, including failed or near-chance ("degenerate") runs, is logged with its full config.

## What this can and can't show

- It can describe how dropout behaves when fine-tuning DistilBERT on SST-2 under this recipe.
- It **can't** show that *pretraining* is the reason for any difference from the 2014 results, because the model, task, and data also differ. That needs a pretrained-vs-random-init comparison on the same architecture (planned follow-up).
- With 3 repeats and 436 eval examples (one example = 0.23 points), results are preliminary. Spread is reported as standard deviation across repeats, not as significance.
- Dropout during fine-tuning has been studied before (e.g., Mixout, 2020; guided dropout, 2024). This is a small, careful replication-style study, not a first.

## Results

*Pending.* Planned figures:
1. Eval accuracy vs. training size, one line per dropout rate, each repeat shown.
2. Paired accuracy difference from p = 0 at each size (the key plot).
3. The step-matched check at n = 256.
4. Training-loss curves, to spot broken or surprising runs.

## Repository layout

```text
notebooks/02_experiments.ipynb   # the full pipeline: data, subsets, model, runs, analysis
notes/reading_notes.md           # what the key papers did and didn't show
notes/protocol_v1.md             # locked settings + predictions, written before the main runs
notes/research_log.md            # short log per work session
configs/  data_manifests/  results/  figures/   # filled in by the notebook
```

## Reproduce

1. Open `notebooks/02_experiments.ipynb` in Google Colab and select a T4 GPU.
2. Run all cells. Outputs are saved to Google Drive under `MyDrive/dropout-finetuning/`; finished runs are skipped if the notebook is re-run after a disconnect.
3. Every run can be rebuilt from `configs/protocol_v1.json`, the saved subset row IDs in `data_manifests/`, and its line in `results/runs.jsonl`.

## Next steps

1. **Pretrained vs. random initialization** on the same small architecture (BERT-Mini), to test whether *pretraining* changes how dropout depends on data size.
2. **Dropout placement:** vary encoder dropout with the classifier head fixed, and the reverse.
3. **Generalization:** rerun key settings on a second task or model.

## References

1. Srivastava et al. (2014). [Dropout: A Simple Way to Prevent Neural Networks from Overfitting](https://jmlr.org/papers/volume15/srivastava14a/srivastava14a.pdf). JMLR. Sections 7.3–7.4, Figure 10.
2. Lee, Cho & Kang (2020). [Mixout](https://arxiv.org/abs/1909.11299).
3. Sharma et al. (2024). [Information Guided Regularization for Fine-tuning Language Models](https://arxiv.org/abs/2406.14005).
4. Mosbach, Andriushchenko & Klakow (2021). [On the Stability of Fine-tuning BERT](https://arxiv.org/abs/2006.04884).
5. Sanh et al. (2019). [DistilBERT](https://arxiv.org/abs/1910.01108).
6. Socher et al. (2013). [Recursive Deep Models for Semantic Compositionality Over a Sentiment Treebank](https://aclanthology.org/D13-1170/) (SST).

Author: Joseph Musenge
