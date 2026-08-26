# Chapter 4: Test Optimization, Mutation, Regression, and CI

> Draft v0.2 (2026-08-08). Revised against HT26 Lectures 6-7 and current WalkMates.

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

1. Explain why dependencies make tests slower, less predictable, and harder to diagnose.
2. Distinguish stubs, mocks, spies, fakes, and dummies.
3. Use behavior verification to test interactions with dependencies.
4. Explain regression testing, test selection, prioritization, and minimization.
5. Interpret mutation-testing results and design tests that kill surviving mutants.
6. Describe the red-green-refactor cycle of test-driven development.
7. Connect mocking, regression, mutation, and CI into a practical test-optimization workflow.

## Before You Read: Retrieve and Predict

1. Why can a covered branch still be protected by a weak test?
2. What wallet state must remain unchanged if an external top-up charge fails?

Revisit both answers after Sections 3 and 6.

## 1. The Optimization Problem

As a project grows, a test suite has to satisfy competing goals. It should find faults early,
run often, provide useful diagnostic information, protect important behavior, and avoid wasting
time on redundant or fragile checks. Test optimization is the discipline of improving that
feedback.

📘 **Concept: Test optimization.** Test optimization improves the value, speed, reliability,
and maintainability of testing activities. It includes isolating dependencies, prioritizing
tests, selecting relevant regression tests, removing redundancy, and assessing suite strength.

The goal is not simply "run fewer tests" or "make the pipeline green faster." The goal is to
run the right tests at the right time, with enough evidence to support the decision being made.

**WalkMates proxy.** A small local change to price calculation should trigger fast unit tests
for pricing, relevant booking-service tests, and perhaps a few workflow tests. It should not
need to call a real payment provider, send real notifications, or wait for an LLM response.

## 2. Dependencies and Test Doubles

Dependencies make tests realistic, but they can also make them slow, fragile, costly, or unsafe.
External payment services, databases, notification systems, authentication providers, file
systems, time, network calls, and LLM clients can all create problems:

- external systems may be unavailable,
- calls may be slow or costly,
- tests may create side effects,
- shared data may pollute later tests,
- failure cases may be hard to trigger reliably.

🛠️ **Technique: Test double.** A test double is a controlled replacement for a real
collaborator. It lets the test isolate the system under test and make dependency behavior
predictable.

| Test double | Purpose | WalkMates example |
|---|---|---|
| Stub | Return predetermined values | Payment client always approves or rejects |
| Mock | Verify expected interactions | Notification client should be called after booking |
| Spy | Use real behavior while recording calls | Repository spy records which query was made |
| Fake | Lightweight working implementation | In-memory repository instead of database |
| Dummy | Fill a parameter that is not used | Placeholder user object for an unrelated argument |

📘 **Concept: State verification and behavior verification.** State verification checks the
resulting state or output. Behavior verification checks that interactions happened as expected.
Mocks are especially useful for behavior verification.

Example distinction:

| Verification type | Question | Example assertion |
|---|---|---|
| State | What result did the method produce? | Wallet balance decreased by total price |
| Behavior | Which collaborator was called? | `NotificationService.sendBookingConfirmed(...)` was called once |

> **Common student mistake.** "Mock everything."
>
> **Better.** Mock dependencies that are slow, costly, nondeterministic, hard to set up, or
> whose interactions matter. Prefer real simple objects when they keep the test clearer.

## 3. Mocking Practice

Mocking frameworks such as Mockito let tests configure collaborator behavior and verify
interactions.

Typical mocking steps:

1. Create mocks for dependencies.
2. Configure expected responses or exceptions.
3. Execute the system under test.
4. Assert returned values or state changes.
5. Verify important interactions.

Representative example:

```java
@Test
void topUpCreditsWalletAfterSuccessfulCharge() throws PaymentService.PaymentException {
    Seeker seeker = new Seeker("ada@example.se", "Ada", "0701234567");
    when(seekers.findById(seeker.getId())).thenReturn(Optional.of(seeker));
    when(payments.charge(seeker.getId(), "card-1", 250.00)).thenReturn("tx-17");
    when(seekers.save(seeker)).thenReturn(seeker);

    Seeker result = service.topUp(seeker.getId(), "card-1", 250.00);

    assertThat(result.getBalance()).isEqualTo(250.00);
    verify(payments).charge(seeker.getId(), "card-1", 250.00);
}
```

Mocks are also valuable for testing error conditions:

```java
@Test
void declinedChargeLeavesWalletUnchanged() throws PaymentService.PaymentException {
    Seeker seeker = new Seeker("ada@example.se", "Ada", "0701234567");
    when(seekers.findById(seeker.getId())).thenReturn(Optional.of(seeker));
    when(payments.charge(seeker.getId(), "card-1", 250.00))
        .thenThrow(new PaymentService.PaymentException("declined"));

    assertThatThrownBy(() -> service.topUp(seeker.getId(), "card-1", 250.00))
        .isInstanceOf(PaymentService.PaymentException.class);
    assertThat(seeker.getBalance()).isZero();
    verify(seekers, never()).save(any());
}
```

