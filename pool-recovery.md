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
C2b.6.1, C2b.3.1–3 and C2b.4.1–3. Words are pool-v2's: note, commitment,
nullifier, anchor, accepted-root forest, segment, scope, checkpoint, history,
finalized prefix, import. "Judged at an index" means judged against the
record strictly before that index (Construction §C2b.4). The **lag** is the
venue's ([§C2.3.5](construction.md#c23-witnessing)): an act signed now is
witnessed no earlier than the latest witnessed index plus the lag.

## 1. Objects

**C3.1 A claim's public name is its tag.** `tag = H(T_TAG, nf)`: the
construction's in-circuit hash, under a tag distinct from every other, of the
note's immutable nullifier. This is Construction §C3's `H(nullifier)`. A tag
appears in a notice; the nullifier appears only in a spend or a settlement.
Whoever later sees the nullifier can compute the tag, so a demand, its
withdrawal, its settlement and any request over the note are linkable to each
other and to the note's eventual spend. That is [§C1.5](construction.md#c15-what-still-leaks-and-what-a-wallet-does-about-it)'s
aborted-presentation leak: a wallet spends a withdrawn note to a fresh one
before using it again.

**C3.2 The holding proof** is one relation with two public forms. Over the
construction's `inputs` note positions, all of one backing, it proves for each
position `i`: `secret_i ≠ 0`, `owner_i = H(T_OWNER, secret_i)`,
`nf_i = H(T_NULLIFIER, domain, cm_i, secret_i)`; where `value_i > 0`, that a
path carries `cm_i` to the public `anchor_i` and `tag_i = H(T_TAG, nf_i)`;
where `value_i = 0`, that the position is padding, `tag_i = 0` and
`anchor_i = 0`, so that no public input of this relation is one the relation
does not read. Every note
names the public `backing`, at least one value is positive, every value is
below `2^64`, and the positions' nullifiers are distinct, so one note fills
one position: two positions over one note would double the quantity the
demand claims and leave it unsettleable, a dishonour the holder manufactured
against itself. The proof consumes nothing and creates nothing.

