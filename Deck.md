# CodeStory
### An AI tool that reads any codebase and tells you exactly what it does
#### Pitch for Managing Directors

---

## The Problem Nobody Wants to Talk About

Every engineering organisation has the same silent crisis.

A new developer joins a team. They are handed access to a repository. The codebase is thousands of files — services, pipelines, queries, configurations, infrastructure definitions. There is no documentation. Or worse, there is documentation written two years ago that bears no resemblance to what the code actually does today.

They spend the first two to three weeks not contributing. They spend it reading. Asking questions. Drawing diagrams on whiteboards that they erase and redraw. Slowly building a mental model that an experienced team member carries in their head but has never written down.

This is not a people problem. It is a systems problem. Documentation is treated as a task that comes after the work. It never gets done. And when someone leaves, they take the architecture with them.

**I built CodeStory because I was that new developer.** I joined a project mid-flight, inherited a complex multi-system codebase with no documentation, and spent weeks just trying to understand what the system did before I could contribute anything of value. There had to be a better way.

---

## What CodeStory Does

CodeStory is a lightweight service that reads any code repository and automatically generates a complete, structured architecture document — in under 60 seconds.

You give it a repository link or a zip file. You tell it what the application does in one or two sentences. You click Run.

It reads every relevant file. It understands the codebase — regardless of language, framework, or domain. It generates a full technical document covering architecture, data flows, component inventories, error handling, dependencies, and infrastructure configurations. It publishes that document wherever your team keeps documentation.

**No templates to fill in. No diagrams to draw. No meetings to extract tribal knowledge. Just the document, generated from the code itself.**

---

## Why This Is Different From What Exists Today

Existing tools do one of two things badly.

AI coding assistants read the files you have open in your editor. They are reactive and local — they help you write the next line of code, they do not help you understand a system. They see the tree, not the forest.

Documentation tools are empty canvases. They require humans to fill them in. They go stale the moment someone commits a change.

CodeStory is neither of these. It reads the entire repository in one pass, understands the intent of the system, and generates documentation that reflects what the code actually does — not what someone thought it would do when the project started.

The key innovation is the **prompt engine**. A dynamic, layered prompt is assembled at runtime, tailored to the specific characteristics of the repository being analysed. Each layer is independently tunable. The output is consistent, structured, and grounded entirely in the actual code.

---

## Application Agnostic by Design

CodeStory makes no assumptions about what kind of application it is reading. It works across the full spectrum of software systems:

- Data transformation pipelines and SQL logic
- Workflow orchestration and scheduled jobs
- Streaming and batch data ingestion systems
- Backend API services in any language or framework
- Frontend applications and single-page apps
- Infrastructure-as-code and cloud configurations
- Mixed repositories containing multiple application types

The service detects what it is looking at — from the file structure, imports, and code patterns — and applies the appropriate analysis lens automatically. A repository of Java microservices gets the same quality of output as a repository of Python data pipelines or a monorepo containing both.

This is not a tool for a specific technology team. It is a tool for any team that writes software and needs to understand it.

---

## The Technical Approach

### How it works in six steps

**1. Ingest** — The service fetches the repository via API or accepts a zip upload. It applies an intelligent file filter, discarding everything that carries no architectural signal: build artefacts, test files, lock files, binary assets, and any files that may contain credentials or secrets. A typical repository of three hundred files is reduced to forty to seventy high-signal files.

**2. Prioritise** — The remaining files are ranked by their likely importance to understanding the architecture. Entry points, orchestration definitions, and schema files come first. Configuration and documentation come last. The most important files are given the most prominent position in the context window sent to the model.

**3. Secure** — Before any code reaches the language model, a pattern-matching scrubber redacts passwords, connection strings, API keys, and private key material. This runs on every file, every time, before any other processing.

**4. Prompt** — A dynamic prompt is assembled from four layers: a base layer with output format rules and quality constraints, a type-aware layer with targeted extraction instructions, a focus layer from the user's optional hint, and a context layer with the application name and description. The assembled prompt and the filtered files are sent to the language model in a single streaming call.

**5. Generate** — The language model produces a structured document with eight standard sections. The output streams back to the user in real time so they can see the document being written section by section, rather than waiting for a monolithic response.

**6. Publish** — The generated markdown is converted to the target documentation format and published. The raw markdown is stored with full version history so every regeneration creates a diff-able record of how the architecture evolved over time.

### What the output contains

Every generated document follows the same eight-section structure, with content that adapts to what the code actually does:

1. **Executive summary** — what the system does in plain English, for any audience
2. **Architecture diagram** — a clear representation of components and their relationships
3. **System flow** — the step-by-step journey through the system, from trigger to output
4. **Source inventory** — everything the system reads from: databases, APIs, queues, files
5. **Target inventory** — everything the system writes to: databases, services, storage
6. **Core logic** — transformation rules, business logic, key decisions embedded in code
7. **Infrastructure** — services, configurations, and external dependencies
8. **Error handling** — failure modes, retry logic, alert mechanisms extracted from code

