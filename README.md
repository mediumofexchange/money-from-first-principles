# Money from First Principles

A paper and protocol specification for money represented as transferable claims
against promises. The paper derives a common structure for bank deposits,
stablecoins, bills of exchange and informal credit; the **Medium of Exchange
Protocol** defines how those claims can be issued, exchanged and verified.

The aim is open entry: anyone can offer a promise on public terms, and a holder
can independently check the evidence for accepting it. The design makes promises
verifiable; it does not guarantee that they will be kept.

## Start reading

| Document | Purpose |
|---|---|
| [Money from First Principles](money-from-first-principles.md) | The argument, derivation, examples and limits. Start here for the ideas. |
| [Construction](construction.md) | Normative protocol rules, invariants and threat model. Start here to implement it. |
| [Extensions](extensions.md) | Optional features and alternative claim-layer profiles. |

The central object is a backing, `B = (K, P, R, E)`:

| Field | Meaning |
|---|---|
| K | Who owes. |
| P | What one unit pays. |
| R | What must accompany redemption. |
| E | How a claim is verified as unspent. |

The core constraint is authorization: an obligation cannot increase without
its issuer's signature, and a holding cannot move without its holder's
authorization. The core claim layer is a shielded pool, intended to hide
ownership, amounts and histories while making supply publicly verifiable.

## Specification status

The protocol is under development. Versioned layouts distinguish the implemented
construction from successor work; a normative rule is not itself evidence that
the implementation supports it.

| Documents | Scope |
|---|---|
| [Pool v2](pool-v2.md), [authority](pool-authority.md) | Current reference-runtime layouts, circuits, keys and authority rules. |
| [Pool v3](pool-v3.md) | Successor proof and record layouts. Incomplete: configuration, artifact pins and adoption remain undefined. |
| [Recovery](pool-recovery.md), [fault evidence](pool-fault.md) | Presentation, settlement, operator silence and faulty checkpoints for a later construction. |
| [Delivery](pool-delivery.md), [fees](pool-fees.md), [spent set](pool-spent.md) | Successor note delivery, transfer shape, fee and replay contracts. |

[Pool v1](pool-v1.md) is retained as a historical layout reference, superseded
by v2. Earlier versions and rejected alternatives explain compatibility and
design constraints; they are not additional active implementations.

## Implementation and decisions

[reference-ts](https://github.com/mediumofexchange/reference-ts) is the experimental
TypeScript reference. It implements private notes, public supply replay,
canonical history, receipt readers and durable sequencing. Runtime recovery,
a pool wallet and external witness publication remain to be built. It has no
published npm release or completed security audit.

The implementation README provides source setup and pins the specification
revision it follows. The shared [decision index](https://github.com/mediumofexchange/reference-ts/blob/main/DECISIONS.md)
records choices, rationale, alternatives, evidence and specification changes.
[AGENTS.md](AGENTS.md) describes the editing and review process.

Project overview: [mediumofexchange.org](https://mediumofexchange.org).

## Licence

[CC0 1.0 Universal](LICENSE) — public domain dedication.
