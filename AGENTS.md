# Money from First Principles

The main documents change at different rates.

| | | |
|---|---|---|
| `money-from-first-principles.md` | **the paper** | The argument. Why the object is what it is. Cited, not versioned. |
| `construction.md` | **the protocol** | Normative. A change here changes what every implementation must do. |
| `extensions.md` | **profiles** | Optional, on top of the core, or replacing its claim layer. Each names the need that summons it and the price it charges. |
| `pool-v2.md` | **the core construction's layouts** | `moe/pool/v2` bytes, circuits and keys. A change is a new version and a successor backing. |
| `pool-v3.md` | **successor layouts, incomplete construction** | Six relations, canonical statement/publication records, history/evidence chains, snapshots, receipts, segment headers and fault-evidence records. Configuration, artifact pins and adoption remain unset; no backing may yet declare v3. |
| `pool-authority.md` | **authority and history contract** | Normative C1.2/C2.10 rules for private scopes and independent backing replacement, which v2 instantiates. |
| `pool-recovery.md` | **presentation and recovery contract** | Normative C3/C2b rules over the pool: the demand, lock and settlement, the non-service request and count, snapshot redemption at the venue, and the return from silence. Requires a later construction version. |
| `pool-fault.md` | **fault and evidence contract** | Normative for a later construction: how a checkpoint whose committed evidence fails the rules is classified and passed by descent, snapshot, count, clock and receipts; the revised clock; the amendments its adoption applies. Authenticated exclusion, the snapshot clock and continuation from the last valid prefix. |
| `pool-delivery.md` | **delivery and restoration contract** | Receiver-prepared exact outputs, seed-encrypted capsules bound by statements, public-evidence restoration and its availability limits. Requires a later construction; v2 delivery remains unchanged. |
| `pool-fees.md` | **transfer shape and fee contract** | Fixed successor spend shape, ordinary receiver-controlled fees, replay/finality and privacy/service-grade limits. Requires a later construction. |
| `pool-spent.md` | **spent-set root contract** | Canonical compressed binary root, set-determined imports and per-statement replay. Requires a later construction. |
| `pool-v1.md` | **historical construction layouts** | Fixed-operator `moe/pool/v1` bytes, superseded by v2; never reinterpreted. |

The core's claim layer is the shielded pool (Construction §C1). Transparent,
accumulator and Chaumian claim layers are Extensions profiles. Implementation
profiles, pilots and research contracts live beside the code in `reference-ts`,
not here.

