# Chapter 5: Research Trends and Testing AI

> Draft v0.2 (2026-08-08). Revised against HT26 Lecture 9 and verified primary sources.

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

1. Explain the difference between using AI for testing and testing AI-enabled systems.
2. Describe major research trends in AI-augmented software testing.
3. Map AI support across the testing lifecycle.
4. Identify risks of AI-generated tests, including weak oracles, hallucinated assertions, data
   leakage, and overreliance.
5. Design deterministic, fallback, metamorphic, golden-set, and robustness tests for an AI
   feature.
6. Compare a practitioner source with a research source and use the comparison to justify a
   test improvement.

## Before You Read: Retrieve and Predict

An AI tool produces ten passing tests. Before reading further, list three reasons why the suite
may still be weak and one independent signal that could reveal each weakness. Revisit the list
after Section 4.

## 1. The AI-Augmented Testing Era

Software testing is entering an AI-augmented era. Large language models and related techniques
can suggest test ideas, generate unit tests, summarize failures, explain logs, propose fixes,
write documentation, and support agent-like workflows that generate, execute, critique, and
revise tests.

This does not make testing less important. It changes what testers must evaluate. The tester is
no longer only asking whether the system under test behaves correctly. The tester must also ask
whether the AI-generated testing work is useful, valid, reproducible, ethical, and grounded in
the real test basis.

📘 **Concept: AI for testing.** AI for testing uses AI techniques to support testing work:
planning, test design, generation, execution, debugging, maintenance, reporting, and analysis.

📘 **Concept: Testing AI systems.** Testing AI systems evaluates software whose behavior
depends on AI or machine-learning components. The challenge is often that expected behavior is
probabilistic, context-sensitive, or difficult to express as one exact output.

The distinction matters:

| Direction | Main question | WalkMates example |
|---|---|---|
| AI for testing | Can AI help us design, run, repair, or explain tests? | Ask an LLM to propose boundary tests for booking duration |
| Testing AI systems | Is an AI-enabled feature behaving acceptably? | Test `MatchExplanationService` explanations, fallback, and deterministic prompt-structure controls |

🧭 **Principle: Test the tester.** AI can help generate testing work, but that work still needs
review, execution, oracles, and human judgment.

## 2. Research Trends in Software Testing

Recent research and technical guidance on AI and software engineering describe several active
streams [R1-R6]. The exact tool landscape changes quickly, but the underlying testing questions
are stable.

| Trend | What it tries to improve | Testing concern |
|---|---|---|
| Test generation and repair | Produce or update tests automatically | Are generated tests meaningful and executable? |
| Defect prediction and vulnerability detection | Predict where faults or security problems may occur | Are predictions calibrated and actionable? |
| Debugging and log analysis | Explain failures and rank root causes | Are explanations grounded in evidence? |
| Test documentation and summarization | Translate technical evidence into readable reports | Are summaries accurate and complete? |
| Agentic testing systems | Use multi-step AI workflows to plan, run, critique, and revise | Who verifies the agent's choices and side effects? |
| Flaky-test detection | Identify tests that pass and fail nondeterministically | Is failure caused by code, test, environment, timing, or data? |
| Search-based software testing | Use search or optimization to find inputs and test suites | Is the fitness function aligned with real risk? |

This chapter focuses on the AI-facing parts because they connect directly to Lab 3. Flakiness
and search-based software testing remain important research trends and can be strong project
topics.

## 3. AI Across the Testing Lifecycle

AI support can appear throughout the testing process.

| Testing activity | AI-supported tasks | Human responsibility |
|---|---|---|
| Requirements and planning | Identify ambiguities, risks, acceptance criteria, quality attributes | Check against real stakeholder needs and test basis |
| Test case design | Suggest equivalence partitions, boundaries, data, assertions | Verify oracles and missing cases |
| Execution and debugging | Summarize logs, localize failures, suggest fixes | Reproduce evidence and avoid accepting plausible guesses |
| Evaluation and maintenance | Detect redundant, flaky, outdated, or weak tests | Decide what evidence is still needed |
| Reporting and knowledge | Summarize coverage, defects, and rationale | Ensure claims match actual results |

