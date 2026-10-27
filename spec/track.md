# Track: trust

Status: **DRAFT v0.4**, 2026-09-27. Specification: [`trust.md`](trust.md). It moved here from relux-works/curator-playbook at that core's draft v4.1. The core is relux-works/curator-playbook `spec/process-configuration.md`; its track owns the delivery milestones this track names (PC1, PC3).

## Why

Agents run as the operator's own OS user, so anything that account can reach, an agent can reach too. The process configuration keeps authority outside the model: some steps belong to people, some changes wait for a person, and some acts need a person's signature. That rule is only as strong as the way the system knows a person acted. This track makes it strong in steps:

- a guardrail stub now;
- presence-bound keys and signed records;
- the companion app;
- a separate OS account for agents.

The rules stay the same at every step, and so does the aim: a person is asked only for what is critical.

## Milestones

| # | Delivers | Done when |
| --- | --- | --- |
| TR0 | this draft: principles, persons and keys, the web of trust, confirmations and the chain, capabilities, yolo mode, narrowing acts, signatures, grants, the companion app and pairing, headless confirmation, the operator's files, privileged operations, multisignatures, stages | its own refuter rounds find no contradiction with the core and with waggle A1, and the operator confirms the decisions this document marks as the author's |
| TR1 (in PC1) | the guardrail stub (trust.md §13): unsigned confirmations, capability grants, delegation grants and narrowing acts, recorded with the fields of their signed forms, through the command line of trust.md §2.1 (`task-board trust init|status|confirm|grant|revoke|disable|enable|keys`, with `task-board playbook confirm` as an alias for one release); `task-board trust init` without keys; capability coverage and the chain derived from history; the monthly review with sealing | the PC1 demos run end to end under the stub on a single-host local board |
| TR2 (in PC3) | presence-bound keys and enrollment, the web of trust, the anchor with keys, signed confirmations, capability grants and narrowing acts as board records verified against the roster by every machine, the workstation client for headless nodes (`task-board trust confirm --node`), the operator-file confirmations (`curator <noun> confirm`) | a change confirmed on one machine takes effect on every machine that shares the board, and none that an agent signed does |
| TR3 (with CM2) | waggle A1's signed approvals, delegation grants and delegated acts, and budgets and revocations across machines | the cafe demo runs its signed approvals and its delegated approvals |
| TR4 | the operator companion app, iOS first, behind one abstraction over each operating system's secure storage | a person confirms and approves from the phone, and a headless node is served by the app alone |
| TR5 | multisignatures (L1) | a group of operators confirms with M of N |
| TR6 | a separate OS account for agents and the broker | agents can no longer change the stores, keys and executables of the operator's account |

## Dependencies

- **The process configuration core** (relux-works/curator-playbook): the classification of changes by capability kind, the version in force, the authority report, and the kernel's executor.
- **waggle** (relux-works/swarma-envelope): envelopes, key providers and their assurance, the roster, Amendment A1's board acts, and multi-signature envelopes.
- **The session host** (relux-works/agent-session-host): marks that say who started a session and which role it runs, the doorbell for requests to confirm, and the refusal of confirmation commands inside agent sessions.
- **The main loop** (relux-works/curator-keeper): how capability requests, reviews and approvals reach the person, through `keeper inbox`, `confirm`, `grant`, `revoke`, `review` and `device` over `task-board trust`; the system-administration role's restore of trusted skills.
- **Curator**: signed audit records (the registry protocol); the attestation-based install path (curator-spec#94); export and import of profile declarations for restore (curator-spec#93).

## Risks

- **Until TR6, the operator's account is shared with agents.** Every rule that keeps agents from changing stores, keys or binaries is a guardrail until then.
- **Presence is only as good as the platform's attestation.** A platform that attests the app but not the key's biometric access control leaves the project to decide (trust.md §6).
- **Capabilities trade safety for fewer touches.** They stay safe only while every kind stays section-scoped and never covers workflow structure or authority.
- **yolo mode is dangerous by design.** It stays confined to single-operator, single-machine processes without money delegations.
