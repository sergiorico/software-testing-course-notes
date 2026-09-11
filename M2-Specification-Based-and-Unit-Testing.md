# Chapter 2: Specification-Based and Unit Testing

> Revised v0.3 (2026-09-11). Standalone conceptual reading with source-linked WalkMates examples.
>
> Examples use WalkMates commit `0a00e35b2b312a6765cced1602f5306a05ebbb0c` [R8].
> Requirements supply expected behaviour. Code supplies the public API, not an independent oracle.

## Learning Outcomes

By the end of this chapter, you should be able to:

1. Explain the difference between static, dynamic, and specification-based testing.
2. Use static review techniques to find defects before execution.
3. Derive test cases using equivalence partitioning and boundary value analysis.
4. Design decision-table, state-transition, and use-case tests for rule-heavy behavior.
5. Translate a designed test case into a JUnit 5 unit test using Arrange-Act-Assert.
6. Choose between single-case and parameterized tests.
7. Evaluate the strength and limitations of an executable test's evidence.

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

Static and dynamic testing classify whether the software under test executes. Specification-based
and structural techniques classify how cases are selected. These are different dimensions, not
three competing categories. A requirement-derived JUnit test is both specification-based and
dynamic when executed [R7, §§3.1, 4.1]. These perspectives complement each other.
A requirement review may reveal that a booking rule is
ambiguous. A specification-based test may show that an implementation violates the clarified
rule. A structural coverage report may later show that some code branch was never exercised.

**WalkMates example.** A requirement such as "a booking duration must be between 30 and 1440
minutes inclusive" is a test basis. The lower boundary, upper boundary, below-minimum case, and
above-maximum case are test conditions. Concrete values such as `29`, `30`, `1440`, and `1441`
become test data.

> **Common misconception.** "Specification-based testing means guessing examples from the
> requirements."
>
> **Better.** Specification-based testing is systematic. It uses techniques that explain why a
> selected set of examples provides particular evidence while leaving other behaviour unexamined.

## 2. Static Testing: Defects Before Execution

Testing does not have to wait for runnable code. Static testing evaluates artifacts without
executing the program. It can be applied to requirements, design sketches, database schemas,
source code, test cases, manuals, and generated AI output. Static testing is valuable because it
can find defects before they are embedded in code and before many downstream decisions depend on
them.

Static analysis uses tools to inspect properties of source code or other structured artifacts.
The analysis tool executes, but the software under test does not execute as it would in a dynamic
test. A finding can identify an issue such as a suspicious data flow or convention violation.
It can also be a false alarm. A clean report leaves unexamined properties outside the tool's
analysis. Human review contributes domain understanding, intent, and questions about omissions [R7, §3.1].

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

A focused change, relevant reviewer expertise, and a clear requirement help make review useful.
Mechanical checks can reduce style noise, while discussion addresses behaviour and trade-offs.
These are practical responses to review challenges, not a guarantee of a fixed productivity gain.

Heander and colleagues propose AI-supported review that retains human understanding and team
benefits [R4]. Their paper is an ideas and vision contribution with a proposed architecture and
research agenda. It does not establish that a particular AI review tool outperforms human review.
Generated summaries and findings still need checks against the actual artifact and contract.

### Review process and responsibilities

A review begins with scope, artifact version, criteria, and participants. Initiation makes the
context available. Reviewers then inspect individually and record anomalies. Communication and
analysis distinguish confirmed defects, open questions, and other improvements. Correction and
reporting close the loop by checking agreed actions and completion criteria [R7, §3.2].

The author supplies the artifact and context. A review leader organises the review, a moderator
facilitates discussion, reviewers examine the material, and a recorder captures findings.
In the CTFL inspection model the author cannot also act as review leader or recorder [R7, §3.2.4].
Less formal reviews may combine responsibilities. A review evaluates the work product, not the
personal worth or performance of its author.

**Worked review example.** Consider the deliberately vague sentence “accept a booking when the
user is below the limit and can afford it.” It leaves several questions unanswered. FR-1.2 and
FR-4.2 define the tier maximum and which statuses count as active. FR-4.4 requires the active count
to be strictly below its limit and permits price equality with the wallet balance. It also defines
which rejection takes precedence. The vague sentence is a teaching paraphrase, not the current
requirement wording [R8].

