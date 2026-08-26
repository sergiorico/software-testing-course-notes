# Chapter 6: Ethical and Societal Aspects of Software Testing

> Draft v0.2 (2026-08-08). Revised against Lecture 8, the HT26 Lab 4 brief, and current
> sustainability and accessibility references.

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

1. Explain why sustainability, equity, diversity, inclusion, and ethics belong in software
   quality discussions.
2. Describe sustainability as a multidimensional concern rather than only an environmental one.
3. Distinguish Green IT, sustainable software, AI for sustainability, and sustainability of AI.
4. Distinguish direct, enabling, and systemic sustainability effects.
5. Use the Sustainability Awareness Framework (SusAF) to identify threats, opportunities, and
   actions.
6. Propose accessibility, equity, privacy, and sustainability evidence with an explicit system
   boundary and baseline.
7. Reason about ethical trade-offs in AI-augmented testing.
8. Prepare the Lab 4 analysis as a paper-based evidence artifact.

## 1. Why Testing Has Societal Consequences

Software testing is often presented as a technical activity: inputs, outputs, assertions,
coverage, tools, and reports. Those are necessary, but they are not the whole story. Software
quality has consequences for people, organizations, infrastructure, and the environment.

When software fails, some groups may be harmed more than others. When software is inefficient,
the cost may appear as energy use, cloud bills, slow devices, or premature hardware
replacement. When AI-supported testing is used carelessly, sensitive data may leak into prompts,
weak tests may be accepted without judgment, or teams may become dependent on tools they do not
understand.

📘 **Course working concept: Societal quality.** In this course, *societal quality* is a
convenient label for asking how software affects people and communities beyond narrow functional
correctness. It is not presented as a standardized quality-model term. It includes
sustainability, accessibility, fairness, inclusion, accountability, privacy, safety, and
long-term maintainability.

🧭 **Principle: Quality includes consequences.** A system can pass functional tests and still
create negative effects through exclusion, energy waste, dark patterns, unsafe automation, or
poor maintainability.

**WalkMates proxy.** A booking system can be technically correct but still create unfair effects
if rejection messages are unclear, if accessibility is poor, if trust-tier limits systematically
burden newcomers, or if AI explanations expose sensitive information.

## 2. Sustainability as a Software Concern

The widely used Brundtland definition describes sustainable development as meeting present needs
without compromising future generations' ability to meet their own needs [R1]. In software
engineering, that definition becomes practical when we ask how design, implementation,
operation, testing, maintenance, and retirement affect long-term social, environmental,
economic, technical, and individual conditions.

📘 **Concept: Sustainability in software engineering.** Sustainability is the capacity of a
software-intensive system and its surrounding practices to support long-term well-being across
multiple dimensions.

Sustainability is not only about electricity. It also involves:

- whether the system remains maintainable,
- whether it excludes or burdens some users,
- whether it encourages wasteful use,
- whether it depends on short-lived hardware,
- whether it creates hidden labor or social costs,
- whether its business model rewards harmful behavior.

Testing contributes by making these effects visible. A tester may not decide the whole product
strategy, but testing artifacts can expose risks, trade-offs, and missing evidence.

### 2.1 Direct, Enabling, and Systemic Effects

Sustainability analysis needs a time horizon and a view beyond the software process itself. A
useful model distinguishes three orders of effect [R7]:

| Effect | Focus | WalkMates or testing example |
|---|---|---|
| Direct | Resources and consequences of building and operating the system | Compute, storage, devices, CI runners, test data, and retained artifacts |
| Enabling | Changes in what users and organizations can do | Easier matching changes how seekers find care and how providers allocate time |
| Systemic | Wider and longer-term changes to structures and behavior | Dependence on one platform, exclusion of low-history users, or rebound from increased use |

The categories are analytical aids, not predictions. A claim should identify the affected
stakeholders, causal assumptions, evidence, uncertainty, and time scale. A locally efficient
feature can still increase total impact if it makes the activity much more frequent.

## 3. Green IT and Sustainable Software

📘 **Concept: Green IT.** Green IT focuses on reducing the environmental impact of information
technology, including energy use, hardware use, cooling, data centers, networks, and electronic
waste [R2].

