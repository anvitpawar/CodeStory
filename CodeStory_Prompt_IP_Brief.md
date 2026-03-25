# CodeStory — Prompt Engine IP Brief
## How the on-the-run prompt is built from user inputs and file scanning

---

## The Core Idea

The prompt is never pre-written. It is assembled fresh for every single run from four sources: a fixed base, a detected project type, what the user typed, and what the files contain. Each source is a separate, independently tunable layer. The intelligence is in how the layers are combined — not in any single hardcoded string.

---

## Step 1 — File Scanning (before the prompt is built)

Before a single word of the prompt is written, the ingestion module scans the repository and produces two outputs that directly shape the prompt:

**The file list** — after filtering, the ranked list of files that made it through. The order matters: highest-priority files appear first in the context window because models pay more attention to content near the start of a long input.

**The type signal** — from the file list alone (not the content), a cheap pre-pass call asks the model:

```
Look at this file tree. Return JSON only:
{
  "project_types": [],      // from known types
  "primary_language": "",
  "has_dags": true/false,
  "has_sql": true/false,
  "has_api_layer": true/false,
  "key_files": [],          // top 10 most important file paths
  "confidence": 0-100
}
```

This call costs almost nothing — it sends only filenames, not content. It returns in 2–3 seconds. The result drives which type blocks are included in the main prompt.

If confidence is above 80 the detection is used silently. Between 50 and 80 the user is shown what was detected and asked to confirm. Below 50 the user picks from a dropdown.

---

## Step 2 — The Four Layers

### Layer 1 — Base (always sent, never changes)

This is the contract the model must honour on every single run:

```
You are a senior engineer and technical writer.
Read the source code below and produce a precise architecture document
a new developer can use to immediately understand and work on this system.

RULES — non-negotiable:
- Only extract values you can see directly in the code
- If something is unclear write: "Not determinable from source"
- Never invent table names, column names, schedule strings, or config values
- Every diagram must be ASCII art inside a triple-backtick code block
- ASCII diagrams must be max 70 characters wide
- Use only: ─ │ ┌ ┐ └ ┘ ├ ┤ ┬ ┴ for borders · ▼ → for arrows

OUTPUT — always produce these sections in this exact order:
## 1. Executive Summary
## 2. Architecture Diagram
## 3. Data Flow (numbered steps)
## 4. Source Inventory
## 5. Target Inventory
## 6. Core Logic / Transforms
## 7. Infrastructure & Services
## 8. Error Handling & Failure Modes
```

This layer is ~300 tokens. It never changes per run. It is the contract.

---

### Layer 2 — Type Block (swaps per detected project type)

Selected from a dict by app type key. Multiple types = multiple blocks concatenated. Each block is a targeted list of what to look for:

```python
TYPE_BLOCKS = {

  "sql_only": """
FOCUS — SQL PIPELINES:
Find every: CREATE TABLE · INSERT INTO · SELECT FROM · WITH (CTE) clause
Map each source table to each target table explicitly
Extract every column with its transform: CAST · COALESCE · CASE WHEN · JOIN condition
Note data types, nullability, filter conditions applied
If no orchestration exists, skip schedule and DAG sections entirely
""",

  "gcp_dags": """
FOCUS — GCP DAG PIPELINES:
Find every DAG definition: schedule_interval · start_date · catchup · max_active_runs
Map task dependencies from >> operators or depends_on_past
Extract operator types: BigQueryOperator · DataflowHook · GCSOperator · PythonOperator
Pull on_failure_callback · retries · retry_delay · SLA values
Note BQ dataset · table · write_disposition · partition config
""",

  "ingestion": """
FOCUS — INGESTION PIPELINES:
Find source connectors: JDBC · GCS · Kafka · Pub/Sub · SFTP · API
Trace .read() · .readStream() · spark.read.format() calls
Extract every .withColumn() · .cast() · .filter() · .drop() transform
Map write destinations: .write() · .writeStream() · output sinks
Note partitionBy() · repartition() strategies
Flag schema mismatches or type casting risks explicitly
""",

  "java_spring": """
FOCUS — JAVA BACKEND:
Find every @RestController and map: method · path · request body · response type
Trace: Controller → Service → Repository → DB chain
Extract @Entity classes: table name · columns · relationships
Find external calls: RestTemplate · FeignClient · WebClient
Extract @ControllerAdvice error handlers and HTTP status codes
Note application.yml config keys
""",

  "python_api": """
FOCUS — PYTHON API:
Find every route: @app.route · @router.get/post · @app.get
Extract Pydantic models or dataclasses as request/response schemas
Find SQLAlchemy models: table · columns · relationships
Note every external HTTP call made
Extract HTTPException raises and status codes
Find config loaded from environment variables or .env
""",

  "angular_react": """
FOCUS — FRONTEND:
Find every HTTP call and which component makes it — note endpoint URL
Extract routing config: URL paths to components
Find state management: NgRx · BehaviourSubject · Redux · React Context
Map component parent-child hierarchy from imports
Note environment.ts API base URLs per environment
""",
}
```

