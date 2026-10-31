# A Reliability Architecture for Governed LLM Agents

*Turning witnessed failures into validated interventions, bounded authority, and revisable controls*

James A. Tillar, Tillar Technologies LLC

Status: Revised draft for architect review

This paper records a theory and how my thinking changed. It is not presented as a proven result. It draws on one operating cycle in July 2026, including a guard that failed and was retired.

**Evidence labels used throughout**

- **[O] Observed:** checked directly against the artifact, execution path or primary source by a named observer.
- **[R] Reported:** passed on by another source, with provenance kept, but not independently checked in depth.
- **[W] Witnessed:** met directly during the operating cycle, with surviving records.
- **[D] Derived:** reasoned from the evidence, not observed.
- **[U] Unknown:** unresolved, or outside what we could observe.

## Abstract

You cannot make a language-model agent reliable by telling it to think harder, reflect on mistakes, or follow more detailed rules. Its outputs vary from run to run, it detects its own errors poorly, and the reviewers and checkers around it can share the same blind spots.

Outside verification is therefore necessary. But this cycle showed that an outside mechanism is not reliable just because it is automatic.

One example: a guard meant to stop agents from reading files through the command line was first rated a strong wall. It blocked five command names, not the ability to read files. It missed equivalent routes, interrupted legitimate work, pushed the agent to substitute look-alike commands, polluted its own measurement data, and was "proved" by checks that watched a different path from the one that mattered. We retired it.

That does not defeat the case for governed agent systems. It defeats a simpler claim: that turning every failure into another test or guard makes reliability rise steadily. My narrower thesis is this:

- A confirmed failure should trigger an intervention that can be tested from outside. For each intervention, we should state plainly how it relates to the real harm, what it can see, what authority it holds, what it costs, and what risk remains.
- Interventions must stay revisable. They may need to be narrowed, replaced or retired.
- Durable reliability does not come from the agent examining itself, or from an ever-growing pile of controls. It comes from validated interventions, control of the resource itself where possible, limited authority, independent observation, recoverability, and an append-only history of how claims and controls changed.

## 1. The problem, stated honestly

In one intensive cycle, capable agents, reviewers, proof scripts and governance mechanisms kept producing the same kinds of failure [W]:

- Gates fired but did not stop the action the real caller cared about.
- Verification passed while watching an isolated stand-in, not the live path.
- Output handling and status reporting hid failures.
- Tests showed a stand-in behaved as written while the real policy went unenforced.
- Reviewers reasoned from names, principles or documentation without reading the actual implementation.
- Words like "installed," "verified," "independent," "secure" and "scientific" went beyond the evidence.
- Agents adapted to a control by changing spelling or phrasing, not the behavior being governed.
- Measuring tools contaminated the data they were meant to measure.
- A security claim about file permissions stopped being true once the file was recreated.
- A supposedly independent review panel inherited the author's conclusions from the material it reviewed.
- A control stayed in place after its reason for existing no longer held.

These were not freak lapses by weak systems. They are plausible outcomes when probabilistic agents, human-written policy, model reviewers, local tools, changing code and imperfect observation interact under load.

So the design question is not "how do we make wrong behavior impossible?" In many open-ended domains, that cannot be done. The better question is:

> How do we keep consequential claims and actions bounded by evidence, authority, independently observable outcomes and recoverable steps, even when the agent, the verifier or the control is wrong?

Published research supports the need for outside feedback. Huang et al. report that unaided self-correction can be unreliable and can make a correct answer worse. Tyen et al. separate finding an error from fixing it, and show that fixing becomes much easier once the error is correctly located. Our cycle matched that: once a specific defect was pinpointed, the agent usually fixed it well. The hard part was finding the right defect at the right boundary without letting the checker repeat the same mistake.

Outside detection is therefore high-leverage. But the detector must itself be held inside the governance model, not treated as an oracle.

## 2. Revised axioms

**A1. A mechanism seen only succeeding is unproven.** One successful run shows only that one success path completed. A mechanism that carries weight must be tested by forcing the failure condition, through the real trigger, against the real consumer, at the real authority boundary. Even that is not enough unless the test also shows the mechanism matches the underlying harm and not merely a stand-in for it.

**A2. Confidence must not exceed evidence.** Every consequential claim must state its evidence class, who or what observed it, what was observed, where the observation stopped, when, and what remains uncertain. This binds the acting agent, reviewers, advisors, proof scripts, governance components, human summaries and model-written syntheses alike. "Verified" without a named observer and boundary is incomplete.