**Worked code review example.** The public `Seeker.addFunds` implementation rejects
`amount < MIN_TOP_UP`. FR-1.3 accepts the minimum itself. A hypothetical change to `<=` would
violate the rule. The 10.00 test detects that change dynamically, while review can identify the
conflict without executing it. Review also needs the surrounding method: passing the minimum
does not waive the maximum transaction or resulting-balance limits [R8].

**Responsibility matters.** FR-1.1 assigns email uniqueness to `SeekerService`, which can query
stored accounts. The `Seeker` constructor checks field format and length. Terms acceptance is
outside this teaching model. A constructor-only test cannot establish service-level uniqueness [R8].

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

**WalkMates example.** Trust tiers, listing statuses, wallet states, and booking durations all
have natural partitions. For example, wallet balance can be partitioned into less than total
price, exactly total price, and greater than total price.

Partitions should cover the selected domain without overlap. They may describe outputs or states
as well as inputs. Their adequacy depends on whether the hypothesised similarity in behaviour is
justified [R1, R7]. Testing one representative of each duration class covers three selected
partitions. It does not establish all boundary values or combinations with other fields.

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

> **Common misconception.** "I tested a value inside the valid range, so I tested the range."
>
> **Better.** A normal valid value tests one representative. Boundaries test where programmers
> commonly make comparison mistakes.

### Boundary criteria and controlled preconditions

For the two specified limits, two-value BVA selects `29, 30, 1440, 1441`. Three-value analysis
around those limits adds the inside neighbours `31` and `1439`. State which boundaries and
neighbours form the coverage model before reporting a percentage. Additional partitions can
introduce additional boundaries [R7, §4.2.2].

Wallet tests require state as well as an amount. The following examples use cent-valued amounts:

| Constraint | Starting balance | Amounts | Expected outcome |
|---|---:|---|---|
| Minimum single top-up | 0.00 | 9.99, 10.00, 10.01 | Reject, accept, accept |
| Maximum single top-up | 0.00 | 4999.99, 5000.00, 5000.01 | Accept, accept, reject |
| Maximum resulting balance | 15010.00 | 4989.99, 4990.00, 4990.01 | Accept, accept, reject |

The last row holds the single-top-up constraint valid while moving the resulting balance across
20,000.00. The rejected case must leave the original 15,010.00 unchanged. Public top-ups of
5,000.00 three times followed by 10.00 establish that starting state [R8].

Amounts with more precision introduce rounding questions. For example, 10.005 rounds to 10.01
under the half-up rule, with no transaction-boundary conflict. At a transaction limit, however,
the contract should clarify whether validation applies before or after rounding. The current
`addFunds` implementation checks the raw amount before rounding the resulting balance. That is
an observed implementation order, not a substitute for clarifying an ambiguous requirement [R8].

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

A table should distinguish irrelevant conditions from infeasible combinations. Five Boolean
conditions yield 32 abstract combinations, but concrete domain constraints may rule some out.
The compressed FR-4.4 table groups combinations with the same first failing condition. One
feasible test per rule covers that rule model. Additional cases with competing failures test
whether rejection precedence is implemented correctly [R7, §4.2.3; R8].

For example, a provider-rejection case needs an available Listing, a Seeker below the tier limit,
a Provider already at capacity, a valid duration, and sufficient funds. A second case with both
an unavailable Listing and a full Provider should report the Listing failure first. These are
expected results from the specification, not a claim that a service run has been observed.

A smaller decision table maps one mutually exclusive input to two outputs:

| Trust tier | Maximum concurrent active bookings | Platform fee |
|---|---:|---:|
| NEW | 1 | 15% |
| VERIFIED | 3 | 12% |
| TRUSTED | 5 | 8% |
| PRO_SITTER | 10 | 5% |

Each row is one FR-1.2 rule. The test should check both outputs against literal requirement data.
Deriving expected values by calling the production getter would fail to check the mapping [R8].

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

### State and transition coverage

Visiting every state does not necessarily exercise every valid transition. A sequence that
cancels a REQUESTED Booking visits CANCELLED but does not exercise cancellation from CONFIRMED.
All-valid-transition coverage requires each allowed edge. All-transition coverage also exercises
attempted invalid transitions in the selected state model [R7, §4.2.4].

