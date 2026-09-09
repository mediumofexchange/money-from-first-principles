# The shielded pool, construction v3

## 1. Status and scope

This document fixes the successor's six proof relations, public-input orders
and statement, authorization, publication, snapshot and receipt records,
including the history and exact-evidence chains. It implements the selected [recovery](pool-recovery.md),
[delivery](pool-delivery.md) and [transfer/fee](pool-fees.md) contracts without
reinterpreting any [v2](pool-v2.md) bytes, notes or keys.

**This is an incomplete construction, not an adoptable profile.** There is no
v3 configuration preimage, configuration hash or approved circuit/key identity
yet. No backing may declare `moe/pool/v3` on the basis of this document, and no
runtime may accept its statements as v2. A synthetic domain used to test these
relations is not a construction domain. Conformance tooling may compile and
prove the relations below; passing it does not establish runtime conformance.

Before adoption this document must also fix the full configuration (including
delivery profile identity), source/helper/toolchain/bytecode/key identities,
segment headers, served-trail/certificate encoding, replay/import rules and resource
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
verify as the other relation. No v3 key or configuration is committed here.

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
configuration, segment/trail encoding and replay integration remain required
before v3 adoption.
