# Chapter 1: Quality Assurance and Fundamentals of Software Testing

> Draft v0.5 (2026-08-08). Revised against HT26 Lectures 1-2 and current WalkMates.

## Slides

_Placeholder: lecture slides for this module will be linked here._

## Icon Key

- 📘 **Concept**: a core idea or term.
- 🧭 **Principle**: a rule of thumb that guides testing decisions.
- 🛠️ **Technique**: something testers actively do.
- 🧱 **Artifact**: a document, test, report, or other work product.
- 🧪 **Exercise**: something to produce, discuss, or reuse in lab work.

## Learning Outcomes

By the end of this chapter, you should be able to:

1. Explain why testing is necessary even when software appears to work.
2. Distinguish quality assurance, quality control, verification, validation, testing, and
   debugging.
3. Explain the relation between human error, fault, error state, and failure.
4. Identify a test basis, test condition, test case, test suite, and oracle.
5. Select a suitable test level for a given risk.
6. Relate one WalkMates requirement to at least one testable quality attribute.
7. Write a minimal defect report with expected behavior, observed behavior, reproduction steps,
   and risk.

## Before You Read: Retrieve and Predict

Without searching the chapter, write a short answer to both questions:

1. If every existing test passes, why might release still be risky?
2. What is the difference between observing a failure and locating its cause?

Keep the answers. Revisit them after Sections 4 and 5 and correct any vague use of “bug,”
“proof,” or “works.”

📘 **Core idea: Testing as evidence.** A test is not only code that runs. A test is a small
evidence-producing argument: under these conditions, with this input and this oracle, the system
behaved acceptably or problematically. A test does not prove quality. It contributes evidence
for a limited claim.

## 1. Why Software Testing Matters

Software testing matters because software has become infrastructure. It is no longer confined to
desktop tools or websites that can fail quietly. Software controls medical devices, trading
systems, aircraft, cars, banking apps, logistics, public services, authentication, classrooms,
media platforms, and increasingly the tools used to create more software. When software fails,
the consequences can be personal, financial, social, legal, and sometimes physical.

📘 **Concept: Software quality.** Software quality is the degree to which a software product or
system satisfies stated and implied needs under specified conditions. This definition is more
useful than saying that software is "good" or "bad", because it forces us to ask: whose needs,
which conditions, and what evidence?

Testing is one of the ways we collect that evidence. It can reveal defects, increase confidence,
expose uncertainty, improve design discussions, and make risk visible before users carry the
cost. A passing test suite does not mean that the software is correct. It means that the
selected tests did not reveal a problem under the selected conditions. The strength of a test
suite depends on how well the selected cases represent relevant risks, requirements, boundaries,
states, and usage scenarios.

### 1.1 Mission-Critical Software

Mission-critical software is software whose failure can threaten life, safety, essential
service, financial stability, or major public trust. The history of software engineering is full
of cases where a small technical defect became a large system failure because it was embedded in
an unsafe process, a fragile architecture, or a weak organizational culture.

The Therac-25 radiation therapy accidents are a classic safety case. Several patients received
massive radiation overdoses in the 1980s, and the later analysis by Leveson and Turner became a
foundational warning about overconfidence in software-controlled safety systems [R1]. The lesson
is not simply "a programmer made a bug." The lesson is that safety requires independent checks,
clear error reporting, hazard analysis, usable interfaces, conservative design, and skepticism
about assumptions.

Other cases show the same pattern in different domains. The Boeing 737 MAX accidents are a
safety-critical aviation example [R2]. The Knight Capital trading incident is a financial
automation example where a deployment and control failure produced a loss of more than $460
million in about 45 minutes [R3]. We will return to such cases later; for now, the important
idea is that defects become dangerous when they meet scale, speed, weak controls, or high
consequence.

**WalkMates proxy.** WalkMates is not safety-critical in the same way as aircraft or medical
devices, but the same reasoning still applies. A booking rule can affect trust, fairness, money,
and user confidence. A small pricing or capacity defect is a low-stakes classroom example of the
same principle: software decisions have consequences.

### 1.2 Software in Daily Life

Most people interact with software before breakfast: phone alarms, payment apps, transit
schedules, digital identity, messaging, course platforms, maps, music, health apps, smart home
devices, and authentication flows. Because these systems are ordinary, their reliability can
become invisible. We notice them only when they fail.