- The **segment-bound form** is a demand's. Its public inputs are the domain,
  the segment identity, the scope root, the backing, the `quantity` (the sum
  of the values, in 128-bit arithmetic), the anchors, the tags, and the
  notice's own remaining fields (C3.3): the presenter key, the instant and
  the deadline. The relation reads none of the three for its own conclusion
  and constrains all three anyway: the presenter key's limbs are
  range-checked as every identifier's are
  ([pool-v2 §1](pool-v2.md#1-fields-hashes-and-encodings)), and the instant
  and the deadline are below `2^64`. **A relation of this contract constrains
  every public input it declares that no signature binds**, so a proof binds
  it under any proof system rather than only under one that carries unread
  inputs into its verification equation: a demand whose presenter key a relay
  could rebind is a demand that relay could release. A demand and a request
  carry no signature, so every public input of theirs enters a constraint; a
  withdrawal and a settlement are signed over their own statement's identity,
  which binds every field of it, so a settlement's padding position keeps a
  spend's free anchor ([pool-v2 §7.2](pool-v2.md#72-spend)). Scope membership
  is proven as in a spend.
- The **segment-free form** is a request's (C2b.5.1). Its public inputs are
  the domain, the backing, one anchor, one tag and, last, the refresh value
  of C2b.5.1: no segment, no scope root and no quantity. It proves one real
  note without revealing its size.

Which forest an anchor must belong to is the reader's rule: an operator reads
it against its segment's accepted-root forest (C3.7), a venue reader against
the snapshot's (C2b.3.2), a non-service reader against the canonical state it
counts at (C2b.5.2). A proof is not a holding until a reader has placed its
anchor.

**C3.3 The demand** is a notice and the segment-bound holding proof for it.
The notice names the backing, the quantity, the tags (the nonzero tags of the
proof, in position order), the **presenter key**, the evaluation instant and
the deadline. The presenter key is the holder's choice, fresh per demand as a
receiver's secret is fresh per payment (pool-v2 [§3](pool-v2.md#3-notes)),
and signs the release and the withdrawal; the proof binds the notice, so
nothing else signs the demand. The **instant** is a witnessed index no later
than the latest index witnessed when the notice was signed (invariant 24),
checked by every reader as a window on the index `w` at which the demand is
witnessed, or would first be: `w − 2·lag ≤ instant ≤ w − lag`. A demand whose
instant falls outside that window is refused at the door and has no force at
the venue. The **deadline** is a witnessed index strictly ahead of `w`
(Construction §C3): it bounds the lock (C3.7) and marks when non-payment
becomes public (C3.8), and the holder who bears the lock chooses it. A demand
names at most `inputs` claims: a holder presenting more, or part of a note,
first spends to itself ([§C1.2](construction.md#c12-the-shielded-pool),
packing). The notice is the proof's public inputs, so the demand's identity
is the statement's own identity, its `statementHash`
(pool-v2 [§7](pool-v2.md#7-statements)), and the notice is neither hashed
nor signed apart from it: one object, one identity, and no second hash in
the circuit.

**C3.4 The acceptance** is the obligor **K**'s strict signature over the demand
identity, an `owner` value and the acceptance deadline. **K** generates the
spend secret behind `owner` and never reveals it (invariant 25); `owner` is
public. The acceptance deadline is a witnessed index at or before the demand's
deadline, or the object is not an acceptance. Its identity is the hash of
those fields. The acceptance need not be published: it is carried by the
release (C3.6). A backer publishes it at the venue where it wants the
evidence that it answered, and it is evidence of an answer only where its
deadline is later than the index it was witnessed at by more than the lag,
since no release could otherwise be witnessed inside it (C3.8).

**C3.5 The settlement statement** is a statement of kind `settle`. Its public
inputs are the domain, the segment identity, the scope root, the backing, the
quantity, the `owner`, the output's `rho_out`, one anchor and one nullifier
per input position, one output commitment `cm_out`, and the demand identity.
The proof establishes, for each input, what a spend establishes
([pool-v2 §7.2](pool-v2.md#72-spend)): ownership, the immutable nullifier,
membership at the anchor where the value
is positive, padding where it is zero, and scope membership; that every input
and the output name the public backing; that the sum of input values equals
the quantity and the output's value, in 128-bit arithmetic; and that
`cm_out = H(T_NOTE, domain, backing, quantity, owner, rho_out)` with `rho_out`
and `cm_out` nonzero and the positions' nullifiers distinct, as the holding
proof requires them (C3.2). The reader checks in the clear that
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
came from beyond the anchors. `rho_out` is public so that the backer rebuilds
the note's opening from the record alone: every other field of that opening is
public already, so publishing it discloses nothing the lit settlement did not,
and it removes the delivery a private `rho_out` would need from the party the
settlement is a remedy against.

**C3.6 The release and the withdrawal.** The release is the presenter key's
strict signature over the demand identity, the acceptance identity and the
settlement's `statementHash`; it is served with the acceptance and the
settlement statement, and with nothing else: every reader that decides its
force replays the snapshot already (C2b.3.3), so it holds the spent set and
checks absence itself, and no non-membership proof is carried.

The withdrawal is a statement of kind `withdraw` carrying no proof, whose
public inputs are the domain, the segment identity, the scope root and the
demand identity, authorized by the demand's presenter key signing that
statement's `statementHash`. So a withdrawal is signed for one segment as a
demand is proven for one: a withdrawal an operator kept back cannot be
applied under a segment the holder never named, and withdrawing the same
demand under a later segment is another statement needing a fresh signature,
which costs no proof.

Each admitted kind carries one **authorization** field beside its statement
and its proof, generalizing the obligor signature an issue carries: empty for
a demand, the presenter's signature for a withdrawal, and for a settlement
the acceptance's deadline with **K**'s signature over the acceptance and the
presenter's signature over the release. The construction fixes its bytes.
`signatureHash` (C2.10.10) is over that field exactly as admitted, and the
zero digest where it is empty, so a receipt and the evidence chain name the
exact authorization the operator verified. The acceptance is bound to the
settlement through that field alone, since C3.5's public inputs carry the
demand and the `owner` but not the acceptance's identity. Each is effective
when witnessed:
under service, by admission into the segment's history; in a gap, by the
venue (C2b.3.2). An exact resubmission or republication is the same act and
returns the same answer (invariant 26).

**C2b.5.1 The request** is the segment-free holding proof over one note. No
key signs it; the proof is the holder's act. It is published at the backing's
declared venue naming the backing. It is framed and identified as a statement
of its own kind is, over its own public inputs — the domain, the backing, the
anchor, the tag and a **refresh value** the holder chooses — and without the
segment and scope prefix every admitted kind carries (pool-v2 §7), since a
request is never admitted to a history. Two requests with one identity are
one request whatever their proof bytes, as two statements with one identity
are one statement. The relation reads the refresh value no more than a demand
reads its presenter key, and constrains it as a field element; because the
proof binds it, only a party that can produce the proof — the note's holder —
can mint another identity for one tag, while anybody can copy the request
that exists. The relation range-checks it below `2^64` as it does a demand's
instant and deadline, so it is a public input the relation reads and no other
value carries one proof. A wallet **derives** it, as it derives every other
blinding value (invariant 26, [pool-v2 §3](pool-v2.md#3-notes)): from its own
root secret, the note's nullifier and a refresh counter it advances only to
file again, so a wallet rebuilt after a crash refiles the same request rather
than a second one, and its requests over different notes are not linked by a
shared sequence. It is Construction §C2b.5's non-service
object and §C3's demand shape without the backer: a real note, its size
hidden, waiting to be served.

## 2. Presentation under service

**C3.7 The standing-demand record and the pending-lock set are the segment's.**
A segment's history holds statements of kind `demand`, `withdraw` and `settle`
beside issue, spend and burn, in one order, and the history hash binds them
(pool-v2 [§9](pool-v2.md#9-the-ordered-history-the-snapshot-and-the-receipt)).
A commitment therefore commits to the standing demands and the locks
(invariant 23), and a replayer recomputes both by applying those statements
in the history's order. Admission reads one committed view
([pool-v2 §8](pool-v2.md#8-admission)):

- A **demand** is admitted where its proof verifies under this segment and
  scope root, the anchor of every position whose tag is nonzero is in the
  accepted-root forest — a padding position's anchor is `0` (C3.2) and names
  no history — the backing is in
  the scope and its terms are held, its instant and deadline pass C3.3 and
  C3.8, its nonzero tags are distinct, and no nonzero tag is **locked** or
  **spent**. A tag is spent where it
  is the tag of a nullifier in the spent set; the operator keeps that set of
  tags beside the spent set, and a replayer recomputes it. Its effect: each
  nonzero tag enters the pending-lock set under this demand's identity, and
  the demand is standing. Nothing moves.
- A lock **stands** at an index while its demand's deadline is at or after
  that index, and reserves nothing after it: the deadline is the holder's own
  bound on its lock (Construction §C3: a demand outlives its locks, and the
  protection against stalling is one the backer cannot wait out). A tag
  carries one lock for each demand standing over it, since a demand whose
  lock no longer stands does not leave the record and a later demand may
  name the same tag; the tag is **locked** while any of those locks stands,
  and discharging a demand releases that demand's locks and no other's. A
  demand
  leaves the standing-demand record only by its settlement or its withdrawal
  (Construction §C3: the claims are committed against payment until
  withdrawal or settlement), so the record is a function of the history, and
  in a gap of the history with the publications that had force (C2b.3.2). A
  demand past its deadline stays in it, reserving nothing, and stands in the
  history as evidence (C3.8); it can no longer be settled,
  because no settlement of it passes the door after its deadline — the
  acceptance's deadline is at or before the demand's (C3.4) and the door
  refuses an acceptance deadline behind its horizon (C3.8) — and not because
  a reader removed it. A settlement admitted before the deadline may be
  witnessed by a checkpoint long after it, and a replayer that pruned the
  record at the index it read would refuse that settlement and exclude an
  honest checkpoint (C2.10.11); so no replayer removes a demand for its
  deadline. The record is the price: an unanswered demand nobody withdraws
  is carried by every import, handover and return for the backing's life,
  and a holder may found another over the same note once its lock stops
  standing. The index a lock is read at is the one the reader judges at: the
  door's horizon (C3.8), a checkpoint's witnessed index in replay, a
  publication's own index at the venue.
- A **spend** or **burn** whose nullifier's tag is under a standing lock is
  refused: a lock reserves without consuming, and only the settlement of its
  own demand consumes what it reserves.
- A **settle** statement is admitted where its proof verifies under this
  segment, its anchors are in the forest, its demand is standing, its backing
  and quantity are the demand's, its tags match (C3.5), the acceptance
  verifies under the backing's **K** with the settlement's `owner` and a
  deadline not behind (C3.8), the release verifies under the presenter key,
  and its nullifiers and output are new. Its effect is C3.5's, and the
  demand's locks are released with it.
- A **withdrawal** is admitted where its demand is in the standing-demand
  record, past its deadline or not, and the demand's presenter key signed
  that withdraw statement (C3.6). Its effect: the demand is discharged and
  its locks are released.
- A demand whose tags are under another standing lock, a settlement of a
  demand not standing, and a settlement whose own segment binding is not
  this segment's are refused — except a statement of the adopted block,
  which keeps the binding it was judged under (C2b.4.2). A demand this
  segment imported or adopted stands here and is settled here, by a
  settlement bound to this segment and naming that demand's own identity,
  which the demand keeps: the acceptance still verifies over it, and only
  the release, which signs the settlement, is signed afresh.
  **Imports** apply the closure's demand, withdrawal and settlement events in
  the ancestry order the import edges fix (C2.10.5); two prefixes with no
  ancestry relation whose events lock, settle or spend
  one tag are a conflict the import refuses, as a duplicate nullifier is
  (C2.10.6), so the lock set is a function of the closure and not of the
  order a replayer happened to merge it in.

The core presentation is single-phase: one operator serves the whole set
(Construction §C3), so there is no attempt timeout here; the demand's own
deadline bounds its lock. Cross-operator presentation, with its prepare,
decision venue and lock timeout, remains the [Extensions profile](extensions.md#cross-operator-presentation).

**C3.8 Deadlines at the door, and dishonour.** The operator reads every
index-relative condition against its **horizon**: the index at which an act
it co-signs now would first be witnessed, its latest read index plus the lag.
It refuses a demand whose deadline is not strictly ahead of the horizon or
whose instant is outside C3.3's window at it, and a settlement whose
acceptance deadline is behind it. These refusals keep each party to its own
terms; they are not supply rules, and a replayer does not re-judge an
admitted statement's instant or deadlines, since the parties consented to
them and the lock's own bound (C3.7) is read at the checkpoint's index.
Dishonour is Construction §C3's branch read from the history and the venue
record: a demand past its deadline with no settlement is the backer's visible
failure, unless an acceptance of it, witnessed before its own deadline by
more than the lag (C3.4), stood unreleased, which is the holder's lapse. A
demand one of whose tags is the tag of a nullifier spent **otherwise than by
that demand's own settlement**, in any history or settlement with force the
reader holds, is **voided by its holder** from the index that spend was
witnessed at — for a spend admitted into a history, the index at which the
earliest canonical checkpoint whose history holds it was witnessed; for one
with force at the venue, the publication's own index; and a reader that
cannot place it draws no verdict over the interval (C2.10.13) — since only
the note's holder can sign a spend of it, and
Construction §C3 makes spending a demanded note the holder's own void. From
that index it is neither the backer's failure nor settleable. It stood
unanswered at the earlier indices and is read there as it stood, which is
[Extensions](extensions.md#the-trigger)'s latch: a void is prospective, so a
holder that takes its liquidity back does not erase the dishonour already
recorded. No reader re-judges the index at which a door read its lock.

## 3. Non-service

**C2b.5.2 The count.** **E** declares the non-service duration `d`, the count
`m` and the window `W`. At index `t` a reader counts, for backing `b`,
against the **canonical state** at `t`: the state of the last canonical
finalized checkpoint carrying `b`, witnessed strictly before `t`, by a party
then in force for `b` (C2.10.4). It counts the distinct tags of requests
naming `b` — the backing the request's proof names, which must be the backing
the publication names — witnessed at indices in `[t − W, t − d]` that are
**valid** against that state, the proof verifying and the anchor a root of
the state's forest, and **unserved** at it: the tag is neither the tag of a
nullifier in its spent set nor under a lock standing at `t` (C3.7). A count
of at least `m` fires the grade against the party in force for `b` at `t`. A
request is unsigned, so anyone can copy it; a request's index is therefore
the first at which a request of its identity (C2b.5.1) was witnessed naming
`b`, and a republication by anybody, with these or other proof bytes, is
that same request and extends no window. A holder whose request is ageing out
of the window files another under a fresh refresh value before it does, which
only that holder can prove and which an operator that is stalling cannot
prevent, since it needs no new root and no service. A tag requested twice
counts once, however many identities it carries. A request whose proof fails
or whose anchor the state does not certify counts for nothing; a request the
canonical state has spent, or locked under a demand not yet past its
deadline, has been served, whether by this party or its predecessor. A
handover neither resets nor moves the count: the successor inherits the
standing requests and clears them by serving them, and an operator that drops
a backing from its scope while committing the rest is reached here, read
against the last checkpoint that carried the backing (Construction §C2b.5,
§C2.4.5). The count reads against the snapshot (C2b.3.1), passing excluded
checkpoints and refusing on unresolved ones: authenticated fault does not
shelter the operator from the count. The cost stands as declared: one holder
can split a holding into `m` notes and file `m` requests, and the pool hides
the sizes that would weight them.

## 4. Silence and the snapshot

**C2b.6.1 The no-commitment clock.** **E** declares the no-commitment
duration. Every backing of one scope declares the same duration, or none:
a segment header whose scope mixes durations, or clause with no clause, is
malformed, so a scope has one interval as it has one clock (C2.10.2) and a
backing that declared no silence clause is never lapsed by a sibling's. For
backing `b` at index `t`, let `c(t)` be the index of its snapshot at `t`
(C2b.3.1): the last valid checkpoint carrying `b`, by a party then in force
for `b`, witnessed strictly before `t`, passing whole-scope lapses and
excluded checkpoints (C2.10.11–13); `c(t) = 0` where there is none. The
**gap is open** at `t` exactly when `t − c(t)` exceeds the declared duration.
Only a valid checkpoint carrying `b` closes its interval. A non-carrying,
excluded or lapsed checkpoint closes nothing for `b`; an unresolved carrying
checkpoint leaves the read unresolved. The clock uses the snapshot's evidence,
including its classification dependencies, with no extra classification of
non-carrying checkpoints. A backing declaring no silence clause has no gap.
The transparent profile's challenge window is not read under the pool
(C2b.3c); the duration binds alone.

**C2b.3.1 The snapshot** for `b` at `t` is the last valid carrying checkpoint by a
party then in force for `b`, witnessed strictly before `t`, whichever term of
the chain it fell in (Construction §C2b.3, C2.10.4), passing checkpoints
lapsed for their whole scope and excluded checkpoints. An excluded carrying
checkpoint (C2.10.11) is passed with its authenticated evidence; an unresolved
one blocks the read. The snapshot is the last valid carrying checkpoint,
never a state the reader chooses. A reader replays the snapshot
([pool-v2 §10](pool-v2.md#10-import-and-replay))
for its accepted-root forest, spent set, output set, totals and standing
demands. A reader that cannot replay it draws no verdict: unavailable history
is not an empty spent set and not an empty forest.

Every checkpoint's state accounts for the venue record up to one index, its
**adoption index**: for a checkpoint of a segment other than that segment's
opening checkpoint, the index at which the segment's opening checkpoint was
witnessed, since C2b.4.2 requires the segment's history to have adopted
everything with force for its scoped backings up to there; for an opening
checkpoint, the adoption index of the checkpoint it imports for `b`, and `0`
for genesis. The **recovery state** of `b` at `t` is the snapshot's state
with the effects of every publication with force for `b` (C2b.3.2), witnessed
after the snapshot's adoption index and strictly before `t`, applied in venue
order. This is what "valid recovery settlements witnessed since that snapshot"
(C2b.3a) reads: a settlement adopted by no checkpoint yet still counts, so a
note cannot settle twice across two silences either.

## 5. Recovery at the venue

**C2b.3.2 Publications and their force.** Five kinds are published at the
backing's declared venue, each naming a backing: demand, acceptance, release,
withdrawal and request. The name the venue carries is routing; a publication
is read **for the backing its statement names as a public input** — the
demand's or settlement's `backing`, a withdrawal's demand's, a request's, an
acceptance's demand's — and a publication whose venue name differs from it
has no force and is no evidence. An acceptance and a withdrawal name their
demand and no backing of their own (C3.4, C3.6), so their routing name is
checkable only beside that demand: a reader deciding a withdrawal's force
holds it, since the demand stands in the snapshot or had force in the gap,
and an acceptance is evidence for C3.8 only to a reader that holds the demand
it names. A mis-routed acceptance is therefore noise such a reader ignores
rather than a claim it must refute. A publication is judged at the index the
venue witnessed it, against the record strictly before that index, and
publications at one index are read in the venue's own order. An exact
republication is the same publication: it has force at the first index at
which it has any, and never a second time.
Three kinds can have **force**, changing the recovery state; the rest are
evidence:

- A **demand** has force at `t` where the gap of its backing is open at `t`,
  its proof is bound to the snapshot's segment and scope root, the anchor of
  every position whose tag is nonzero is a root of the snapshot's forest,
  its instant is within C3.3's window at `t`,
  its deadline is strictly after `t`, and no nonzero tag is under a standing
  lock or spent in the recovery state at `t`. Its effect is C3.7's: it stands
  and its tags are locked in the recovery state. A demand already standing in
  the snapshot needs no publication.
- A **withdrawal** has force at `t` where the gap is open at `t`, its
  statement is bound to the snapshot's segment and scope root, its demand is
  in the recovery state's standing-demand record at `t`, and that demand's
  presenter key signed the statement (C3.6). Its effect is C3.7's.
- A **release** has force at `t` where the gap is open at `t`; its demand
  stands in the recovery state at `t` and is not past its deadline; its
  acceptance verifies under the backing's **K** with a deadline at or after
  `t`; the presenter key signed it over that demand, that acceptance and the
  settlement; the settlement is bound to the snapshot's segment and scope
  root, names the backing, the demand's quantity, the acceptance's `owner` and
  the demand; each anchor is a root of the snapshot's forest; the proof
  verifies; the tags match (C3.5); each nullifier is absent from the
  snapshot's spent set, which the reader holds from replaying the snapshot
  (C2b.3.3) and checks directly, no proof object being published with the
  release, and absent from every earlier settlement with force since the
  adoption index; and `cm_out` is absent from
  the snapshot's outputs and those earlier settlements'. Its effect is C3.5's.
- An **acceptance** and a **request** have no force; they are evidence for
  C3.8 and C2b.5.2.

A publication of nullifiers alone, a statement of any other kind, a spend or
receipt however co-signed, and anything that does not parse have no force and
adopt nothing (C2b.3b–c). A release whose gap is not open at its index — the
operator was still committing, or a commitment had already closed the gap —
has no force at that index; its holder presents again once the gap is open,
with a fresh demand since C3.3's window dates the old one, or resubmits under
the segment then serving. Force, once had, is not revoked by a later
commitment, and a duplicate does not confer it twice.

**C2b.3.3 What a holder needs, and what a stranger checks.** To redeem in a
gap a holder needs the snapshot's trail and ancestry, from replicas, to place
its anchors and to establish its nullifiers' absence; the backer's acceptance;
and the venue. A stranger judging the release needs the same trail and the
venue record from the adoption index on. Every reader of force replays the
snapshot for its forest, spent set and standing demands, so all three
conditions are read from one replayed state and a non-membership proof
published beside the release would restate what its reader already holds.
The accumulator still supports such a proof, as Construction §C1.2 and
invariant 23 require; this contract publishes none. The venue holds bytes it
does not interpret and decides no force: force is a reader's verdict, and
every reader draws it alike from the same record. A note whose history nobody
replicated waits, as Construction §C2b.3 says it will.

## 6. Return

**C2b.4.1 The return is a new segment.** Return to service after silence
requires the **opening checkpoint** of a new segment — an empty local history
at the segment's opening sequence (C2.10.9a's form) — whose opening for each scoped
backing is that backing's snapshot at the commitment's index (C2.7.1,
C2.10.4), witnessed by whoever is then in force (C2b.6.1). What the
returning operator co-signed after its last witnessed commitment was never
witnessed, and finality means witnessed (Construction §C2b.4): that tail is
discarded as a unit, and its receipts read under C2b.4.3 and C2.10.9b.
For a segment whose opening checkpoint was witnessed at `r`, its **silence
boundary** is the first index `g` strictly after `r` at which a backing in
its scope has an open gap. A later clock reset does not remove that boundary.
A non-opening checkpoint witnessed at `t` lapses if a scoped backing's gap
is open at `t`, or if the segment has a silence boundary `g < t`. It is
**lapsed for its whole scope**: it is held at its exact sequence, supplies no
finalized state for any scoped backing, closes no interval, and C2.7's
descent passes it with the authenticated scope, opening index, declared
clause and clock record proving the lapse, as it passes a whole-scope term
lapse (C2.10.4). The reader must establish the relevant record intervals and
their clock resets; missing evidence is unresolved, not proof that no gap
occurred. Validation reads the candidate's original record prefix. A later
gap cannot lapse an earlier finalized prefix. An excluded checkpoint of the
segment (C2.10.12) is likewise held and closes no interval; it neither
retires the segment nor moves its silence boundary.

A gap at `r` does not itself retire the fresh opening: its adopted block
covers publications through `r` (C2b.4.2). A non-opening checkpoint at `r`
still lapses if that gap is open, irrespective of same-index sequence order.
At or after a proven silence boundary the old segment cannot serve again;
only a new segment can return, even if there were no recovery publications.
Service checks the earliest witnessing horizon, but a predicted future gap
does not authorize discard or receipt lapse: those require the boundary at
or before the reader's witnessed index. The same-operator guard against an
elective scope change over a live tail
(C2.10.9) does not apply to a return: the gap is the operator's own doing,
and the tail is not live once that boundary is proven. This generalizes the
former current-index lapse test and preserves the fixed opening adoption
index; the clock is now the snapshot's (C2b.6.1).

**C2b.4.2 The segment adopts the gap before it serves.** Let `r` be the index
at which the segment's opening checkpoint was witnessed and `a` the adoption
index of the state it imported for a scoped backing. The segment's
**adopted block** is every publication with force for a scoped backing,
witnessed after `a` and at or before `r`, in venue order. Its statements are
admitted as the segment's first local statements, at positions `1 … k` in
that order, before any other statement. An **adopted statement** is one whose
segment binding is the segment of the opening it was judged against rather
than this segment's, whose anchors that opening certifies, and which the
venue record shows with force at or before `r`. It was judged once, under
C2b.3.2 at its own index, and is not judged again: the segment reads no door
condition (C3.8) and no index condition against `r`, reads its locks at the
publication's own index (C3.7), and checks only that it is the next statement
of the block and that its nullifiers and output are new in the segment's
state, which the block's order guarantees. It takes a position and a receipt
naming the opening checkpoint, and has C3.5's or C3.7's effect, so the
segment's history replays the recovery state exactly. A checkpoint of the
segment binds each adopted event's exact publication proof and signature bytes
in its evidence chain (C2.10.10); the new receipt retains their digests.
A checkpoint of the segment witnessed after `r` whose history omits a statement of the block,
orders them otherwise, or admits any other statement before the block is
complete, is invalid, and the operator that signed it is at fault (invariant
22). Publications witnessed at `r` itself are in the block: they were judged
strictly before `r`, when the gap was open (Construction §C2b.4), and the
operator reads the record through `r` after its opening lands and before it
co-signs anything. A settlement witnessed after `r` has no force and is not
adopted; its holder re-proves under the new segment. A publication with force
for a backing the segment does not carry waits for the next segment that
does, whose block reaches back to that backing's adoption index. Adoption
inserts the backer's note with every consumed set, so `outstanding` is
unchanged by a return (invariants 10 and 12); nullifiers published without a
settlement are not adopted (C2b.3b). A demand standing at `r`, whether
imported from the snapshot or adopted from the gap, stands in the new segment
until it is settled or withdrawn there, reserving nothing past its deadline
(C3.7).

**C2b.4.3 Receipts at the silence boundary.** Under this recovery contract,
C2.10.9b's receipt walk ends at the earlier of the segment's silence boundary
and its earliest scoped term end. The opening and the complete scope must
be authenticated; no receipt signing time is inferred from `after`. A held,
unheld or post-boundary reference does not move the silence boundary.

Read canonical inclusion, contradiction, repair and the first carrying
transition strictly before that boundary, with C2.10.9a/b's sequence and
precedence rules. Earlier inclusion remains final, an earlier contradiction
remains evidence, and an earlier abandonment is not excused by later silence.
Otherwise the receipt is **lapsed** at the silence boundary, even when no
new carrying opening follows. Checkpoints at or after the boundary neither
include nor contradict the receipt. A supplied repair checkpoint at or after
the boundary is not a C2.10.9a repair of that live tail; use this boundary
instead. Missing clock or interval evidence supplies no new verdict and
does not erase an inclusion or contradiction already proven. A backing
without a silence clause retains C2.10.9b unchanged.

This adds no signed object or privileged transfer. It extends the existing
whole-scope receipt lapse to the same public silence boundary that ends
continuation, rather than leaving a permanently unfinalizable tail pending
or inferring the time at which its receipt was signed.

## 7. What E declares

Beside pool-v2 [§2](pool-v2.md#2-what-e-declares-and-what-the-configuration-fixes)'s
construction and configuration, a backing under this contract declares the
silence clause's no-commitment duration and the non-service duration, count
and window, each a witnessed-index quantity, inside its name. A backing that
declares no silence clause is never silent and has no gap; one that declares
no non-service terms has no count. Backings served in one scope declare one
no-commitment duration (C2b.6.1). The construction version that carries
these objects names them in its configuration; a v2 backing declaring a
clause binds nothing by it ([§7.4](pool-v2.md#74-redemption-and-what-this-version-does-not-carry)).

## 8. Choices made here, and what they cost

Under Construction [§C0a](construction.md#c0a-what-may-be-added), this contract
names what it adds, what it chose where the Construction left room, and what
it charges.

**It adds** three statement kinds (`demand`, `withdraw`, `settle`), one
relation family (the holding proof in two forms and the settlement), one hash
tag (`T_TAG`), one per-kind **authorization** field generalizing pool-v2's
obligor-signature slot (C3.6), five venue publication kinds, and the adoption
index. It
retires nothing further: the challenge window left the shielded core under
C2b.3c, and every rule here reads objects Construction already names.

**It chose**, and each choice can be revisited at its number:

- C3.3: a demand names whole notes, at most `inputs` of them, and partial
  or larger presentation is consolidation by the wallet first. The
  alternative, a settlement with change, would need a second private output
  in the settlement and a demand that names a quantity below its notes' sum;
  it was refused for one relation and one reading of invariant 27.
- C3.3: the instant is checked as a window of one lag below `w − lag`, the
  latest index a signer could have seen, rather than trusted; and the
  presenter key is fresh per demand.
- C3.4–C3.5: the acceptance's `owner` and the settlement's quantity and
  `cm_out` are public. Redemption is lit at the backer already (§C1.4); the
  alternative, a hidden output, would leave the operator unable to check that
  the settlement pays the backer and put that dispute off-protocol.
- C3.7: no attempt timeout in the single-operator core; the demand's own
  deadline bounds its lock, so a refusing operator cannot freeze a note past
  the term its holder chose. The alternative, carrying the cross-operator
  profile's lock timeout, would add a clock nothing in the core reads.
- C2b.5.1: the request is segment-free. The alternative, a request bound to
  a segment, would die at every handover and reset the count that Construction
  §C2b.5 says a handover never resets.
- C2b.6.1: one no-commitment duration per scope. The alternative, mixed
  clauses in one scope, would let one backing's silence lapse a sibling that
  promised no clause, and the whole-scope lapse would fail to fail in
  compartments.
- C2b.4.1: the return is always a new segment. The alternative, continuing
  the old segment with an adopted block in mid-history, needs an operator
  assertion of the index it adopted through, or a two-step adoption across
  commitments; the new segment makes the adopted block a public function of
  the record and the empty opening, at the price of a segment identity, a
  resynchronization and re-proof of anything submitted since the snapshot.
- C2b.4.1/3: intervening silence retires continuation and lapses its unfinished
  receipts even after an unrelated clock reset. This replaces the insufficient
  current-index-only test; it preserves earlier finality and liability. The
  cost is authenticated interval evidence and whole-scope reopening and
  re-proof even where nobody published a recovery act. A moving adoption
  boundary would need new ordering and proof rules; merely checking for a
  release would omit demands and withdrawals with force.
- C2b.3.1: the adoption index, so a settlement not yet adopted still counts
  against the next gap's holdings. The alternative, reading only the
  snapshot's own state, lets a note settle once per silence.
- C2b.3.1, C2b.5.2, C2b.6.1: the [fault contract](pool-fault.md)
  resolves the former blocked-descent remedy: authenticated excluded
  checkpoints are passed, unresolved ones block, and the clock reads the
  snapshot. Exact evidence retention and classification dependencies remain
  required; a dropped backing's duration prices the drop as darkness.

Decided on 2026-09-09, before the layouts that instantiate them, each from
the same rule that the layout work read back:

- C3.2, C3.3: the notice is the proof's public inputs and the demand's
  identity is its `statementHash`. The alternative, a notice hash as one
  public input, adds a hash inside the circuit and gives one object two
  identities that a reader must check agree.
- C3.2: the positions' nullifiers are distinct in the relation. The
  alternative, leaving it to the door, admits a proof over one note in two
  positions at any reader that checks tags only for locks, and the demand it
  makes can never settle.
- C3.5: `rho_out` is public. The alternative, a private `rho_out`, adds a
  delivery from the holder to the backer, for the one field of a note whose
  every other field the settlement publishes, at the moment the two parties
  are least able to rely on each other.
- C3.6: the withdrawal is a `withdraw` statement its presenter signs by
  identity. The alternative, a signature over the demand identity alone, is
  bytes an operator can hold back and apply under a segment the holder never
  named; dropping the segment from the statement instead would make one
  admitted kind unbound to the segment that admits it.
- C3.6, C2b.3.2, C2b.3.3: the release publishes no non-membership proof.
  Every reader that decides force replays the snapshot for its forest and
  standing demands, so it holds the spent set; a proof at `spentRoot` would
  serve only a reader that judges force without replaying, and no such
  reader exists. Binding `spentRoot` in the snapshot digest to make one
  possible was refused for the same reason. This removes two proofs from
  every release — about 1.2 KB where the spent set holds 10⁵ nullifiers, and
  growing with its logarithm — and the accumulator's non-membership
  capability stands unchanged for any later compact certificate.
- C3.7: only the settlement and the withdrawal discharge a demand. The
  alternative, discharge at the deadline, makes the standing record depend
  on the index each reader judges at, and excludes an honest checkpoint that
  witnesses a settlement admitted before the deadline after it. The price is
  that an unsettled demand nobody withdraws, and the locks it holds, are
  retained for the backing's life.
- C3.8: a spent tag voids its demand, from the index the spend was witnessed
  at, in whatever history or venue record holds it. The alternative,
  re-judging at replay the index a door read a lock at, cannot be decided
  from the history and protects nobody, since only the holder can sign the
  spend.
- C2b.3.2: an acceptance is read beside the demand it names. The
  alternative, a backing field inside the signed acceptance, spends 32
  signed bytes on a routing name no reader can use without the demand.
- C2b.5.1, C2b.5.2: a request is read at the first index it was witnessed
  at, and carries a holder-chosen refresh value so that the holder can file
  another. The alternative, reading each publication at its own index, lets
  a stranger's copy carry a holder's request into a fresh window and fire a
  grade the holder did not press. Refreshing by re-proving the same note
  under another root was refused: an operator that stalls admits nothing, so
  no later root appears, and the only other roots are ones the note's own
  leaf already sat under — none at all for a note appended by the last
  statement admitted, which is the holder most in need of the grade.
  The void is prospective: reading a voided demand as never unanswered would
  let a holder erase a recorded dishonour by taking its liquidity back —
  which §C1.5 tells it to do — and would contradict the latch
  [Extensions](extensions.md#the-trigger) reads.

**It costs** what Construction §C2b already prices — illiquidity during a
silence, the discarded tail, re-proof under the new segment — and, on top: a
public tag per demanded note at filing, linkable to the note's later spend;
a public tag and anchor per non-service request, which a holder cannot
remove by respending while the operator refuses to serve, so a request names
the anonymity set of one root's leaves and marks one note in it for as long
as the refusal lasts; the holding proof and the settlement in the
configuration, so a change to either is a new version; a replayer that reads
the venue record from a checkpoint's adoption index to the next opening,
since the adopted block is a function of it; a wallet that keeps the
snapshot's leaves to re-prove after a return; a standing-demand record that
holds every demand until it is settled or withdrawn, with one lock per tag
it named, so a holder that walks away from an unanswered demand leaves those
entries in every replayer's state for the backing's life, carried by every
import, handover and return (C3.7); and, for every reader, the requests to
count and the publications to judge, whose sizes the byte layouts will
state.

**It excludes**, and a later document must specify before a construction
claims them: reliance sets (invariant 13's accompanying claims), payouts
paying in claims (Construction §C3's swap inside the settlement, §C1.7),
cross-operator presentation ([Extensions](extensions.md#cross-operator-presentation)),
the bridge between venues (C2.10.2), adoption across constructions, and
identified issuance.
