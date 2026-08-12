# Title: Calibration of Tiny Neural Networks Under Compute Constraints

## Keywords
calibration, uncertainty quantification, label smoothing, temperature scaling, small models

## TL;DR
When and why do simple calibration interventions (temperature scaling, label smoothing, small ensembles) help tiny neural networks, and how does the answer change under distribution shift?

## Abstract
Modern uncertainty-quantification research focuses on large models, but many deployed models are small classifiers trained on modest data. This work studies confidence calibration in deliberately tiny neural networks (MLPs and small CNNs with under one million parameters) trained on small image and tabular datasets. We ask which of the standard, nearly-free calibration interventions (temperature scaling, label smoothing, deep ensembles of two to four members, and mixup) most improves expected calibration error in-distribution, and whether the ranking of these interventions is preserved under simple distribution shifts such as image corruptions and label noise. All experiments must run in minutes on CPU or at most a small GPU slice; datasets should be small (MNIST, FashionMNIST, or UCI tabular tasks, subsampled to at most ten thousand training examples) and models must remain under one million parameters. The contribution is a controlled empirical map of when cheap calibration methods succeed or fail at small scale, with practical recommendations.