The 2024 CrowdStrike incident made this dependence visible. A faulty content update affected an
estimated 8.5 million Windows devices, less than one percent of Windows machines, yet the impact
was broad because many affected devices belonged to organizations running critical services
[R4]. CrowdStrike's own root-cause summary identified a mismatch between expected and provided
input fields that resulted in an out-of-bounds memory read and a system crash [R5]. A seemingly
technical defect became a public disruption because the software was widely deployed in important
environments.

Daily-life software also carries social expectations. A payment app must be secure and fast. A
learning platform must be available when assignments are due. An accessibility feature must work
for users who depend on it, not only for users who treat it as optional. Testing is therefore not
only about finding crashes. It is about checking whether software continues to support ordinary
life under realistic conditions.

**WalkMates proxy.** A WalkMates user may rely on the service to book a dog walk, volunteer at a
shelter, or arrange pet sitting. The system is educational, but the domain lets us practice
thinking about real users: providers, seekers, payments, trust, scheduling, and explanations.

### 1.3 Complex, Autonomous, and AI-Assisted Systems

Modern software is complex because it is interconnected. A single user action may involve a web
front end, backend services, databases, authentication providers, payment services, logging,
cloud infrastructure, third-party APIs, machine-learning components, and background jobs. No one
person can hold all interactions in memory.

Modern software is also increasingly autonomous. It observes, predicts, recommends, filters,
prioritizes, and sometimes acts. This changes the tester's question. Instead of asking only
"Does this function return the right value?", we also ask "What happens if the system is
confident but wrong?", "Can the system recover?", and "How will humans understand and override
it?"

Software is now often written with AI assistance. Tools such as GitHub Copilot can suggest code,
explain code, propose edits, validate files, and support agent-style workflows [R6].
AI-generated code is not automatically wrong, but it is not automatically trustworthy either. In
security-relevant prompt scenarios, Pearce et al. found that approximately 40% of generated
programs contained vulnerabilities [R7]. This does not mean that 40% of all AI-generated code is
vulnerable, but it shows why AI-generated code still requires review, testing, and
security-aware oracles. Later work comparing AI-assisted code generation tools also found
differences in correctness, security, reliability, and maintainability across tools and tasks
[R8].

**WalkMates proxy.** In WalkMates, AI assistance is welcome, but the reflection asks what AI
suggested, what you kept, what you changed, and why. The assessed skill is not pressing a
button; it is judging the evidence.

## 2. Faults, Errors, Failures, and Debugging

The failure stories above are easier to understand once we have a vocabulary for how problems
move from human work into system behavior.

📘 **Concept: Human error.** A human error is a mistake made by a person: misunderstanding a
requirement, choosing the wrong operator, forgetting a boundary, misconfiguring a deployment, or
making an unsafe assumption.

📘 **Concept: Fault / defect / bug.** A fault is the static defect introduced into an artifact.
It may be in source code, requirements, test data, configuration, documentation, or a prompt.

📘 **Concept: Error state.** An error state is an incorrect internal state during execution.
The system is now wrong internally, even if the user has not yet observed a failure.

📘 **Concept: Failure.** A failure is externally visible behavior that violates expectation:
wrong output, crash, rejection, acceptance, delay, unsafe action, misleading message, or missing
response.

**Terminology note.** ISTQB commonly uses *error* or *mistake* for the human action that
introduces a defect. Dependability literature also uses *error* for an incorrect internal
state [R15]. This course says **human error** and **error state** so the two meanings remain
visible.

The chain is important:

```text
human error -> fault/defect -> error state -> failure
```

Not every fault causes a failure every time the program runs. A fault may remain hidden until a
specific input, state, timing condition, configuration, or dependency activates it. This is why
boundary cases, state transitions, and realistic combinations matter. Testing tries to choose
conditions that are likely to expose faults before users do.

The observer and context matter. A method-level test may observe a wrong return value, an API
client may observe an incorrect status, an operator may observe latency or logs, and a user may
observe that the task cannot be completed. This does not make correctness arbitrary: the
expected behaviour, observer, and context must be specified.

🛠️ **Technique: Debugging.** Debugging is the systematic activity of localizing,
understanding, and fixing a defect. It is related to testing but not the same activity. Testing
reveals evidence of a problem; debugging investigates and repairs the cause. Confirmation testing
then checks whether the fix resolved the original failure, and regression testing checks for
unintended effects.

**Running example: FR-4.1.** WalkMates requirement `FR-4.1` says that a booking duration must
be between 30 and 1440 minutes inclusive. Suppose a developer implements the lower boundary as
`durationMinutes > 30` instead of `durationMinutes >= 30`. The human error is misunderstanding
or overlooking the inclusive boundary. The fault is the wrong comparison. The error state occurs
when the system treats exactly 30 minutes as invalid. The failure is visible when a user cannot
book the minimum allowed duration.

