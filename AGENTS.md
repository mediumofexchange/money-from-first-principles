# Money from First Principles

Three documents in this repository, and they change at different rates.

| | | |
|---|---|---|
| `money-from-first-principles.md` | **the paper** | The argument. Why the object is what it is. Cited, not versioned. |
| `construction.md` | **the protocol** | Normative. A change here changes what every implementation must do. |
| `extensions.md` | **profiles** | Optional, on top of the core. Each names the need that summons it and the price it charges. |

The **Medium of Exchange Protocol** is what Construction and Extensions
define. The paper keeps its own name; an argument is cited, not versioned.
`mediumofexchange.org` is the front door, with the org and the
`@mediumofexchange` npm scope behind it. The reference implementation is
[reference-ts](https://github.com/mediumofexchange/reference-ts), which tracks
this repository as it stands rather than a snapshot.

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
  written: stop, quote the exact passage, explain the problem plainly, and
  propose a fix here. Never build around a spec bug silently, and never pick
  one of several readings without flagging the choice.
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

Prose changes only; there is nothing to build or test. Explain in plain
language when asked, and prefer a readable sentence to a clever one, in the
text as much as in the code — a reader has to be convinced by one pass.

Proceed autonomously with routine, reversible corrections inside an approved
goal. Stop when a change would select a new protocol rule or trust model, and
present the exact ambiguity and tradeoff rather than silently resolving it.

Work in coherent argument- or protocol-sized slices and make logical commits
at completed milestones. For coordinated specification and implementation
work, keep the companion branch and next action current in
`../reference-ts/WORK.md` when that checkout is available. Never push or merge
without maintainer authorization.
