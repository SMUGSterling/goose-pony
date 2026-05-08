# Security Model

goose uses a **defense-in-depth** approach. No single layer is assumed to be sufficient on its own. The three layers below are designed to complement each other; a failure in one layer should be caught by the others.

---

## Layer 1: Prompt Policy (`system.md`)

`system.md` is the behavioral policy enforced by the model itself. It covers:

- **Risk classification**: Actions are classified as Level 1 (read-only), Level 2 (mutating but reversible), or Level 3 (high-risk / potentially irreversible). Level 3 actions always require explicit confirmation before execution.
- **Confirmation behavior**: Level 3 confirmations must include a structured prompt with Action, Target, Risk, and Rollback fields. The model must not proceed on ambiguous responses.
- **FERPA and student-data handling**: Identifiable student data is always Level 3. The model must minimize data exposure, de-identify before summarizing, avoid persisting student data to uncontrolled artifacts, and refuse external transmission without explicit authorization.
- **Secret redaction**: The model must not echo values that appear to be credentials, tokens, or keys, even if they appear in file content or tool output.
- **Project-local boundaries**: The model operates within the project directory tree and must not access or modify paths outside it.
- **`--ok` scope restriction**: `--ok` authorizes only non-destructive, project-local, reversible work. It does not authorize deletions, Git mutations, package installs, network access, code execution, or FERPA-sensitive operations.
- **Remediation mode**: The model must not attempt to work around runtime controls, even if instructed to do so.

Prompt policy relies on the model following its own instructions. It is not a hard enforcement boundary. Runtime controls (Layer 2) are required to enforce what the model cannot self-enforce.

---

## Layer 2: Runtime Controls

The following controls should be enforced by the tooling or execution environment, independent of the model's behavior:

| Control | Description |
|---|---|
| Project-root filesystem sandboxing | File read/write access is restricted to the project directory tree. Attempts to access paths outside it are blocked at the OS or container level. |
| Deny-by-default tool access | Only explicitly approved tools are exposed to the model. Tools not on the allowlist are not available, even if the model requests them. |
| Allowlisted read/write paths | File operations are limited to an explicit list of approved paths or directories. |
| No external network by default | Outbound network access is blocked unless the operator explicitly enables specific destinations. |
| Confirmation gates for high-risk actions | The runtime intercepts Level 3 actions and requires user approval before execution, independent of the model's own confirmation prompts. |
| No package installation without approval | Package manager commands (npm, pip, cargo, gem, etc.) require explicit operator approval before execution. |
| Audit logging | All file operations, shell commands, Git operations, package manager invocations, and network requests are logged with timestamps, arguments, and outcomes. Logs are append-only and not accessible to the model for modification. |
| Secret redaction | Credentials, tokens, and keys are scrubbed from logs and model-visible context (tool output, file content summaries, etc.) before the model sees them. |

These controls reduce the blast radius of a model error or a successful prompt injection attack. Operators should verify that each control is enforced before deploying goose in sensitive environments.

---

## Layer 3: Human Review

Humans remain the final decision-makers for high-risk operations. Human review is required—not optional—for:

- **FERPA and student data**: any operation involving identifiable student records, regardless of apparent authorization.
- **Credentials and secrets**: any rotation, transmission, or storage of credentials or API keys.
- **Legal matters**: contracts, compliance attestations, regulatory filings, or actions with legal consequences.
- **HR matters**: performance records, disciplinary actions, hiring decisions, or personnel data.
- **Research data**: datasets covered by IRB protocols, data use agreements, or HIPAA.
- **Security incidents**: suspected breaches, anomalous access patterns, or disclosure of sensitive data.

No combination of `--ok`, chat confirmation, or model reasoning substitutes for human review of actions in these categories. Institutional policy governs the approval process; this document does not override it.