> **Common student mistake.** "A failing test always means the code is wrong."
>
> **Better.** A failing test may indicate a code defect, a test defect, an environment problem,
> missing test data, or an unclear oracle.

🧪 **Exercise: Fault-chain artifact.** Start with this visible failure: "A booking request is
rejected, but the user receives no useful explanation." Write four short lines: possible error
state, possible fault, possible human error, and the test level that should have caught it.

## 3. Why Exhaustive Testing Is Impossible

Software testing is difficult because software behavior grows combinatorially. Every input,
state, configuration, dependency, device, permission, network condition, and timing variation can
interact with the others. Exhaustively testing every possible situation is usually impossible.

Consider a mobile payment app such as Swish. Even a simplified test space can explode quickly:
operating systems, phone models, network conditions, battery states, app permissions, app
versions, authentication states, banks, payment amounts, notification settings, accessibility
settings, and error recovery paths. If these factors have only a few values each, their product
can already produce hundreds of thousands of scenarios. Testing each scenario manually for five
minutes would require years of continuous work.

🛠️ **Technique: Combinatorial reasoning.** When testers list variables and possible values, they
are not claiming that every combination will be tested. They are making complexity visible so
that selection becomes deliberate. The tester can then choose boundaries, risks,
representative cases, pairwise combinations, high-impact workflows, or automated checks.

Even a tiny WalkMates model grows quickly. If listing status has 5 values, trust tier has 4
values, and wallet relation has 3 values, then there are `5 x 4 x 3 = 60` possible combinations
before duration, provider capacity, listing type, browser behavior, database state, concurrency,
invalid formats, or AI explanations are considered.

A slightly larger model is already much bigger:

| Factor | Example values |
|---|---|
| Listing status | `AVAILABLE`, `BOOKED`, `IN_PROGRESS`, `COMPLETED`, `CANCELLED` |
| Trust tier | `NEW`, `VERIFIED`, `TRUSTED`, `PRO_SITTER` |
| Seeker booking count | below limit, at limit, above limit |
| Provider capacity | below capacity, at capacity, above capacity |
| Duration | below 30, exactly 30, normal, exactly 480, above 480, exactly 1440, above 1440 |
| Listing type | dog walk, day visit, pet sitting, house sitting, shelter volunteer |
| Wallet relation | less than total, equal to total, greater than total |

This reduced model gives `5 x 4 x 3 x 3 x 7 x 5 x 3 = 37,800` combinations before we add
technical environment details. The answer is not to give up. The answer is to test
intelligently.

🧭 **Principle: Selection is unavoidable.** Since exhaustive testing is impractical, every test
strategy is a selection strategy. The quality question is whether the selection is justified by
risk, requirements, prior failures, boundaries, usage, and cost.

🛠️ **Technique: Risk-to-evidence loop.** Use the same five steps throughout the course:

1. Identify a plausible failure.
2. Estimate consequence, likelihood, and uncertainty.
3. Select evidence that could expose or reduce that uncertainty.
4. Run the test or evaluation and interpret the result.
5. Reassess the remaining risk and choose the next action.

🧪 **Exercise: Missing-test artifact.** Choose one missing WalkMates test. Write: one
requirement or risk it relates to, one testing principle that explains why it matters, one
expected outcome, and one sentence explaining residual risk.

> **Common student mistake.** "I tested valid input, so boundary behavior is covered."
>
> **Better.** Valid input includes normal values and boundary values. For `FR-4.1`, 30 and 1440
> minutes are valid, not merely nearby examples.

## 4. What Software Testing Is

📘 **Concept: Software testing.** Software testing is a lifecycle activity that evaluates
software products and related work products to find defects, assess quality, reduce risk, and
support decisions about whether the product is fit for its intended use.

First, testing is lifecycle-wide. It can begin before code exists by reviewing requirements,
models, user stories, interface contracts, risks, and acceptance criteria. Waiting until the end
of development makes defects more expensive to understand and fix.

Second, testing includes both static and dynamic work. Static testing evaluates artifacts without
executing the program: reviews, inspections, walkthroughs, requirement checks, static analysis,
and code reading. Dynamic testing executes software with selected inputs and observes behavior.
Static testing can examine requirements, software models, design documents, and source code;
dynamic testing executes the program to expose possible failures.

Third, testing needs evidence and an oracle. A test is not only an input; it also needs a way to
decide whether the observed result is acceptable.

