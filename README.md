# From Estimated to Measured: A Denominator for Probe Completeness

## A small, enumerable map of where specs leak, with a closing rule for each

*A focused preview of one result from a larger body of work.*

---

## The problem: every catch rate is a numerator without a denominator

The usual way to report how good a review or spec process is, is a catch rate. Bare prompt finds 20 to 25 percent. Scope-only, 40 to 50. Comprehensive, 70 to 80. Specialist passes plus synthesis, 85 to 95. Multi-model, 95 and up.

These figures share one defect: the denominator is unknown. A finding count tells you what was caught. It does not tell you what fraction of the real defects that count represents, because nobody knows the real total. When the denominator is taken as the union of findings across runs, the percentages become upper bounds on the true catch rate. The real total includes every defect no run found, and the ladder is blind to those by construction.

To convert estimated into measured you need a known total. Inject defects you can count, then measure what fraction the process kills. That is mutation testing, and it is the right instrument for this problem.

## Three denominators, kept distinct

Three different quantities get called a catch rate. Conflating them is how the claim loses a careful reader.

**One. Review catch-rate against injected spec defects.** Inject a defect into the spec, a foreign key to a table that does not exist, a state machine with no exit, an audit event with no actor, a contradiction between sections, and measure what fraction a review flags. This is the natural target for spec review.

**Two. Probe completeness against injected implementation mutants.** Inject a defect into the implementation, one deliberately wrong interaction seam in an otherwise correct build, and measure whether an automatically generated behavior probe kills it. A kill is when the probe's trace against the mutant differs from its trace against the correct reference. This is oracle-free: it measures whether the probe, on its own, would have caught a divergence that otherwise slips through.

**Three. Independent-build divergence as a spec-hole detector.** Have several models implement the same spec independently and diff their outputs across many inputs. Every disagreement is a question the spec failed to answer. The divergence count is a direct measure of spec ambiguity, and it surfaces holes a single reviewer would miss, because a second reviewer shares the first one's blind spots. The multiversion-programming literature adds the necessary caution in the other direction: independently developed versions fail in correlated ways more than independence would predict (Knight and Leveson, 1986), so agreement across builds is not evidence of correctness. This uses only disagreement, which is a sound signal of underspecification even where agreement is not.

This work measures denominators two and three. It does not yet measure denominator one. The bridge is stated, not implied: denominator three locates the spec holes, denominator two measures whether the generated test catches the implementation consequence of a hole. One worked example threads all three.

## The worked example: a spec hole, measured end to end

A spec for an averaging function never stated its rounding tie-break. Round-half-even and round-half-up are both valid readings of the prose. Independent builds therefore diverge only on exact-half inputs with an even floor: a mean of 2.5 returns 2 under half-even and 3 under half-up, while a mean of 3.5 returns 4 under both. Reading the spec never surfaces this. The divergence detector did. The implementation mutant, the same correct build with the rounding mode flipped, was killed by the auto-generated probe about 85 percent of the time, until the probe generator was instructed to construct the exact even-floor boundary case, after which it killed at 100 percent. The gap sat in the spec. Independent builds exposed it. The kill rate measured it. One targeted instruction closed it.

## The instrument at scale

Probe completeness is a rate, not a binary. Each probe is a fresh stochastic generation, so a given mutant is killed with some probability at or below 1, and a single observed kill is one undersampled sample. The rate is measured over K independent generations, where one generation tests every mutant in its domain, so the cost is K times the number of domains.

A full clean run at K equals 20, across twelve domains and 25 mutants, killed 24 of 25 at rate 1.00. The single survivor was the rounding tie-break above, at 0.85, the one class theory predicts a behavioral probe will stumble on. A methodology result fell out of the run: near-1.0 claims require K of at least 40, not a single K of 20, because the K-equals-20 read is itself noisy enough to misattribute a genuine 0.97 mutant as either a clean 1.00 or a false regression.

## The taxonomy is small, enumerable, and each gap has a closing rule

Here is the part that changes the shape of the conversation, and it needs stating carefully so the claim does not outrun the evidence. The working assumption is that you would need to enumerate 30 to 50 spec-defect types to get real signal on a review process. The set that has emerged is small, it is enumerable, and every entry carries a stated closing rule. It is not proven closed, and that distinction is load-bearing.

The basis has two layers. Fifteen structural primitives are the pieces: accumulator, key-value map, append-only log, bounded buffer with eviction, priority collection, state machine, undo and redo history, partition, sequence diff, recursive-descent parse, encode and decode bijection, interval predicate, sliding-window counter, fold, tally. Each has at least one isolating domain and pegs 100 percent.

Thirteen interaction primitives are the seams where two pieces join: input validation, boundary and overflow, eviction policy, time and expiry, implicit invariant and atomicity, per-key isolation, error versus sentinel versus clamp, round-trip type fidelity, tie-breaking, simultaneous-event ordering, cascade discipline, mutation during iteration, branch truncation. This is the layer where closure stops being free. A composition manufactures new ambiguity at the seam exactly when the seam carries shared mutable state or simultaneity, and that ambiguity is invisible to prose review because it only appears when you construct the colliding case the customer never volunteers.

