# Succinct reading

## 1. Status and scope

This document fixes a reader's option. In place of some deterministic checks
a complete read runs over bytes it holds, a reader may take a proof that the
check passed over bytes bound by digests it computes itself. It is written for
a holder without a complete reader's resources, a phone first, but any reader
may use it. It fixes one contract (§2) and two instances: the proofs of a
[pool-v3](pool-v3.md) trail's statements (§3) and the exhaustion of
[venue-ergo](venue-ergo.md) record ranges (§4). Section 4 also serves the
backings of other constructions on venue-ergo, such as [lit-v1](lit-v1.md)
and pool-v2 (venue-ergo §1); §3 serves only pool-v3, since lit records carry
no proof.

**It is not a verdict rule.** Pool-v3, lit-v1 and venue-ergo are unchanged.
Their verdicts remain the complete read's, from the closure and venue evidence
they name (pool-v3 §§12.1, 13.2, 14; C2.10.11–13), and pool-v3 §9.1 remains
the only compact replacement of part of a closure. A read under this document
gives a **succinct judgment**, which is not a verdict under C2.10.11–13 or
pool-v3 §12.1. It is the reader's own basis for its own acts, such as
accepting a payment, holding or spending a note, presenting a demand or
reading supply. It is never served, never a certificate and never another
reader's evidence (C2.10.13). It agrees with the complete read's verdict
wherever the adopted circuits accept exactly what the checks they replace
accept, barring a false proof and collisions of SHA-256, Blake2b-256 and `H`
(§2). It never changes a complete reader's verdict.

An act other parties judge is judged by them completely: an operator's
admission, imports, adoption, openings and commitments, a backer's
acceptances, a replica's evidence. A party that decides such an act on a
succinct judgment bears the risk that complete readers judge it otherwise,
so an operator or backer should read completely. A party that shows another
party an exclusion or a verdict reads completely; a succinct reader's own
exclusions (§3.4) serve only its own descent.

The option is not a venue-evidence verifier under pool-v3 §13.2 or
venue-ergo §7: §4's result is not an answer in their sense, though it is laid
out in pool-v3 §13.1's frame for the reader's own pool-v3 §13.3 derivation.

No backing, term or operator declares this document, and **E** does not name
it. It adds no byte, identity, record, signature or authority to any
construction or venue. The name `moe/succinct/v1` fixes these rules; a change
to a digest, frame or check is a later name, except that before a set is
adopted under it, a correction restoring this document's agreement with the
complete read, or a rule over the circuits' internal digests, which no
reader computes and the set identity binds, keeps the name. A reader relies
on a circuit set only under an adoption (§6), and none is adopted yet; until
one is, an implementation runs these rules only over test identities
labelled as such.

