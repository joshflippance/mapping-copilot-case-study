# Decisions

Each with the alternative rejected and the reason.

## 1. The model writes an artifact; it never transforms a message

**Chosen:** a language model generates a declarative mapping artifact. A deterministic engine executes the approved artifact in production.
**Rejected:** an LLM in the message path, transforming records as they arrive.
**Why:** the rejected version demos better and cannot be operated. You cannot promise a customer what their data will do tomorrow, you cannot reproduce what it did yesterday, and every message costs a model call. Determinism is verified: same message plus same artifact, byte-identical output, checked on every re-run.

## 2. Silent error rate leads the benchmark

**Chosen:** report silent error rate first, then error-surfacing rate, then accuracy, then calibration.
**Rejected:** leading with accuracy, the way almost every AI mapping tool does.
**Why:** accuracy is the flattering number, not the safety number. Optimising the system so its mistakes are visible produced a better product than optimising for fewer mistakes would have.

## 3. Human review is an enforced gate

**Chosen:** low-confidence and unmapped fields must be resolved before deploy unlocks. Deploy exports a decision log.
**Rejected:** advisory review, where the console shows confidence and the analyst decides how much to read.
**Why:** advisory review works for the first ten mappings and stops working around the fortieth, which is exactly when the volume makes it matter. The slower workflow was the point.

## 4. "I could not map this" is a first-class output

**Chosen:** the artifact carries an explicit unmapped list, and nothing is guessed silently.
**Rejected:** a complete mapping with low-confidence entries for uncertain fields.
**Why:** a model asked for a complete mapping will produce one, and the uncertain fields become plausible wrong answers rather than visible gaps. Naming the gap turns it into review work instead of a defect.

## 5. Profile the feed before the model sees it

**Chosen:** parse and profile first, produce an anomalies list, and give the model the profile plus samples.
**Rejected:** handing raw messages to the model and letting it infer structure.
**Why:** the anomalies list is independently useful, since it is the analyst's manual archaeology, and a structured profile is a far smaller and more reliable prompt than a feed dump. It also means the interesting judgement (this feed hides allergies in free text) is legible to a human before any mapping exists.

## 6. Synthetic data only, and say so everywhere

**Chosen:** a synthetic generator that emits machine-readable ground truth with every message. Fictional companies, facilities, people and schemas throughout.
**Rejected:** sourcing real de-identified feeds.
**Why:** ground truth per field is the only way to compute a silent error rate, and de-identified real data carries risk with no measurement benefit. The cost is stated plainly rather than buried: the difficulty of these feeds is my invention, which is a real limitation on what the numbers prove.

## 7. An offline reference generator, quarantined

**Chosen:** a profile-driven reference generator so the pipeline and benchmark run end to end with no API key, clearly labelled as a baseline and never substituted for a live result.
**Rejected:** requiring an API key to run anything.
**Why:** anyone should be able to clone it and see the pipeline work. The danger is the same feature: a silent fallback would make every failed generation look like a success, so the benchmark reports failures as failures instead.

## 8. Nothing is silently dropped

**Chosen:** an error queue with reasons, and an immutable audit log tying every processed message by content hash to the artifact version that transformed it.
**Rejected:** logging failures and moving on.
**Why:** in healthcare integration the question is never "did it work" but "what happened to this specific record on this specific day." A system that cannot answer that is not deployable regardless of accuracy.
