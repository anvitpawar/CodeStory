# AIDE — AI Integrated Document Generator
## Complete Project Context & Conversation Summary
> Feed this document to any LLM or chatbot to continue discussions on specific topics.
> Last updated: March 2026 | Author: Customer Intelligence Data Engineering, Compant Name

---

## 0. How to Use This Document

This document captures the full context of the AIDE project as designed across an extended brainstorming session. Each section is self-contained. To continue a specific discussion, paste this entire document into a new chat and say:

> "I'm continuing Discussion D[N] — [title]. Here is the full project context. Pick up from where D[N] was left off."

The discussion backlog at the end lists what has been covered and what is pending for each topic.

---

## 1. What is AIDE?

**AIDE** (AI Integrated Document Generator) is an internal Compant Name tool that:

1. Reads any GitHub Enterprise (GHE) repository or zip file
2. Filters and understands the source code intelligently
3. Generates a complete, structured architecture document automatically
4. Publishes it directly to Confluence

**Tagline:** *Point. Read. Document.*

**Primary users:** Data engineers, platform engineers, and developers at Compant Name who need to document legacy or new pipelines without manual effort.

**Business problem solved:** companyname has thousands of undocumented pipelines. New engineers spend 2–3 weeks understanding a single pipeline before contributing. Compliance teams spend days tracing data lineage manually. AIDE eliminates both problems by treating documentation as a generated artefact, not a manual task.

---

## 2. Target Project Types (All Supported)

| # | Type | Key Signals | Primary Files |
|---|------|-------------|---------------|
| 1 | SQL only | CREATE TABLE, INSERT INTO, CTEs | `.sql` |
| 2 | GCP DAGs / Airflow | `from airflow import DAG`, `schedule_interval`, BigQueryOperator | `.py` `.yaml` `.cfg` |
| 3 | Spark / Dataflow ingestion | SparkSession, `.read()`, `.write()`, `.withColumn()` | `.py` `.scala` |
| 4 | Java Spring Boot backend | `@RestController`, `@SpringBootApplication`, JpaRepository | `.java` `.yml` |
| 5 | Angular / React frontend | `@Component`, `HttpClient`, `useEffect`, axios | `.ts` `.tsx` `.html` |
| 6 | Python FastAPI / Flask | `@app.route`, `@router.get`, Pydantic models | `.py` |
| 7 | Mixed (multiple in one repo) | Multiple signals detected | Union of all filters |

---

## 3. The Example Project Used Throughout Discussions

**Project name:** Customer Summary Curation Pipeline
**Team:** Customer Intelligence — Data Engineering, Compant Name
**Repo:** `github.companyname.com/customer-intelligence/cust-summary-curation`
**Branch:** main

### What it does
Ingests data from 9 discrete customer feature stores, independently summarises each using a Tachyon LLM API call with a feature-specific prompt template, aggregates all 9 summaries into a unified customer profile, and serves it to relationship managers and customers via a FastAPI retrieval service on Cloud Run backed by Bigtable and BigQuery.

### Architecture summary
```
9 FEATURE STORES (BigQuery)        LLM LAYER              STORAGE & SERVE
+---------------------------+
| companyname_features               |   +--------------------+   +-----------+
|  .spend_features          |-->|  llm_summarise     |-->| Bigtable  |
|  .demographics            |-->|  (9x parallel)     |   | cust#{id} |
|  .risk_scores             |-->|  Tachyon API       |   +-----------+
|  .transactions_summary    |-->|  max_tokens: 512   |   +-----------+
|  .product_holdings        |-->|  temperature: 0.1  |-->| BigQuery  |
|  .digital_engagement      |-->+--------------------+   | cust_summ |
|  .complaints              |       merge_task            +-----------+
|  .relationship            |           |                      |
|  .credit_utilisation      |           v                +-----------+
+---------------------------+   +-----------+            | FastAPI   |
                                |  Merged   |            | GCR       |
Cloud Composer                  |  Profile  |            | /summary  |
cust_summary_dag                +-----------+            +-----------+
schedule: 0 2 * * *                                           |
                                                        +-----------+
                                                        | customer UI |
                                                        +-----------+
```

### 9 Feature Store Tables (source tables)