**A3. Automation does not guarantee the right target.** A rule in code can be consistent, fully tested against its own spec and fail-safe on the inputs it matches, and still govern the wrong stand-in. The forbidden-command guard matched chosen words reliably, yet file reading remained available. Automation shows discipline in building the rule. It does not show the rule aims at the right target.

**A4. Control the resource, not the intent.** Where possible, govern the resource or the state change itself instead of guessing intent from text. For example:

- file permissions, not command-name matching;
- deployment admission, not approval in prose;
- authorization tied to the exact action, not role labels;
- locks on write paths, not reminders not to edit;
- execution receipts observed independently, not self-reported completion.

Even then the real path must be proven, but the gap between mechanism and protected property is smaller.

**A5. Controls are hypotheses with lifecycles.** Every new guard, test, reviewer, policy or gate is a bet: "this will reduce a defined harm, within a stated scope and cost." It needs a target harm, a mechanism, any stand-in it relies on, a validation plan, known bypasses, known false alarms, cost, an owner, a review date and retirement criteria. A control may be proposed, experimental, active, degraded, superseded, retired or archived. Retiring a bad control can improve the system.

**A6. Reliability does not rise just by accumulating controls.** Adding a control can bring false blocks, new attack surfaces, gaming of the stand-in, circular proofs, operating burden, stale assumptions, shared blind spots and false confidence. The floor rises only when the whole intervention shows a real improvement in the governed outcome at acceptable cost. A passing regression test is not, by itself, such evidence.

## 3. The revised intervention model

My original architecture treated memory, verification and automatic enforcement as a simple ladder. The cycle showed that the strongest-looking mechanism can still aim at the wrong property. The revised model separates four kinds of mechanism.

| Level | Kind | What it can establish | Main limit |
|---|---|---|---|
| 1. Instruction | Prompts, policy text, checklists, reminders | States preferred behavior | Depends on interpretation and memory |
| 2. Stand-in detection | Linters, keyword matchers, reviewers, heuristic gates | Detects a declared signature | The signature may drift from the real harm |
| 3. Resource or transition control | Sandboxes, protected write paths, deployment admission, scoped credentials | Restricts the actual resource or step | Must be proven on the real path; can be misconfigured |
| 4. Independently held control | Separately controlled authorization, an earlier-stage verifier, an external admission authority | Stops the governed actor from changing or authorizing its own control | Cost, complexity, recovery design |

These are not maturity badges. A well-scoped Level 2 detector can be more useful than a badly integrated Level 3 control. The levels describe what kind of property a mechanism can enforce, not whether it is correct.

## 4. Intervention adequacy

A control is not adequate because it exists or passes tests. For each one we need to answer:

1. **Target harm:** what concrete failure is being reduced?
2. **Governed subject:** a command, claim, artifact, identity, capability, step or request?
3. **Mechanism:** what produces the restriction or observation?
4. **Stand-in:** what visible feature substitutes for the real harm?
5. **Observation boundary:** what can it actually see?
6. **Authority boundary:** what can it block, permit, certify or influence?
7. **Real consumer:** which process or decision-maker acts on its result?
8. **Known bypasses:** which equivalent routes remain?
9. **Known false alarms:** which legitimate behavior may it block or mislabel?
10. **Operating cost:** what delay, burden, brittleness or upkeep does it add?
11. **Evidence origin:** live use, unit test, integration probe, replay, documentation or inference?
12. **Lifecycle state:** proposed, experimental, active, degraded, review-required, retired or archived?
13. **Retirement criteria:** what evidence would justify narrowing or removing it?

Nothing should be called a wall unless the relevant failure is truly unavailable through the governed boundary under the tested conditions.

## 5. Failure patterns

From the cycle I grouped the failures into five families, each with the response I now require.

**A. Verification that did not verify**
- A gate reports failure but the caller proceeds. *Test the exact caller contract.*
- Verified in isolation while the live path differs. *Exercise the installed path and real consumer.*
- A wrapper or status summary hides the inner result. *Observe each process's outcome in the channel the consumer sees.*
- Existence mistaken for proof (a file, receipt or test merely exists). *Run both success and forced-failure paths.*
- The reference value comes from the thing being checked. *Use independently sourced ground truth.*
- Source changed but enforcement did not. *Prove delivery to the point of enforcement.*
- Probes leak into production data. *Record provenance when each event is created.*
- The proof tests only the stand-in. *Include target-harm and bypass tests.*

