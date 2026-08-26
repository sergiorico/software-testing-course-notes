# Chapter 3: Structural Testing and Coverage

> Draft v0.2 (2026-08-08). Revised against HT26 Lecture 5 and current WalkMates.

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

1. Explain test adequacy and why coverage is evidence, not a goal by itself.
2. Distinguish statement, branch/decision, path, and data-flow adequacy criteria.
3. Use boundary, exception, timeout, and loop cases as explicit heuristics unless a formal
   criterion is defined.
4. Use a control-flow view to identify missing tests.
5. Interpret a coverage report and decide what to test next.
6. Explain why 100% coverage can still miss defects.
7. Prepare the structural-testing part of Lab 2 using coverage evidence.

## Before You Read: Retrieve and Predict

1. What can a green line in a coverage report establish?
2. What can it **not** establish without an oracle?

After Section 3, revise your answer using the distinction between execution and checking.

## 1. Why Structural Testing Matters

Specification-based testing starts from requirements. Structural testing starts from the
implemented structure of the software. It asks which statements, decisions, paths, loops, and
data interactions have actually been exercised by tests.

📘 **Concept: Structural testing.** Structural testing designs or evaluates tests using the
internal structure of the software, especially code structure and control flow.

Structural testing does not replace specification-based testing. A test suite can execute every
line of code and still check the wrong requirement. A test suite can also be excellent from a
requirements perspective but leave a risky branch unexecuted. The two perspectives answer
different questions:

| Perspective | Main question |
|---|---|
| Specification-based | Have we tested the expected behavior? |
| Structural | Which implementation structures have we exercised? |

Structural testing is especially useful after a first suite exists. Coverage evidence can show
where the suite is blind, where dead code may exist, and where new tests should be targeted.

**WalkMates proxy.** `PricingCalculator` has paths for missing inputs, a free
`SHELTER_VOLUNTEER` Listing, and the overnight surcharge; its final calculation also applies the
trust-tier platform fee. Duration range validation belongs to the `Booking` model before pricing.
Specification-based tests tell us which rules should be tested; coverage tells us which
implementing structures executed.

## 2. Test Adequacy

📘 **Concept: Test adequacy.** Test adequacy is the degree to which a test set satisfies a
chosen criterion. A criterion may be based on requirements, risks, statements, branches, paths,
mutants, or other evidence.

Adequacy is about the quality and relevance of testing, not the number of tests. A suite with
three well-chosen boundary and exception tests may provide stronger evidence than twenty tests
that all execute the same happy path.

🧭 **Principle: Coverage guides improvement.** Coverage should help testers decide what to test
next. It should not become a scoreboard that rewards shallow tests.

> **Common student mistake.** "Higher coverage always means better testing."
>
> **Better.** Higher coverage can reveal progress, but only meaningful assertions and relevant
> cases turn execution into evidence.

## 3. Code Coverage as a Map

📘 **Concept: Code coverage.** Code coverage measures which parts of code were executed during
testing. Different criteria define different "parts": source statements or lines, branch
outcomes, atomic conditions, paths, or data-flow pairs.

JaCoCo reports **bytecode instructions, branches, source lines, methods, and classes**. Its line
counter is useful, but it is not a direct source-language statement counter.

Coverage can help testers:

- identify untested code,
- flag code for investigation as possibly unreachable or dead,
- focus new tests on gaps,
- compare suites over time,
- support refactoring confidence.

Coverage can also mislead testers:

- a test can execute code without assertions,
- a test can assert something too weak,
- a covered line can contain untested boundary behavior,
- a suite can reach 100% statement coverage but miss important branches or paths.

The key sentence is worth repeating: coverage tells us what was executed, not whether the
behavior was checked well.

## 4. Statement Coverage

🛠️ **Technique: Statement coverage.** Statement coverage asks whether each executable
statement has been executed by at least one test.

Statement coverage is easy to understand and often easy to measure. It is a useful first
indicator because code that is never executed cannot have its runtime behavior checked by the
current suite. However, statement coverage can miss branch behavior.

Consider a simplified rule:

```java
if (durationMinutes < 30 || durationMinutes > 1440) {
    throw new IllegalArgumentException("Invalid duration");
}
return totalPrice;
```

A test with `durationMinutes = 60` executes the return statement but never executes the
exception statement. A test with `durationMinutes = 29` executes the exception statement but not
the return. To cover both statements, the suite needs at least one valid and one invalid case.