**Scope note.** SWEBOK's compact definition of software testing emphasizes dynamic execution,
while ISTQB also teaches static testing activities within the testing process. In this course,
*testing* is used as the broad lifecycle practice; **dynamic testing** is named explicitly when
software execution is required.

📘 **Concept: Test basis.** A test basis is any source of information used to derive tests. It
may be a requirement, user story, regulation, design model, interface contract, bug report,
source code, risk analysis, or stakeholder expectation.

📘 **Concept: Test condition.** A test condition is something about the system that could be
tested. For `FR-4.1`, useful conditions include lower boundary behavior, upper boundary
behavior, rejection below the minimum, rejection above the maximum, and ordinary valid duration.

🧱 **Artifact: Test case.** A test case is the smallest unit of testing evidence. It describes
the situation to be checked: preconditions, inputs or actions, execution conditions, and expected
outcome. In executable tests, the expected result is usually encoded as an assertion.

🧱 **Artifact: Test suite.** A test suite is a collection of related test cases selected for a
purpose, such as testing a feature, a risk, a regression area, or a quality attribute. A suite
should provide breadth, not just repetition. Ten tests that all check the same happy path do not
give the same evidence as a suite that covers normal behavior, boundaries, invalid inputs, and
important interaction cases.

🧱 **Artifact: Test oracle.** A test oracle is the principle or mechanism used to recognize
whether behavior is correct or problematic. Without an oracle, a test can execute a program but
cannot judge the result.

| Oracle type | How it decides expected behavior | Example |
|---|---|---|
| Specified oracle | Uses requirements, models, contracts, or standards. | `FR-4.1` says duration must be between 30 and 1440 minutes. |
| Derived oracle | Uses another artifact, older version, reference implementation, or calculation. | A spreadsheet or known-good implementation computes the expected price. |
| Implicit oracle | Uses general assumptions such as "the system should not crash." | A web request should not produce a 500 error for ordinary invalid input. |
| Human oracle | Uses expert or stakeholder judgment. | A teacher judges whether an AI-generated explanation is clear enough for students. |

The oracle problem appears when the expected result is uncertain, subjective, expensive to
compute, or missing from the specification. AI systems often have this problem: two different
answers may both be acceptable, but unsafe, irrelevant, biased, or instruction-following output
may still be clearly wrong. The tester then needs alternative oracles, such as invariants,
metamorphic relations, safety rules, review rubrics, or comparison against curated examples.

🛠️ **Technique: Basic testing activity loop.** A simple testing loop is: identify the
objective, select inputs, determine the expected outcome, set up the execution environment,
execute the program, and analyze the result. This loop appears simple, but every step contains
judgment.

**Running example: FR-4.1 as testing evidence.**

| Testing concept | FR-4.1 example |
|---|---|
| Test basis | Requirement `FR-4.1`: booking duration must be 30-1440 minutes inclusive. |
| Test condition | Lower boundary behavior. |
| Test case | Try to create a booking with duration `30`. |
| Oracle | The requirement says `30` is valid. |
| Expected outcome | Booking duration validation accepts `30`. |
| Related boundary cases | `29`, `30`, `31`, `1439`, `1440`, `1441`. |

> **Common student mistake.** "The test passed, so the program is correct."
>
> **Better.** The test passed for this input, environment, and oracle. That is useful evidence,
> but only for a limited claim.

### 4.1 Testing Principles as Evidence Rules

ISTQB expresses several testing principles that are useful because they protect us from
overclaiming. They are not abstract slogans; they are reminders about what kind of evidence a
test can and cannot provide [R9].

| 🧭 Principle | Meaning | WalkMates implication |
|---|---|---|
| Testing shows the presence, not the absence, of defects. | A passing suite gives evidence for selected cases, not proof that no defects exist. | Passing `FR-4.1` boundary tests does not prove all booking rules are correct. |
| Exhaustive testing is impossible. | Real systems have too many inputs, states, and environments to test everything. | The simplified booking model already produces thousands of combinations. |
| Early testing saves time and money. | Defects found in requirements or design are cheaper than defects found after implementation. | Ambiguous wording around "inclusive duration limit" should be fixed before tests and code diverge. |
| Defects cluster together. | Some components, workflows, or changes often contain more defects than others. | If many failures involve booking creation, focus deeper analysis on `BookingService` and its rules. |
| Tests wear out, also called the pesticide paradox. | Repeating the same tests becomes less effective for finding new defects; suites need review and renewal. | If every test uses 60-minute bookings, add boundary, invalid, and state-based cases. |
| Testing is context-dependent. | Testing priorities depend on domain, risk, users, technology, and consequences. | WalkMates testing emphasizes rule clarity, learning value, CI evidence, and AI-explanation risks. |
| Absence of defects is a fallacy. | Software with few observed defects can still fail if it does not meet user needs. | A technically correct booking flow can still fail validation if users cannot understand rejection messages. |