---

## Why It Is Always Accurate

The document is generated from the code. Not from interviews. Not from memory. Not from a wiki page someone edited eighteen months ago.

When the code changes, you run CodeStory again. The new document reflects the new reality. You are not maintaining a wiki — you are regenerating an artefact from source truth.

Every regeneration is versioned. The system stores a plain-English summary of what changed between versions, derived from comparing the two outputs: which components were added, which configurations changed, which data flows were modified. This creates a living changelog of how the system evolved, derived entirely from the code rather than from commit messages or human recall.

For teams that use continuous integration pipelines, CodeStory can be triggered automatically on every significant code change. The documentation is always one commit behind the code, which is as accurate as documentation can realistically ever be.

---

## Why It Is Maintainable and Scalable

### As a tool

The core engine is model-agnostic. It works with any language model that supports a sufficiently large context window. As models improve — longer context, better code understanding, cheaper inference — CodeStory improves without architectural changes.

Support for new application types, frameworks, or languages is added by writing a new analysis block: a targeted set of extraction instructions that tell the model what signals matter for that technology. This is a configuration change, not an engineering effort. A small team can extend coverage to a new framework in a day.

The prompt engine is also independently testable. Each layer can be validated against a suite of real codebases before being deployed. Changes to one layer do not affect others. This is what makes the output quality improvable over time without introducing regressions.

### As a product

Every engineering organisation in the world has this problem. The total addressable market is not a niche — it is every team that writes software and needs other people to understand it, which is every team that writes software.

The natural expansion path is clear:

**Phase 1 — Document generation.** What CodeStory does today. Read a repository, generate a structured document, publish it.

**Phase 2 — Conversational access.** A question-answering layer over the generated documents. Developers ask "where does this field come from?" or "what happens when this service fails?" and get an answer grounded in the actual code, without reading files.

**Phase 3 — Cross-repository lineage.** Understanding how System A's output becomes System B's input across an entire organisation's codebase. Full end-to-end data and call-chain tracing across teams and repositories.

**Phase 4 — Compliance and audit mode.** Generating audit-ready documentation — data lineage reports, access maps, transformation inventories — directly from code rather than from manual interviews. This addresses a significant compliance overhead in regulated industries.

Each phase builds on the same core: a reliable, current, accurate understanding of what code does.

---

## Why Now

Three things have converged to make this possible today in a way it was not two years ago.

**Context windows.** Modern language models can process one hundred thousand to two hundred thousand tokens in a single call. The relevant files from a focused codebase fit comfortably within this limit. Complex retrieval and chunking strategies are not required for the common case, which means the architecture stays simple and the output quality stays high.

**Code understanding.** The current generation of language models has been trained on vast amounts of code across every major language and framework. They can reason about structure, intent, and relationships at a level of reliability that makes production use viable.

**Inference cost.** The cost per document generation is now in the range of cents to low dollars. At that price point, generating documentation automatically on every significant code change is economically trivial compared to the engineering hours it replaces.

The window to build this well is open now. The underlying capabilities exist. The infrastructure is mature. The problem has never gone away.

---

## The Ask

CodeStory can be built by a small team, deployed internally, and delivering measurable value within weeks. It does not require large infrastructure investment, complex organisational change, or lengthy procurement cycles to prove its worth.

The work that matters is getting the prompt engine right — validating it against real codebases of different types, tuning the extraction logic, establishing that the output is accurate enough to be trusted by engineers who did not write the code. That validation work is the hard part. Everything around it is straightforward.

The measurable outcomes are clear: reduction in onboarding time for new engineers, reduction in time spent answering "how does this work" questions, reduction in undocumented systems across the codebase, and the creation of a documentation baseline that compliance and audit processes can rely on.

**The opportunity is to make every codebase in the organisation as understandable on day one as it is after three months. That is worth building.**

---

## Summary

| | |
|---|---|
| **Problem** | Codebases are undocumented. New engineers lose weeks to understanding. Documentation goes stale and is abandoned. |
| **Solution** | Automated architecture documentation generated directly from source code — always current, versioned, published. |
| **Key differentiator** | Application agnostic. Works across any language, framework, or domain without configuration. |
| **Approach** | Dynamic prompt engine · intelligent file filtering · streaming LLM generation · version history |
| **Why it stays accurate** | Generated from code, not written by hand. Regenerates when code changes. Diff-able version history. |
| **Why now** | Large context windows + reliable code understanding + low inference cost — all mature today. |
| **Expansion path** | Document generation → conversational Q&A → cross-repo lineage → compliance and audit reporting |
| **What it takes** | Small team, weeks to first value, prompt validation is the hard work — infrastructure is straightforward. |

---

*Built from a real problem. Designed for every team that writes software.*