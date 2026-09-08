# The shielded pool's authenticated faults

**Normative contract, selected 2026-09-08 under the maintainer's approval
to adopt recommended decisions.** This contract refines the authority and
recovery contracts for a later construction version. It does not reinterpret
`moe/pool/v2` bytes, circuits, keys or receipts. The applied amendments are
indexed in [§8](#8-the-amendments); the selected choices and executable-model
limits are in [§10](#10-the-selected-rules).

This defines how a checkpoint that its own operator signed, and that fails
the rules on the evidence it committed, is read by the descent (C2.7,
C2.10.3–4), the snapshot (C2b.3.1), the count (C2b.5.2), the no-commitment
clock (C2b.6.1), the return (C2b.4.1–2) and the receipt verdicts
(C2.10.9a–b, C2b.4.3). It refines the [authority contract](pool-authority.md)
and the [recovery contract](pool-recovery.md) with four rules, C2.10.10–13,
one receipt rule, C2.10.9c, and one revised clock, C2b.6.1, for a
construction version that commits to its exact evidence. Words are pool-v2's
and the two contracts'. "At a checkpoint's own record prefix" means against
earlier witnessed indices and, at its own index, only lower signed sequences
of the same operator (C2.10.4).

## 1. The failure this repairs

Under the contracts before this amendment, a carrying checkpoint by the party in force
that a reader finds invalid blocks the descent, the snapshot and the count
(C2.10.3–4, C2b.3.1, C2b.5.2), while an invalid commitment resets the
no-commitment clock (C2b.6.1: "an invalid commitment is provable fault, not
silence"). The reference reproduces the consequence
(`model/pool-fault-boundary.test.ts`): one signed checkpoint whose copy of an
admitted statement carries a proof that does not verify makes every backing of
its scope unrecoverable at the venue, uncountable, and impossible for a
successor to take over, while the payment finalized before it stays final. A
recurring stream of such commitments keeps the clock reset; stopping opens the
gap but leaves the snapshot blocked. **E**'s replacement rule names a
successor who faces the same blocked descent. This contract supplies the remedy previously left open in the recovery contract.

The failure has a second half. In v2 the history binds each statement's
identity, roots and position, and the receipt binds the digests of the proof
and obligor signature the operator verified (pool-v2 [§9](pool-v2.md#9-the-ordered-history-the-snapshot-and-the-receipt)),
but nothing the operator signs binds those digests to the checkpoint. A trail
whose proof fails could be the operator's evidence or a replica's substitute,
and the two read alike (`model/pool-fault-evidence.test.ts`). So "the reader
finds invalid" is not yet a fact about the operator, and no rule may pass a
checkpoint on it. The first rule below makes it one.

## 2. Evidence

**C2.10.10 A checkpoint commits to the exact evidence it admitted.** In a
construction implementing this contract, the segment commits, beside its
history, to an **evidence chain** over the exact evidence the operator
verified when it admitted each statement:

```text
evidenceHash_0 = H(T_EVIDENCE ‖ segmentId)
evidenceHash_i = H(T_EVIDENCE ‖ evidenceHash_{i−1} ‖ statementHash_i ‖ proofHash_i ‖ signatureHash_i)
```

with `proofHash_i` and `signatureHash_i` as pool-v2 §9's receipt already
carries them, and the snapshot digest of every scoped backing binds
`evidenceHash_n` beside `historyHash_n`. The history hash and every root stay
functions of the statements alone, so pool-v2 [§7](pool-v2.md#7-statements)'s
identity ("two statements with one `statementHash` are one statement,
whatever their proof bytes"), C2.10.6's deduplication and pool-v2 §8's
idempotence are unchanged: an exact resubmission, and a re-proof of an
admitted statement with other valid bytes, return the original receipt with
the original digests. Every later valid checkpoint preserves the exact
evidence of its segment's last valid prefix as well as its statements.
Evidence at positions beyond that prefix may change after an excluded
checkpoint (C2.10.12); exclusion never finalized those positions. An honest
re-proof resubmission still returns the original receipt and admitted bytes.
A reader holding other valid proofs can recompute semantic state and statement
identity, but cannot establish canonical finality or evidence-bound receipt
inclusion without the authenticated admitted evidence.

The **served trail** of a checkpoint carries, for every event, the exact
bytes the operator verified, and a reader reproduces both chains from it. A
proof or signature with another digest at a position is not that checkpoint's
evidence, whether it verifies or not, and a trail that does not reproduce the
signed snapshot digests is not the checkpoint's trail. Receipt inclusion
under C2.10.9a–c compares its position, statement identity, resulting history
hash, **proof hash and signature hash** with the valid checkpoint's committed
event at that position. A mismatch in any of those fields is a contradiction
at an occupied position. This extends the receipt comparison without changing
statement identity. A repaired statement can become final while a receipt
attesting different evidence remains contradicted; earlier inclusion in a
valid checkpoint remains final under the existing precedence.

What the rule buys is attribution. A reader holding a checkpoint's served
trail recomputes the evidence chain to the signed snapshot digest; if the
trail reproduces it and a proof in it fails, the operator signed for evidence
that fails, which is its provable fault (invariant 22). If the trail does not
reproduce the digest, the reader holds something other than the operator's
evidence and has learned nothing about the checkpoint.

What it costs is the committed bytes. The classes of C2.10.11 are read from
the *committed* evidence, so a reader holding only other valid bytes for a
statement can verify semantic state and the receipt signature but cannot classify the
checkpoint: it is unresolved for that reader until the committed bytes are
served. Today a valid re-proof replays a trail whose original proof was lost;
under this rule it does not classify it. The exact admitted bytes therefore
become a retention obligation on the operator and its replicas, beside the
trail they already serve. The alternative, reading validity from any valid
proof a reader happens to hold, was refused: a reader holding valid bytes and
a reader holding the operator's failing committed bytes would classify one
checkpoint differently and disagree about the snapshot and about force, which
is the disagreement C2.10.11 exists to prevent.

## 3. Classification

**C2.10.11 Every held checkpoint is read in one of four classes, at its own
record prefix.** A reader classifies a checkpoint against the record strictly
before its witnessed index and the lower same-operator sequences at that
index, never against what was witnessed later or against the time the reader
obtained its evidence. The classes, in the order they are judged:

- **Lapsed.** The checkpoint is witnessed at or after a scoped term's end
  (C2.10.4), or is a non-opening checkpoint witnessed while a scoped
  backing's gap is open or at or after its segment's silence boundary
  (C2b.4.1). Term lapse reads only the authenticated header and scope with
  the witnessed replacement chain. Silence lapse reads the scope's clock
  record: under the revised C2b.6.1 that is the classification of every
  commitment carrying a scoped backing back to the last valid one
  (C2.10.13), so an unresolved carrying commitment leaves silence lapse
  unresolved. No non-carrying commitment resets this clock. Lapse is judged
  before validity, and that order decides what a read does, not attribution: a lapsed checkpoint
  whose served trail fails deterministically is still its operator's provable
  fault (invariant 22), though no read passes it on that ground.
- **Valid.** The reader holds the checkpoint's authenticated directory, scope
  and terms, its served trail (C2.10.10) and the evidence its imports and its
  descent require; every scoped term is in force at its index (C2.10.3); it
  extends its segment's last valid prefix (C2.10.12); its imports satisfy
  C2.10.5–7; where C2b.4.2 requires adoption, its history begins with its
  adopted block (the opening checkpoint itself remains empty under C2b.4.1);
  and replay under the construction's admission rules, including C2b.4.2's
  adopted-statement rule, admits every statement at its position, the
  committed proof verifying against the
  configuration's key for its kind and the committed obligor signature under
  **K** where one is required.
- **Excluded.** The checkpoint is not lapsed, its served trail reproduces its
  signed snapshot digests, and validity fails deterministically on that
  evidence: a committed proof or obligor signature that does not verify; a
  statement the applicable admission rules, including C2b.4.2's adopted-statement
  exception, would refuse at its position against the replayed
  state (a wrong domain, segment or scope root, a backing outside the scope,
  a spent nullifier, a duplicate or missing output, an uncertified anchor, a
  supply bound); a history that does not extend the segment's last valid
  prefix, including that prefix's committed evidence; an import conflict
  (C2.10.6), or an import naming a commitment the
  complete record (C2.10.13) does not hold, or one that is excluded or
  lapsed; a header the construction calls malformed (C2.10.1, C2b.6.1); a
  history that omits, reorders or precedes its adopted block where C2b.4.2
  requires it (excluding the empty opening checkpoint); or a scope naming a term not yet in force at its index or one
  the replacement chain does not hold. A checkpoint witnessed after a scoped
  term ended is lapsed, not excluded, and a checkpoint by a party never in
  force for a backing is not read for that backing at all (C2.7.1).
  Exclusion is a function of the checkpoint's bytes and the record before it:
  a fact witnessed later never turns a valid checkpoint into an excluded one.
- **Unresolved.** The reader lacks, or cannot authenticate, something either
  verdict needs: the trail or any part of it, the directory, the scope or its
  terms, an imported prefix or its evidence, a checkpoint its descent or
  clock must classify first, or the record range and same-index order
  (C2.10.13). A trail that does not reproduce the signed digests is
  unresolved, not excluded. So is a verifier the reader does not support, a
  resource failure and a programming failure: none is a certificate of the
  operator's fault. An unresolved checkpoint yields no verdict. It is not
  passed and not imported, it neither resets a clock nor fails to, and every
  read that depends on it refuses. Evidence obtained later resolves it at
  the same prefix and reverses nothing, because the classes are functions of
  the bytes and the record.

A valid verdict for a checkpoint requires every checkpoint its descent passes
and every prefix it imports to be classified, as valid, excluded or lapsed;
one unresolved dependency leaves the checkpoint unresolved. A reader that
reads an unresolved checkpoint as excluded rolls a finalized spend back and
lets its note settle again; one that reads it as valid reverses its own
verdict when the bytes arrive. The model keeps both as departures.

## 4. An excluded checkpoint

**C2.10.12 An excluded checkpoint is held, supplies nothing, and is passed.**
It remains held at its exact sequence (C2.3.4): the sequence is consumed, it
is no hole for C2.10.9a, and the operator cannot reuse it. It supplies no
finalized state for any scoped backing, no import target, no accepted root,
no carrying transition for a receipt (C2.10.9c) and no reset of any clock
(C2b.6.1). C2.7's descent (C2.7.1, C2.10.4), the snapshot (C2b.3.1), the
count (C2b.5.2), currency (C2.7.5) and the receipt walk (C2.10.9a–b) pass it
with its authenticated evidence, as they pass a checkpoint lapsed for its
whole scope, and as with a lapse, missing evidence proves no step: a
checkpoint the reader cannot classify is unresolved and blocks the read. What
descent reaches past an excluded checkpoint is the segment's or the backing's
last valid state, never a state the reader or the operator chooses.

**The segment continues from its last valid checkpoint.** A later checkpoint
of the same segment is valid where it extends the segment's last *valid*
prefix, including its committed evidence (C2.10.4, C2.10.10); the excluded
checkpoint is passed inside the segment as it is in the record. Three cases
follow. A process that lost its book and published a stale checkpoint
(C2.7.5's superseded twin) is excluded once, and the live process's next
checkpoint is valid without a new segment. An operator that committed bad
evidence and then commits the same statements with valid evidence finalizes
them, and its excluded checkpoint remains provable fault. A receipt naming
other evidence at those repaired positions is still contradicted (C2.10.10). An operator that
commits other statements at those positions contradicts the receipts it gave
(C2.10.9b), and answers for them. Each case holds where the segment's next
valid checkpoint is witnessed before a scoped backing's gap opens (C2b.4.1):
an excluded checkpoint resets no clock, so the gap is measured from the last
valid one, and a segment that does not recommit within the duration is
retired by silence and returns as a new segment under the selected rule. The
honest process that commits at its interval is inside that bound; a stale
checkpoint no longer buys it time, which is the point of exclusion.

## 5. The clock

**C2b.6.1, revised: the clock is the snapshot's.** For backing `b` at index
`t`, `c(t)` is the index of `b`'s snapshot at `t` (C2b.3.1): the last valid
checkpoint carrying `b`, by a party then in force for `b`, witnessed strictly
before `t`, passing checkpoints lapsed for their whole scope and excluded
checkpoints; `c(t) = 0` where there is none. The gap is open at `t` exactly
when `t − c(t)` exceeds the declared duration. Only a valid checkpoint
carrying `b` closes `b`'s interval: a commitment carrying nothing for `b`
closes nothing for `b`, and an excluded or lapsed one closes nothing for
anyone. The rest of C2b.6.1 stands: one duration per scope, a backing that
declares no clause has no gap, only a commitment closes the interval.

This makes the clock and the snapshot one read over one set of evidence: the
operator's held commitments before `t` with their directories, to
authenticate what each carries (C2.4.2), and for those carrying `b`, the
evidence C2.10.11 names. No non-carrying checkpoint needs classification
merely to reset `b`'s clock. Carrying checkpoints still require their entire
scope, clock dependencies, imports and transitive ancestry; the rule removes
the additional dependency on non-carrying resets, and the residue of every
operator-wide rule, that a live
operator can keep a dropped backing's gap shut with fresh valid commitments
carrying other backings, does not arise either.

The clock inherits the snapshot's dependency as well as its evidence. Where a
commitment carrying `b` is unresolved for a reader, `b`'s gap is unresolved
for that reader, and with it the silence lapse of `b`'s receipts (C2b.4.3)
and the silence boundary of `b`'s segment (C2b.4.1), whereas an operator-wide
clock answers "closed" from a later commitment carrying other backings
without reading the unresolved one. That answer is determinate only because
the rule lets a commitment carrying nothing for `b` speak for `b`; under the
revised clock the reader refuses instead, as it already does for the
snapshot. The clock therefore adds no evidence a redemption did not already
need, but it does make the gap verdict wait on the same evidence.

It changes two sentences of Construction §C2b.6, the second and the last of
its first paragraph. The grade runs from the last valid checkpoint carrying
the backing rather than from the last commitment by the party in force, and
only such a checkpoint closes it; and a backing an operator drops from its
scope while committing the rest (C2.4.5) is reached by the non-service grade
first and, once the declared duration has passed since its last valid
carrying checkpoint, by this grade too: snapshot redemption opens for it
while the operator commits other backings. The rest of §C2b.6 is unchanged
and its reasoning is kept: the clock is the backing's, a handover neither
starts nor stops it, a successor inherits the remainder, an honest heir is
graded until its first commitment lands, and a clock that reset on an event
the rule-holder chooses would be cancellable by the rule-holder. A valid
checkpoint carrying `b` is the operator's own act of committing `b`'s state,
which nobody else can perform for it. The cost is the backer's: the duration
it declares now prices a drop as it prices darkness. The gain is the holder's:
a dropped backing whose replacement rule is inert or absent has a remedy the
count alone could not give.

With this clock, two sentences of the recovery contract are retired: "a
commitment carrying nothing for `b` closes `b`'s interval while adopting
nothing for it" (C2b.6.1) and "closing that interval does not restore a
segment retired by intervening silence" (C2b.6.1), which had nothing left to
guard. C2b.4.2's rule that a publication with force for a backing the
segment does not carry waits for the next segment that does, whose block
reaches back to that backing's adoption index, stands and does more work: a
publication for `b` has force while `b`'s gap is open, whoever else commits.
The silence boundary (C2b.4.1) is unchanged in form: every scoped backing's
`c(t)` is a valid checkpoint carrying it by the party then in force, the
segment's own while the segment carries it, and the boundary follows from
those checkpoints and the duration; the reader still establishes the
complete range of the operator's commitments (C2.10.13), and retains every carrying checkpoint's classification dependencies.

## 6. Receipts

**C2.10.9c An excluded checkpoint decides no receipt.** In C2.10.9a–b's walk
an excluded checkpoint is passed: it is neither an inclusion nor a
contradiction nor the first carrying transition, and it occupies its
sequence, so it is not a hole and does not make one. A receipt whose `after`
is excluded is read from its segment's last valid checkpoint below `after`,
as one whose `after` was lapsed is. The receipt is final where a valid
checkpoint of the segment includes its position, statement identity,
history hash and evidence hashes (C2.10.10); contradicted where one occupies the position otherwise, at or
below `after`, or omits it above a held `after`; and otherwise pending until
a boundary decides it: a proven repair (C2.10.9a), an elective carrying scope
change (C2.10.9b), a term end (C2.10.9) or the silence boundary (C2b.4.3).
The precedence is unchanged: final, then contradicted, then abandoned, then
lapsed, then pending.

Fault alone decides nothing for a receipt. It is provable against the
operator's key (invariant 22), but it is not the payee's inclusion and it is
not an excuse. *Alternative refused:* abandoning every unfinished receipt of
the segment at the excluded checkpoint. Its cost is that an honest twin's
stale checkpoint abandons the live tail the operator was about to commit, and
it adds a boundary and a precedence rule where the existing four boundaries
already decide every tail; a receipt is pending indefinitely only where no
valid checkpoint, repair, scope change, term end or silence ever follows,
which is a live segment that never commits again, and that segment's
silence boundary arrives after one duration.

## 7. What a reader establishes

**C2.10.13 No verdict without the complete range.** Before drawing a verdict
about `b` at `t`, a reader establishes:

1. the complete set of commitments the venue holds for the party in force for
   `b` over the range the read spans, with their witnessed indices and their
   same-index sequence order (C2.3.4), under the venue's finality rule. For
   the snapshot and the clock the range runs from the last valid carrying
   checkpoint to `t`, and a reader that cannot show the range complete has
   not found the last one. A venue read that is not complete for the range
   is unresolved;
2. for every commitment in the range, its authenticated directory, to read
   whether it carries `b` (C2.4.2);
3. for every commitment carrying `b`, the evidence C2.10.11 names: scope and
   terms, the served trail with its committed evidence, the imports, and the
   classification of every checkpoint its own descent passes;
4. no classification of a non-carrying checkpoint solely for the clock.
   The dependencies of the carrying checkpoints in item 3 still apply.

A verdict is a function of retained evidence and the record. A reader that
discards the evidence and keeps the verdict cannot re-derive it, cannot
resolve a dependent checkpoint that arrives later, and must not serve the
verdict to another reader as evidence; a cached verdict is not a certificate.
Retention is the holders' and the backer's, as it is for the trail (§C0b),
and replication remains the remedy for withheld evidence (C2.7.2).

A stranger shown one excluded checkpoint needs its signed commitment, the
directory proof for one scoped backing with the segment identity and the
totals the snapshot digest binds, the terminal `historyHash_n` and the
evidence chain's inputs from the failing event to the committed length (96
bytes per later event: `statementHash`, `proofHash`, `signatureHash`), and
the failing statement's public inputs with its exact evidence (about 14.7 KB
for a v2 proof). A tree over the evidence chain would make the suffix
logarithmic. Which form pool-v3 fixes is its choice and does not change these
rules.

## 8. The amendments

The amendment checklist below records the changes applied to Construction,
authority and recovery. The pool-v3 row remains future layout work. Historical
alternatives are retained at the [proposal revision](https://github.com/mediumofexchange/money-from-first-principles/blob/a6edadc/pool-fault.md).

| Document, rule | Previous | Adopted |
|---|---|---|
| Construction §C2b.6 | "It opens snapshot redemption and runs from the last commitment witnessed from a party then in force for this backing until commitments resume, and only a commitment closes it." | "It opens snapshot redemption and runs from the last valid checkpoint carrying this backing, witnessed from a party then in force for it, until such a checkpoint is witnessed again, and only such a checkpoint closes it." |
| Construction §C2b.6 | "The grade fires on the operator publishing nothing; one that drops a single backing while committing the rest on time is reached by the non-service grade instead." | "The grade fires on the operator not committing this backing; one that drops a single backing while committing the rest on time is reached by the non-service grade first, and by this grade once the duration has passed since the last valid commitment carrying it." |
| Construction §C2.10 | "A late checkpoint lapses for its entire scope; the authenticated scope and replacement evidence prove that step in C2.7's descent, whereas missing scope data proves nothing." | Add: "A checkpoint whose committed evidence fails the rules is excluded and passed on that evidence (C2.10.10–12); a checkpoint the reader cannot classify blocks." |
| Authority C2.10.3 | "An invalid candidate is not accepted; unavailable evidence is not an empty state." | "An excluded candidate is passed to the last valid one (C2.10.12); an unresolved one blocks; unavailable evidence is not an empty state." |
| Authority C2.10.4 | "Later checkpoints of the same segment extend its already final prefix" | "...extend its last valid prefix" and, in the lapse paragraph, "as it passes an excluded checkpoint (C2.10.12)"; the sentence "A wrong proof or inconsistent history under otherwise live terms is invalid data, not this public lapse condition or a license to choose an older state" becomes "A wrong proof or inconsistent history under live terms is C2.10.11's excluded class where the evidence is authenticated and unresolved otherwise; neither licenses choosing an older state, since descent reaches the last valid checkpoint." |
| Authority C2.10.6, pool-v2 §7 and §10 | "different valid proof bytes may attest the same statement"; "Two statements with one `statementHash` are one statement, whatever their proof bytes." | Both stand, for identity, deduplication and admission. Add to pool-v2 §10's successor: "Classification under C2.10.11 reads the committed evidence (C2.10.10); a trail served with other bytes reproduces the state and is unresolved for finality." |
| Authority C2.10.9a–b | "Invalid or unavailable checkpoint evidence does not establish lapse." / "An invalid R proves neither lapse nor abandonment." | "An unresolved checkpoint establishes nothing; an excluded one is passed and is not the transition (C2.10.9c)." |
| Authority C2.10.9a–c | Receipt inclusion compares position, statement and history. | Include the committed proof and signature hashes in the comparison (C2.10.10). Preserve evidence of the last valid prefix, while excluded positions may be repaired. |
| Recovery C2b.3.1 | "A carrying checkpoint by the party then in force that the reader finds invalid blocks the read, as it blocks C2.7's descent (C2.10.3): it is neither a snapshot nor a licence to read an older one." | "An excluded carrying checkpoint (C2.10.11) is passed with its evidence, as a whole-scope lapse is; an unresolved one blocks the read. The snapshot is the last valid carrying checkpoint, never a state the reader chooses." |
| Recovery C2b.5.2 | "A carrying checkpoint by the party then in force that the reader finds invalid blocks the count as it blocks the snapshot (C2b.3.1): provable fault is not a clean count." | "The count reads against the snapshot (C2b.3.1), passing excluded checkpoints and refusing on unresolved ones: a provably faulting operator is counted against, not sheltered by its fault." |
| Recovery C2b.6.1 | The definition of `c(t)`; "The commitment that sets `c(t)` need not carry `b` … (C2b.4.1)."; "The commitment need not be valid: an invalid commitment is provable fault, not silence." | The C2b.6.1 of [§5](#5-the-clock); the two quoted passages are deleted. |
| Recovery C2b.4.1 | "It is lapsed for its whole scope: it is held at its exact sequence, supplies no finalized state for any scoped backing, closes no interval" | Add: "An excluded checkpoint of the segment (C2.10.12) is likewise held and closes no interval; it neither retires the segment nor moves its silence boundary." |
| Recovery §8 | The bullet recording the invalid-live-evidence remedy as open. | Replaced by a pointer to this contract. |
| Construction Appendix | — | Two retired sentences with their cost: an invalid commitment resets the clock (a stream of bad commitments suppresses redemption forever); a commitment carrying nothing for a backing closes its interval (a dropped backing has only the count, and its clock reads other scopes' evidence). |
| pool-v3 | — | The evidence chain and `T_EVIDENCE`, `evidenceHash_n` in the snapshot digest, the served trail's exact evidence (C2.10.10); the form of the chains and the certificate encoding. |

## 9. What this adds, replaces and costs

Under Construction [§C0a](construction.md#c0a-what-may-be-added):

**It adds** one chain over evidence digests the receipt already carries
(C2.10.10), bound by the existing snapshot digest, and one classification
(C2.10.11) that every reader already performs in part, since each already
decides whether a checkpoint is final or lapsed. The chain is one mechanism
for one property, attribution, and leaves identity to the history hash; the
excluded class is read exactly where the lapsed class is already read, so the
passing rule is one generalized mechanism, not a second one. It adds no
frame, no publication kind, no party, no venue role and no primitive.

**It replaces** the three sentences that let a reader's finding of invalidity
block a read, with a class that authenticated evidence establishes and a
class that its absence establishes; the sentence that let an invalid
commitment reset the clock; and, under the revised C2b.6.1, the non-carrying
reset and the two sentences that qualified it. It retires the recovery
contract's open question.

**It derives** from the law and invariant 22. A checkpoint that fails its own
rules on the evidence its operator signed is that operator's provable fault;
the law forbids a fault from reducing what a holder holds, and passing the
fault to the last valid state is what preserves the holder's finalized
holdings and opens the remedies. The revised clock derives from §C2b.3's own
definition of the snapshot as the last commitment whose directory carries the
backing: the clock reads the same object.

**It costs** 32 bytes in the snapshot digest and 96 bytes per event in the
evidence chain; a fault certificate that is linear in the segment's later
events under a chained history; the exact admitted bytes as a retention
obligation on the operator and its replicas, since a valid re-proof no longer
classifies a checkpoint whose original evidence was lost; readers that retain
evidence rather than verdicts; under the revised clock, a backer whose
declared duration prices a drop as darkness, and a gap verdict that waits on
the same evidence as the snapshot rather than being answered by a commitment
carrying other backings; under segment continuity, an excluded checkpoint's
tail that waits for the segment's next valid checkpoint or a boundary rather
than being decided at once, and that is saved only where that checkpoint
lands before the gap opens.

**It leaves** the residue every intrinsic rule leaves: a checkpoint whose
committed evidence nobody serves is unresolved for every reader, as the
snapshot is today when a trail is withheld, and under the revised clock the
gap is unresolved with it; no rule may pass it, since a reader cannot tell a
withheld preimage from an honest replica's loss and missing data must never
authorize rollback. Availability is a separate contract: either the venue
admits a commitment only with its data or under a declared availability
requirement, or independent replication is arranged and measured. Prospective
fault publication (a challenger publishes the failing evidence at the venue,
and exclusion applies to reads after that index) was refused as the primary
mechanism because it adds a frame and a watcher and repeats a proof-sized
publication per bad commitment, while readers holding the required evidence can already
agree on the class; it remains available as a way for a stranger to share a
certificate.

**It is core.** Two readers that classify one record differently disagree
about a holding; every implementation must read these classes alike.

## 10. The selected rules

The maintainer authorized recommended decisions on 2026-09-08. The selection is:

1. Authenticated exclusion (C2.10.10–13, C2.10.9c).
2. The snapshot clock (C2b.6.1): a dropped backing's declared duration prices
   the drop as it prices darkness; carrying evidence is needed for the verdict.
3. Segment continuity from the last valid prefix (C2.10.12), preserving its
   committed evidence and permitting repairs only beyond it.
4. Existing receipt precedence and boundaries; an excluded checkpoint decides
   no receipt. Inclusion additionally compares the committed evidence hashes.

The refused alternatives cost either indefinite suppression of a dropped
backing's redemption (operator-wide clocks), mandatory new segments after an
honest twin's stale checkpoint (R7), or premature abandonment of the live tail
(fault as a receipt boundary). The earlier proposal and its counterexamples
remain in Git history and the reference's model tests.

The reference's `model/pool-fault.ts` models the selected classification,
snapshot clock, segment continuity and receipt boundaries using ideal evidence.
It is not the production claim layer. The separate evidence chain, exact-byte
receipt comparison and compact certificate encoding need executable coverage
before a later construction's byte layout is fixed. Runtime stays pinned to v2.
