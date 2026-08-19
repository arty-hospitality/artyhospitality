HEAD
TenantSage — What it actually is

TenantSage is a Governance Execution Platform and Authority Control Plane for AI.

Works authored by Arthit Pukhampuang.

It sits between users/agents and enterprise data, AI models, and external actions. Its job is not to be the chatbot, vector database, identity provider, or LLM. Its job is to determine—and prove—what an AI operation is permitted to access, infer, disclose, and execute before those operations occur.

The central idea is:

Decision Before Retrieval.

A conventional RAG architecture often resembles:

Question
   ↓
Vector Search
   ↓
Retrieve candidates
   ↓
Filter / permissions
   ↓
LLM

TenantSage changes the order:

Verified Identity
       ↓
Authority
       ↓
Governance Decision
       ↓
Eligible Evidence Boundary
       ↓
Retrieval inside that boundary
       ↓
Governed Context
       ↓
Generation
       ↓
Validation
       ↓
Authorized Execution
       ↓
Release

The difference is fundamental: retrieval does not determine authority.

⸻

The canonical S0–S8 pipeline

S0 — Request Ingress

TenantSage receives a request together with verified identity, ultimately through an enterprise identity mechanism such as OIDC/JWKS.

Enterprise IAM
     ↓
S0
     ↓
VerifiedIdentity

Network location does not confer authority.

If identity cannot be verified:

No Verified Identity → No Authority.

⸻

S1 — Deterministic Authority Resolution (DAR)

DAR answers:

Who is this principal, and what authority applies to this exact request?

It resolves things such as:

customerId
principalId
activeScopeId
assignment
role
effective period
authority state

and produces a cryptographically committed:

AuthoritySnapshot

Critically, DAR cannot ask an LLM or vector search what the user should be allowed to see.

Authority is deterministic.

⸻

S2 — Governance

Authority alone does not mean permission.

S2 evaluates the proposed use against policy:

AuthoritySnapshot
       +
action
purpose
environment
classification
retention
legal hold
source restrictions
approval requirements
       ↓
GovernanceDecision

The result is deterministic ALLOW or DENY, plus the constraints governing that operation.

So:

Authority ≠ permission.

⸻

S3 — Eligible Evidence Boundary (EEB)

This is one of TenantSage’s defining mechanisms.

Before retrieval occurs, TenantSage constructs a sealed universe of evidence that the request is eligible to access.

AuthoritySnapshot
       +
GovernanceDecision
       ↓
Eligible Evidence Boundary
       ↓
      eebHash

The critical invariant is:

Query text, embeddings, similarity scores, retrieval algorithms and model outputs cannot enlarge the EEB.

The retrieval system may rank or reduce the eligible evidence.

It cannot create authority.

⸻

S4 — TrustRAG / Governed Retrieval

Only now may retrieval occur.

Sealed EEB
   ↓
Retrieval Admission
   ↓
Search / Vector / Existing RAG
   ↓
Retrieved Evidence

And TenantSage requires:

RetrievedEvidence \subseteq SealedEEB

This is why TenantSage does not need to replace a customer’s existing RAG platform.

A customer could use pgvector, Elasticsearch, Pinecone, Azure AI Search, an internal RAG system, or another retrieval engine.

TenantSage governs what universe that engine is permitted to search.

The retrieval engine is deliberately not an authorization authority.

⸻

S5 — Governed Context

Retrieved material still does not automatically become model context.

TenantSage verifies:

* EEB binding
* retrieval receipt
* evidence lineage
* freshness
* authority state
* governance state
* customer/scope boundaries

Only admitted evidence becomes:

GovernedContext

Therefore:

No verified retrieval proof → No Governed Context.

⸻

Durable write-ahead gate

Before the LLM sees that context, TenantSage requires durable evidence that the governed context was committed.

GovernedContext
      ↓
Durable ledger commitment
      ↓
Generation permitted

This gives another important invariant:

No durable context proof → No generation.

TenantSage therefore does not rely on:

generate first
→ log later

The proof precedes the governed operation.

⸻

S6 — Generation

The model receives only GovernedContext.

TenantSage binds generation to provenance such as:

model provider
model version
model configuration
prompt template
rendered prompt
governed context
S1–S5 lineage

The model’s output is not trusted simply because the model produced it.

It remains a:

CandidateOutput

⸻

S7 — Validation & execution-authority grant

S7 verifies the candidate against the complete governed lineage and disclosure/execution constraints.

For an executable operation there is an important distinction:

CandidateOutput
      ↓
ValidatedAction
      ↓
ValidationApproval

ValidatedAction = what may be executed.

ValidationApproval = authority to execute it.

A valid-looking action by itself therefore cannot trigger S8.

⸻

S8 — Governed Execution

External actions—sending an email, modifying a database, invoking an API, changing a record, etc.—follow the canonical execution chain:

ValidationApproval
        ↓
ExecutionIntent
        ↓
durable intent proof
        ↓
execute exact approved material
        ↓
CompletedExecutionReceipt
        ↓
durable completion proof
        ↓
ExecutionReceipt
        ↓
replay proof
        ↓
response release

This prevents an agent from changing the action after approval or claiming success before execution has been durably proven.

Hence:

No durable intent + completion proof → No successful execution claim.

And ultimately:

No replay proof → No response release.

⸻

The Evidence Plane

There is another part of TenantSage that is just as important as S0–S8.

              GOVERNANCE / EXECUTION PIPELINE
S0 → S1 → S2 → S3 → S4 → S5 → WA → S6 → S7 → S8
│     │     │     │     │     │           │     │
└─────┴─────┴─────┴─────┴─────┴───────────┴─────┘
                         ↓
                 EVIDENCE PLANE
                         ↓
        snapshots / hashes / decisions
        receipts / denials / failures
        intent proof / completion proof
                         ↓
                       REPLAY

The Evidence Plane makes the governance decision reconstructable.

Instead of saying:

“The AI should have followed the policy.”

TenantSage aims to answer:

Who requested this?
What authority existed?
Which policy version applied?
What scope was active?
What evidence was eligible?
What evidence was actually retrieved?
What context reached the model?
Which model/prompt produced the candidate?
What validation occurred?
Who/what authorized execution?
What exactly was executed?
Was it durably committed?
Can the chain be replayed and verified?

That is a substantially different proposition from ordinary AI logging.

⸻

What TenantSage is not

This boundary matters.

TenantSage should not become another:

* IAM system
* vector database
* document-management system
* general-purpose RAG product
* LLM
* chatbot
* Microsoft 365 replacement
* workflow application
* enterprise data lake

Instead:

                Enterprise IAM
                     │
                     ▼
Enterprise Data → TENANTSAGE ← Policies
                     │
             ┌───────┼────────┐
             ▼       ▼        ▼
           RAG      LLM      Tools
                              │
                              ▼
                       Enterprise Systems

The enterprise can keep its existing infrastructure.

TenantSage provides the governed boundary between those systems.

That is also why integrating it with something like Microsoft Copilot is strategically interesting: Copilot remains the user-facing AI; TenantSage becomes the governance engine controlling eligible evidence and governed actions rather than replacing Copilot.

⸻

The simplest explanation

If I had to explain TenantSage to an enterprise architect in one sentence:

TenantSage determines and cryptographically proves what an AI operation is authorized to know and do before the AI is allowed to retrieve data, generate a response, disclose information, or execute an action.

And the architecture can be summarized by five laws:

\text{Authority Before Retrieval}

RetrievedEvidence \subseteq EligibleEvidenceBoundary

\text{Proof Before Generation}

\text{Approval Before Execution}

\text{Replay Proof Before Release}

That is the core of TenantSage—not the UI, Cloudflare, Supabase, pgvector, OpenAI, Microsoft Copilot, or any particular cloud provider. Those are integrations and deployment substrates around the same governance engine.