## 5. QA, QC, Verification, Validation, Testing, and Debugging

Testing sits among several related concepts. Separating them prevents confusion.

| Concept | Short description | Main question |
|---|---|---|
| 📘 Quality Assurance (QA) | Process-oriented activities that help prevent defects and improve confidence in the way software is built. | Are we using practices that make quality more likely? |
| 📘 Quality Control (QC) | Product-oriented checking activities that evaluate whether an artifact meets criteria. | Does this artifact meet the expected standard? |
| 📘 Verification | Evaluation against specifications, rules, models, or agreed expectations. | Are we building the product right? |
| 📘 Validation | Evaluation against real stakeholder needs and intended use. | Are we building the right product? |
| 🛠️ Testing | Static and dynamic activities used to find defects, assess quality, and reduce risk. | What evidence do we have about behavior and quality? |
| 🛠️ Debugging | Locating, understanding, and fixing a defect after a failure or suspicious behavior. | What caused the observed problem, and how should it be repaired? |

Testing contributes to quality control and supports quality assurance, but QA is broader than
testing. QA includes preventive process activities such as standards, reviews, risk management,
CI practices, coding guidelines, and improvement routines. A team can test heavily and still
have weak QA if it has unclear requirements, uncontrolled deployment, poor reviews, or no
learning loop after failures.

Verification checks conformance: requirements, rules, designs, contracts, formats, boundaries,
and expected outputs. Validation checks fitness: usefulness, user value, understandability,
appropriateness, and real-world effectiveness.

A system can pass verification and fail validation. For example, a booking form may enforce
every rule correctly but still confuse users so badly that they abandon the workflow. A system
can also appear valid in a demo while failing verification because hidden boundary rules are
wrong.

**WalkMates proxy.** Verifying WalkMates means checking rules such as duration `[30, 1440]`,
trust-tier limits, provider capacity, and price calculation. Validating WalkMates means asking
whether the system helps seekers and providers understand and complete realistic care
arrangements.

## 6. Quality Attributes and ISO/IEC 25010

📘 **Concept: Quality attribute.** A quality attribute is a named property of a system that
matters to stakeholders. Examples include reliability, security, performance efficiency,
maintainability, interaction capability, compatibility, flexibility, and safety. In the 2023
model, portability is addressed below the top-level flexibility characteristic rather than as a
tenth top-level characteristic.

Quality attributes are important because they turn vague judgments into discussable
requirements. "The system should be good" is not testable. "The booking confirmation should
respond within two seconds under expected course-load conditions" is much closer to testable.
"The system should be secure" is too broad. "A Listing description remains inside the declared
untrusted-data boundary without changing the prompt's control structure" is testable
deterministically; that check alone does not establish how a live model will behave.

ISO/IEC 25010:2023 defines a product quality model for ICT and software products, organized into
nine characteristics and subcharacteristics [R12]. For this chapter, focus on the nine top-level
characteristics. Later modules can use subcharacteristics selectively when they are useful.

### 6.1 Applying Quality Attributes to WalkMates

WalkMates handles money, identity data, trust, booking state, and user interaction. The stakes
are educational, but the quality questions are realistic.

| Quality attribute | What it asks | WalkMates example |
|---|---|---|
| Functional suitability | Does the software provide the needed functions correctly? | Is a valid Listing booked at the required price and state? |
| Performance efficiency | Does it respond within acceptable time and resource limits? | Does booking confirmation complete within an agreed workload target? |
| Compatibility | Does it work with other systems and environments? | Does the web/API contract work across supported clients and services? |
| Interaction capability | Can users understand and operate it effectively? | Can a Seeker understand a rejection and recover? |
| Reliability | Does it keep working and recover from problems? | Is the wallet unchanged after a gateway decline or timeout? |
| Security | Does it protect data and prevent unauthorized action? | Does untrusted Listing text remain data without changing the prompt's control structure? |
| Maintainability | Can it be modified, analyzed, and tested efficiently? | Can a rule change be located, tested, and released safely? |
| Flexibility | Can it adapt to new contexts or stakeholder needs? | Can a new Listing type or trust tier be introduced without scattered changes? |
| Safety | Does it avoid unacceptable harm? | Do failure modes prevent avoidable financial, privacy, or animal-care harm? |

