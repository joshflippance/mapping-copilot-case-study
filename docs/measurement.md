# Measurement

The benchmark is the part of this project I care most about, because a mapping tool that cannot prove its output is safe is a demo.

## Metric order is a product decision

Reported in this order, deliberately:

1. **Silent error rate.** A wrong value that nobody flagged. The only failure that reaches a live system unnoticed.
2. **Error-surfacing rate.** Of the fields that were wrong, how many were caught for review.
3. **Field accuracy.** How many fields were right.
4. **Confidence calibration.** Whether the fields the model flagged as uncertain really are the wrong ones.

Accuracy is third on purpose. It is the number a reader reaches for and the number that tells you least about whether the system is safe to run. A system at 97% accuracy with 3% silent errors is more dangerous than one at 91% with none, because the first one has thirty wrong values per thousand loose in production and no idea which ones.

Calibration is fourth and is the one that would quietly invalidate the other three. If the model's confidence scores do not correlate with correctness, then the review gate is filtering on noise and the error-surfacing rate is luck.

## Rules that make the numbers defensible

**Blind.** The model only ever sees the feed profile, the target schema, and sample messages. Never the ground truth.

**Leakage check.** The harness scans every prompt before it is sent and aborts the run if any ground-truth answer value appears in it. Without this, an accuracy number tells you the model read the answer. This check is the difference between a benchmark and a self-congratulation.

**Honest failure.** A generation that errors, or returns unparsable output, is reported as a failure. It is never silently swapped for the known-good reference artifact. The reference generator exists so the pipeline runs end to end without an API key, and that is exactly why it is a tempting thing to fall back on: it would make every run look complete.

**Variance.** The model is non-deterministic, so the benchmark runs five times by default and reports mean and range. One run is an anecdote. Reporting a single number from a stochastic system is the most common way benchmarks mislead, usually without anyone intending it.

**Labelled baselines.** A run with no API key produces the offline reference baseline and says so in the output. It is never presented as a live result.

## What the numbers do not cover

The data is synthetic and I wrote the generator. The feeds are modelled on real patterns of messiness, but I chose which patterns and how much of each, which means the difficulty is my invention.

What synthetic data buys is ground truth for every field, and there is no other way to compute a silent error rate. Real feeds would give realism and nothing to score against.

The honest next step is a real feed from a real vendor, scored the same way against a mapping an analyst produced by hand. Everything before that is a proof that the measurement approach works, not a proof that the accuracy holds.
