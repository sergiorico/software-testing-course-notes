# Chapter 2: Specification-Based and Unit Testing

> Draft v0.2 (2026-08-08). Revised against HT26 Lectures 3-4 and current WalkMates.

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

1. Explain the difference between static, dynamic, and specification-based testing.
2. Use static review techniques to find defects before execution.
3. Derive test cases using equivalence partitioning and boundary value analysis.
4. Design decision-table, state-transition, and use-case tests for rule-heavy behavior.
5. Translate a designed test case into a JUnit 5 unit test using Arrange-Act-Assert.
6. Choose between single-case and parameterized tests.
7. Prepare the specification-based part of Lab 1 with clear test basis, conditions, cases, and
   oracles.

## Before You Read: Retrieve and Predict

1. Write the six boundary values for a 30..1440 inclusive interval.
2. Predict when a decision table is stronger than testing each condition separately.

After Sections 4-6, check both answers against the worked examples and correct them.

## 1. From Test Basis to Test Design

Specification-based testing begins with a simple discipline: before writing test code, identify
what the test is based on. The test basis may be a requirement, user story, contract, regulation,
design model, bug report, or stakeholder expectation. The tester studies that basis and turns it
into test conditions, test data, expected outcomes, and eventually executable tests.

📘 **Concept: Specification-based testing.** Specification-based testing designs tests from
requirements and externally visible behavior rather than from the implementation. It asks what
the system should do, what it should reject, and what outcomes stakeholders expect.

Specification-based testing is often called black-box testing because it treats the internal code
as hidden. That does not mean testers never look at code. It means the design logic of the test
comes from the specification. Code can later be inspected, instrumented, or executed, but the
oracle comes from the expected behavior.

📘 **Concept: Static, dynamic, and specification-based perspectives.**

| Perspective | What is examined | Program executed? | Typical purpose |
|---|---|---|---|
| Static testing | Requirements, models, code, tests, manuals, pull requests | No | Find defects early and improve artifacts |
| Dynamic testing | Running software behavior | Yes | Observe actual behavior under selected conditions |
| Specification-based testing | Requirements and externally visible behavior | Not necessarily; design can precede execution | Derive cases from expected behavior rather than implementation |

These perspectives complement each other. A requirement review may reveal that a booking rule is
ambiguous. A specification-based test may show that an implementation violates the clarified
rule. A structural coverage report may later show that some code branch was never exercised.

**WalkMates proxy.** A requirement such as "a booking duration must be between 30 and 1440
minutes inclusive" is a test basis. The lower boundary, upper boundary, below-minimum case, and
above-maximum case are test conditions. Concrete values such as `29`, `30`, `1440`, and `1441`
become test data.

> **Common student mistake.** "Specification-based testing means guessing examples from the
> requirements."
>
> **Better.** Specification-based testing is systematic. It uses techniques that explain why a
> small set of examples is strong enough to provide useful evidence.

## 2. Static Testing: Defects Before Execution

Testing does not have to wait for runnable code. Static testing evaluates artifacts without
executing the program. It can be applied to requirements, design sketches, database schemas,
source code, test cases, manuals, and generated AI output. Static testing is valuable because it
can find defects before they are embedded in code and before many downstream decisions depend on
them.

🛠️ **Technique: Review.** A review is a human evaluation of an artifact. Reviews vary in
formality:

| Review type | Typical form | Strength | Example artifact |
|---|---|---|---|
| Informal review | Quick peer comments | Cheap and fast | A teammate skims a test class |
| Walkthrough | Author explains artifact and receives questions | Shared understanding | A developer walks through a booking rule |
| Technical review | Prepared expert review | Technical accuracy and standards | Review of API contracts or validation rules |
| Inspection | Formal roles, checklist, logging, metrics | Rigorous defect detection | Requirement or safety-critical design inspection |

Modern code review is often tool-supported and asynchronous. A pull request can expose code,
tests, comments, discussions, CI results, and review decisions in one place. Modern code reviews
can find defects, improve code quality, transfer knowledge, support onboarding, and make the
team aware of changes [R3]. They can also become expensive when changes are too large, reviewers
are overloaded, or the purpose of the review is unclear.

AI-supported review tools can summarize changes, suggest likely defects, or connect code to
issue-tracker context. The important distinction is support versus replacement. If review is
fully automated, the team may lose interpersonal benefits such as shared ownership, design
discussion, and knowledge transfer.