Here, `service` is a `SeekerService` constructed with the mocked `seekers`, `payments`, and
`notifications` collaborators. A timeout test uses the same arrangement with
`PaymentService.PaymentTimeoutException`. These tests are hard to write with the real gateway
because making it fail on demand is not reliable. A mock makes the local failure behaviour
controlled and repeatable; a separate contract or integration test is still needed for the real
gateway.

🧭 **Principle: Verify behavior that matters.** Interaction verification should express a
contract. Verifying every internal call makes tests brittle. Verifying a payment, notification,
or no-call safety property can be valuable.

## 4. Regression Testing

📘 **Concept: Regression testing.** Regression testing checks whether changes have broken
behavior that previously worked. It is necessary because software evolves: new features, bug
fixes, refactoring, dependency updates, configuration changes, and data changes can all create
unexpected failures.

Regression testing has three common optimization strategies:

| Strategy | Main idea | Example |
|---|---|---|
| Test case selection | Run tests relevant to recent changes | Run pricing and booking tests after price-rule change |
| Test case prioritization | Run high-value tests earlier | Run historically failing or risk-critical tests first |
| Test suite minimization | Remove redundant tests while preserving a criterion | Remove duplicate tests that cover the same behavior and risk |

Regression testing is not only about coverage. Useful metrics include:

- code coverage,
- fault detection rate,
- test execution time,
- cost per fault detected,
- historical failure rate,
- flakiness rate.

🧱 **Artifact: Regression triage note.**

| Field | Example |
|---|---|
| Change | Modified booking capacity rule |
| Risk | Overbooking or false rejection |
| Tests selected | Booking capacity unit tests, booking-service integration tests |
| Tests prioritized first | Boundary tests at capacity and above capacity |
| Deferred tests | Unrelated AI explanation tests |
| Residual risk | UI wording and concurrency not covered by selected fast suite |

**WalkMates proxy.** If only `MatchExplanationService` prompt wording changes, the first
regression tests should focus on deterministic prompt construction, fallback behavior, and any
golden/metamorphic checks. Delimiter tests show prompt structure; provider-backed evaluation is
needed for claims about live prompt-injection robustness. A full booking-price suite may still
run later in CI, but it is not the most relevant first feedback.

## 5. Continuous Integration as Feedback Infrastructure

📘 **Concept: Continuous integration (CI).** CI automatically builds, tests, and reports on
software changes. It helps teams notice regressions quickly and creates a shared evidence trail.

CI is not magic. It is only as useful as the tests and checks it runs. A CI pipeline that runs
slow, flaky, weak tests may create delay without confidence. A useful testing pipeline is
layered:

| Stage | Typical checks | Purpose |
|---|---|---|
| Fast local checks | Unit tests, formatting, static analysis | Immediate developer feedback |
| Pull-request CI | Unit, integration, coverage, selected regression tests | Shared review evidence |
| Deeper scheduled checks | Mutation testing, long system tests, performance checks | Expensive evidence without blocking every edit |

🧭 **Principle: Match cost to cadence.** Fast tests can run on every edit. Expensive checks
such as mutation testing may run on selected targets, pull requests, or scheduled builds.

## 6. The Coverage Paradox and Mutation Testing

Coverage tells us what was executed. Mutation testing asks a different question: if the code
were slightly wrong, would the test suite notice?

📘 **Concept: Mutation testing.** Mutation testing injects small artificial faults into the
program and checks whether the tests fail. The artificial faulty versions are called mutants.

Common mutation operators include:

| Operator | Original | Mutated | Typical risk |
|---|---|---|---|
| Boundary change | `>=` | `>` | Off-by-one defect |
| Arithmetic replacement | `+` | `-` | Wrong calculation |
| Logical replacement | `&&` | `||` | Incorrect condition interaction |
| Return replacement | `true` | `false` | Weak assertion or missing case |
| Conditional removal | `if (condition)` | `if (false)` | Dead or untested path |

Mutation results use a useful vocabulary:

| Result | Meaning |
|---|---|
| Killed mutant | At least one test failed when the mutation was introduced |
| Survived mutant | Tests still passed; the suite may be weak |
| No coverage | Mutated code was not executed by tests |
| Timed out / non-viable / run or memory error | The mutant or analysis needs investigation |

🧭 **Principle: A surviving mutant is a question, not automatically a bug.** It may reveal a
missing test, a weak assertion, untested code, or an equivalent mutant. The tester must inspect
the mutant and decide what evidence is missing.

An **equivalent mutant** is a manual/conceptual classification: the change has no relevant
observable effect. PIT does not report “equivalent” as an automatic outcome.

**WalkMates proxy.** If a mutant changes top-up rejection from `amount < MIN_TOP_UP` to
`amount <= MIN_TOP_UP` and the tests still pass, the suite is missing the accepted boundary
`10.00`. A test for exactly `10.00` should kill that mutant.

PIT exposes related but different metrics:

| Metric | Definition in this course |
|---|---|
| WalkMates CI mutation score | killed / all generated mutants |
| PIT test strength | killed / (killed + survived), excluding no-coverage mutants |
| Adjusted theoretical mutation adequacy | killed / non-equivalent mutants, only after a documented human classification |

## 7. PIT Workflow

PIT is a mutation testing tool for Java. A typical Maven workflow is:

```bash
mvn clean test org.pitest:pitest-maven:mutationCoverage
```

The report usually includes line coverage, mutation coverage, test strength, killed mutants,
survived mutants, and no-coverage mutants. The useful workflow is:

1. Run the current test suite.
2. Run PIT on a focused package or class.
3. Inspect survived mutants first.
4. Ask whether the mutant represents a meaningful behavioral difference.
5. Add or improve a test with a clear oracle.
6. Re-run the relevant tests and mutation analysis.

🧪 **Exercise: Kill-the-mutant artifact.** Choose one surviving mutant and write:

1. Original expression.
2. Mutated expression.
3. Why the existing tests did not fail.
4. The test case that should kill it.
5. Whether the mutant is meaningful or probably equivalent.

## 8. Test-Driven Development

📘 **Concept: Test-driven development (TDD).** TDD is a development process where tests are
written before the production code they require. The classic cycle is red, green, refactor.

| Phase | Meaning | Evidence |
|---|---|---|
| Red | Write a test and confirm the expected failure | The test can fail in the current state for the observed reason; the oracle still needs review |
| Green | Write the simplest code that passes | The selected example now passes; the full behaviour is not yet established |
| Refactor | Improve design without changing behavior | Passing tests protect the behavior during cleanup |

TDD can improve focus, design feedback, and refactoring confidence. It can also feel slow or
awkward at first, and it does not automatically produce complete testing. A TDD suite still
needs review for missing boundaries, weak assertions, and integration risks.

**WalkMates proxy.** The minimum top-up rule is a good TDD sequence: start with `10.00` accepted,
then add `9.99` rejected, the 5,000.00 single-top-up maximum, and the 20,000.00 resulting-balance
maximum. Each step adds one behaviour before refactoring duplicated setup.

## 9. Integrated Workflow for Lab 2

M4 connects several testing ideas into one feedback loop:

1. Use mocks or fakes to isolate dependencies.
2. Run targeted regression tests after changes.
3. Inspect coverage to see what was exercised.
4. Run mutation testing on critical logic.
5. Add tests that kill meaningful surviving mutants.
6. Let CI report the evidence.

For WalkMates, use complementary targets rather than forcing every technique onto one class:

| Concern | Example evidence |
|---|---|
| Mocking | `PaymentService` or `NotificationService` is controlled in service tests |
| Regression | Relevant tests selected after a pricing or booking change |
| Coverage | `PricingCalculator` and status transitions have interpreted branch evidence |
| Mutation | Meaningful boundary and arithmetic mutants are killed |
| CI | Build summary reports tests, coverage, and mutation score |

🧪 **Lab 2 preparation artifact.** Prepare one page with:

1. Target class or feature.
2. Dependency to mock and why.
3. One regression scenario and selected tests.
4. One surviving mutant or expected mutant.
5. One test you will add to improve the suite.
6. CI evidence you expect to collect.

## 10. Summary and Self-Check

Test optimization is about better feedback. Test doubles make tests more controllable.
Regression strategies help decide what to run after change. CI makes feedback shared and
repeatable. Mutation testing checks whether tests detect generated mutants. Deliberate defect
seeding is a related but different evaluation technique. TDD uses tests to guide development,
but still needs thoughtful test design.

Before continuing, check whether you can answer these questions:

1. Why can real dependencies make unit tests weak or unsafe?
2. What is the difference between a stub and a mock?
3. When is behavior verification useful?
4. How do selection, prioritization, and minimization differ?
5. Why might mutation testing be too expensive to run everywhere?
6. What does a survived mutant mean?
7. What is the red-green-refactor cycle?
8. Which evidence should CI produce for Lab 2?

## References and Further Reading

[R1] Gerard Meszaros. *xUnit Test Patterns: Refactoring Test Code*. Addison-Wesley, 2007.

[R2] Mockito Project. *Mockito Documentation*. https://site.mockito.org/

[R3] PIT. *Mutation Testing for Java*. https://pitest.org/

[R4] Yue Jia and Mark Harman. "An Analysis and Survey of the Development of Mutation Testing."
*IEEE Transactions on Software Engineering*, 37(5), 2011.
https://doi.org/10.1109/TSE.2010.62

[R5] Shin Yoo and Mark Harman. "Regression testing minimization, selection and prioritization:
a survey." *Software Testing, Verification and Reliability*, 22(2), 2012.
https://doi.org/10.1002/stvr.430

[R6] Kent Beck. *Test-Driven Development: By Example*. Addison-Wesley, 2002.

[R7] GitHub. *Building and testing Java with Maven: GitHub Actions documentation*.
https://docs.github.com/en/actions/use-cases-and-examples/building-and-testing/building-and-testing-java-with-maven