🧪 **Exercise: Quality-attribute artifact.** Choose one WalkMates requirement and one quality
attribute. Write one testable question. Use this pattern: "For `[quality attribute]`, I would
test whether `[system behavior]` under `[condition]`."

**WalkMates proxy.** For `FR-4.1`, functional suitability asks whether valid durations are
accepted and invalid durations are rejected. Interaction capability asks whether the user can
understand why `29` minutes is rejected. Maintainability asks whether duration rules are easy to
find, test, and change.

## 7. Test Levels, SUT, Pyramid, and V-Model

📘 **Concept: System Under Test (SUT).** The SUT is the thing being tested. It may be a method,
a class, a component, a service, an API, a user workflow, or the whole deployed system.

The testing pyramid is a way to reason about granularity, dependencies, cost, and confidence.
At the bottom are many small tests that run quickly and isolate narrow behavior. Higher levels
exercise broader behavior with more dependencies and more realistic workflows, but they usually
cost more to write, run, and diagnose.

| Test level | Typical SUT | Dependency level | Cost and speed | What it is good for |
|---|---|---|---|---|
| Component (unit) | Separately testable class, function, or module | Very low | Fast and cheap | Business rules, calculations, boundary behavior |
| Component integration | Interfaces among components inside the system | Low to medium | More setup | Service/repository or class collaboration |
| System | Complete integrated system | High | Slower and broader | Functional flows and system qualities |
| System integration | Interfaces between this system and external systems | High / environment-dependent | Contract and infrastructure cost | Payment gateway or external identity contract |
| Acceptance | Product behavior against stakeholder expectations | Varies | Often scenario-driven | Readiness, business value, user/stakeholder fit |

Unit testing focuses on the smallest useful parts of the system. Integration testing checks
collaboration between parts. System testing evaluates the complete integrated software in a
realistic environment. Acceptance testing asks whether the product deserves to be accepted for
its intended use. A mature strategy uses these levels together.

🧭 **Principle: Match the level to the risk.** A pricing formula should usually be tested with
unit tests because the rule is precise and isolated. A login flow with external authentication
needs broader integration or system tests because the risk lives in collaboration. A user
journey may need acceptance tests because the risk lives in whether the workflow makes sense.

**WalkMates proxy.** `FR-4.1` can be tested at several levels:

| Level | Example evidence |
|---|---|
| Component | The `Booking` validation accepts `30` and `1440`, rejects `29` and `1441`. |
| Component integration | `BookingService` does not persist an invalid request. |
| System | A web request with `29` minutes returns a useful validation response. |
| System integration | Usually not the first evidence for this local rule; use it only if an external client contract is at risk. |
| Acceptance | A user can understand the allowed duration range before submitting. |

> **Common student mistake.** "When a unit test passes, the feature works."
>
> **Better.** A unit test can show that a small rule behaves correctly in isolation. It does not
> prove that the full workflow, data flow, UI, and user expectation are correct.

### 7.1 V-Model

The V-model connects development artifacts on the left side with corresponding test activities
on the right side. Requirements connect to acceptance testing. High-level design connects to
system testing. Detailed design connects to integration testing. Coding connects to unit
testing.

For Chapter 1, you only need the mapping idea: different artifacts create different testing
questions. Unit tests cannot validate a whole business workflow by themselves. Acceptance tests
cannot efficiently diagnose every small calculation error. The levels complement each other.

One useful trace is:

| Development artifact | Corresponding evidence question |
|---|---|
| Stakeholder need | Does acceptance evaluation show that the workflow solves the real need? |
| System requirement | Does system testing demonstrate the required end-to-end behaviour? |
| Interface/design contract | Do component- and system-integration tests establish collaboration? |
| Component design and code | Do component tests establish the local rule and boundaries? |

The V-model is a traceability aid, not a claim that development must follow a rigid waterfall.

## 8. The Testing Process and Core Artifacts

Testing is not only execution. Professional testing moves from planning to closure, producing
artifacts that help a team decide what to test, why to test it, how to judge results, and what
evidence remains after execution.