🧪 **Exercise: Static review artifact.** Choose one WalkMates requirement or test case. Write:
one ambiguity, one missing condition, one likely defect if the ambiguity reaches code, and one
question you would ask in a review.

## 3. Equivalence Partitioning

Software often accepts large input spaces. Equivalence partitioning reduces that space by
grouping inputs that should be treated similarly. Instead of testing every value, testers choose
representatives from each partition.

🛠️ **Technique: Equivalence partitioning (EP).** Divide the input domain into valid and invalid
classes. Select representative values from each class because values inside the same class are
expected to exercise similar behavior.

For a simple duration rule, the partitions are clear:

| Partition | Meaning | Representative |
|---|---|---|
| Invalid low | Duration below the minimum | `29` |
| Valid | Duration between 30 and 1440 inclusive | `60` |
| Invalid high | Duration above the maximum | `1441` |

Missing, non-numeric, or malformed duration values form additional partitions at an API or user-
interface parsing boundary. They are not values in the integer domain accepted by the
`Booking` constructor.

The power of EP is not that one representative proves the entire partition correct. It is that
the partition makes the tester's assumption visible: "I believe these values are handled by the
same logic." If the assumption is weak, the partition should be split.

**WalkMates proxy.** Trust tiers, listing statuses, wallet states, and booking durations all
have natural partitions. For example, wallet balance can be partitioned into less than total
price, exactly total price, and greater than total price.

## 4. Boundary Value Analysis

Many defects occur at the edges of ranges: one less than the minimum, exactly the minimum, one
above the minimum, one below the maximum, exactly the maximum, and one above the maximum. This is
why boundary value analysis is often paired with equivalence partitioning.

🛠️ **Technique: Boundary value analysis (BVA).** Identify edges in ordered domains and test
values on and around those edges.

For `FR-4.1`, the useful boundary set is:

| Purpose | Value | Expected result |
|---|---:|---|
| Just below lower boundary | `29` | Reject |
| Lower boundary | `30` | Accept |
| Just above lower boundary | `31` | Accept |
| Just below upper boundary | `1439` | Accept |
| Upper boundary | `1440` | Accept |
| Just above upper boundary | `1441` | Reject |

The values `30` and `1440` matter because the requirement is inclusive. A test suite that checks
only `60` minutes and `2000` minutes may look reasonable but can miss an off-by-one defect.

> **Common student mistake.** "I tested a value inside the valid range, so I tested the range."
>
> **Better.** A normal valid value tests one representative. Boundaries test where programmers
> commonly make comparison mistakes.

🧪 **Exercise: EP/BVA artifact.** Choose one numeric WalkMates rule. Produce a table with
partitions, representatives, boundary values, and expected outcomes.

## 5. Decision Table Testing

Some behavior depends on several conditions at once. In that situation, individual boundary
tests are not enough because the risk lives in combinations. Decision table testing makes those
combinations explicit.

🛠️ **Technique: Decision table testing.** List conditions, list actions, and create rules that
map condition combinations to expected outcomes.

A simplified FR-4.4 booking decision, preserving first-failing-reason order, is:

| Condition / action | Accept | Listing | Seeker | Provider | Duration | Balance |
|---|---:|---:|---:|---:|---:|---:|
| Listing is available | Y | N | Y | Y | Y | Y |
| Seeker is below active-booking limit | Y | Any | N | Y | Y | Y |
| Provider is below capacity | Y | Any | Any | N | Y | Y |
| Duration is within 30..1440 | Y | Any | Any | Any | N | Y |
| Total price is within wallet balance | Y | Any | Any | Any | Any | N |
| **Expected action** | accept | reject listing | reject seeker | reject provider | reject duration | reject balance |

“Any” means “does not matter for this rule” because an earlier condition already determines
the outcome. This kind of table helps find missing rules, duplicated rules, contradictory rules,
and unclear rejection precedence.

🧪 **Exercise: Decision-table artifact.** Build a decision table for a WalkMates rule with at
least three conditions. Each rule must have one expected action and, when rejected, one primary
reason.

## 6. State Transition Testing

Many systems are not defined only by one input. They are defined by state and allowed movement
between states. A state-transition defect occurs when the system allows an invalid transition or
blocks a valid one.

🛠️ **Technique: State transition testing.** Model behavior as states and transitions. Test
valid transitions, invalid transitions, and important sequences.

