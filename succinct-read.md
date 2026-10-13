# Succinct reading

## 1. Status and scope

This document fixes a reader's option: in place of some deterministic checks
a complete read runs over bytes it holds, a reader may take a proof that the
check passed over bytes bound by digests it computes itself. It is written for
a holder without a complete reader's resources, a phone first, but any reader
may use it. It fixes one contract (§2) and two instances: the proofs of a
[pool-v3](pool-v3.md) trail's statements (§3) and the exhaustion of
[venue-ergo](venue-ergo.md) record ranges (§4). Section 4 also serves a
[lit-v1](lit-v1.md) backing on venue-ergo; §3 does not apply to lit, whose
records carry no proof.

**It is not a verdict rule.** Pool-v3, lit-v1 and venue-ergo are unchanged.
Their verdicts remain the complete read's, from the closure and venue evidence
they name (pool-v3 §§12.1, 13.2, 14), and pool-v3 §9.1 remains the only
compact replacement of part of a closure. A read under this document gives a
**succinct verdict**: the reader's own judgment for its own acts, such as
accepting a payment, holding or spending a note, presenting a demand or
reading supply. It is never served, never a certificate and never another
reader's evidence (C2.10.13). It equals the complete read's verdict wherever
the adopted circuits accept exactly what the checks they replace accept, and
the proof system is sound (§2). A reader that must ground an exclusion, or
show another party a verdict, reads completely.

No backing, term or operator declares this document, and **E** does not name
it. It adds no byte, identity, record, signature or authority to any
construction or venue. A holder chooses it as it chooses its own verifier
(pool-v3 §12.1, §13.2). The name `moe/succinct/v1` fixes these rules; a change
to a digest, frame or check is a later name. A reader relies on a circuit set
only under an adoption (§6), and none is adopted yet; until one is, an
implementation runs these rules only over test identities labelled as such.