| Feature Store | BQ Table | Key Columns | LLM Prompt Template |
|---|---|---|---|
| Spend Features | companyname_features.spend_features | cust_id, avg_monthly_spend, top_cat, spend_30d, spend_90d, merchant_diversity_score, spend_trend | spend_summary_prompt |
| Demographics | companyname_features.demographics | cust_id, age_band, income_band, state_code, tenure_years, wealth_segment, employment_status | demo_summary_prompt |
| Risk Scores | companyname_features.risk_scores | cust_id, credit_score, risk_tier, default_probability_12m, churn_probability_90d, fraud_risk_score, collections_flag | risk_summary_prompt |
| Transactions Summary | companyname_features.transactions_summary | cust_id, txn_count_30d, txn_count_90d, avg_txn_amount, atm_withdrawal_pct, digital_txn_pct, declined_txn_count_30d | txn_summary_prompt |
| Product Holdings | companyname_features.product_holdings | cust_id, product_codes[], total_product_count, has_mortgage, has_investment, total_deposit_balance, cross_sell_score | prod_summary_prompt |
| Digital Engagement | companyname_features.digital_engagement | cust_id, login_count_30d, mobile_login_pct, primary_channel, feature_adoption_score, last_login_days_ago | digi_summary_prompt |
| Complaints | companyname_features.complaints | cust_id, open_complaint_count, total_complaint_count_12m, escalated_complaint_flag, nps_score, satisfaction_trend | comp_summary_prompt |
| Relationship Depth | companyname_features.relationship | cust_id, rm_assigned, rm_employee_id, relationship_score, relationship_tier, next_best_action | rel_summary_prompt |
| Credit Utilisation | companyname_features.credit_util | cust_id, total_credit_limit, overall_utilisation_pct, utilisation_trend, min_payment_miss_count_12m, payment_behaviour | cred_summary_prompt |

### DAG structure
```
cust_summary_dag  [schedule: 0 2 * * * · catchup: False · max_active_runs: 1]

check_feature_freshness
        |
        v
[PARALLEL — all 9 run simultaneously]
extract_spend --> llm_spend
extract_demo  --> llm_demo
extract_risk  --> llm_risk
extract_txns  --> llm_txns
extract_prod  --> llm_prod
extract_digi  --> llm_digi
extract_comp  --> llm_comp
extract_rel   --> llm_rel
extract_cred  --> llm_cred
        |
        v (all 9 must complete)
merge_summaries
        |
   +----+----+
   v         v
write_bigtable  write_bq
        |
        v
notify_success (Slack: #cust-intel-alerts)
```

### Error handling (from code)
| Failure | Task | Action | Alert |
|---|---|---|---|
| Feature store stale >24h | check_feature_freshness | DAG fails fast | PagerDuty + Slack |
| LLM API timeout >30s | llm_{feature}_task | Retry x3, 300s delay | Slack on 3rd fail |
| LLM invalid JSON | llm_{feature}_task | Retry x2, use cached | Log warning |
| BQ write fail | write_bq_task | Retry x2 | PagerDuty |
| Bigtable write fail | write_bigtable_task | Not determinable | Not determinable |

---

## 4. The AIDE Service Architecture

### Tech stack
| Layer | Choice | Why |
|---|---|---|
| Frontend | React + Vite (HTML/JS for MVP) | Fast to build the 4-step UI |
| Backend | FastAPI (Python) | Best LLM tooling, async, streaming |
| LLM | Tachyon (companyname internal) | Internal endpoint, no code egress |
| Auth | GitHub Apps / OAuth → installation tokens | Service-level, not person-tied |
| Secret store | HashiCorp Vault | companyname standard |
| Confluence | REST API (Cloud or Data Center) | Page create/update |
| PDF export | Pandoc / WeasyPrint | Markdown → PDF server-side |

### The 5 internal modules
```
aide/
├── ingestion/
│   ├── fetcher.py        # GHE API zip download
│   ├── filter.py         # should_ignore(), priority_score()
│   ├── scrubber.py       # secret redaction
│   └── context_builder.py # token count, FileContext assembly
├── prompt/
│   ├── builder.py        # 4-layer prompt assembly
│   ├── type_blocks.py    # all 7 type-specific instruction blocks
│   └── detector.py       # optional pre-pass project type detection
├── llm/
│   └── tachyon_client.py # streaming call, retry, backoff
├── output/
│   ├── parser.py         # split output into named sections
│   └── validator.py      # check all 8 sections present
├── publisher/
│   └── confluence.py     # create/update page, code macro, PDF attach
└── main.py               # FastAPI app, 4 endpoints
```