The type block is the **IP core** — it is what makes the model look at the right signals for each application type. A generic prompt produces a generic document. A typed prompt produces a precise one.

---

### Layer 3 — Focus Block (built from user input)

Built entirely from what the user typed in the "anything to focus on?" field:

```python
def build_focus_block(hint: str) -> str:
    if not hint or not hint.strip():
        return "\nCover all standard sections with equal depth.\n"
    return f"""
SPECIAL FOCUS — treat this as the highest priority instruction after format rules:
{hint.strip()}

Allocate extra depth and detail to whatever this instruction points at.
"""
```

If the user typed: *"focus on the FX rate join logic and the casting layers — these are where failures happen"* — that exact text becomes a bolded instruction the model treats as its primary task after format compliance. This is what makes the output feel like it was written by someone who understood the specific concern, not a generic scanner.

---

### Layer 4 — Context Block (from user form fields)

```python
def build_context_block(app_name, team, description, repo_url, branch) -> str:
    return f"""
APPLICATION DETAILS:
- Name        : {app_name}
- Team        : {team}
- Description : {description}
- Repo        : {repo_url}   branch: {branch}

The description above is the stated intent of this system.
Use it to interpret ambiguous code — when variable names are unclear,
the description tells you what the system is trying to do.

Analyse all files below and begin the document now.
"""
```

The `description` field is the most important user input in the entire tool. It is the anchor. When a file is called `processor.py` with a class called `DataHandler` and a method called `run()`, the description is what tells the model whether this is a payment processor, a feature aggregator, or a report generator. Without it, the model guesses. With it, the model interprets.

---

## Step 3 — Final Assembly

```python
def assemble_prompt(
    app_types: list[str],
    hint: str,
    app_name: str,
    team: str,
    description: str,
    repo_url: str,
    branch: str,
    file_context: FileContext
) -> dict:

    # Layer 2: concatenate all relevant type blocks
    type_block = "\n\n".join(
        TYPE_BLOCKS[t] for t in app_types if t in TYPE_BLOCKS
    )

    system_message = (
        LAYER_1_BASE        # fixed contract
        + type_block        # what to look for
        + build_focus_block(hint)  # user's priority
    )

    user_message = (
        build_context_block(app_name, team, description, repo_url, branch)
        + "\n\n"
        + file_context.files_text  # ### FILE: path\n[content] x N files
    )

    return {
        "system": system_message,
        "user":   user_message,
        "model":  "tachyon-large",
        "max_tokens": 8192,
        "temperature": 0.1,  # low — we want precise extraction, not creativity
        "stream": True
    }
```

Temperature 0.1 is deliberate. You do not want a creative response. You want a precise extraction. Low temperature means the model commits to what it finds rather than generating plausible-sounding alternatives.

---

## How to Tune and Test the Prompt

The process for improving output quality is surgical, not wholesale:

**Identify which layer caused the problem:**
- Wrong table name or invented value → Layer 1 hallucination guard needs strengthening
- Missing section → Layer 1 output format contract needs an explicit requirement added
- DAG schedule not found → Layer 2 `gcp_dags` block needs the signal added
- User's specific concern was ignored → Layer 3 injection needs to be more prominent
- Wrong app interpretation → Layer 4 description field needs to anchor harder

**Fix only that layer. Re-run all test cases. All must pass before committing.**

Each type block should have at least two real reference repositories as regression tests. A change to the DAG block that improves one repo must not degrade the other. This is what makes the prompt engine maintainable as a product over time.

---

## Why This Is the IP

The technology — calling an LLM with a prompt — is not proprietary. What is proprietary is:

1. **The type block library** — accumulated through validation against real codebases of many types, tuned to extract the right signals for each. This takes time and real codebases to build well. It cannot be replicated quickly from first principles.

2. **The 4-layer assembly architecture** — the separation of concerns between the base contract, the type-specific logic, the user focus, and the file context. This is what makes the output improvable without introducing regressions.

3. **The hallucination guard design** — the specific combination of output format enforcement, "not determinable" fallback instruction, and low temperature that produces reliable, citable output rather than confident-sounding invented values.

4. **The file priority ranking** — knowing that DAG files should precede SQL files should precede config is not obvious. It is the result of learning which files carry the most architectural signal per token of context spent on them.

These four things, combined and tested against real codebases, are what make the output trustworthy enough for an engineer who did not write the code to rely on.

---

*End of prompt IP brief.*