AI is most useful when paired with traditional evidence. Static analysis, dynamic execution,
coverage reports, mutation reports, regression histories, and human domain knowledge can all
constrain AI suggestions.

🧭 **Principle: Hybrid evidence beats fluent text.** A confident AI explanation is not testing
evidence by itself. Evidence comes from execution, oracles, traceability, review, and justified
reasoning.

Use independent signals together: requirement traceability, assertion review, coverage change,
mutation-based fault sensitivity, repeated execution, and comparison with a human-designed
baseline. Agreement among several weak signals is still not proof, but disagreement is a useful
prompt for investigation.

> **Common student mistake.** "The AI generated a long test suite, so it must be comprehensive."
>
> **Better.** A generated suite may overrepresent happy paths, miss boundaries, assert the
> implementation instead of the requirement, or include tests that do not compile.

## 4. AI-Generated Unit Tests

AI tools can generate plausible unit tests quickly. This is useful for brainstorming, learning a
new API, creating initial scaffolds, and noticing cases a human forgot. It is also risky because
the generated tests may:

- use nonexistent methods or wrong imports,
- assert behavior that is not specified,
- miss edge cases,
- duplicate weak examples,
- encode current bugs as expected behavior,
- hide test-basis assumptions,
- pass without meaningful assertions.

🛠️ **Technique: Generated-test critique.** When reviewing AI-generated tests, ask:

1. What is the test basis?
2. What behavior is asserted?
3. Does the test compile and run?
4. Which partitions and boundaries are missing?
5. Are negative and exceptional paths included?
6. Does the assertion check the requirement or merely the current implementation?
7. What residual risk remains?

**WalkMates proxy.** If an AI proposes only `60`-minute and `120`-minute booking-duration tests,
the suite is weak. It has missed the boundary values `29`, `30`, `1440`, and `1441`, which are
more likely to reveal off-by-one defects.

## 5. Testing AI-Enabled Features

AI-enabled features often have an oracle problem. There may not be one exact correct output.
Two explanations can be different and both acceptable. A recommendation can be useful without
being deterministic. A generated summary can be mostly correct but omit a critical risk.

📘 **Concept: Oracle problem in AI testing.** The oracle problem occurs when it is difficult to
decide whether an observed output is correct, acceptable, safe, or useful.

Testing AI-enabled features therefore relies on several complementary strategies.

Start by separating system layers:

| WalkMates layer | Behaviour | Suitable first oracle |
|---|---|---|
| `recommendBestMatch` | Deterministic ranking from structured fields | Exact result plus order-invariance relation |
| `buildPrompt` | Deterministic prompt construction | Exact required fields and untrusted-data boundaries |
| `LlmClient.complete` | Model-generated match explanation | Repeated, rubric-based evaluation |
| Fallback and HTTP controller | Recovery and interface contract | Exact exception/fallback/status assertions |

The current LLM generates an **explanation**; deterministic code selects the Listing.

### 5.1 Deterministic Unit Tests

Not every part of an AI feature is nondeterministic. Prompt construction, data filtering,
fallback selection, formatting, and policy checks can often be tested deterministically.

**WalkMates proxy.** `MatchExplanationService` builds a prompt from the Seeker trust tier and
structured Listing fields. Test prompt construction separately. Assert that required fields are
included and the Listing description is placed inside the declared untrusted-data boundary.
This establishes prompt structure, not the behaviour of a live model.

### 5.2 Fallback and Failure Tests

AI calls can fail, time out, return malformed output, or become unavailable. A robust system
must define what happens then.

🛠️ **Technique: Fallback testing.** Configure the AI client mock to signal failure or timeout,
then assert that the system returns the specified safe fallback and avoids leaking internal
details. This proves handling of a signalled exception; it does not prove that a real client
enforces a deadline. Test deadline enforcement at a provider-backed or integration boundary.

Lab 3 also checks the HTTP contract with `MockMvc`: a valid request, including the service's
declared fallback result, produces the expected success status and response shape; a missing
Seeker or Listing produces the specified `404` response.

### 5.3 Metamorphic Testing