📘 **Concept: Sustainable software.** Sustainable software considers how software design and
operation influence sustainability outcomes. Efficient algorithms, caching, data minimization,
modular architecture, and maintainable code can all reduce waste or extend useful life.

Examples:

| Design choice | Possible positive effect | Possible negative effect |
|---|---|---|
| Caching | Less repeated computation and network traffic | Stale data, privacy retention, storage growth |
| AI test generation | Faster exploration of test ideas | Energy use, weak tests, data leakage |
| Push notifications | Useful reminders and engagement | Attention burden, overuse, dark patterns |
| Modular architecture | Easier maintenance and replacement | More integration complexity |
| Rich media | Better interaction for some users | Higher bandwidth and device demands |

🧭 **Principle: Efficiency is not automatically sustainability.** An efficient feature that
encourages harmful overuse can still be unsustainable. Sustainability needs systemic thinking.

## 4. The Karlskrona View: Five Dimensions

The Karlskrona Manifesto argues that sustainability should be treated as a fundamental concern
in software engineering [R5]. It encourages thinking across several dimensions rather than
collapsing sustainability into only environmental impact.

| Dimension | Focus | Testing and quality questions |
|---|---|---|
| Environmental | Energy, emissions, water, materials, ecosystems | Does the system waste computation or data transfer? |
| Economic | Financial viability, resource allocation, long-term value | Does the system create avoidable cost or lock-in? |
| Social | Communities, institutions, trust, participation | Who benefits, who is excluded, and who bears risk? |
| Individual | Health, dignity, autonomy, learning, privacy | Does the system support or burden individual users? |
| Technical | Maintainability, evolvability, resilience, longevity | Can the system be changed and operated sustainably? |

These dimensions can conflict. A feature may improve convenience but increase energy use. A
strict fraud rule may reduce financial loss but unfairly block some users. A faster AI-assisted
testing workflow may save human time but increase tool dependence and resource use.

## 5. Sustainable AI

📘 **Concept: AI for sustainability.** AI for sustainability uses AI to support sustainability
goals, such as optimizing energy systems, reducing waste, improving logistics, or monitoring
environmental change.

📘 **Concept: Sustainability of AI.** Sustainability of AI examines the environmental, social,
economic, and technical costs of building, deploying, and maintaining AI systems [R4].

AI systems can require substantial computation, data, hardware, cooling, human labeling,
monitoring, and repeated experimentation. The impact depends on model size, hardware, data
center energy sources, cooling, usage volume, development practices, and geographic context.

For testing, this creates a practical trade-off. AI-assisted testing can help find defects or
generate ideas, but repeated large-model calls for trivial checks may be wasteful. A sustainable
testing strategy uses AI where it adds value and simpler deterministic techniques where they are
enough.

**WalkMates proxy.** Use AI to critique a complex match-explanation test strategy or generate
edge-case ideas. Do not use AI calls inside every ordinary unit test when a simple deterministic
assertion would do.

### 5.1 Make Resource Claims Comparable

Statements such as "this test strategy uses less energy" are incomplete unless the comparison is
defined. Before drawing a conclusion, record:

- the **system boundary**: local machine, CI infrastructure, model API, network, and retained data,
- the **baseline** and alternative being compared,
- a **functional unit**, such as one validated change or 1,000 evaluated cases,
- workload, repetitions, model and software versions, hardware, location, and time period,
- which impacts are measured directly and which are estimated,
- whether savings could create a rebound effect through increased use.

Runtime, API-call count, transferred bytes, compute time, cost, and retained storage can be useful
proxies. No single proxy establishes overall sustainability across all five dimensions.

## 6. Sustainability as a Quality Attribute

Sustainability can be treated as a cross-cutting software-quality concern. Lago et al. argue for
framing sustainability as a property of software quality, connected to economic, social,
environmental, and technical dimensions [R3]. Sustainability is **not** one of the nine top-level
product-quality characteristics in ISO/IEC 25010:2023 [R9]; the framework adds a wider system and
time perspective rather than silently changing the ISO model.

Examples of sustainability-related quality questions:

| Quality concern | Testable or reviewable question |
|---|---|
| Environmental | Does a feature avoid unnecessary repeated API calls? |
| Technical | Can the test suite run reliably without fragile external dependencies? |
| Social | Do rejection messages support user trust and understanding? |
| Individual | Is the interface accessible to users with different abilities? |
| Economic | Does the system avoid expensive unnecessary AI calls? |

🧭 **Principle: Make sustainability observable.** If a sustainability concern cannot be
measured directly, testers can still create proxy evidence: logs, usage counts, resource
profiles, accessibility checks, review findings, or SusAF impact maps.

## 7. The Sustainability Awareness Framework

🛠️ **Technique: SusAF.** The Sustainability Awareness Framework helps teams identify positive
and negative sustainability effects across five dimensions: social, individual, environmental,
economic, and technical [R6].

SusAF is useful because it encourages chains of effects. A feature can have an immediate
benefit, enable a later change in behavior, and contribute to a wider systemic cost or
opportunity.

🧱 **Artifact: SusAF impact entry.**

| Field | Example |
|---|---|
| Feature or decision | AI-generated match explanation |
| Dimension | Individual |
| Immediate positive effect | User understands why a listing may fit |
| Enabling or longer-term risk | User overtrusts an explanation that hides uncertainty |
| Affected stakeholders | Seekers, providers, platform maintainers |
| Possible action | Show concise rationale and use a clear fallback when the explanation service fails |

Good SusAF work should include threats, opportunities, actions, and affected stakeholders. It
should not be a list of generic positives and negatives. The point is to reason about the system.

## 8. Equity, Diversity, Inclusion, and Accessibility

Ethical testing also asks who is represented in test cases, data, requirements, and review
conversations. A system can appear correct for the majority case while failing users with
different languages, abilities, devices, socioeconomic conditions, identities, or contexts of
use.

Testing can support equity and inclusion by checking:

- accessibility behavior,
- language clarity,
- assumptions in sample data,
- fairness of trust-tier limits and matching logic,
- whether error messages blame users unnecessarily,
- whether workflows require resources not all users have,
- whether AI outputs behave differently for different groups.

**WalkMates proxy.** Trust-tier limits can disproportionately reject newcomers with little
history, and a validator that accepts only Swedish phone formats excludes otherwise valid users
outside the assumed context. These may be intentional scope decisions, but they should be
visible, justified, and tested. Tests alone cannot establish fairness; they can reveal subgroup
differences, hidden assumptions, missing explanations, and weak recovery paths.

### 8.1 Accessibility Requires Several Kinds of Evidence

WCAG 2.2 provides testable success criteria for web accessibility under the principles
perceivable, operable, understandable, and robust [R8]. Useful WalkMates evidence can include:

- automated checks for machine-detectable issues such as missing names or some contrast failures,
- keyboard-only operation, visible focus, zoom, reflow, text scaling, and error recovery,
- checks with relevant browsers, devices, and assistive technologies,
- manual inspection of understandable labels, instructions, status messages, and time limits,
- evaluation with disabled users when the decision and resources permit it.

An automated score covers only part of accessibility, and a conforming interface can still be
difficult in a particular context of use. Record the standard, level, scope, tools, manual
checks, user involvement, and remaining limitations behind an accessibility claim.

## 9. Ethics of AI-Augmented Testing

AI-augmented testing raises ethical questions beyond technical correctness.

| Ethical issue | Testing implication |
|---|---|
| Privacy and confidentiality | Do not paste sensitive code, data, or user information into tools without approval |
| Accountability | Generated tests and summaries need human ownership |
| Bias | Generated examples may underrepresent some users or risks |
| Skill erosion | Students and teams must still understand the tests they submit |
| Reproducibility | Model changes can make results difficult to reproduce |
| Sustainability | Repeated AI calls have resource costs |
| Labor and professionalism | AI should support judgment, not hide responsibility |

🧭 **Principle: Automation does not remove responsibility.** If an AI tool proposes a test,
summary, or defect explanation, the person using it remains responsible for evaluating and
communicating the evidence.

Follow the current **Generative AI instructions in Canvas**, which are authoritative for allowed
course use. Never send student work, assessment data, credentials, confidential code, or
identifiable user data to an unapproved public AI service. Prefer synthetic or properly governed
test data, and remember that synthetic data can still reproduce sensitive patterns or identities
if it was derived carelessly.

