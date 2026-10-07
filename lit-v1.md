# Lit notes, construction v1

## 1. Status and scope

This document fixes the bytes of **`moe/lit/v1`**, the [transparent
profile](extensions.md#the-transparent-profile): lit notes under the pool's
rules. It fixes the note and its derived output, the seven statement records
and their signatures, the signed objects and publications, the history,
evidence, snapshot and receipt frames, the segment header, fault evidence,
the served trail, the evidence package, the configuration, the root terms and
the wallet's key derivation. Everything else is the pool's: this document
instantiates the [authority](pool-authority.md), [recovery](pool-recovery.md),
[fault](pool-fault.md) and [spent-set](pool-spent.md) contracts as
[pool-v3](pool-v3.md) reads them, and says where a rule reads differently
because openings are public. It replaces the [delivery](pool-delivery.md)
contract and changes the [transfer](pool-fees.md) shape (§§8, 11).

**Status: draft until adopted.** No backing may declare `moe/lit/v1` before a
decision adopts it, which needs the reference implementation's conformance
with this text. Until then an edit here is a reviewed change, not a new
version. Once adopted, everything here is fixed by the name, as pool-v3 §1
fixes v3: a change to a byte, identity or verdict rule is `moe/lit/v2`, and a
backing moves to it by successor (Construction §C1.6).

A lit backing's scope holds lit backings only, and each scope has its own
segments, history, commitments and directory (Construction §C1.2: a scope is
per operator, construction domain and venue). A lit record is no pool record,
and no reader accepts one as the other. What lit shares with the pool is the
venue and its record ranges (§10), the root-terms frame (§9), the commitment
and directory frames, the receipt and recovery clocks, and the replay model
(pool-v3 §14).

Notation follows pool-v3: `||` is byte concatenation, contexts are literal
ASCII with no length prefix, integers are unsigned big-endian of the stated
width, and `SHA256` is SHA-256 over the concatenation. No context in this
document is a prefix of another or of any context the pool or the reference
declares. A **key** is a 32-byte Ed25519 public key that is canonical and not
of small order; a **signature** is 64 bytes under pool-v3 §11.2's strict
Ed25519 rule. A **value** or **quantity** is a `u64` and is positive; sums are
widened and never wrap.

## 2. Notes and the derived output

A **note** is an opening `(domain, backing, value, owner, rho)`: the
configuration hash (§9), a backing name, a positive value, the owner's key and
32 bytes of randomness. The receiver generates the owner key and keeps its
signing secret (invariant 25); the payer learns only the public key.

```text
opening = backing[32] || u64 value || owner[32] || rho[32]        (104 bytes)
cm      = SHA256("moe/lit/v1/note" || domain[32] || opening)
nf      = SHA256("moe/lit/v1/nullifier" || cm)
tag     = SHA256("moe/lit/v1/tag" || nf)
```

The nullifier is computed from the public commitment alone, so it is fixed when
the note is created and appears in the clear when the note is spent. The tag is
pool-recovery C3.1's public name of a note; here anyone can compute it, so it
hides nothing, and it keeps the pool's lock and spent-tag rules unchanged.

**An output is named by the statement that creates it.** A statement carries
each output as `backing[32] || u64 value || owner[32]` (72 bytes), with no
randomness and no commitment. Every reader derives both:

```text
issued output:   rho = SHA256("moe/lit/v1/rho/issue" || backing[32] || u64 quantity ||
                              owner[32] || nonce[32])
spent outputs:   rho_j = SHA256("moe/lit/v1/rho/spend" || u8 k || nf_1[32] ... nf_k[32] || u8 j)
```

For an issue the fields are its statement's (§3). For every other statement
that creates outputs, `nf_1 ... nf_k` are the nullifiers it consumes, in its
input order, and `j` is the output's position from zero. Each output's `cm`
then follows from its opening. Because no statement can carry another value, the
derivation is a validity rule, not a wallet convention, and three things hold:

- **No other statement can create the same output.** A spend-side output binds
  the nullifiers its statement consumes, which only one admitted statement can
  ever consume; an issued output binds an issuance only **K** can sign; the two
  derivations have distinct contexts. Openings are public, so with free
  randomness anyone who saw a statement before admission could create one of
  its outputs first and have the original refused on every retry.
- **Outputs are stable across segments.** Neither derivation reads the segment.
  A statement signed again after a handover, a return or a lapse keeps its
  outputs, so a payee's expected output does not change, and an issuance
  that **K** signs again for a new segment with the same nonce cannot be
  admitted twice: its second output would equal the first.
- **Outputs within a statement are distinct**, since `j` differs.

Every statement that creates outputs consumes at least one note or is an
issuance (§3), so every derivation reads a value nobody else can spend.

## 3. Statements and records

```text
statementBytes = "moe/lit/v1/statement" || configHash[32] || u8 kind || body
statementHash  = SHA256(statementBytes)
recordBytes    = u32 statementLength || statementBytes[statementLength] ||
                 u32 authorizationLength || authorization[authorizationLength]
```

The statement's identity, `statementHash`, excludes the authorization. An
exact replay is the same statement bytes, and returns the prior answer whatever
its signature bytes (invariant 26). The two length fields let a reader split a
committed record without decoding it (§5); the record splits only where
`8 + statementLength + authorizationLength` is its whole length. In the bodies below, `input` is an
`opening` (§2) and `output` is the 72-byte output of §2.

| Kind | Body | Body length |
|---|---|---|
| 1 issue | `segment[32] \|\| backing[32] \|\| u64 quantity \|\| owner[32] \|\| nonce[32]` | 136 |
| 2 spend | `segment[32] \|\| u8 k \|\| input_1 ... input_k \|\| u8 m \|\| output_1 ... output_m` | 34 + 104k + 72m |
| 3 burn | `segment[32] \|\| u64 quantity \|\| u8 k \|\| input_1 ... input_k \|\| u8 m \|\| output_1 ... output_m` | 42 + 104k + 72m |
| 4 demand | `segment[32] \|\| u8 k \|\| input_1 ... input_k \|\| presenter[32] \|\| u64 instant \|\| u64 deadline` | 81 + 104k |
| 5 withdraw | `segment[32] \|\| demand[32]` | 64 |
| 6 settle | `segment[32] \|\| demand[32] \|\| owner[32]` | 96 |
| 7 request | `input \|\| u64 refresh` | 112 |

The prefix before the body is 53 bytes. `k` is 1 or 2 for kinds 2–4; `m` is 1
through 4 for a spend and 0 or 1 for a burn. Every key field is a key (§1),
every value and quantity positive; `instant`, `deadline` and `refresh` are any
`u64`. `demand` is a kind-4 `statementHash`. No optional field, alternate
spelling or trailing byte is accepted, and the statement length is exactly
53 plus the body length.

| Kind | Authorization | Signed by | Over |
|---|---|---|---|
| 1 issue | 64 bytes | the backing's **K** | `statementBytes` |
| 2 spend, 3 burn, 4 demand | 64k bytes, one signature per input in input order | the owner of that input | `statementBytes` |
| 5 withdraw | 64 bytes | the demand's presenter key | `statementBytes` |
| 6 settle | `u64 acceptanceDeadline \|\| acceptanceSignature[64] \|\| ownerSignature[64] \|\| releaseSignature[64]`, 200 bytes | **K**; the settlement's `owner`; the presenter key | `acceptanceBytes`; `acceptanceBytes`; `releaseBytes` (§4) |
| 7 request | 64 bytes | the owner of the input | `statementBytes` |

The authorization length must equal the table. Two inputs owned by one key
carry one signature each. The statement context and kind byte separate every
signed statement from every other message a key signs, so one key may be an
owner, a presenter key and **K**. Settlement is authorized as in the pool
(invariant 27): the owners' signatures on the demand name the presenter key,
and the presenter key's release signs the settlement. The acceptance carries
the signatures of **K** and of the owner it names (§4). The settlement consumes
the notes its demand names, which it does not repeat.

**Derived values.** A demand's backing is its inputs' common backing and its
quantity their summed value, which must be a `u64` (§7). A burn's backing is
its inputs' common backing. A settlement's output is `(the demand's backing,
the demand's quantity, owner)` at position 0, derived over the demand's
nullifiers in the demand's input order. A request's backing is its input's.

Decoding establishes structure, key encodings and positive values. It does
not establish signatures, membership, spentness, locks, time, conservation or
finality, and a parser cannot classify a well-formed record as an accepted
statement.

## 4. Signed objects and publications

```text
acceptanceBytes  = "moe/lit/v1/acceptance" || configHash[32] || demand[32] ||
                   owner[32] || u64 deadline                          (125 bytes)
acceptanceId     = SHA256(acceptanceBytes)
releaseBytes     = "moe/lit/v1/release" || configHash[32] || demand[32] ||
                   acceptanceId[32] || settlementHash[32]             (146 bytes)
publicationBytes = "moe/lit/v1/publication" || configHash[32] || backing[32] ||
                   u8 publicationKind || u32 bodyLength || body[bodyLength]
publicationId    = SHA256(publicationBytes)
```

**K** signs `acceptanceBytes`. Its `owner` is a key **K** generates (§8), and
that key signs the same `acceptanceBytes` too, so an acceptance shows its
signer holds the owner key's secret (§7). An acceptance is both signatures; with either
missing or invalid it is no acceptance. The presenter key signs `releaseBytes`.
A settlement reconstructs the acceptance
from its own demand and owner and its authorization's deadline, and the release
from that acceptance's identity and its own `statementHash`, as pool-v3 §6
does. Publication kinds, bodies and the routing rule are pool-v3 §6's: 1 a
demand's record, 2 `acceptanceBytes || signature[64] || ownerSignature[64]`
(exactly 253 bytes),
3 a settlement's record, 4 a withdrawal's record, 5 a request's record. The
routing backing must equal the backing the body names or derives (§3): a
demand's and a request's from their own openings. An acceptance, a release and
a withdrawal inherit their demand's, so a release's routing, like an
acceptance's and a withdrawal's in the pool, is checkable only beside its
demand (C2b.3.2, pool-v3 §13.3). The parser bounds
`bodyLength` by the largest valid body before copying: 478 bytes, a demand
over two notes. The largest publication is 569 bytes.

## 5. History, evidence, snapshots and receipts

For a segment `S`, before any local statement and after the statement at
position `i`:

```text
historyHash_0  = SHA256("moe/lit/v1/genesis" || S[32])
historyHash_i  = SHA256("moe/lit/v1/history" || historyHash_(i-1)[32] || statementHash_i[32] ||
                        spentRoot_i[32] || u64 i)
evidenceHash_0 = SHA256("moe/lit/v1/evidence-seed" || S[32])
evidenceHash_i = SHA256("moe/lit/v1/evidence-link" || evidenceHash_(i-1)[32] ||
                        statementHash_i[32] || signatureHash_i[32] || u64 i)
snapshotBytes(b) = "moe/lit/v1/snapshot" || b[32] || S[32] || historyHash_n[32] ||
                   evidenceHash_n[32] || u64 issued(b) || u64 burned(b)      (163 bytes)
receiptBytes   = "moe/lit/v1/receipt" || configHash[32] || S[32] || u64 i ||
                 statementHash_i[32] || historyHash_i[32] || signatureHash_i[32] || u64 after
receiptRecord  = receiptBytes || operator[32] || signature[64]
```

`signatureHash_i` is SHA256 of the exact authorization field, whatever its
length; every valid kind carries one, so there is no empty sentinel. A lit
record has no proof, so the evidence chain, the receipt and fault evidence carry
no proof digest. This is how lit reads C2.10.10's evidence: its digests are the
record's, and a lit record has two. Otherwise pool-v3 §7, §7.1 and §7.2 apply as
written, with these frames. The receipt's signed message is 194 bytes and its
record 290.

`spentRoot_i` is pool-spent C1.2.8–9's root over the nullifiers of the
validated imported closure and positions 1 through `i`, with that function's
contexts unchanged; a statement that adds no nullifier keeps the preceding
root. The spent root keeps absence answerable without replay (invariant 23).
The history binds no note root: there are no anchors, and the output set is a
function of the ordered statements the chain already binds. Lit's supply
objects (invariant 23) are therefore the ordered statement log and the spent
set. A note's membership is read by replay, as every current reader of force
reads it (C2b.3.3); a compact membership proof would need an output root and a
later version. No root has a capacity bound below the `u64` positions.

The receipt drops pool-v3's scope root because the segment identity already
binds the scope (§6), and no lit statement names a scope root.

## 6. Header, fault evidence, trail and package

**Segment header.** Pool-v3 §8's layout with the context `"moe/lit/v1/segment"`:
a 126-byte prefix and 136 bytes per entry, from 262 to 8,913,022 bytes. No
scope root is computed; the segment identity binds the scope.

**Fault evidence.** Pool-v3 §9 with the pair in place of the triple:

```text
faultEvidenceBytes = "moe/lit/v1/fault-evidence" || snapshotBytes[163] ||
                     u64 i || u64 n || previousEvidenceHash[32] ||
                     u32 statementLength || statement[statementLength] ||
                     u32 authorizationLength || authorization[authorizationLength] ||
                     pair_(i+1) ... pair_n
pair_j             = statementHash_j[32] || signatureHash_j[32]
```

Each target field is 0 through 4096 bytes, a transport bound over the largest
valid field (a 583-byte spend statement). The exact length is
`244 + statementLength + authorizationLength + 64*(n-i)`.

**Compact intrinsic exclusion.** Pool-v3 §9.1 applies with these intrinsic
failures in place of its two. The record splits and its statement decodes
canonically under §3, its domain is the configuration's, its authorization has
exactly §3's length for its kind (a wrong length is malformed, not intrinsic),
and one of:

- a signature of its authorization fails strict verification, where the key is
  an input's owner (kinds 2–4) or the scoped backing's **K** from its
  authenticated signed terms (kind 1);
- the statement's own arithmetic fails §7: two inputs with one nullifier, an
  output whose backing no input names, inputs of more than one backing in a
  burn or a demand, unequal widened sums in a spend or burn, or a demand's
  summed value past a `u64`.

A withdrawal's or settlement's signature failure needs the demand, which is
state, so it is not intrinsic; a settlement has no intrinsic failure, its
acceptance owner's signature included, and all three of its signatures are
judged with its demand. Every other case needs its ordinary evidence.

**Served trail.** Pool-v3 §10's frame with the context `"moe/lit/v1/trail"`
(28 fixed bytes), this document's header and records, and record lengths 0
through 8200, two fields at the 4096-byte bound with their lengths. Pool-v3
§10.1 applies with §5's evidence pair. The pair needs only §3's split, so
§10.1's procedure, and §12.1's match of a checkpoint's served trail, cover
every record that splits where pool-v3's cover records that decode: they
authenticate both fields whether or not the statement decodes. A record that
splits but whose statement does not decode under §3, or whose authorization
is not §3's length for its kind, fails replay at its position, as a committed
kind-7 record does, so its checkpoint can be excluded on its complete trail
(C2.10.11). A record that does not split remains inconclusive under §10.1.
The 8200-byte bound is on the record; a field longer than the fault
evidence's 4096-byte bound (above) splits and fails replay, but no fault
evidence carries it.

**Evidence package.** Pool-v3 §12 with the context `"moe/lit/v1/package"`
(22 fixed bytes). Kind 1 carries §9's configuration bytes and kinds 4, 6, 7
and 10 carry this document's snapshot, trail, fault evidence and receipt
record. Kinds 2 and 3, the commitment and the directory, are unchanged: an
operator commits a lit segment exactly as a pool segment, under
`"moe/commitment/v2"`. **An operator key serves one construction on a venue.**
Commitments carry no construction, so a key committing for a pool scope and a
lit scope would share one sequence and one commitment in flight (C2.4.3), each
scope's receipts would read `after` across both (pool-v2 §9, C2.10.9a), and
each construction's readers would need the other's directories (C2.10.13): a
failure of one would reach the other's payments. An operator serving both
constructions uses two keys. A key that commits for both bears those
consequences as its own conduct, and no reader rule refuses it: readers pass
the other construction's checkpoints as non-carrying, by their directories
alone (C2.10.13, C2b.6.1), and a receipt whose `after` names one of them is not
of the receipt's segment (pool-v3 §7.1). A payee checks that its receipt's
`after` is of the receipt's segment before it relies on the receipt (C4.5).

## 7. Admission and validity

Admission, replay, adoption and venue force judge a lit statement by the
pool's rules (pool-v2 §8 as pool-v3 and the contracts amend it), with these
readings. The **output set** and **spent set** are the validated imported
closure's plus the local prefix's, as in C2.10.6–7; there is no forest.

- **Common.** The domain is the configuration's and the segment this one, except
  for a statement of the adopted block, which keeps the binding it was judged
  under (C2b.4.2, C3.7). Every
  backing named or derived is in the segment's scope. An exact resubmission
  returns the original receipt. An input is **live** when its `cm` is in the
  output set and its `nf` is not in the spent set; the inputs' nullifiers are
  distinct. Each derived output's `cm` is absent from the output set.
- **Issue.** **K**'s signature verifies over `statementBytes` under the signed
  terms of the backing, and `issued(backing) + quantity < 2^64`. Effect: the
  output is inserted and `issued` rises.
- **Spend.** Every input is live and every input's tag is under no standing
  lock (C3.7). Owner signatures verify. Every output's backing is an input's,
  and for each input backing the inputs' widened sum equals its outputs'.
  Effect: nullifiers and outputs are inserted together.
- **Burn.** As a spend, with one input backing; any output names it, and the
  inputs' sum equals `quantity` plus the output's value. `outstanding(backing)`
  is at least `quantity`. Effect: as a spend, and `burned` rises.
- **Demand.** Every input is live, its tag neither locked nor spent; owner
  signatures verify; the inputs share one backing whose terms are held, and
  their summed value is a `u64`; the instant and deadline pass C3.3 and C3.8
  at the door. Effect: C3.7's lock.
- **Withdraw.** C3.7's rule; the presenter's signature is over the withdraw
  statement's bytes.
- **Settle.** C3.7's rule: the demand stands, the acceptance verifies under
  **K** and under this owner with a deadline not behind the horizon, the release
  verifies under the presenter key, the demand's nullifiers are unspent and its
  tags under no other demand's standing lock. Effect: the demand's nullifiers
  and the derived output are inserted, and the demand is discharged.
- **Request.** Never admitted. C2b.5.2 counts it where its owner's signature
  verifies and its `cm` is an output of the canonical state, in place of the
  proof and the anchor.

At the venue, C2b.3.2's force rules read the same substitutions: a demand's
inputs must be outputs of the snapshot's state, where the pool reads anchors in
the snapshot's forest, a release's settlement needs no proof, and a release's
acceptance verifies under its owner as well as under **K**.

**What reads differently, named rather than branched.** C3.5's per-segment
`rho_out` and its disclosure count, and C2.10.8's settlement exception, are
replaced by §2's derivation: a settlement signed again keeps its output,
since nobody can create it first. C3.8's taken release cannot occur: equal
settlement outputs need equal nullifiers, so a release whose output was taken
has also lost its notes. For the same reason C3.4's distinct acceptance owner
is not needed; reusing one only links the backer's settlements. That **K** holds C3.4's
owner (Construction §C3: owner values the backer generated) is checked rather
than trusted. Every key in a lit statement is public, and **K** sees the keys
of a holder who presents to it: the presenter key, the inputs' owners, and keys
the spend graph links to them, such as a change key. An acceptance naming one
of them would send the settlement's output back to the holder, and left
unreleased it would read as the holder's lapse under C3.8, so **K** would
never take the claims and never be dishonoured. The owner key's signature on
the acceptance (§4) closes this for every reading at once: only a party with a
key's secret can produce it, and a holder signs no `acceptanceBytes` (§8).
C3.8 reads an acceptance, published or carried by a release, as an answer only
where both its signatures verify. An acceptance whose owner is **K** itself
carries two equal signatures and is valid. C2b.5's request
is the holder's act by its owner's signature, and C2b.7's targeted refusal is in
scope for transfers, since a lit statement shows its keys.

## 8. What a wallet derives

**A payment request names a backing, a quantity and an owner key**, in place
of pool-delivery C4.1's exact output: the payer cannot prepare the output's
randomness, so the receiver names only what it controls. **Each request names
its own owner key**, never one another request named: outputs are public, so a
payer could otherwise present another payer's statement to a reused key as its
own. The payee checks that the statement carries or derives an output to that key, of
that backing and quantity, holds the receipt, and credits each output
commitment to one request only, persisting that before reporting it; final
acceptance needs canonical finality and current spentness, as C4.5 says. There
is no capsule or delivery digest: every output's opening is public.

A wallet root seed is pool-delivery C4.2's 32 random bytes. Under it:

```text
ownerRoot      = HKDF-SHA256(seed, salt=domain, info="moe/wallet/lit/v1/owner")
ownerSecret    = HMAC-SHA256(ownerRoot, backing[32] || u64 i)
settlementRoot = HKDF-SHA256(seed, salt=domain, info="moe/wallet/lit/v1/settlement")
acceptSecret   = HMAC-SHA256(settlementRoot, demand[32] || u64 acceptanceDeadline)
presenterRoot  = HKDF-SHA256(seed, salt=domain, info="moe/wallet/lit/v1/presenter")
presentSecret  = HMAC-SHA256(presenterRoot, tag_1[32] || tag_2[32] || u64 instant || u64 deadline)
```

Each root is HKDF's 32-byte output (RFC 5869, `L = 32`), as pool-delivery's
roots are, and `domain` is the configuration hash (§9).
Each secret is an Ed25519 private seed (RFC 8032), whose public key is always a
key (§1). **Owner indices are per backing**: `ownerSecret` binds the backing
the key receives, so each backing's indices, window and restoration below read
that backing's trails alone, and one seed used for backings in several scopes
or on several venues never names one key for two of them. A wallet allocates
a backing's indices in order for every output of it that it expects, change
included, never at or below an index it has exposed or its history shows
(except the window move below), and persists the next index before exposing a
key.
**K** names `acceptSecret`'s public key as an acceptance's owner and signs the
acceptance with `acceptSecret` as well as with its own key (§4). A wallet signs
`acceptanceBytes` only as a backing's **K**, for a demand of that backing,
and with no key but that demand's `acceptSecret`. A demand's
presenter key is `presentSecret`'s, from the demand's tags in its input order
(`tag_2` is 32 zero bytes for one input), instant and deadline, all public in
the demand, so a restored wallet finds its standing demands and can release
or withdraw them.

Restoration (C4.6's scope and evidence) replays, for each backing it
restores, that backing's authenticated trails and recognizes outputs, in every
trail it replays, by the owner keys of every backing it restores. It derives
each backing's owner keys from index 0 until 256 consecutive indices appear in
no output of that backing in its trails; an output of another backing to one
of a backing's keys is the wallet's note but moves no index of either. For each settlement it
derives `acceptSecret` from its demand and deadline, and for each demand over
its notes `presentSecret` from the demand's tags, instant and deadline. Let `h`
be the highest index this rule finds in an output of the backing in the
backing's finalized history (−1 where it finds none). A wallet exposes no owner key of
the backing past index `h + 256`; past that it creates no request for the
backing until `h` moves. A restored wallet cannot see which indices it exposed
in requests nobody paid, so it treats every index through `h + 256` as
exposed. A wallet whose window is full, whether by restoration or by abandoned
requests, moves `h` itself: it closes any request that named its highest
exposed index and pays itself an output to that index, at the cost of one
statement and its fee (one statement may move the windows of several backings,
each output to its own backing's highest exposed index), and creates requests
again once that statement finalizes. No request is ever credited with an
output of a statement that consumes notes, all of them the wallet's own (notes
to keys its seed derives): a spend's or burn's inputs, or the notes a
settlement's demand names. An issue consumes none, and is credited unless the
wallet itself made it; **K**'s wallet tells by its own nonce derivation, and
one that cannot credits no issue to its own requests. A wallet with no
notes of a backing waits for a payment to an exposed key of it. **K** derives an issue's
nonce and a holder its refresh values deterministically from its own secrets
(invariant 26), in derivations this document does not fix, since no reader or
restoring wallet reads them; **K** keeps an issuance's nonce to sign it again
for a new segment (§2).

## 9. Configuration and terms

```text
configurationBytes = "moe/lit/v1/config" || u8(2) || u8(4)
configHash = SHA256(configurationBytes)
           = 17835aa2cc5e76c4cc1df8ec6486b5ca3419a96c732256ccb5abccd39c9a77c1
```

The two bytes are the maximum input and output counts. There is no circuit,
key, parameter or helper identity. The configuration hash is the domain of
every lit statement and object above.

**Root terms** reuse pool-v3 §11.2's `MOEB` version-1 frame, name hash and
backing-signature message, with this clause table:

| Tag | Payload, after the tag |
|---|---|
| 2, witnessing | venue[32], u64 witnessInterval |
| 3, replacement | replacementRuleKey[32] |
| 4, non-service | u64 duration, u32 count, u64 window |
| 5, construction | u32(10), ASCII `moe/lit/v1`, configHash[32] |
| 6, silence | u64 noCommitmentDuration |

Tags 2 and 5 are required and tags 3, 4 and 6 optional, strictly ascending, so
the clause count is 2–5 and the terms are at most 1296 bytes. Tag 1, the
pool's silence clause with its challenge window, is refused here, and tag 6
under pool-v3. A clause tag therefore has one payload whatever the
construction, and terms decode under at most one construction, since tag 5
names it. Pool-v3 §11.3's checks apply with this construction and hash. Every
backing of one scope declares the same silence duration or none (C2b.6.1).
This encoding has no identified-issuance clause, as pool-v3's has none.

## 10. Venue and record ranges

A lit backing reads its venue through pool-v3 §13 unchanged, frame and
context included: the range request and answer are construction-neutral, and
kind 4's records are §4's publications under §13.1's parser bound. A lit
backing may declare a venue under the [Ergo venue profile](venue-ergo.md)
without a change to it. That profile excludes only a construction whose venue
records §6 does not attribute (venue-ergo §1); lit's records are kinds 1–4,
and §6 attributes a kind-4 object by its location and shape without reading
its content. Its largest publication, 569 bytes, fits one box, which venue-ergo
§8's condition for publishing asks. Pool-v3 §14's replay, kept state, kept
classes, incremental retrieval and retention apply with §5's spent root and
chains in place of v3's roots and chains. A reader resuming kept state
recomputes the spent root, the totals and both chains against the
authenticated snapshot, as §14 requires, and rebuilds the output set from the
kept statements and imports, since no root in the snapshot checks it.

## 11. Shape, visibility and supply

**Shape.** A spend has one or two inputs and one to four outputs, and a burn
one or two inputs and at most one change output, as pool-fees C1.2.3 bounds
the pool's. No position is padding: every input is a real note and every
output positive, since lit hides nothing that zero values would cover. A fee
is an ordinary output to the operator's requested key (C1.2.4). The bounds are
the configuration's.

**Who sees what.** Everyone sees every statement's openings: backings, values,
owner keys and the spend graph linking each note to the notes created from it.
An issuance shows the key that received credit. Nobody sees the civil identity
behind a key, unless its holder discloses it. A holder may take a fresh key for
each note (§8); this is pseudonymity, not privacy.

**Supply.** A reader recomputes `outstanding = issued − burned` from the
openings and K's signatures, with no proof. A lit backing reports its figures
as public, and invariant 3 ranks a closure that includes it transparent.

## 12. What this replaces and costs

Relative to pool-v3 and its contracts, lit removes, for its own construction:
the six relations, proofs, keys and parameters; anchors, the note tree and the
accepted-root forest; the scope tree and scope root; padding inputs and zero
outputs; recovery capsules and the delivery digest (pool-delivery C4.1–4.4 and
C4.7); C3.5's `rho_out` and disclosure count and C2.10.8's settlement
exception; the proof digest in the evidence chain, receipt and fault evidence;
and the challenge window. It adds the output derivation (§2), owner signatures
in place of proofs, an issue nonce, and the acceptance owner's signature (§4). It authenticates a served trail's
records by §3's split rather than their decoding, so a committed record that
splits but does not decode can exclude its checkpoint instead of leaving it
unresolved until a term ends (§6). Every other mechanism is the pool's.
The pool's note root has no lit counterpart (§5).

Costs: the profile's lit visibility (§11 and Extensions). Records are 189 to
719 bytes against the pool's records of about 15 KB, and a reader checks one
Ed25519 signature per input (three per settlement) and a few SHA-256 hashes per note, beside the
spent-root update the pool also pays, in place of a proof; both effects are estimated, not measured. A restored wallet scans owner
keys within a 256-index look-ahead per backing, so a key exposed past it is not
seed-recoverable, which §8's rule prevents, and a wallet whose 256-key window
is full, after a restoration or abandoned requests, pays itself once before it
can make another request: anyone who takes requests from a public source and
never pays costs it one such fee per 256 requests. An operator serving both
constructions holds two keys (§6). Limits of v1: no identified-issuance
clause, though Extensions names a lender wanting it as one of the profile's
needs; no compact membership proof (§5); no reliance.

Falsifiers: a reader that admits an output it did not derive; a path by which
someone other than a note's owner moves, presents, withdraws or locks it; an
acceptance read as an answer under C3.8 whose owner key's secret **K** did not hold; a
pool rule in C2b or C3 that needs a lit branch beyond §7's readings; or a
measurement showing lit replay not materially cheaper than pool replay at the
design point.