### FastAPI endpoints
```
POST /generate           # main endpoint — ingest + prompt + LLM + return stream
POST /publish            # take generated markdown, publish to Confluence
GET  /projects           # list past projects + versions
GET  /health             # health check
```

---

## 5. The 4-Layer Dynamic Prompt System

The prompt is assembled at runtime from 4 independent layers. Each is small, testable, and tunable independently.

### Layer 1 — Base (always sent, ~300 tokens)
Role definition, output format rules, section order (8 sections always in the same order), hallucination guard ("only extract values you can see directly in the code"), ASCII diagram rules (max 70 chars wide, box-drawing character set), quality rules.

### Layer 2 — Type Block (~200 tokens, swaps per app_type)
Selected from a dict keyed on project type. Multiple types = multiple blocks concatenated.
- `sql_only` → find every SELECT/INSERT/CTE/CAST/JOIN
- `gcp_dags` → find DAG tasks, schedule_interval, BigQueryOperator, retries
- `ingestion` → find .read()/.write()/.withColumn(), schema StructType, connectors
- `java_spring` → find @RestController, @Entity, service chain, external calls
- `angular_react` → find HTTP calls, routing config, state management
- `python_api` → find @app.route, Pydantic models, SQLAlchemy models
- `mixed` → combine relevant blocks + add "how they connect" section

### Layer 3 — Focus Block (~100 tokens, from user form)
Built from the free-text "anything to focus on" field. Injected as `SPECIAL FOCUS:` instruction treated as highest priority after format rules.

### Layer 4 — Context Block (~50 tokens, user inputs)
```
APPLICATION DETAILS:
- Name        : {app_name}
- Team        : {team}
- Description : {user_description}  ← most important field
- Repo        : {repo_url}  branch: {branch}
```

### Final prompt structure
```python
system_message = LAYER_1_BASE + LAYER_2_TYPE_BLOCK(app_types) + LAYER_3_FOCUS(hint)
user_message   = LAYER_4_CONTEXT + "\n\n" + file_context.files_text
# files prefixed with: ### FILE: path/to/file.py
```

### Output — universal 8-section doc skeleton
```
## 1. Executive Summary
## 2. Architecture Diagram       (ASCII in code block)
## 3. Data Flow                  (numbered steps)
## 4. Source Tables              (markdown table)
## 5. Target Tables              (markdown table)
## 6. Column Lineage             (ASCII mapping)
## 7. GCP Services / Infra       (markdown table)
## 8. Error Handling             (table + decision tree)
```

---

## 6. File Filtering System

### Hardcoded default ignore list (always applied)
**Directories (any path component):**
`node_modules` `__pycache__` `.git` `.idea` `.vscode` `venv` `.venv` `target` `build` `dist` `out` `tests` `test` `__tests__` `spec` `coverage` `.pytest_cache` `.gradle` `vendor` `.m2` `site-packages` `htmlcov` `.mypy_cache`

**Extensions:**
`.pyc` `.pyo` `.class` `.jar` `.war` `.so` `.dylib` `.exe` `.dll` `.png` `.jpg` `.jpeg` `.gif` `.ico` `.svg` `.pdf` `.zip` `.tar` `.gz` `.bz2` `.whl` `.map` `.lock` `.sum`

**Exact filenames:**
`.env` `.env.local` `.env.prod` `.env.dev` `secrets.yaml` `credentials.json` `package-lock.json` `yarn.lock` `poetry.lock` `Pipfile.lock` `.DS_Store` `id_rsa` `id_ed25519`

**Security patterns (never overridable by user):**
`serviceaccount*.json` `*secret*` `*credential*` `*.pem` `*.key` `*.p12` `*.pfx` `*.env.*` `vault_token*`

### User-supplied patterns (UI textarea, merged at runtime)
Standard `.gitignore` syntax. Lines starting with `#` are comments. Directories end with `/`.

### Priority order per app type (files included first = higher in context window)
- `sql_dags`: DAG `.py` files → `.sql` → other `.py` → `.yaml`/`.yml` → `.json` → `.cfg` → `README`
- `ingestion`: main pipeline `.py` → schema defs → `.scala` → `.yaml` → config → `README`
- `java_spring`: controllers → services → repos → `.java` other → `.yml` → `.properties`
- `angular`: services → routing → components → `.ts` → `.html`
- `python_api`: `main.py` → routers → models → config → `README`

