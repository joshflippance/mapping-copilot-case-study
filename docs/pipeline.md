# The pipeline

Seven stages. The ordering is the design: each one narrows what the next has to figure out.

## 1. Synthetic feed generation

Ten legacy feed variants, each with its own brand of messiness: undocumented Z-segments, missing fields, swapped components, local date formats, combined name fields, allergies buried in free text. Admit/transfer/discharge messages and diet orders.

Every message ships with machine-readable ground truth. This is the part that makes everything downstream measurable, and it is why the project starts here rather than with the interesting AI work.

## 2. Source analysis

Parse the feed and profile it: message types, segment inventory, field population rates, value patterns, and an anomalies list that names exactly what makes this feed hard.

The anomalies list is the payoff. It is the archaeology an analyst would otherwise do by hand, and it is what the model reasons over in the next stage. The model never sees the raw feed dump; it sees a structured profile plus samples.

## 3. Mapping generation

The only place a language model is used in the entire system.

Prompt is assembled from the feed profile, the target schema, and sample messages. Output is a mapping artifact.

## 4. The artifact

The contract between the model and everything that follows:

- **Declarative.** It describes the mapping, it does not execute it.
- **Versioned and git-diffable.** Two mappings can be compared as text, and a change is reviewable like a code change.
- **Per-field confidence and rationale.** Every field carries how sure the model was and why it chose what it chose.
- **An explicit unmapped list.** Anything it could not map is named. Nothing is silently guessed.

The unmapped list matters more than it looks. A model asked to produce a complete mapping will produce a complete mapping, and the gaps become plausible-looking wrong answers. Making "I could not map this" a first-class output is what turns a guess into a flag.

## 5. Human review

A self-contained review console. For each proposed field mapping the analyst sees the confidence, live sample values pulled from the actual messages, and the model's rationale, then approves, edits, or rejects.

The gate: fields below the confidence threshold and every unmapped field must be resolved before deploy unlocks. Deploying exports a decision log.

Sample values are doing quiet work here. Reviewing a mapping in the abstract is guesswork; reviewing it next to three real values from the feed is a judgement a domain expert can actually make in seconds.

## 6. Deterministic runtime

Executes an approved artifact against a live message stream. Pure and deterministic: same message plus same artifact gives byte-identical output, verified on every re-run. No model, no network in the message path.

Every processed message goes to an immutable audit log tied by content hash to the artifact version that transformed it. Anything that would produce a schema-invalid or incomplete record goes to an error queue with reasons.

## 7. Benchmark

Scores the mappings the live model produced against ground truth, under rules designed to make the numbers hard to fudge. Covered in [measurement.md](measurement.md).

---

## Why this order

The instinct is to start at stage 3, because that is where the AI is. Starting there means you have no way to tell whether the output is any good, so you evaluate it by reading it, which means you evaluate it by whether it looks plausible.

Stages 1 and 2 exist so that stage 3 can be measured. Stages 4 through 6 exist so that stage 3 being wrong is survivable.