An invalid case should establish its source state through legal operations, attempt one forbidden
transition, and check both rejection and unchanged state. For example, construct a Booking,
confirm it, start it, complete it, and then attempt confirmation again. The contract requires an
`IllegalStateException`, a remaining COMPLETED status, and inactive classification [R8].
Null is an additional API input rather than a state in this five-state model.

## 7. Use Case Testing

Use case testing starts from realistic user goals. Instead of asking only whether a single input
is accepted, it asks whether a user can complete a meaningful workflow and whether alternative
or exceptional paths are handled.

🛠️ **Technique: Use case testing.** Derive tests from user scenarios: main success path,
alternative paths, error paths, and recovery paths.

Example use case: a verified seeker books an available dog walk.

| Path type | Scenario | Expected result |
|---|---|---|
| Main success | Listing available, duration valid, capacity available, wallet sufficient | Service returns a confirmed Booking; UI confirmation requires separate interface evidence |
| Alternative | Wallet balance exactly equals total price | Booking is created and balance becomes zero |
| Error | Duration is `29` minutes | Booking is rejected with duration message |
| Error | Provider capacity is full | Booking is rejected with capacity message |
| Recovery at service level | Wallet insufficient; `SeekerService.topUp` succeeds; booking is retried | Second attempt can succeed |

Use case tests are often broader than unit tests. They can become system or acceptance tests if
they cross user interface, API, service, database, and notification layers.

Use-case tests need explicit preconditions, actor actions, and observable postconditions.
A service test can check returned state, wallet effects, and repository state. It cannot establish
that a UI message appeared. A broader test must cross the interface whose behaviour it claims
to evaluate. Distinguish desired behaviour from observed implementation results [R1, R8].

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

The examples use JUnit Jupiter and AssertJ supplied by the WalkMates Maven configuration [R8].
Maven must run with Java 21. From the repository root, `mvn -v` checks the runtime, and
`mvn -Dtest=SeekerSpecBasedTest test` runs the existing focused test class.

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

A second-top-up test illustrates fault sensitivity. Arrange a wallet with 100.00, add 25.00,
and expect 125.00. A test that starts from zero might not distinguish addition from replacement.
The non-zero setup makes that difference observable without checking private fields [R8].

For a state sequence, `new Booking("s-1", "l-1", 60)` begins in REQUESTED. Calls to
`transitionTo(CONFIRMED)`, `transitionTo(IN_PROGRESS)`, and `transitionTo(COMPLETED)`
establish a completed Booking. Another confirmation must fail and leave state unchanged [R8].

## 9. Fixtures, Assertions, and Good Unit-Test Style

🧱 **Artifact: Test fixture.** A fixture is the setup needed to execute tests: objects, test
data, fake collaborators, database state, files, or configuration. In JUnit 5, `@BeforeEach`
can create fresh setup before every test, while `@BeforeAll` is used for class-level setup.

JUnit Jupiter's default lifecycle creates a new test-class instance per test. `@BeforeEach`
and `@AfterEach` surround each test or parameterised invocation. `@BeforeAll` and `@AfterAll`
run once per class and are normally static. With the explicitly selected PER_CLASS lifecycle,
they can be instance methods. Fresh instances do not isolate static fields or external state [R5].

A small factory returning a new valid Seeker removes repetitive registration data while leaving
the important top-up or starting balance visible. A shared mutable wallet can introduce
order dependence. Setup should be local, predictable, and no broader than the behaviour requires.

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

> **Common misconception.** "A long test is a strong test."
>
> **Better.** A strong unit test is focused, readable, and has a clear oracle. Long setup can
> hide the behavior being tested.

### Assertion choices and their limits

`assertEquals(expected, actual)` compares values, whereas `assertSame` checks reference
identity. `assertAll` reports related assertion failures together. `assertThrows` checks
the specified exception type or a subtype. It should enclose the intended action, not a broad
block whose setup might throw first. After a rejected top-up, also check that the balance is
unchanged. This checks a meaningful postcondition [R5, R8].

