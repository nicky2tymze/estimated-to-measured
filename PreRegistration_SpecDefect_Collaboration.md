# Pre-Registration: Measuring Denominator One (Spec-Review Catch Rate)

**Author:** Dominick Trolian
**Destination repo:** `nicky2tymze/estimated-to-measured`
**Status:** PRE-REGISTRATION. Frozen and committed before receipt of the collaborator's injection set.
**Collaborator:** Kartik Raghavan (supplies the spec-defect grammar and per-tier catch predictions).
**Date committed:** _set at commit; the commit timestamp and hash are the registration record._

---

## 0. Why this document exists

`estimated-to-measured` puts a sampled denominator under two of the three quantities people call a catch rate. It does not yet measure the third. This collaboration measures the third. Before the collaborator's injection set arrives, the metric, the terminology, and the scoring rules are frozen here. Nothing in this file is editable after the injection set is received. That is the whole point of pre-registering it: the measurement cannot be tuned to the data.

Two ground rules were agreed in the public thread. Both are honored here:

1. Agree on terminology up front. Section 1.
2. Pre-register the weak-implementer metric before the run. Section 4.

---

## 1. Terminology (locked)

Three different quantities get called a "catch rate." They are kept distinct, using the same language as the published writeup.

- **Denominator one — review catch-rate against injected spec defects.** A spec carries known, deliberately injected defects. A review process is run against the spec. The catch rate is the fraction of injected spec defects the review flags. This is the quantity this collaboration measures.
- **Denominator two — probe completeness against injected implementation mutants.** An auto-generated behavior probe is run against an implementation carrying known mutants. The kill rate is the fraction of mutants the probe kills. Already measured in `estimated-to-measured`.
- **Denominator three — independent-build divergence as a spec-hole detector.** Several models implement the same spec; their outputs are diffed; each disagreement is a question the spec failed to answer. Already measured.

This program previously measured two and three and stated plainly that it did not measure one. The collaborator's grammar is what makes one measurable: a set of injected spec defects with ground-truth presence supplies the denominator that was missing.

"Catch rate," unqualified, is not used in any result. Every reported number names which of the three it is.

---

## 2. What is measured

**Unit under test:** a review configuration (a model plus a review prompt at a named depth tier).

**Material under test:** a spec carrying a known set of injected spec defects supplied by the collaborator. Each injected defect has a defined, objective detection criterion (Section 3).

**Measurement:** for each review configuration, the fraction of injected spec defects whose detection criterion is satisfied by that configuration's review output. This is denominator one.