For WalkMates, the **Booking** lifecycle in FR-4.2 is:

```text
REQUESTED -> CONFIRMED -> IN_PROGRESS -> COMPLETED
REQUESTED -> CANCELLED
CONFIRMED -> CANCELLED
```

Invalid transitions are just as important:

| Current state | Attempted transition | Expected result |
|---|---|---|
| `COMPLETED` | `CONFIRMED` | Reject; completed booking is closed |
| `CANCELLED` | `IN_PROGRESS` | Reject; cancelled booking cannot start |
| `IN_PROGRESS` | `REQUESTED` | Reject; cannot move backward |
| `REQUESTED` | `COMPLETED` | Reject; booking must be confirmed and started first |

Do not confuse this with the separate **Listing** state machine in FR-3.2, whose states include
`AVAILABLE` and `BOOKED`.

State-transition testing is especially useful for workflows, lifecycles, approval processes,
and resource management.

## 7. Use Case Testing

Use case testing starts from realistic user goals. Instead of asking only whether a single input
is accepted, it asks whether a user can complete a meaningful workflow and whether alternative
or exceptional paths are handled.

🛠️ **Technique: Use case testing.** Derive tests from user scenarios: main success path,
alternative paths, error paths, and recovery paths.

Example use case: a verified seeker books an available dog walk.

| Path type | Scenario | Expected result |
|---|---|---|
| Main success | Listing available, duration valid, capacity available, wallet sufficient | Booking is created and confirmation is shown |
| Alternative | Wallet balance exactly equals total price | Booking is created and balance becomes zero |
| Error | Duration is `29` minutes | Booking is rejected with duration message |
| Error | Provider capacity is full | Booking is rejected with capacity message |
| Recovery at service level | Wallet insufficient; `SeekerService.topUp` succeeds; booking is retried | Second attempt can succeed |

Use case tests are often broader than unit tests. They can become system or acceptance tests if
they cross user interface, API, service, database, and notification layers.

## 8. From Designed Case to Unit Test

Unit testing turns small behavioral expectations into executable evidence. A unit test focuses
on a small part of the system, such as a method, class, or small module. It should be automated,
fast, repeatable, and isolated enough that failures are easy to diagnose.

📘 **Concept: Unit test.** A unit test checks a small unit of behavior in isolation. It is most
valuable when the expected behavior is precise and the test can run quickly without external
systems.

A unit is selected around coherent behaviour; it is not forced to be exactly one method or one
class. The useful boundary is the smallest one that keeps the behaviour observable and the
failure diagnostic.

🧭 **Principle: Test the contract, not the implementation.** Two implementations can satisfy
the same behavior. A good unit test should usually verify the externally expected result, not
the exact internal algorithm.

JUnit 5 tests commonly follow the Arrange-Act-Assert pattern:

```java
@Test
void minimumTopUpIsAccepted() {
    // Arrange
    Seeker seeker = new Seeker("ada@example.se", "Ada", "0701234567");

    // Act
    seeker.addFunds(10.00);

    // Assert
    assertThat(seeker.getBalance()).isEqualTo(10.00);
}
```

The pattern keeps the test readable:

| Phase | Question | Example |
|---|---|---|
| Arrange | What objects, data, and preconditions are needed? | Create a valid `Seeker` |
| Act | What behavior is executed? | Call `addFunds(10.00)` |
| Assert | What should be true afterward? | Assert the wallet contains 10.00 SEK |

Parameterized tests are useful when several inputs follow the same structure:

```java
@ParameterizedTest
@ValueSource(doubles = {10.00, 100.00, 5_000.00})
void acceptedTopUpsCreditTheWallet(double amount) {
    Seeker seeker = new Seeker("ada@example.se", "Ada", "0701234567");
    seeker.addFunds(amount);
    assertThat(seeker.getBalance()).isEqualTo(amount);
}
```

Single-case tests are easier to diagnose when one edge case fails. Parameterized tests are more
compact when the same rule applies to many values. A useful test suite often uses both.

Compare two oracles for a created booking:

```java
// Weak: almost any returned object satisfies this.
assertThat(booking).isNotNull();

// Stronger: checks behaviour promised by the booking contract.
assertThat(booking.getStatus()).isEqualTo(BookingStatus.CONFIRMED);
assertThat(booking.getPrice()).isCloseTo(expectedPrice, within(0.005));
```

