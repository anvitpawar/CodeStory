# This is the entire code lifecycle
zip_buffer = io.BytesIO(await http_client.get(ghe_zip_url))  # memory only
files = extract_and_filter(zip_buffer)   # memory only
files = scrub_all(files)                 # memory only, redactions logged
prompt = assemble_prompt(files)          # memory only
response = await llm.stream(prompt)     # response to UI, not stored
del zip_buffer, files, prompt           # explicit memory release
# source code is now gone
```

### What does get stored — and only this
```
generated_document.md    → encrypted at rest, access-controlled
audit_log_entry          → immutable, SIEM-forwarded
redaction_count          → how many secrets were found and scrubbed
files_included_list      → filenames only, not content
token_count              → numeric, not content
```

---

## Layer 4 — The LLM Boundary

### Private endpoint — code never leaves the network
For any enterprise deployment, the LLM runs on a private endpoint inside the organisation's own cloud tenancy. This means:

- Azure OpenAI with a private endpoint — traffic stays inside the Azure tenant
- A self-hosted open-source model (Llama, Mistral) on internal GPU infrastructure
- Any LLM accessed via a VPC peering arrangement with zero public routing

No code traverses the public internet to reach the model. The prompt travels from the CodeStory service to the LLM over a private network connection, just like any other internal service call.

### Zero data retention contract
When using a managed LLM service (Azure OpenAI, Vertex AI), the deployment configuration must enforce zero data retention — meaning the model provider does not log, store, or train on prompt content. This is a contractual and configuration requirement, not just a request. Azure OpenAI and Vertex AI both support this with explicit configuration. This must be verified before go-live, not assumed.

### Prompt injection defence
User-supplied content — the application description and the focus hint — is sanitised before being injected into the prompt. Characters that could be interpreted as prompt control sequences are escaped. The system prompt and user content are sent as separate message roles in the API call, not concatenated into a single string, which is the standard defence against injection at the API level.

---

## Layer 5 — Data at Rest

### Encryption
All stored data — generated documents, audit logs, version history — is encrypted at rest using AES-256. Encryption keys are managed by the organisation's key management service, not by CodeStory itself.

### Secret storage
No secret ever lives in an environment variable, a config file, or a database in plaintext. Every credential — GHE token, LLM API key, Confluence token — is stored in the organisation's approved secret store (HashiCorp Vault, Azure Key Vault, or GCP Secret Manager) and retrieved at runtime over an authenticated API call. The application has no secrets baked in at build time.

### Data classification inheritance
Every generated document inherits the data classification of the source repository. If the repository is classified as Internal, the document is Internal. If it is Confidential, the document is Confidential. The classification label is written into the document header and applied to the storage record at creation time. Publishing a Confidential document to a space that does not support that classification is blocked before the API call is made.

---

## Layer 6 — Output Controls

### DLP scan before publish
Before any generated document is published to Confluence, a Data Loss Prevention scan runs on the output. If the scan detects patterns that suggest the scrubber missed something — a pattern that looks like a credential, a PII pattern, a classification marker — the publish is blocked and the user is notified. The document is available for download but not published until the issue is resolved.

### Publish destination verification
The user specifies a Confluence space and parent page. Before publishing, CodeStory verifies that the user has write access to that destination using the Confluence API. This prevents a user from accidentally or deliberately publishing a document to a space they can write to but should not — for example, a public-facing space for an internal-only repository.

---

## Layer 7 — Audit Log

Every generation event writes an immutable record containing:
```
timestamp           ISO 8601 UTC
user_id             from SSO token, not user-supplied
user_email          from SSO token
repo_url            full URL including org and repo name
repo_branch         exact branch name
commit_sha          the exact commit that was fetched
app_type_detected   what the pre-pass detected
files_included      list of filenames (not content)
files_excluded      count and reason (pattern matched)
secrets_redacted    count of redactions made by scrubber
token_count         input token count sent to LLM
model_version       exact model identifier used
prompt_version      version of the prompt configuration used
generation_status   success / partial / failed
output_doc_id       reference to the stored document
publish_destination if published — the Confluence page URL
duration_ms         total processing time