🛠️ **Technique: Metamorphic testing.** When exact expected output is hard to define, identify
relations that should hold across related inputs.

Metamorphic relations must name one controlled input transformation and an observable output
relation. In WalkMates, useful relations include:

| Relation | Input change | Expected relation |
|---|---|---|
| Order invariance | Reorder candidate Listings | Selected best Listing is unchanged |
| Irrelevant-detail stability | Change only text that deterministic ranking does not use | Selected best Listing is unchanged |
| Prompt-structure preservation | Add instruction-like text inside Listing description | Prompt control text and untrusted-data boundary remain structurally unchanged |

Avoid “should not change substantially” unless a similarity rule, tolerance, or review rubric is
defined. Metamorphic tests are partial oracles, but they are valuable when exact output equality
is too brittle.

### 5.4 Golden Sets

🧱 **Artifact: Golden set.** A golden set is a curated collection of inputs and expected
properties or approved outputs. It gives a stable regression reference for behavior that is
difficult to specify fully.

For AI output, a golden set should usually avoid exact long-string equality. Instead, it can
check required facts, forbidden content, tone constraints, safety constraints, or rubric scores.

Treat the set as a versioned evaluation artifact. Document why each item is included, who
labelled it, reviewer disagreement, subgroup coverage, and possible leakage into prompts or model
training. Keep evaluation examples held out where feasible. Golden sets are lecture enrichment;
follow the current Lab 3 instructions for the required artifacts.

### 5.5 Prompt-Injection and Robustness Tests

AI features may process untrusted text. A listing description, user profile, or message can
contain instructions such as "ignore previous instructions" or "reveal hidden policy." The test
question is whether the system treats that text as data, not as authority.

🛠️ **Technique: Prompt-injection test.** Include adversarial text in an untrusted field and
observe whether system policy, data access, output validation, or tool behaviour changes.
Delimiters and standing instructions are useful prompt-construction controls, but they do not
establish live-model resistance by themselves. Stronger evidence also covers least privilege,
tool/input/output validation, isolation, and audit logging.

### 5.6 Repeated Evaluation and Reproducibility

Model behaviour needs a protocol, not a single demonstration. Before evaluation, define:

- a held-out, versioned dataset and important subgroups,
- acceptance criteria and thresholds,
- number of repeated trials and sampling configuration,
- model, prompt, retrieval, and tool versions,
- how labels and reviewer disagreement are handled,
- uncertainty and residual risk in the conclusion.

Record enough context to reproduce an observation: model and prompt version, configuration,
retrieval sources, tool calls, latency, failures, retries, fallback, evaluation labels, and
reviewer disagreement. Aggregate results should include subgroup sample sizes because an overall
average can hide concentrated harm.

### 5.7 Agentic Systems

An agent can plan, call tools, change state, and continue across steps. Test more than the text
response:

- permission and least-privilege boundaries,
- confirmation before consequential actions,
- idempotency, retry, and duplicate-action behaviour,
- maximum steps, time, and cost,
- recovery after partial completion,
- traceability from decision through tool call to state change.

An agentic extension is not part of the current WalkMates implementation, but it is a valid
project context and a useful way to see why component, system, security, and recovery evidence
must be combined.

## 6. Benefits and Risks of AI-Augmented Testing

AI can reduce routine effort, broaden idea generation, help novice testers, and improve
documentation. These benefits are real but context-dependent. They should be measured against
actual project evidence rather than assumed.

Risks include:

- hallucinated tests, APIs, and expected results,
- fragile tests that pass only by coincidence,
- data leakage through prompts,
- overreliance that weakens tester judgment,
- biased or incomplete generated examples,
- reproducibility problems when models change,
- sustainability costs from repeated AI use.

🧭 **Principle: Keep humans accountable.** AI tools can contribute suggestions, but humans are
responsible for the claims made from those suggestions.

## 7. Reading Research Critically

Lab 3 asks you to compare sources and use research to justify a test improvement. A useful
comparison does not merely summarize two texts. It asks what kind of evidence each source
provides.