| Process activity | Purpose | Minimal artifacts | WalkMates proxy |
|---|---|---|---|
| Planning | Establish objectives, scope, risks, responsibilities, schedule, and approach. | Test plan or lightweight strategy. | Decide that Lab 1 focuses on quality attributes, bug analysis, first test, and specification-based rules. |
| Analysis | Identify test conditions from the test basis. | Test conditions, traceability notes. | Mark `FR-4.1` as a duration rule worth testing. |
| Design | Create test cases, test data, expected outcomes, and procedures. | Test cases, test data, expected results. | Choose `29`, `30`, `31`, `1439`, `1440`, and `1441`. |
| Implementation | Prepare executable tests, suites, environments, scripts, and automation. | Test suites, test environment, CI configuration. | Implement JUnit 5 tests and make sure Maven/GitHub Actions runs them. |
| Execution | Run tests, compare actual and expected results, and record outcomes. | Test logs, failure reports, defect reports. | Run `mvn test`, inspect failures, and connect failed assertions to requirement IDs. |
| Monitoring and control | Track progress, quality, risk, and adapt the plan. | Progress reports, metrics, dashboards. | Use CI status, coverage, mutation score later, and reflection notes to decide what needs attention. |
| Completion | Summarize testing, archive evidence, and communicate residual risk. | Test summary or closure report. | Submit lab reflection: what was tested, what evidence exists, what surprised you, and what you would test next. |

🧱 **Artifact: Defect report.** A defect report should make a problem reproducible and useful.
At minimum it should describe the observed failure, expected behavior, steps or data needed to
reproduce it, environment information, severity or risk, and supporting evidence such as logs or
screenshots.

Minimal defect report for the running example:

| Field | Example |
|---|---|
| Requirement | `FR-4.1`: duration must be 30-1440 minutes inclusive. |
| Expected behavior | A booking duration of `30` minutes is accepted. |
| Observed behavior | The system rejects `30` minutes as invalid. |
| Reproduction steps | Create or validate a booking request with `durationMinutes = 30`. |
| Risk | Minimum valid bookings cannot be made; boundary rule is implemented incorrectly. |
| Evidence | Failing test name, assertion message, log, or CI run. |

🧪 **Exercise: Write a defect report.** Given the observed failure “a top-up of exactly 10.00 is
rejected,” produce a minimal report using the fields above. Cite the relevant requirement, give
reproducible steps and environment information, and distinguish the observed failure from your
hypothesis about its cause.

🧱 **Artifact: Traceability.** Traceability connects requirements, tests, results, and defects.
It answers questions such as: which requirements have tests, which tests failed, which defects
relate to which rules, and what evidence supports a release decision. Traceability can be a
formal matrix in regulated systems or a lightweight mapping in a course project.

**WalkMates proxy.** When a test method name or comment cites `FR-4.1`, it creates a small trace
from requirement to test. When a reflection explains that a failing test exposed a boundary
misunderstanding, it creates a trace from result to learning.

## 9. Professional Context: ISTQB, SWEBOK, and Testing Education

Software testing is a professional discipline with its own vocabulary, standards, roles,
certifications, research venues, tools, and bodies of knowledge. Good testers are not people who
"just click around." They design evidence.

The International Software Testing Qualifications Board (ISTQB) provides a widely recognized
certification scheme and syllabus for software testing. The Foundation Level v4.0.1 syllabus covers
fundamentals of testing, testing principles, test levels, static testing, test analysis and
design, black-box techniques, white-box techniques, experience-based techniques, test
management, defect management, and test tools [R9]. It is useful here because it gives us common
language.

The Guide to the Software Engineering Body of Knowledge (SWEBOK) is an IEEE Computer Society
body of knowledge that organizes software engineering knowledge into areas. Testing is one of
those areas, connected to requirements, design, construction, maintenance, configuration
management, quality, security, economics, and professional practice [R10].

Garousi et al. mapped research on software-testing education and found that educators repeatedly
face challenges such as motivating students, teaching complex concepts, keeping material
industry-relevant, integrating testing into the curriculum, and providing adequate tools and
realistic exercises [R11]. This is why this course uses a realistic system under test, CI, test
reports, reflections, and project work instead of treating testing as a final-week add-on.

For further study, Aniche's *Effective Software Testing* is a pragmatic developer-oriented book
on writing useful automated tests [R13]. Ammann and Offutt's *Introduction to Software Testing*
is a deeper theoretical anchor for input-space partitioning, coverage, logic-based testing, and
other core techniques [R14].

## 10. Summary, Self-Check, and Lab 1 Preparation

This chapter introduced five core skills: vocabulary, evidence, selection, oracles, and test
levels. Keep returning to the evidence sentence: a test is a small claim under selected
conditions, with selected input, judged by a selected oracle.

Before starting Lab 1, check whether you can answer these questions:

1. Why can testing show the presence of defects but not their absence?
2. What is the difference between a fault and a failure?
3. What makes a test oracle weak?
4. Why is exhaustive testing impossible in WalkMates?
5. Which quality attribute is most relevant for prompt injection in `MatchExplanationService`?
6. When would a unit test be insufficient?
7. What should a defect report contain?
8. What evidence would convince you that `FR-4.4` is adequately tested?

🧪 **Lab 1 preparation artifact.** Choose one rule from `FR-1.1`, `FR-1.2`, or `FR-1.3`, the
requirements used by `SeekerSpecBasedTest`, and write:

1. The requirement ID and its rule.
2. One quality attribute it relates to.
3. One test condition.
4. One concrete test case with input and expected outcome.
5. The oracle you used.
6. One sentence about residual risk after that test passes.

Good starting points are email/name/phone partitions, the 9.99/10.00/10.01 top-up boundary,
the 5,000.00 single-top-up maximum, the 20,000.00 resulting-balance maximum, or the trust-tier
limit/fee table.

The next step is to learn test-design techniques that make selection disciplined rather than
random: equivalence partitioning, boundary value analysis, decision tables, and later structural
coverage, mutation testing, regression testing, and AI testing.

## References and Further Reading

[R1] Nancy G. Leveson and Clark S. Turner. "An Investigation of the Therac-25 Accidents."
*Computer*, 26(7), 1993. https://doi.org/10.1109/MC.1993.274940

[R2] U.S. House Committee on Transportation and Infrastructure. *Final Committee Report: The
Design, Development & Certification of the Boeing 737 MAX*. 2020.
https://transportation.house.gov/committee-activity/issue/the-boeing-737-max-investigation

[R3] U.S. Securities and Exchange Commission. *In the Matter of Knight Capital Americas LLC*,
Release No. 70694. 2013. https://www.sec.gov/files/litigation/admin/2013/34-70694.pdf

[R4] Microsoft. "Helping our customers through the CrowdStrike outage." Official Microsoft Blog,
2024-07-20. https://blogs.microsoft.com/blog/2024/07/20/helping-our-customers-through-the-crowdstrike-outage/

[R5] CrowdStrike. "Remediation and Guidance Hub: Channel File 291 Incident." 2024.
https://www.crowdstrike.com/falcon-content-update-remediation-and-guidance-hub/

[R6] GitHub. "GitHub Copilot: Your AI pair programmer." https://github.com/features/copilot

[R7] Hammond Pearce, Baleegh Ahmad, Benjamin Tan, Brendan Dolan-Gavitt, and Ramesh Karri.
"Asleep at the Keyboard? Assessing the Security of GitHub Copilot's Code Contributions." 2022
IEEE Symposium on Security and Privacy. https://doi.org/10.1109/SP46214.2022.9833571

[R8] Burak Yetistiren, Isik Ozsoy, Miray Ayerdem, and Eray Tuzun. "Evaluating the Code Quality
of AI-Assisted Code Generation Tools: An Empirical Study on GitHub Copilot, Amazon CodeWhisperer,
and ChatGPT." 2023. https://arxiv.org/abs/2304.10778

[R9] ISTQB. *Certified Tester Foundation Level Syllabus v4.0.1*.
https://www.istqb.org/wp-content/uploads/2024/11/ISTQB_CTFL_Syllabus_v4.0.1.pdf

[R10] Pierre Bourque and Richard E. Fairley, editors. *Guide to the Software Engineering Body of
Knowledge (SWEBOK Guide), Version 4.0*. IEEE Computer Society, 2024. https://www.swebok.org

[R11] Vahid Garousi, Austen Rainer, Per Lauvas Jr., and Andrea Arcuri. "Software-testing
education: A systematic literature mapping." *Journal of Systems and Software*, 165, 2020.
https://doi.org/10.1016/j.jss.2020.110570

[R12] ISO/IEC 25010:2023. *Systems and software engineering: Systems and software Quality
Requirements and Evaluation (SQuaRE): Product quality model*. https://www.iso.org/standard/78176.html

[R13] Mauricio Aniche. *Effective Software Testing: A Developer's Guide*. Manning, 2022.
https://www.manning.com/books/effective-software-testing

[R14] Paul Ammann and Jeff Offutt. *Introduction to Software Testing*, 2nd edition. Cambridge
University Press, 2016.

[R15] Algirdas Avizienis, Jean-Claude Laprie, Brian Randell, and Carl Landwehr. “Basic Concepts
and Taxonomy of Dependable and Secure Computing.” *IEEE Transactions on Dependable and Secure
Computing*, 1(1), 2004. https://doi.org/10.1109/TDSC.2004.2
