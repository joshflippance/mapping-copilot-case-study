# Mapping Copilot: a case study in making AI output safe enough to trust

An AI copilot that turns a legacy healthcare feed into a reviewable, deterministic integration mapping, so a vertical SaaS vendor can onboard a new EHR customer in days instead of months.

Mapping Copilot is the AI feature inside Plumb, an integration platform you can try in the browser at [integrationplatform.vercel.app](https://integrationplatform.vercel.app). Synthetic data only, zero real patient data ever. The code is private. This repo is the write-up.

---

## The problem I already lived

I spent a year at MealSuite running data integrations, and cut EHR integration delivery from 65 business days to under 30. Most of that came from process: better scoping, better sequencing, better handoffs.

The part that would not move was the mapping itself. For every new customer, an analyst sits down with that facility's HL7 v2 feed and hand-translates it into the vendor's own schema, field by field. Weeks of work and five figures of cost, repeated per customer, forever.

Vendors serving long-term care and senior living wear this worst. They have the same integration problem as a hospital system and none of the budget. No product serves that segment below the incumbent network platforms' price floor, and the AI integration copilots that do exist are built for hospital integration teams.

## The bet

An AI can generate the first-pass mapping. A human approves every field. The runtime stays deterministic and no model ever touches a live message.

That third clause is the whole thing. It is easy to build a demo where a model transforms messages on the fly and the output looks right. It is not defensible, because you cannot tell a customer what their data will do tomorrow, and you cannot reproduce what it did yesterday.

So the model's job ends before production begins. It writes an artifact. The artifact is a declarative, versioned, git-diffable document with per-field confidence, the model's rationale, and an explicit list of what it could not map. A human reviews it. A deterministic engine executes the approved version.

## The number that matters

Every mapping is scored against machine readable ground truth across ten synthetic HL7 feeds. Three runs, labelled for what they are:

| Run | Silent errors | Wrong fields surfaced | Field accuracy | Held for review |
|---|---|---|---|---|
| Offline reference mapper: a pipeline check with no model | **0%** | **100%** (220 of 220) | 91% | Not reported |
| Live model, calibrated, on a fresh run never used for tuning | **0.5%** (4.7% with the model's own confidence) | **99%** (87% raw) | about 63 to 65% | 46% |
| Live model, CSV census exports, same calibration | **0.5%** | **100%** | 91% | 18% |

The first row checks that the pipeline, the review gate and the scoring work, without any AI in the loop. The second and third rows are the live model.

Accuracy is the number people ask for. It is not the number that matters.

A wrong value that nobody flagged is the only failure that reaches a live system unnoticed. Everything else is a delay: a flagged field costs an analyst two minutes, a failed validation costs a re run. A silently wrong diet order on a patient with an allergy is a different category of event.

So the product is not optimised for accuracy. It is optimised so that when the model is wrong, the wrongness is visible. The live model's own confidence turned out to be overconfident: on its raw scores it auto accepted wrong fields 4.7% of the time. Replacing that score with evidence based calibration, built from signals measurable without ground truth, brought silent errors to 0.5% on feeds it never saw. Calibration changes which errors a person sees, not how many the model makes.

What calibration does not fix: the remaining silent errors are one plausible looking pattern in a single feed, where the model reversed the component order of a combined location string. Running the model five times and voting does not catch it either; most wrong mappings are unanimous across runs.

The full measurement design, including how the benchmark is built so I cannot fool myself with it, is in [docs/measurement.md](docs/measurement.md).

## Designing so the human stays in charge

Three rules, and I held all three when they were inconvenient.

**The LLM generates the artifact; the runtime is deterministic.** Never a model per message. Same message plus same artifact produces byte-identical output, verified on every re-run.

**Human review is a gate, not a suggestion.** Fields below the confidence threshold and every unmapped field must be resolved before the deploy button unlocks. This was the tempting one to soften, because an enforced gate is slower and a reviewer who has approved forty mappings will want to skip the forty-first. Advisory review is how you end up with an audit finding.

**Nothing is silently dropped.** Any message that would produce a schema-invalid or incomplete record goes to an error queue with reasons. Every processed message is written to an immutable audit log tied by content hash to the exact artifact version that transformed it. When someone asks in six months why a record looks the way it does, the answer is retrievable rather than reconstructed.

Runtime results across the ten feeds: 270 messages processed, 261 delivered, 332 records emitted, 9 to the error queue with reasons, determinism verified byte-identical on every feed.

## What the analyzer does before the model sees anything

The first-pass mapping is only as good as what the model knows about the feed. So the pipeline profiles it first: message types, segment inventory, field population rates, value patterns, and an anomalies list.

On the worst synthetic feed the analyzer flags, unprompted:

- Two undocumented Z-segments, likely facility info and diet orders
- Local MM/DD/YYYY dates, which are not HL7 and must be normalised first
- A name field that never uses component separators, so "FIRST LAST" needs splitting
- Allergy data buried in free-text notes, requiring extraction
- A patient-visit field whose first component is numeric, suggesting swapped component order

That list is the archaeology an analyst would otherwise do by hand over several days. It is also the on-ramp: the profile, not the raw feed, is what the model reasons over.

The seven stages end to end are in [docs/pipeline.md](docs/pipeline.md).

## What I would tell a skeptic

The results are on synthetic data, produced by a generator I also wrote. That is a real limitation and I have not pretended otherwise anywhere in the project. What it buys is machine-readable ground truth for every single field, which is the only way to measure a silent error rate at all. Real feeds would give me realism and no ground truth.

The honest next step is not more sprints. It is one real vendor with one real feed and a signed agreement, scoring the same three metrics against a mapping an analyst produced by hand.

The decisions, each with the alternative I rejected, are in [docs/decisions.md](docs/decisions.md).

## How this was built

I directed Claude Code through the build across six sprints, and made the product calls myself: what the artifact format has to guarantee, where the review gate sits, which metric leads the benchmark, and what counts as an honest failure.

The most useful thing I did was refuse the easy version twice. Once when an LLM-per-message pipeline would have demoed better and been indefensible. Once when a bare accuracy number would have looked stronger than the metric I actually report.

---

## In this repo

| File | What it covers |
|---|---|
| [docs/pipeline.md](docs/pipeline.md) | The seven stages, what each one produces, and why the order matters |
| [docs/measurement.md](docs/measurement.md) | The benchmark: blind design, leakage check, honest failure, variance, calibration |
| [docs/decisions.md](docs/decisions.md) | Design decisions with the alternative rejected and the reason |

All companies, facilities, people and schemas referenced in this project are fictional. No HIPAA compliance is claimed; the architecture is designed to be BAA-ready and is not certified.

Josh Flippance · [joshflippance.com](https://joshflippance.com) · [LinkedIn](https://linkedin.com/in/joshflippance)
