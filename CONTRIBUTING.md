# Contributing

Keep all executable modeling, training, evaluation, and figure-generation code in `laterally_loaded_piles.ipynb`. Keep notebook explanations, comments, documentation, and plot text in English. Use the paper's symbols in prose and corresponding readable variable names in code.

The current implementation uses physics-only training. Propose broader boundary conditions, nonlinear soil models, or supervised training separately.

Before submitting a change:

1. Run the notebook from a clean kernel in `smoke` mode. This exercises training and checks finite gradients without a full training run.
2. Run the default `pretrained` mode when changing plots, metrics, or predictions. Inspect the PNG figures for clipped labels, incorrect units, and inverted depth axes.
3. Explain the change, its reason, and the validation performed. Include before/after figures when changing presentation.
4. Preserve the distinction between original FEM output, digitized paper values, PINN predictions, and new training histories. Never relabel predictions as FEM data.

Do not overwrite the bundled model or archived history as part of a routine notebook change. If source assets change, document their provenance, update their hashes, and regenerate the outputs. Do not add full copyrighted papers, local paths, private helper modules, or environment-specific kernel definitions.

Bug reports should include the operating system, Python and package versions, run mode, failing cell, and complete error text. If reporting a numerical difference, include the `PileCase` parameters.
