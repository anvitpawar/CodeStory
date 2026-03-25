# CodeStory — Executive Demo Script
## For Managing Director · Story-Led Walkthrough
**Format:** Live demo · 15–20 minutes · No ask · Let the tool speak
**Tone:** Personal, grounded, confident, curious

---

## Presenter Notes

This is not a sales pitch. It is a demonstration of something that already works, built to solve a real problem that was personally experienced. Let the tool do the talking. Your job is to give the MD the context to understand what they are seeing and why it matters.

Have the CodeStory UI open, logged in. Have a completed document ready to show alongside the live run. Know the IBB system well enough to talk about it naturally — the MD will feel the difference between someone narrating a demo and someone showing them something they built.

---

## Part 1 — The Story (3 minutes)

*Step away from the screen. No clicks. Just talk.*

"I want to start with a moment most engineers will recognise.

You join a project. You are handed access to a repository. Maybe it is a system that has been running for two years, maybe five. There are DAGs, feature stores, ingestion pipelines, cloud functions — all interconnected, all doing something important. And there is no documentation. Not outdated documentation. No documentation.

You ask the team. Everyone knows their piece. Nobody knows the whole picture. The person who designed the original architecture left eight months ago.

So you read. You trace. You draw diagrams on whiteboards, erase them, draw them again. Three weeks later you understand enough to contribute. Three weeks of a senior engineer's time — not building, not solving problems, just catching up.

That was my experience when I joined the IBB project. And it is not an IBB problem. It is the default state of every complex system in every engineering organisation. Knowledge lives in code. Code is not readable by most of the people who need to understand it. And the gap between what the system does and what anyone can explain about it grows every day.

That gap has a cost. It costs onboarding time. It costs production support. It costs modernisation — because you cannot safely re-platform something you do not fully understand. And it costs people — because the engineers who carry this knowledge in their heads become single points of failure.

CodeStory is what I built to close that gap."

---

## Part 2 — The Market Context (2 minutes)

*Stay off screen. This is important framing.*

"Before I show you the tool, it is worth understanding why this has not been solved before — and why it can be solved now.

The problem is not new. Teams have tried to solve it with wikis, with architecture review boards, with documentation sprints, with knowledge-sharing sessions. None of it sticks. Documentation written by hand goes stale the moment someone commits a change. People do not maintain wikis when there is a deadline. The incentive to document and the incentive to ship are in direct conflict.

Other tools have approached this from the outside in. GitDiagram reads your file tree and draws a surface-level diagram — but it does not read the code. GitHub Copilot reads the file you have open in your editor — but it does not understand the system. Swimm lets you link documentation to code — but a human still has to write it. Sourcegraph helps you search and navigate — but it does not generate understanding.

What none of them do is read the entire codebase, understand what it does, and produce a complete, structured, human-readable document automatically.

Three things changed in the last 18 to 24 months that make this possible now.

Language models can now hold an entire focused codebase in a single context window — 100,000 to 200,000 tokens. They can reason about structure and intent, not just syntax. And the cost of a single generation run is now in the range of cents to low dollars — economically trivial compared to the engineering time it replaces.

The window to build this well is open right now. CodeStory is built for it."

---

## Part 3 — The Live Demo (8 minutes)

### Login

*Click into the browser.*

"The first thing you notice is that there is no new account to create. You log in with your existing corporate credentials — SSO, the same identity you use for everything else.

Your GitHub permissions carry over automatically. If you cannot read a repository, you cannot generate its documentation. Access control is not a feature — it is the foundation."

---

### The Connections

*Show the settings or connection status.*

"Behind this tool, three connections are live.

GitHub Enterprise — CodeStory can reach any repository the user has access to, using secure short-lived tokens that rotate automatically. No permanent credentials, no shared secrets.

Confluence — every document generated here can be published to the right space in one click. No copy-paste, no formatting work.

And the AI model — running privately, inside the network perimeter. No code leaves the building. This is not a call to an external API. The intelligence runs on infrastructure the organisation controls.

