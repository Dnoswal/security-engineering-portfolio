# Enterprise LLM / AI Agent Security Threat Model

> Portfolio design exercise — not represented as production deployment experience.

## Scenario
An enterprise introduces an AI assistant that can summarize internal content, analyze security telemetry, and invoke approved automation through APIs.

## Primary Assets
- Enterprise documents and sensitive data
- User and service identities
- API credentials and tokens
- Security telemetry
- Automation privileges
- Model prompts, system instructions, and retrieved context
- Audit records

## Trust Boundaries
1. User → AI application
2. AI application → model
3. AI application → enterprise data sources
4. AI agent → tools/APIs
5. External/untrusted content → retrieval pipeline
6. Automation output → production systems

## Key Threats

### Prompt Injection
Untrusted content attempts to override application instructions or manipulate downstream tool use.

**Controls:** Separate trusted instructions from retrieved content; treat retrieved text as data; restrict tools by identity and policy; validate actions independently of model output; require approval for high-impact actions.

### Sensitive Data Leakage
The assistant returns information outside the requesting user's authorization scope.

**Controls:** Enforce authorization at the data source; use user-scoped retrieval; classify sensitive data; apply DLP where appropriate; minimize prompt/context data; log access decisions.

### Excessive Agent Permissions
An agent receives broad credentials and can perform actions beyond the user's intended task.

**Controls:** Dedicated workload identities; least privilege; short-lived credentials; tool allowlists; parameter constraints; just-in-time elevation; approval gates for destructive or privileged operations.

### Unsafe or Hallucinated Actions
The model proposes or initiates an incorrect operational action.

**Controls:** Deterministic validation; dry-run modes; schema validation; policy engines; human approval for high-risk changes; rollback plans; idempotent automation where possible.

### Poisoned Retrieval Content
Malicious or compromised documents influence model behavior or recommendations.

**Controls:** Source provenance; content trust classification; ingestion controls; malware scanning; retrieval filtering; explicit separation between evidence and executable instruction.

### Insufficient Observability
Teams cannot reconstruct why an AI-assisted action occurred.

**Controls:** Log user identity, model/application version, data sources accessed, tool calls, approvals, policy decisions, outcomes, and relevant correlation identifiers while avoiding unnecessary sensitive prompt retention.

## Reference Security Flow

```text
User
  |
  v
Identity + Authorization
  |
  v
AI Application -----> Policy / Guardrail Layer
  |                         |
  |                         v
  |                   Approval Workflow
  v
Retrieval Gateway ----> Authorized Data Sources
  |
  v
LLM
  |
  v
Tool Broker -----> Allowlisted APIs / Automation
  |
  v
Validation + Logging + Outcome Verification
```

## Design Principle
The model should not become the authorization system. Identity, policy, data access, tool permissions, validation, and auditability should remain enforceable outside the model.
