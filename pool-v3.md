# The shielded pool, construction v3

## 1. Status and scope

This document fixes the successor's six proof relations, public-input orders
and statement, authorization, publication, snapshot, receipt, segment-header
and fault-evidence records, plus served-trail transport,
including the history and exact-evidence chains. It implements the selected [recovery](pool-recovery.md),
[delivery](pool-delivery.md) and [transfer/fee](pool-fees.md) contracts without
reinterpreting any [v2](pool-v2.md) bytes, notes or keys.

**This is an incomplete construction, not an adoptable profile.** There is no
adopted v3 configuration hash or approved circuit/key identity yet. Section 11
fixes configuration and signed-terms bytes for conformance only.
No backing may declare `moe/pool/v3` on the basis of this document, and no
runtime may accept its statements as v2. A synthetic domain used to test these
relations is not a construction domain. Conformance tooling may compile and
prove the relations below; passing it does not establish runtime conformance.

Before adoption this document must also approve the configuration's
source/helper/toolchain/bytecode/key identities,
complete-certificate encoding, replay/import rules and resource
bounds beyond the records below. The [fault](pool-fault.md) and [spent-set](pool-spent.md) contracts
remain binding requirements for that work. None is replaced by a proof check.

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

Use pool-v2 §12's selected UltraHonk proof system and toolchain for this
conformance milestone. A verifier selects the committed key by statement
kind, never by public-input count or a key supplied with a proof. In
particular, spend and burn both have 15 public inputs; neither proof can
verify as the other relation. No approved v3 key or configuration hash is
committed here; §11 fixes the configuration frame for conformance.

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

This section fixes bytes for conformance before configuration adoption; it
does not assign a v3 domain or authorize a v3 backing. All contexts below are
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
imports, last-valid continuity and replay; unsupported verifiers, resource
failures and programming failures remain unresolved, never exclusion.

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
tree, second history or new certificate transport is introduced. The final
configuration, complete certificate encoding and replay integration remain required
before v3 adoption.

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
Complete certificate formats and replay/import/adoption rules remain required.

**C0a cost and replacement.** This replaces the deferred successor header
layout by reusing v2's fields and bounds with one new context. No scope root,
adoption index, silence duration, signature or primitive is added: those
values are already derived from the scope or the record. At maximum scope
the header is about 8.50 MiB; that is a format bound, not a phone or transport
budget. The alternative of adding such derived fields would require extra
consistency rules without authenticating missing history. Every v2 header
and identity remains unchanged; approved v3 configuration identity and
adoption remain unset.

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
verdict. The snapshot's history hash and totals are authenticated assertions,
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
the terms encoding, name function or signing message, or authorize declaring
v3 before §1's configuration and adoption work is complete. An outer codec
treats the terms and signature as supplied bytes; a reader must independently
decode them, derive the expected backing name, verify K's strict signature,
and check the terms for every scoped entry against the record. Supplying an
opaque byte field does not satisfy those checks.

The count n is a u64, including zero. Events occupy consecutive positions
1 through n, without repeated position fields. A trail is cut at the checkpoint
being read: it carries exactly those n records, with no uncommitted tail.
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
closure, replays every event and reproduces every scoped snapshot, with lapse
priority and C2.10.11–13's dependency rules. Missing dependencies remain
unresolved; they are not empty openings or permission to fall back to an older
checkpoint. The complete certificate/dependency format and replay integration
remain prerequisites in §1.

**C0a cost and replacement.** This replaces the deferred outer served-trail
frame by composing the existing header, signed terms and record bytes. It adds
29 fixed framing bytes, 68 per scoped terms field (including the existing
64-byte signature), and four per event. It adds no roots, signatures, proof
relations or authority. Repeating backing names, positions, force indices or
adoption flags would introduce redundant assertions and consistency rules;
omitting scoped terms would leave silence/force checks underdetermined. Inner
terms and record checks remain where their definitions place them. Every v2
byte, finality rule and private note opening remains unchanged.

## 11. Configuration and backing evidence before adoption

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
Poseidon2 helper source. Proof system, verifier target, toolchain, common
relations, public-input orders, record bounds and spent-root rules are fixed
by this construction and its referenced contracts; the frame is not a way
to override them. No operator, venue, backing, segment or mutable authority
enters the configuration.

An implementation holds an independently selected manifest of source,
shared-helper, compiler/backend/version, verifier-target, bytecode and key
identities for all six relations. It checks source/toolchain identities,
compiles the relations together, derives keys under §4, and compares the
result with that manifest and the configuration. It refuses missing,
reordered or mismatched identities, including for relations absent from a
particular trail. A served package may supply the configuration preimage,
but cannot select that manifest or a key. Routing uses the kind's own checked
key, never the public-input count. Hash equality alone proves neither that a
relation is correct nor that its setup is trustworthy.

Conformance tooling may use a manifest of reviewed candidate identities and
the corresponding configHash to remove synthetic-domain fixtures. Such a
hash remains a **candidate domain**, not an adopted construction. Successful
byte, source, key or proof checks cannot enable backing declaration. Section
1's adoption prerequisites remain binding; no adoption flag in a manifest,
configuration, terms or served package can close them. Final adoption must
identify the approved configuration and artifacts in this document after
complete-certificate, replay/import and resource requirements are fixed.
Changing those semantics before adoption requires renewed review and
conformance; candidate notes carry no migration or spendability promise.

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
`moe/pool/v3`, configuration to equal the checked §11.1 hash, and venue to
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
additional bytes. It replaces deferred configuration framing, not an adopted
configuration. Constant-root terms reuse one name and one existing signature;
there is no certificate signature, configuration authority, registry of
approved issuers or extra proof relation. A variable list, duplicate source
hashes inside the domain or a mutable key service would add parsing or
operating costs without replacing manifest verification. Omitting a recovery
key would let a local-only check silently certify an incomplete configuration.
The manifest/source checks are local verification work, not a publication
service or a proof of currentness. V2 bytes, keys and runtime support stay fixed.