The expected number should come from the specification. A 10.005 top-up rounds to 10.01 under
FR-1.3. With the current double API, a very small tolerance such as 0.000001 accommodates
representation differences without allowing a one-cent error. Exact checks are suitable for
the simple exactly representable amounts used in other examples. AssertJ offers fluent
equivalents such as `assertThat(balance).isEqualTo(100.00)` and exception assertions [R6, R8].

A timeout assertion is a limited check, not a performance study. JUnit's `assertTimeout`
evaluates elapsed time after the operation completes and does not stop an endless operation.
Preemptive timeout assertions execute differently and can interact with thread-local state.
Tight timing bounds make tests sensitive to machine load [R5].

Parameterised tests should preserve one story. `@ValueSource` supplies individual values,
while `@CsvSource` can pair inputs with expected results such as the trust-tier table.
`@ParameterizedTest(name = "...{0}...")` creates input-aware invocation names. Putting
placeholders in `@DisplayName` does not provide that parameter substitution [R5].

## 10. Evaluating a Test Set

A test's value depends on the claim its assertions support. A call to `addFunds(10.00)`
followed only by `assertNotNull(seeker)` would pass even if the wallet never changed. Checking
the expected balance makes that missing update observable. A rejected-input test also needs
the relevant unchanged-state assertion.

For any test set, trace expected outcomes to the basis, inspect its partitions and boundaries,
check combinations and history, and identify plausible faults that would still escape.
Requirements and code generated from the same mistaken assumption can share a blind spot.
Independent oracle reasoning matters regardless of who or what proposed the tests.

Specification-based coverage describes the chosen model. Visiting every selected partition
does not establish every program path. Executing every line does not establish a correct oracle.
These measurements answer different questions and should be reported with their scope.

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
and Practice*. Wiley, 2008 (original publication, also available as a 2011 electronic edition).
https://doi.org/10.1002/9780470382844

[R3] D. Badampudi, M. Unterkalmsteiner, and R. Britto. "Modern code reviews: survey of
literature and practice." *ACM Transactions on Software Engineering and Methodology*, 32(4),
2023. https://doi.org/10.1145/3585004

[R4] Lo Gullstrand Heander, Emma Söderberg, and Christofer Rydénfält. "Support, Not
Automation: Towards AI-Supported Code Review for Code Quality and Beyond." *Proceedings of the
33rd ACM International Conference on the Foundations of Software Engineering Companion
(FSE Companion '25)*, 2025. Ideas and vision paper.
https://doi.org/10.1145/3696630.3728505

[R5] JUnit Team. *JUnit 5 User Guide*, version 5.12.2. Sections on assertions,
parameterised tests, lifecycle, and test-instance lifecycle.
https://docs.junit.org/5.12.2/user-guide/

[R6] AssertJ. *AssertJ Core documentation*. https://assertj.github.io/doc/

[R7] ISTQB. *Certified Tester Foundation Level Syllabus*, v4.0.1 (2024),
§3 Static Testing and §4.2 Black-Box Test Techniques. Section 4.2 covers EP, BVA,
decision tables, and state transitions. This chapter also retains use-case testing as broader
scenario design, not as a fifth technique listed in that section.
https://istqb.org/wp-content/uploads/2024/11/ISTQB_CTFL_Syllabus_v4.0.1.pdf

[R8] Sergio Rico. *WalkMates*, source snapshot verified 2026-09-11, commit
`0a00e35b2b312a6765cced1602f5306a05ebbb0c`.
[Requirements](https://github.com/sergiorico/walkmates-test/blob/0a00e35b2b312a6765cced1602f5306a05ebbb0c/docs/REQUIREMENTS.md),
[Seeker](https://github.com/sergiorico/walkmates-test/blob/0a00e35b2b312a6765cced1602f5306a05ebbb0c/src/main/java/com/walkmates/model/Seeker.java),
[Booking](https://github.com/sergiorico/walkmates-test/blob/0a00e35b2b312a6765cced1602f5306a05ebbb0c/src/main/java/com/walkmates/model/Booking.java),
[TrustTier](https://github.com/sergiorico/walkmates-test/blob/0a00e35b2b312a6765cced1602f5306a05ebbb0c/src/main/java/com/walkmates/model/TrustTier.java).