These three connections transform this from a prototype into something you can trust with production codebases."

---

### Starting a Run

*Click New Project. Fill in the form naturally.*

"A developer — let's say someone who joined IBB this week — comes here.

They paste the repository link. They choose the branch. Then they answer three questions that a human can answer without reading a line of code: what is the application called, who owns it, and in one or two sentences, what does it do.

That description field is the most important thing in the form. When you tell the tool what the system is supposed to do, the output changes significantly. It stops pattern-matching and starts interpreting. The difference shows in the document."

*Select the project type chips.*

"They select what kind of system this is — in IBB's case it is a combination of orchestrated DAGs, ingestion pipelines, and Python APIs. Multi-select. The tool adjusts what it pays attention to based on what you tell it."

---

### What Gets Ignored

*Show the ignore patterns briefly.*

"Before a single file reaches the AI model, the tool runs a filter.

Test files, build artefacts, lock files, compiled assets — gone. More importantly, any file that looks like it contains a password, a connection string, a certificate, or an API key — scrubbed. Replaced with a redaction marker. Logged in the audit trail.

The model sees architecture. It never sees credentials. This is not optional and it cannot be bypassed by any user."

---

### The Generation Run

*Click Run. Show the progress screen.*

"Now watch what happens in the next 50 seconds.

The tool fetches the IBB repository — over two hundred files. Filters down to the forty or so that carry real architectural signal. Scrubs those files for sensitive patterns. Counts the tokens to stay within safe limits. Assembles a structured prompt tailored to the type of system we described. Sends one call to the model.

And then — it writes."

*Point to the streaming preview.*

"This is live. You are watching the document being written in real time. Architecture diagram first. Then the data flow. Then the source tables. Then error handling. Each section drawn from the actual code."

---

### The Output

*Navigate to the completed IBB document.*

"This is what came out for IBB.

Fifty-plus pages. End-to-end technical documentation. Generated in under a few hours. The equivalent document, done manually by a senior engineer who knew the system well, would have taken weeks — and that assumes it got prioritised at all.

Let me walk through what is here."

*Go section by section, slowly. One sentence each.*

**Architecture diagram** — "Every component in the IBB system. Every connection between them. Generated from the code — not drawn in Visio, not described from memory."

**Data flow** — "Step by step, what happens from trigger to output. Which DAG fires at 2am. Which feature store it reads from. What transformation it applies. Where the result lands in BigQuery and Bigtable."

**Feature store schemas** — "Every source table. Every column. Data types, what the field represents, how it feeds into the downstream computation. The kind of detail that normally lives in one senior engineer's head."

**DAG orchestration** — "The task dependency tree. What runs in parallel, what waits for what, what the retry logic is, what fires a PagerDuty alert."

**Error handling** — "What breaks, where, and what the system does about it. Extracted from the code — not inferred, not guessed."

*Pause.*

"And then there is this."

*Show or describe the conversational Q&A on the document.*

"Once this document exists, it becomes a context you can talk to.

A developer on production support at 2am can ask — in plain English — 'what happens if the feature freshness check fails?' They get a precise answer in seconds. Without reading fifty pages. Without waking up the engineer who built it. Without raising a ticket.

A new joiner can ask 'which DAG writes to the customer summary table?' and get the answer before their second coffee.

This is not a static document. It is queryable institutional memory."

---

### Confluence

*Show the published Confluence page.*

"One click. Published. Right space, right parent page, right format.

And the next time a significant change is merged — new feature store, new DAG task, changed schema — the developer runs CodeStory again. The document updates. The version history shows exactly what changed between runs, in plain English.

The documentation does not need to be maintained. It regenerates."

---

### The Audit Trail

*Show briefly.*

"Every run is logged. Who ran it, which repository, which commit, which files were included, how many secrets were redacted, which model version was used. Immutable. Forwarded to the SIEM. Queryable.

This is enterprise-grade from day one — not something added later when compliance asks for it."

---

## Part 4 — Impact Observed (2 minutes)

*Step back from the screen.*