Integers are unsigned big-endian, `||` is byte concatenation, `F` is a
canonical field element of [pool-v2 §1](pool-v2.md#1-fields-hashes-and-encodings),
and `H` is its sponge. A 32-byte value `v` enters `H` as the two limbs
`v.hi` and `v.lo` that pool-v2 §1 defines. Section 5 is the C0a record.

## 2. The contract

A **check** is a deterministic function a complete read evaluates over bytes
it holds. A **check proof** is a proof under the key of a circuit of an
adopted set (§6) whose public inputs bind the check's inputs through digests.
It is a proof of pool-v3 §4's proof system: UltraHonk over BN254 with the
Ignition parameters, under the same assumptions and the same verification
parameters. Proving reads more of Ignition's leading G1 points than pool-v3's
relations do (§6); verification reads none beyond §4's.

1. **Pinned keys.** A reader selects a circuit set by its adoption's
   identities, independently of any supplier, as it selects a configuration
   and a venue. A circuit that verifies a proof of a pool relation holds that
   relation's key as a constant, one circuit per kind; no key travels with a
   proof (pool-v3 §4). A circuit that verifies proofs of the set's own
   circuits admits each child by its key hash, from a fixed list of the set's
   key hashes and the hash of its own key. It takes its own key hash as a
   public input and requires every child to carry the same value.
2. **The reader's checks of a proof.** The reader verifies each check proof
   it uses with the pinned backend, under the pinned key of the circuit the
   rule names, and compares every public input with a value it derived
   itself: each key-hash input with the pinned hash, each digest with the
   digest it computed from bytes it holds, and each header input with a
   header it accepted itself. A proof with an input the reader cannot derive
   is not used. A circuit's own key hash is a free input, so a tree of proofs
   under a made-up circuit verifies under the pinned key; only this comparison
   refuses it.
3. **A passing check only.** A check proof establishes that its check passed,
   and nothing else. A missing, unverifying, malformed or mismatched check
   proof establishes nothing: no failure, absence, exclusion, fault or lapse.
   The reader then runs the check itself over the bytes (the **native check**,
   §§3.5, 4.6), or the read stays unresolved where the bytes are withheld, as
   in a complete read.
4. **Failures are native.** Every refusal a complete read grounds on a failing
   check, such as a proof's rejection, a deterministic failure in replay or a
   §9 fault target, the reader establishes itself by that check over the
   exact bytes.
5. **Everything else is the complete read's.** Where this document does not
   replace a check, the reader applies the complete read's rules unchanged,
   in the same order, over the same authenticated bytes.
6. **No authority.** Any party may prove. A check proof carries no signature
   and no party's word, whoever supplies it is untrusted, and a reader may
   obtain each one from any source. A withheld proof only returns the reader
   to the native check.
7. **Disagreement.** A check proof that passes rules 1–2 over bytes the
   native check refuses shows the circuit set unsound for that check. A
   reader holding such a pair stops using the set for every read, discards
   what it derived under it (rule 8) and reads natively or completely. Any
   party can hand the pair to another reader, which checks both itself. The
   absence of such a pair proves nothing.
8. **Results stay local.** A succinct verdict, and state or classes kept from
   a succinct read (pool-v3 §14), are reused only by succinct reads under the
   same circuit set and rules; a complete read discards them. A succinct
   reader cannot hand another reader a verdict's closure (pool-v3 §12.1)
   beyond the bytes it holds.

**What a succinct verdict rests on.** Forging one needs a false proof, which
rests on the proof system and its setup, as every statement already does
under **E**, or a circuit that accepts what its check refuses. The second is
new: the adopted circuits' agreement with the checks they replace. One set is
held by every reader that selects it, so a defect is shared across the
backings those readers hold. It never reaches a complete reader. Adoption
therefore needs the agreement evidence and review §6 names.

**Digests.** Five domain tags extend pool-v2 §1's table and pool-v3 §2's
`T_TAG`:

| Tag | Value | Used for |
|---|---|---|
| `T_CHECK_LEAF` | `1008` | a statement proof's digest (§3.2) |
| `T_CHECK_NODE` | `1009` | two spans' digest (§3.2) |
| `T_ENTRY` | `1010` | a key's entry chain (§4.2) |
| `T_MAP_LEAF` | `1011` | a subject-map leaf (§4.2) |
| `T_MAP_NODE` | `1012` | a subject-map node (§4.2) |

## 3. The trail's statement proofs

### 3.1 What is replaced

A complete read replays each served trail its read needs, the imported
closure's included (pool-v3 §§10.1, 14). For each record of kind 1, 2, 3, 4 or
6 it reads the proof bytes, computes `proofHash` from them for the evidence
recurrence (§7) and verifies the proof under §11.4's key for its kind. A
succinct reader instead receives each record's proof-free form (§3.3) with its
`proofHash`, which the evidence recurrence authenticates. It takes "the proof
verifies" from a check proof over a span covering the record (§3.2), or checks
the record natively (§3.5). Kind 5 has no proof, and kind 7 is never a trail
event. Venue publications are not replaced (§3.6).

### 3.2 Digests, spans and their proofs

For a record of kind `k` in `{1, 2, 3, 4, 6}` with public inputs
`x_1 … x_m` in pool-v3 §5's order (`m` is 11, 15, 15, 16 and 17 respectively)
and `proofHash` `h`:

```text
leafDigest     = H(T_CHECK_LEAF, k, x_1, …, x_m, h.hi, h.lo)
nodeDigest(a, b) = H(T_CHECK_NODE, a, b)
```

A trail's **proved records** are its records of those five kinds in position
order, numbered from zero: `r_0, r_1, …`. For `s >= 0`, the **span** `(s, j)`
covers `r_(j·2^s)` through `r_((j+1)·2^s − 1)` and exists where the trail has
all of them. Its digest `D(s, j)` is the leaf digest of `r_j` at `s = 0`, and
`nodeDigest(D(s−1, 2j), D(s−1, 2j+1))` above.

The set's trail circuits are:
- **five leaves,** one per kind `k`, each holding §11.4's key for `k` as a
  constant. A leaf verifies one proof of 14,656 bytes (458 field elements,
  the length §11.4's backend proves) under that key with public inputs
  `x_1 … x_m`, computes `h` as SHA-256 over the proof's bytes, and outputs
  `leafDigest`. Its public inputs are `(nodeKeyHash, digest)`;
- **one node,** which verifies two children, each under one of the five leaf
  keys or under the node's own key, admitted by key hash (§2 rule 1), and
  outputs `nodeDigest` of the children's digests, left then right. Its public
  inputs are `(nodeKeyHash, digest)`.

A check proof **covers** span `(s, j)` where it passes §2 rule 2: at `s = 0`
under the pinned leaf key of `r_j`'s kind, at `s >= 1` under the pinned node
key; its `nodeKeyHash` equals the pinned node key's hash; and its digest equals
`D(s, j)` as the reader computes it. It then establishes that every record of
the span has a proof of 14,656 bytes whose SHA-256 is that record's
`proofHash` and which §11.4's verifier for its kind accepts on its public
inputs. The reader requires every record it takes as covered to state a proof
length of 14,656 (§3.3).

The two tags, and each kind's fixed number of inputs, separate a leaf's digest
from a node's. Without a collision of `H`, a covering proof therefore attests
exactly the reader's records, in the reader's order and shape. The leaf
digest carries the configuration through `x_1, x_2`, so digests of two
configurations never meet. A supplier answers a request for a digest with a
proof whose digest it is: one proof serves every trail holding those records
in that order, such as a prefix two checkpoints share or an adopted event's
source trail.

### 3.3 The proof-free trail

The **proof-free form** of a record that decodes under pool-v3 §5 is the
record with its proof bytes replaced by their SHA-256, the length kept:

```text
freeRecordBytes = statementBytes || u32 proofLength || proofHash[32] (only where proofLength > 0) ||
                  u32 authorizationLength || authorization[authorizationLength] ||
                  u32 capsuleCount || capsule_1[89] ... capsule_capsuleCount[89]
freeTrailBytes  = "moe/succinct/v1/trail" || u32 headerLength || segmentBytes[headerLength] ||
                  signedTerms_1 ... signedTerms_m || u64 n || freeEvent_1 ... freeEvent_n
freeEvent_i     = u32 recordLength || freeRecordBytes[recordLength]
```

The head (header, signed terms and count) and its checks are pool-v3 §10's,
and so are the size arithmetic, budgets, exact end and the rule that budgets
are local. Record lengths run from zero through 938, the largest proof-free
form: a spend's, of 538 statement bytes, 36 for its proof's length and
`proofHash`, 8 of lengths and 356 of capsules. The context is literal ASCII.

A proof-free record **decodes** exactly where a record with its statement,
proof length, authorization and capsules decodes under §5. §5's rules apply
unchanged, its proof-length rule to `proofLength`: zero exactly for kind 5,
otherwise positive, a multiple of 32 and at most 131,072. `proofHash` is
present exactly where `proofLength` is positive, and nothing follows the
capsules. §5's delivery association is checked as there. A record that §5
cannot decode has no proof-free form. A supplier holding one serves that
checkpoint's complete trail (§10), which the reader reads as a complete read
does: inconclusive by local authentication (§10.1), or under §9 where it
applies. A record therefore decodes in one form exactly where it decodes in
the other.

**Authentication** is pool-v3 §10.1's, each record's evidence triple
computed from its proof-free form: `statementHash` from `statementBytes`,
`proofHash` as carried or the zero sentinel where `proofLength` is zero, and
`signatureHash` from the authorization. The recurrence's terminal value must
equal the authenticated snapshot's `evidenceHash_n`. That authenticates, in
order, every statement, authorization, capsule vector, proof length and
`proofHash`; barring a SHA-256 collision, each `proofHash` names the committed
proof's bytes. Pool-v3 §12.1's prefix rule and §14's incremental retrieval
apply with this frame's head and events.

Whoever holds a complete trail derives its proof-free trail; nothing in it is
signed, and a replica may serve both.

### 3.4 Reading a trail succinctly

For each served trail the complete read would replay, the reader:

1. obtains and authenticates its proof-free trail (§3.3);
2. partitions the proved records into consecutive spans that exist and single
   records. Each span in the partition must be covered (§3.2); a span the
   reader finds no cover for, it splits, down to single records. A record
   outside every covered span is checked natively (§3.5);
3. replays under pool-v3 §14, taking "the proof verifies under the
   configuration's key for its kind" from the cover for a covered record and
   from the native check for every other. Every other check of replay runs
   as the complete read runs it: decoding, delivery association, Ed25519
   authorizations, note and scope paths, the spent set, totals, demands,
   locks, deadlines, clocks, lapse, imports, adoption, recovery, succession
   and fault classification;
4. compares, where replay compares a trail record with an adopted publication
   byte for byte (C2b.4.2), the publication's proof-free form with the trail's
   proof-free record. The reader holds the publication in full (§3.6), and two
   proof-free forms, which carry the proof's length and SHA-256, are equal
   exactly where the records are, barring a SHA-256 collision.

A record that the native check rejects is a deterministic failure as in the
complete read (pool-v3 §§7.1, 14), established over the bytes the reader
verified. A record that no cover reaches, and whose proof bytes nobody
supplies, leaves the read unresolved, as withheld evidence does. A
non-extension decided by the evidence recurrence (§7.1) and a lapse
(C2.10.4) read no proof and are reached as in the complete read.

For the trail of one segment the canonical partition of `N` proved records is
one span for each set bit of `N`, highest first. A later checkpoint that
extends a kept one adds only spans after its proved records, so a reader
that kept its state (pool-v3 §14) reads about `2·log2` of the new records'
count in check proofs.

### 3.5 Native checks and the tail

The native check of a proved record fetches, from any supplier, the bytes
whose length is the record's `proofLength` and whose SHA-256 is its
`proofHash`, which authenticate themselves, and verifies them under §11.4's
key for the record's kind, as the complete read does. The **tail** is the
records after the last a cover reaches, usually those no prover has reached
yet. Each costs its proof's bytes and one verification. A payee judging a
receipt's checkpoint (pool-v3 §7.2) judges it once every record through it is
covered or checked natively.

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
frames every transaction in it (§5). A succinct reader still accepts headers
itself from the anchor (venue-ergo §3) and reads the venue's chain only from
them. For heights from the adopted start through a fold's end, it takes the
outputs with venue-ergo §6's shape for its subject from the subject map
(§4.2) under a fold proof (§4.3). It fetches each such transaction with an
inclusion path (§4.5) and attributes its objects itself (venue-ergo §§5–6).
Above the fold's end it reads sections natively (§4.6).

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
from the leaves at zero, bit `l` of the key, least significant first, is one
where the path enters its parent as the right child. Empty subtrees are
`z_0 = 0` and `z_(l+1) = H(T_MAP_NODE, z_l, z_l)`, and the empty map's root is
`z_32`.

The **map at a header** `E` starts empty at the adopted start header (§6) and
appends every entry of every block from the start's child through `E`, in
order of height, position and output, to its key's chain. It is a function of
the start and the chain through `E`. For a transaction inside venue-ergo §5's
grammar, the entries are exactly its outputs with §6's shape, in output order.
For a transaction outside the grammar they are the outputs the adopted scan
reads as having that shape, its own deterministic rule fixed by the set's
identities; such a transaction carries no record (venue-ergo §5), so a reader
passes over its entries after reading it. Two subjects that share a key share
its chain, and a reader passes over entries that name another subject.

### 4.3 The fold proof

A fold proof's public inputs are, in order, `startHi, startLo, startHeight,
endHi, endLo, endHeight, mapRoot, nodeKeyHash, foldKeyHash`, the ids as limbs.
It establishes three things. The blocks from the child of the start header
through the end header form one chain by parent ids, one height apart. Each
block's transactions are exactly those whose ids, with witness ids where the
root uses them, reproduce its header's transactions root under one of
venue-ergo §4's rules, in that order. `mapRoot` is the map at the end header
(§4.2). The set's leaves, node and folds prove this in periods: a first fold
starts at the adopted start, and each next fold verifies the previous fold,
under the first fold's key (a constant) or its own (`foldKeyHash`), ending
where its period starts.

The reader uses a fold proof where, under §2 rule 2:
1. it verifies under the pinned first-fold or next-fold key;
2. `nodeKeyHash` and `foldKeyHash` equal the pinned node and next-fold key
   hashes;
3. `startHi, startLo, startHeight` are the adopted start's;
4. `endHi, endLo` are the id of the header at `endHeight` on the reader's
   best chain, at least the venue's depth below its tip.

The proof attests no header's work. The reader's own header chain carries the
work, and the fold's end binds the scanned blocks to that chain by parent ids.
A fold at the depth holds only blocks the venue's finality rule fixes.

A **path** for a key at a fold is its leaf's count and chain value and 32
siblings; the reader recomputes `mapRoot` from them.

### 4.4 Reading a range

For a request of record kind `k`, subject `s` and indices `from` through `to`,
with `to` witnessed (venue-ergo §2) and the adopted start at height `A` or
below, the reader:

1. takes a fold proof under §4.3 with end height `E`;
2. reads the path for `s`'s key at that fold, and the key's entries from any
   supplier. It recomputes the chain from `chain_0`, or from a count and chain
   value it kept at an earlier fold of the same set, to the leaf's count and
   chain value; the entries must be exactly as many as the count grew;
3. for each entry naming `s` with height from `A + 1 + from` through the
   lesser of `A + 1 + to` and `E`, fetches the transaction at that height and
   position with an inclusion path (§4.5) to the transactions root of its
   header on the reader's chain. It reads the transaction under venue-ergo §5
   and attributes its objects under §6 for kind `k`: outputs at `script_k`
   with subject `s`, and for kind 4 their runs reassembled from the
   transaction's adjacent outputs. Each object enters at index
   `height − A − 1` with its venue-ergo §6 ordinal. A transaction outside the
   grammar, or an output at another location, contributes nothing;
4. reads every height above `E` through `A + 1 + to` natively (§4.6).

The answer assembled is the succinct read's answer to the request, in
pool-v3 §13.1's frame, and the reader applies §13.3 to it unchanged. Absence
follows from the count: where the count and chain value at the fold equal
those the reader kept at an earlier fold `E′` of the same set, the venue
holds no output with the shape for any subject of that key from `E′ + 1`
through `E`; a count of zero means none since the start. The reader keeps
the key's count, chain value and `E` to continue from.

### 4.5 Inclusion paths

An inclusion path for the transaction at position `j` of a block is its
unsigned bytes and a list of at most 32 steps, each a 32-byte sibling or a
lift. The reader starts from `Blake2b-256(0x00 || id)`, where `id` is the
Blake2b-256 of the unsigned bytes. At step `l`, where bit `l` of `j` is zero a
sibling is hashed to the right as `Blake2b-256(0x01 || current || sibling)`
and a lift as `Blake2b-256(0x01 || current)`; where it is one, the sibling is
hashed to the left, and a lift is refused. The last value must equal the
header's transactions root, and `j` must have no set bit at or above the
path's length. These are venue-ergo §4's leaf, node and lone-child hashes.
Without a Blake2b collision, a path reproducing the root is the real one, for
the transaction at position `j` of the section, under either of §4's rules.

### 4.6 Native fallback and costs

Where a fold, path, entry list, transaction or inclusion path is missing or
fails, the reader reads the range's sections natively (venue-ergo §§4–7), or
the read stays unresolved. Above the latest usable fold's end it reads
natively; how far that lies behind the tip is the provers' pace. A fold of
another set or start is not used.

A path names its key, so a supplier learns which subjects a reader follows.
Reading the whole map, or paths for other keys as well, is the reader's lever.
Every output with the shape that names a key costs that key's readers a
44-byte entry, and one naming their subject costs a transaction fetch; the
venue's minimum prices both (venue-ergo §8), and a stranger can already add
such outputs to a complete reader's sections.

## 5. C0a cost and replacement

This replaces no rule of any construction or venue. For the reader that
chooses it, it replaces the trail's proof bytes and their verification, 14,656
bytes and one verification a proved record, by a proof-free record of at most
938 bytes and a share of a span's check proof. It also replaces a range's
section bytes, about 6 MB a day of the Ergo mainnet, by a fold proof, a path
of about 1 KB, the subject's entries and their transactions. A succinct reader
still accepts every header from its venue's anchor, replays every proof-free
record and verifies venue publications, signatures, failures and the tail
natively. Measurements are in the reference's
[aggregation](https://github.com/mediumofexchange/reference-ts/blob/main/docs/POOL_DEPLOYMENT_PROBES.md#the-trails-proof-aggregation-slice-33)
and [exhaustion](https://github.com/mediumofexchange/reference-ts/blob/main/docs/VENUE_ALTERNATIVES.md#the-complete-exhaustion-leaf-slice-32)
probes.

It is one mechanism: one contract (§2) with two instances, one hash
(pool-v2 §1's `H`, five tags), one proof system (pool-v3 §4's) and one frame.
Its costs are a circuit set of twelve circuits, which needs adversarial review
against its pinned sources; provers, which any party runs (one 4-vCPU machine
kept up with the design point's mean trail 1.6 times, and with an ordinary
Ergo day 3.4 times, in those probes); and the trust in §2, held alike by
every reader of one set. A circuit defect reaches every such reader, across
backings, and no complete reader.

Alternatives:
- **A successor construction.** A pool-v4 permitting aggregation as a compact
  replacement beside §9.1, and a venue-ergo context taking a fold as evidence,
  would make the circuits' agreement part of every reader's verdict, so a
  circuit defect would split complete readers too. Every backing would reach
  it only by successor and swap, to gain what no backer or operator acts on:
  any party proves, and no verdict is served (pool-v3 §12.1). The circuits a
  reader pins are the same either way.
- **A reader's verdict.** A phone asking another reader for its verdict takes
  a gatekeeper (C2.10.13).
- **Checking only one's own notes.** Counterfeit value passes.
- **Fault evidence alone.** Its absence proves nothing; an eclipsed reader
  would accept counterfeit value. It stays a complement (§2 rule 7 is its
  form for circuits).
- **Every proof natively.** No circuit: about 13 MB a day at the design
  point's mean and 14.7 GB for a restore at 10⁶ statements. It stays the
  fallback, and serves a small scope.
- **A relation over the whole verdict.** One circuit reproducing every
  kind, lock, clock, recovery and fault, with Ed25519 refusing exactly what
  the host refuses, as every reader's verifier. It is costlier to prove and
  to get right than proving only the proofs (pool-v3 §14's list).
- **The probes' digests.** Chained permutations under ad hoc tags; pool-v2
  §1's sponge with new tags serves with no second hash rule.

No pool-v4 candidate is needed. A Poseidon2 proof digest in a later
construction would roughly halve a leaf, since SHA-256 is most of it; it is a
prover's lever, not a reader's need.

## 6. Adoption

No circuit set is adopted. An adoption names, under `moe/succinct/v1`:
- for each circuit (five trail leaves and the trail node; the exhaustion's
  three leaves, node, first fold and next fold), its sources', bytecode's and
  key's SHA-256 identities and its key hash, the compiler, and the pinned
  backend and verifier target. Proving reads Ignition's leading G1 points up
  to the largest circuit, 2^22 in the probes, under hashes the adoption names
  as pool-v3 §4 does;
- the start header of the exhaustion: its id and height on the Ergo mainnet,
  at or below the anchors of the venues it serves;
- the agreement evidence: each leaf's verdict equal to §11.4's verifier on
  genuine and hostile proofs, the in-circuit verifier's range and curve
  constraints read from the pinned backend's source, and the scan's entries
  equal to venue-ergo §§5–6 on every transaction inside the grammar, with
  adversarial review of every circuit.

As in pool-v3 §11.4, the identities are named after every rule they
implement. A defect needs a new adoption, which each reader selects; a reader
that drops a set discards what it derived under it (§2 rule 8).