### Token budget
- Hard cap: **80,000 tokens** input (adjust if Tachyon context window > 128k)
- Token counter: `tiktoken` with `cl100k_base` encoding
- Add 10% safety margin if Tachyon tokeniser is unknown: use 72,000
- If over budget: drop lowest-priority files first (truncated flag set in FileContext)
- Output reserve: **8,192 tokens**

### FileContext object (D1 → D2 interface)
```python
@dataclass
class FileContext:
    files_text: str        # concatenated ### FILE: header + content blocks
    file_count: int        # files that made it in
    token_count: int       # actual token count
    files_ignored: int     # filtered out count
    secrets_redacted: int  # number of redactions made
    truncated: bool        # True if token budget was hit
    files_included: list[str]  # list of included filenames for logging/UI
```

---

## 7. GHE Auth & Fetch (D1 — Completed)

### Auth decision
- **GitHub Apps / OAuth** is the right architecture for a shared service (PATs break when people leave)
- GitHub App issues short-lived installation tokens (1hr TTL) scoped to specific repos
- For MVP: PAT from Vault is acceptable; swap to GitHub App in v2
- **Action item:** Confirm with companyname IT whether a GitHub App is already registered on the GHE instance or needs to be registered (requires GHE admin access)

### GHE API endpoint
```
GET https://{ghe_host}/api/v3/repos/{org}/{repo}/zipball/{branch}
Headers: Authorization: Bearer {token}
         Accept: application/vnd.github+json
         X-GitHub-Api-Version: 2022-11-28
```
- Always `follow_redirects=True` — GHE zip downloads issue a 302
- `timeout=120.0` — large repos can take 15-30s to zip server-side
- Stream to `io.BytesIO` — no disk writes

### Secret scrubber patterns (run after filter, before token count)
- JDBC/MongoDB/PostgreSQL connection strings
- `password=`, `passwd=`, `secret=` in config
- `api_key=`, `token=` (>12 chars)
- `Bearer {token}` in code
- GCP service account `private_key` and `client_email` fields
- All replaced with `[REDACTED]` or `[REDACTED_{TYPE}]`

### Three open decisions from D1
1. **GHE_BASE URL** must be an environment variable: `https://{ghe_host}/api/v3`
2. **Token limit** — confirm Tachyon context window size with companyname AI team before locking 80k
3. **Tiktoken vs Tachyon tokeniser** — if Tachyon uses different vocab, add 10% safety margin

---

## 8. Confluence Integration

### What we know
- Confluence publish is a **later feature** (after MVP)
- MVP focuses on: LLM prompting → document creation → local download
- Confluence comes in a subsequent sprint

### API approach (when ready)
- Check Cloud vs Data Center (different API versions: v2 vs v1)
- Cloud: `POST /wiki/api/v2/pages`
- Data Center: `POST /wiki/rest/api/content`
- ASCII diagrams wrapped in `{code:language=text}` macros
- Create-or-update logic: GET by title → if exists, update with version+1 → else create new
- Store `page_id` after first publish for idempotent future re-runs

### Pre-check before building
- Confirm Cloud vs Data Center with IT
- Manual curl test: create a test page and verify `{code}` macro renders ASCII correctly in monospace font

---

## 9. UI Design

### 4-step wizard flow
1. **Source** — GHE repo URL + branch + optional subfolder (or zip upload)
2. **Context** — App name, team, description, project type chips, focus hint, Confluence settings, publish toggle
3. **Generate** — Live progress screen with per-step status, streaming preview pane, token composition display
4. **Output** — Section checklist, doc preview, Confluence view, download options

### Key UX decisions
- GHE token resolved from SSO session — user never types it
- "What does this application do?" field is the most important input — anchors LLM interpretation
- Project type chips are multi-select, all 7 types supported
- Live streaming preview shows the doc generating in real time (best UX, eliminates "is it working?" anxiety)
- Partial sections flagged with amber badge, not silent failure
- A working HTML UI framework has been built (see `AIDE_UI.html`)

### Diagram output
- **Not Mermaid** (no plugin available at companyname Confluence)
- **ASCII diagrams** using box-drawing characters, rendered in `{code:language=text}` Confluence macros
- Use DejaVu Sans Mono font in any PDF export (supports box-drawing chars — Courier does not)
- Max 70 chars wide per diagram for clean Confluence code block rendering

---

## 10. Pre-Commit Checklist (Blockers)