"The IBB implementation delivered four things we can measure.

A single source of truth for IBB's technical flows — for the first time, the full picture exists in one place, accessible to anyone on the team.

Significantly reduced time to understand feature-level dependencies — what used to require a meeting with three engineers now requires reading one section of a document.

Improved production support and impact analysis — when something breaks, the team can trace the flow quickly because the flow is documented accurately.

And faster onboarding without senior-engineer dependency — new joiners can get to contribution-ready in days, not weeks.

These are not projections. These are outcomes from the IBB proof point."

---

## Part 5 — Why This Extends Beyond IBB (2 minutes)

*Stay off screen. This is the vision.*

"IBB was the first use case. It will not be the last.

The capability is not tied to any specific technology, framework, or domain. The same tool that documented IBB's DAG orchestration will document a Java microservices platform, a React frontend, a Spark ingestion pipeline, or a legacy system that has been running since before anyone on the current team joined.

Four categories of system benefit from this most:

**Legacy platform unravelling** — complex, long-standing systems where the documentation is outdated, missing, or was never written. The systems that are hardest to touch because nobody is sure what they affect.

**Modernisation readiness** — before you re-platform, migrate to cloud, or refactor, you need to understand current-state behaviour with confidence. CodeStory produces that understanding from the code itself, not from interviews.

**Ingestion and pipeline use cases** — shared pipelines, data products, complex ingestion frameworks where the data lineage crosses multiple systems and teams.

**Scalable knowledge capture** — documentation that evolves with the system rather than becoming obsolete the moment it is written.

IBB is the reference implementation. The pattern it demonstrated applies to every platform with similar complexity."

---

## Part 6 — Where This Goes (1 minute)

*Brief. Confident. Forward-looking.*

"What exists today is document generation. It already works.

The natural next step is conversational access — every generated document becomes something you can ask questions of in plain language. Support engineers, new joiners, and architects all getting answers grounded in the actual code without reading through it.

After that, cross-system lineage — understanding how data flows not just within one system but across many, tracing a field from its source through every transformation to its final destination across repositories and teams.

And eventually, compliance-grade output — the kind of data lineage documentation that regulators ask for, generated from code rather than from manual attestation.

Each of these is a natural extension of the same core capability. The more diverse the systems this runs against, the more robust and reusable it becomes."

---

## Anticipated Questions

**"How do we know the output is accurate?"**
The IBB document was reviewed by engineers who know the system. Accuracy on explicitly declared logic — schemas, schedules, task dependencies — is high. Where the tool cannot determine something from the code, it says so explicitly rather than guessing. The output is honest about its gaps.

**"What about proprietary code or sensitive IP?"**
Nothing leaves the network. The model runs privately on infrastructure the organisation controls. Credentials are scrubbed before any file reaches the model. The code is processed in memory and never stored. What persists is the generated document — not the source.

**"What happens when the codebase changes?"**
Regenerate. One run, one click. The document reflects the new code. Version history captures what changed between runs in plain English — which is more useful than a git diff for anyone who is not a developer.

**"How hard is this to maintain?"**
The prompt engine is the intellectual property of the tool — the intelligence that knows what to look for in different types of systems. Adding support for a new framework is a configuration change, not an engineering sprint. The infrastructure around it is straightforward.

**"Who else is doing this?"**
GitDiagram reads file trees, not code. Swimm requires humans to write the documentation it links. Sourcegraph is a navigation tool. GitHub Copilot is a code assistant. No one in the market today reads the full codebase, understands the system, and produces a complete structured document automatically. That is the gap CodeStory fills.

---

## Closing

*After questions. Final words before leaving the room.*

"IBB went from no documentation to a fifty-plus page, queryable, published architecture document in under a few hours.

That is not a one-off. That is repeatable. That is the baseline.

The question is not whether this capability is useful. The question is how many systems in this organisation are running right now with no one who can fully explain them — and what that costs every time someone new joins, every time something breaks, and every time someone needs to change it."

*Stop there. Do not add anything after this line.*

---

*End of script.*
