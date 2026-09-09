# Pool transfer shape and fees

Normative contract for the successor to `moe/pool/v2`, implementing
Construction C1.2 and invariant 9. It adopts [pool delivery](pool-delivery.md)
C4.1–8 and does not reinterpret v2 bytes, notes, keys or configuration.
It selects four spend outputs; `pool-v3.md` fixes the final statements,
relations and configuration together before adoption.
The first supported profile uses constant-payout roots and no reliance graph.

## 1. Ordinary outputs

**C1.2.3 The spend has two input positions and four output positions.**
Every position is an ordinary note. There is no reserved fee position, fee
asset, public fee amount, operator debit, extra statement kind or fee-specific
proof relation. A holder chooses which recipients to pay; public order does
not label payment, change or fee. Unused output positions contain distinct
zero-value notes with ordinary owner/rho and delivery capsules, occupy leaves
and are never counted as holdings (C4.6). The wallet may choose output order
before submission and persists it for exact retry.

Both input positions retain pool-v2 §7.2's ownership, immutable nullifier,
positive-note membership and scope membership checks. A zero input names the
other input's backing; two different input backings therefore require two
positive inputs. Nullifiers are distinct and nonzero, and total input value
is positive. Every output names an input backing, has a nonzero owner and
rho, and recomputes its nonzero public commitment. All output commitments are
pairwise distinct. Every value is a `u64`; conservation sums widen each value
before addition and compare the total inputs to ALL outputs separately for
each input backing in `u128`. No output can introduce an absent backing,
shave a quantity, mint a fee or convert one backing's units into another's.
The ordered C4.4 capsule vector covers every output, and the proof binds its
digest through two constrained public `u128` limbs.

Issue still creates one output; burn still consumes two inputs and creates
one change output. Demand, withdrawal, request and lit settlement retain
the recovery contract's shape. Changing the spend bound changes its circuit,
key and configuration domain; it requires a successor backing. It does not
add inputs, multi-operator atomicity, reliance settlement or a general batch.

## 2. A fee is a requested transfer

**C1.2.4 A direct fee uses the recipient's exact output request.** Backer-paid,
crowd-funded or sponsored service remains possible (C2.1.1); a spend does not
have to pay a direct fee to be a valid statement. For direct compensation,
the fee recipient prepares a fresh C4.1–3 request for its chosen backing and
exact amount, using its own secret and seed-encrypted capsule. The payer
obtains it over an authenticated recipient channel, agrees to its price and
includes that exact output/capsule beside payment and change. The fee
recipient checks only its requested opening/capsule and their inclusion in
the proven statement; it needs neither the payer's spend secret nor the
other recipients' openings. The request identifies the intended recipient,
not the payer's civil identity. Output order may be chosen before submission,
but all recipients check their actual position and the bound vector.

Prices may be privately quoted or publicly advertised. Pricing and quote
expiry are service agreements outside shared validity: they add no required
field in E or consensus fee schedule. The
operator may evaluate an agreed fee for a new submission before accepting
it. Once admitted, a valid fee-free statement remains valid on replay, and
an operator cannot invalidate it retrospectively by changing its price.
The protocol proves transfer of claims, not the provision or price of an
external service. Pricing terms do not become a supply or custody authority.

**C1.2.5 All outputs have the same statement fate.** Admission and durable
receipt commit the input nullifiers and every output together, or none of
them. Canonical finality applies to the whole statement; there is no partial
fee payment, no fee-first commitment and no privileged fee receipt. The fee
recipient accepts pending operator liability or finalized holdings under the
same C4.5 and C2.10.9 rules as any other recipient. A receipt alone proves
acceptance, not a finalized holding. Spending uses the ordinary accepted-root
forest: an admitted local output may fund another pending local statement,
and those descendants retain the tail's lapse/finality risk. A fee note is an
ordinary holding in recovery and public supply verification.

An exact replay returns the original receipt before current quote or pricing
checks: a changed price or expired quote cannot defeat idempotency. A reproof
after lapse preserves every output, including the fee, and all capsules
(C4.4); paying a different current operator is not an exact retry. A different
fee after submission requires a separately authorized new spend, only after
canonical evidence establishes the old statement lapsed and the original
inputs remain unspent. Before a statement is submitted, revising an unexposed
draft is not a retry; the payer agrees to any new requested fee before proving.
It cannot rewrite the old receipt or imply the old payment request was paid.
The wallet must reconcile pending acceptance/accounting before reporting a
replacement payment. Fee-only or payment-only refunds are separate holder-
authorized transfers, never reversal of a finalized statement. An unfinalized
fee that lapses creates no fee-recipient snapshot right.

**C1.2.6 Private prices cannot change public remedies.** C2b.5.1–2 counts
valid unserved holding requests without reading a submitted transfer, fee
quote, payment or price dispute. A holder may decline every quote and still
file a counted request; an unpaid fee neither clears that count nor excuses
silence. A service price cannot add a predicate to fault classification,
C3 settlement, snapshot redemption, venue force or canonical adoption.
Operators/backers must price and provision the declared service with those
obligations; the contract does not solve free-request grief by removing the
holder's remedy. Issue, burn and recovery need backer/sponsor funding or a
separate ordinary transfer when priced. They gain no shaved quantity or fee
output here. Venue publication costs in the venue's own currency are separate
from pool transfers and still require actual publication funding.

## 3. Privacy and practical bounds

**C1.2.7 The fee recipient is also a payee.** Public observers still see only
commitments, nullifiers, anchors and fixed-size capsules for a spend. A
sequencer receiving a fee privately knows that output's backing, quantity,
opening and association with the statement, like any payee. In a same-backing
payment/change/fee flow this reveals the payment backing to that recipient by
profile inference. It does not reveal the payment amount, input leaves or
other output openings. A fee computed as a disclosed function of payment size
can leak that size; the smallest supported direct-fee flow uses a quoted
amount independent of the private payment amount. Timing, scope and transport
correlation remain (C1.5).

A fee in an independently chosen second scoped backing does not by itself
identify the payment backing, but narrows the candidates to the same public
scope and still links the fee output to the statement. It is no guaranteed
anonymity set. Sponsored service avoids the direct-fee recipient disclosure.
Wallets must describe the selected funding mode and its actual disclosure;
they cannot promise that an operator receiving a fee sees nothing as payee.

The four-output shape supports, for example, inputs of 100 units of A and
10 units of B, paying 73 A, returning 27 A, paying a fee of 2 B and returning
8 B in one statement. With three positions this case needs an already exact
fee input or a prior preparation transfer, finalized when independent
acceptance is required. Splitting into two unrelated spends does not make
them atomic. A same-backing payment/change/fee uses the fourth position for
an ordinary zero output; sponsored or exact-change cases pad more positions. The output
bound cannot hide an unsupported case by dropping change or paying a partial
amount. Coins spanning more inputs need prior consolidation; cross-scope and
reliance arrangements remain outside this profile.

Each additional output costs one commitment (32 public bytes), one capsule
(89 bytes), one note-tree leaf, a capsule trial during recovery and note
constraints: 121 extra statement bytes before any record framing. It adds
no input ownership/path proof. The fourth output adds this cost to every spend, including those that
would fit three outputs, in return for avoiding routine fee-note preparation
and its extra statement. Four capsule positions total 356 bytes; the two
digest limbs add 64 encoded public bytes, making delivery 420 bytes before
record framing. The reference's decision log records the measured tradeoff;
phone, full-history and venue budgets remain separate evidence obligations.