| # | Question | Who to ask | Priority |
|---|---|---|---|
| 1 | Is calling an internal/external LLM API approved? Is Tachyon accessible from service? | companyname AI Platform team | BLOCKER |
| 2 | Can we reach GHE API (`/api/v3`) from the deployment server? | Platform / IT | BLOCKER |
| 3 | Confluence API accessible? Cloud or Data Center? Write permission? | IT / Confluence admin | BLOCKER |
| 4 | Where do secrets/tokens live? HashiCorp Vault confirmed? | Security team | BLOCKER |
| 5 | GitHub App registration needed or PAT sufficient for MVP? | GHE admin | IMPORTANT |
| 6 | AppSec review process started? (takes 4-6 weeks) | Manager | IMPORTANT |
| 7 | What is Tachyon's context window size? What tokeniser does it use? | companyname AI team | IMPORTANT |
| 8 | Prompt tested manually against 3 real companyname repos? | Self | CHECK |
| 9 | ASCII box-drawing chars render correctly in companyname Confluence code block? | Manual browser test | CHECK |

---

## 11. Discussion Backlog

### D1 — GitHub & File Ingestion ✅ COMPLETED
**Status:** Fully specced and ready to implement.
**What was covered:** GHE API zip download (endpoint, auth, redirects, timeout), `should_ignore()` three-layer filter function, priority ranker per app type, secret scrubber regex patterns, tiktoken counter, 80k token hard cap, truncation strategy, `FileContext` dataclass, user pattern parsing from UI textarea, security patterns that cannot be overridden.
**Open items:** Confirm GHE base URL format with IT, confirm Tachyon context window with AI team, confirm tiktoken vs Tachyon tokeniser.
**To continue:** "Continue D1 — we are implementing the ingestion module. The spec is complete. Help me write unit tests for `should_ignore()` and `build_context()`."

---

### D2 — Smart Prompt Maker 🔜 NEXT
**Status:** Design started but not fully specced. Ready to go deep.
**What needs covering:**
- The complete text of all 7 type blocks written out fully
- The optional pre-pass detection call (cheap 5k-token call to detect project type from file list)
- How to write the base system prompt hallucination guards
- How to structure the focus block injection
- Output format contract — exactly what the LLM must return for each of the 8 sections
- Prompt regression testing methodology — how to validate changes across 7 types
- How to handle the "Mixed" type (multiple blocks concatenated)
- Temperature and other inference params for Tachyon
**To continue:** "Continue D2 — Smart Prompt Maker. Build the complete prompt builder module including all 7 type blocks, the base system prompt text, and the assembly function."

---

### D3 — Token Strategy, Model Selection & Embeddings 🔜 PENDING
**Status:** Not started. Key decisions needed before finalising prompt design.
**What needs covering:**
- Model comparison: Tachyon vs Claude Opus vs GPT-4o vs Gemini 1.5 Pro (context window, cost, quality tradeoffs)
- Token budget split: input files vs prompt overhead vs output reserve
- RAG vs direct context stuffing for large repos (is RAG even needed for MVP?)
- When embeddings help vs when they add complexity without benefit
- How streaming affects perceived latency and UX
- Cost per run estimates across model options
- Retry and fallback model strategy if Tachyon is unavailable
- Whether to use `cl100k_base` or wait for Tachyon tokeniser confirmation
**To continue:** "Continue D3 — Token Strategy and Model Selection. We are using Tachyon as the primary LLM at Compant Name. Help us decide the optimal token budget split, when RAG makes sense, and how to handle fallbacks."

---

### D4 — ASCII Diagram & Data Flow Generation 🔜 PENDING
**Status:** Not started. Feeds into D2 (diagrams are generated by the LLM as part of the prompt output).
**What needs covering:**
- The exact prompt instructions that reliably produce clean ASCII within 70 chars
- The 5 diagram types: architecture overview, DAG dependency tree, column lineage map, error decision tree, sequence diagram
- Programmatic width validation (check each line of ASCII output <= 70 chars)
- Whether to post-process LLM ASCII or trust it raw
- Fallback if a diagram is malformed (regenerate that section only, or skip with a warning)
- How to embed diagrams in the right sections of the markdown doc
- The specific box-drawing character set to instruct the LLM to use
**To continue:** "Continue D4 — ASCII Diagram Generation. Design the prompt instructions and validation logic for all 5 diagram types that AIDE needs to generate."

---