The **Medium of Exchange Protocol** is what Construction and Extensions
define. The paper keeps its own name; an argument is cited, not versioned.
`mediumofexchange.org` is the front door, with the org and the
`@mediumofexchange` npm scope behind it. The reference implementation is
[reference-ts](https://github.com/mediumofexchange/reference-ts), which tracks
this repository. Its README pins the specification revision it implements.

## What this is for

The goal is a working protocol for a medium of exchange that anybody can issue
into and anybody can verify — so that a village, a supply chain, a trade
association, or two people who trust each other can run money that works,
without asking a licensed institution for permission.

The paper already states this precisely, and its wording is the wording to use:

- **"The aim is money creation anyone can contest."** (Abstract)
- **"What goes is the legal moat. Entry is free, terms are public, and anyone
  whose promise is better can undercut an incumbent."** (§1)
- **"The system should work for ordinary people where public infrastructure and
  institutions cannot be relied on: where courts are captured, where the
  currency is dying, where the bank will not open an account."** (§2)

That last sentence is the one to hold onto. It is the human point of the whole
exercise, and it is what makes the expensive choices — no permission, no
censorship, privacy for spenders — worth their cost.

**As a decision rule.** Where two designs are both correct, prefer the one
that:

1. still works when the institutions do not — no court, no regulator, no
   functioning national currency, no bank willing to open an account;
2. needs permission from nobody, and can be entered by a stranger;
3. can be checked by the person being asked to accept the money, on their own
   device, from the published record;
4. fails in compartments, so one backer's collapse is not everyone's.

A feature that is elegant but quietly reintroduces a gatekeeper has lost, no
matter how much it buys elsewhere.

## Simplicity is a correctness property here

The core is one object, one law, and a short list of invariants. Everything is
read by somebody deciding whether to accept a promise, and audited by somebody
deciding whether an implementation is honest. **Every mechanism added is a tax
on both, forever.** A protocol nobody can audit is not secure, whatever its
proofs say.

So a change to `construction.md` has a bar to clear, and it is the same bar
`reference-ts` holds its code to:

- **One mechanism per property.** If a property is enforced in two places, a
  reader must check two places and an implementer can break it in two ways.
- **Never a patch on a patch.** A fix that layers a second mechanism over the
  first to cover its gap is a signal the first is in the wrong place. Ask what
  existing mechanism should be generalised before adding one.
- **Say what it replaces.** A change that adds a rule should name the rule it
  removes, or say plainly why nothing could be generalised to cover it.
- **One throughline.** The object and the law are the core; a change that
  cannot be derived from them, or that needs a special case to sit beside them,
  is probably solving the wrong problem.
- **Say it plainly, and number it.** A normative rule names the objects and
  events it reads — commitment, directory, record, effective index — in the
  words Construction already uses. No new metaphors in normative text: two
  readers, or two models, must not be able to disagree about one sentence.
  Rules in §C2 and §C2b carry numbers (`C2.5.3`); cite the number in commits,
  tests and decisions rather than quoting the paragraph. A retired rule goes
  to Construction's Appendix with what it cost, so it is not rediscovered.

Additions are welcome where they make the whole thing better. Additions that
only make one case work are how a protocol becomes unauditable.

## What may change, and how

- **The paper** takes direct edits, on the standard of *minimal change for
  accuracy*. It is an argument, so corrections are corrections — not rewrites,
  and not new claims smuggled into old sentences.
- **Construction and Extensions** are normative, and a change is a change to
  every implementation. Propose it, say what it costs, and clear the bar above.
- **Implementation work is the best source of spec bugs.** When building
  reveals a contradiction, an ambiguity, or something that cannot work as
  written: pause dependent implementation, quote the exact passage, explain
  the problem, and resolve a fix through the delegated decision/review process
  below. Continue independent work. Never build around a spec bug silently
  or pick one of several readings without recording the choice.
- Spec fixes land **here first**, then the code follows. Divergence between the
  two is a bug in one of them.
- Resolved questions are recorded in
  [reference-ts's decision log](https://github.com/mediumofexchange/reference-ts/blob/main/DECISIONS.md),
  which is an index over `decisions/` — so reopening one is done knowingly,
  with the earlier reasoning in view. A spec change earns a `**Spec change:**`
  pointer on its entry.

## Two words the text deliberately does not use

Worth knowing before "improving" the language.

**"Monopoly."** The paper says *licensed activity*, *the licence*, *the legal
moat*, *the franchise*. These are narrower and harder to argue with — banking
is an oligopoly behind a licensing moat, not literally a monopoly, and the
looser word invites a fight about definitions instead of about the design.

**"Decentralised"** appears as an axis on §17's plane, never as a banner. That
is deliberate and load-bearing. The object's power is that it is a *grammar*
general enough to write fiat, a bank deposit, a stablecoin and a neighbour's
word — and the decentralised-reputation corner is the one it *reopens*, per
§18. Declaring the protocol "a decentralised protocol" would trade that
generality for a slogan, and the derivation is what makes the argument land.

The goal is decentralised. The object is neutral. Both are true, and the text
keeps them apart on purpose.

## Working here

This repository contains prose; normative changes still need evidence from
the reference's model, source or a concrete argument. Check affected links and
cross-document consistency. Prefer a readable sentence to a clever one — a
reader has to be convinced by one pass.

The maintainer delegated development and protocol decisions to AI on
2026-09-08, with independent review, and authorized merge and push when ready.
This is standing authority until superseded. A new rule or ambiguity within
the project's intent is a decision to resolve, not a request for permission.
Retain open entry, independent verification, private payments with public
supply verification, immutable terms, holder authorization and compartmentalized
failure. Prefer practicality, simpler mechanisms, security and lower measured
resource/operating costs within those boundaries.

For a consequential choice, identify the exact rule/ambiguity, compare the
smallest viable alternatives (including reuse or omission), and recommend one.
Explain invariant, trust/privacy, compatibility and practical costs; name a
counterexample or measurement that could falsify it. Have a fresh independent
reviewer inspect the actual proposal and relevant sources. Resolve material
findings with evidence, record the decision and review disposition in the
reference's existing decision log, then commit the specification before code.
No vote count or generated review prompt substitutes for that inspection.
If the companion checkout is unavailable, retain the decision and evidence in
the spec commit body and reconcile the shared log when available.

Use one strong reviewer by default for normative changes; add another only
for a distinct risk or unresolved disagreement. Give agents bounded outcomes,
sources and acceptance criteria, with disjoint write ownership. Review critical
fixes and adjacent variants; stop review when no material finding remains and
the obligations are supported. Routine prose cleanup needs focused self-review.
If review is unavailable, continue safe work and record the review owed; do
not merge the unreviewed normative change.

Ask only for an unavoidable departure from core intent, unavailable access or
physical input, or actions outside existing authority. Merge/push authority
does not itself authorize public releases, live deployment, spending/moving
real funds, destructive operations or access-control changes. Prepare a
concrete recommendation and continue unaffected work when an action is blocked.

Work in coherent argument- or protocol-sized slices and make logical commits
at completed milestones. For coordinated specification and implementation
work, keep the companion branch and next action current in
`../reference-ts/WORK.md` when that checkout is available. Choose slices by
product dependencies and costly uncertainties; validate device, wallet and
venue assumptions early rather than specifying unused mechanisms indefinitely.
Under standing authority, inspect upstream changes, satisfy required reviews
and repository checks/protections, merge and push, then verify remote parity.
Do not bypass a failed gate or describe unavailable evidence as a pass.

Keep instructions in `AGENTS.md`; `CLAUDE.md` only imports `@AGENTS.md`.
Keep current handoff concise and reasoning in the decision log. End reports
with delivered behavior, evidence, integration state and limitations, plus a
rough percentage done/remaining toward the smallest usable product with a
range for roadblocks. Use the reference's production requirements and handoff
when available; do not infer product completion from prose or test counts.

When work exposes a clearer project structure or workflow, make the low-risk
improvement if it belongs to the active goal; otherwise leave a concise
suggestion in the companion handoff with the benefit and cost, rather than
letting the side issue interrupt the protocol change.