> **Common student mistake.** "AI use is ethical if the answer looks correct."
>
> **Better.** Ethical use depends on privacy, attribution, allowed tool use, environmental cost,
> human understanding, and the consequences of acting on the output.

## 10. Activity and Lab 4 Preparation

🧪 **Activity: SusAF analysis artifact.** For one familiar software product, produce:

1. At least one relevant effect in each of the five SusAF dimensions.
2. One chain containing a direct, an enabling, and a systemic effect.
3. One stakeholder who benefits and one stakeholder who may be harmed.
4. Two tests or measurements, each with a boundary and baseline.
5. One mitigation action and one uncertainty that remains.

🧪 **Lab 4 preparation artifact.** For your chosen software or AI system, prepare:

1. A short system description and stakeholder list.
2. A SusAF map across social, individual, environmental, economic, and technical dimensions.
3. At least two positive impacts and two threats.
4. One chain of effects.
5. One ethical trade-off involving AI-augmented testing or software quality.
6. One reasoned position: what should testers or developers do differently?

The Canvas Lab 4 instructions define the assessed submission. The activity above prepares the
analysis but does not add a separate deliverable.

## 11. Summary and Self-Check

Testing is a technical practice with societal consequences. Sustainability broadens quality
from "does it work?" to "what effects does it create over time, for whom, and at what cost?"
SusAF gives a structured way to reason about those effects. AI-augmented testing adds benefits
and risks, so testers need both technical evidence and ethical judgment.

Before finishing the course notes, check whether you can answer these questions:

1. Why is sustainability part of software quality?
2. What are the five SusAF dimensions?
3. How are Green IT and sustainable software different?
4. What is the difference between AI for sustainability and sustainability of AI?
5. How do direct, enabling, and systemic effects differ?
6. What boundary, baseline, and functional unit would make a resource claim interpretable?
7. Why can an automated accessibility score not establish accessibility by itself?
8. How can testing support equity and inclusion?
9. What ethical risks appear when using AI tools for testing?
10. What should a Lab 4 SusAF analysis contain?

## References and Further Reading

[R1] World Commission on Environment and Development. *Our Common Future*. United Nations,
1987. https://sustainabledevelopment.un.org/content/documents/5987our-common-future.pdf

[R2] San Murugesan. "Harnessing Green IT: Principles and Practices." *IT Professional*, 10(1),
2008. https://doi.org/10.1109/MITP.2008.10

[R3] Patricia Lago, Sedef Akinli Kocak, Ivica Crnkovic, and Birgit Penzenstadler. "Framing
Sustainability as a Property of Software Quality." *Communications of the ACM*, 58(10), 2015.
https://doi.org/10.1145/2714560

[R4] Aimee van Wynsberghe. "Sustainable AI: AI for sustainability and the sustainability of AI."
*AI and Ethics*, 1, 2021. https://doi.org/10.1007/s43681-021-00043-6

[R5] Christoph Becker et al. "Sustainability Design and Software: The Karlskrona Manifesto."
*Proceedings of the 37th IEEE/ACM International Conference on Software Engineering*, 2015.
https://doi.org/10.1109/ICSE.2015.179

[R6] Leticia Duboc, Birgit Penzenstadler, Jari Porras, Sedef Akinli Kocak, Stefanie Betz,
Ruzanna Chitchyan, Ola Leifler, Norbert Seyff, and Colin C. Venters. "Requirements Engineering
for Sustainability: An Awareness Framework for Designing Software Systems for a Better
Tomorrow." *Requirements Engineering*, 25(4), 2020.
https://doi.org/10.1007/s00766-020-00336-y

[R7] Lorenz M. Hilty and Bernard Aebischer, editors. *ICT Innovations for Sustainability*.
Springer, 2015.

[R8] W3C. *Web Content Accessibility Guidelines (WCAG) 2.2*. W3C Recommendation, 2024.
https://www.w3.org/TR/WCAG22/

[R9] ISO/IEC 25010:2023. *Systems and software engineering: Systems and software Quality
Requirements and Evaluation (SQuaRE): Product quality model*.
https://www.iso.org/standard/78176.html