### D5 — Document Output Format & Structure 🔜 PENDING
**Status:** Partially covered in UI discussions. Needs full spec.
**What needs covering:**
- The universal 8-section markdown skeleton in full detail per project type
- Exact markdown table formats for source tables, target tables, column lineage
- The `Not determinable from source — check with app owner` convention
- Version storage schema in the database
- How doc diffs work between v1 and v2 (LLM-generated change summary)
- PDF export via Pandoc — exact command, font flags, DejaVu for ASCII sections
- What "Partial" means operationally vs "Not determinable"
- How to store and retrieve docs (database schema)
**To continue:** "Continue D5 — Document Output Format. Design the full markdown doc schema, version storage model, and Pandoc PDF export pipeline."

---

### D6 — Confluence Publisher 🔜 LATER (post-MVP)
**Status:** Not started. Confirmed as a later feature after core MVP works.
**What needs covering:**
- Detect Cloud vs Data Center automatically from the Confluence URL
- The create-or-update logic with version number handling
- Converting markdown to Confluence storage format (XHTML with macros)
- Wrapping ASCII in `{code:language=text}` macros
- Attaching the PDF to the page
- Error handling: duplicate titles, missing parent pages, expired tokens
- Storing `page_id` for idempotent re-runs
- Notification after successful publish (Slack webhook)
**To continue:** "Continue D6 — Confluence Publisher. We are ready to build the publish module. Confluence is Data Center version. Build the full publisher with create-or-update logic and code macro conversion."

---

### D7 — UI Layer 🔜 LATER (after MVP)
**Status:** A working HTML UI framework exists (`AIDE_UI.html`). Logic and endpoints to be wired.
**What needs covering:**
- Wiring the 4-step form to the FastAPI `/generate` endpoint
- Server-sent events (SSE) for streaming the LLM output to the preview pane
- Project history: database schema, version list, diff view
- SSO integration: companyname Okta / Azure AD via SAML 2.0 or OAuth 2.0
- Settings page: GHE token / Confluence token storage via Vault API
- Error states and retry UX
- The progress screen real-time updates via WebSocket or polling
**To continue:** "Continue D7 — UI Layer. The HTML framework exists. Wire the FastAPI streaming endpoint to the preview pane using SSE, and build the project history view."

---

## 12. Deliverables Created So Far

| File | Description |
|---|---|
| `AIDE_Presentation.pdf` | 5-page manager presentation with Priya's story, UI mockup, backend pipeline, generated doc preview, Confluence preview |
| `AIDE_UI.html` | Working interactive HTML UI with all 4 steps, live progress animation, Confluence toggle, settings screen |
| `AIDE_Project_Context.md` | This document — full project context for LLM handoff |

---

## 13. Build Order (Priority)

### MVP (build this first)
1. `ingestion/` module — fetch, filter, scrub, tokenise → `FileContext` ✅ **D1 specced**
2. `prompt/` module — 4-layer builder, all 7 type blocks → assembled prompt 🔜 **D2 next**
3. `llm/` module — Tachyon streaming call with retry 🔜 **D3 informs this**
4. `output/` module — parser, validator, markdown save
5. Basic FastAPI wrapper — `POST /generate` endpoint
6. Minimal HTML form — repo URL, context fields, run button, preview pane

### Post-MVP
7. Confluence publisher (`publisher/`)
8. PDF export via Pandoc
9. Full React UI with SSO
10. Project history and version diffs
11. Scheduled re-runs on repo push (GitHub webhook)

---

## 14. Key Constraints & Decisions Already Locked

| Decision | Choice | Reason |
|---|---|---|
| Diagram format | ASCII in `{code}` blocks | No Mermaid plugin at companyname Confluence |
| Diagram font in PDF | DejaVu Sans Mono | Only free font with full box-drawing char support |
| Token hard cap | 80,000 (confirm with AI team) | Safe budget for 128k context window |
| Token counter | tiktoken `cl100k_base` (pending Tachyon tokeniser confirmation) | Best available for GPT-4 family |
| Secret scrub | Pre-LLM, before token count, always runs | Security non-negotiable at a bank |
| Auth method | GitHub Apps / OAuth (PAT for MVP) | Service-level auth, not person-tied |
| LLM call pattern | Single streaming call per project | Simpler, faster, no RAG needed for most repos |
| Prompt structure | 4-layer dynamic assembly | Each layer independently tunable, testable |
| Output | Markdown as source of truth | Everything else (Confluence, PDF) derived from it |
| Confluence | Post-MVP | Core value is doc generation, not publishing |

---

*End of context document. Feed this to a new chat to continue any discussion.*
