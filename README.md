# curator-trust

Trust for agent systems. The design says who counts as a person, how a person's acts are shown, and how an agent system stays safe while agents run as the operator's own OS user. It covers:

- confirmations bound to a key that needs a person's presence;
- capabilities a person grants once;
- how delegated acts are verified;
- the operator companion app and its pairing;
- how a system grows from one operator to many.

## Status

**Draft design.** Nothing is implemented here. The specification is [`spec/trust.md`](spec/trust.md), with its track in [`spec/track.md`](spec/track.md). Both arrive through pull requests.

The repository is private until the licensing work and the counsel review are done.

## How it fits

- **Process configuration** (relux-works/curator-playbook, `spec/process-configuration.md`) decides which changes to a process wait for a person and which acts need a signature. It reaches this design through one interface: a version takes effect when the trust component reports it confirmed.
- **waggle** (relux-works/waggle) defines the wire forms: signed envelopes, keys and their providers, the roster, board acts. This design is the policy that uses them.
- **The main loop** (relux-works/curator-keeper) and **the session host** (relux-works/agent-session-host) carry a person's confirmations and approvals to a node, often from the companion app on a phone.
- **Curator** (relux-works/curator-spec) supplies the audit registry. Its signed audit records are one of the trust anchors this design builds on.
