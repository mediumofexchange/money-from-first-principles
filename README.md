# Money from First Principles

Almost all money is somebody's promise. A banknote, a bank deposit, a gift card, an IOU between neighbours: in each case someone has said they will pay.

This is a derivation of the smallest object that can carry a promise, and a demonstration that the rest of money is built by combining copies of it.

Today, making money is a licensed activity, and [§1](money-from-first-principles.md#1-what-is-actually-wrong) lays four charges against that: who gets the new money, who is left outside, what happens to credit when a currency dies, and who pays for the failures. The usual answers are to fix the institution, which keeps the licence, or to escape it with money fully backed by something scarce. Full backing builds something real, and it stays small, because it creates almost no money.

This takes a third route: a **grammar** instead of a money. One object, general enough to write fiat, a bank deposit, a bill of exchange, a stablecoin and a neighbour's word.

```
  one object   a backing    B = (K, P, R, E)
                 K  who owes
                 P  what one unit pays
                 R  what must be handed over alongside it
                 E  who says a claim has not already been spent

  one carrier  a claim      a quantity held against a backing.
                            Bearer: whoever holds it, owns it.
  one holding  a wallet     a set of claims, plus any bare assets

  the law      nothing you owe (your written maximum) grows
               without your signature.
               nothing you hold leaves without your signature.

  asymmetry    what must accompany a claim is fixed when the backing
               is signed. what it pays may move with time.
```

The four fields are derived in three steps, and each step answers a failure in the one before. Where the text picks between options rather than following a forced step, it says so.

The object spans the two questions that sort money: reputation or collateral behind the promise, and one hand or many running it. A wallet holds positions at several points at once and moves between them as trust grows in one place and dies in another. Every move is a trade. Move toward collateral and you buy trustlessness, paying in elasticity and liquidity. Move toward reputation and you buy money creation, paying in cyclicality. Move toward a single operator and you buy convenience, paying in that operator's failure modes. The holder decides what they value and where the risk sits.

Out of the object and the law come some of what merchant law took five centuries to find, plus banks, market makers, credit ratings, derivatives and a private lender of last resort. Each appears as a role anyone can take, rather than a licence somebody has to grant. What does not appear is anything needing a judgement made after the fact, and [§18](money-from-first-principles.md#18-limits) lists what that leaves out.

The design substitutes pricing for enforcement. It makes promises readable and continuously repriced. It does nothing to make them kept.

Concentration will still happen, and large backers and banks will grow back. What goes is the legal moat. A village, a supply chain or a trade association could run its own money alongside the established one. The transparent reference implementation is being built toward that goal.

## Where you stand

Most readers arrive holding a position, and nearly all of them sit somewhere inside this object.

| If you come from | You get | Where |
|---|---|---|
| **Credit theory** (Innes, Graeber, chartalism) | money is credit, made to work | [§4](money-from-first-principles.md#4-letting-it-pass), [§17](money-from-first-principles.md#17-every-money-is-a-setting) |
| **Hard money** | your asset as the thing hardness is measured against, and where value runs in a panic without leaving the system | [§10](money-from-first-principles.md#10-roots-grounding-and-hardness) |
| **Free banking** (Hayek, Selgin, White) | the same programme, without appraising each issuer's structure by hand | [§15](money-from-first-principles.md#15-valuation) |
| **Riegel and enterprise money** | a business's own promise as money | [Appendix B](money-from-first-principles.md#enterprise-money) |
| **Mutual credit and community currencies** (WIR, Sardex) | the boundary object | [Appendix B](money-from-first-principles.md#mutual-credit) |
| **DeFi** | the collateral-heavy corner, useful and small | [Appendix B](money-from-first-principles.md#on-chain-backing-and-custody), [§18](money-from-first-principles.md#18-limits) |
| **Cypherpunks** | Chaum's design made composable, with the issuer generalised. His portfolio money is one setting | [§14](money-from-first-principles.md#14-privacy-and-disclosure), [Appendix B](money-from-first-principles.md#portfolio-money) |
| **Gesellians** | demurrage, money that decays to push spending, as one payout setting | [§5](money-from-first-principles.md#5-what-it-pays) |

## Read

The paper is the argument. Construction and Extensions are the **Medium of
Exchange Protocol** — the normative part, and what an implementation tracks.

| | |
|---|---|
| **[Money from First Principles](money-from-first-principles.md)** | The paper. Why, the derivation, the law, what emerges, and the limits. Stands alone. [§17](money-from-first-principles.md#17-every-money-is-a-setting) and [Appendix B](money-from-first-principles.md#appendix-b-the-field-worked) locate every money you already know inside the object. There is a [glossary](money-from-first-principles.md#glossary) at the end. |
| **[Construction](construction.md)** | The protocol. Reference card, invariants, claim layer, sequencing, dishonour, threat model, build order, and every alternative refused. For building it. |
| **[Extensions](extensions.md)** | Optional profiles on top of the core: triggers, pro-rata, references, unitload, the Chaumian profile. Each with the need that summons it and the price it charges. |

[Private-payment research](private-payment-research.md) describes a non-normative
experiment toward one private implementation with public supply verification.
It does not amend Construction or claim a deployable private profile.

## Build

The protocol has one experimental reference implementation. It makes the
transparent rules executable and provides a local payment pilot for testing
durable operation.

| | |
|---|---|
| **[reference-ts](https://github.com/mediumofexchange/reference-ts)** | TypeScript. The transparent setting: encoding, issuance, swaps, sequencing, presentation, dishonour, succession, recovery, and invariant tests. |

The package is not published to npm. With Node.js 20 or newer:

```sh
git clone https://github.com/mediumofexchange/reference-ts.git
cd reference-ts
npm ci
npm run check
```

The [transparent pilot profile](transparent-pilot.md) defines the narrower
initial payment system and its acceptance criteria. The optional
[local pilot](https://github.com/mediumofexchange/reference-ts/blob/main/docs/PILOT.md)
requires Node.js 24 and runs with `npm run pilot:demo`. Its witness is local
and trusted. The API and wire format remain experimental; the implementation
has not undergone a completed security audit.

## Names

Three, because they do three different jobs and change at different rates.

- **Money from First Principles** is the paper, and keeps its name. An
  argument is cited, not versioned.
- **The Medium of Exchange Protocol** is what Construction and Extensions
  define: the object, the law, and the machinery around them. This is the
  thing that gets built.
- **[mediumofexchange.org](https://mediumofexchange.org)** is the front door,
  with the [org](https://github.com/mediumofexchange) and the
  `@mediumofexchange` npm scope behind it.

## Licence

CC0 1.0 Universal — public domain dedication. No permission needed, for
anything, ever. If you build a money on this, you owe nobody here a thing.