**WalkMates proxy.** If a report shows the free-Listing return or overnight branch in
`PricingCalculator` is not executed, the next question is not "how do I increase the number?"
It is "which requirement or risk does this uncovered structure represent?"

## 5. Decision and Branch Coverage

🛠️ **Technique: Branch coverage.** Branch coverage asks whether each outcome of each decision
has been exercised. For a simple `if`, this means both the true and false outcomes.

For conventional reachable control flow, complete branch coverage normally demands more than
complete statement coverage because tests must exercise alternative outcomes. The exact counters
reported by a tool are not interchangeable. Branch evidence is especially useful for validation
logic, pricing rules, authorization, state transitions, and error handling.

Example decisions for a booking validator might include:

| Decision | True case | False case |
|---|---|---|
| `durationMinutes < 30` | `29` | `30` |
| `durationMinutes > 1440` | `1441` | `1440` |
| `walletBalance >= totalPrice` | enough balance | insufficient balance |
| `providerCapacity > activeBookings` | capacity available | capacity full |

A branch report can reveal that tests only exercise "accept booking" paths and never exercise
rejection paths. That is a weak suite even if the happy path is covered many times.

### Condition coverage is a different question

A decision can contain several atomic conditions:

```java
if (durationMinutes < 30 || durationMinutes > 1440) {
    throw new IllegalArgumentException("Invalid duration");
}
```

Tests at `29` and `60` exercise the decision's true and false outcomes, but the atomic condition
`durationMinutes > 1440` is never true. A `1441` case adds condition evidence. Decision/branch
coverage and condition coverage therefore answer related but different questions.

**Calculation check.** If a report shows 18 of 24 instructions and 4 of 6 branch outcomes,
instruction coverage is `18 / 24 = 75%` and branch coverage is `4 / 6 ≈ 66.7%`. Those numbers
still do not reveal whether assertions are meaningful or requirements are covered.

## 6. Path Coverage and Independent Paths

🛠️ **Technique: Path coverage.** Path coverage asks whether execution paths through the
control-flow graph have been exercised.

Path coverage is powerful because defects often depend on sequences of decisions. It is also
expensive because the number of paths can grow quickly, especially with loops and nested
conditions. Binary conditions describe input combinations, not automatically feasible
control-flow paths. An early-return function with four decisions has up to sixteen truth-value
combinations but only five feasible outcome paths. For practical testing, we often focus on
independent paths: paths that add at least one new edge or decision outcome to the set already
covered.

For a simplified booking validator:

```text
P1: invalid duration -> reject
P2: valid duration, unavailable listing -> reject
P3: valid duration, available listing, insufficient wallet -> reject
P4: valid duration, available listing, sufficient wallet -> accept
```

These paths are not the whole system. They are a practical structural model that helps decide
which tests provide distinct evidence.

🧱 **Artifact: Coverage mapping table.**

| Path | What it exercises | Suggested test |
|---|---|---|
| P1 | Duration validation fails | `durationMinutes = 29` |
| P2 | Listing-state rejection | Listing status `BOOKED` |
| P3 | Wallet rejection | Balance below total price |
| P4 | Successful booking | Available listing and enough balance |

## 7. Cyclomatic Complexity

📘 **Concept: Cyclomatic complexity.** Cyclomatic complexity is a structural measure of the
number of linearly independent paths through a program. A common simplified rule is: number of
decisions plus one.

Cyclomatic complexity does not tell us that a method is good or bad by itself. It tells us that
there are more paths to reason about. A method with many decisions may need more tests, clearer
decomposition, or both.

For a method with four decisions, a simplified control-flow model gives a rough complexity of
five. The `decisions + 1` shortcut depends on how compound Boolean expressions and the graph are
counted. Complexity prompts explicit path reasoning; it does not prescribe an exact minimum test
count or measure code quality by itself.

🧭 **Principle: Complexity creates testing obligations.** When code contains more decisions,
test design should become more explicit. A single happy-path test rarely carries enough
evidence for complex logic.

## 8. Data-Flow Coverage (Extension)

🛠️ **Technique: Data-flow coverage.** Data-flow coverage focuses on where variables are
defined, where they are used, and whether important definition-use pairs are exercised.

Data-flow testing is useful when defects involve values being initialized, transformed, stored,
and later used incorrectly. Pricing, age calculation, discount calculation, and state-update
logic often contain data-flow risks.

Example data-flow questions for WalkMates use the real pricing names:

- Where is `baseCost` defined, and where is it used?
- Is `overnightExtra` applied only after duration is known?
- Are `subtotal`, `fee`, and the rounded total used consistently?
- Can a rejection path leave partially updated booking or wallet state?

Data-flow coverage pushes testers beyond "did this line run?" and toward "did this value move
through the system correctly?"

## 9. Boundary, Exception, Timeout, and Loop Heuristics

Structural testing is often combined with practical test-design heuristics from earlier
chapters. Unless a precise adequacy criterion is defined, boundary, timeout, exception, and loop
"coverage" below should be read as prompts rather than JaCoCo counters.

| Heuristic | What it asks | WalkMates example |
|---|---|---|
| Boundary-case heuristic | Have edge values been executed? | `29`, `30`, `1440`, `1441` |
| Exception-path heuristic | Have expected error paths been executed? | Invalid duration throws or returns validation error |
| Timeout heuristic | Does behavior complete within acceptable time? | A signalled AI-client timeout selects fallback; a separate time-bound integration test checks deadline enforcement |
| Loop heuristic | Have zero, one, two, many, and limit iterations been considered? | Candidate Listing scans and booking-history collections |

Loop coverage often uses practical values: skip the loop, run once, run twice, run many times,
and test around maximum limits (`n-1`, `n`, `n+1`). For nested loops, avoid combinatorial
explosion by varying one loop while holding others at simple values.

## 10. Reading Coverage Reports

Coverage tools such as JaCoCo report which instructions, lines, branches, and methods were
covered by tests. The useful workflow is not "open the report and chase green." A better
workflow is:

1. Run the test suite.
2. Open the coverage report.
3. Identify an uncovered or partially covered decision in a relevant class.
4. Ask which requirement, risk, or failure mode that decision represents.
5. Add a meaningful test with a clear assertion.
6. Re-run tests and inspect both the result and the changed coverage.

🧱 **Artifact: Coverage improvement note.** A good coverage note should include the uncovered
element, why it matters, the added test, the expected behavior, and the remaining risk.

Example:

| Field | Example |
|---|---|
| Uncovered element | Branch where wallet balance equals total price |
| Risk | Off-by-one money check could reject exact payment |
| Added test | Booking with `walletBalance == totalPrice` |
| Oracle | Requirement says sufficient balance covers the total |
| Remaining risk | Does not test concurrent bookings or stale provider-capacity information |

## 11. Activity and Lab 2 Preparation

🧪 **Activity: Coverage gauntlet artifact.** Given a small function, produce:

1. One input that covers a first path.
2. The statements and branches reached by that input.
3. One additional input that covers an uncovered branch.
4. A short explanation of why 100% statement coverage may still be insufficient.

🧪 **Lab 2 preparation artifact.** Start with an actual Lab 2 target such as
`PricingCalculator`, `ListingStatus`, or `BookingStatus`, and prepare:

1. Target class and reason for selecting it.
2. Current coverage observation from the report.
3. One uncovered or weakly covered branch.
4. One test you will add, with input and expected outcome.
5. One sentence explaining why the new test is meaningful rather than merely increasing a
   metric.

## 12. Summary and Self-Check

Structural testing gives visibility into what the current suite exercises. Statement, branch,
path, and data-flow criteria answer different structural questions. Boundary, exception, timeout,
and loop heuristics prompt additional cases unless a precise adequacy criterion is defined.
Coverage is useful when it guides better tests; it is dangerous when it becomes a target detached
from assertions and risk.

Before continuing, check whether you can answer these questions:

1. What does test adequacy mean?
2. Why can a test suite with high statement coverage still be weak?
3. What is the difference between statement and branch coverage?
4. Why is full path coverage often impractical?
5. How does cyclomatic complexity help test planning?
6. What is a definition-use pair?
7. What should you do after finding an uncovered branch in JaCoCo?
8. What makes a coverage-improvement test meaningful?

## References and Further Reading

[R1] Mauricio Aniche. *Effective Software Testing: A Developer's Guide*. Manning, 2022.
https://www.manning.com/books/effective-software-testing

[R2] Paul Ammann and Jeff Offutt. *Introduction to Software Testing*, 2nd edition. Cambridge
University Press, 2016.

[R3] Thomas J. McCabe. "A Complexity Measure." *IEEE Transactions on Software Engineering*,
SE-2(4), 1976. https://doi.org/10.1109/TSE.1976.233837

[R4] JaCoCo. *Java Code Coverage Library*. https://www.jacoco.org/jacoco/