**Review depth tiers** (from the collaborator's ladder; final names fixed at terminology sign-off, Section 9):
bare prompt, scope-only, comprehensive, specialist-passes-plus-synthesis, multi-model.

The collaborator supplies a predicted catch per tier for each injected defect. Those predictions are hypotheses under test. They are not used in scoring. See Section 3.

---

## 3. Ground-truth contract

This is the rule that keeps the measurement honest.

- Each injected spec defect carries an **objective detection criterion** authored before any review is run: a concrete, checkable statement of what a review output must contain to count as having caught that defect. Example shape: "names the foreign key to a table absent from the schema," not "mentions schema problems."
- A defect is scored caught for a configuration if and only if that configuration's review output satisfies the defect's detection criterion. Catch is scored **against the defect's criterion, not against the collaborator's predicted catch rate.**
- The collaborator's per-tier predictions are sealed at receipt and revealed only in analysis (Section 8). The rig operator does not see the predictions while authoring or applying detection criteria. The predictions never enter the score.
- Scoring of a review output against a criterion is itself specified before the run: criterion application is a literal match against the stated condition, adjudicated blind to which tier or model produced the output where feasible.

Rationale: the collaborator's predicted catch per tier is the thing being tested. Grading the rig against his predictions would measure agreement with an estimate, which is the exact substitution this whole line of work exists to remove. The ground truth is the injected defect and its criterion, decided in advance.

---

## 4. The weak-implementer metric (pre-registered)

The intervention claim is that richer review configurations lift catch rate above a weak baseline. Every effect size is measured relative to that baseline, so the baseline is pre-registered and frozen.

**Definition.** The weak-implementer metric is the denominator-one catch rate of a single designated baseline review configuration, run alone, against the injected spec-defect set, computed as the fraction of injected defects whose detection criterion it satisfies, sampled over K independent review generations (Section 5) and reported as the mean across those generations.

**Designated baseline configuration:** the bare-prompt tier on the designated weak reviewer model. The specific model and the exact bare-prompt text are fixed at terminology sign-off (Section 9) and frozen there, before the injection set is received.

**Frozen choices:**
- The baseline is the bare-prompt tier. It is not re-selected after seeing results, and it is not redefined as whichever configuration happens to score lowest.
- The catch-rate computation is the fraction defined above. No alternative scoring (weighted by predicted difficulty, restricted to a subset of categories, etc.) is substituted after the fact.
- Effect size for any richer tier is reported as that tier's catch rate minus this frozen baseline, on the same injected set, same criteria, same K.

This is the analog, on denominator one, of the implementer floor reported elsewhere in this program ("weak implementer alone"). The lever under test is whether review structure compensates for reviewer weakness, the same shape as spec structure compensating for implementer weakness.

---

## 5. Sampling and K

- Any stochastic measurement (every per-configuration catch rate, including the baseline) is sampled over **K independent review generations, K at least 40.** A single near-ceiling or near-floor pass is not reported as a point estimate.
- Per-defect catch is reported as a fraction over K, not as a single binary. Per-configuration catch rate is the mean over the injected set of those per-defect fractions.
- K at least 40 is the floor agreed in the thread, chosen because near-1.0 and near-0.0 rates need more samples than a single K equals 20 pass to separate true rate from sampling noise.

---

## 6. Freeze and integrity discipline

- This pre-registration is committed before the injection set is received. The commit timestamp and hash are the registration record.
- On receipt, the collaborator's injection set is hashed and committed unchanged. The hash is recorded. The set is not edited.
- After receipt of the injection set, the following are not edited: this metric, the terminology in Section 1, the detection-criteria authoring rules in Section 3, the baseline definition in Section 4, and K in Section 5.
- If a genuine defect in the rig or the criteria is found after receipt, it is not silently fixed. It is recorded as a deviation, with what changed and why, in a dated amendment appended below this document, and the affected runs are re-run or excluded explicitly. Deviations are reported in the result, not absorbed into it.

---

## 7. Roles and audit surface

- **Collaborator supplies:** the spec-defect grammar (the injection rules), the concrete injected defect instances, and a predicted catch per tier per defect (sealed per Section 3).
- **Rig operator supplies and shares for audit, before the run:** the domain list, the mutant set, and the probe contract, so the rig can be audited rather than trusted. Operator authors the detection criteria and runs the review configurations at K at least 40.
- The collaboration measures denominator one. It does not re-measure denominators two and three; those stand as published.

---

## 8. Analysis plan (fixed before the run)

- Report, per tier, measured denominator-one catch rate with K and sample fractions.
- Report measured catch minus the frozen weak-implementer baseline (Section 4) per tier.
- Report the collaborator's predicted catch per tier alongside the measured catch, and report where the predictions and the measurement disagree, per defect and per tier. Disagreement is a result, not a failure to be smoothed.
- No metric is switched, no tier is re-binned, and no defect is dropped from the denominator after results are seen. Any post-hoc analysis is labeled post-hoc and reported separately from the pre-registered measurement.

---

## 9. Open items to settle at terminology sign-off (before receipt)

These are agreed with the collaborator and frozen into this document before the injection set is received. Until they are filled, the pre-registration is not closed.

- Final names and definitions of the review depth tiers (Section 2).
- The designated weak reviewer model and the exact bare-prompt text (Section 4).
- The size of the injected set (the thread anchored on roughly 30 injections).
- The format of a detection criterion, so criteria are authored consistently across the set (Section 3).

---

## Appendix: one-line glossary

- **Injected spec defect:** a known, deliberately planted flaw in a spec, with ground-truth presence.
- **Detection criterion:** the pre-authored, objective condition a review output must meet to count as catching a given defect.
- **Review configuration / tier:** a model plus a review prompt at a named depth.
- **Weak-implementer metric:** the frozen baseline denominator-one catch rate, bare-prompt tier on the designated weak reviewer, over K at least 40.
- **K:** number of independent review generations sampled per configuration, at least 40.
- **Denominator one / two / three:** review catch against injected spec defects / probe completeness against implementation mutants / independent-build divergence as spec-hole detector.