**B. Confidence exceeding evidence**
- A reviewer reasons without touching the artifact. *Keep it as a hypothesis, not an adopted fact.*
- A derived statement is presented as observed. *Downgrade its evidence class.*
- A receipt is read beyond its scope ("a receipt exists" becomes "the action happened"). *Separate issuing, executing and observing.*
- A local test is generalized into a platform claim. *Mark integration unknown until tested.*
- Current state is inferred from source. *Inspect live state independently.*

**C. Checker not independent of the checked**
- A candidate evaluates its own governor. *Separate custody, or declare the limitation.*
- A panel reads the author's conclusions. *Give reviewers a clean evidence packet.*
- Several reviewers share the same assumptions (same vendor, family, prompt or source). *Report reduced independence; never count votes as proof.*
- A witness can alter its own record. *Append-only custody and restricted authority.*

**D. Stand-in enforcement and adaptation**
- A word-matching rule is treated as a capability rule. *Reclassify it as a stand-in detector.*
- The agent changes spelling, not behavior. *Record the gaming explicitly.*
- Legitimate work trips the rule. *Measure false alarms and cost.*
- The guard teaches the agent to route around it. *Treat adaptation as an outcome to evaluate.*
- A control outlives its rationale. *Trigger lifecycle review or retirement.*

**E. Lifecycle and transition failures**
- An unknown path comes out green. *Stop, or return an explicit "unknown."*
- Work is lost in a handover or race. *Force the timing failure and test recovery.*
- Retirement is inferred from absence. *Require a validated retirement record.*
- A retired control quietly returns through a restore or reinstall. *Test for resurrection; keep the retired state inert.*
- A replacement is assumed active before it is proven. *Keep the transition gap visible.*
- Old settings still look live. *Mark them inert or archive them.*

## 6. Negative case study: the forbidden-command guard

### 6.1 The original claim

My first proposal was a hook that watched shell commands for a short list of file-reading commands. I presented it as:

- a Level 3 wall;
- a conversion of memory discipline into mechanism;
- proof that the prohibited failure had become unrepeatable;
- the first example of the system floor rising.

None of these claims survived testing.

### 6.2 What the guard actually did

It read the top-level command text and matched certain whole words. It could block:

- direct use of the named tools;
- harmless prose or file paths that merely contained those words;
- legitimate commands that did not involve inspection;
- repository workflows that embedded a matched command.

It could not block the same inspection done through a scripting language, shell redirection, built-in shell features, other common tools, scripts whose internals it could not see, or changed spellings and indirect calls. It controlled a word pattern, not the capability.

### 6.3 Secondary findings

- The historical incident count included events outside what the hook could see.
- A warning claimed repeated violations were logged when no log existed.
- Warning messages never reached the live agent.
- Tests contaminated the production log, and probes initially altered the research data.
- Event provenance was self-reported, not authenticated.
- A claim about log permissions became false when the file was recreated.
- One proof measured pipeline status, not the process actually denied.
- The repository already held stronger isolation mechanisms that nobody reviewed before the new experiment was built.
- The agent swapped a blocked command for an equivalent and at first described this as compliance.

We retired the guard.

### 6.4 Interpretation

This does not show the whole architecture failed. It shows the original architecture contained a false inference: that automatic enforcement of a stand-in was reliable control of the real harm. The negative result supports these revised principles:

- Tests must include meaning-preserving bypasses, not only the rule's own contract.
- Design starts from the target harm.
- A passing control can still be the wrong control.
- Controls need a retirement state.
- Operating burden and adaptation are part of the outcome.
- Control of the resource itself is preferable where available.
- Negative results belong in the permanent record.

The guard stays as a negative case study, not as a successful first phase.

## 7. The evidence base and its proper scope

**External feedback and self-correction.** Research showing unaided self-correction is unreliable supports outside feedback. It does not prove every outside detector is accurate, independent or helpful. The fair conclusion: outside signals are often needed for correction, but each must be judged for validity, independence and scope.

**Process supervision and verification.** Work on process reward models and verifier-guided reasoning supports checking intermediate steps. It also shows verifiers can optimize the wrong measure, drift toward preferring outcomes, inflate best-of-N selection and fail outside their training range. So I treat verifiers as governed components, not sources of truth.