Integers are unsigned big-endian, `||` is byte concatenation, `F` is a
canonical field element of [pool-v2 §1](pool-v2.md#1-fields-hashes-and-encodings),
and `H` is pool-v2 §1's sponge. A 32-byte value `v` enters `H` as the two
limbs `v.hi` and `v.lo` that pool-v2 §1 defines. "Bit `l`" of an integer
counts from the least significant bit, at zero. Section 5 is the C0a record.

## 2. The contract

A **check** is a deterministic function a complete read evaluates over bytes
it holds. A **check proof** is a proof under the key of a circuit of an
adopted set (§6) whose public inputs bind the check's inputs through digests.
It is a proof of pool-v3 §4's proof system: UltraHonk over BN254 with the
Ignition parameters, under the same assumptions and the same verification
parameters. Proving reads more of Ignition's leading G1 points than pool-v3's
relations do (§6); verification reads none beyond those pool-v3 §4 names.

1. **Pinned keys.** A reader selects a circuit set by its set identity (§6),
   independently of any supplier, as it selects a configuration and a venue.
   A circuit that verifies a proof of a pool relation holds that relation's
   key as a constant, one circuit per kind; no key travels with a proof
   (pool-v3 §4). A circuit that verifies proofs of the set's own circuits
   admits each child by its key hash, from a fixed list of hashes it holds as
   constants. Where a circuit admits proofs of itself, its own key hash cannot
   be a constant: it takes that hash as a public input and requires every
   child to carry the same value. That input is free in the circuit, so
   whoever admits a proof of such a circuit under its key, a parent holding
   the key as a constant or the reader under the pinned key (rule 2),
   requires the proof's key-hash input to equal that key's hash.
2. **The reader's checks of a proof.** The reader verifies each check proof
   it uses with the pinned backend, under the pinned key of the circuit the
   rule names, and compares every public input with a value it derived
   itself: each key-hash input with the pinned hash, each digest with the
   digest it computed from bytes it holds, and each header input with a
   header it accepted itself. A proof with an input the reader cannot derive
   is not used. A circuit's own key hash is a free input, so a tree of proofs
   under a made-up circuit verifies under the pinned key; only this comparison
   refuses it. The pairing points an inner proof defers travel inside the
   outer proof as its **pairing-point object**: the last eight public inputs
   the pinned backend appends to the circuit's own, two points of two
   coordinates in two limbs each, a point at infinity as four zeros. They are
   not compared as inputs; the outer proof's verification checks them with its
   own opening. The reader refuses a check proof whose object has exactly one
   point at infinity: the pinned backend's verifier then adopts the proof's own
   opening in place of the object and checks no pairing an inner proof
   deferred, and a prover reaches that by choosing an inner proof's last
   opening commitment. An object with both points at infinity satisfies the
   pairing alone, so adopting the opening then drops nothing: a circuit that
   verifies no proof publishes it, and an honest proof of one that does
   publishes neither point at infinity, barring negligible chance.
3. **A passing check only.** A check proof establishes that its check passed,
   and nothing else. A missing, unverifying, malformed or mismatched check
   proof establishes nothing: no failure, absence, exclusion, fault or lapse.
   The reader then runs the check itself over the bytes (the **native check**,
   §§3.5, 4.6), or the read stays unresolved where the bytes are withheld, as
   in a complete read.
4. **Failures are native.** Every refusal a complete read grounds on a failing
   check, such as a proof's rejection, a deterministic failure in replay or a
   pool-v3 §9 fault target, the reader establishes itself by that check over
   the exact bytes.
5. **Everything else is the complete read's.** Where this document does not
   replace a check, the reader applies the complete read's rules unchanged,
   in the same order, over the same authenticated bytes. A field a supplier
   states and nothing authenticates is a supplier's failure where it is
   wrong, never the read's ground.
6. **No authority.** Any party may prove. A check proof carries no signature
   and no party's word, whoever supplies it is untrusted, and a reader may
   obtain each one from any source. A withheld proof only returns the reader
   to the native check.
7. **Disagreement.** Check proofs that pass rules 1–2, set against bytes the
   native check refuses, show the circuit set unsound for that check: for §3,
   a cover (§3.2) of a record whose proof the native check rejects; for §4, a
   fold, path and entries (§4.4) that lack an output with the shape for that
   key which an authenticated section shows, or that name an output a
   transaction inside venue-ergo §5's grammar does not have. A reader holding
   such evidence stops using the set for every read, discards what it derived
   under it (rule 8) and reads natively or completely. Any party can hand the
   evidence to another reader, which checks it itself. Its absence proves
   nothing.
8. **Results stay local.** A succinct judgment, and state or classes kept
   from a succinct read (pool-v3 §14), are reused only by succinct reads under
   the same set identity and rules; a complete read discards them. A succinct
   reader holds no proof bytes it did not fetch, so it cannot hand another
   reader a verdict's closure (pool-v3 §12.1).

**What a succinct judgment rests on.** Forging one needs a false proof, which
rests on the proof system and its setup, as every statement already does
under **E**, or a circuit that accepts what its check refuses. The second is
new: the adopted circuits' agreement with the checks they replace. One set is
held by every reader that selects it, so a defect is shared across the
backings those readers hold. It changes no complete reader's verdict, though
an act decided on it can reach other parties (§1). Adoption therefore needs
the agreement evidence and review §6 names.

**Digests.** Eight domain tags continue pool-v2 §1's numbering after pool-v3
§2's `T_TAG`. They are used only in this document's circuits and digests.
The last three are internal to the exhaustion circuits (§4.3): their inputs
after the tag are fixed by the set's sources (§6), and no reader computes
them.

| Tag | Value | Used for |
|---|---|---|
| `T_CHECK_LEAF` | `1008` | a statement proof's digest (§3.2) |
| `T_CHECK_NODE` | `1009` | two spans' digest (§3.2) |
| `T_ENTRY` | `1010` | a key's entry chain (§4.2); a period's entry stream (§4.3) |
| `T_MAP_LEAF` | `1011` | a subject-map leaf (§4.2) |
| `T_MAP_NODE` | `1012` | a subject-map node (§4.2) |
| `T_EX_ID` | `1013` | a period's chain of transaction ids with their heights and positions (§4.3) |
| `T_EX_STATE` | `1014` | an exhaustion proof's state at either end (§4.3) |
| `T_EX_CHALLENGE` | `1015` | a transaction leaf's two challenges (§4.3) |

## 3. The trail's statement proofs

### 3.1 What is replaced

A complete read replays each served trail its read needs, the imported
closure's included (pool-v3 §§10.1, 14). For each record of kind 1, 2, 3, 4 or
6 it reads the proof bytes, computes `proofHash` from them for the evidence
recurrence (pool-v3 §7) and verifies the proof under pool-v3 §11.4's key for
its kind. A succinct reader instead receives each record's proof-free form
(§3.3), whose `proofHash` the evidence recurrence authenticates. It takes "the
proof verifies" from a check proof over a span covering the record (§3.2), or
checks the record natively (§3.5). Kind 5 has no proof. A committed kind-7
record fails replay where it decodes (pool-v3 §7, §10.1) and its proof is
never verified, but its proof field must still decode under pool-v3 §5, so
its bytes are fetched for their length (§3.4). Venue publications are not replaced (§3.6).

### 3.2 Digests, spans and their proofs

For a record of kind `k` in `{1, 2, 3, 4, 6}` with public inputs
`x_1 … x_m` in pool-v3 §5's order (`m` is 11, 15, 15, 16 and 17 respectively)
and `proofHash` `h`:

```text
leafDigest       = H(T_CHECK_LEAF, k, x_1, …, x_m, h.hi, h.lo)
nodeDigest(a, b) = H(T_CHECK_NODE, a, b)
```

A trail's **proved records** are its records of those five kinds in position
order, numbered from zero: `r_0, r_1, …`. For `s >= 0`, the **span** `(s, j)`
covers `r_(j·2^s)` through `r_((j+1)·2^s − 1)` and exists where the trail has
all of them. Its digest `D(s, j)` is the leaf digest of `r_j` at `s = 0`, and
`nodeDigest(D(s−1, 2j), D(s−1, 2j+1))` above.

The set's trail circuits are:
- **five trail leaves,** one per kind `k`, each holding pool-v3 §11.4's key
  for `k` as a constant. A leaf verifies one proof of 14,656 bytes (458 field
  elements, the length pool-v3 §11.4's backend proves) under that key with
  public inputs `x_1 … x_m`, computes `h` as SHA-256 over the proof's bytes,
  and outputs `leafDigest`. Its public inputs are `(nodeKeyHash, digest)`;
- **the trail node,** which verifies two children, each under one of the five
  trail-leaf keys (constants) or under its own key, admitted by key hash (§2
  rule 1), and outputs `nodeDigest` of the children's digests, left then
  right. Its public inputs are `(nodeKeyHash, digest)`.

A check proof **covers** span `(s, j)` where it passes §2 rule 2: at `s = 0`
under the pinned trail-leaf key of `r_j`'s kind, at `s >= 1` under the pinned
trail-node key; its `nodeKeyHash` equals the trail node's pinned key hash; and
its digest equals `D(s, j)` as the reader computes it. It then establishes
that every record of the span has a proof of 14,656 bytes whose SHA-256 is that
record's `proofHash` and which pool-v3 §11.4's verifier for its kind accepts
on its public inputs.

The two tags, and each kind's fixed number of inputs, which the sponge's
initial state carries, separate a leaf's digest from a node's. Without a
collision of `H`, a covering proof therefore attests exactly the reader's
records, in the reader's order and shape. The leaf digest carries the
configuration through `x_1, x_2`, so digests of two configurations never
meet. A supplier answers a request for a digest with a proof whose digest it
is: one proof serves every trail holding those records at those proved-record
numbers, such as the prefix a later checkpoint of the segment extends.

A cover attests each record of its span by that record's leaf digest. So a
cover of a span of another served trail of the same segment, one the reader
authenticated against a checkpoint's snapshot (§3.3), also **covers** the
proved record `r_i` of the trail being read where the span holds that other
trail's `r_i` and its leaf digest is `r_i`'s, whether or not the span exists
in the trail being read (§3.4).

A proof that the backend cannot read as 458 field elements, such as one with a
word of `p` or more, enters no leaf; the native check (§3.5) reads it as the
complete read does.

### 3.3 The proof-free trail

The **proof-free form** of a record is the record with its proof field,
length and bytes, replaced by the field's `proofHash` (pool-v3 §5: SHA-256 of
the proof, or 32 zero bytes exactly where the proof is empty):

```text
freeRecordBytes = statementBytes || proofHash[32] ||
                  u32 authorizationLength || authorization[authorizationLength] ||
                  u32 capsuleCount || capsule_1[89] ... capsule_capsuleCount[89]
freeTrailBytes  = "moe/succinct/v1/trail" || u32 headerLength || segmentBytes[headerLength] ||
                  signedTerms_1 ... signedTerms_m || u64 n || freeEvent_1 ... freeEvent_n
freeEvent_i     = u32 recordLength || freeRecordBytes[recordLength]
```

The context is literal ASCII. The head (header, signed terms and count), its
checks, the budgets, the exact end and the rule that budgets are local are
pool-v3 §10's. The exact size is `33 + headerLength + sum_j(68 + termsLength_j)
+ sum_i(4 + recordLength_i)` bytes. A proof-free form is at most 934 bytes, a
spend's: 538 statement bytes, 32 of `proofHash`, 8 of lengths and 356 of
capsules. By kind 1 to 7 the forms are 603, 934, 667, 610, 386, 778 and 322
bytes; the complete records of the kinds with a proof are 14,950 to 15,562.

A proof-free form **decodes** where its statement, authorization and capsules
satisfy pool-v3 §5, and its `proofHash` is the zero sentinel exactly for kind
5. Pool-v3 §5's delivery association is checked as there. The proof's length is not
in the form: it is established for each record of a kind other than 5 by a
cover, which fixes 14,656 bytes, or by the fetched bytes (§3.5), to which
pool-v3 §5's length rule applies.

**Authentication** is pool-v3 §10.1's, each record's evidence triple read
from its proof-free form: `statementHash` from `statementBytes`, `proofHash`
as carried, and `signatureHash` from the authorization, the zero sentinel
where it is empty. The recurrence's terminal value must equal the
authenticated snapshot's `evidenceHash_n`. That authenticates, in order,
every statement, authorization and `proofHash`. Barring a SHA-256 collision,
each `proofHash` names the committed proof's bytes, and the zero sentinel an
empty proof. Capsules are bound by the delivery association for kinds 1–3 and
by the fixed count zero for the others, as pool-v3 §10.1 binds them. A trail
that fails to authenticate, or a form whose capsules fail pool-v3 §5 (count,
profile or delivery association), is a supplier's failure: the reader obtains another, or
the complete trail (pool-v3 §10), or the trail stays unresolved unless
C2.10.11 or pool-v3 §9.1 classifies the checkpoint. Where an authenticated
field of a form among the served trail's records does not decode (its
statement, its authorization's length, or its `proofHash` against the
sentinel rule), the operator committed bytes pool-v3 §5 cannot decode: the
trail is inconclusive, as pool-v3 §10.1 leaves the complete trail, and those
rules classify the checkpoint as they would. Pool-v3
§12.1's prefix rule and §14's incremental retrieval apply with this frame's
head and events; forms after the served trail's last record neither block nor
affect it.

Whoever holds a complete trail derives its proof-free trail; nothing in it is
signed, and a replica may serve both.

### 3.4 Reading a trail succinctly

For each served trail the complete read would replay, the reader:

1. obtains and authenticates its proof-free trail (§3.3);
2. partitions the proved records into consecutive spans that exist and single
   records. A record its segment's partition covers (below) needs no span of
   its own. Each span in the partition must be covered (§3.2); a span the
   reader finds no cover for, it splits, down to single records. Each proved
   record that no cover covers (§3.2), and each kind-7 record, has its proof's
   bytes fetched, or in the adopted block read from the held publication,
   without verifying them (§3.5);
3. establishes the proof length of every record of the served trail (its
   first `n` records, `n` being the position whose evidence value is the
   snapshot's `evidenceHash_n`, pool-v3 §12.1) of a kind other than 5, before
   replay and before the evidence recurrence decides a non-extension: 14,656
   for a covered record, the fetched length for the others. A fetched proof,
   whose bytes hash to its `proofHash`, that pool-v3 §5 cannot decode leaves
   the trail inconclusive, as pool-v3 §10.1 leaves a complete trail with that
   record. A withheld one leaves the trail unresolved unless C2.10.11 or
   pool-v3 §9.1 classifies the checkpoint, as in a complete read without the
   bytes. A reader may stop a supplier's answer past pool-v3 §5's bound of
   131,072 bytes before hashing it: that supply is withheld (§2 rule 3), and
   another may serve the field;
4. replays under pool-v3 §14, taking "the proof verifies under the
   configuration's key for its kind" from the cover for a covered record and
   from the native check for the others, each only where and when replay
   verifies that proof: never in the adopted block, for a kind-7 record or for
   a non-extension. Every other check of replay runs as
   the complete read runs it: decoding, delivery association, Ed25519
   authorizations, note and scope paths, the spent set, totals, demands,
   locks, deadlines, clocks, lapse, imports, adoption, recovery, succession
   and fault classification. A record that the native check rejects is a
   deterministic failure, as in the complete read (pool-v3 §§7.1, 14);
5. compares, where replay compares a trail record with an adopted publication
   byte for byte (C2b.4.2), the publication's proof-free form, which it
   derives from the bytes it holds in full (§3.6), with the trail's. Equal
   forms are equal records, barring a SHA-256 collision, since equal
   `proofHash` values name equal proof bytes and lengths.

A non-extension decided by the evidence recurrence (pool-v3 §7.1) verifies no
proof and is reached after step 3, as the complete read reaches it only over a
trail whose every record decodes. A lapse (C2.10.4) reads none of the
checkpoint's records. Both are reached as in the complete read.

The canonical partition of a trail's `N` proved records is one span for each
set bit of `N`, highest first. A reader that kept its state (pool-v3 §14)
and reads a later checkpoint extending it partitions only the proved records
after its kept ones, in at most about `2·log2` of their count spans.

**A segment's partition.** A reader that replays a segment checkpoint by
checkpoint may, within one read, partition one trail of the segment once, as
step 2 partitions a trail, after the records it kept (pool-v3 §14). That trail
is the served trail of a commitment of the operator whose checkpoints the read
is judging. It is the commitment of highest sequence that is held (C2.3.3)
within that operator's term in the backing being judged (C2.5), carries that
backing and names the segment, among those whose directory, snapshot and
served trail the reader holds. The trail is authenticated against that
snapshot, which the operator signed. A longer trail that no checkpoint
commits, which any supplier could lengthen with records of its own, is never
partitioned, nor one a key signed past its term. Each checkpoint's trail then
takes that partition's covers for its records (§3.2) and partitions its other
records under step 2 itself. Within the read, a record the reader already
asked about alone and found no cover for needs no second ask, nor does any
span holding it; and a reader may ask about fewer spans than step 2 names (§2
rule 3), since a missing cover only leaves a record to its native check. Only
covers and the answers to asks pass between trails: each served trail's
fetches, kind-7 lengths and refusals stay its own (steps 3–5).

A fresh read of `N` proved records over `C` checkpoints then asks about
`2·log2 N` covers for what the provers reached, rather than about
`C·log2(N/C)` for checkpoints partitioned one by one. The tail costs its
proofs (§3.5) and, under step 2, the asks for its records' spans.

### 3.5 Native checks and the tail

The native check of a proved record fetches, from any supplier, the bytes
whose SHA-256 is its `proofHash`, which authenticate themselves, applies
pool-v3 §5's length rule to them (§3.4 step 3) and, when replay reaches the
record's proof check, verifies them under pool-v3 §11.4's key for the
record's kind, as the complete read does. A record in the adopted block whose
`proofHash` is the adopted publication's takes its length from the
publication the reader holds (§3.6); otherwise its bytes are fetched as for
any record. The **tail** is the
records after the last a cover reaches, usually those no prover has reached
yet. Each costs its proof's bytes and one verification. A payee judging a
receipt's checkpoint (pool-v3 §7.2) judges it once every proved record of
that checkpoint's trail and of its imported closure is covered or checked
natively, and every other record's proof length is established (§3.4). A
kind-7 record in a trail, which only an operator's fault commits, costs its
proof's bytes and no verification, and so does each uncovered record of a
trail that does not extend its segment's last valid checkpoint in the
checkpoint's own record prefix (pool-v3 §7.1), unless the checkpoint is
lapsed.

The pinned backend proves a leaf over an inner proof that does not verify,
and that leaf's proof then fails verification. A prover therefore verifies
each statement proof natively before proving its leaf; a leaf over an
unverifying proof covers nothing.

### 3.6 Venue publications

Publications the reader reads from the venue (pool-v3 §6: demands,
acceptances, releases, withdrawals and requests) are not replaced. Their force
(C2b.3.2, C2b.5.2) rests on their exact bytes and on their proofs verifying,
judged at the venue, outside the evidence chain. The reader verifies each
proof-carrying publication natively, over bytes its venue read returns in
full (§4.4). No leaf exists for kind 7. Each publication a stranger places at
a backing's location costs the reader its bytes and one verification, priced
by the venue's minimum (venue-ergo §8).

## 4. Venue exhaustion

### 4.1 What is replaced

For a request under a venue-ergo identity whose anchor is at height `A`, the
complete read's verifier reads every section of the range (venue-ergo §4) and
frames every transaction in it (venue-ergo §5). A succinct reader still
accepts headers itself from the anchor (venue-ergo §3) and reads the venue's
chain only from them. For heights from the adopted start through a fold's
end, it takes the outputs with venue-ergo §6's shape for its subject from the
subject map (§4.2) under a fold proof (§4.3). It fetches each such
transaction with an inclusion path (§4.5) and attributes its objects itself
(venue-ergo §§5–6). Above the fold's end it reads sections natively (§4.6).

### 4.2 The subject map

An **entry** is an output, at any location, with venue-ergo §6's shape:
`(subject, height, position, output)`, where `height` is its block's height,
`position` its transaction's index in the block's section and `output` its
index in the transaction, both from zero. A subject's **key** is its last four
bytes read as an integer. Each key has a count `c` and a chain value:

```text
chain_0 = 0
chain_i = H(T_ENTRY, chain_(i−1), subject.hi, subject.lo, height, position, output)
leaf    = 0 where c = 0, otherwise H(T_MAP_LEAF, c, chain_c)
node    = H(T_MAP_NODE, left, right)
```

The map is a sparse Merkle tree of depth 32 over keys: at level `l`, counted
from the leaves at zero, bit `l` of the key is one where the path enters its
parent as the right child. Empty subtrees are `z_0 = 0` and
`z_(l+1) = H(T_MAP_NODE, z_l, z_l)`, and the empty map's root is `z_32`.

The **map at a header** `E` starts empty at the adopted start header (§6) and
appends every entry of every block from the start's child through `E`, in
order of height, position and output, to its key's chain. It is a function of
the start and the chain through `E`. For a transaction inside venue-ergo §5's
grammar, the entries are exactly its outputs with venue-ergo §6's shape, in
output order. For a transaction outside the grammar they are the outputs the
adopted scan reads as having that shape, its own deterministic rule fixed by
the set identity. Such a transaction carries no record (venue-ergo §5), so a
reader passes over its entries after reading it. Two subjects that share a
key share its chain, and a reader passes over entries that name another
subject.

### 4.3 The fold proof

The set's exhaustion circuits prove the map in **periods**, runs of
consecutive blocks each ending at a block boundary: three exhaustion leaves
(transactions, transaction trees and headers, map updates), the exhaustion
node, the first fold and the next fold. A first fold proves a period from the
adopted start. A next fold proves a period and verifies the previous fold,
under the first fold's key (a constant) or its own (`foldKeyHash`), ending
where its period starts. Each fold verifies its period's tree under the
exhaustion-leaf keys or the exhaustion node's key, all constants, and
requires a root under the node's key to carry that key's hash as its own
key-hash input (§2 rule 1): a tree under a made-up circuit otherwise verifies
under the genuine node key.

Inside a period the proofs join by their state at either end, hashed under
`T_EX_STATE`: a node requires its left child's end to be its right child's
start. The transaction leaves and the tree-and-header leaves each chain the
period's transaction ids with their heights and positions under `T_EX_ID`;
the transaction leaves pass the entries they scan to the map-update leaves as
one chain of all keys' entries in §4.2's `T_ENTRY` form; and a fold requires
each pair of chains to start at zero at the period's start and to be equal at
its end. A transaction leaf lists the entries its scan emits, in strictly
increasing order of height, position and output, each joining the entry
stream, and checks the list against them as a multiset under two challenges,
each an `H` under `T_EX_CHALLENGE` over the leaf's two state hashes with a
distinct last input. Its end state carries the entry stream, the id chain,
the open transaction's Blake2b state and the scan's registers and labels, so the
challenges follow both multisets.

A fold proof's public inputs are, in order, `startHi, startLo, startHeight,
endHi, endLo, endHeight, mapRoot, foldKeyHash`, the ids as limbs. It
establishes three things. The blocks from the child of the start header
through the end header form one chain by parent ids, one height apart. Each
block's transactions are exactly those whose ids, with witness ids where the
root uses them, reproduce its header's transactions root under one of
venue-ergo §4's rules, in that order. `mapRoot` is the map at the end header
(§4.2).

The reader uses a fold proof where, under §2 rule 2:
1. it verifies under the pinned first-fold or next-fold key;
2. `foldKeyHash` equals the next fold's pinned key hash;
3. `startHi, startLo, startHeight` are the adopted start's;
4. `endHi, endLo` are the id of the header at `endHeight` on the reader's
   best chain, at least the venue's depth below its tip.

The proof attests no header's work. The reader's own header chain carries the
work, and the fold's end binds the scanned blocks to that chain by parent ids.
A fold at the depth holds only blocks the venue's finality rule fixes.

A **path** for a key at a fold is its leaf's count and chain value, the chain
value zero where the count is zero, and 32 siblings; the reader recomputes
`mapRoot` from them.

### 4.4 Reading a range

For a request of record kind `k`, subject `s` and indices `from` through `to`,
with `to` witnessed (venue-ergo §2) and the adopted start at height `A` or
below, the reader:

1. takes a fold proof under §4.3 with end height `E`;
2. reads the path for `s`'s key at that fold, and the key's entries from any
   supplier. It recomputes the chain to the leaf's count and chain value,
   from `chain_0` or from a kept tuple (below); the entries must be exactly as
   many as the count grew;
3. takes each distinct `(height, position)` among the entries naming `s`
   with height from `A + 1 + from` through the lesser of `A + 1 + to` and `E`.
   For each, it fetches the transaction with an inclusion path (§4.5) to the
   transactions root of its header on the reader's chain, reads it under
   venue-ergo §5 and attributes its objects of kind `k` and subject `s` under
   venue-ergo §6, a kind-4 run reassembled from the transaction's adjacent
   outputs. A transaction outside the grammar contributes nothing;
4. reads every height above `E` through `A + 1 + to` natively (§4.6);
5. enters the objects as venue-ergo §7 enters them: index `height − A − 1`,
   for kind 4 in ordinal order with its ordinal, and for kinds 1–3 with
   ordinal zero in ascending record-byte order.

The result is the succinct read's range, in pool-v3 §13.1's frame, from which
the reader derives the record under pool-v3 §13.3 unchanged.

**Kept tuples and absence.** After a read the reader may keep, for the key,
the tuple `(set identity, key, c, chain_c, E, id of E)`. A later read at a
fold of the same set whose end `E2` is at or above `E`, on a best chain that
still holds the kept id at `E`, continues from the tuple. Where the count and
chain value at `E2` equal the kept ones, the chain holds no output with the
shape for any subject of that key from `E + 1` through `E2`. A count of zero
means none since the adopted start. The map is canonical in the start and the
chain, so folds of one set whose periods end at different heights agree on
every key at every end.

### 4.5 Inclusion paths

An inclusion path for the transaction at position `j` of a block is its
unsigned bytes and a list of at most 32 steps, each a 32-byte sibling or a
**lone child** (venue-ergo §4: a node without a right sibling). The reader
starts from `Blake2b-256(0x00 || id)`, where `id` is the Blake2b-256 of the
unsigned bytes. At step `l`, where bit `l` of `j` is zero, a sibling is
hashed to the right as `Blake2b-256(0x01 || current || sibling)` and a lone
child as `Blake2b-256(0x01 || current)`; where it is one, the sibling is
hashed to the left, and a lone child is refused. The last value must equal
the header's transactions root, and `j` must have no set bit at or above the
path's length. Without a Blake2b collision, a path reproducing the root is
the real one, for the transaction at position `j` of the section, under
either of venue-ergo §4's rules.

### 4.6 Native fallback and costs

Where a fold, path, entry list, transaction or inclusion path is missing or
fails, the reader reads the range's sections natively (venue-ergo §§4–7), or
the read stays unresolved. Above the latest usable fold's end it reads
natively; how far that lies behind the tip is the provers' pace. A fold of
another set is not used.

A path names its key, so a supplier learns which subjects a reader follows.
Reading the whole map, or paths for other keys as well, is the reader's lever.
A reader's first read of a key takes every entry of that key since the
adopted start, not since its venue's anchor. Every output with the shape that
names a key costs that key's readers an entry, about 44 bytes, and one naming
their subject costs a transaction fetch; the venue's minimum prices both
(venue-ergo §8), and a stranger can already add such outputs to a complete
reader's sections.

### 4.7 Transport

Check proofs, fold proofs with their public inputs, paths, entries and
transactions with inclusion paths travel in any framing a supplier and a
reader agree. Their bytes carry no protocol meaning: every value is
recomputed or compared by the rules above, and a malformed object is a
supplier's failure (§2 rule 5). The proof-free trail (§3.3) is the one frame
this document fixes, because its bytes are authenticated as such.

## 5. C0a cost and replacement

**What it replaces.** No rule of any construction or venue. For the reader
that chooses it, it replaces a trail's proofs, 14,656 bytes and one
verification a proved record, by a 32-byte `proofHash` and a share of a
span's check proof: a spend's record goes from 15,562 bytes to 934. It also
replaces a range's section bytes, about 6 MB a day of the Ergo mainnet, by a
fold proof, a path of about 1 KB, the subject's entries and their
transactions. A succinct reader still accepts every header from its venue's
anchor, replays every proof-free record, and checks venue publications,
signatures, failures, the tail and a kind-7 record's proof length natively.
Measurements are in the
reference's
[aggregation](https://github.com/mediumofexchange/reference-ts/blob/main/docs/POOL_DEPLOYMENT_PROBES.md#the-trails-proof-aggregation-slice-33)
and [exhaustion](https://github.com/mediumofexchange/reference-ts/blob/main/docs/VENUE_ALTERNATIVES.md#the-complete-exhaustion-leaf-slice-32)
probes.

**One mechanism (C0a items 1 and 2).** One contract (§2) with two instances,
one hash (pool-v2 §1's `H`, eight tags), one proof system (pool-v3 §4's) and
one frame. Nothing existing is generalised or fenced: the complete read's
rules stay where they are, and each instance replaces one input of them, a
proof's verification and a range's bytes, with a proof that the same check
passed. No rule here closes a gap another rule here leaves.

**Its derivation (C0a item 4).** A holder must verify the supply it accepts
itself (invariant 12, C2.10.13), and a phone without a node cannot read 13 MB
of proofs and 6 MB of Ergo sections a day, or 14.7 GB of proofs to restore at
10⁶ statements, at the design point. Asking another reader brings back the
gatekeeper the bound excludes. The checks' passing is what a phone lacks the
bytes for, so it is what a proof attests; their failures it can afford, so
they stay native.

**Why a document of its own (C0a item 6).** Extensions are terms a backing
declares in **E** and every reader of it must work out. This is neither: no
backing declares it, and each reader chooses it for its own reads. It is
optional per reader, not per deployment, so it
belongs neither in the core's verdict rules nor among Extensions.

**Costs (C0a item 5).**
- A circuit set of twelve circuits (five trail leaves and the trail node;
  three exhaustion leaves, the exhaustion node and two folds), which needs
  adversarial review against its pinned sources.
- Provers, which any party runs. One 4-vCPU machine kept up with the design
  point's mean trail 1.6 times, and with an ordinary Ergo day 3.4 times, in
  those probes. The departures from the probes are unmeasured: the sponge
  `H` over seven inputs costs three permutations for each chain an entry
  joins (the stream in two leaves and its key's chain) where the probe's
  chain cost one, the id chain two a transaction in two leaves where it cost
  one, big-endian limbs of Blake2b's little-endian words add range checks,
  and a leaf's digest drops from about ten permutations to seven. Step 4 of the reference's plan measures them.
- The trust in §2, held alike by every reader of one set: a defect reaches
  every such reader across backings, and acts others judge (§1).
- Round trips: each cover ask and each fetch is one, which over an onion
  service is about a second. A segment's partition (§3.4) keeps a fresh
  read's cover asks for what the provers reached near `2·log2 N`; the tail
  costs its proofs and its spans' asks.
- Retention: a succinct reader keeps no proof bytes, so complete closures'
  availability (C2.10.13, C2.7.2) rests on the complete readers and replicas
  that keep them.

**Alternatives.**
- **A successor construction.** A pool-v4 permitting aggregation as a compact
  replacement beside pool-v3 §9.1, and a venue-ergo context taking a fold as
  evidence, would make the circuits' agreement part of every reader's
  verdict, so a circuit defect would split complete readers too. Every
  backing would reach it only by successor and swap, to gain what no backer
  or operator acts on: any party proves, and no verdict is served (pool-v3
  §12.1). The circuits a reader pins are the same either way. No pool-v4
  candidate is needed: the reader's option needs neither the order-independent
  spent tree, a venue-bound commitment context, the venue-moving record nor a
  standalone value bound. A Poseidon2 proof digest in a later construction
  would bring a leaf from about 1.83 M gates to about 0.7 M (an estimate),
  since SHA-256 is 1.13 M of them: a prover's lever, not a reader's need.
- **A reader's verdict.** A phone asking another reader for its verdict takes
  a gatekeeper (C2.10.13).
- **Checking only one's own notes.** Counterfeit value passes.
- **Fault evidence alone.** Its absence proves nothing; an eclipsed reader
  would accept counterfeit value. It stays a complement, and §2 rule 7 is its
  form for circuits.
- **Every proof natively.** No circuit, at the bytes above. It stays the
  fallback, and serves a small scope.
- **A relation over the whole verdict.** One circuit reproducing every kind,
  lock, clock, recovery and fault, with Ed25519 refusing exactly what the
  host refuses, as every reader's verifier. It is costlier to prove and to
  get right than proving only the proofs (pool-v3 §14's list).
- **Aggregating venue publications by `proofHash`.** Their bytes are fetched
  in full by the venue read anyway, so a leaf saves one verification each
  and adds a kind-7 circuit.
- **The probes' digests.** Chained permutations under ad hoc tags, ids in
  little-endian words, and a fold input chaining every period's end. Pool-v2
  §1's sponge and limbs serve with no second hash rule, and a canonical map
  needs no record of past periods.

## 6. Adoption

No circuit set is adopted. An adoption names, under `moe/succinct/v1`:
- for each of the twelve circuits, its sources', bytecode's and key's SHA-256
  identities and its key hash, with the compiler, the pinned backend and the
  verifier target. Proving reads Ignition's leading G1 points up to the
  largest circuit, 2^22 in the probes, under hashes the adoption names as
  pool-v3 §4 does;
- the exhaustion's start header: its id and height on the Ergo mainnet, at or
  below the anchors of the venues it serves;
- the **set identity**: SHA-256 over a frame that begins with the literal
  ASCII context `moe/succinct/v1/set` and holds all the above, its bytes given
  by the adoption, so that no identity carries across a later name. A reader
  pins it, and kept state, classes and tuples record it (§2 rule 8, §4.4);
- the agreement evidence, with adversarial review of every circuit and the
  argument that each one computes its check:
  - each trail leaf's verdict equal to pool-v3 §11.4's verifier on genuine
    and hostile proofs, and the in-circuit verifier's limb-range and on-curve
    constraints read from the pinned backend's source;
  - for every circuit that verifies a proof, the pinned backend's native
    verification of the pairing it defers, with §2 rule 2's refusal;
  - the exhaustion's entries equal to venue-ergo §§5–6 over a named corpus of
    mainnet days and a hostile set, with the scan's reading of the grammar
    argued from its source;
  - the transaction-tree and header checks, for every header version (parent
    id, transactions root, height), the map updates, a transaction leaf's
    multiset check, and the joins of nodes and folds, each against hostile
    witnesses.

As in pool-v3 §11.4, the identities are named after every rule they
implement. A defect needs a new adoption, with a new set identity, which each
reader selects; a reader that drops a set discards what it derived under it.
