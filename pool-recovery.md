# The shielded pool's presentation and recovery

This is the normative contract for Construction [§C3](construction.md#c3-presentation-and-dishonour)
and [§C2b.3–C2b.6](construction.md#c2b-failure-silence-and-recovery) over the
shielded pool: how a holder presents notes to the backer and how the operator
locks and settles them; what a non-service request is and how the grade is
counted; how snapshot redemption runs at the venue while the operator is
dark; and what the first commitment after a silence adopts. The maintainer
approved the finality boundary on 2026-09-07 (Construction C2b.3a–c): a
recovery holding is a positive note in canonical finalized history, and an
unwitnessed spend or receipt neither creates a holding nor redirects one.
This contract instantiates that boundary. It adds objects, so it requires a
new construction version: [`moe/pool/v2`](pool-v2.md) carries none of them
([§7.4](pool-v2.md#74-redemption-and-what-this-version-does-not-carry)), and
its bytes, circuits and keys are unchanged. The byte layouts and circuits that
instantiate this contract are a later document's, `pool-v3.md`.

The contract covers a backing whose reliance set is empty and whose payout
settles outside the claim layer: the smallest supported profile. Its rules are
numbered under the Construction rules they refine: C3.1–C3.8, C2b.5.1–2,
C2b.6.1, C2b.3.1–3 and C2b.4.1–2. Words are pool-v2's: note, commitment,
nullifier, anchor, accepted-root forest, segment, scope, checkpoint, history,
finalized prefix, import. "Judged at an index" means judged against the
record strictly before that index (Construction §C2b.4).

## 1. Objects

**C3.1 A claim's public name is its tag.** `tag = H(T_TAG, nf)`: the
construction's in-circuit hash, under a tag distinct from every other, of the
note's immutable nullifier. This is Construction §C3's `H(nullifier)`. A tag
appears in a notice; the nullifier appears only in a spend or a settlement.
Whoever later sees the nullifier can compute the tag, so a demand, its
withdrawal and its settlement are linkable to each other and to the note's
eventual spend. That is [§C1.5](construction.md#c15-what-still-leaks-and-what-a-wallet-does-about-it)'s
aborted-presentation leak: a wallet spends a withdrawn note to a fresh one
before using it again.

**C3.2 The holding proof** is one relation with two public forms. Over the
construction's `inputs` note positions, all of one backing, it proves for each
position `i`: `secret_i ≠ 0`, `owner_i = H(T_OWNER, secret_i)`,
`nf_i = H(T_NULLIFIER, domain, cm_i, secret_i)`; where `value_i > 0`, that a
path carries `cm_i` to the public `anchor_i` and `tag_i = H(T_TAG, nf_i)`;
where `value_i = 0`, that the position is padding and `tag_i = 0`. Every note
names the public `backing`, at least one value is positive, and every value is
below `2^64`. The proof consumes nothing and creates nothing.

- The **segment-bound form** is a demand's. Its public inputs are the domain,
  the segment identity, the scope root, the backing, the `quantity` (the sum
  of the values, in 128-bit arithmetic), the anchors, the tags and the hash of
  the notice it belongs to (C3.3). Scope membership is proven as in a spend.
- The **segment-free form** is a request's (C2b.5.1). Its public inputs are
  the domain, the backing, one anchor and one tag: no segment, no scope root
  and no quantity. It proves one real note without revealing its size.

Which forest an anchor must belong to is the reader's rule: an operator reads
it against its segment's accepted-root forest (C3.7), a venue reader against
the snapshot's (C2b.3.2), a non-service reader against the canonical state it
counts at (C2b.5.2). A proof is not a holding until a reader has placed its
anchor.

**C3.3 The demand** is a notice and the segment-bound holding proof for it.
The notice names the backing, the quantity, the tags (the nonzero tags of the
proof, in position order), the **presenter key**, the evaluation instant and
the deadline. The instant is a witnessed index no later than the latest index
witnessed when the notice was signed (invariant 24); the deadline is a
witnessed index strictly ahead of the index at which the demand is admitted or
witnessed (Construction §C3). The presenter key is the holder's choice and
signs the release and the withdrawal; the proof binds the notice, so nothing
else signs the demand. Its identity is the hash of the notice with the
proof's public inputs. A demand names at most `inputs` claims: a holder
presenting more, or part of a note, first spends to itself
([§C1.2](construction.md#c12-the-shielded-pool), packing).

**C3.4 The acceptance** is the obligor **K**'s strict signature over the demand
identity, an `owner` value and the acceptance deadline. **K** generates the
spend secret behind `owner` and never reveals it (invariant 25); `owner` is
public. The acceptance deadline is a witnessed index at or before the demand's
deadline, or the object is not an acceptance. Its identity is the hash of
those fields. The acceptance need not be published: it is carried by the
release (C3.6), and a backer publishes it at the venue only where it wants the
evidence that it answered (C3.8).

**C3.5 The settlement statement** is a statement of kind `settle`. Its public
inputs are the domain, the segment identity, the scope root, the backing, the
quantity, the `owner`, one anchor and one nullifier per input position, one
output commitment `cm_out`, and the demand identity. The proof establishes,
for each input, what a spend establishes ([pool-v2 §7.2](pool-v2.md#72-spend)):
ownership, the immutable nullifier, membership at the anchor where the value
is positive, padding where it is zero, and scope membership; that every input
and the output name the public backing; that the sum of input values equals
the quantity and the output's value, in 128-bit arithmetic; and that
`cm_out = H(T_NOTE, domain, backing, quantity, owner, rho_out)` with `rho_out`
and `cm_out` nonzero and `nf_1 ≠ nf_2`. The reader checks in the clear that
the demand's tag at each position is either `0` or `H(T_TAG, nf_i)`: the
demand already proved that its tagged notes sum to the quantity, so a
position tagged `0` carries no value in the settlement either, and the
settlement consumes exactly the claims the demand named (invariant 27).

Its effect is a spend's: the nullifiers enter the spent set, `cm_out` is
appended, and the demand is discharged. `outstanding` is unchanged
(invariants 10 and 12): the backer becomes the holder of `cm_out`, and burns
it where its terms say so in a separate lit act. Redemption is lit at the
backer ([§C1.4](construction.md#c14-who-sees-what)): the public learns that
`quantity` units of `backing` settled to `owner`, and nothing about where they
came from beyond the anchors.

**C3.6 The release and the withdrawal.** The release is the presenter key's
strict signature over the demand identity, the acceptance identity and the
settlement's `statementHash`; it is served with the acceptance and the
settlement statement, and with the non-membership proofs C2b.3.2 requires
when it is published at the venue. The withdrawal is the presenter key's
strict signature over the demand identity. Each is effective when witnessed:
under service, by admission into the segment's history; in a gap, by the
venue (C2b.3.2). An exact resubmission or republication is the same act and
returns the same answer (invariant 26).

**C2b.5.1 The request** is the segment-free holding proof over one note. No
key signs it; the proof is the holder's act. It is published at the backing's
declared venue naming the backing. It is Construction §C2b.5's non-service
object and §C3's demand shape without the backer: a real note, its size
hidden, waiting to be served.

## 2. Presentation under service

**C3.7 The standing-demand record and the pending-lock set are the segment's.**
A segment's history holds statements of kind `demand`, `withdraw` and `settle`
beside issue, spend and burn, in one order, and the history hash binds them
(pool-v2 [§9](pool-v2.md#9-the-ordered-history-the-snapshot-and-the-receipt)).
A commitment therefore commits to the standing demands and the locks
(invariant 23), and a replayer recomputes both from the deduplicated events
of the imported closure and the local history, exactly as it recomputes the
spent set. Admission reads one committed view ([pool-v2 §8](pool-v2.md#8-admission)):

- A **demand** is admitted where its proof verifies under this segment and
  scope root, every anchor is in the accepted-root forest, the backing is in
  the scope and its terms are held, the deadline is strictly ahead (C3.8), and
  no nonzero tag is **locked** or **spent**. A tag is spent where it is the
  tag of a nullifier in the spent set; the operator keeps that set of tags
  beside the spent set, and a replayer recomputes it. Its effect: each nonzero
  tag enters the pending-lock set keyed to the demand identity, and the demand
  is standing. Nothing moves.
- A **spend** or **burn** whose nullifier's tag is locked is refused: a lock
  reserves without consuming, and only the settlement of its own demand
  consumes what it reserves.
- A **settle** statement is admitted where its proof verifies under this
  segment, its anchors are in the forest, its demand is standing, its backing
  and quantity are the demand's, its tags match (C3.5), the acceptance
  verifies under the backing's **K** with the settlement's `owner` and a
  deadline not behind (C3.8), the release verifies under the presenter key,
  and its nullifiers and output are new. Its effect is C3.5's, and the
  demand's locks are released with it.
- A **withdrawal** is admitted where its demand is standing and the signature
  is the presenter's. Its effect: the demand is no longer standing and its
  locks are released.
- A demand whose tags are locked under another standing demand, a settlement
  or withdrawal of a demand not standing, and a settlement of a demand under
  another segment are refused. A tag demanded in two distinct events of the
  imported closure with neither settled nor withdrawn between them is a
  conflict the import refuses, as a duplicate nullifier is (C2.10.6).

The core presentation is single-phase: one operator serves the whole set
(Construction §C3), so there is no attempt timeout here. A demand stands until
it is settled or withdrawn, and the deadline governs evidence, not the lock.
Cross-operator presentation, with its prepare, decision venue and lock
timeout, remains the [Extensions profile](extensions.md#cross-operator-presentation).

**C3.8 Deadlines at the door, and dishonour.** The operator reads a deadline
against the index at which an act it co-signs now would first be witnessed:
the latest index it has read plus the venue's lag ([§C2.3.5](construction.md#c23-witnessing)).
It refuses a demand whose deadline is not strictly ahead of that index and a
settlement whose acceptance deadline is behind it. These refusals keep each
party to its own terms; they are not supply rules, and a replayer does not
re-judge an admitted settlement's deadlines, since both parties consented to
it. Dishonour is Construction §C3's branch read from the history and the
venue record: a demand standing past its deadline with no settlement is the
backer's visible failure, unless an acceptance of it, witnessed before its
own deadline, stood unreleased, which is the holder's lapse. A backer that
wants that evidence publishes its acceptance at the venue; nothing else reads
it there.

## 3. Non-service

**C2b.5.2 The count.** **E** declares the non-service duration `d`, the count
`m` and the window `W`. At index `t` a reader counts, for backing `b`,
against the **canonical state** at `t`: the state of the last canonical
finalized checkpoint carrying `b`, witnessed at or before `t`, by a party then
in force for `b` (C2.10.4). It counts the distinct tags of requests naming `b`
witnessed at indices in `[t − W, t − d]` that are **valid** against that
state — the proof verifies and the anchor is a root of the state's forest —
and **unserved** at it: the tag is neither the tag of a nullifier in its spent
set nor in its pending-lock set. A count of at least `m` fires the grade
against the party in force for `b` at `t`. A tag requested twice counts once.
A request whose proof fails or whose anchor the state does not certify counts
for nothing; a request the canonical state has spent or locked has been
served, whether by this party or its predecessor. A handover neither resets
nor moves the count: the successor inherits the standing requests and clears
them by serving them, and an operator that drops a backing from its scope
while committing the rest is reached here, read against the last checkpoint
that carried the backing (Construction §C2b.5, §C2.4.5). The cost stands as
declared: one holder can split a holding into `m` notes and file `m`
requests, and the pool hides the sizes that would weight them.

## 4. Silence and the snapshot

**C2b.6.1 The no-commitment clock.** **E** declares the no-commitment
duration. For backing `b` at index `t`, let `c(t)` be the greatest index
strictly before `t` at which the record holds a commitment by the party in
force for `b` at that index, other than a checkpoint lapsed for its whole
scope (C2.10.4, C2b.4.1); `c(t) = 0` where there is none. The **gap is open**
at `t` exactly when `t − c(t)` exceeds the declared duration. The commitment
that sets `c(t)` need not carry `b` — a drop is C2b.5.2's — and need not be
valid: an invalid commitment is provable fault, not silence. Only a
commitment closes the interval, and a backing that declares no silence clause
has no gap. The transparent profile's challenge window is not read under the
pool (C2b.3c); the duration binds alone.

**C2b.3.1 The snapshot** for `b` at `t` is the last canonical finalized
checkpoint carrying `b`, witnessed strictly before `t`, by a party then in
force for `b`, whichever term of the chain it fell in (Construction §C2b.3,
C2.10.4), passing checkpoints lapsed for their whole scope. A carrying
checkpoint by the party then in force that the reader finds invalid blocks
the read, as it blocks C2.7's descent (C2.10.3): it is neither a snapshot
nor a licence to read an older one. A reader replays the snapshot ([pool-v2 §10](pool-v2.md#10-import-and-replay))
for its accepted-root forest, spent set, output set, totals and standing
demands. A reader that cannot replay it draws no verdict: unavailable history
is not an empty spent set and not an empty forest.

Every checkpoint's state accounts for the venue record up to one index, its
**adoption index**: for a checkpoint of a segment other than that segment's
opening checkpoint, the index at which the segment's opening checkpoint was
witnessed, since C2b.4.2 requires the segment's history to have adopted
everything with force up to there; for an opening checkpoint, the adoption
index of the checkpoint it imports for `b`, and `0` for genesis. The
**recovery state** of `b` at `t` is the snapshot's state with the effects of
every publication with force (C2b.3.2), witnessed after the snapshot's
adoption index and strictly before `t`, applied in venue order. This is what
"valid recovery settlements witnessed since that snapshot" (C2b.3a) reads:
a settlement adopted by no checkpoint yet still counts, so a note cannot
settle twice across two silences either.

## 5. Recovery at the venue

**C2b.3.2 Publications and their force.** Five kinds are published at the
backing's declared venue, each naming the backing: demand, acceptance,
release, withdrawal and request. A publication is judged at the index the
venue witnessed it, against the record strictly before that index, and
publications at one index are read in the venue's own order. An exact
republication is the same publication: it has force at the first index at
which it has any, and never a second time. Three kinds can have **force**,
changing the recovery state; the rest are evidence:

- A **demand** has force at `t` where the gap of its backing is open at `t`,
  its proof is bound to the snapshot's segment and scope root, every anchor is
  a root of the snapshot's forest, its deadline is strictly after `t`, and no
  nonzero tag is locked or spent in the recovery state at `t`. Its effect is
  C3.7's: it stands and its tags are locked in the recovery state. A demand
  already standing in the snapshot needs no publication.
- A **withdrawal** has force at `t` where the gap is open at `t`, its demand
  stands in the recovery state at `t`, and the presenter key signed it. Its
  effect is C3.7's.
- A **release** has force at `t` where the gap is open at `t`; its demand
  stands in the recovery state at `t`; its acceptance verifies under the
  backing's **K** with a deadline at or after `t`; the presenter key signed it
  over that demand, that acceptance and the settlement; the settlement is
  bound to the snapshot's segment and scope root, names the backing, the
  demand's quantity, the acceptance's `owner` and the demand; each anchor is a
  root of the snapshot's forest; the proof verifies; the tags match (C3.5);
  each nullifier is absent from the snapshot's spent set, shown by the
  non-membership proof at its `spentRoot` ([pool-v2 §11](pool-v2.md#11-the-spent-set)),
  and absent from every earlier settlement with force since the adoption
  index; and `cm_out` is absent from the snapshot's outputs and those earlier
  settlements'. Its effect is C3.5's.
- An **acceptance** and a **request** have no force; they are evidence for
  C3.8 and C2b.5.2.

A publication of nullifiers alone, a statement of any other kind, a spend or
receipt however co-signed, and anything that does not parse have no force and
adopt nothing (C2b.3b–c). A release whose gap is not open at its index — the
operator was still committing, or a commitment had already closed the gap —
has no force at that index; its holder publishes it again once the gap is
open, or resubmits it under the segment then serving. Force, once had, is
not revoked by a later commitment, and a duplicate does not confer it twice.

**C2b.3.3 What a holder needs, and what a stranger checks.** To redeem in a
gap a holder needs the snapshot's trail and ancestry, from replicas, to place
its anchors and prove non-membership; the backer's acceptance; and the venue.
A stranger judging the release needs the same trail and the venue record from
the adoption index on. The venue holds bytes it does not interpret; it is a
reader that decides force, and every reader decides it alike from the same
record. A note whose history nobody replicated waits, as Construction §C2b.3
says it will.

## 6. Return

**C2b.4.1 The return is a new segment.** The first commitment witnessed while
a backing's gap is open closes it, whoever is then in force (C2b.6.1). That
commitment must be the **opening checkpoint** of a new segment — an empty
local history at the segment's opening sequence (C2.10.9a's form) — whose
opening for each scoped backing is that backing's snapshot at the commitment's
index (C2.7.1, C2.10.4). What the returning operator co-signed after its last
witnessed commitment was never witnessed, and finality means witnessed
(Construction §C2b.4): that tail is discarded as a unit, and its receipts read
under C2.10.9b. A checkpoint witnessed while the gap of any backing in its
scope is open, other than a new segment's opening checkpoint, is **lapsed for
its whole scope**: it is held at its exact sequence, supplies no finalized
state for any scoped backing, closes no interval, and C2.7's descent passes it
with the declared clause and the record's own emptiness as the evidence of
that step, as it passes a whole-scope term lapse (C2.10.4). This is a public
condition every reader computes alike. The same-operator guard against an
elective scope change over a live tail (C2.10.9) does not apply to a return:
the gap is the operator's own doing, and the tail is not live.

**C2b.4.2 The segment adopts the gap before it serves.** Let `r` be the index
at which the segment's opening checkpoint was witnessed and `a` the adoption
index of the state it imported for a scoped backing. The segment's
**adopted block** is every publication with force naming a scoped backing,
witnessed after `a` and at or before `r`, in venue order. Its
statements are admitted as the segment's first local statements, at positions
`1 … k` in that order, before any other statement: an **adopted statement**
is one whose segment binding is the segment of the opening it was judged
against rather than this segment's, whose anchors that opening certifies, and
which the venue record shows with force at or before `r`. It passes every
other check of [pool-v2 §8](pool-v2.md#8-admission), takes a position and a
receipt naming the opening checkpoint, and has C3.5's or C3.7's effect, so the
segment's history replays the recovery state exactly. A
checkpoint of the segment witnessed after `r` whose history omits a statement
of the block, orders them otherwise, or admits any other statement before the
block is complete, is invalid, and the operator that signed it is at fault
(invariant 22). Publications witnessed at `r` itself are in the block: they
were judged strictly before `r`, when the gap was open (Construction §C2b.4),
and the operator reads the record through `r` after its opening lands and
before it co-signs anything. A settlement witnessed after `r` has no force
and is not adopted; its holder re-proves under the new segment. Adoption
inserts the backer's note with every consumed set, so `outstanding` is
unchanged by a return (invariants 10 and 12); nullifiers published without a
settlement are not adopted (C2b.3b). A demand standing at `r`, whether
imported from the snapshot or adopted from the gap, stands in the new segment
until it is settled or withdrawn there.

## 7. What E declares

Beside pool-v2 [§2](pool-v2.md#2-what-e-declares-and-what-the-configuration-fixes)'s
construction and configuration, a backing under this contract declares the
silence clause's no-commitment duration and the non-service duration, count
and window, each a witnessed-index quantity, inside its name. A backing that
declares no silence clause is never silent and has no gap; one that declares
no non-service terms has no count. The construction version that carries
these objects names them in its configuration; a v2 backing declaring a
clause binds nothing by it ([§7.4](pool-v2.md#74-redemption-and-what-this-version-does-not-carry)).

## 8. Choices made here, and what they cost

Under Construction [§C0a](construction.md#c0a-what-may-be-added), this contract
names what it adds, what it chose where the Construction left room, and what
it charges.

**It adds** three statement kinds (`demand`, `withdraw`, `settle`), one
relation family (the holding proof in two forms and the settlement), one hash
tag (`T_TAG`), five venue publication kinds, and the adoption index. It
retires nothing further: the challenge window left the shielded core under
C2b.3c, and every rule here reads objects Construction already names.

**It chose**, and each choice can be revisited at its number:

- C3.3: a demand names whole notes, at most `inputs` of them, and partial
  or larger presentation is consolidation by the wallet first. The
  alternative, a settlement with change, would need a second private output
  in the settlement and a demand that names a quantity below its notes' sum;
  it was refused for one relation and one reading of invariant 27.
- C3.4–C3.5: the acceptance's `owner` and the settlement's quantity and
  `cm_out` are public. Redemption is lit at the backer already (§C1.4); the
  alternative, a hidden output, would leave the operator unable to check that
  the settlement pays the backer and put that dispute off-protocol.
- C3.7: no attempt timeout in the single-operator core. The alternative,
  carrying the cross-operator profile's lock timeout, would add a clock
  nothing in the core reads.
- C2b.5.1: the request is segment-free. The alternative, a request bound to
  a segment, would die at every handover and reset the count that Construction
  §C2b.5 says a handover never resets.
- C2b.4.1: the return is always a new segment. The alternative, continuing
  the old segment with an adopted block in mid-history, needs an operator
  assertion of the index it adopted through, or a two-step adoption across
  commitments; the new segment makes the adopted block a public function of
  the record and the empty opening, at the price of a segment identity, a
  resynchronization and re-proof of anything submitted since the snapshot.
- C2b.3.1: the adoption index, so a settlement not yet adopted still counts
  against the next gap's holdings. The alternative, reading only the
  snapshot's own state, lets a note settle once per silence.

**It costs** what Construction §C2b already prices — illiquidity during a
silence, the discarded tail, re-proof under the new segment — and, on top: a
public tag per demanded note at filing, linkable to the note's later spend;
the holding proof and the settlement in the configuration, so a change to
either is a new version; a replayer that reads the venue record from a
checkpoint's adoption index to the next opening, since the adopted block is a
function of it; a wallet that keeps the snapshot's leaves to re-prove after a
return; and, for every reader, the requests to count and the publications to
judge, whose sizes the byte layouts will state.

**It excludes**, and a later document must specify before a construction
claims them: reliance sets (invariant 13's accompanying claims), payouts
paying in claims (Construction §C3's swap inside the settlement, §C1.7),
cross-operator presentation ([Extensions](extensions.md#cross-operator-presentation)),
the bridge between venues (C2.10.2), adoption across constructions, and
identified issuance.