Twenty-eight entries, each with a stated closing rule, lands in the same numeric range as the 30-to-50 estimate, but that coincidence should not be oversold. The 30-to-50 estimate counted spec-defect mutations to inject; these 28 count construction primitives and their seams. Those are different taxonomies at different granularities, and landing in the same range is not evidence they are the same set. The honest scope of the comparison is only that the count of distinct gap-kinds is small and can be written down, not that one list equals the other. The stronger claim, that this is a closed basis from which every composition inherits closure, is not made here, and the paragraph above is the reason. The interaction layer manufactures new ambiguity at the seam, which makes it a second generative layer rather than a set the structural primitives close over. Whether the thirteen interaction entries are themselves closed under seam-on-seam composition, three-way collisions and the rest, is open. What is claimed is narrower and survives scrutiny: this set was grown by falsification across the domain set, every entry has a closing rule, and no domain run has yet produced a fourteenth interaction class. That is carefully enumerated and not yet falsified, which is weaker than derived and is the true statement.

## Two channels, and the honest status of the seams

Each interaction seam closes through exactly one of two channels.

A mandatory question forces the spec to carry the answer, guarded by a structural trigger so it fires only when the API has the relevant shape. Five seams are named this way, and the guards were verified to fire on the right domains and stay silent on the rest.

A numbered edge-class checklist forces the generated test to build the separating case. Twelve classes, grown by falsification: every surviving mutant named a missing class, the class was added, the survivor closed with no regression.

Closing a seam means placing it into one channel or the other. The honest status of the interaction layer has three tiers, and flattening them into a single number would be the narrativizing move to avoid. One seam is genuinely open: tie-breaking and order-among-equals, which is opaque and divergent. Models diverge, and the rule is a specific algorithm's internal choice that no behavioral answer conveys, so it closes by injecting the exact procedure rather than by asking. Five seams are named as mandatory questions and verified to fire on the right domains and stay silent on the rest, but verified-to-fire is not measured-to-rescue: whether forcing the question actually saves a build that a weaker model would otherwise lose is a separate measurement, and it is listed below as open. The twelve probe edge-classes were grown by falsification and are exactly as strong as the domains that have probed them. One case sits inside that last tier and shows its limit honestly: mutation during iteration, whose canonical mutant was killed 52 of 52 across two runs, which closes that one mutant rather than the whole class. So the frontier is not a single clean cell. It is one open seam, a set of fire-but-unmeasured rules, and a falsification-grown checklist, and those are three different confidence levels.

## Limits, stated plainly

The kill rate measures probe completeness and build divergence. It does not measure review catch-rate against injected spec defects. That is a different instrument on a different denominator, and the honest position is that this work supplies the second and third, not the first. There is a denominator problem one level down as well: a kill rate of 24 of 25 says the probe is good at the mutants that were written, not that the mutant set covers the space of injectable defects. The kill rate now has a denominator. The mutant set does not. The bridge between the three denominators is also currently threaded through a single worked example, and it needs to hold across many holes, not one clean rounding case, before it is more than suggestive. Completeness is asymptotic in the same spirit: every surviving mutant names a missing class, so the set is driveable but never stamped done.

## Relation to prior work

These ideas have a literature, and most of this was reached independently before that literature was in hand, which is worth stating plainly rather than implying novelty. Kill-rate-as-completeness is mutation testing (DeMillo, Lipton, and Sayward, 1978; Jia and Harman, 2011), already applied to scoring model-generated specifications. Detecting ambiguity by checking consistency across independent generations and then asking clarifying questions is the core of ClarifyGPT (FSE 2024). Cross-model disagreement as an underspecification signal, model routing to cheaper implementers, and the limits of multiversion independence are all established or concurrent work. What is offered here is the integration, the sampled-denominator framing applied to an auto-generated probe, and the trade below.

## The invitation

The methodology converts catch rate from estimated against the union of findings to measured against injected ground truth. Two extensions are open and worth building with anyone who holds the other half.

Port the injection grammar to spec-level defects. The implementation-mutant grammar, flip a boundary, swap an operator, alias an argument, has a spec-level analogue: inject a polymorphic foreign key, a metric with a unit mismatch, a state with no exit, a contradiction between sections. Thirty to fifty such mutations would put a measured denominator under spec-review catch rate specifically, the one denominator this work does not yet measure. The basis suggests that set is small and enumerable rather than open-ended.

Run it against a deliberately weak implementer, to convert a named seam rule from verified-to-fire into measured-to-rescue a build that an unstructured prompt loses.

The argument is no longer whether AI reviews are worth it. It is what fraction of the real defect total a process kills, measured against a ground truth you injected, and the taxonomy you need to inject is small and enumerable, because the software is composed from a basis you can write down.
