# Threat Model

This document catalogs threats relevant to goose deployments, describes the expected model behavior for each threat, and identifies the runtime control that enforces safety when the model alone is insufficient.

---

## 1. Prompt Injection

**Risk**: Malicious instructions embedded in repository files, web pages, log output, tool results, or generated code cause the model to take actions the user did not authorize—such as exfiltrating data, modifying files, or bypassing policy.

**Expected goose behavior**: goose treats `system.md` rules as unconditional. Content from files, tool output, or the environment is treated as data, not as instructions. goose does not execute, relay, or act on instructions found in repo files, web content, or tool output that contradict policy.

**Recommended runtime control**: Deny-by-default tool access; audit logging to detect unexpected action sequences; confirmation gates so injected commands cannot execute silently.

---

## 2. Secret Exposure

**Risk**: Credentials, API keys, tokens, or passwords present in `.env` files, config files, log output, or tool results are echoed into model responses, written to generated documents, or included in commit messages or issue comments.

**Expected goose behavior**: goose must not echo or repeat values that appear to be secrets. When encountering what looks like a credential, goose references its presence without displaying the value (e.g., "a token is present in `.env`" rather than printing it).

**Recommended runtime control**: Secret redaction in the runtime pipeline scrubs known secret patterns from tool output and file content before the model sees them. Audit logs are checked for credential-shaped strings before storage.

---

## 3. Destructive Filesystem Actions

**Risk**: Files are deleted, overwritten, moved, or renamed without the user's informed consent, resulting in data loss that may not be recoverable.

**Expected goose behavior**: All destructive file operations are Level 3 and require a structured confirmation prompt including rollback information. `--ok` does not authorize any destructive file operation. goose does not infer authorization from context.

**Recommended runtime control**: Project-root filesystem sandboxing prevents writes outside the project tree. Soft-delete or backup behavior routes deletes through a recovery location. Confirmation gates block execution until the user approves.

---

## 4. External Network Exfiltration

**Risk**: Data is transmitted to an external host via `curl`, `wget`, HTTP client libraries, DNS lookups, or other network mechanisms—either through direct model action or through injected commands in tool output.

**Expected goose behavior**: External network requests are Level 3 and require explicit confirmation including identification of the destination and educational/business purpose. goose does not initiate network requests under `--ok`.

**Recommended runtime control**: Network egress is blocked by default at the OS, container, or proxy level. Specific destinations may be allowlisted by the operator. All network attempts are audit-logged.

---

## 5. Package Manager Risk

**Risk**: Package installation introduces malicious, vulnerable, or unexpected dependencies, or modifies lock files in ways that affect reproducibility or security.

**Expected goose behavior**: Package installation is Level 3 and requires explicit confirmation naming the package, version, and package manager. `--ok` does not authorize package installs.

**Recommended runtime control**: Package manager commands are blocked by default and require operator approval. Network egress controls limit which registries are reachable.

---

## 6. Project-Code Execution Risk

**Risk**: Arbitrary project code (test suites, build scripts, CLI tools) is executed by the model, causing unintended side effects such as database writes, API calls, file mutations, or resource consumption.

**Expected goose behavior**: Executing project code is Level 3 and requires explicit confirmation. goose does not run project code under `--ok` and does not infer authorization from the presence of run scripts.

**Recommended runtime control**: Shell command execution is routed through a confirmation gate. Audit logging records all shell invocations. Network egress controls limit what executed code can reach.

---

## 7. Git History Leaks

**Risk**: Commit messages, branch names, diff content, or stashed changes contain sensitive data (credentials, student records, internal identifiers) that gets pushed to a remote or included in generated summaries.

**Expected goose behavior**: All Git mutations (commit, push, reset, rebase, merge, branch deletion, history edits) are Level 3. goose must not include sensitive data in commit messages or branch names. goose does not push under `--ok`.

**Recommended runtime control**: Confirmation gates for all Git push operations. Secret redaction applied to commit message content before logging or display. Pre-receive hooks on the remote can block pushes containing credential patterns.

---

## 8. FERPA / LMS / SIS / Advising / Gradebook Data Exposure

**Risk**: Identifiable student records (names, IDs, grades, accommodations, conduct records) are included in model responses, generated files, commit messages, issue comments, or transmitted externally.

**Expected goose behavior**: All operations involving identifiable student data are Level 3. goose de-identifies before summarizing where possible, avoids writing student data to uncontrolled artifacts, and requires explicit authorization with stated destination and purpose before any external transmission.

**Recommended runtime control**: Audit logging captures all file reads of known student-data paths (Canvas exports, SIS exports, gradebook files). Confirmation gates require operator approval for operations on these files. Network egress controls prevent unauthorized transmission.

---

## 9. Overbroad `--ok` Behavior

**Risk**: A user appends `--ok` to a destructive, network-accessing, or FERPA-sensitive request and the model treats it as blanket authorization.

**Expected goose behavior**: `--ok` is explicitly scoped to non-destructive, project-local, reversible Level 1/Level 2 work. goose must refuse the prohibited portion of any `--ok` request that includes a Level 3 action and explain why. goose does not honor combined `--ok` + prohibited-action instructions.

**Recommended runtime control**: Confirmation gates enforce approval requirements independent of the model's interpretation of `--ok`. Audit logging flags cases where `--ok` was present on a Level 3 action for operator review.

---

## 10. Privilege Escalation and System-Wide Changes

**Risk**: Commands are run with elevated privileges (`sudo`, `su`, administrative APIs) or modify system-wide configuration (host files, system libraries, global environment variables) outside the project directory.

**Expected goose behavior**: Privilege escalation and system-wide changes are Level 3. goose does not execute commands requiring elevated privileges under `--ok`. goose does not access or modify paths outside the project directory.

**Recommended runtime control**: Project-root filesystem sandboxing enforced at the OS or container level. Deny-by-default tool access prevents exposure of privileged tools. Audit logging records all shell commands and flags elevated-privilege invocations.