The stronger version is still incomplete if wallet, Listing state, persistence, or notification
effects are part of the test's stated objective. The tolerance reflects the current `double` API;
a decimal money type with an explicit rounding contract would support stronger exact assertions.
The snippet assumes AssertJ's static `within` import.

## 9. Fixtures, Assertions, and Good Unit-Test Style

🧱 **Artifact: Test fixture.** A fixture is the setup needed to execute tests: objects, test
data, fake collaborators, database state, files, or configuration. In JUnit 5, `@BeforeEach`
can create fresh setup before every test, while `@BeforeAll` is used for class-level setup.

Assertions are the oracle in executable form. Common JUnit assertions include equality
assertions, truth assertions, null assertions, exception assertions, grouped assertions, and
timeouts.

Good unit tests tend to share several properties:

| Property | Why it matters |
|---|---|
| Independent | Tests should not depend on execution order |
| Descriptive | Names should explain behavior and condition |
| Focused | One failing test should point to a small behavior |
| Fast | Unit tests should support frequent local and CI execution |
| Deterministic | Same code and same setup should produce same result |
| Meaningful assertions | Executing code without checking behavior gives weak evidence |

> **Common student mistake.** "A long test is a strong test."
>
> **Better.** A strong unit test is focused, readable, and has a clear oracle. Long setup can
> hide the behavior being tested.

## 10. Activity and Lab 1 Preparation

🧪 **Activity: Live test-design challenge artifact.** Given a small input specification,
produce:

1. Valid and invalid equivalence partitions.
2. Boundary values.
3. At least one expected result for each test case.
4. One case that an AI-generated list missed or explained weakly.
5. One allowed and one forbidden Booking state-transition sequence, including the expected final state.
6. One use-case path with preconditions, actions, expected result, and remaining interface assumption.

🧪 **Lab 1 preparation artifact.** Prepare the artifacts used by the actual `Seeker` target:

1. EP tables for email, display name, and phone (`FR-1.1`).
2. BVA at 9.99/10.00/10.01 and around the 5,000.00 single-top-up maximum (`FR-1.3`).
3. One case where a top-up would exceed the 20,000.00 resulting-balance maximum.
4. A trust-tier decision table for fee and maximum concurrent bookings (`FR-1.2`).
5. JUnit/AssertJ tests in `SeekerSpecBasedTest` using Arrange-Act-Assert and explicit oracles.

The combined FR-4.4 table remains useful enrichment for later booking-service work; it is not
the primary Lab 1 implementation target.

## 11. Summary and Self-Check

This chapter moves from test ideas to test design. Static testing finds defects before
execution. Specification-based techniques explain how to choose test cases from expected
behavior. Unit testing turns selected cases into fast, repeatable evidence.

Before continuing, check whether you can answer these questions:

1. What is the difference between static and dynamic testing?
2. What makes a review more formal than an informal peer check?
3. Why does equivalence partitioning reduce effort?
4. Which values would you choose for a 30-1440 inclusive boundary?
5. When is a decision table more useful than a list of separate test cases?
6. What is the difference between a state-transition test and a use-case test?
7. What do Arrange, Act, and Assert mean?
8. When would a parameterized test be clearer than many separate tests?

## References and Further Reading

[R1] Mauricio Aniche. *Effective Software Testing: A Developer's Guide*. Manning, 2022.
https://www.manning.com/books/effective-software-testing

[R2] Kshirasagar Naik and Priyadarshi Tripathy. *Software Testing and Quality Assurance: Theory
and Practice*. Wiley, 2011.

[R3] D. Badampudi, M. Unterkalmsteiner, and R. Britto. "Modern code reviews: survey of
literature and practice." *ACM Transactions on Software Engineering and Methodology*, 32(4),
2023. https://doi.org/10.1145/3585004

[R4] Lo Gullstrand Heander, Emma Söderberg, and Christofer Rydénfält. "Support, Not
Automation: Towards AI-Supported Code Review for Code Quality and Beyond." *Proceedings of the
33rd ACM International Conference on the Foundations of Software Engineering Companion
(FSE Companion '25)*, 2025. Ideas and vision paper.
https://doi.org/10.1145/3696630.3728505

[R5] JUnit Team. *JUnit 5 User Guide*. https://junit.org/junit5/docs/current/user-guide/

[R6] AssertJ. *AssertJ Core documentation*. https://assertj.github.io/doc/
