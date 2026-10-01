# The shielded pool, construction v3

## 1. Status and scope

This document fixes the successor's six proof relations, public-input orders
and statement, authorization, publication, snapshot, receipt, segment-header
and fault-evidence records, plus served-trail, evidence-package and
record-range transport, including the history and exact-evidence chains. It implements the selected [recovery](pool-recovery.md),
[delivery](pool-delivery.md) and [transfer/fee](pool-fees.md) contracts without
reinterpreting any [v2](pool-v2.md) bytes, notes or keys.

**`moe/pool/v3` is adopted with one configuration, §11.4's.** That section
names its configuration hash and approves its source, helper, backend,
verifier-target, parameter, bytecode and key identities; it records the
compiler that produced them. The adoption also covers the replay, retention
and resource model that §14 consolidates. **E** names this construction and
that configuration hash ([Construction
§C1.3](construction.md#c13-what-e-declares-for-the-construction)), and
everything here is fixed by that name: the bytes, identities and verdict
rules of this document, and the rules of the contracts it instantiates as
this document reads them, including the limits they state. A change to any
of them is `moe/pool/v4`, and a backing moves to it by successor. A later
edit here may only correct text in a way that changes no byte, identity or
verdict. Those contracts may change for a later version; v3 reads the text
they had in the revision that added §11.4. A venue profile's rules are fixed
by the venue identity that names them (§13.1), not by this name.

A verdict's dependencies are those §12.1 derives; there is no certificate
encoding. The [fault](pool-fault.md) and [spent-set](pool-spent.md)
contracts remain binding, and no proof check replaces either. Section 12
fixes source-neutral evidence transport, not a complete-certificate verdict.
Section 13 fixes the record-range answer a
venue-evidence verifier returns. The [Ergo venue profile](venue-ergo.md) is
the selected profile that establishes it; each reader still selects its own
verifier of that profile. A backing may declare `moe/pool/v3` only with
§11.4's configuration hash. No runtime may accept v3 statements as v2. A
domain other than §11.4's, such as a synthetic one used to test these
relations, is not a construction domain. Adoption fixes this construction.
It is not a deployment: a venue, custody and an implementation's
release remain each party's own choice.

Under Construction C0a this instantiates the existing contracts, replacing
their deferred proof-layout descriptions while keeping v2 immutable. Relative
to v2, issue and burn each add two public digest fields; spend adds two output
commitments and two digest fields. Demand, settle and request add three
relations and keys. The recovery, delivery and transfer/fee contracts already
select those costs and capsule bytes. Fixing
their order adds no primitive, signature, history commitment or authority.

## 2. Common relation rules

The field, canonical encodings, identifier limbs, Poseidon2 sponge and tags
`1001` through `1006` are [pool-v2 §1](pool-v2.md#1-fields-hashes-and-encodings).
The note, owner, commitment and immutable nullifier formulas are
[pool-v2 §3](pool-v2.md#3-notes). The note tree has depth 32 and the scope tree
depth 16, with paths and hash ordering as in pool-v2 §§4–5. Add `T_TAG = 1007`
and `tag = H(T_TAG, nf)` (pool-recovery C3.1). No other primitive is added.

Every identifier or digest pair consists of two public `u128` limbs in high,
low order. Every note value and public quantity, instant, deadline or refresh
is a `u64`. Other public scalars are canonical field elements. Witness note
backings and links are also `u128` limb pairs; witness values are `u64`.
Every integer range is enforced by the relation, not merely by a caller's
input encoder. Widen values before summing; no field or `u64` wraparound can
stand for conservation. Every note has nonzero owner, rho and commitment;
every authenticated input has a nonzero secret and nullifier.

In §3, `P` expands to
`domainHi, domainLo, segmentHi, segmentLo, scopeRoot`, and `D` expands to
`deliveryHi, deliveryLo`. These are ordered lists, not nested public inputs.
The domain is eventually the final v3 configuration hash; segment identity
and scope authority are separately checked by the reader. The relation binds
all its public inputs under the selected proof system, including fields it
otherwise only range-checks. C3.2's proof-binding obligation applies to
unsigned demand/request metadata without asserting presenter-key participation.

## 3. Six proof relations

### 3.1 Issue, kind 1

```text
publicInputs = [P, backingHi, backingLo, quantity, cm, D]                 n = 11
```

The witness and relation are pool-v2 §7.1: positive quantity, exact nonzero
note commitment from private owner/rho, and the scope path for backing/link.
Append D after cm. It binds the one output's C4.4 delivery vector. The backer's
issuance signature must cover the entire eventual statement, including D;
the circuit does not establish that signature or permission to issue.

### 3.2 Spend, kind 2

```text
publicInputs = [P, anchor_1, anchor_2, nf_1, nf_2,
                cm_1, cm_2, cm_3, cm_4, D]                            n = 15
```

The witness contains two inputs, each with note, secret, note path and scope
path/link as in pool-v2 §7.2, and four output notes. Pool-fees C1.2.3 fixes the
relation: authenticate both inputs, prove membership for positive inputs,
prove both scope memberships, require distinct nonzero nullifiers and positive
total input value. A zero input names the other input's backing. Each output
names an input backing, recomputes its nonzero commitment and has nonzero
owner/rho. All six pairs of output commitments are distinct. For each input
backing, the widened sum of its two input positions equals the widened sum
of ALL four output positions naming it. D binds all four capsules in order.
Zero outputs remain ordinary notes and occupy leaves. No output is reserved
for a fee and no fee-specific validity predicate is introduced.

Padding input membership and its anchor remain unconstrained by this
relation as in pool-v2 §7.2; the reader checks both anchors against the forest.

### 3.3 Burn, kind 3

```text
publicInputs = [P, backingHi, backingLo, quantity, anchor_1, anchor_2,
                nf_1, nf_2, cm_change, D]                             n = 15
```

The witness and relation are pool-v2 §7.3: authenticate two distinct inputs,
prove the public backing's scope membership, require both inputs and change
to name that backing, require positive burn quantity, and compare the widened
input sum with quantity plus change. Bind the exact change commitment with
nonzero owner/rho/cm even when change is zero. Append D after cm_change; it
binds the one change capsule. There is no additional fee output or deduction.

### 3.4 Demand, kind 4

```text
publicInputs = [P, backingHi, backingLo, quantity, anchor_1, anchor_2,
                tag_1, tag_2, presenterHi, presenterLo, instant, deadline] n = 16
```

The witness contains two notes, secrets and note paths plus one scope
path/link for their common public backing. Enforce pool-recovery C3.2's
segment-bound form: ownership, the exact private nullifier of each note,
distinct nonzero nullifiers, public backing equality, positive exact summed
quantity, and scope membership. A positive position proves membership at its
anchor and its public tag equals `H(T_TAG, nf)`; a zero position has both
`tag = 0` and `anchor = 0`. Keep the zero-anchor rule specific to demand;
applying it to shared input authentication would change spend/burn/settle.
Presenter limbs, instant and deadline are range-constrained public inputs.
The circuit establishes no timing window or signature by the presenter.
Demand creates no output and has no D or capsule.

### 3.5 Settle, kind 6

```text
publicInputs = [P, backingHi, backingLo, quantity, owner, rho_out,
                anchor_1, anchor_2, nf_1, nf_2, cm_out, demandHi, demandLo] n = 17
```

The witness contains two input notes, secrets and paths plus one scope
path/link. Enforce pool-recovery C3.5: authenticated distinct inputs of the
public backing, positive-note membership, scope membership, positive quantity
equal to their widened whole-value sum, and the exact public output opening's
nonzero commitment. No change output exists. Owner and rho_out are public and
nonzero. The demand identity is range-constrained public metadata. Padding
keeps spend's free anchor under C3.2; the release binds this statement's
identity. The reader must separately check demand/tag correspondence,
acceptance and release signatures, current locks, time and forest/spentness.
The lit output is recovered from its public opening; no D or capsule is
required (pool-delivery C4.7).

### 3.6 Request, kind 7

```text
publicInputs = [domainHi, domainLo, backingHi, backingLo, anchor, tag,
                refresh]                                              n = 7
```

The witness is one note, secret and note path. Enforce pool-recovery C3.2's
segment-free form and C2b.5.1: positive value, ownership, exact nonzero
nullifier, membership at anchor, public backing equality and exact tag.
Refresh is a public `u64`, last in the order; it changes no ownership or
membership calculation but must be proof-bound. Request has no segment,
scope or public quantity. It creates no output, carries no signature, D or
capsule, and is never admitted to a history. Copying an existing proof cannot
authorize a new refresh value; independently proving the note can.

Withdrawal, kind 5, has no proof relation or verification key. Its statement
layout and authorization are fixed in §5 under pool-recovery C3.6.

## 4. Proof and conformance obligations

This construction uses pool-v2 §12's selected UltraHonk proof system and
backend; compilers qualify under Reproduction below. A verifier selects the
committed key by statement kind, never by public-input count or a key
supplied with a proof. In particular, spend and burn both have 15 public
inputs; neither proof can verify as the other relation. §11.4 names the
approved keys and configuration hash.

**Proving parameters.** The proof system commits with KZG over BN254, using
the powers of one secret `x` from the Aztec Ignition setup. Given a key,
verification reads the generators `[1]_1` and `[1]_2` and the point `[x]_2`.
`[x]_2` is the first G2 point of Ignition's `transcript00.dat`: as 128 bytes,
`x.c0 || x.c1 || y.c0 || y.c1`, each 32-byte big-endian, its SHA-256 is
`01797bfc4de5a96f0e516a9ea4537d18786dc30cb991aca4274c95822b69c32f`, and the
pinned backend's parameter loader refuses any other G2 point. Key derivation
and proving read that transcript's leading G1 points `[x^i]_1`, those below
each relation's circuit size. §11.1's key identities fix the key bytes a
verifier uses, not those points: a key is a few G1 commitments, which other
points can reproduce. Soundness rests on the key bytes, the generators,
`[x]_2` and at least one Ignition participant having destroyed its
contribution. Zero-knowledge also assumes the prover's G1 points are the
genuine powers. As pool-v2 §12 requires, an implementation records the hashes
of the parameter copies it loaded and where they came from, and it loads only
copies whose hashes match its independently selected manifest (§11.1); a
copy's hash depends on its file layout, so the manifest names each layout it
accepts. This construction names both inputs; neither enters the
configuration. It adds no ceremony and no parameter authority.

**Reproduction.** A configuration identity names an artifact, not a compiler
version. A compiler that reproduces every bytecode identity from the pinned
sources gives the same relations and is conforming evidence for §11.1's
check; the manifest names the compiler that was used. An artifact's other
fields, such as its ABI, lie outside the identity: a prover builds witnesses
with them, so a mismatch can only make proving fail. Key derivation, proof
bytes and verification remain the pinned backend's.

**Security margin.** Published estimates put BN254 at about 100-bit security
against discrete logarithms in its pairing target group, whose prime has the
special form BN curves give it (the special extended tower number field
sieve; Barbulescu and Duquesne, "Updating key size estimations for
pairings", Journal of Cryptology 32, 2019). That is below the 128-bit level
the curve was designed for. This construction accepts that margin. Another
curve is another proof system and needs a successor construction; no
configuration, term or served object selects one. These three paragraphs
state assumptions the proof system already makes and pool-v2 §12's parameter
record. They add one implementation check, the parameter hashes before
loading, and no field or authority.

One combined build must compile all six relations against the same helper
sources, derive six keys, verify genuine proofs and check every exact public
input order. Mutation checks cover every scalar under that relation's own
key. Include otherwise-valid alternative metadata, range overflow below the
ABI encoder, duplicate inputs/outputs, padding, scope, ownership, membership,
cross-backing conservation and widened quantity boundaries. Test spend/burn
proof substitution in both directions with equal input counts. Observed
rejection is evidence under this backend, not a proof of nonmalleability for
every proof system or of correct deployment/setup provenance.

The digest relation reads no cipher or SHA256 preimage. C4.4's exact capsule
count/width/profile, output association and digest are host checks at admission
and replay. Proof-only conformance cannot establish those state transitions,
issuance permission, signatures, witnessed time, unspentness, locks, finality,
restoration completeness or permanent evidence availability. Retained tests
must label any synthetic domains, capsules and prevalidated state as such.

## 5. Canonical statement records

This section fixes the statement bytes; §11.4's configuration hash is the
domain every v3 statement carries. All contexts below are
literal ASCII. Integers, fields and identifier limbs use §2 and pool-v2 §1.
No optional field, alternate spelling or trailing byte is accepted.

```text
statementBytes = "moe/pool/v3/statement" || configHash[32] || u8 kind ||
                 u32 n || publicInputs[0] ... publicInputs[n-1]   (each F)
statementHash  = SHA256(statementBytes)
recordBytes    = statementBytes || u32 proofLength || proof[proofLength] ||
                 u32 authorizationLength || authorization[authorizationLength] ||
                 u32 capsuleCount || capsule_1[89] ... capsule_capsuleCount[89]
```

`kind` is exactly 1 through 7. Counts for kinds 1–7 are respectively
11, 15, 15, 16, 7, 17, 7. Kinds other than 5 use §3's public-input order.
Kind 5 uses `[domainHi, domainLo, segmentHi, segmentLo, scopeRoot,
demandHi, demandLo]`. Its last pair names the standing demand to withdraw.
The first two inputs must reconstruct exactly the outer `configHash`, even
for a proofless withdrawal. All fields are canonical; every identifier limb
is below `2^128` and every quantity, instant, deadline and refresh is below
`2^64`. Public quantities are positive. These are checked before proof or
signature verification, including by an encoder given external objects.

| Statement kind | Proof | Authorization | Capsule count |
|---|---|---|---|
| 1 issue | §4 | backer K's signature over `statementBytes`, 64 bytes | 1 |
| 2 spend | §4 | empty | 4 |
| 3 burn | §4 | empty | 1 |
| 4 demand | §4 | empty | 0 |
| 5 withdraw | empty | presenter's signature over `withdrawalBytes`, 64 bytes | 0 |
| 6 settle | §4 | `u64 acceptanceDeadline || acceptanceSignature[64] || releaseSignature[64]`, 136 bytes | 0 |
| 7 request | §4 | empty | 0 |

An empty proof has length zero and is permitted only for kind 5. Every other
proof is nonempty, at most 131072 bytes and a multiple of 32 bytes, as in
pool-v2 §12. The authorization length and capsule count must equal the table,
not merely fit a maximum. A capsule is exactly profile 1's 89 bytes, beginning
with `u8(1)` (pool-delivery C4.3). No capsule length or commitment is repeated
inside the vector: counts and widths are fixed by kind and commitments are
already public inputs. The reader recomputes C4.4's `deliveryHash` using the
outer domain, those output commitments in order, and these capsules, and
requires equality with the final two public inputs of kinds 1–3. The same
checks apply at admission and replay. Zero outputs still require capsules.

The statement identity excludes proof and authorization. Capsules enter that
identity through the proof-bound delivery digest. Exact evidence retains the
entire record, including the vector; a proof variant does not change the
statement identity. In the fault contract's evidence triple, `proofHash` is
SHA256 of the proof and `signatureHash` is SHA256 of the complete authorization,
each replaced by 32 zero bytes exactly when its field is empty. An empty
field is not represented by SHA256 of the empty string. This fixes the triple's
components; §7 fixes their evidence-chain and snapshot frames.

Decoding establishes canonical structure and delivery association, not proof
validity, signature validity, authority, demand standing, locks, spentness,
time or finality. Those checks remain mandatory under the contracts. A parser
cannot classify a syntactically valid record as an accepted statement.

## 6. Signed objects and publication records

```text
acceptanceBytes = "moe/pool/v3/acceptance" || configHash[32] || demandId[32] ||
                  owner[F] || u64 deadline
acceptanceId    = SHA256(acceptanceBytes)
releaseBytes    = "moe/pool/v3/release" || configHash[32] || demandId[32] ||
                  acceptanceId[32] || settlementHash[32]
withdrawalBytes = "moe/pool/v3/withdrawal" || configHash[32] || withdrawStatementHash[32]
publicationBytes = "moe/pool/v3/publication" || configHash[32] || backing[32] ||
                   u8 publicationKind || u32 bodyLength || body[bodyLength]
publicationId  = SHA256(publicationBytes)
```

All signatures use pool-v2's strict Ed25519 verification. K signs the exact
`acceptanceBytes`; the demand's presenter signs the exact `releaseBytes` or
`withdrawalBytes`. The acceptance owner is a nonzero canonical field element
and its deadline a `u64`. For settlement, reconstruct `acceptanceBytes` from
its domain, demand identity and public owner, and the authorization's deadline;
verify the first signature under the backing's K. Reconstruct `releaseBytes`
from that acceptance identity and this settlement's `statementHash`; verify
the second signature under the named demand's presenter. No separate release
payload, acceptance owner or demand identity is stored in the authorization.
The reader checks the demand's backing, quantity, tags, presenter and deadlines
under C3.4–8. A withdrawal signs its statement identity, binding its segment
and scope as well as its demand (C3.6); a new segment needs a new signature.

Publication kinds form their own enumeration:

| Publication kind | Body |
|---|---|
| 1 demand | kind-4 `recordBytes` |
| 2 acceptance | `acceptanceBytes || signature[64]` |
| 3 release | kind-6 `recordBytes` |
| 4 withdrawal | kind-5 `recordBytes` |
| 5 request | kind-7 `recordBytes` |

The inner and outer domains must match exactly. Each body must be exhausted
after reading the exact kind above. The parser bounds `bodyLength` before
copying: a statement body is at most its fixed statement length plus three
u32 lengths/counts, its kind's maximum proof length, exact authorization
length and exact capsule bytes; an acceptance body is exactly 190 bytes.
The fixed statement length is 58 + 32n bytes. Thus no unbounded payload is
introduced. Venue chunking or transport envelopes are outside this frame.

Demand, release and request name their backing directly in their public
inputs; the publication's routing backing must equal it. Acceptance and
withdrawal name a demand and inherit that demand's backing. Their routing
backing cannot be verified without the demand, which must be resolved and
checked before using either as evidence or giving it force (C2b.3.2). A codec
returning either object leaves that contextual check outstanding. No published
object carries a spent-set non-membership proof (C3.6, C2b.3.3).

An exact republication has force at the first index at which it has any
(C2b.3.2), which need not be its first witnessed index: an earlier copy may
have lacked force. A changed proof, authorization or routing frame can change
`publicationId` without creating a new statement. Readers must also enforce
the contracts' semantic identity rules. For request counting specifically,
the request identity is its `statementHash`, read at the first index a request
of that identity was witnessed naming the backing (C2b.5.2); proof variants
or republication cannot extend that count window. Demand repetition cannot
create a second lock or extend its deadline. Parsing and publication hashing
alone do not enforce these replay rules.

**C0a cost and replacement.** The authorization field generalizes v2's
obligor-signature field and the capsule vector instantiates C4.4. The vector
costs four count bytes plus 89 bytes per output, without duplicating cm. The
publication body length costs four bytes and bounds parsing before copying.
No signature, proof, replay authority or history chain is added. These records
replace the deferred successor records, leaving every v2 byte unchanged.

## 7. History, evidence, snapshots and receipts

The history retains pool-v2 §9's meaning under new contexts. For a segment
identity `S`, before any local statement, and after the statement at position
`i`, respectively:

```text
historyHash_0 = SHA256("moe/pool/v3/genesis" || S[32])
historyHash_i = SHA256("moe/pool/v3/history" || historyHash_(i-1)[32] ||
                      statementHash_i[32] || noteRoot_i[F] || spentRoot_i[32] || u64 i)
evidenceHash_0 = SHA256("moe/pool/v3/evidence-seed" || S[32])
evidenceHash_i = SHA256("moe/pool/v3/evidence-link" || evidenceHash_(i-1)[32] ||
                       statementHash_i[32] || proofHash_i[32] || signatureHash_i[32] || u64 i)
snapshotBytes(b) = "moe/pool/v3/snapshot" || b[32] || S[32] || historyHash_n[32] ||
                   evidenceHash_n[32] || u64 issued(b) || u64 burned(b)
snapshot(b) = SHA256(snapshotBytes(b))
```

Every digest and identifier is exactly 32 bytes. Positions start at 1 and
are `u64`; position zero is represented only by the two seed formulas.
Advancing past `2^64 - 1` refuses before any state changes; it never wraps.
`noteRoot_i` is the segment's local note-tree root after position i.
`spentRoot_i` is the canonical compressed root of pool-spent C1.2.8–9 over
the validated imported closure and the local prefix; this adopts that root
definition for the v3 history frame, not v2's sparse root. A statement with
no output or nullifier retains the respective preceding root. It still has
its own position and advances both chains. Request, kind 7, is never a history
event. These hash functions are not admission or replay validators.

`issued(b)` and `burned(b)` are the per-backing totals over the deduplicated
imported closure plus this segment's local history, as in pool-v2 §10. Valid
state has `burned(b) <= issued(b) < 2^64`. Both totals are unsigned 64-bit
fields in the snapshot preimage. Framing and hashing accept every pair of
u64 totals, including a pair that violates that inequality: otherwise a
reader could not authenticate a signed assertion of invalid supply before
rejecting it. Hash equality establishes what was committed, not its truth.

### 7.1 Authentication precedes validity

The evidence recurrence consumes §5's three exact digests. Unlike admission,
it must not depend on successful proof, authorization or state verification.
For example, a well-framed issue with a bad backer signature has the same
statement identity as one with a good signature, but a different evidence
hash and snapshot. A replica's substitution of either signature cannot
reproduce the original signed snapshot. An operator that commits the bad
signature has instead authenticated that failing evidence (C2.10.10–11).
Malformed or missing record data is never silently represented by a zero
digest: the zero sentinel is used only for an actually empty proof or
authorization field under §5, not for failed verification or parsing.
Digest calculation can read exact proof and authorization byte fields even
when their lengths violate §5. An actually empty field uses the zero sentinel,
but that does not make its absence valid for that kind. A strict record
decoder's refusal does not substitute a digest for the supplied bytes.

A reader authenticates a supplied snapshot preimage against the digest in
the signed commitment's authenticated directory, with the expected backing,
segment and commitment identity. The preimage does not authenticate itself.
It reproduces the evidence chain over the exact supplied evidence before
using a deterministic verification failure as grounds for exclusion. A
mismatching chain means unresolved evidence, not operator fault. Complete
classification still requires C2.10.11–13's record prefix, scope, terms,
imports, last-valid continuity and replay, except that §9.1 may replace the
target's local event trail with its required dependencies intact; unsupported verifiers, resource
failures and programming failures remain unresolved, never exclusion.

A checkpoint's expected segment and scope are the ones its directory's first
entry names: that entry's snapshot's segment and the segment's authenticated
header, whichever scoped backing the reader holds. Readers of every backing
therefore judge a checkpoint in one segment and read one class (C2.10.11).
After lapse, validity requires every scoped backing's snapshot to name that
segment with the same history and evidence hashes, so an entry whose snapshot
names another segment excludes the checkpoint; a reader whose backing that
scope does not hold finds the directory and scope unequal and excludes it
too. A first entry whose snapshot does not authenticate against that
segment's header, or whose segment's head the reader lacks, leaves the
checkpoint unresolved for every reader, as withheld evidence does. A
continuation's opening is its segment's where the opening's own first entry
names that segment, including an opening that omits the reader's backing. A
first entry whose backing the segment does not scope cannot name it, so the
directory alone shows that such an opening is not the segment's, and that
such a receipt's `after` (§7.2) is not of the receipt's segment, without
another backing's snapshot.
The rule adds no mechanism: the directory's order is already fixed (§12).
Its cost falls on a reader of another backing, where the checkpoint is
lapsed or excluded: it reads the first entry's snapshot preimage, that
segment's head and scoped terms, and the record views and clocks of the
backings it scopes, which judging lapse there needs; validity already needs
every scoped snapshot. §10.1's expected segment is this one.

To authenticate the evidence triple at a known position i against a held
terminal `evidenceHash_n`, the reader can use the chain value before i, the
triple at i and the triples at every later position through n. Apply the
recurrence at i and then at consecutive positions, and compare the result
with the terminal hash. At i = 1 the preceding value must be the seed for S.
The last position must be n, with `1 <= i <= n < 2^64`; neither gaps nor
overflow are accepted. For i > 1, the preceding hash need not be replayed to
authenticate this suffix against an already authenticated terminal hash;
this says nothing about the prefix's validity. Suffix digest data alone does
not establish the target statement's contents: its claimed three digests
must separately match the supplied target bytes under §5. Nor does such an
opening validate the later events or a claimed history root. This fixes the
hash-opening relation described in pool-fault §7, not a standalone certificate
wire format, a signed directory format or a complete exclusion verdict.

A later checkpoint of a segment extends an earlier valid checkpoint of that
segment (C2.10.4, C2.10.12) exactly where its trail carries at least that
checkpoint's `n` events and the recurrences above over its first `n` events
reproduce that checkpoint's `historyHash_n` and `evidenceHash_n`. Equal length
is extension, and `n = 0` is extended by every trail of the segment. The reader
compares against the segment's last valid checkpoint in the child's own record
prefix (C2.10.11), passing excluded and lapsed ones, and reads no other
checkpoint's roots or totals. Where the segment has no valid checkpoint in that
prefix, the prefix is the seed, `n = 0`. A trail that is shorter, or that
reproduces either hash differently at `n`, does not extend the prefix.

The evidence recurrence alone decides a shorter trail, or one whose
`evidenceHash_n` differs, without replaying any record. The segment identity
and the record fix the imports and adopted block. So under the same replay
inputs, equal evidence at `n` means equal records through `n`, and
therefore equal history. A non-opening checkpoint that does not extend the
prefix is excluded on that ground, where four things hold:
- its own trail authenticates (§10.1);
- it is not lapsed, and its lapse is resolved (C2.10.11);
- its directory, scope and terms are authenticated;
- its complete range is established (C2.10.13).

Where a condition fails, non-extension is no ground: the checkpoint is lapsed
where it is lapsed, and otherwise unresolved unless another rule (C2.10.11,
§9.1) classifies it. An exclusion needs one ground, so the reader need
not replay the trail's records looking for an earlier failure. Such a replay
could meet a different deterministic failure before `n`, or stop there as
unresolved on an unsupported verifier or a resource refusal. This adds no
hash; both chains already bind every statement, proof and authorization byte
at each position.

### 7.2 Receipts bind the exact event evidence

The receipt's signed bytes retain pool-v2 §9's fields under the v3 context:

```text
receiptBytes = "moe/pool/v3/receipt" || configHash[32] || S[32] || scopeRoot[F] ||
               u64 i || statementHash_i[32] || historyHash_i[32] ||
               proofHash_i[32] || signatureHash_i[32] || u64 after
receiptRecord = receiptBytes || operator[32] || signature[64]
```

The operator signs exactly `receiptBytes` with strict Ed25519. All widths
are fixed: the signed message is 259 bytes and the record is 355 bytes,
without lengths or trailing bytes. The position i is a positive u64; `after`
is a u64 commitment sequence, zero representing no previous signed commitment
as in pool-v2 §9. An operational segment still commits its opening before
issuing receipts (C2.10.9), so a zero `after` does not establish compliant
service. The public key must be the expected segment operator and the
domain, segment and scope root must match that expected segment. The record
format accepts any 32-byte operator and 64-byte signature; strict signature
verification, including key validity, is a separate mandatory check.

No evidence-chain hash is added to the receipt: it already signs the
position, statement, resulting history and both exact evidence digests that
C2.10.10 requires. A receipt compared with a valid checkpoint must match all
five values at its position. A receipt signature alone establishes neither
that inclusion nor current authority, witnessed finality, an unspent note
or a claim on an unrelated operator. Receipt precedence remains C2.10.9a–c.

Exact resubmission of an admitted statement returns its original receipt
and evidence, including when a different valid proof is supplied. An adopted
statement retains the exact §5 record from the publication that had force,
with its original source-segment binding and statement identity (C2b.4.2).
Its evidence is chained at the new position under the adopting segment's seed.
Its new receipt names that adopting segment and scope, its new position and
resulting history, and the original proof/authorization digests; `after` is
the adopting segment's opening checkpoint sequence. It does not re-sign or
re-prove the statement or rejudge its original force. Matching a receipt to
an adopted statement cannot require the receipt's segment to equal the
statement's source segment. The record-derived adopted block establishes why
that mismatch is authorized; a byte parser cannot grant the exception.

**C0a cost and replacement.** This instantiates C2.10.10's chosen SHA256
chain with the existing receipt digests, adding 32 bytes to each backing's
snapshot preimage. It preserves the semantic history recurrence and receipt
field count, and adds no signature or privileged state transition. Interior
evidence openings retain the selected linear suffix cost: 96 digest bytes
per later position, plus the preceding hash and target evidence. No evidence
tree, second history or new certificate transport is introduced. §12.1
derives the dependencies without a certificate encoding, and §14
consolidates replay.

## 8. Segment headers

The scope and segment retain pool-v2 §§5–6's meaning under the v3 context.
The scope tree, entry ordering and depth are unchanged. The canonical header is:

```text
segmentBytes = "moe/pool/v3/segment" || configHash[32] || venue[32] || operator[32] ||
               u64 sequence || u32 n || entry_1 ... entry_n
entry_i      = backing[32] || link[32] || u64 openingSequence ||
               openingOperator[32] || openingRoot[32]
segmentId    = SHA256(segmentBytes)
```

The context is literal ASCII; integers are unsigned big-endian. `sequence`
is positive and below `2^64`, naming this operator's first signed commitment
of this segment on this venue. Its required relation to the operator's
earlier signing history remains pool-v2 §6's. The number n is 1 through
65536 inclusive. Entries are strictly ascending by the unsigned bytes of
their backing names; duplicate backings and reordered entries are malformed,
even if their links or openings differ. An encoder rejects unordered input
rather than sorting it. The scope root is computed from these backing/link
pairs by pool-v2 §5, and is not a second field in the header.

`openingSequence = 0` requires both opening byte fields to be exactly zero.
This is the sole empty-opening encoding, asserting that the record pins
nothing for that backing (C2.7.3). Otherwise `openingSequence` is a positive
u64, and the three opening fields name the exact commitment's sequence,
operator and directory root. Where `openingOperator` equals this header's
operator, `openingSequence` must be below `sequence` (C2.10.5). A different
operator's sequence is independent and need not be below it. Positive
opening sequences do not create an alternate empty sentinel when either
byte field is zero: whether that reference names an authentic commitment
must be checked against the record.

The fixed prefix is 127 bytes and every entry is 136 bytes. A canonical
header therefore has exactly `127 + 136*n` bytes, from 263 to 8913023 bytes.
A decoder checks the input byte bound, context, count and exact total length
before allocating or iterating entries; no trailing bytes, optional fields
or alternate contexts are accepted. Both encoders and decoders enforce all
the structural rules above. An implementation's lower processing budget
may leave a larger header unresolved; it does not establish operator fault.

Header decoding and identity hashing establish neither authority nor an
empty or finalized opening state. The reader still authenticates the header
identity through the expected checkpoint's signed directory and §7 snapshot
preimages, checks domain, venue, scoped signed terms and current links, and
resolves every nonempty opening through the record and its complete evidence
(C2.10.1–5, C2.10.11–13). The header has no standalone signature. An unknown
opening remains unresolved; it is never replaced with an empty opening or
an older checkpoint. A strict header codec is not a classifier of malformed
committed evidence and its rejection alone supplies no exclusion verdict.
Section 12.1 derives the dependencies without a certificate format; §14
consolidates replay and import.

**C0a cost and replacement.** This replaces the deferred successor header
layout by reusing v2's fields and bounds with one new context. No scope root,
adoption index, silence duration, signature or primitive is added: those
values are already derived from the scope or the record. At maximum scope
the header is about 8.50 MiB; that is a format bound, not a phone or transport
budget. The alternative of adding such derived fields would require extra
consistency rules without authenticating missing history. Every v2 header
and identity remains unchanged.

## 9. Fault-evidence records

This fixes the portable evidence opening described by §7.1 and pool-fault §7.
It carries the actual target bytes rather than a caller's claim about their
digests. It is one component of a fault certificate, not an exclusion verdict
or a complete served trail. The record is:

```text
faultEvidenceBytes = "moe/pool/v3/fault-evidence" || snapshotBytes[164] ||
                     u64 i || u64 n || previousEvidenceHash[32] ||
                     u32 statementLength || statement[statementLength] ||
                     u32 proofLength || proof[proofLength] ||
                     u32 authorizationLength || authorization[authorizationLength] ||
                     triple_(i+1) ... triple_n
triple_j           = statementHash_j[32] || proofHash_j[32] || signatureHash_j[32]
```

`snapshotBytes` is the complete §7 frame including its context. The target
position and terminal length satisfy `1 <= i <= n < 2^64`; the suffix has
exactly `n-i` consecutive triples in that order, with no repeated count or
position field. Each target byte field is length-prefixed and has length
0 through 131072 inclusive. This per-field transport bound reuses §5's
maximum proof width; it does not widen any valid statement, proof or
authorization layout. It permits carrying, for example, an overlength
authorization or an ill-framed statement as evidence. A claimed target field
above this bound is not representable by this record; failure to carry or
process it supplies no exclusion verdict. Other evidence may resolve that
checkpoint under C2.10.11–13.

The reader hashes the target's supplied `statement` bytes with SHA256 to
obtain `statementHash_i`. It hashes the exact `proof` and `authorization`
fields under §5's digest rule: an actually empty field has the zero digest,
otherwise its digest is SHA256 of those bytes. An empty statement instead
has SHA256 of the empty string; it is not a zero statement identity. This
calculation neither requires nor establishes successful §5 decoding. An
invalid context, field encoding, input count or inner domain is retained as
the claimed statement bytes; it is never repaired or re-encoded before
hashing. A decoder's error is not evidence about bytes it did not receive.

The reader obtains the expected backing, segment and snapshot digest from
the expected signed commitment's authenticated directory and header context
(§§7.1, 8). It requires exact backing and segment equality, reproduces that
snapshot digest, then applies §7.1's seed/position/suffix checks with the
target digests computed above. An altered target field, suffix, preceding
hash or snapshot fails authentication, even if the replacement proof is
valid. At i = 1 the preceding hash is the segment's seed. For an interior
target the suffix authenticates its supplied preceding hash without proving
that earlier prefix valid. The record does not authenticate its own expected
digest and carries no new signature or publication identity.

The exact byte length is `250 + statementLength + proofLength +
authorizationLength + 96*(n-i)`. Readers check the outer and snapshot
contexts, field bounds, indices and exact remaining suffix length before
allocating target fields or suffix entries. Length arithmetic must not wrap
or round through an unchecked machine number. There are no optional fields
or trailing bytes. Encoders apply the same structural checks; raw target
fields deliberately remain separate from §5's strict record encoder.

The suffix has the existing linear cost, with no new protocol cap below
the u64 position bound. Readers must bound their local processing before
allocation or hashing, for example by a caller-supplied maximum number of
suffix entries. That local budget is not a consensus rule: an exceeded
budget, unavailable input or unsupported verifier leaves the read unresolved,
never excluded. Partial processing or an omitted suffix is not successful
authentication. Implementations may stream the same bytes; chunk boundaries
have no protocol meaning and introduce no additional signed values.

Authentication proves only that this snapshot commits to these target bytes
at this position. It does not prove that the target violates a rule, that its
prefix or later events are valid, or that the checkpoint is live, complete,
current or final. The target's configuration and verification key, relevant
terms and authorization identities, record prefix, lapse priority, imports
and continuity are still resolved under C2.10.11–13 before an exclusion
verdict. Section 9.1 permits only the stated replacement of the target's event
trail. The snapshot's history hash and totals are authenticated assertions,
not replayed state. Capsules are not fields of this evidence triple: this
record neither attests that a capsule was served nor proves its absence or
corruption. Their association is separately checked against the statement's
C4.4 digest. Faults needing omitted dependencies require their own evidence.

**C0a cost and replacement.** This replaces the deferred wire form of §7.1's
evidence opening by reusing the snapshot and three-digest recurrence. It costs
250 framing/snapshot bytes, the three exact target fields, and 96 bytes per
later event. Supplying only the target digests would not authenticate the
bytes a verifier rejects. A tree would shorten the suffix but replace the
selected chain; a new certificate signature would add an authority without
establishing record completeness. Neither is added. No v2 bytes, v3 proof
relation, valid record bound or checkpoint classification rule changes.

### 9.1 Compact intrinsic exclusion

This rule permits a reader to replace only the event trail of a non-opening
checkpoint for C2.10.11's **excluded** verdict. It does not supply a valid
checkpoint, state, clock reset, receipt verdict or import target. It applies
only when all the following hold:

1. The reader establishes the checkpoint's held signed commitment, complete
   authenticated directory, snapshot preimages for every scoped backing,
   authenticated header and every scoped signed term under the independently
   selected configuration and venue. The scoped snapshots name that segment and
   agree on the shared history and evidence hashes. All record-range and
   same-index-order evidence required by C2.10.13 is retained.
2. At the checkpoint's own record prefix, every scoped term is in force and the
   checkpoint is not lapsed. Silence, where declared, still requires every
   applicable carrying clock dependency. A lapsed checkpoint remains lapsed even
   when these target bytes prove a fault; unknown lapse remains unresolved.
3. The reader resolves the segment's valid opening, its canonical imports and
   transitive state, the last valid prefix and every checkpoint the relevant
   descent passes. It derives the adopted block under C2b.4.2 from the complete
   required publication evidence. These dependencies must be classified under
   their own original prefixes. They cannot be omitted, inferred from the target
   or replaced by the target's asserted state. The target need not successfully
   extend or replay against that state to be excluded, but its required
   predecessor state and adoption context must be known.
4. A §9 record authenticates the exact failing target fields and position against
   a scoped snapshot of that same commitment. The position is outside the
   required adopted block. The statement decodes canonically under §5, its domain
   equals the selected configuration, and one of these intrinsic failures holds:
   - For kind 1, 2, 3, 4 or 6, the committed proof satisfies §5's length/encoding
     bounds, has an encoding the selected verifier supports, and
     the configuration's verifier for that kind returns rejection on the exact
     ordered public inputs and proof bytes.
   - For kind 1, the statement names a scoped backing, the exact 64-byte committed
     authorization fails §5's strict signature verification over the exact
     statement bytes under that backing's K from its authenticated signed terms.
   A verifier exception, unsupported encoding/verifier or resource failure is
   not rejection. No supplied signer hint or cached verdict is evidence.

Only the target checkpoint's local event trail is replaced. Its §9 preceding
hash and suffix do not establish prefix validity, admission of any event,
capsule association or reproduction of history/totals. The excluded checkpoint
still occupies its held sequence and supplies nothing under C2.10.12. Passing it
reaches the actual last valid state; it never licenses selecting an older one.
A later repaired checkpoint must extend that last valid prefix, including its
committed evidence, and satisfy all ordinary validity rules. Removing any still
required dependency leaves the dependent read unresolved, even with an authentic
intrinsic failure. Existing complete-trail exclusion remains available.

This exception does not cover opening checkpoints, targets inside the adopted
block, malformed statements, admission/state/capsule faults or other signature
roles. Those require their ordinary evidence. Adopted statements retain their
original segment, proof and authorization identities and admission context;
this rule does not recheck them against a successor's segment or current standing.
An unsupported compact case is no permission to skip that checkpoint. A reader
may support a bounded subset of these cases only by refusing unresolved cases;
it must not label an unsupported case valid or excluded.

**C0a cost and replacement.** This replaces the full target event-trail
availability requirement only for the two stated intrinsic failure classes.
The existing §9 bytes and §12 kind-7 item suffice: 250 framing/snapshot bytes,
the exact target fields and 96 bytes per later event, plus the unchanged required
dependency closure. No wire tag, new signature, hash tree, private witness,
configuration authority or verifier key is added. Retaining full trails is the
unchanged alternative; a smaller target proof cannot replace range or predecessor
evidence. This changes the v3 evidence contract, not v2 or §11.4's
configuration.

## 10. Served-trail transport

A served trail carries the header, the terms and obligor signature for every
scoped backing, and the exact local records in position order (C2.10.10–11).
Its transport frame is:

```text
trailBytes = "moe/pool/v3/trail" || u32 headerLength || segmentBytes[headerLength] ||
             signedTerms_1 ... signedTerms_m || u64 n || event_1 ... event_n
signedTerms_j = u32 termsLength || terms[termsLength] || obligorSignature[64]
event_i = u32 recordLength || recordBytes[recordLength]
```

`segmentBytes` is exactly §8's canonical header, including its context. Its
entry count m determines the number of signed terms fields, in the same
backing order; there is no second scope count or backing-name field. Each
`terms` field carries the backing's exact canonical terms encoding, with K
inside it, and the signature is over its name under that encoding's declared
signature frame (Construction invariants 1–2). This transport does not change
the terms encoding, name function or signing message. An outer codec
treats the terms and signature as supplied bytes; a reader must independently
decode them, derive the expected backing name, verify K's strict signature,
and check the terms for every scoped entry against the record. Supplying an
opaque byte field does not satisfy those checks. A field that fails is
ignored and the backing's terms are resolved from another supplied field
(§12.1).

The count n is a u64, including zero. Events occupy consecutive positions
1 through n, without repeated position fields. A checkpoint's served trail
carries exactly its n records, with no uncommitted tail; a reader may obtain
it as the prefix of a longer supplied trail (§12.1), whose later records are
not evidence for that checkpoint.
Each record carries §5's exact bytes, including capsules. An adopted event
retains the source publication's record without changing its segment binding;
the trail carries no asserted adoption flag or force index. The reader derives
the adopted block and each force index from the venue record (C2b.4.2).

The outer transport accepts terms lengths from zero through `2^32-1` and
record lengths from zero through 131978. The latter is §5's largest valid
record: a spend with a 131072-byte proof and four capsules. These are transport
bounds, not permission to admit empty terms, malformed records, or kind 7.
The outer codec preserves record bytes without strict §5 decoding, repair or
re-encoding. Inner validity is a separate check. A fault whose malformed
record exceeds this bound needs other evidence, including §9 where applicable;
failure to represent it does not classify the checkpoint.

The exact size is `29 + headerLength + sum_j(68 + termsLength_j) +
sum_i(4 + recordLength_i)` bytes. The context is literal ASCII and all lengths
and counts are unsigned big-endian. There are no optional or trailing fields.
Before allocating payloads, decoding terms/records or hashing them, readers
check a local total-byte budget, §8's header bound/count/size, a local event
budget, every outer field boundary and the exact end. They bound header/term
scanning by the already checked byte budget and header scope bound. Count and
length arithmetic must not wrap or round through unchecked machine numbers;
the event count must fit the remaining bytes even for empty records. Encoders
apply the same shape and budget checks before allocating the output. Budgets
are local processing limits, not additional consensus bounds. Exceeding one,
an unsupported inner encoding or missing input remains unresolved, never
excluded. Streaming/chunking may preserve these bytes without adding identities.

### 10.1 Local evidence authentication and its limits

For a supplied §7 snapshot preimage, first require the expected backing,
segment and snapshot digest obtained from the expected signed commitment's
authenticated directory and header context. The trail's header identity must
equal that segment. Recompute the supplied snapshot digest and require equality
with the expected digest. The trail's scope must contain that backing. For every record
that can be decoded under §5, retain its exact proof/authorization and compute
its evidence triple. Apply §7's evidence recurrence from the segment seed at
positions 1 through n; the terminal hash must equal the snapshot's evidence
hash. At n = 0 compare the seed directly. This authenticates the ordered local
statement/proof/authorization bytes and their length. §5's delivery association
also checks the supplied capsules against the authenticated statement digest.
A substituted proof, signature or capsule cannot stand in for committed bytes.
For inner bytes §5 cannot decode, this procedure is inconclusive; §9 can
authenticate raw target fields without strict decoding within its own bounds.
Section 12.1 states when a prefix of a longer supplied trail is the served trail.

This check authenticates neither the supplied terms/signatures nor their force.
It does not recompute the history hash, roots, totals or imported state. Even
an authenticated zero-event trail can name nonempty imports. A committed
kind-7 record, wrong source domain or unauthorized source-segment binding can
authenticate as bytes and still fail replay. Local evidence authentication
grants no adoption exception and establishes no valid, complete, current or
final opening or exclusion verdict.

Complete opening/classification still requires the configuration and artifacts,
the signed commitment and its full authenticated directory, snapshot preimages
for every scoped backing, all scoped signed terms, the complete record prefix
and same-index order, recursively supplied opening evidence, passed checkpoints
needed for descent/clock classification, last-valid-prefix continuity and the
record-derived adopted block. The reader deduplicates and checks the imported
closure and, for validity, replays every event and reproduces every scoped
snapshot, with lapse priority and C2.10.11–13's dependency rules. A deterministic
failure after complete evidence authentication can establish exclusion without
finishing successful state replay. Section 9.1 alone permits replacing a target's
event trail by compact intrinsic evidence with its stated dependencies intact.
Other missing dependencies remain
unresolved; they are not empty openings or permission to fall back to an older
checkpoint. Section 12.1 derives these dependencies without a certificate
format, and §14 consolidates replay.

**C0a cost and replacement.** This replaces the deferred outer served-trail
frame by composing the existing header, signed terms and record bytes. It adds
29 fixed framing bytes, 68 per scoped terms field (including the existing
64-byte signature), and four per event. It adds no roots, signatures, proof
relations or authority. Repeating backing names, positions, force indices or
adoption flags would introduce redundant assertions and consistency rules;
omitting scoped terms would leave silence/force checks underdetermined. Inner
terms and record checks remain where their definitions place them. Every v2
byte, finality rule and private note opening remains unchanged.

## 11. Configuration and backing evidence

### 11.1 Configuration frame

The configuration uses the domain-only rule of pool-v2 §2. Its exact bytes are:

```text
configurationBytes = "moe/pool/v3/config"
  || bytecode(issue)[32]   || vk(issue)[32]
  || bytecode(spend)[32]   || vk(spend)[32]
  || bytecode(burn)[32]    || vk(burn)[32]
  || bytecode(demand)[32]  || vk(demand)[32]
  || bytecode(settle)[32]  || vk(settle)[32]
  || bytecode(request)[32] || vk(request)[32]
  || helper[32] || u8(32) || u8(16) || u8(2) || u8(4) || u8(1)
configHash = SHA256(configurationBytes)
```

The last five bytes are note-tree depth, scope-tree depth, maximum input
count, maximum output count and pool-delivery C4.2–4's delivery profile.
They are fixed constants, not selectable parameters. The frame is 439 bytes;
there is no count, kind list, optional field or trailing byte. The six pairs
are in kinds 1, 2, 3, 4, 6, 7 order; withdrawal has no key. The bytecode and
key hashes have pool-v2 §2's meanings, including decoded compiler bytecode
and the exact backend key bytes. The helper is pool-v2 §1's SHA-256 of the
Poseidon2 helper source. Proof system, verifier target, backend, common
relations, public-input orders, record bounds and spent-root rules are fixed
by this construction and its referenced contracts; the frame is not a way
to override them. No operator, venue, backing, segment or mutable authority
enters the configuration.

An implementation holds an independently selected manifest of shared-helper,
compiler/backend/version, verifier-target and parameter identities, and of
source, bytecode and key identities for all six relations. It checks source
and backend identities, compiles the relations together (§4 says which
compilers qualify), derives keys under §4, and compares the result with that
manifest and the configuration. It refuses missing, reordered or mismatched
identities, including for relations absent from a particular trail. A served
package may supply the configuration preimage, but cannot select that manifest
or a key. Routing uses the kind's own checked key, never the public-input
count. Hash equality alone proves neither that a relation is correct nor that
its setup is trustworthy.

This document names the adopted configuration and its manifest identities
(§11.4). An implementation's manifest holds them, checked as above; this
document is not a key source, and no package, terms or served object
supplies or selects them. No flag in those objects can adopt another
configuration. Changing any identity, or the
semantics this document fixes for them, is a new configuration, hence a new
E and a new backing (Construction C1.6).

### 11.2 Constant-payout root terms

For the smallest supported pool-delivery profile, the following is the exact
canonical terms encoding. It reuses the reference's `MOEB` version-1 framing,
name hash and backing-signature message; only the construction string's
value changes. This section supplies no encoding for a payout in claims or
a nonempty reliance graph. An unsupported encoding remains unresolved, not
an empty root or evidence that the checkpoint is faulty.

```text
terms = "MOEB" || u8(1) || u8(1) || K[32]
  || u8(1) || u32 thingLength || thing[thingLength] || i8 quantumExponent
  || u32 perUnitLength || perUnit[perUnitLength]
  || u32(0)
  || u8(5) || originalOperator[32] || u32 clauseCount || clauses
backingName = SHA256(terms)
backingSignatureMessage = "moe/backing-signature/v1" || backingName[32]
```

K and originalOperator are canonical non-small-order Ed25519 public keys.
`thing` is nonempty strict UTF-8, at most 1024 bytes, with no normalization
or byte-order-mark removal. `quantumExponent` is one signed two's-complement
byte. `perUnit` is a positive integer below `2^256`, in 1–32 big-endian bytes,
with no leading zero. The zero u32 is the reliance count. These are payout
terms, not the pool's u64 note values or running supply. All u32/u64 values
are unsigned big-endian. Clauses are strictly ascending by their one-byte
tag, with no duplicates, unknown tags or extra bytes:

| Tag | Payload, after the tag |
|---|---|
| 1, silence | u64 noCommitmentDuration, u64 challengeWindow |
| 2, witnessing | venue[32], u64 witnessInterval |
| 3, replacement | replacementRuleKey[32] |
| 4, non-service | u64 duration, u32 count, u64 window |
| 5, construction | u32(11), ASCII `moe/pool/v3`, configHash[32] |

Tags 2 and 5 are required; tags 1, 3 and 4 are optional. Thus clauseCount is
2–5. Durations, intervals, count and window have their full encoded unsigned
range; this encoding does not impose an additional service-grade calibration.
The replacement key, when present, must be a canonical non-small-order
Ed25519 public key. No clause payload has an extra length prefix. No optional
clause is inferred from a missing one. Terms are at most 1305 bytes in this
profile; readers check that bound before decoding. The signature is exactly
64 bytes, R[32] followed by little-endian S[32]. It uses the existing strict
Ed25519 rule: canonical A = K and R encodings, A non-small-order,
`0 <= S < l`, and `[8]([S]B - R - [k]A) = 0`, where B is the Ed25519 base
point, l its order and k is SHA-512(R || A || backingSignatureMessage)
interpreted little-endian modulo l. There is no additional small-order R
restriction and no ZIP215 decoding.

### 11.3 Evidence checks and their limit

For each §10 scoped terms field, decode exact bytes under §11.2, derive the
name, require equality with that header entry's backing, and verify K's
signature over the name message. Require the construction to be exactly
`moe/pool/v3`, configuration to equal §11.4's hash, and venue to
equal the header venue. Derive the issuance-verification key from those
signed terms, never a separately supplied issuer key. A changed payout,
clause, key or configuration changes the backing name and invalidates reuse
of the original signature. A v2 backing or its signature cannot be relabeled.

These checks establish terms identity and signature, not registration,
current operator, replacement-link force, revocation absence, a valid empty
opening, record-range completeness or finality. The original operator is
not necessarily the current operator: the reader still proves the current
link and operator from the witnessed chain under C2.5 and C2.10. A local
initial-segment conformance experiment may explicitly restrict itself to
the original operator and genesis link; even then it has no evidence that
no replacement or revocation has occurred. Missing authority evidence stays
unresolved. Terms/configuration checks must not produce spendable notes or
upgrade §10.1 to a complete opening or exclusion verdict.

**C0a cost and replacement.** The 439-byte fixed configuration generalizes
v2's three-pair frame to six pairs and adds one delivery-profile byte, 193
additional bytes. It replaces deferred configuration framing. Constant-root
terms reuse one name and one existing signature;
there is no certificate signature, configuration authority, registry of
approved issuers or extra proof relation. A variable list, duplicate source
hashes inside the domain or a mutable key service would add parsing or
operating costs without replacing manifest verification. Omitting a recovery
key would let a local-only check silently certify an incomplete configuration.
The manifest/source checks are local verification work, not a publication
service or a proof of currentness. V2 bytes, keys and runtime support stay fixed.

### 11.4 The adopted configuration

The adopted configuration's identities are these SHA-256 values, in §11.1's
order:

| Relation | Kind | Bytecode | Key |
|---|---|---|---|
| issue | 1 | `0c6c904321ff16bc8fdce2298257f3cd54035bc20e53bc6e711d0be5356d211c` | `65cc0ada6618c79df403d8276cd6a201e36936fb6264e516a34c4bf225729da0` |
| spend | 2 | `47f3a125a0fcfc22cd15e482db2c75173ed70eb413f5df43b38ec00f5fa51087` | `850131fcd564ce15a238142051a0780623c8c759ccaffb4e389aad5a5056acba` |
| burn | 3 | `d36344f2b2424fa6569c01bb7d8450da98fab641251ee385914332f5437c4b72` | `ce55384238c5448505f7c34b938ae92152ea75c3c412c779b20aea57bf82f3be` |
| demand | 4 | `cab1f519db1b93cb5e2b6ec89052c5c69706990960fc3c0e617f996d843e5187` | `cf3912b74b1c72eef4114049f792b76462cb3cf38f19181a90dd279175b61aa4` |
| settle | 6 | `ccad2b665539a22c1be4692e05a87bae01ccd6756ce464fc327d8a41b7938b23` | `ea5b3e66ced10ded8cd72bae33c6cf4dd7d2704735713ce3b7f2f76126967654` |
| request | 7 | `4b1a0e76a48b651a33f19a25dcad2e45e708aabd70d757528153a2098a929084` | `1f048f189568b0b6ffdac39c71957aee34ed3485490d8594aba86d800e2aeaf4` |

The helper is `44f3a3d1abe7d5fa2da5c0339e52018195d55f295c320e530d355f9cc62159d8`.
With §11.1's five fixed bytes, these give the 439 configuration bytes and

```text
configHash = 7ddbb7e86dbfaf5e7bbc28541be28fdef04da02b30214109420d840fe664a618
```

which is the domain of every v3 statement and the hash in every v3 backing's
construction clause.

The artifacts are compiled from eight sources, together:

| Source | SHA-256 |
|---|---|
| `issue.nr` | `0160497d77500bec7a62548b13908f46be122dfaf6f54cc0b720cf328a9c6ad6` |
| `spend.nr` | `920c4702d2ae0c3e2d0f251fe298c39c0d658a390220b0c4d68b4e06c31ae695` |
| `burn.nr` | `28548a85406ce232303c8bd7bb8e3506337a7b3999054678f8327ce2d979845d` |
| `demand.nr` | `b48f7f86c8ee80aa5fea7e6d2ea0b709d2a95f670b3473b2a1abc7a0afd5bf9e` |
| `settle.nr` | `cb865bea2f8087213f3d79d2eee5e07c5221cc1511d72ae6f5ee727bd690fadd` |
| `request.nr` | `2da922cbc425297aa09d231562de48eaf6f55fd392a1e060c523bd2ac981baee` |
| `notes.nr`, the relations' shared module | `034f123da4aafbac55ecaa75a3d37bbb016b70f06040a8c7092f598fbc1662d3` |
| `poseidon2.nr`, the helper | `44f3a3d1abe7d5fa2da5c0339e52018195d55f295c320e530d355f9cc62159d8` |

They are published in the reference implementation at
[`103004e`](https://github.com/mediumofexchange/reference-ts/tree/103004e3ad810bda1f68130fc3db5864d423253e/scripts/pool/v3/circuits),
with the helper under `src/pool/circuits/vendor/`. Noir `1.0.0-beta.26`
compiled the bytecode; any compiler that reproduces it qualifies (§4,
Reproduction). The pinned backend, `@aztec/bb.js` 5.2.0, derives the keys under the
verifier target `noir-recursive`, its zero-knowledge target.

The proving parameters are §4's. In the layout the reference accepts, G1 is
the leading 2^15 points of `transcript00.dat`, each `x || y` in 32-byte
big-endian, 2,097,152 bytes with SHA-256
`50d2f4e9567be2b8e382cedfd078b96a3428a94597b7e88c4116e105d578ce77`. G2 is
§4's `[x]_2`. Another implementation may accept another layout of the same
points, under the hashes its manifest names. Spend is the largest relation,
and it fits the 2^15 points.

The pinned backend proves every relation in 14,656 bytes, and the
conformance check finds a valid proof refused with a word appended, its last
word repeated or its last word removed. So the configuration's longest
publication is a 15,498-byte release, the size [venue-ergo
§8](venue-ergo.md#8-publishing) requires one transaction to carry.

**C0a cost and replacement.** Adoption adds no field, byte, relation, key or
authority. It names the identities §11.1's frame already held, unchanged
since their sources' text was fixed, and the manifest identities §4 already
required. Its cost is permanence: a defect later found in a relation, the
helper or the backend needs `moe/pool/v4` with a configuration of its own,
which every backing reaches only by a successor (C1.6). That is why the
identities are named after every rule they implement.

## 12. Evidence packages and dependency retention

A package transports retained evidence for C2.10.13. It contains exact byte
objects, not a reader's cached classifications, asserted dependency edges or
an assertion that a range is complete. The request being answered, including
the selected venue, construction, backing, checkpoint and judging prefix, is
an independent reader input; the package does not select or change it.

```text
packageBytes = "moe/pool/v3/package" || u32 count || item_1 ... item_count
item = u8 kind || u64 length || payload[length]
```

The literal context is ASCII; integers are unsigned big-endian. Count may be
zero, each length may be zero through `2^64-1`, and there are no trailing or
optional fields. The following kind numbers select payload interpretations:

| Kind | Payload |
|---|---|
| 1 | Section 11.1 configuration bytes |
| 2 | Existing signed commitment record: `u64 sequence || root[32] || operator[32] || signature[64]` |
| 3 | Complete directory preimage, as defined below |
| 4 | Section 7 snapshot bytes |
| 6 | Section 10 served-trail bytes |
| 7 | Section 9 fault-evidence bytes |
| 10 | Section 7.2 `receiptRecord` bytes (355 bytes, including operator and signature) |

Kinds 5, 8, 9 and 11 are unassigned and are not reassigned within v3, so a
package carrying one is unsupported like any unknown kind (below). A segment
header with its scoped signed terms travels as a trail of count zero (§10).
Publications are venue records, and venue evidence is read only through the
reader's own venue-evidence verifier (§12.1, §13), so no kind carries either.

The complete directory preimage is the existing directory-root frame:
`0x4d4f4544 || u8(1) || u32 m || (name[32] || digest[32])_1 ... _m`.
Names are strictly increasing in unsigned byte order, including when m is
zero. Its SHA256 is the commitment's directory root. This exposes the existing
hash preimage for transport; it does not change any directory root or signature.

Items are strictly increasing by `(kind, SHA256(payload))`, comparing the
32-byte hashes in unsigned byte order. A repeated pair is refused, even if its
payload bytes are identical. The same payload under different kinds is
structurally permitted but gains no validity under either interpretation.
The hash is computed, not carried in another field. It is a local inventory
key, not a new signed identity: the checkpoint identity remains its operator,
sequence and root; statement identity and all existing commitments remain as
defined. Packaging the same objects in a different order is noncanonical.

The outer codec preserves all payloads without decoding, repairing or
reserializing their inner fields. Empty or malformed inner evidence can be
transported; it does not become valid. Unknown kinds are unsupported evidence
and leave a read unresolved. A trail of count zero is the served trail
exactly of a checkpoint whose snapshot's evidence hash is the segment's seed
(n = 0), such as its opening checkpoint (§12.1). For any other checkpoint of
the segment it supplies only the header and scoped terms, from which a lapse
is read (C2.10.4, C2b.4.1), and claims no records. Where trails
of one segment overlap, the reader uses only authenticated bytes: a header by
its segment identity and a signed-terms field by its backing name and strict
signature (§12.1). Merely sharing a header identity does not make two trails
evidence for the same checkpoint.

The exact package size is `23 + sum(9 + length)` bytes. Before allocating
payloads or hashing them, a reader checks explicit local total-byte and item
budgets, every field boundary and the exact end. Count must fit the remaining
bytes even for empty payloads. Arithmetic must not wrap or round through
unchecked machine numbers. Only after that scan does it hash and check order.
Encoders apply the same shape, bounds and ordering checks before allocating
output. Readers own input bytes before asynchronous work and refuse shared
mutable storage. These budgets are local processing limits, not consensus
bounds. Exceeding them is a resource refusal, never operator fault.

### 12.1 A package is not a complete certificate

The reader derives dependencies from the requested read and the existing
rules, including C2.10.3–5 and C2.10.10–13. It resolves each dependency against
its existing signed or hashed identity and context, authenticates the exact
bytes, and then performs its semantic checks. A lookup miss or unresolved
dependency is not an empty opening, directory absence or permission to select
an older checkpoint. Distinct inventory keys do not resolve conflicting
evidence for one required identity. Identical shared dependencies may be used
by several reads without duplicating their payloads. A flat inventory adds no
global graph traversal or global cycle-detection requirement.

**Served trails and signed terms.** A checkpoint's served-trail dependency
(C2.10.13 item 3) is resolved by any supplied trail of its segment whose
first n records decode under §5 and whose §7 evidence recurrence over them
reproduces the evidence hash of the scoped snapshot being authenticated; at
n = 0 the seed is compared. That prefix, the trail's header, the count n
and those n records with each scoped signed-terms field resolved as below,
is the checkpoint's served trail under §10.1. It is the complete trail, not
a compact replacement under §9.1. Records after n,
decodable or not, neither block nor affect it, and receipt inclusion or
omission for that checkpoint (§7.2) reads only positions 1 through n.
Barring a SHA256 collision the recurrence binds each position, so at most
one n matches a trail and every matching prefix has identical header and
records: the matching prefixes of several trails are one dependency, not
conflicting evidence. Matching reads the evidence recurrence alone; whether the
checkpoint extends another (§7.1) is decided afterwards, as for a separately
supplied trail: from the evidence recurrence where the trail is shorter or its
evidence value at n differs, and by replay otherwise. Where a checkpoint's
scoped snapshots disagree, each is authenticated separately and judged as
before.

Each scoped signed-terms field is resolved separately by its backing name.
Any field supplied in a trail of the segment, including a trail of count
zero, whose terms reproduce the name and whose obligor signature verifies
strictly is that backing's terms evidence, checked further under §11.3. A
field that fails either check is ignored: it is not conflicting evidence, and
no outcome depends on which valid field is used, since every field
reproducing the name carries the same terms bytes.

A package may therefore carry, per segment, one trail for each set of trails
whose records are prefixes of one another, and a reader accepts either form.
A producer must not drop a trail whose prefix is some required checkpoint's
served trail unless a remaining trail also supplies it. Reading an
early checkpoint from a longer trail still scans that trail's outer frame to
its exact end (§10): a local budget that refuses the longer trail, or a
trail the outer codec refuses, leaves the dependency unresolved as any
budget refusal does.

**C0a cost and replacement.** No field, identity, hash, signature or
authority is added; a reader computes each supplied trail's recurrence once
and compares hashes. This replaces repeating every carrying checkpoint's
full trail, whose bytes grow as the sum of prefix lengths: about N²/2K
records for N events and a checkpoint every K, 8.5 times the unique records
at 1,024 events every 64 in the reference measurement. The cost is that an
early checkpoint read from a long trail is budgeted by the long trail's
bytes. A suffix trail form or record items referenced by hash would reach
linear bytes only by adding a frame, a base reference and consistency
rules. Treating differing unauthenticated terms bytes as conflicting would
let an added copy leave a classified checkpoint unresolved, which C2.10.11
excludes ("evidence obtained later ... reverses nothing"); the conflict rule
governs authenticated bytes only.

In particular a venue-evidence verifier is selected independently of the
package. Its request fixes the venue, whose identity carries the finality rule,
the record kind and subject, and an inclusive index range (§13.1); the reader
derives a checkpoint's own-prefix restriction (earlier indices and lower
same-operator sequences at its index) from the answer (§13.3). Successful
authentication must establish exactly that request's complete records, their
witnessed indices and, for publications, the venue's order within an index
from retained evidence, including an authenticated empty result where
applicable. A proof
of inclusion alone, a latest-value response, a caller-supplied completeness
flag, or a response for another range cannot meet that contract. Unsupported
venue evidence, a failed source, missing history or a local budget refusal
leaves the read unresolved. The package must not choose the verifier, a trust
anchor or an executable decoder. No finality rule or new authority is created
here. Section 13 fixes the answer such a verifier returns, and a venue profile
(for Ergo, [venue-ergo](venue-ergo.md)) fixes the evidence that establishes it.
The verifier takes that evidence from suppliers it uses itself, as the profile
permits; a package carries none.

Only after deriving and verifying the complete closure may the reader give
the verdict its requested read permits. Package decoding or local replay
alone gives no currentness, complete-zero-balance, spendability or exclusion
claim. Retain the evidence needed to reproduce a verdict, not the verdict in
place of it. A local experiment limited to one configuration and checkpoint
may refuse other package shapes; that is unsupported coverage, not a judgment
against otherwise valid protocol evidence.

**No certificate encoding.** A verdict's certificate is its closure: the
objects this section derives from the requested read and the existing rules,
with the venue answers the reader's own verifier establishes. To show another
reader a verdict, a reader hands it a package of the closure's objects, which
it must still hold to do so (C2.10.13). The receiving reader derives the same
dependencies, makes its own venue reads and judges as any reader does. A
stranger shown one excluded checkpoint is handed that verdict's closure; where
§9.1 applies, its compact evidence replaces the target's trail with §9.1's
dependencies intact. Section 9.1 is the only compact replacement of part of a
closure. The smaller set C2.10.13 lists shows the operator's provable fault;
the excluded verdict also needs §9.1's dependencies.

In v3, C4.6's complete restoration package is likewise a §12 package of the
closure's objects together with the reader's own venue reads. The venue
evidence behind those reads is retained by the reader's verifier (§13.2) and
passed on only as a supplier the venue profile admits; its continued
availability is the venue's and its suppliers', not the package's.

**C0a cost and replacement.** This replaces ad hoc evidence-object transport
with one source-neutral inventory over existing byte formats. It costs 23
fixed bytes, nine bytes per item, and one SHA256 per payload for canonical
ordering; inner verification remains necessary. It adds no package signature,
publication requirement, proof relation or trusted certificate issuer.
Nested dependency copies would repeat evidence and impose recursive parsing;
serialized dependency assertions would duplicate the rules that derive them.
This frame leaves all v2 behavior unchanged.

No certificate encoding is added, and four kinds are removed. A certificate
encoding would have to serialize the dependency graph §12.1 already derives,
or assert a range's completeness, which only the reader's own verifier
establishes. Either would duplicate rules or add an authority. The kinds
removed are 5 (a header), 8 (one signed-terms field), 9 (a publication) and
11 (venue evidence). Kinds 5 and 8 duplicated a trail of count zero. It
carries the header and every scoped field for 29 framing bytes, and a field it
leaves empty costs 68 bytes: about 4.5 MB at §8's largest scope, beside the
header itself. A field one trail of the segment lacks is supplied by any other
trail of the segment that carries it. Kinds 9 and 11 would have given a
package a second path to the record. Kind 2 stays because it names the read's
selection under the operator's signature; its standing still comes from the
range answer (§13). A reader authenticates a venue profile's evidence itself,
whatever supplies it (venue-ergo §3's header work from the anchor, §4's
section roots), within §3's light-client limit. Packaging that evidence would
add an encoding without adding evidence.

## 13. Record-range evidence

C2.10.13 reads the record over a range: the complete set of records the venue
holds for one subject between two witnessed indices, with each record's
witnessed index and the venue's order within an index. Section 12.1 requires
the venue-evidence verifier to establish exactly that for the reader's request.
This section fixes the request, the answer the verifier returns and the rules
by which the reader consumes it. It is source-neutral: which venue evidence
establishes an answer, and how, is the venue profile's. The selected profile
is the [Ergo venue profile](venue-ergo.md); another venue needs a profile of
its own, which is a new venue identity, not a reading of this section.

### 13.1 Requests and answers

```text
rangeBytes = "moe/pool/v3/range" || venue[32] || u8 recordKind || subject[32] ||
             u64 fromIndex || u64 toIndex || u32 count || entry_1 ... entry_count
entry_i    = u64 index || u64 ordinal || u32 length || record[length]
```

The context is literal ASCII and integers are unsigned big-endian. The request
is the 81 bytes after the context: the venue identity every scoped backing
declares (C2.10.2), one record kind, its subject and an inclusive index range
with `fromIndex <= toIndex`. `toIndex` must be witnessed under the venue's
finality rule (C2.3.2) when the answer is computed. Count may be zero: an
answer with no entries is the authenticated statement that the venue witnessed
no object of that kind for that subject in the range.

Each entry carries the object's witnessed index and an ordinal field. For
kind 4 the **ordinal** is the object's position in the venue's order within
that index, one total order over every object the venue witnessed at the
index, so that the ordinals of publications of different backings at one
index compare as the venue's order (C2b.3.2, C2b.4.2). For a chain that order is transaction
order, then output order within a transaction. The venue profile fixes how
the ordinal is derived and, for an object it reassembles from several pieces,
fixes the object's index and ordinal as a function of the pieces' positions;
distinct objects have distinct ordinals, reassembled or not. Kind-4 entries
are strictly increasing by `(index, ordinal)`. No rule reads the venue's
order within an index for kinds 1–3, so their ordinal is zero and their
entries at one index are in ascending unsigned order of their record bytes,
identical records adjacent, which keeps the answer canonical: a source that
establishes only the index can answer for them, while one that cannot
establish the order within an index cannot answer for kind 4. Entries lie
within the range and are non-decreasing by index.

| Kind | Subject | Record |
|---|---|---|
| 1 commitment | operator key | `u64 sequence \|\| root[32] \|\| operator[32] \|\| signature[64]`, exactly 136 bytes |
| 2 replacement | backing name | `backing[32] \|\| u8 role \|\| successor[32] \|\| predecessor[32] \|\| u64 effective \|\| signature[64] \|\| successorSignature[64]`, exactly 233 bytes |
| 3 revocation | obligor key K | `obligor[32] \|\| signature[64]`, exactly 96 bytes |
| 4 publication | backing name | Section 6 `publicationBytes`, at most 131914 bytes |

These are the existing records as the venue carries them, after the venue
profile's reassembly and before any decoding under the construction. Their
signed messages are unchanged; for readers they are: a commitment signs
`"moe/commitment/v2" || u64 sequence || root[32]` under `operator`, the
reference's current context, which pool-v2 §9 leaves the implementation free
to version, a versioned context being a new record kind here; a replacement's
rule holder and its successor each sign `"moe/replacement/v1" || backing[32]
|| u8 role || successor[32] || predecessor[32] || u64 effective`, with role 1
the operator (C2.5.1–2); a revocation signs `"moe/revocation/v1" ||
obligor[32]` under K (C2b.1). The kind-4 bound is §6's 92 framing bytes over
its largest body, a kind-6 record of 131822 bytes. All signatures use
pool-v2's strict Ed25519 rule. This transport changes no record, message or
signature. The four kinds are the closed set this section answers; other
venue objects, such as C3's bundle commits and the state-reading extension's
locks and refusals, are not answerable under this frame.

An entry is an object the venue witnessed at the subject's location; nothing
here says it is a record the venue holds or a valid one. An object that does
not decode as its kind, does not verify, or names a subject other than the
answer's is not a record under C2.3.3 and is disregarded by §13.3. The answer
carries it nonetheless, because the verifier applies no signature, sequence,
kind or content rule: the construction's rules are applied in one place, by
the reader. The venue profile attributes objects to a kind and subject by
their location and shape at the venue and reassembles their exact bytes. That
attribution rule, like the finality rule and the lag (C2.3.2, C2.3.5), is part
of what the venue identity names; two attribution rules are two venues. The
profile omits an object only where it does not reassemble to the profile's
shape, or its length is not the kind's exact length (kinds 1–3) or exceeds
the kind's bound (kind 4), since no such object can be a record of the
construction. Two entries with identical bytes at two positions are two
witnessings of one object.

The exact size is `102 + sum(20 + length)` bytes. Before allocating or decoding
entries, a reader checks the context, the request against the request it made,
`fromIndex <= toIndex`, a local total-byte budget and entry budget, every
entry's index, length and boundary, the order of entries — by `(index,
ordinal)` for kind 4, by index and record bytes with a zero ordinal for kinds
1–3 — and the exact end. Count
must fit the remaining bytes even for empty records, and arithmetic must not
wrap or round through unchecked machine numbers. Readers own answer bytes
before asynchronous work and refuse shared mutable storage. Encoders apply
the same checks before allocating output. Budgets are local processing
limits, not consensus bounds; exceeding one leaves the read unresolved.
Objects a stranger publishes at a location cost the reader bytes and
signature checks, priced by the venue's publication cost; a budget the
reader later raises leaves nothing unresolved for good.

### 13.2 What the verifier establishes

An answer is the output of the venue-evidence verifier the reader independently
selected (§12.1), computed under the reader's control from retained venue
evidence for exactly this request. The verifier establishes that `toIndex` is
witnessed under the venue's finality rule and that the answer holds every
object the venue profile attributes to the kind and subject at an index in the
range, with its exact bytes, witnessed index and ordinal, and nothing else. It
establishes this from evidence it can authenticate over the whole range, so
that absence is proven by exhaustion rather than reported: a source's word on
its own completeness, an inclusion proof, a latest-value answer or an answer
for another range does not meet the contract. A verifier that cannot establish
this — unsupported evidence, a failed or incomplete source, an unauthenticated
order, an index not yet witnessed, a local budget — returns no answer, and the
read is unresolved (C2.10.11). It retains the venue evidence it consumed, so
the answer can be reproduced (C2.10.13). The venue's lag (C2.3.5) and its
current witnessed index are constants and reads of the venue profile, not
fields of this frame; a reader asking about the present sets `toIndex` from
that read.

The frame carries no signature and creates no authority. Supplied by an
operator, a replica, a package or any other party — a §12 package carries no
venue evidence (§12) — it is not an answer and establishes nothing, as a
cached verdict establishes nothing. A reader's own components, including a separately contained decoding
process, are under its control; the frame is their interface and the form of
conformance vectors. A reader may keep its own answers for reuse across reads
of one request while the venue's finality rule stands, beside the evidence
that reproduces them; a reorganization past the declared depth is the venue's
failure, not a fact an answer survives.

### 13.3 What the reader derives

**Commitments (kind 1).** Which commitments the venue holds is C2.3.3 read
index by index, from the records alone. Among the entries at one index that
decode, name the subject and verify, a commitment is **held** where its
sequence exceeds every sequence held at earlier indices. The held commitments
at one index are read in ascending sequence order, which is the same-index
sequence order of C2.3.4, C2.10.4 and C2.10.13. Two such records with one
sequence at one index are one held sequence, the lesser record in unsigned
byte order standing for it, the first of the two in a canonical answer. The
highest sequence held before an answer's first index is zero where
`fromIndex` is zero. Otherwise it is either the highest the reader itself
derived through `fromIndex − 1` from the adjacent earlier answer for the same
venue, kind and subject, or the sequence of a commitment the reader has
already established as held at `fromIndex` itself, whose lower same-index
sequences belong to its own prefix and were read with it. A prior highest
supplied by anyone else is not evidence. A held commitment's witnessed index
is its entry's index. A checkpoint's own record prefix (C2.10.4, C2.10.11) is
therefore the held commitments at earlier indices and those with lower
sequences at its own index. A sequence between held sequences is one the
record moved past (C2.3.3, C2b.4). A record that is not held supplies nothing to the record: no state, no hole,
no era. Whether two roots at one sequence, at one index or at two, are the
operator's provable fault is invariant 22's question over the records
themselves, not this rule's.

**Replacements (kind 2).** A record's identity is the SHA256 of its signed
message, the value its successor names as predecessor; two entries with one
identity are one record, witnessed at the first, whatever their signature
bytes. A backing's chain is C2.5's walk over the kind-2 answers for the
backing from index zero. A record counts where it decodes, names the backing,
has role 1 and verifies under the replacement-rule key the backing's terms
declare and under its successor; a backing whose terms declare no rule has no
chain beyond its original operator (C2b.5). The lead floor (C2.5.3), the
strictly-later rule (C2.5.4) and supersession (C2.5.5) read the record's
witnessed index, and two records witnessed at one index resolve by the lesser
identity. The party in force for the
backing at every index in the range follows.

**Revocations (kind 3).** K is revoked from the index of the first entry that
decodes, names K and verifies (C2b.1); later entries are copies. An answer
with no such entry over `[0, t]` establishes that K is not revoked at `t` on
this venue; an answer over a shorter range establishes that for its range only.

**Publications (kind 4).** A backing's publications are read under C2b.3.2 in
venue order: by index, then ordinal, and across the answers for every scoped
backing where a rule reads them together, as C2b.4.2's adopted block does.
Each is judged at its entry's index against the record strictly before that
index. An entry that does not decode under §6, whose routing backing is not
the subject, or whose statement names another backing has no force and is no
evidence; an acceptance's or withdrawal's routing backing is checked beside
the demand it names (§6), which this answer does not resolve. Identical bytes
at two positions are one publication, with force at the first index at which
it has any. A request's index for the count is the first entry index of its
identity (C2b.5.2).

The range each read needs is the rule's, not the frame's: the snapshot and the
clock read the party in force's commitments from the last valid carrying
checkpoint to `t` (C2.10.13, C2b.6.1), anchored on that checkpoint as above;
the adopted block reads publications from the adoption index through `r`
(C2b.4.2); the chain and revocation are read from index zero. A read that
spans a change of the party in force reads each operator's held commitments
and the chain that places it in force. Answers establish the record and
nothing else: no directory, snapshot, trail, classification, term, force or
verdict, and every other item of C2.10.13 still applies.

**C0a cost and replacement.** This fixes the answer form §12.1 deferred, over
the existing record encodings and the existing venue rules C2.3.3–4, C2.5,
C2b.1 and C2b.3.2. It costs 102 fixed bytes and 20 per entry beside the
records themselves. It fixes the reading of C2.3.3 within one index, ascending
sequence from the records alone, and carries the venue's order within an index
for the rules that read it; it adds no signature, identity, authority or
publication requirement. A signed answer would add a trust anchor the reader
must select and a party whose word replaces the record. An answer carrying
only held or decodable records would apply the construction's rules inside the
venue profile and let two profiles read one record differently. A held flag or
an asserted prior sequence would be a redundant assertion with a consistency
rule. One frame over several subjects would add parser states without adding
an order the ordinal does not already carry. Reading one index's commitments
in the venue's order rather than by sequence would let the position of a
transaction in a block decide which of an operator's commitments the record
holds; the sequence rule answers alike from the records, as C2.5.5 does. A
venue profile fixed before its source is shown able to read complete ranges
would assume evidence not yet demonstrated; the Ergo profile was selected
after its verifier read complete ranges from real mainnet sections under
headers the reader verified itself. Every v2 byte and rule is unchanged.

## 14. Replay, retention and resource bounds

This section consolidates how a reader replays and what it must hold. It
restates rules fixed elsewhere and adds requirements on kept replay state
and streamed input. The only frame change it records is §12's
u64 item length.

**Complete replay.** A valid verdict about a backing (C2.10.13) rests on
replaying each segment its read needs from that segment's seed. The reader
authenticates the served trail (§10.1, §12.1) and replays its events in
position order under §§3–7, over the imported closure (C2.10.5–7) counted
once (C2.10.6). It checks adopted events against the adopted block it derives
from the record (C2b.4.2, §10), and classifies checkpoints under C2.10.11–12.
Reads that need force derive it from the record over the ranges §13.3 names
(C2b.3.2). A successor segment starts its own note tree and positions, but
its imports are replayed from their own evidence, not read from a
predecessor's snapshot. Four verdicts read less, as their rules say: an
exclusion from a deterministic failure found before replay completes
(C2.10.11), an exclusion for non-extension decided by the evidence recurrence
(§7.1), §9.1's compact intrinsic exclusion in place of the target's trail, and
a term lapse read from scope, terms and the record (C2.10.4). A first
valid verdict therefore costs time and bytes linear in the closure's events.
The venue ranges of §13.3 are linear in the deployment's age, since the chain
and revocation are read from index zero. Nothing in v3 proves a checkpoint's
history succinctly.

**Resumed replay.** Replaying records 1 through n yields the note roots, the
spent set, the totals, standing demands and recovery effects, and both chains.
These are a deterministic function of the replay inputs, which are every
input the replay reads. They include the configuration, the segment's header,
its scoped terms, the imported closure, the adopted block and the opening
index, the venue answers the replay reads through position n, the reader's own
selected verifier, and the records. Venue answers under the finality rule do not change as the record
grows. Index-relative conditions are read at the judged checkpoint's own
index, such as a lock's own bound (C3.7–8). So kept state records each such
condition with its bound, not with its standing at an earlier checkpoint. It
may drop one only where it can stand at no later index, as a lock past its
bound cannot.

A reader that replayed a checkpoint's trail itself may keep that state. For a
later checkpoint of the same segment that extends it (§7.1), with the same
replay inputs apart from the new records and the venue answers after them, it
may replay only positions n + 1 onward. Before resuming, it does two checks.
First, the whole kept state must match one SHA256 digest held in storage
only the reader writes. Second, the note and spent roots, the totals and both
chain values at n, recomputed from that state, must match the checkpoint's
authenticated snapshot preimages (`historyHash_n`, `evidenceHash_n` and the
totals), not values stored beside the cache. On any mismatch it discards the
kept state and replays in full: a damaged cache never grounds an exclusion
(C2.10.11). The verdict is the one full replay gives. Kept state
is a local cache. It is not evidence and is never served, and the evidence
bytes that produced it are still retained (C2.10.13).

**Kept classes.** A reader may also keep the class it derived for each
checkpoint: valid, excluded or lapsed, with its ground or a lapse's clock
record. It keeps the class beside the checkpoint's held commitment, its
witnessed index and the authenticated snapshot preimage of the backing it
classified it for. It may reuse the class in a later read only under the same
reader rules, configuration, venue identity and verifier circuit identities.
The reader rules are the specification revision and the implementation
version it classified under; any change to either discards the kept classes.
The reader must also still retain the evidence that grounded the class and its
dependencies (C2.10.13). A class is a function of the checkpoint's bytes, the
record strictly before its index and the lower same-operator sequences at that
index (C2.10.11). Finality fixes that record,
so reclassifying the checkpoint later under the same rules gives the same
class. Before reusing a kept class, the reader does three checks:
- the whole-state digest check above;
- the venue holds the same commitment at the same index;
- the kept snapshot preimage authenticates again against the checkpoint's
  directory.

A kept valid class at position m, at or below its segment's kept tip n,
supplies state read at m. Its stored chain values at m must equal the
re-authenticated snapshot's `historyHash_m` and `evidenceHash_m`, and its
stored totals at m the snapshot's totals. The digest vouches for its other rows
at or below m. Using the state at n, for a later checkpoint's records or with
none, also requires §14's resumption check, which recomputes the roots,
totals and chains from the kept state against the authenticated snapshot.
An excluded checkpoint supplies no state (C2.10.12), and neither does a lapsed
one (C2b.4.1). So their kept classes stand on the three checks alone. On any
mismatch the reader discards its kept state, classes included, and classifies
again from its retained evidence. A class received from another party is not
reused, nor is one whose evidence the reader no longer retains: a cached
verdict is not a certificate (C2.10.13). An unresolved checkpoint has no class
to keep. A kept class is a local cache, like kept state.

**Incremental retrieval.** A reader that holds a checkpoint's authenticated
trail of n records may assemble a later checkpoint's trail from two parts. It
fetches the later trail's head (header, scoped terms and count) and the
records after position n, then checks §10's frame over the assembled bytes to
the exact end. The retained trail is not a byte prefix of the later one,
because the count precedes the events. The reader continues §7's evidence
recurrence from its own `evidenceHash_n`; the history chain follows from
replay. The assembled trail is the complete trail, which §10.1 and §12.1 read
unchanged. This is the chunked transport §10 permits. There is no suffix
frame or base reference, and a supplier's stated position or length
authenticates nothing. A later checkpoint that does not reproduce by
continuation is judged from a complete trail from position 1 like any other.
A failed continuation is no evidence about it.

**Streamed input.** Budgets bound memory and per-object work (§§8, 10, 12,
13). A total-byte budget over a trail, package or closure also bounds the
history a reader can judge. So a reader meant for long-lived pools streams
evidence and keeps its replay state in storage, while the per-record (§5,
§10) and header (§8) bounds still hold. This extends the rule that readers
own their input (§§12, 13.1) to input read more than once. Such a reader
first copies the supplied bytes into storage only it writes. Otherwise it
takes one SHA256 of the whole input in the first pass, and uses no result of
a later pass until that pass's SHA256 equals it. A refusal on any budget stays
unresolved, never exclusion.

**Retention.** Retention is the holders' and the backer's (C2.10.13), and
replication is the remedy for withheld evidence (Construction §C0b, C2.7.2).
The complete closure is held by whoever replicates it: the operator, the
backer, independent readers, and holders that choose to. To redeem in a gap,
a holder obtains the trail and ancestry from replicas (C2b.3.3). A note whose
history nobody replicated waits (Construction §C2b.3). Nothing here requires a
holder to keep the complete closure. A holder's own evidence is its note
openings, its seed and capsules, its receipts, its own requests and
publications, and the last state it was given (C2.7.2).

**Format bounds.** Apart from local budgets, a pool's life is bounded only by
its u64 positions and sequences (§§7–8) and by 2^32 leaves in each segment's
note tree (§2). Per-object u32 counts remain: a package's item count, a
directory's entries, a range answer's entries, and a header's scope bound.
None of them bounds a lifetime, because a reader can split the objects across
packages, commitments or requests. With u64 item lengths (§12), the package
frame bounds neither a trail nor venue evidence.

**C0a cost and replacement.** This adds no field, identity, relation or
authority. The u64 item length costs four bytes per item. It removes a
per-trail ceiling of `2^32-1` bytes, which §12's u32 imposed: about 276,000
spend-sized events per segment at §11.4's 14,656-byte proof length. Four
alternatives were rejected:
- *A succinct history relation*, a seventh relation proving a checkpoint's
  state from genesis. It would replace a reader's proof checks and trail bytes
  with one proof per checkpoint. It would not remove §13.3's venue ranges,
  linear in the deployment's age, or a restoring wallet's capsule scan,
  linear in statements. Its step circuit would have to:
  - verify all six relations recursively;
  - compute SHA256 over every proof's bytes;
  - hash up to 257 spent-set nodes per nullifier (pool-spent C1.2.8–9);
  - check the Ed25519 authorizations, which are non-native to BN254;
  - be proved by the operator at every checkpoint.
- *A trusted or signed snapshot* would replace independent verification by a
  party's word: a cached verdict is not a certificate (C2.10.13).
- *Keeping u32 and splitting a segment's trail across items* would need a
  chunk frame and consistency rules.
- *A suffix trail frame* was already rejected in §12.1. Incremental retrieval
  assembles the same trail without one.

Kept classes and deciding non-extension by the evidence recurrence (§7.1) add
no field, identity or authority either. Kept classes replace reclassifying
every held checkpoint on each read; a replay of each non-extending checkpoint
from its segment's seed was part of that cost. Deciding non-extension by the
recurrence removes that replay on a first read too. Without it, an operator
that commits C non-extending checkpoints makes its own readers replay up to
C × N records for N records of history. The bytes stay C × N: each such
checkpoint's complete trail from position 1 is still fetched, stored and
hashed. Where a replay would have stopped earlier as unresolved, the
checkpoint is now excluded, and the ground a reader reports can change. Three
alternatives were rejected:
- *Keeping only valid classes* would reclassify every excluded checkpoint on
  every read.
- *Replaying a non-extending trail to find its first failing position* costs
  the C × N replay for the same class, or for an unresolved one.
- *A compact non-extension certificate*, the chain value at n with §7.1's
  suffix opening at 96 bytes per later position, would also cut the bytes. It
  would be a new permitted replacement like §9.1, with its own dependency
  rules. The replay was the cost this removes, so it is deferred.
