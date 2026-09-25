# Trust: persons, keys, confirmations, capabilities and signatures

Status: **DRAFT v0.3**, 2026-09-25. Not normative. This document split out of the process-configuration specification at its draft v4.0 and moved here from relux-works/curator-playbook (`spec/trust.md` at `1f87190`, DRAFT v0), so that the core can be frozen while trust goes through its own refuter rounds. The core is relux-works/curator-playbook [`spec/process-configuration.md`](https://github.com/relux-works/curator-playbook/blob/main/spec/process-configuration.md), cited below as *process configuration*. This document decides who counts as a person and how a person's acts are shown. It covers:

- confirmers, confirmations and their records;
- the capabilities a person grants once (§16), and the explicit mode that trusts every change (§17);
- signatures, key assurance, key enrollment and the web of trust between a person's own keys;
- the operator companion app, its pairing, and confirmation of a headless node;
- roster and revocation checks;
- the verification of delegation grants and delegated acts;
- confirmation of the operator's own files;
- privileged operations (§18) and multisignatures (§19);
- the isolation of a dream sandbox (§20);
- the separate OS account that is the real boundary (§12).

The operator accepted the principles of §1 on 2026-09-25 and decided the points marked *decided*. The rest is the author's and waits for its own review. The refuter's sixth run ([`reviews/2026-09-24-refuter-v5.md`](https://github.com/relux-works/curator-playbook/blob/main/reviews/2026-09-24-refuter-v5.md) in curator-playbook) found the gaps that §15 maps. §21 and §22 are the change logs.

**Why a repository of its own, and why not waggle.** waggle defines forms: signed envelopes, keys and their providers, the roster, the wire forms of board acts. This document is the policy that uses those forms: who counts as a person, what needs a person, and what a person's signature shows. Several components rely on it: the board (process configuration), the main loop and the session host, the operator companion app, and trust in Curator's audits. So it is none of them. It is also private. waggle is a public protocol under Apache-2.0, while this design waits for the licensing work and the counsel review (wiki `open-questions.md`, Licensing) before any part of it is published. waggle's Amendment A1 keeps the wire forms of board acts and points here for what they mean.

## Terms

| Technical term | User-facing name | Meaning |
| --- | --- | --- |
| the whole system | apiary | every node and process of one owner; one node is a small apiary |
| node | yard | one machine |
| main loop | keeper | the always-on loop of a node, which manages its processes |
| process | hive | one domain: its project, board, playbook and root orchestrator |
| root orchestrator | queen | the single orchestrator of a process that is started from outside it |
| board | comb | the process's board; its elements are the cells |
| run | bee | one agent run |
| external participants | wild bees | agents of other owners working on part of a process under leases and signatures |
| process package | nuc | a published playbook from which a new process starts |

The operators approved these names on 2026-09-25, the English forms included *(decided)*. Specifications use the technical terms. The wiki's naming page keeps the mapping.

## 0. The design on one screen

- **The root cause.** Agents run as the operator's OS user. Whatever that account can reach, an agent can reach too: the project's files, `~/.curator/`, `ssh-agent`, any key that does not demand presence, and the `task-board` binary itself. A check that runs inside that account stops a well-behaved agent and nothing else. It is a guardrail, not a boundary.
- **The boundary is a person's presence.** Only a signature made with a key that needs a person's presence for every use shows that a person acted. Three kinds of key qualify:
  - a FIDO2 key with user verification;
  - a key in a Secure Enclave, Windows Hello or a TPM whose access control requires the operating system's user authentication;
  - the attested operator companion app.
- **The lasting boundary is a separate OS account.** Later, agents run under an OS account of their own. The stores, keys and binaries they must not change belong to the operator's account, and a broker makes the edits agents are allowed to make (§12).
- **Confirmations are signed records on the board.** Every machine verifies them against the roster of the version in force, without CM2. So one confirmation serves every machine, and each machine checks it for itself.
- **A person grants capabilities once** *(decided 2026-09-25)*. A process asks once for one kind of change, such as creating scheduled tasks or using one named tool. The person answers always, once or no. Changes that stay within a capability in force take effect without a touch. Everything else waits for a confirmation, and one confirmation covers every pending change (§16). An explicit, loud mode trusts every change for those who want it (§17).
- **A person's keys vouch for each other.** The first key is trusted at initialization, with the operator present. A key that an already trusted key of the same person signs is trusted too, which is how trust moves to a new phone (§3).
- **Stages.** L0 now: one operator, one authoritative board. L1: several operators, with M-of-N confirmations. L2: public participants. The invariants are the same at every stage (§13, §19).
- **Delivery.**
  - PC1 ships a guardrail stub: an unsigned operator act outside any run.
  - PC3 brings presence-bound keys and signed confirmations.
  - CM2 brings waggle's forms for the other board acts.
  - The companion app is a track of its own.

## 1. Principles *(accepted by the operator, 2026-09-25)*

- **(a) Only a person confirms.** A confirmation is valid only when it is signed with a key that requires a person's presence: a Secure Enclave key behind biometrics, a FIDO2 key with user verification, or the attested companion app. No project setting lowers this.
- **(b) The signer computes what it signs.** The signing device renders the change itself and signs what it rendered, never a digest computed elsewhere.
- **(c) A chain, not a set.** Confirmations form a chain. An old confirmation never revives after a revert.
- **(d) A person's narrowing takes effect at once.** A person's signed narrowing acts take effect immediately. There are exactly three: revoking a key, revoking a grant, and disabling a tool *(decided 2026-09-25)*. A grant is a delegation grant or a capability grant.
- **(e) One confirmation for every machine.** Confirmations are signed records on the board. Every machine verifies them against the committed roster of the version the confirmation replaces, without CM2. One confirmation serves all machines. The operator's direction that each machine runs what its operator confirmed still holds, because each machine verifies every confirmation itself.
- **(f) Protected first enrollment, then a web of trust** *(decided 2026-09-25)*. The first key is enrolled on first use at initialization, with the operator present. Every later key is enrolled either by a signature of an already trusted key of the same person, or by a confirmation of the confirmers (§3).
- **(g) Staged trust** *(decided 2026-09-25)*. L0 is one operator and one authoritative board, which is now. L1 is several operators, with M-of-N confirmations. L2 admits public participants. The invariants stay the same across stages. Multisignatures are designed now and switched on from L1 (§19).
- **(h) Transport is not authority.** The channel sits behind an abstraction: a tailnet today, a relay or another private network later. Being able to reach a node, or showing a network identity, grants nothing.
- **(i) Secrets stay out of agents** *(decided 2026-09-25)*. Secrets never enter agent environments. Privileged operations run through a trusted executor after authorization (§18).

## 2. The interface with the core

The core decides:

- which changes to the process wait for a person, and which a capability covers (process configuration §2.4);
- which acts need a signature (§4.2);
- what may be delegated (§4.2);
- what an approval binds (§6.4).

This document gives the kernel four answers:

- **`confirmed(version)`**: whether a proposed version of the resolved configuration carries a valid confirmation (§4). The kernel runs a proposed version only when this answer is yes, or when capabilities in force cover every difference of the version (§16, process configuration §2.4).
- **`granted(process, kind[, name])`**: whether a capability grant for that kind is in force for the process (§16), or yolo mode is on (§17).
- **`narrowed(version in force)`**: the version in force with every person's narrowing act applied on top of it (§5). This is what the kernel runs.
- **`verify(act)`**: whether a signed act was made by a principal allowed to make it, with a key strong enough. The act may be a confirmation, a capability grant, an approval, a delegation grant or a delegated act (§3, §6, §7, §16).

In PC1 the trust component is a guardrail stub (§13). The core's rules are the same in every stage; only the strength of these four answers changes.

## 3. Persons, keys and enrollment

- **A person** is an operator principal (`op:…`, waggle §3) that is present in the roster of the version in force and has at least one enrolled key of one of the classes below. Orchestrator, worker and host principals are never persons.
- **Presence classes.** Every enrolled key's record states its class:

  | Class | Key | What a verifier checks |
  | --- | --- | --- |
  | `fido-uv` | a FIDO2 key created with required user verification (`sk-ssh-ed25519`, waggle §4.4) | the FIDO attestation at enrollment; the user-verified flag in every signature |
  | `attested-app` | a key of the operator companion app in the phone's secure hardware (§8) | the platform's attestation chain at enrollment, naming the app and, where the platform attests it, the key's access control. Presence in each use is enforced by the app and the phone's operating system, not written into the signature |
  | `local-os` | a Secure Enclave, Windows Hello or TPM key whose access control requires the operating system's user authentication | nothing in the signature: the evidence is the enrollment record, made in a local session under that authentication and enrolled like every key (below) |

- **Where enrollments live.** The records sit beside the roster (waggle §3): `.waggle/allowed_signers`, the revocation list `.waggle/revoked` and the enrollment records. They are part of the resolved configuration (process configuration §2.4). Adding a principal is a change to the process and waits for the confirmers. A person's own new key is enrolled through the web of trust (below).
- **The trust anchor** (principle f). A board has no key until its operator enrolls one. `task-board trust init` records the anchor:
  - it runs in a local session and prompts for the operating system's user authentication;
  - it enrolls the operator's first presence key or keys;
  - it records the operator's principal as the board's first confirmer.

  It refuses to run in an agent session, which is a guardrail. It cannot complete without the person's authentication, which is the boundary. A board created before PC1 gets its anchor the first time its operator runs the command. Until then the board runs in legacy mode (process configuration §14).
- **The web of trust** *(decided 2026-09-25)*. A person's trusted key may vouch for another key of the same person: a new phone, a second workstation key, a replacement for a lost device.
  - **The certification.** An enrollment record signed by an already trusted key of that principal certifies the new key. Both devices take part: the new device proves possession of its key, and the certifying device renders the new key's fingerprint, class and attestation and signs them with presence (principle b). The record names its certifier, so every key traces back to the anchor or to a confirmation.
  - **Effect.** A certified key is trusted at once for its principal, without the confirmers. A new principal, or a key vouched for by another person, is a change to the roster and waits for the confirmers (§4).
  - **Revocation.** Revoking a key is a narrowing act (§5). A key revoked as *compromised* also revokes every key it certified, down the chain, because whoever held it may have certified their own keys. A key revoked as *retired* leaves the keys it certified in force.
  - **Verification.** A verifier walks each key's certification chain back to the anchor or to a confirmed roster change. It checks every link against the revocation list of the version in force, and it re-checks each attestation against the platform vendors' roots.
- **Recovery.** When every confirmer's keys are lost or revoked, nobody can confirm. The operator of the board's host re-anchors the board through the same local path, with the operating system's user authentication. The re-anchor is recorded on the board and reported at every load and to every former confirmer. It never runs in an agent session.

## 4. Confirmations

- **What is confirmed.** A proposed version of the resolved configuration, when capabilities in force do not cover all of its differences (process configuration §2.4). One confirmation covers every change since the version in force.
- **The record** (principle c). The signed bytes bind:
  - the board's identity;
  - the proposed head: a commit, or an apply sequence number for a project without git;
  - the configuration digest of the proposed version;
  - the configuration digest of the version it replaces, which must be the version in force;
  - the confirmation's sequence number in the board's chain;
  - the confirmer principal;
  - the time.
- **The floor** (principle a). A confirmation is valid only with a signature by an enrolled key of a class in §3. `policy.signatures` never governs confirmations: it names which other acts need signatures, and may only add requirements to them. Nothing about a confirmation is read from the version it confirms.
- **Who may confirm.** A principal of the `people` group that `policy.confirmers` names in the version in force; without that key, the board's anchor operator. A proposed version's own `policy.confirmers` never decides who confirms it. The anchor operator confirms a board's first version. From L1 the confirmers group may require several confirmations (§19).
- **Verification** (principle e). A machine verifies a confirmation against the version it replaces — the version in force — never against the proposed version or the parent of its head. It checks the roster, the revocation list, the enrollment records and the confirmers of that version. So a batch cannot admit its own confirmer. A record is accepted only when its replaced digest equals the digest of the version in force; a record that names any other replaced version is refused and reported.
- **The chain includes capability links.** A version that takes effect because capabilities in force cover every difference has no confirmation record. Every machine derives such a link itself from history: starting at the last confirmation, it walks the first-parent commits of the default branch (or the apply numbers of a project without git). Each commit whose differences from the running version are all covered by the capabilities in force at that commit takes effect. The first commit that is not covered is pending, and so is every later one. Every machine re-derives each link from the two stored configurations and the recorded grants, and never accepts a stored claim that a version took effect. A confirmation's replaced digest is the version in force derived this way (process configuration §2.4).
- **No revival** (principle c). A proposed version whose configuration equals an older confirmed version is still a new change, and it waits for a new record unless capabilities cover it. An old record never confirms a version again.
- **One confirmation for every machine** (principle e).
  - The record is committed to the board under `.task-board/.trust/`.
  - Every machine that loads the board verifies the chain from the anchor to the version in force, capability links included, and runs that version.
  - A machine that cannot verify a record keeps running the last version it verified and reports why.
  - No machine runs a version that another machine merely claims.
- **The signer computes what it signs** (principle b). The signing device can be the machine's own command, the workstation client (§9) or the companion app (§8).
  - It receives the complete canonical resolved configuration of the proposed version and of the version in force.
  - It computes both digests and the authority report itself, and renders the report.
  - It signs only digests it computed.

  The kernel runs only a configuration that hashes to the signed digest.
- **The change analyst explains, never decides** *(decided 2026-09-25)*. A change-analyst role may explain a pending change to the confirmer. Which changes wait is decided by the kernel's deterministic comparison (process configuration §2.4), and what the person confirms is the rendered report.
- **Commands and prompts.** Three kinds of command refuse to run in an agent session (a guardrail): `task-board playbook confirm`, the commands that grant or revoke a delegation, and the commands that grant or revoke a capability. The report prints in the terminal of the session that asks. The operating system's prompt carries the short form of the proposed digest and the number of marked changes, so that even a quick touch confirms something the person can check.

## 5. A person's narrowing acts *(principle d)*

- **The fast path** *(decided 2026-09-25)*. A person can take exactly three narrowing acts, and each takes effect at once:
  - revoke a key;
  - revoke a grant: a delegation grant or a capability grant, yolo mode included (§16, §17);
  - disable a tool.

  "A person" here means a principal of `people` in the version in force, signing with an enrolled presence key (from PC3; in PC1 the act is unsigned, §13). The act is recorded on the board and applied on top of the version in force, without the confirmers.
- **The matching commit follows as an ordinary change**: a line in `.waggle/revoked`, or a tool removed from `policy.enable.tools`. The narrowing record keeps applying until a confirmed version supersedes it. A version that would restore what the act removed waits for a confirmation, and that confirmation names the narrowing record it lifts. A revoked capability is granted again only by a new grant.
- **Commits get no fast path.** A narrowing that someone commits without a person's signed act waits like any other change: the fast path belongs to people, not to commits. Wider narrowing acts, such as pausing a schedule, are open (§14).

## 6. Signatures and key assurance for board acts

`policy.signatures.require_for` (process configuration §4.2) names which acts need a signature. This section decides what satisfies it.

- **`min_assurance`**: `software` < `hardware-bound` < `attested` (waggle §4.5). Whenever a project sets a signature policy, the default is `hardware-bound`, because any process running as the operator's OS user can obtain `software` signatures from `ssh-agent`.
- **`user_verified`**: `required` or `if-supported`. Whenever a project sets a signature policy, the default is:
  - `required` for approvals, relaxations, halts and delegations;
  - `if-supported` for spending, overrides, human transitions and installs.

  A project may set one value for every category, or a value per category, and may lower `required` to `if-supported`.
- **What meets `required`.**
  - A FIDO2 signature with the user-verified flag always does, because anyone can check the flag.
  - An attested companion key meets it for confirmations and for every board act *(decided 2026-09-25, D3)*. It qualifies because its own access control requires the person's biometric authentication, and every verifier re-checks its enrollment evidence against the platform vendors' roots. People confirm and approve through the phone in the common case. Where a platform attests the app without attesting the key's biometric access control, the enrollment record says so, and the project decides whether that key meets `required` for its board acts.
  - A `local-os` key meets the confirmation floor (principle a), because its presence is proven at enrollment. For other board acts it meets only `if-supported`, because its presence cannot be seen in a signature.
- **Rendering.** A board act is signed only through the operator-facing command that renders its payload ("what you see is what you sign", waggle §4.2). An agent never holds an operator key.
- **Before CM2.** Signed board acts other than confirmations, capability grants and narrowing acts need the forms of waggle A1, which arrive with CM2. The core's validator warns about categories that have no signable form yet (process configuration §10). Confirmations, capability grants and narrowing acts are signed board records from PC3 and need no carrier.

## 7. Grants and delegated acts

The core says what may be delegated, to whom and under which ceilings, and how delegated acts count (process configuration §4.2). This section says how the kernel knows who acted.

- **Whose sessions act.**
  - In PC1, the session host's marks say who started a session and which role alias it runs. The marks are a guardrail, and until the session host sets them the kernel records grants but counts no delegated act (process configuration §15).
  - From CM2, a session acts under a grant when its certificate was issued by the grantor's key (waggle §3) and its extension `waggle-role@relux.works` names the delegate role, checked against the version in force.
  - A grant may also name session principals explicitly, never by pattern.
- **A grant is a person's act.** It needs the grantor's confirmation at the floor of §4: from PC3 a signature by the grantor's enrolled presence key, in PC1 the unsigned stub. Where the project requires signatures for `delegations`, it is also a signed board act under waggle A1 §19.8. The kernel verifies a grant once, when it records it; later delegated acts refer to the recorded grant.
- **Delegated acts.** A signed delegated act carries the delegate session's signature in the `delegate` role, and the kernel verifies the chain from the grantor's key to the act (waggle A1 §19.8).
- **Revocation.** Revoking a grant is a narrowing act (§5).
- **No delegation covers a confirmation or a capability grant.** Both are always a person's own act.

## 8. The operator companion app and pairing

The operator companion app is the common way a person confirms and approves, and the only way a headless node is served once it exists. It is a track of its own.

- **Platforms.** The app sits behind one abstraction over each operating system's secure storage and user authentication: iOS, Android, macOS, Windows and Linux. The track starts with one platform, iOS first. A desktop variant can be the control panel: a Mac that runs the main loop and a system-administration role for the dedicated nodes of an apiary.
- **The key.** The app holds a key in the device's secure hardware (Secure Enclave on Apple platforms, Android Keystore, Windows Hello, a TPM) whose own access control requires the person's biometric authentication or the device's user authentication. So the operating system, not only the app, refuses a use without the person.
- **Pairing.** The app is paired with the node through an out-of-band handshake when the node is initialized. The initialization first offers the app itself: the person chooses a platform and gets a link or a code to install it. Then there are three ways to pair:
  - **QR code.** The node renders a QR code as text in the terminal where the operator runs the initialization, locally or over SSH, with a short code beside it as the fallback.
  - **Short code.** On a node without a screen or camera path, the operator types a short pairing code; both screens then show a short authentication string, which the operator compares.
  - **From a paired workstation.** A workstation that is already paired relays the pairing over SSH, or the pairing is bootstrapped through the tailnet identity. The transport abstraction applies here as everywhere else (principle h).

  Each side pins the other's key. Pairing keeps four invariants:
  - the node verifies the attestation of the app's key at every pairing, and refuses a key without it. The attestation must show that a genuine companion app generated the key in the device's secure hardware, naming the app and, where the platform attests it, the access control. Every verifier re-checks that evidence against the platform vendors' roots and never trusts a stored verification result alone (waggle A1 §19.5);
  - the pairing secret is single-use, expires within minutes, and never enters a hosted session's transcript or any log;
  - pairing refuses to run in an agent session;
  - a later pairing, a second phone included, is an enrollment: a trusted key of the same person certifies it (§3), or an enrolled key confirms it.
- **Transport** (principle h). Requests travel over an authenticated channel behind an abstraction:
  - the operator's tailnet today, or another private network later;
  - otherwise a relay that carries only messages encrypted end to end to the pinned keys, so the relay can read and forge nothing.

  Push notifications wake the app.
- **Rendering.** The app renders what is being approved, and signs only what it rendered and computed itself (§4, principle b).
- **A lost phone** is revoked by a narrowing act from another enrolled key (§5). If it was stolen, it is revoked as compromised, together with every key it certified (§3).
- **Later, not in the first version:** the app may hold an SSH key to the node and offer a shell for administering it, where the infrastructure and the person's skills allow. The design allows for it from the start.
- **Open questions** (§14): the design and platforms; the shell; the relay service.

## 9. Headless nodes before the app

Until the app exists, a headless node is confirmed from the operator's workstation.

- A client command on the workstation fetches the complete resolved configurations of the proposed version and of the version in force from the node over SSH. It computes the digests and the authority report itself, renders the report, and signs (§4, principle b).
- The signing key must be a FIDO2 key created with required user verification. The node checks its FIDO attestation when the key is registered, and the user-verified flag of every signature.
- The first registration on a node happens in a local session under the operating system's user authentication, or is proven by that attestation (principle f).
- A second workstation key is an enrollment: a trusted key of the same person certifies it (§3), or an enrolled key confirms it.
- Registration refuses to run in an agent session.
- The key is never used through a forwarded `ssh-agent`, and the client refuses to sign through one. A forwarded agent lets any process of the operator's account on the node request signatures while the session lasts, and a key that shows nothing approves whichever request reaches it first.

## 10. The operator's files

The operator's private files under `~/.curator/` are listed in process configuration §2.3. Entries that widen admission take effect only once the operator confirms them *(decided 2026-09-24, P3)*.

- **The effective version.** The resolver keeps the effective version of each file: the last confirmed version, narrowed by every narrowing edit made since. A narrowing edit applies at once and becomes the new baseline.
- **Widening entries.** A change that admits more than the effective version is a widening entry:
  - a model or a billing class added to `[allow]`, metered billing included;
  - a `[bindings]` candidate outside the effective admission;
  - a standing answer added or extended, or a lengthened review period;
  - a window widened;
  - a runtime binding added to `runtimes.toml`;
  - a skill source outside the Curator registry admitted in `curator.toml`;
  - a confirmation level lowered.

  Before the first confirmation the baseline is the file-less one: no metered billing and no standing answers.
- **`[confirmation]`** sets, per kind (`billing`, `models`, `answers`, `windows`, `runtimes`, `skills`), what a widening needs:
  - `none`: it takes effect at once;
  - `user`: a person's confirmation, as in this section;
  - `second-person`: the confirmation of another operator. It becomes available with waggle's signed acts in CM2; until then the commands refuse it, and a hand-edited `second-person` reads as `user`.

  The defaults are `user` for metered billing, standing answers and skills from unknown sources, and `none` for the rest. The `skills` kind can never go below `user`. Lowering a level is itself a widening and needs the level it lowers.
- **The record.** The operator confirms with `curator models confirm` and the same verb for the other files. The command refuses to run in an agent session. It computes the digest of the widening entries itself, shows them, and signs with a presence key registered on this machine (`local-os` or `fido-uv`), per principle b. The resolver verifies the record before it honours a widening entry. These confirmations are machine-local, because the files are.
- **Billing classes come from facts.** A runtime's billing class comes from how the harness is logged in, never from what a binding says, so a `runtimes` level of `none` never admits metered billing.
- **Until the commands ship**, the resolver honours no widening entry whose level is not `none`, and reports each one it ignores.

## 11. Cross-board hand-off results

A hand-off result that returns across boards is signed by a host key of the receiving board, in the signer role `result` (waggle A1 §19.4), when that board's kernel records the terminal transition.

- No orchestrator session holds that key.
- An OS account other than the operator's, or a privileged service, holds it, whether or not it is `hardware-bound`. So no process running as the operator can sign a result.
- The offering board verifies the signature against the host principals that its committed project file binds to the other board (process configuration §6.5).

## 12. Where things live, and the separate OS account

| What | Where | Protected by |
| --- | --- | --- |
| roster, revocation list, enrollment records | committed under `.waggle/` | part of the resolved configuration: new principals wait for a person; a person's own new keys are certified by a trusted key (§3); a person's revocation acts at once |
| confirmation records, capability grants, narrowing acts | committed under `.task-board/.trust/`, signed | their signatures, verified against the version in force |
| the resolved configuration of the version in force | committed on the board with its digest | at every load, every machine recomputes the digest and verifies the chain back to the anchor, capability links included. It fails closed: it runs the last version it verified, or the built-in default when none verifies, and reports why |
| confirmations of the operator's files | machine-local under `~/.curator/` | signatures by keys registered on that machine; OS-level protection below |
| keys | secure hardware; public keys in the roster or the machine's registry | presence (§3) |
| hand-off host keys | a separate OS account or a privileged service | the account boundary (§11) |
| the `task-board` and `curator` executables | the operator's account today | open: any process of that account can replace them |

The claim that "no session can edit its own authority" holds only against agents that do not replace the kernel, its stores or the local key registry. Until OS-level protection exists, that part is a guardrail.

**The real boundary is a separate OS account for agents** *(decided 2026-09-25 as the direction)*:

- **The account.** Agents run under an OS account of their own, such as a dedicated agent user.
- **What the operator's account owns:**
  - the stores: the stored version in force, the trust records' checkout, the operator's files under `~/.curator/`, and the memory and persona files of the main loop and the processes;
  - the keys and the local key registry;
  - the `task-board` and `curator` executables.

  The agents' account may read what it needs and cannot write any of it.
- **A broker makes the allowed edits.** The broker is a small service running as the operator. It performs exactly the edits agents may make: the operator-file commands, a memory note within its budget and with its provenance, the recording of a board act. It also requests the confirmations that need a person. Each edit goes through a command the broker defines, never through a file the agent writes.
- **Memory is no exception.** Notes that the main loop or a process remembers become part of every future session's context. An agent that can write them directly could plant instructions that persist. Until the accounts are separate, memory, like every file of the operator's account, is protected only by guardrails: the commands' rules, the provenance recorded with each note, review, and quarantine of notes derived from external input.
- **Why not `sudo`.** `sudo` for each edit does not solve it. Either it asks a person for every write, which an autonomous system cannot afford, or it caches the credential for a while, and then any process of the same account uses the cache. What solves it is the separation of accounts plus the broker.
- **Until the accounts are separate**, every rule that keeps agents from changing these stores is a guardrail, and this document says so wherever it matters.

## 13. Stages and delivery

| Stage | Who | Delivered |
| --- | --- | --- |
| L0 | one operator, one authoritative board | **PC1**: the guardrail stub. A confirmation, a capability grant, a delegation grant or a narrowing act is an operator CLI act outside any run, unsigned, and recorded with the fields of §4. `task-board trust init` records the anchor operator's principal without keys, and every unsigned act outside a run carries that principal. A demo commits a roster or names its principals explicitly. PC1 takes versions only on a single-host local board; every other board stays in legacy mode. Everything the core specifies runs; nothing is a boundary. **PC3**: presence-bound keys and enrollment, the web of trust, the anchor with keys, signed confirmations, capability grants and narrowing acts as board records verified against the roster, the workstation client (§9) and the operator-file confirmations (§10). No CM2 is needed. |
| L0, wider | several machines of one operator | **CM2**: waggle A1's forms for approvals, delegation grants, delegated acts, events and hand-off results, and the carrier and lease store that budgets and revocations need across machines; `second-person` for the operator's files |
| L1 | several operators | M-of-N confirmations by the confirmers group (§19); the same records, chain and floor |
| L2 | external participants | admission under leases and signatures; reputation lowers the cost of verification, never grants trust (later) |
| any | the companion app | its own track (§8) |
| later | a separate OS account for agents, and the broker | §12 |

## 14. Open questions

1. **OS-level protection and the broker.** The design of the separate agent account and of the broker: which edits the broker performs, how it authenticates the session that asks, how it runs on each operating system, and how today's single-account installs migrate to it (§12).
2. *Decided on 2026-09-25 (D3):* an attested companion signature meets `user_verified: required` for board approvals and every other board act (§6). The item keeps its number so that the core's references still point to it.
3. **L1 thresholds.** How the confirmers group states its M-of-N threshold, and which other acts a project may put under a threshold (§19).
4. **Wider narrowing acts**, such as pausing a schedule or applying a first version's data-policy narrowing before it is confirmed.
5. **The companion app**: its design and platforms, the desktop control panel, the shell over SSH, the relay service, push, and losing a phone (§8).
6. **Recovery**: how a re-anchor is announced and how former confirmers can contest it.
7. **How a process asks for a capability**: the wording and the channel of the request, how a person grants it from the phone, and how often a process may ask again after a *no* (§16).
8. **Sealing**: two consecutive re-confirmations seal a capability grant; whether a person may seal at once, and whether some kinds never seal (§16).
9. **yolo mode**: whether its duration is capped, and what exactly counts as a shared board (§17).
10. **The web of trust**: whether a certification may cross devices of different presence classes without limit, and how deep a chain may grow before a verifier asks for a confirmation (§3).
11. **Trust in audits and restored skills**: how signed audit records from the Curator registry, the operator's own registry and the operator's personal endorsements on the phone combine into the trusted-skills list that a restore hands to Curator (the main loop's system-administration role; curator-spec#93, #94).

## 15. Findings of the sixth refuter run answered here

The run reviewed draft v3.5 ([`reviews/2026-09-24-refuter-v5.md`](https://github.com/relux-works/curator-playbook/blob/main/reviews/2026-09-24-refuter-v5.md) in curator-playbook). Process configuration §26 maps every finding; these are the ones this document answers.

| Finding | Resolution |
| --- | --- |
| U1 an agent could sign `board.confirm` from CM2 | the floor of principle (a): only enrolled presence keys confirm, and `policy.signatures` never governs confirmations (§4, §6) |
| U2 a batch could admit its own confirmer | verification against the version in force, never the parent of the head (§4) |
| U3 the pinned revision was the unconfirmed head | the core redefines it as the revision of the version in force; nothing here reads the proposed version (§4; process configuration §2.4) |
| U7 an old record re-confirmed a reverted version | records bind the board, the head, both digests and a chain sequence; no revival (§4) |
| U8 a revocation waited for a confirmation | a person's narrowing acts take effect at once (§5) |
| U9 per-machine versions contradicted §7.1 | one confirmation for every machine, verified by each (§4); the operator's per-machine direction holds through each machine's own check |
| U10 the signer signed a digest the node computed | the signer computes the digests and the report itself (§4, §8, §9) |
| U11 the first workstation key was unprotected | FIDO2 with user verification and attestation only; first registration local or attested; later ones certified or confirmed (§3, §9) |
| U12 confirmers were unenforceable and undefined | confirmers read from the version in force; the anchor and first-version rule; recovery (§3, §4) |
| U14 an agent session could raise the prompt | the commands refuse agent sessions; the prompt carries the digest and the count of marked changes (§4) |
| U19 companion signatures and iOS | aligned with waggle A1 §19.5; decided on 2026-09-25 that they count for every board act (§6, §8) |
| U20 where the confirmed configuration lives | the table of §12; the kernel recomputes and verifies at every load and fails closed; the executables named |

## 16. Capabilities *(decided 2026-09-25, process configuration §17.3 item 13)*

A process should not wait for a touch on every change, and a person should not be asked about the same kind of change again and again. So a process asks once, the way a phone app asks for a permission, and the person answers:

- **always**: changes of that kind take effect on commit from now on;
- **once**: this one pending change of that kind takes effect;
- **no**: nothing of that kind takes effect without a confirmation, and the refusal is recorded.

**The grant.** A capability grant is a board record: the grantor principal, the process, the kind and, for a tool, its name; the limits the kind takes; the answer; the time. It is a person's act, confirmed at the floor of §4: from PC3 signed with a presence key, in PC1 the unsigned stub (§13). No delegation covers it (§7). A grant covers a difference only when it is in force at the commit that makes the difference, and every machine checks that for itself when it derives the chain (§4).

**The kinds** are section-scoped. Each covers exactly the differences that process configuration §2.4 lists for it, so the kernel checks coverage section by section and never reasons about paths:

| Kind | Covers | Default |
| --- | --- | --- |
| `preferences` | a role's `prefer` | on |
| `estimates` | estimate dimensions and scales | on |
| `helpers` | adding or removing helpers whose roles are agent roles, and raising a helper's `min` | on |
| `wip` | raising any limit, and adding or lowering limits that count only agent-owned states, agent roles or the board | on |
| `fields` | adding a field that no guard, delegation ceiling, condition, budget, `record_source` or slot contract reads | on |
| `schedules` | adding, changing or removing schedules within the grant's limits (a minimum interval, a maximum number of elements per firing), creating only types the version in force declares, and setting only fields no guard reads | asked |
| `tools:<name>` | listing that tool in `policy.enable.tools`, binding it to an audited provider, and later audited versions of that provider | asked |
| `skills` | audited new versions of the bound role skills and template-carrying skills whose resolved content is unchanged, so that only a role's instructions change | follows `policy.enable.skills`, on by default |

- **The default kinds** are granted with the first confirmed version. The anchor operator's first confirmation lists them, and the operator may decline each.
- **Never a capability** *(decided 2026-09-25)*:
  - people, services, confirmers, the roster and enrollments;
  - thresholds (`field` conditions on routes to a person) and money (`approvals` guards, `risk` markers);
  - human-, tool- and process-owned states, and verification states;
  - delegations;
  - data policy and model admission;
  - signatures;
  - needs other than `prefer`, and the effort floor;
  - check sets;
  - `extends`, `uses`, slots and `bind`;
  - events and their signers;
  - the board's default branch.
- **Workflow structure always waits for a touch.** States and transitions, entities and their workflows, `policy.goals` and `policy.capacity` are never a capability. Touches are batched: one confirmation covers every pending change (§4).
- **Activity capabilities** *(proposed, 2026-09-25)*. One kind covers an activity rather than a difference of configuration:
  - **`dreams`** lets the main loop put this process's orchestrators into dreams (process configuration §6.13), under the process's `policy.dreams` and within the node's capacity.
  - The grant is asked once per process, and it is never granted by default.
  - It covers no configuration difference, confirmation, delegation or other grant. It lets a dream start, nothing more.
  - Changing `policy.dreams` is a change to the process, and it waits like any other policy (process configuration §2.4).
  - Revoking the grant stops future dreams at once. A dream already running is parked, and its outputs stay quarantined.
- **Revocation** of a capability grant is revoking a grant, one of the three narrowing acts: it takes effect at once (§5).
- **Review.** Once a month, together with the review of standing answers (process configuration §9), the main loop lists every capability grant that is not sealed. Until the main loop exists, a board notice does it. The person re-confirms each grant or revokes it.
  - **Sealing.** A grant re-confirmed at two consecutive reviews is sealed: the operator asked to "seal the decision after a couple of reminders", and the number two is the author's. A sealed grant is not reviewed again until it is revoked or its scope changes.
  - **An unanswered review** leaves the grant in force, shown as unreviewed. It is asked again at the next review, and the person can revoke it at any time. This is the author's choice: the kinds are narrow by construction. Standing answers differ: they relax hard requirements, and an unanswered review stops them (process configuration §9).

## 17. yolo mode *(decided 2026-09-25)*

For those who accept the risk, an explicit mode trusts every change:

- **What it does.** While yolo mode is on for a process, or for every process of a node, every change to the process takes effect on commit. Each version is still a link in the chain (§4) and appears in the authority report, so the history stays auditable.
- **How it is switched on.** It is a capability grant of the special kind `yolo`, covering every difference. It needs one touch and may carry a duration, for example `24h`. Switching it off is revoking a grant (§5), and it also ends when its duration runs out.
- **It is loud.** A banner shows in every status, attach and chat view, and in every authority report, while yolo mode is on.
- **Where it is refused:**
  - on a shared board, meaning one with more than one confirmer, one used from more than one machine, or any board from L1;
  - on a process where a delegation covers money: approvals or transitions whose rule has ceilings on a money field, or transitions marked `risk: spending`.
- **What always works.** The emergency stop and the three narrowing acts work in yolo mode as everywhere.
- **When it ends**, the versions that took effect under it stay in force, and the authority report lists them for the person's review.

## 18. Privileged operations *(decided 2026-09-25)*

A privileged operation spends money, acts in the world on the person's behalf, or cannot be undone:

- payments and transfers;
- signing and publishing builds and packages;
- deploys to production;
- the use of secrets: API keys, tokens, passwords;
- messages to external people on the person's behalf;
- physical devices, such as a lock or an alarm;
- deleting data or backups;
- database migrations.

**The rule.** The agent never holds the secret or the right. It requests the operation, and a trusted executor performs it after checking the authority for it: the project's rules, a delegation within its ceilings, or a person's approval with presence where the project requires one.

- **Tools are the vehicle.** The kernel's executor runs them outside every agent's process, in a sanitized environment (process configuration §6.11).
- **Secrets are injected only into the executor's environment**, never into an agent's. The operator's files name a secret only by an environment variable or by a command that yields it (process configuration §2.3).
- **Where the executor lives.** Until the separate agent account exists (§12), the executor runs under the operator's account, and its separation from agents is a guardrail.

## 19. Multisignatures *(decided 2026-09-25 as the design; switched on from L1)*

- **What they are for.** Several operators confirm a change to the process, or approve an act the project marks as critical, with M of N signatures.
- **The record.** It carries one signature per principal over the same bytes, as waggle's multi-signature envelopes do. Each signature is verified at the floor of §4, and the principals must be distinct.
- **Delegated acts** count as their grantor at most once toward M (process configuration §4.2).
- **Stages.** At L0 the threshold is 1. From L1 the confirmers group states its threshold; the shape of that key is open (§14 item 3).

## 20. Dream sandbox isolation *(proposed, 2026-09-25)*

The process configuration proposes dream mode (process configuration §6.13): a process is rehearsed with every tool called with `--dream`, against a copy of the board, and the agent knows it is dreaming. A dream must never become a way around trust.

- **No real secrets.** A dream sandbox receives no real secret. The executor injects no API key, token or password into a dream invocation, and a tool that would need one to act is either simulated by its own dream mode or refused.
- **Privileged operations are only simulated.** Every privileged operation of §18 — a payment, a publication, a deploy, a message to an external person, a physical device, a deletion, a migration — is simulated in a dream and never performed. A tool whose provider does not declare dream support is refused, so an unsupported path cannot reach the real world.
- **Dream evidence never counts for authority.** A confirmation, capability grant, delegation grant, approval, verdict, signed event or narrowing act recorded in a dream has no effect outside it, and the kernel never accepts one as input to the version in force, to `granted()` or to a guard on a real board. A person may read a dream's report before confirming a version; the report is evidence of a rehearsal, and confirming is still a separate act with presence (§4).
- **Read-only access to past sessions** *(proposed)*. In a dream an orchestrator may read the full harness logs of its own previous sessions, read-only and only as the deterministic log distiller prepared them (process configuration §6.13; curator-keeper §13.6). The distiller redacts secret-shaped content — keys, tokens, passwords — that leaked into a transcript. A dream reads only its own process's sessions, never another process's or another person's, and writes nothing back to the logs.
- **What leaves a dream.** Only the dream's report and its insights leave the sandbox. Insights go to dream memory, quarantined until a person reviews them; they never become an instruction to a session, a grant or a confirmation. The proposals the system roles draw from them take the ordinary paths: a batched touch for a change to a process, the person's consent for a change set to the apiary's repository.
- **Signing in a dream.** A dream uses no presence-bound key. A dream that needs a person's act asks for it in dream form, clearly marked, and records it as dream evidence only.
- **Entering and leaving a dream** *(proposed, 2026-09-25)*.
  - A dream starts only while the `dreams` grant (§16) is in force for the process and its `policy.dreams` conditions hold.
  - The dream session runs with the role's dream profile. That profile is data like any Curator profile: it chooses what the context says, never how a process is launched, and it holds no key, secret or grant.
  - Parking the normal session before the dream and resuming it afterwards are the session host's ordinary operations, and they change no authority.
  - A real high-priority doorbell wakes the dreamer. That doorbell is a real message verified as outside a dream, and nothing a dream produces can ring it.
- **The sandbox is not a trust boundary by itself.** While agents run as the operator's OS user (§12), the sandbox's isolation from real secrets and real tools rests on the executor, which is a guardrail until the separate agent account exists. A tool's promise to have no side effect under `--dream` is part of what its audit covers.

## 21. Change log: v0.2 → v0.3

- **The `dreams` capability** (§16): an activity capability asked once per process. It lets the main loop put the process's orchestrators into dreams under `policy.dreams` (process configuration §6.13, curator-playbook DRAFT v4.3), covers no configuration difference, confirmation, delegation or other grant, and is never granted by default.
- **Entering and leaving a dream** (§20). The dream profile is data and carries no key or secret; park and resume change no authority; and only a real, verified high-priority doorbell wakes the dreamer.

## 22. Change log: v0.1 → v0.2

- **Dream sandbox isolation** (§20), for the dream mode proposed in process configuration §6.13 (curator-playbook DRAFT v4.2): no real secrets, privileged operations only simulated, dream evidence never counts for authority.
- **Read-only access to past sessions in a dream** (§20): an orchestrator reads its own process's previous harness logs through the deterministic log distiller, which redacts secret-shaped content; only the dream's report and its quarantined insights leave the sandbox.

## 23. Change log: v0 → v0.1

- **The move.** This document moved from relux-works/curator-playbook into its own private repository, and the reasons are stated in the introduction. Links to the core and to its reviews now point to that repository.
- **Operator decisions of 2026-09-25 recorded in this revision:**
  - capabilities as the answer to which changes wait (§16), replacing the core's allowlist and path check;
  - yolo mode (§17);
  - the web of trust (§3);
  - multisignatures (§19);
  - the separate OS account and the broker as the real boundary (§12);
  - privileged operations (§18);
  - companion signatures counting for every board act (§6, §14 item 2);
  - the change analyst only explains (§4);
  - the English names (Terms);
  - exactly three immediate narrowing acts, where revoking a grant covers delegation and capability grants (§5).
- **The interface** gains `granted()`, and the kernel runs `narrowed(version in force)` (§2).
- **The chain** includes capability-covered links, derived from history by every machine (§4). This answers refuter v6 V3 in the core's review.
- **The PC1 stub** is stated in full (§13): the anchor principal carries every unsigned act, demos commit a roster or name their principals, and only single-host local boards take versions. This answers refuter v6 V11.
- **Drift.** "External participants" no longer carries its user-facing name in §13's text (refuter v6 V16).
