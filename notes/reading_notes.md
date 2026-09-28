# Reading notes

For each paper: (1) what the experiment actually changed, (2) what conclusion would go beyond its evidence, (3) what control it suggests for this project.

## Srivastava et al. (2014), Dropout — Sections 7.3–7.4, Figure 10

1. **What changed:** They changed the number of MNIST training examples and compared training with and without dropout. They kept the dropout settings fixed in this experiment.
2. **Overreach:** "Smaller datasets always need more dropout." Dropout did not help at the smallest sizes, and this experiment did not compare different dropout rates.
3. **Control for my project:** Include a no-dropout baseline at every dataset size. Use the same training examples when comparing dropout rates.

## Sharma et al. (2024), Guided Dropout

1. **What changed:** They reduced training-set size and compared three dropout methods when fine-tuning BERT. Guided dropout assigns different dropout rates to layers based on estimated sensitivity.
2. **Overreach:** "Higher ordinary dropout works better on smaller datasets." Their comparison tested dropout methods, rather than a range of ordinary dropout rates.
3. **Control for my project:** Specify exactly where dropout is applied. In a later experiment, change classifier dropout while holding encoder dropout fixed, then reverse the comparison.

## Mosbach et al. (2021), On the Stability of Fine-tuning BERT

1. **What changed:** They investigated dataset size, training duration, and optimizer settings across repeated BERT fine-tuning runs. Their experiments showed that too few training updates could explain apparent small-data instability.
2. **Overreach:** "Dataset size does not matter for generalization." Their finding concerns training stability under the tested conditions, not whether extra data improves accuracy.
3. **Control for my project:** Repeat selected comparisons with the same number of optimizer updates across dataset sizes. This helps check whether a result reflects less data or simply less training.

## Open questions

- Figure 10: how many runs do the error bars come from, and how long was each dataset size trained?
