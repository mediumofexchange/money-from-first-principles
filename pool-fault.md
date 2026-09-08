# The shielded pool's authenticated faults

**Status: a proposal for the maintainer's selection, not an adopted rule.**
Nothing here changes `moe/pool/v2`, the authority contract or the recovery
contract until the maintainer selects a rule and the amendments in [§8](#8-the-amendments)
are applied to the normative documents. The executable oracle for every rule
below is the reference's fault model (`model/pool-fault.ts` and the test files
named in [§10](#10-what-the-maintainer-selects)); the reference's
[fault recovery document](https://github.com/mediumofexchange/reference-ts/blob/main/docs/POOL_FAULT_RECOVERY.md)
holds the evidence, the reader contract and the review findings this answers.

This proposes how a checkpoint that its own operator signed, and that fails
the rules on the evidence it committed, is read by the descent (C2.7,
C2.10.3–4), the snapshot (C2b.3.1), the count (C2b.5.2), the no-commitment
clock (C2b.6.1), the return (C2b.4.1–2) and the receipt verdicts
(C2.10.9a–b, C2b.4.3). It refines the [authority contract](pool-authority.md)
and the [recovery contract](pool-recovery.md) with four rules, C2.10.10–13,
one receipt rule, C2.10.9c, and one revised clock, C2b.6.1, for a
construction version that binds exact evidence into its history. Words are
pool-v2's and the two contracts'. "At a checkpoint's own record prefix" means
against earlier witnessed indices and, at its own index, only lower signed
sequences of the same operator (C2.10.4).

## 1. The failure this repairs

Under the contracts as written, a carrying checkpoint by the party in force
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
successor who faces the same blocked descent. The recovery contract records
the remedy as open ([§8](pool-recovery.md#8-choices-made-here-and-what-they-cost)).

The failure has a second half. In v2 the history binds each statement's
identity, roots and position, and the receipt binds the digests of the proof
and obligor signature the operator verified (pool-v2 [§9](pool-v2.md#9-the-ordered-history-the-snapshot-and-the-receipt)),
but nothing the operator signs binds those digests to the checkpoint. A trail
whose proof fails could be the operator's evidence or a replica's substitute,
and the two read alike (`model/pool-fault-evidence.test.ts`). So "the reader
finds invalid" is not yet a fact about the operator, and no rule may pass a
checkpoint on it. The first rule below makes it one.

## 2. Evidence

**C2.10.10 A checkpoint authenticates the exact evidence it commits.** In a
construction implementing this contract, the segment's history binds, for
every local statement, the digests of the exact evidence the operator verified
when it admitted the statement: `proofHash` and, for an issuance,
`signatureHash`, as pool-v2 §9's receipt already carries them, beside the
statement identity, roots and position in the history recurrence. A
commitment therefore commits, through each snapshot digest, to the exact
evidence of every event in its history. The **served trail** of a checkpoint
is the evidence whose digests its history binds. A proof or signature with
another digest at a position is not that checkpoint's evidence, whether it
verifies or not, and a trail that does not reproduce the signed snapshot
digests is not the checkpoint's trail.

C2.10.6 is unchanged: an event's identity is its segment identity, position
and statement identity, so two prefixes agree on common events by statement
identity, and the evidence of an event is fixed once, by the segment that
admitted it. Pool-v2 §8's idempotence is unchanged: an exact resubmission,
and a re-proof of an admitted statement with other valid bytes, return the
original receipt with the original digests, and the history carries the
original evidence. A receipt's inclusion already "binds its position,
statement identity and resulting history hash" (C2.10.9a); with this rule the
history hash binds the evidence too, so a receipt whose evidence digests
differ from the checkpoint's at its position is contradicted by the existing
comparison, and no new comparison is added.

What the rule buys is attribution. A reader holding a checkpoint's served
trail recomputes the history to the signed snapshot digest; if the trail
reproduces it and a proof in it fails, the operator signed for evidence that
fails, which is its provable fault (invariant 22). If the trail does not
reproduce the digest, the reader holds something other than the operator's
evidence and has learned nothing about the checkpoint.

## 3. Classification

**C2.10.11 Every held checkpoint is read in one of four classes, at its own
record prefix.** A reader classifies a checkpoint against the record strictly
before its witnessed index and the lower same-operator sequences at that
index, never against what was witnessed later or against the time the reader
obtained its evidence. The classes, in the order they are judged:

- **Lapsed.** The checkpoint is witnessed at or after a scoped term's end
  (C2.10.4), or is a non-opening checkpoint witnessed while a scoped
  backing's gap is open or at or after its segment's silence boundary
  (C2b.4.1). Lapse is read from the authenticated header and scope
  with the witnessed replacement chain and, for silence, the scope's own
  clock record. No event evidence is read, and lapse does not depend on
  validity.
- **Valid.** The reader holds the checkpoint's authenticated directory, scope
  and terms, its served trail (C2.10.10) and the evidence its imports and its
  descent require; every scoped term is in force at its index (C2.10.3); it
  extends its segment's last valid prefix (C2.10.12); its imports satisfy
  C2.10.5–7; where the construction implements recovery, its history begins
  with its adopted block (C2b.4.2); and replay under pool-v2 §8 admits every
  statement at its position, the proof verifying against the configuration's
  key for its kind and the obligor signature under **K** where one is
  required.
- **Excluded.** The checkpoint is not lapsed, its served trail reproduces its
  signed snapshot digests, and validity fails deterministically on that
  evidence: a proof or obligor signature that does not verify; a statement
  §8's admission would refuse at its position against the replayed state (a
  wrong domain, segment or scope root, a backing outside the scope, a spent
  nullifier, a duplicate or missing output, an uncertified anchor, a supply
  bound); a history that does not extend the segment's last valid prefix; an
  import conflict (C2.10.6) or an import of a prefix that is not finalized; a
  header the construction calls malformed (C2.10.1, C2b.6.1); an opening
  history that omits, reorders or precedes its adopted block (C2b.4.2); or a
  scope whose terms the signer did not hold at its index. Exclusion is a
  function of the checkpoint's bytes and the record before it: a fact
  witnessed later never turns a valid checkpoint into an excluded one.
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
prefix (C2.10.4, read with "valid" in place of "final"); the excluded
checkpoint is passed inside the segment as it is in the record. Three cases
follow. A process that lost its book and published a stale checkpoint
(C2.7.5's superseded twin) is excluded once, and the live process's next
checkpoint is valid without a new segment. An operator that committed bad
evidence and then commits the same statements with valid evidence finalizes
them, and its excluded checkpoint remains provable fault. An operator that
commits other statements at those positions contradicts the receipts it gave
(C2.10.9b), and answers for them.

*Alternative, R7: a fault ends the segment.* Every later checkpoint of the
segment is excluded, the operator refuses further service under it, and
repair is a new segment on the canonical state, by this operator or a
successor, in C2.10.9a's opening form. Its cost is that an honest twin's
single stale checkpoint forces every scoped backing into a new segment, with
resynchronization and re-proof, and the live tail's receipts read abandoned
at that new segment under C2.10.9b although no scope was changed electively.
Nothing final differs between the two: earlier valid prefixes and their
receipts are unaffected under both. Both are modelled
(`faultContinuesSegment`).

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
evidence C2.10.11 names. Another scope's history, terms or clock are never
read for `b`'s clock, so the dependency the review found (a non-carrying
commitment's silence lapse requiring that scope's fault evidence, recursively)
does not arise, and the residue of every operator-wide rule, that a live
operator can keep a dropped backing's gap shut with fresh valid commitments
carrying other backings, does not arise either.

It changes one sentence of Construction §C2b.6. A backing an operator drops
from its scope while committing the rest (C2.4.5) is reached by the
non-service grade first and, once the declared duration has passed since its
last valid carrying checkpoint, by this grade too: snapshot redemption opens
for it while the operator commits other backings. The rest of §C2b.6 is
unchanged and its reasoning is kept: the clock is the backing's, a handover
neither starts nor stops it, a successor inherits the remainder, an honest
heir is graded until its first commitment lands, and a clock that reset on an
event the rule-holder chooses would be cancellable by the rule-holder. A
valid checkpoint carrying `b` is the operator's own act of committing `b`'s
state, which nobody else can perform for it. The cost is the backer's: the
duration it declares now prices a drop as it prices darkness. The gain is the
holder's: a dropped backing whose replacement rule is inert or absent has a
remedy the count alone could not give.

With this clock, two sentences of the recovery contract are retired: "a
commitment carrying nothing for `b` closes `b`'s interval while adopting
nothing for it" (C2b.6.1) and "closing that interval does not restore a
segment retired by intervening silence" (C2b.6.1), which had nothing left to
guard. C2b.4.2's rule that a publication with force for a backing the
segment does not carry waits for the next segment that does, whose block
reaches back to that backing's adoption index, stands and does more work: a
publication for `b` has force while `b`'s gap is open, whoever else commits.
The silence boundary (C2b.4.1) is unchanged in form and simpler in evidence:
for a segment, every scoped backing's `c(t)` after the opening is one of the
segment's own valid checkpoints, so the boundary is that checkpoint's index
plus the duration plus one.

*Alternatives that retain the operator-wide clock.* Each keeps "any
commitment by the party in force for `b` closes `b`'s interval" and differs
in which commitments it excludes.

- *Term-only non-carrying resets.* A commitment carrying `b` resets only
  where valid; a commitment carrying nothing for `b` resets unless a scoped
  term had ended at its index, whether or not silence lapsed it for its own
  scope. Evidence: every commitment's directory, scope and terms; the
  classification evidence of the carrying ones. Cost: a continuation of a
  segment retired by silence, signed by the in-force operator, resets a
  dropped backing's clock, and term lapse and silence lapse are read
  differently in one rule. This is the reference's recommendation where the
  operator-wide clock is retained.
- *A″.* As term-only, but a non-carrying commitment lapsed by silence for its
  own scope resets nothing (C2b.6.1 as written, with excluded carrying
  commitments added). Cost: establishing that lapse needs that scope's clock,
  which can need its fault evidence, recursively across scopes and back in
  time; the reference's dependency table prices it.
- *A′.* An excluded carrying commitment still resets (C2b.6.1 as written).
  Cost: a stream of bad commitments keeps the gap shut forever, and the
  count with **E**'s replacement rule is the only remedy.

## 6. Receipts

**C2.10.9c An excluded checkpoint decides no receipt.** In C2.10.9a–b's walk
an excluded checkpoint is passed: it is neither an inclusion nor a
contradiction nor the first carrying transition, and it occupies its
sequence, so it is not a hole and does not make one. A receipt whose `after`
is excluded is read from its segment's last valid checkpoint below `after`,
as one whose `after` was lapsed is. Its statement is final where a valid
checkpoint of the segment includes its position, statement identity and
history hash; contradicted where one occupies the position otherwise, at or
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
   terms, the served trail and the imports, and the classification of every
   checkpoint its own descent passes;
4. under the revised C2b.6.1, nothing else. Under an operator-wide clock,
   also each non-carrying commitment's scope and terms, and under A″ its
   scope's clock record.

A verdict is a function of retained evidence and the record. A reader that
discards the evidence and keeps the verdict cannot re-derive it, cannot
resolve a dependent checkpoint that arrives later, and must not serve the
verdict to another reader as evidence; a cached verdict is not a certificate.
Retention is the holders' and the backer's, as it is for the trail (§C0b),
and replication remains the remedy for withheld evidence (C2.7.2).

A stranger shown one excluded checkpoint needs its signed commitment, the
directory proof for one scoped backing, the recurrence inputs from the failing
event to the committed length, and the event's exact evidence. Under a linear
history that is 168 bytes per later event, or 160 with the position derived,
plus the evidence (about 14.7 KB for a v2 proof); a tree over the history
would make it logarithmic. Which form pool-v3 fixes is its choice and does not
change these rules.

## 8. The amendments

Adopting the rules above changes the following sentences and no others; each
is applied by the maintainer, in the document named, when a rule is selected.

| Document, rule | Present | Proposed |
|---|---|---|
| Construction §C2b.6 | "The grade fires on the operator publishing nothing; one that drops a single backing while committing the rest on time is reached by the non-service grade instead." | "The grade fires on the operator not committing this backing; one that drops a single backing while committing the rest on time is reached by the non-service grade first, and by this grade once the duration has passed since the last commitment carrying it." (Only under the revised C2b.6.1.) |
| Construction §C2.10 | "A late checkpoint lapses for its entire scope; the authenticated scope and replacement evidence prove that step in C2.7's descent, whereas missing scope data proves nothing." | Add: "A checkpoint whose committed evidence fails the rules is excluded and passed on that evidence (C2.10.10–12); a checkpoint the reader cannot classify blocks." |
| Authority C2.10.3 | "An invalid candidate is not accepted; unavailable evidence is not an empty state." | "An excluded candidate is passed to the last valid one (C2.10.12); an unresolved one blocks; unavailable evidence is not an empty state." |
| Authority C2.10.4 | "Later checkpoints of the same segment extend its already final prefix" | "...extend its last valid prefix" and, in the lapse paragraph, "as it passes an excluded checkpoint (C2.10.12)"; the sentence "A wrong proof or inconsistent history under otherwise live terms is invalid data, not this public lapse condition or a license to choose an older state" becomes "A wrong proof or inconsistent history under live terms is C2.10.11's excluded class where the evidence is authenticated and unresolved otherwise; neither licenses choosing an older state, since descent reaches the last valid checkpoint." |
| Authority C2.10.9a–b | "Invalid or unavailable checkpoint evidence does not establish lapse." / "An invalid R proves neither lapse nor abandonment." | "An unresolved checkpoint establishes nothing; an excluded one is passed and is not the transition (C2.10.9c)." |
| Recovery C2b.3.1 | "A carrying checkpoint by the party then in force that the reader finds invalid blocks the read, as it blocks C2.7's descent (C2.10.3): it is neither a snapshot nor a licence to read an older one." | "An excluded carrying checkpoint (C2.10.11) is passed with its evidence, as a whole-scope lapse is; an unresolved one blocks the read. The snapshot is the last valid carrying checkpoint, never a state the reader chooses." |
| Recovery C2b.5.2 | "A carrying checkpoint by the party then in force that the reader finds invalid blocks the count as it blocks the snapshot (C2b.3.1): provable fault is not a clean count." | "The count reads against the snapshot (C2b.3.1), passing excluded checkpoints and refusing on unresolved ones: a provably faulting operator is counted against, not sheltered by its fault." |
| Recovery C2b.6.1 | The definition of `c(t)`; "The commitment that sets `c(t)` need not carry `b` … (C2b.4.1)."; "The commitment need not be valid: an invalid commitment is provable fault, not silence." | The revised C2b.6.1 of [§5](#5-the-clock); the two quoted passages are deleted. Under a retained operator-wide clock, only the last sentence changes: "An excluded commitment carrying `b` closes nothing." |
| Recovery C2b.4.1 | "It is lapsed for its whole scope: it is held at its exact sequence, supplies no finalized state for any scoped backing, closes no interval" | Add: "An excluded checkpoint of the segment (C2.10.12) is likewise held and closes no interval; it neither retires the segment nor moves its silence boundary." |
| Recovery §8 | The bullet recording the invalid-live-evidence remedy as open. | Replaced by a pointer to this contract. |
| Construction Appendix | — | Two retired sentences with their cost: an invalid commitment resets the clock (a stream of bad commitments suppresses redemption forever); a commitment carrying nothing for a backing closes its interval (a dropped backing has only the count, and its clock reads other scopes' evidence). |
| pool-v3 | — | The history recurrence carries `proofHash_i` and `signatureHash_i` (C2.10.10); the form of the history, chain or tree, and the certificate encoding. |

## 9. What this adds, replaces and costs

Under Construction [§C0a](construction.md#c0a-what-may-be-added):

**It adds** two digests per event to an existing recurrence (C2.10.10) and
one classification (C2.10.11) that every reader already performs in part,
since each already decides whether a checkpoint is final or lapsed. It adds
no frame, no publication kind, no party, no venue role and no primitive. The
excluded class is read exactly where the lapsed class is already read, so the
passing rule is one generalized mechanism, not a second one.

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

**It costs** 64 bytes per event in the history; a fault certificate that is
linear in the segment's later events under a chained history; readers that
retain evidence rather than verdicts; under the revised clock, a backer whose
declared duration prices a drop as darkness; under segment continuity, an
excluded checkpoint's tail that waits for the segment's next valid checkpoint
or a boundary rather than being decided at once.

**It leaves** the residue every intrinsic rule leaves: a checkpoint whose
preimage nobody serves is unresolved for every reader forever, exactly as
today, and no rule may pass it, since a reader cannot tell a withheld preimage
from an honest replica's loss and missing data must never authorize rollback.
Availability is a separate contract: either the venue admits a commitment only
with its data or under a declared availability requirement, or independent
replication is arranged and measured. Prospective fault publication (a
challenger publishes the failing evidence at the venue, and exclusion applies
to reads after that index) was refused as the primary mechanism because it
adds a frame and a watcher and repeats a proof-sized publication per bad
commitment, while readers holding the trail can already agree on the class;
it remains available as a way for a stranger to share a certificate.

**It is core.** Two readers that classify one record differently disagree
about a holding; every implementation must read these classes alike.

## 10. What the maintainer selects

1. **Authenticated exclusion** (C2.10.10–13, C2.10.9c) in place of blocking
   on a reader's finding of invalidity. The alternative is the present text
   with the reproduced failure standing.
2. **The clock:** the revised C2b.6.1, the clock is the snapshot's
   (recommended, and it changes one sentence of Construction §C2b.6); or
   term-only non-carrying resets, recommended where the operator-wide clock
   is kept; or A″.
3. **Segment continuity:** the segment continues from its last valid
   checkpoint (recommended); or R7, a fault ends the segment.
4. **Receipts:** an excluded checkpoint decides no receipt and the existing
   boundaries decide the tail (recommended); or permanent abandonment at the
   fault.

The model exercises each choice as a switch on one class: the default
candidate and its variants in `model/pool-fault.test.ts`,
`model/pool-fault-reader.test.ts`, `model/pool-fault-clock.test.ts`,
`model/pool-fault-dependency.test.ts` and `model/pool-fault-liability.test.ts`;
the revised clock and segment continuity, against the defaults, in
`model/pool-fault-alternatives.test.ts`; the v2 evidence boundary in
`model/pool-fault-evidence.test.ts`. Independent adversarial review of the
selected rules is owed before the amendments are applied; the questions that
review must answer are in the reference's fault recovery document.