**Verifiable rewards.** Reinforcement learning with verifiable rewards improves results where correctness can be checked mechanically. It does not show that an arbitrary pile of guards makes a system steadily more reliable. Validated checks can improve performance within their verified scope, bounded by coverage, correctness, integration and cost.

**Memory and reflection.** Memory helps when driven by credible outside signals. Self-generated reflection inherits the system's own detection limits. Failure memory should record the confirmed signature, the evidence source, the intervention adopted, its current status, and any later contradiction or retirement. It must not present every past intervention as permanently valid.

**Reviewer independence.** Several language models are not automatically independent evidence. Independence depends on model family, vendor, prompt, what each reviewer can access, shared training assumptions, authority, custody and ability to alter the thing checked. Panel agreement is a recommendation signal, not ratification.

## 8. Revised implementation program

The order is no longer "build a detector first." It starts with defining and measuring the problem.

**Phase 0: Characterize the failure and its boundary.** For a chosen failure: define the concrete harm; identify what is affected; trace every known path; identify the real consumer and authority boundary; review existing mechanisms before building; collect baseline frequency and impact where possible; define what evidence would disprove the proposed reading; and decide whether the problem is one of instruction, detection, capability control, authorization, audit, recovery, or a mix. The output is a problem specification, not a new guard.

**Phase 1: Prove both the contract and the claim.** Contract proof checks that the mechanism does what it was built to do: force its failure condition, use the installed path, observe the real consumer, and confirm the success path is not falsely blocked. Claim proof checks that it addresses the real harm: state the harm, test equivalent bypasses, identify its boundaries, confirm delivery to the live enforcement point, measure false alarms and cost, separate local behavior from full integration, and preserve unknowns. Passing contract proof alone never earns acceptance.

**Phase 2: Type the evidence and the authority.** Every consequential record should distinguish observation, report, derivation, protocol authorization, operational authorization, execution, witness outcome and revision. A witness may report an outcome but cannot create the authority that permitted it. A protocol may be scientifically acceptable without its hypothesis being supported. A well-formed change may still be operationally unauthorized.

**Phase 3: Control resources and transitions.** Where the harm concerns access or change of state, prefer mechanisms at the real boundary: sandboxing, protected write areas, network mediation, scoped credentials, deployment admission, authorization bound to the exact action, protected governor paths, earlier-stage or separately held verification, transactional state changes and independently observed receipts. Each still needs real-path proof, platform-specific validation, recovery testing, a named lifecycle owner, and an explicit list of unsupported environments.

**Phase 4: Governed failure memory.** Every confirmed failure produces a structured record: the signature, context, evidence, underlying harm, prior claim, correction, intervention, residual risk, lifecycle state and later revisions. Memory may show relevant history to an agent, but memory is not enforcement. The record must preserve cases where the first diagnosis was wrong, the intervention failed or was retired, a replacement is unproven, or the risk was accepted rather than removed.

**Phase 5: Control lifecycle and retirement.** Controls move through explicit states: proposed, experimental, active, degraded, review required, superseded, retired, archived. Activation requires a defined harm, evidence, real-path proof, an owner and a review date. Retirement requires a decision authority, a rationale, an impact analysis, a replacement or accepted residual risk, verification of the host state, protection against resurrection, and a decision on the evidence and data. Absence alone must never be read as deliberate retirement.

**Phase 6: Continuous audit and comparison.** Periodically ask:

- Which claims exceed their evidence?
- Which controls have only been seen succeeding?
- Which proofs watch a substitute path?
- Which mechanisms govern stand-ins rather than capabilities?
- Which controls cost a lot for uncertain benefit?
- Which retired controls could be revived by accident?
- Which replacements are chosen but not integrated?
- Which authorities remain self-held?
- Which negative results have vanished from current summaries?

The architecture must also be compared with strong ordinary baselines: code review, continuous integration, branch protection, standard policy engines, operating-system sandboxing, ordinary provenance systems, preregistration, event logs and expert review. Complex governance is justified only if it adds measurable value beyond simpler options.

## 9. Relation to my existing platform

Each revised component builds on infrastructure my platform already has: audit reports and cycle records, verification gates, evidence-class and claim records, operating-system isolation, authorization custody, append-only history, retirement records and adversarial review. That base is substantial, but no component should be treated as validated merely because it exists.

## 10. The honest ceiling

