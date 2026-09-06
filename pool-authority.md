# The shielded pool across operator replacement

This is the normative authority and history contract for Construction §C1.2
and §C2. It repairs the boundary identified in the reference's
[review](https://github.com/mediumofexchange/reference-ts/blob/aa72c09/docs/POOL_SEQUENCING_BOUNDARY.md).
The maintainer approved preserving private spends and independent backing
replacement on 2026-09-06. This contract requires a new construction version;
it does not change the bytes, keys or interpretation of existing `moe/pool/v1`
notes. The replacement-capable construction is not instantiable until its
complete layouts, circuits, keys and remaining supported objects are pinned.

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
after a stale record requires a new segment. A returning key receives no
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
candidate before replay checks it. An invalid candidate is not accepted;
unavailable evidence is not an empty state. A commitment whose scope has lost
a term cannot be used to finalize its retained backings' tail either.

**C2.10.4 Continuity is per backing, event storage is shared.** For a segment's
first checkpoint, each backing's opening state is C2.7's state from the record;
for continued service it is the latest carrying state, including where the
same operator changes scope without a replacement. Later checkpoints of the
same segment extend its already final prefix without changing any earlier
statement or opening state. Every valid checkpoint becomes the new state for
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
omitted backing with directory absence evidence. It remains held at its exact
sequence by the venue, but supplies no finalized state for any scope backing.
This is not an absence claim, and missing scope data cannot prove the step.
A wrong proof or inconsistent history under otherwise live terms is invalid
data, not this public lapse condition or a license to choose an older state.
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
segment requires a new proof and statement identity; note nullifiers and
deterministically chosen outputs remain unchanged.

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
the held state is stale. Otherwise the scope change waits; a new segment may
not be used to turn a live receipt into an excused one.

A new segment commits its opening state before it co-signs so its receipts
name a commitment of that segment (C2.7.4). One commitment stands in flight
per operator on the venue, across its segments (C2.4.3). A process restarting
within the same still-current segment restores its latest signed commitment
and co-signed tail durably (C2.8.1), then waits the lag before signing.
Restart alone is not a scope reset and does not discard live receipts.
If the record or scope has moved past that state, C2.7 and C2.10.5 determine
the new opening state. Publication failure does not reuse a signed sequence.

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
atomic exchange across operators, a cross-venue bridge, byte layouts or pinned
artifacts. Those outstanding objects must be specified before a construction
claims to support them. `pool-v1.md` remains the historical byte contract;
its fixed-operator implementation is not a replacement-capable deployment.
