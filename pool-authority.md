# The shielded pool across operator replacement

This is the normative authority and history contract for Construction §C1.2
and §C2. It repairs the boundary identified in the reference's
[review](https://github.com/mediumofexchange/reference-ts/blob/aa72c09/docs/POOL_SEQUENCING_BOUNDARY.md).
The maintainer approved preserving private spends and independent backing
replacement on 2026-09-06. This contract requires a new construction version;
it does not change the bytes, keys or interpretation of existing `moe/pool/v1`
notes. [pool-v2.md](pool-v2.md) fixes the replacement-capable construction
bit for bit and records its pinned circuits and keys; the objects neither
document carries are listed in its §7.4. The [fault contract](pool-fault.md)
amends classification, continuity and evidence-bound receipts for a later
version; existing v2 bytes and their pinned interpretation remain unchanged.

## 1. Immutable claims and current authority

**C1.2.1 The claim's domain is immutable; its operator is not its domain.**
The construction configuration fixes the note/statement relations, hashes,
proof system, circuit identities and bounds. Its hash is the claim domain.
It contains neither a current operator nor a service segment. A backing's E
names this immutable configuration and its original operator separately.
Replacement follows E's witnessed chain; it changes neither configuration,
backing name, note commitment nor nullifier. Changing the construction still
requires a successor backing and the holder's participation (C1.6).

A note commits to `(domain, backing, quantity, owner, randomness)`; its
nullifier binds `(domain, note commitment, spend secret)`. A change of
operator, segment, anchor, tree position or membership path never changes
that nullifier. Hashes and integer conservation retain C1.2's requirements.

**C1.2.2 A proof covers current authority without naming its backing.** A
service scope is a public, sorted set of `(backing name, replacement link)`
entries for one operator and immutable construction domain. Each term is
derived from the witnessed replacement chain. The scope has a canonical
authenticated root under the construction's existing in-circuit hash.
The proof privately establishes scope membership for every input and output
backing, including zero-valued inputs and outputs. Each input proves ownership
and its immutable nullifier; each positive input proves note membership at an
accepted anchor. Conservation is per hidden backing in bounded integer
arithmetic. Issue/burn retain their lit backing and quantity and issuance
retains K's signature, now also bound to this statement's service segment.

A spend binds publicly the immutable domain, segment identity, scope root,
input anchors, nullifiers and output commitments. Input anchors may differ;
each must be an accepted root even where its input is padding. Padding has no
public flag and proves no note membership. The scope root is checked against
the actual scope by the host; a root merely asserted by an operator is not
authority. A previous scope's proof is not a proof for a new scope. No party
other than the holder gains a debit path by selecting the scope.

## 2. Segments and complete commitments

**C2.10.1 A segment fixes one scope.** Its authenticated header names the
construction domain, venue, operator, the full scope with term links, and the
exact opening commitment for every backing (or genesis where C2.7.3 permits
it). Its identity also distinguishes successive segments by the operator's
monotone signed commitment sequence. A changed scope, changed term, or reset
to a different opening requires a new segment. An excluded stale checkpoint
does not itself end a segment (C2.10.12). A returning key receives no
exception. Backings may choose operators independently and an operator need
not obtain a departed backing's consent to serve its remaining scope.

**C2.10.2 One clock per scope.** All scoped backings declare the same venue,
and imports described here are on that venue. Pools group by operator,
construction domain and venue. No global venue is prescribed. Moving an
operator on a backing's existing venue is supported; a venue change needs a
separate, specified bridge between the two records and is not authorized by
an operator-replacement record. Indices of different venues are never
compared. Atomic spends across different operators or venues require C1.7's
separate mechanism; a scope proof cannot stand in for it.

**C2.10.3 An opaque segment is committed in full.** A signed commitment
authenticates the segment header, its ordered local statements and the roots
and totals obtained by replay, through its directory's snapshot digests.
Every backing in the scope is in that directory and commits to the same
segment state. The checkpoint is final only if every scoped term is in force
at its witnessed index and it extends the record-derived prior state of every
scoped backing. Selective directory carriage does not finalize any part of
the segment, even if some of its statements happen to involve only the
remaining backings. A stale scope cannot be made current by dropping names.

The complete directory is still the public carriage evidence. A name present
with missing or invalid history is not an absence proof and does not authorize
falling back to a convenient earlier state. C2.7's exact descent fixes the
candidate before replay checks it. An excluded candidate is passed to the last
valid one (C2.10.12); an unresolved one blocks; unavailable evidence is not an
empty state. A commitment whose scope has lost a term cannot be used to finalize
its retained backings' tail either.

**C2.10.4 Continuity is per backing, event storage is shared.** For a segment's
first checkpoint, each backing's opening state is C2.7's state from the record;
for continued service it is the latest carrying state, including where the
same operator changes scope without a replacement. Later checkpoints of the
same segment extend its last valid prefix, including its committed evidence
(C2.10.10), without changing any earlier statement or opening state. Every
valid checkpoint becomes the new state for
all its scoped backings together. A stale process cannot omit a newer state
by presenting a valid older one. Same-key reappointment imports the state of
the intervening term, not the state that key remembers.

The predecessor read is relative to the child checkpoint: consider earlier
witnessed indices and, at the child's own index, only checkpoints of the same
operator with lower signed sequences. Exclude the child itself and later
same-index sequences. All eligible carrying checkpoints are ordered this way
before selecting and replaying the candidate; discovering a later checkpoint
does not retroactively change the child's predecessor.

An authenticated checkpoint witnessed after any scoped term ends is **lapsed
for its whole scope**. C2.7's descent may pass it with its authenticated scope
and the witnessed replacement evidence proving that lapse, as it passes an
omitted backing with directory absence evidence or an excluded checkpoint
with its authenticated evidence (C2.10.12). It remains held at its exact
sequence by the venue, but supplies no finalized state for any scope backing.
This is not an absence claim, and missing scope data cannot prove the step.
A wrong proof or inconsistent history under live terms is C2.10.11's excluded
class where the evidence is authenticated and unresolved otherwise; neither
licenses choosing an older state, since descent reaches the last valid checkpoint.
A public whole-scope lapse is excluded consistently from the carrying-state
reads for takeover (C2.7.1), currency (C2.7.5), snapshot selection (C2b.3) and
the no-commitment clock (C2b.6). It cannot close a silence interval. The venue
still holds its exact sequence for C2.3.4 and that signed sequence is consumed.

## 3. Import and replay

**C2.10.5 Import finalized prefixes, never raw tails.** A segment imports
exactly the opening checkpoints its scoped backings require and their
transitive authenticated predecessors. Every imported checkpoint has itself
passed C2.10.3–4. Import grants access to evidence, never authority over its
other backings. A source prefix accepted under only part of its scope, an
unwitnessed root, or an abandoned tail is not an importable checkpoint.

Each imported edge goes to an earlier witnessed index; a same-index edge is
permitted only between commitments of the same operator with strictly
increasing sequences toward the child. These locally checked ranks make the
supplied ancestry acyclic. Cross-operator state is read strictly before a
takeover's effective index (C2.7.1). Validation walks supplied evidence, caches
verified checkpoints and never discovers a global graph. Missing a required
ancestor stops replay. The cost includes all transitive shared histories.

**C2.10.6 Count common history once.** A local event's identity is its segment
identity and position, bound to one statement identity. Imported prefixes of
one segment must agree byte for byte on their common statement identities;
different valid proof bytes may attest the same statement. Deduplicate common
events before combining outputs, nullifiers and public issue/burn deltas.
One nullifier or output commitment in distinct events is a conflict, never
something a set union silently repairs. A checkpoint that conflicts with a
required canonical ancestor is not a replacement for that ancestor.

**C2.10.7 Imported state has one spent set and a forest of accepted roots.**
The new segment starts an empty local output tree. It accepts certified roots
from its imported finalized prefixes, as well as its own roots after each
accepted local statement. It never copies raw or abandoned leaves into that
tree. The spent set and duplicate-output check cover the deduplicated imported
closure plus its local statements. Each positive input may choose an accepted
root independently. Wallets sync the supplied history in bulk and compute
paths locally; requests for individual leaves or notes remain disallowed.

The snapshot binds the header and import references, ordered local history,
local note root, combined spent-set root and the backing's issued/burned totals.
The accepted-root forest is determined by this replay, not by an unbound list
the operator supplies. A replayer applies the proof relation under the exact
segment scope and recomputes every public state change.

**Why no private backing-to-anchor relation is added.** By induction, any
event involving backing b was finalized by a checkpoint scoped over b and
extending b's prior canonical state. Importing it through another backing
cannot create a new b event. Before b enters the current scope, b's complete
canonical predecessor, including every spent nullifier, is also imported.
Thus an old b note encountered through another backing's history is either
still live or already nullified. This argument depends on complete-scope
finality and exact predecessor import. It is false for raw history unions.

## 4. Receipts, handover and restart

**C2.10.8 Receipt authority includes the segment.** A receipt signs the
immutable construction domain, segment identity and scope root, local
statement position and identity, resulting history hash, exact evidence
hashes, and the last commitment sequence the signer had signed (`after`).
Strict verification establishes the signature; the authenticated scope and
witnessed term chain establish the signer's authority. A receipt under an
old scope does not acquire a new term because the key is appointed again.
It proves acceptance, not a holding or finality. Re-proving a statement in
the same segment returns the original receipt. Rebuilding under a changed
segment requires a new statement identity and fresh evidence for it — a new
proof, and for a kind whose authorization names the segment, a fresh
signature; note nullifiers and deterministically chosen outputs remain
unchanged.

**C2.10.9 Change scope at a committed boundary.** The C2.6.1 schedule applies
to the earliest boundary of any scope backing, and the operator keeps itself
free for that last commitment. After the boundary, it may serve remaining
backings in a new scope from their finalized state without the departing
backing's cooperation. Any incompatible unwitnessed tail is discarded as a
unit; its roots and receipts cannot be imported into the new segment. Those
payments require reproof/resubmission. Slow inclusion can therefore lose the
unwitnessed tail for unaffected backings as well: shared opacity has this cost.

Final inclusion is checked before lapse: a finalized statement stays final
after any scope term ends. A receipt contradicted by an earlier live-scope
checkpoint remains evidence of that historical contradiction. Lapse excuses
only the still-unfinal incompatible tail, from the actual effective boundary;
a pending replacement or earlier signing deadline is not early discard
authority. Removing a tail from active state retains its signed receipts and
checkpoints as durable evidence.

An elective scope change while every old term remains live must first witness
the entire admitted tail and latest signed commitment under the old scope.
Adding or dropping a backing is not itself permission to abandon receipts.
Discard requires the public whole-scope lapse above or C2.7's evidence that
the held state is stale, including C2.4.3 failed-publication repair as defined
below. Otherwise the scope change waits; a new segment may
not be used to turn a live receipt into an excused one.

**C2.10.9a Failed-publication repair can lapse an unfinalized receipt.**
For a receipt whose `after` is held, authenticate that checkpoint as belonging
to the receipt's segment. In the same operator's sequence order after `after`,
let R be the first held checkpoint that belongs to a different segment and
carries a backing of the receipt's scope. A checkpoint carrying none of the
scope's backings changes no scoped backing's state and is passed over. R
proves a repair boundary for this receipt when all of the following hold:

- R is a canonical finalized empty opening: its sequence equals its
  authenticated header's opening sequence and its local history is empty.
  Its imports satisfy C2.10.4–5; a claimed header or directory omission alone
  is not an opening proof.
- Between R and the last held checkpoint of the receipt's segment before R,
  `after` at the least, some sequence is absent from the exact venue record:
  a commitment of this operator signed after that checkpoint expired
  unwitnessed, and R proves the record has moved past it. A hole followed by
  continued checkpoints in the receipt's segment does not itself establish
  repair. A held checkpoint carrying none of the scope occupies its sequence:
  passing it over neither proves a hole nor is one.
- Every original scope term is live at R's witnessed index. If a term ended
  at or before R, use C2.10.9's actual scope boundary instead.

Read the held checkpoints from `after` through R against one complete venue
view, including lower sequences at the same index. Validate their canonical
histories before classifying the receipt. Inclusion in the receipt's segment
binds its position, statement identity, resulting history hash and committed
proof and signature hashes (C2.10.10). A held checkpoint after `after` but
before R in that segment which does not include
the receipt proves a historical live-scope contradiction. At `after` itself,
an absent position is not a contradiction, but an occupied receipt position
with a different statement identity, history hash or evidence hashes is.
Inclusion and contradiction are independent facts; neither can be erased by R. An unresolved
checkpoint establishes nothing; an excluded one is passed and is not the
transition (C2.10.9c).

At R, only a receipt with neither final inclusion nor an earlier proven
contradiction lapses under this rule. The verdict is at that boundary; it
does not suppress final inclusion proved by a later checkpoint. No unfinalized
prefix is imported, and signed receipts remain durable evidence. This
generalizes C2b.4's failed-publication lapse to receipts preceding the failed
commitment, even though their own `after` remains held. It adds no signed
object and does not depend on a silence clause. The cost is that an operator
can deliberately leave a sequence unwitnessed and abandon an unfinalized
payment through canonical repair. The public record cannot distinguish that
choice from failed publication, and proves neither the hidden commitment's
contents nor its signing time. Requiring that tail to survive would instead
need a different continuity mechanism across segments.

**C2.10.9b A carrying scope change without repair abandons the unfinalized
tail.** Let R be the checkpoint C2.10.9a selects: the first held checkpoint
after `after`, in the same operator's sequence order, that belongs to a
different segment and carries a backing of the receipt's scope. Read inclusion
and contradiction from the receipt's segment before R, as C2.10.9a defines
them, in ascending sequence order and ending at the first inclusion. Where R
is canonical, witnessed while every original term is live, and does not prove
a repair boundary, the operator changed scope without first witnessing the
receipt (C2.10.9). The receipt is **abandoned**: neither final nor excused. No
later checkpoint of the receipt's segment extends that backing's carrying
state (C2.10.4), so nothing finalizes it afterwards. A receipt included before
R was witnessed first, and R is then an ordinary scope change. An unresolved R
establishes nothing; an excluded one is passed and is not the transition
(C2.10.9c). A pending term end or a passed signing
deadline does not excuse abandonment; the key that signed the receipt and R
answers for it.

Read at one venue index, a receipt is therefore **final** where a canonical
checkpoint of its segment, witnessed while every scoped term was in force,
includes its position, statement identity, history hash and committed proof
and signature hashes (C2.10.10); otherwise
**contradicted** where such a checkpoint after `after` omits it before R, or
where such a checkpoint at or below `after` holds its position otherwise;
otherwise **abandoned** at R; otherwise **lapsed** where the record moved past `after` (C2b.4), where
R proves repair, or where a scoped term has ended (C2.10.9); otherwise
**pending**. Inclusion is read first and survives every later fact; a
contradiction stays evidence beside later inclusion. A receipt whose `after`
was moved past is not contradicted by omission: what its tail claimed died
with that commitment, while a position fixed before its era still is.
Checkpoints witnessed at or after the earliest term end are
lapsed for the whole scope and neither finalize nor contradict. Evidence the
reader cannot validate yields no verdict beyond the facts already proven: an
inclusion once proven is final, and a proven contradiction remains evidence.

A new segment commits its opening state before it co-signs so its receipts
name a commitment of that segment (C2.7.4). One commitment stands in flight
per operator on the venue, across its segments (C2.4.3). A process restarting
within the same still-current segment restores its latest signed commitment
and co-signed tail durably (C2.8.1), then waits the lag before signing.
Restart alone is not a scope reset and does not discard live receipts.
For constructions implementing [the recovery contract](pool-recovery.md#6-return),
C2b.4.1/3 additionally ends continuation and the receipt walk at the segment's
proven silence boundary, preserving every earlier finalized prefix and
earlier liability. This refinement does not add silence to pool-v2.
If the record or scope has moved past that state, C2.7 and C2.10.5 determine
the new opening state. Publication failure does not reuse a signed sequence.

For later constructions, [C2.10.9c](pool-fault.md#6-receipts) passes excluded
checkpoints without creating holes or receipt boundaries; C2.10.10–13
define committed evidence, classification and the complete record read.

## 5. What this replaces and costs

This replaces authorization by an immutable operator key, a single tree that
could silently retain abandoned history, and selective finalization of opaque
mixed-backing statements. The directory becomes both the complete public
scope and the place its shared state is committed; the existing replacement
chain remains the source of authority. No coordinator, permission from the
old operator, private routing label, or additional cryptographic primitive is
introduced. Scope membership uses the construction's existing Merkle/hash
machinery and requires new circuit relations and keys.

Costs are public service scopes and term boundaries; privacy limited to the
current operator/domain/venue scope; source-anchor provenance visible without
revealing the spent leaf; complete-scope finality for shared tails; permanent
transitive history and availability dependencies; and new frames, circuits,
configuration semantics and wallet resynchronization. Independent replacement
preserves authority independence, not independent historical data availability.
Mixed-backing spends remain one private statement when their backings share
service; across operators they require the separately specified atomic
mechanism or separate payments.

This contract does not define presentation, snapshot-redemption adoption,
atomic exchange across operators or a cross-venue bridge. Those outstanding
objects must be specified before a construction claims to support them. The
byte layouts and pinned artifacts that instantiate this contract are
`pool-v2.md`; `pool-v1.md` remains the historical byte contract, and its
fixed-operator implementation is not a replacement-capable deployment.