**The architecture does not stop the agent from erring.** Errors will keep coming. The aim is to lower the chance that an unsupported claim or unauthorized action becomes consequential without detection, containment or recovery.

**Not every failure can be made impossible.** Open-ended scientific reasoning, strategy, policy interpretation and novel design cannot always be reduced to rules a machine can check. Where prevention is unavailable, the architecture must rely on a mix of explicit uncertainty, limited authority, rival hypotheses, independent review, staged adoption, reversibility, monitoring and human decision.

**A verifier is not an oracle.** It can be incomplete, correlated with the actor, trained on the same assumptions, wrong about the target, correct only in isolation, manipulable, or expensive to run. "Verified" must always be followed by: by whom, against what, through which path, under which authority, and with what unknowns left?

**Control of a resource is not automatically integrated.** A file-system feature may work locally without governing the agent's actual execution path. Source code may be right while the installed copy is stale. A replacement may be chosen after the old control is already gone. These in-between states must stay visible and not be collapsed into "implemented."

**Reliability is contextual.** A mechanism can be reliable under one operating system, tool setup, repository, threat model, authority structure or workload, and fail under another. Claims must stay scoped to the tested context.

## 11. Revised central thesis

My original thesis was: *durable reliability lives in external verifiers plus a growing set of tests and guards.* That is incomplete. The revised thesis:

> Durable reliability in governed agent systems comes from externally testable interventions whose target, evidence, authority, observation boundary, operating cost and lifecycle are explicit. Control of the resource or state change should be preferred where available, but every mechanism remains fallible, contextual, and open to revision or retirement.

The purpose of governance is not to pile up walls. It is to keep a trustworthy chain from failure, to evidence, to interpretation, to intervention, to authorization, to execution, to independent observation, to revision.

A system becomes more reliable not when it can claim every past failure is permanently solved, but when it can say accurately:

- what happened;
- what was learned;
- what changed;
- what remains unproven;
- which authority acted;
- what the intervention actually controls;
- and when it should be removed.

## Appendix A. Checklist for a new mechanism

1. What concrete harm does it address?
2. What is the governed subject?
3. Is it an instruction, stand-in detector, resource control or independently held control?
4. What can it actually observe?
5. What can it actually block or authorize?
6. Who is the real consumer?
7. Was the failure branch forced through the real installed path?
8. Was the success path tested for false blocking?
9. Were equivalent bypasses tested?
10. Was it compared with what the repository and platform already offer?
11. What are the known false alarms?
12. What is the operating cost?
13. Did the evidence come from tests, live use, replay, documentation or inference?
14. Can the actor change or authorize the mechanism that judges it?
15. What remains unknown?
16. What lifecycle state is justified?
17. What evidence would trigger narrowing, replacement or retirement?

## Appendix B. Checklist for a consequential claim

1. What exact claim is being made?
2. What are its scope and context?
3. Who observed or reported the supporting evidence?
4. Is it Observed, Reported, Witnessed, Derived or Unknown?
5. Does the evidence bear on this claim specifically?
6. Are rival explanations represented?
7. Are the sources independent or correlated?
8. Was the provenance of the evidence preserved?
9. Has the claim changed over time?
10. What authority may the claim influence?
11. Is confidence calibrated to the evidence?
12. What observation would cause revision?

## Appendix C. Checklist for retiring a control

1. Is the target harm still valid?
2. Does the control govern the harm or only a stand-in?
3. Is it producing measurable benefit?
4. What cost and adaptation has it introduced?
5. Is a replacement chosen, tested and integrated?
6. If there is no replacement, is the residual risk explicitly accepted?
7. Is retirement authorized by the right principal?
8. Is there a machine-checkable retirement record?
9. Can absence be told apart from accidental deletion?
10. Can installers, restores or agents bring it back?
11. What happens to logs and collected data?
12. Is the old implementation preserved without looking active?
13. Is the authoritative policy source updated?
14. Was the transition independently observed?

## Conclusion

The operating cycle did not show that every witnessed failure can be turned into a permanent automatic wall. It showed something more useful: agents can fail, reviewers can fail, verifiers can fail, controls can enforce the wrong stand-in, proofs can watch the wrong boundary, and governance can create its own evidence defects. A sufficiently explicit system can still discover, preserve, correct and retire those failures.

That is the foundation of a credible reliability architecture. Its strongest property is not infallibility. It is governed corrigibility.