| Question | Why it matters |
|---|---|
| What problem is being studied? | Avoids applying a result outside its scope |
| What system, dataset, or tasks were used? | Shows whether the evidence transfers to WalkMates |
| What metrics were used? | Reveals what "better" means |
| What baselines were compared? | Prevents overclaiming |
| What threats to validity are acknowledged? | Shows limitations |
| What practical guidance follows? | Connects research to your test improvement |

🧱 **Artifact: Trend comparison.** A strong mini-review includes one practitioner source, one
academic or standards source, a comparison of what each contributes, and one concrete testing
decision justified by that comparison.

## 8. Activity and Lab 3 Preparation

🧪 **Activity: AI-generated test critique artifact.** Ask an AI tool to generate tests for a
moderately complex function. Produce:

1. Three tests it generated well.
2. Three missing or weak cases.
3. One weak assertion or oracle.
4. One improved prompt.
5. One hand-designed test that is stronger than the generated version.

🧪 **Lab 3 preparation artifact.** For the WalkMates AI feature, prepare:

1. One deterministic unit test for prompt construction or input filtering.
2. One fallback or timeout test using a mock AI client.
3. One metamorphic relation.
4. One `MockMvc` HTTP-contract test.
5. One prompt-structure security test for the required untrusted-data delimiters, with a written statement of what that test does not establish.
6. One research or practitioner source that justifies a test improvement.

🧪 **Lecture extension artifact.** Add one small, versioned golden-set item with an input,
expected property or rubric, label rationale, and known limitation. This meets the chapter outcome
but is not an additional Lab 3 requirement.

A live-model injection evaluation or small repeated evaluation is recommended enrichment when
the chosen claim cannot be judged with an exact deterministic oracle.

## 9. Summary and Self-Check

AI changes testing by adding new tools, new risks, and new systems under test. It can generate
test ideas, but generated tests must be reviewed. It can summarize failures, but summaries must
be grounded in execution evidence. It can power features, but AI-enabled behavior needs special
testing strategies because exact oracles may be unavailable.

Before continuing, check whether you can answer these questions:

1. What is the difference between AI for testing and testing AI systems?
2. Why are exact assertions often difficult for AI-generated explanations?
3. What is a metamorphic relation?
4. Why is fallback behavior important for AI features?
5. What is a prompt-injection test?
6. What makes a generated test suite weak?
7. How can research evidence justify a test improvement?
8. Why should AI-generated testing work be treated as a draft rather than evidence?

## References and Further Reading

[R1] A. Fan, B. Gokkaya, M. Harman, M. Lyubarskiy, S. Sengupta, S. Yoo, and J. M. Zhang.
"Large language models for software engineering: Survey and open problems." *IEEE/ACM
ICSE-FoSE*, 2023. https://doi.org/10.1109/ICSE-FoSE59343.2023.00008

[R2] Junjie Wang, Yuchao Huang, Chunyang Chen, Zhe Liu, Song Wang, and Qing Wang. "Software
Testing With Large Language Models: Survey, Landscape, and Vision." *IEEE Transactions on
Software Engineering*, 50(4), 911-936, 2024. https://doi.org/10.1109/TSE.2024.3368208

[R3] Tsong Yueh Chen, Fei-Ching Kuo, Huai Liu, Pak-Lok Poon, Dave Towey, T. H. Tse, and
Zhi Quan Zhou. "Metamorphic Testing: A Review of Challenges and Opportunities." *ACM Computing
Surveys*, 51(1), Article 4, 2018. https://doi.org/10.1145/3143561

[R4] Chloe Autio et al. *Artificial Intelligence Risk Management Framework: Generative
Artificial Intelligence Profile*. NIST AI 600-1, 2024.
https://doi.org/10.6028/NIST.AI.600-1

[R5] OWASP GenAI Security Project. *OWASP Top 10 for LLM Applications 2025*.
https://genai.owasp.org/resource/owasp-top-10-for-llm-applications-2025/

[R6] Cristian Augusto, Antonia Bertolino, Guglielmo De Angelis, Francesca Lonetti, and Jesús
Morán. "Large Language Models for Software Testing: A Research Roadmap." arXiv preprint
arXiv:2509.25043, 2025. https://doi.org/10.48550/arXiv.2509.25043
