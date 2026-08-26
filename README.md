# Software Testing Course Notes

Narrative course notes supporting **DV033G, Principles & Practices in Software Testing**, taught
by [Sergio Rico](https://www.miun.se) at Mid Sweden University. The current edition is HT26.

## Author and provenance

- **Author:** Sergio Rico (sergio.rico@miun.se), Senior Lecturer, Mid Sweden University.
- **Derivation:** this content is derived from the lecture decks, delivery plans, and lab
  material developed for this course across several editions (HT24, HT25, HT26), and from the
  cited literature referenced within each chapter. It is maintained as a standalone, versioned
  companion to the internal course workspace so it can evolve independently as the course
  develops.
- **License:** unless noted otherwise, text is © Sergio Rico. Contact the author before reusing
  chapters outside this course. Cited third-party material (papers, standards, books) is not
  relicensed by inclusion here; follow the citation to the original source.

The notes are not slide transcripts. Each module note connects lecture content, activities, and
lab/project work into one readable study document, and each chapter records its own draft
version and date in its opening line.

## Building the website

This repository is a small [MkDocs](https://www.mkdocs.org/) site (using the
[Material theme](https://squidfunk.github.io/mkdocs-material/)) built from the Markdown chapters
in this folder.

```bash
pip install -r requirements.txt
mkdocs serve -f mkdocs/mkdocs.yml   # local preview at http://127.0.0.1:8000
mkdocs build -f mkdocs/mkdocs.yml   # static site in ./site
```

Pushes to `main` are published automatically to GitHub Pages by
[`.github/workflows/deploy-docs.yml`](.github/workflows/deploy-docs.yml).

## Version control

Changes are tracked with Git and tagged releases (e.g. `v0.5`) as chapters reach a stable draft
state. See the per-chapter draft version noted at the top of each file, and the production
status table below, for the current state of each module.

## Production status

| Module | Topic | Source lectures | Status |
|---|---|---|---|
| M1 | Quality Assurance & Fundamentals of Testing | L1, L2 | Draft v0.5: aligned, final copyedit due |
| M2 | Specification-based & Unit Testing | L3, L4 | Draft v0.2: aligned, final copyedit due |
| M3 | Structural Testing & Coverage | L5 | Draft v0.2: aligned, final copyedit due |
| M4 | Test Optimization | L6, L7 | Draft v0.2: aligned, final copyedit due |
| M5 | Research Trends & Testing AI | L9 | Draft v0.2: aligned, time-sensitive sources to recheck |
| M6 | Ethical & Societal Aspects | L8 | Draft v0.2: aligned, final copyedit due |

## Chapter files

- [M1: Quality Assurance and Fundamentals of Software Testing](M1-Quality-Assurance-Fundamentals.md)
- [M2: Specification-Based and Unit Testing](M2-Specification-Based-and-Unit-Testing.md)
- [M3: Structural Testing and Coverage](M3-Structural-Testing-and-Coverage.md)
- [M4: Test Optimization, Mutation, Regression, and CI](M4-Test-Optimization-Mutation-Regression-CI.md)
- [M5: Research Trends and Testing AI](M5-Research-Trends-and-Testing-AI.md)
- [M6: Ethical and Societal Aspects of Software Testing](M6-Ethical-and-Societal-Aspects.md)
