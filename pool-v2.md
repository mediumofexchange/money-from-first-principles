# The shielded pool, construction v2

### The replacement-capable construction's layouts, bit for bit

[Construction §C1.2](construction.md#c12-the-shielded-pool) says what the shielded pool is and what a statement proves, and [the authority and history contract](pool-authority.md) says how private spends stay authorized when backings replace their operators independently (C1.2.1–2, C2.10.1–9). This document fixes one construction of both, **`moe/pool/v2`**, at the level two implementations must agree on: field encodings, hash functions and domain tags, the note and its commitment and nullifier, the note tree and the accepted-root forest, the public scope and its authenticated root, the segment header and its identity, the three statements and their public-input order, admission, the ordered history, the snapshot digest and the directory, the receipt's bytes, finalized import and replay, the spent-set accumulator, and the proof system. **E** names this construction and a configuration hash ([§C1.3](construction.md#c13-what-e-declares-for-the-construction)); everything here is fixed by that name. A change to anything below is `moe/pool/v3`, and a backing moves to it by successor.

This supersedes [`moe/pool/v1`](pool-v1.md), whose configuration bound the operator into the immutable domain and whose statements carried no authority relation; [§14](#14-what-this-replaces-and-what-it-costs) lists what changed and why. v1 never carried claims, so nothing crosses from it. [§15](#15-status) records the pinned artifacts and what is still open.

Notation. `‖` is byte concatenation. `frame(…)` is the framed byte string of [§1](#1-fields-hashes-and-encodings); `SHA256(…)` is SHA-256 over such a frame. `H(…)` is the in-circuit hash. `u8`, `u32`, `u64` are unsigned big-endian integers of that width. `F` is a field element written as 32 big-endian bytes. Bits of a 256-bit value are numbered from the most significant as bit `0`.

## 1. Fields, hashes and encodings

**The field.** `p = 21888242871839275222246405745257275088548364400416034343698204186575808495617`, the BN254 scalar field. A field element is canonical only in `[0, p)`; a value at or above `p` is malformed wherever it appears. Written as bytes, a field element is 32 big-endian bytes; written as text, `0x` followed by exactly 64 lowercase hexadecimal digits.

**The in-circuit hash `H`.** Poseidon2 over BN254 with state width 4 and rate 3, the permutation being the Noir standard library's `poseidon2_permutation`, in the variable-length sponge of `noir-lang/poseidon` v0.3.0 (`src/poseidon2.nr`, SHA-256 `44f3a3d1abe7d5fa2da5c0339e52018195d55f295c320e530d355f9cc62159d8`): that file's `Poseidon2::hash`, not its `Poseidon2Hasher`, which is a different sponge. Precisely: to hash `n` field elements, `n` at least one since every object below carries a tag, the state starts as `[0, 0, 0, n · 2^64]`; inputs are added into the first three state elements three at a time, each full block followed by one permutation; a final partial block of one or two inputs is added and permuted once more; when `n` is a positive multiple of three no partial block exists and no extra permutation runs; the output is the first state element. Every in-circuit object is hashed with a **domain tag** as its first input:

| Tag | Value | Used for |
|---|---|---|
| `T_OWNER` | `1001` | an owner from a spend secret |
| `T_NOTE` | `1002` | a note commitment |
| `T_NULLIFIER` | `1003` | a nullifier |
| `T_NODE` | `1004` | a note-tree node |
| `T_SCOPE_LEAF` | `1005` | a scope entry |
| `T_SCOPE_NODE` | `1006` | a scope-tree node |

**Frames and `SHA256`.** A frame is a context string written raw as UTF-8 with no length prefix, followed by fields in the order the definition lists them, each fixed-width (`F`, a 32-byte identifier, `u8`, `u32`, `u64`) or `u32`-length-prefixed where variable. No context string in this document is a prefix of another. Two different inputs never produce one byte string. The reference implementation's `ByteWriter` is this convention, and `SHA256(…)` is SHA-256 over the frame.

**Identifiers as field elements.** A 32-byte identifier (the construction domain, a segment identity, a backing name, a replacement link) enters a circuit as two limbs: `hi` is the first 16 bytes and `lo` the last 16, each read big-endian as an integer below `2^128`. A circuit range-checks both limbs. Reducing a 32-byte value modulo `p` is not a representation of it.

**Quantities.** A quantity is a `u64`, positive where the rule says positive. Sums are computed after widening to 128 bits; equality in the field is never a substitute for integer equality. Running totals per backing stay below `2^64`, and admission refuses what would exceed that ([§8](#8-admission)).

## 2. What E declares, and what the configuration fixes

Beside the original operator, venue, interval, replacement rule and silence clause, **E** carries two fields for the claim layer:

- `construction`: the ASCII string `moe/pool/v2`;
- `configuration`: 32 bytes, the **configuration hash** below.

Both are inside the backing's name. **The configuration hash is the construction domain** (C1.2.1): it fixes the relations, hashes, proof system, circuit identities and bounds, and it contains neither an operator nor a segment. **E**'s `operator` is the original operator, the genesis link of the backing's replacement chain (Construction §C2.5); current authority follows that witnessed chain and never changes the domain, the backing name, a note commitment or a nullifier. A backing that declares identified issuance ([Construction §C1.1](construction.md#c11-issuance-is-logged-in-the-open-and-the-recipient-is-not)) cannot be served under v2, which carries no recipient field.

**The configuration** is a document whose bytes are exactly the preimage below, and whose hash is

```text
configHash = SHA256(
  "moe/pool/v2/config"
  ‖ bytecode(issue)[32] ‖ vk(issue)[32]
  ‖ bytecode(spend)[32] ‖ vk(spend)[32]
  ‖ bytecode(burn)[32]  ‖ vk(burn)[32]
  ‖ helper[32] ‖ u8 noteTreeDepth ‖ u8 scopeDepth ‖ u8 inputs ‖ u8 outputs
)
```

where `bytecode(k)` is the SHA-256 of the bytes that circuit `k`'s compiled artifact's `bytecode` field base64-decodes to, exactly as the compiler emits them, `vk(k)` the SHA-256 of the verification-key bytes the backend's key derivation returns for that bytecode, `helper` the SHA-256 of the Poseidon2 helper source, and the four bounds are `32`, `16`, `2`, `2` in this version. Every implementation that compiles the pinned sources under the pinned toolchain derives the same configuration, so one domain is shared by every backing, operator and venue that names it: that sharing is what lets a note keep its identity across operators.

Before accepting a note a wallet checks that the backing's **E** names `moe/pool/v2`, that its `configuration` equals the hash of the configuration the wallet holds, that the note's domain is that hash, and that the configuration's circuit and key identities are the ones the wallet has pinned. Which operator currently serves the backing is read from the witnessed replacement chain, never from the configuration. A verifier never accepts a verification key supplied with a statement.

## 3. Notes

A **note opening** is `(domain, backing, value, owner, rho)`: the construction domain and the backing name as limbs, a `u64` value, and two field elements. The **spend secret** `secret` is a nonzero field element the note's receiver derives and never reveals; `rho` is a nonzero field element the note's creator derives.

```text
owner = H(T_OWNER, secret)
cm    = H(T_NOTE, domainHi, domainLo, backingHi, backingLo, value, owner, rho)
nf    = H(T_NULLIFIER, domainHi, domainLo, cm, secret)
```

`owner`, `cm` and `nf` must be nonzero. The nullifier is a function of the immutable note and its owner's secret alone: neither the operator, the segment, the scope, the anchor nor the path a note is later proved under enters it, so one note has one nullifier however and wherever it is proved (C1.2.1).

**The receiver generates the secret** (invariant 25) and hands the payer only `owner`; the payer builds the output note and delivers the opening `(backing, value, rho)` to the receiver privately, with the statement's `statementHash` and segment identity. The receiver recomputes `cm`, requires `value > 0` — a zero-value output is not a payment — and checks `cm` among the outputs of a statement in a history the receiver has itself verified up to a witnessed commitment, or a statement whose receipt the receiver accepts as the operator's liability ([§9](#9-the-ordered-history-the-snapshot-and-the-receipt)); an operator's word alone is neither. A receiver derives a **fresh secret per expected payment**; an `owner` reused across payments lets the payers that received it link those payments.

**Randomness is derived, not drawn** (invariant 26): a wallet derives every `rho` it creates, and the `(rho, secret)` of every padding input ([§7.2](#72-spend)), deterministically from its own root secret and the statement's real input nullifiers (for an issuance, from the obligor's root secret and the issuance's `owner`), so that a statement rebuilt after a crash, or re-proven under a new segment after a lapse (C2.10.9), keeps its nullifiers and its outputs. The protocol cannot check this; a wallet that draws fresh randomness on retry is refused as a double spend and has lost nothing but its own receipt.

## 4. The note tree and the accepted-root forest

Each segment ([§6](#6-the-segment)) has its own **note tree**: a binary Merkle tree of depth `32` over `H`, append-only, holding that segment's local outputs only. Leaf `i` is the commitment of the `i`-th output accepted in the segment, counting from zero across every statement in acceptance order, and an unused leaf is the field element `0`. A node is hashed with the level of its **children**, `0` for the two leaves under it and `31` for the two children of the root:

```text
node(l, left, right) = H(T_NODE, l, left, right)         l is the children's level
z_0 = 0,   z_{l+1} = node(l, z_l, z_l)                     the empty subtree with its top at level l+1
```

The root is the node at level `32`, so the empty tree's root is `z_32`. A **path** for a leaf is 32 siblings, one per level from `0` to `31`, with 32 direction bits, `true` where the node on the path is the right child at that level.

**The accepted-root forest** (C2.10.7) is the set of anchors a statement may prove membership against: `z_32`; the segment's own root after each accepted local statement; and every root certified by the segment's imported finalized prefixes ([§10](#10-import-and-replay)), which are those prefixes' own local roots and, transitively, the roots they imported. No leaf of an imported tree is copied into the local tree, and no root of an unfinalized or abandoned tail is in the forest. A statement's inputs may name different anchors, each an accepted root. The tree holds `2^32` leaves; a statement whose outputs would not fit is refused before it changes state, and a segment whose tree is full accepts no further outputs.

**A wallet computes its own paths.** It syncs each segment's full leaf list, in bulk, and builds paths locally against a root of that segment; it never asks a server for one leaf's path or index, and a receiver never fetches one `statementHash`, since either tells the server which note that wallet holds. A note created in an earlier segment is spent against a certified root of that segment, and the wallet keeps the leaves it needs for the path.

## 5. The scope

A **scope** (C1.2.2) is the public, sorted set of backings one operator serves together under one domain on one venue, each with the replacement link under which the operator holds it. An **entry** is `(backing, link)`: the backing's 32-byte name and the 32-byte identity of its current link in the witnessed replacement chain — the backing's own name while the original operator **E** names is in force, and otherwise the identity of the witnessed replacement that seated the current operator, as the publication layer defines that identity (Construction §C0b, §C2.5.1). Entries are sorted strictly ascending by the backing name's bytes, so a scope has no duplicate backing; a scope holds at least `1` and at most `2^16` entries.

The scope is authenticated in-circuit by a Merkle tree of depth `16` over `H`: entry `i` is leaf `i`, counting from zero in sorted order, an unused leaf is `0`, and

```text
leaf(backing, link)   = H(T_SCOPE_LEAF, backingHi, backingLo, linkHi, linkLo)
snode(l, left, right) = H(T_SCOPE_NODE, l, left, right)   l is the children's level
s_0 = 0,   s_{l+1} = snode(l, s_l, s_l)
scopeRoot             = the node at level 16
```

A **scope path** is 16 siblings with 16 direction bits, as a note path is. A statement proves that the backing of every note it spends or creates is an entry of the scope whose root it names ([§7](#7-statements)); which entry, and under which link, stays private. A scope root is the operator's assertion of nothing: admission checks it against the segment's own scope ([§8](#8-admission)), and the segment's scope is checked against the record by the sequencing rules (C2.10.1).

## 6. The segment

A **segment** (C2.10.1) is one operator's service of one scope from one exact opening state per backing. Its **header** is

```text
segmentBytes = frame("moe/pool/v2/segment" ‖ configHash[32] ‖ venue[32] ‖ operator[32] ‖ u64 sequence
                     ‖ u32 n ‖ entry_1 … entry_n)
entry_i      = backing[32] ‖ link[32] ‖ u64 openingSequence ‖ openingOperator[32] ‖ openingRoot[32]
segmentId    = SHA256 over segmentBytes
```

where `venue` is the identity of the venue every scoped backing's **E** declares (C2.10.2), `operator` is the Ed25519 public key that admits statements and signs receipts and commitments for this segment, `sequence` is the sequence of the first commitment the operator signs for this segment — one past the highest it has signed on this venue (Construction §C2.4.1), which is what distinguishes successive segments of one operator over one scope — and the `n` entries are the scope of [§5](#5-the-scope) in the same sorted order, each with its **opening**: `openingSequence = 0` with sixty-four zero bytes where the record pins nothing for the backing (the empty book, Construction §C2.7.3), and otherwise the operator, sequence and directory root of the exact commitment that fixes the backing's opening state (C2.7.1, C2.10.4), with `openingSequence ≥ 1`. Where an opening commitment's operator is this segment's operator, its sequence is below `sequence` (C2.10.5). `segmentId` is the segment's identity in every statement, receipt, history and snapshot below; a changed scope, link, opening or sequence is another segment.

The header fixes what the segment will serve. Which links are in force, whether each opening is the backing's canonical predecessor, and whether the segment's checkpoints are final are the sequencing rules' to check against the record (C2.10.1, C2.10.3–4, Construction §C2.7); this document fixes the bytes those rules read and the state a segment computes from them.

## 7. Statements

A **statement** is `(kind, publicInputs, proof)`, plus `obligorSignature` for an issuance. `kind` is `1` for issue, `2` for spend, `3` for burn. `publicInputs` is the list of field elements below, in the verifier's order and of exactly the stated length; every kind begins with the domain, the segment identity as limbs, and the scope root, so that a proof is a proof for one segment and no other (C1.2.2, C2.10.8). `proof` is the proof bytes of [§12](#12-the-proof-system).

The **statement bytes** frame what a statement asserts, independently of its proof, and their hash names the statement:

```text
statementBytes = frame("moe/pool/v2/statement" ‖ configHash[32] ‖ u8 kind
                       ‖ u32 n ‖ publicInputs[0] … publicInputs[n−1])      each an F
statementHash  = SHA256 over statementBytes
```

`statementHash` is the statement's identity in admission, receipts, the history and a payer's delivery to a receiver. Two statements with one `statementHash` are one statement, whatever their proof bytes. Since the segment identity is among the public inputs, rebuilding a statement under another segment is another statement with another proof; its nullifiers and, with derived randomness, its outputs are unchanged (C2.10.8).

### 7.1 Issue

```text
publicInputs = [domainHi, domainLo, segmentHi, segmentLo, scopeRoot,
                backingHi, backingLo, quantity, cm]                                        n = 9
witness      = owner, rho, link (as limbs), scopeSiblings[16], scopeRight[16]
```

The circuit proves `quantity > 0`, `owner ≠ 0`, `rho ≠ 0`, `cm ≠ 0`, `cm = H(T_NOTE, domainHi, domainLo, backingHi, backingLo, quantity, owner, rho)`, and that the scope path carries `leaf(backing, link)` to `scopeRoot`; every limb is below `2^128` and `quantity` below `2^64`.

`obligorSignature` is the backing's obligor **K** signing `statementBytes` with the strict Ed25519 signature the reference uses for every other signed object, so an issuance is authorized for one domain, one segment and one scope (C1.2.2). A signature over the backing's terms, or over anything but these exact bytes, authorizes nothing.

### 7.2 Spend

```text
publicInputs = [domainHi, domainLo, segmentHi, segmentLo, scopeRoot,
                anchor_1, anchor_2, nf_1, nf_2, cm_1, cm_2]                                n = 11
witness      = for each input i in {1, 2}: note_i (backing_i, value_i, owner_i, rho_i), secret_i,
               siblings_i[32], right_i[32], link_i, scopeSiblings_i[16], scopeRight_i[16];
               for each output j in {1, 2}: note_j
```

The circuit proves, for each input `i`:

- `secret_i ≠ 0` and `owner_i = H(T_OWNER, secret_i)`;
- `nf_i = H(T_NULLIFIER, domainHi, domainLo, cm(note_i), secret_i)` and `nf_i ≠ 0`;
- **where `value_i > 0`**, that the path `(siblings_i, right_i)` carries `cm(note_i)` to `anchor_i`;
- **where `value_i = 0`**, nothing about the tree: the input is **padding**, its commitment need not exist anywhere, and its nullifier is whatever the formula gives for the wallet's derived `rho_i` and `secret_i`;
- that the scope path `(scopeSiblings_i, scopeRight_i)` carries `leaf(backing_i, link_i)` to `scopeRoot`, padding included.

And across the statement:

- every output names the backing of one of the inputs: `backing_out_j ∈ {backing_1, backing_2}` for each `j`, so an output's scope membership is its input's; a padding input names the backing of the other input, so a single-backing spend has one backing throughout and a two-backing spend has two real inputs;
- **quantities conserve per backing**, in 128-bit arithmetic: for each backing `b` among the inputs, the sum of input values naming `b` equals the sum of output values naming `b`; every value is below `2^64`, and `value_1 + value_2 > 0`;
- `nf_1 ≠ nf_2`; `cm_1 = H(T_NOTE, … note_out1)`, `cm_2 = H(T_NOTE, … note_out2)`, `cm_1 ≠ cm_2`, both nonzero; every output's `owner` and `rho` nonzero.

A spend reveals the segment, the scope root, two anchors, two nullifiers and two commitments, and nothing else: the anchors disclose which histories the inputs came from, not the leaves (C1.2.2). The two anchors may differ; the anchor of a padding input is any accepted root, and admission checks both ([§8](#8-admission)). An output of value `0` is allowed and occupies a leaf. A two-backing spend is how a payment from two scoped backings, or a two-backing exchange inside one scope, is one statement (Construction §C1.2, §C1.7); a payment from more than two backings, or across scopes, is several statements. A padding input's nullifier enters the spent set like any other.

### 7.3 Burn

```text
publicInputs = [domainHi, domainLo, segmentHi, segmentLo, scopeRoot,
                backingHi, backingLo, quantity, anchor_1, anchor_2, nf_1, nf_2, cm_change]   n = 13
witness      = for each input i: note_i, secret_i, siblings_i[32], right_i[32];  note_change;
               link, scopeSiblings[16], scopeRight[16]
```

The circuit proves, in full: for each input, `secret_i ≠ 0`, `owner_i = H(T_OWNER, secret_i)`, `nf_i = H(T_NULLIFIER, domainHi, domainLo, cm(note_i), secret_i)`, `nf_i ≠ 0`, and, where `value_i > 0`, the path to `anchor_i`; `nf_1 ≠ nf_2`; every note, input and change, names the public `backing`, and one scope path carries `leaf(backing, link)` to `scopeRoot`; `value_1 + value_2 = quantity + value_change` in 128-bit arithmetic with every value below `2^64` and `quantity > 0`; `cm_change = H(T_NOTE, … note_change)`, `cm_change ≠ 0`, `owner_change ≠ 0`, `rho_change ≠ 0`; every limb below `2^128`.

A burn is destruction of claims: it lowers `outstanding(backing)` by `quantity` and proves nothing about the external payout.

### 7.4 Redemption, and what this version does not carry

Redemption is a spend whose output the backer owns, the backer having generated that output's secret; it is not a statement kind, and it leaves `outstanding` unchanged (invariant 10).

**v2 defines issue, spend and burn, the scope and the segment, admission, the history, the snapshot and directory, the receipt, and finalized import and replay.** The following Construction objects remain outside it and must be specified together before a later version claims them:

- presentation (§C3): the demand naming claims by `H(nullifier)`, the spent-pending lock, acceptance, release, and invariant 27's settlement;
- the non-service object (§C2b.5) — so **E**'s non-service grade is inert for a v2 backing, and its two durations bind nothing;
- snapshot redemption's venue leg and the adoption of venue-witnessed nullifiers on return from silence (§C2b.3–C2b.4) — v2 defines the non-membership proof ([§11](#11-the-spent-set)) but no record that puts a nullifier into the spent set without a statement;
- the atomic swap (§C1.7) across operators, and with it invariant 23's demand record and pending-lock set and §C1.5's aborted-presentation rule; a two-backing exchange inside one scope is one spend already;
- the bridge between venues (C2.10.2) and snapshot adoption across constructions.

Operator replacement itself needs no object beyond Construction §C2.5's witnessed replacement and this document's segment: a successor opens a segment whose entries name the links that seated it and the openings the record fixes.

Construction C2b.3a–c fixes the later shielded recovery contract: finalized holdings, no redirection through unwitnessed spends and no associated challenge window, and recovery settlements that preserve consent and supply. It adds no v2 record, proof or admission path. This version's configuration, frames and circuits are unchanged; the excluded objects above remain excluded. [The presentation and recovery contract](pool-recovery.md) now specifies those objects (C3.1–8, C2b.3.1–3, C2b.4.1–2, C2b.5.1–2, C2b.6.1) for a later version to instantiate; v2 carries none of them.

## 8. Admission

The operator admits a statement against **one committed view**: its segment's header, its imported finalized prefixes ([§10](#10-import-and-replay)) and its own accepted local statements. First, if a statement with this `statementHash` has already been accepted in this segment, it returns that statement's receipt and changes nothing (invariant 26): an exact resubmission is the same statement whatever its proof bytes. Otherwise it changes state only if every check passes:

1. `publicInputs` has the length for `kind`, every element is a canonical field element, `domainHi ‖ domainLo` is this configuration's hash, `segmentHi ‖ segmentLo` is this segment's identity, and `scopeRoot` is this segment's scope root.
2. The proof verifies against the configuration's verification key for `kind` and these public inputs, and against nothing else.
3. For an issue or burn: `backingHi ‖ backingLo` is an entry of this segment's scope, and the operator holds that backing's signed terms, whose **E** names `moe/pool/v2` and this configuration hash — **E**'s original operator is not compared, since current authority is the record's (C1.2.1); for an issue, `obligorSignature` verifies under that backing's **K** over `statementBytes`, and `issued(backing) + quantity < 2^64`; for a burn, `outstanding(backing) ≥ quantity`.
4. For a spend or burn: each of `anchor_1` and `anchor_2` is in the accepted-root forest, the padding input's included; no nullifier of the statement is in the spent set, and the statement's nullifiers are distinct.
5. Every output commitment is absent from the segment's output set — its imported closure's outputs and its own — the statement's outputs are distinct and nonzero, and they fit under `2^32` local leaves.

Then, as one transition: the outputs are appended to the local tree in order and the new root joins the forest, the nullifiers are inserted into the spent set, `issued(backing)` or `burned(backing)` moves, the statement is appended to the history, and the receipt is signed. A statement that fails any check leaves no trace. A spend is admitted without the operator learning which scoped backings it moved; the proof's scope membership is the whole of its authority check (C1.2.2).

## 9. The ordered history, the snapshot and the receipt

The segment's public history is the sequence of accepted local statements `s_1, s_2, …`, numbered from `1`; the number is the statement's **position**, and `(segmentId, position)` is the event's identity (C2.10.6). After statement `i` the segment has a local note-tree root `noteRoot_i`, a combined spent-set root `spentRoot_i` ([§11](#11-the-spent-set)) and per-backing totals; before any local statement, `noteRoot_0 = z_32` and `spentRoot_0` is the root of the spent set holding exactly the imported closure's nullifiers ([§10](#10-import-and-replay)), `e_256` where nothing was imported. The history is bound by a hash chain from the segment's identity:

```text
historyHash_0 = SHA256("moe/pool/v2/genesis" ‖ segmentId[32])
historyHash_i = SHA256("moe/pool/v2/history" ‖ historyHash_{i−1}[32] ‖ statementHash_i[32]
                       ‖ noteRoot_i[32] ‖ spentRoot_i[32] ‖ u64 i)
```

`historyHash_i` binds the segment, its openings and imports, the order of every local statement, every nullifier and every accepted root up to `i`; two histories with equal note roots and different spent sets have different history hashes (invariant 23).

**The receipt** for statement `i` is the operator's signature over the domain, the segment identity and its scope root, `i`, `statementHash_i`, `historyHash_i`, the `SHA256` of the proof bytes it verified and, for an issuance, of the `obligorSignature` it verified — so that the operator's signature attests the exact evidence it admitted and a stranger served bad bytes can tell an operator's fault from a replica's corruption — and the sequence of the commitment the operator last signed (C2.10.8, Construction §C2b.4). Its bytes are

```text
receiptBytes = frame("moe/pool/v2/receipt" ‖ configHash[32] ‖ segmentId[32] ‖ scopeRoot[F] ‖ u64 i
                     ‖ statementHash_i[32] ‖ historyHash_i[32] ‖ proofHash[32] ‖ signatureHash[32] ‖ u64 after)
```

where `proofHash = SHA256(proof)` over the proof bytes exactly as admitted, `signatureHash = SHA256(obligorSignature)` for an issuance and thirty-two zero bytes for a spend or a burn, and `after` is the sequence of the commitment the operator had last signed when it co-signed, `0` where it had signed none. Commitment sequences count from `1` (Construction §C2.4.1), so `0` names no commitment; a segment commits its opening state before it co-signs (C2.10.9), so a receipt of a segment names a commitment of that segment. The receipt is served as these fields, the operator's Ed25519 public key, and the operator's strict signature over `receiptBytes`; a receipt under any other bytes, under an operator other than the segment header's, or for a statement whose `statementHash` the served history does not hold at `i`, attests nothing. Strict verification establishes the signature; whether the signer's scope and links were in force is the record's to say (C2.10.8). A stranger resolves `after` against the record, which answers *holds*, *not reached* or *moved past* (Construction §C2.3.4, §C2b.4).

**The commitment** (Construction §C2.4, C2.10.3) is the operator's signature over its sequence and the root of a directory that carries **every** backing of the segment's scope with its snapshot digest at the segment's length when the commitment was signed. A directory that omits a scoped backing, or carries one under another segment's digest, is not a commitment of this segment. The directory's byte framing and the commitment's signature context are the reference implementation's to fix and to version (Construction §C0b); this construction fixes only the digest.

**The snapshot digest** the directory carries for a scoped backing `b` is

```text
snapshot(b) = SHA256("moe/pool/v2/snapshot" ‖ b[32] ‖ segmentId[32] ‖ historyHash_n[32]
                     ‖ u64 issued(b) ‖ u64 burned(b))
```

at the segment's current length `n`, where `issued(b)` and `burned(b)` are the totals over the deduplicated imported closure plus the local history (C2.10.7). Every backing in one segment shares the segment's history hash; a stranger verifying one backing replays the segment's local history and its supplied ancestry, since a spend does not say which backing it moved.

**The served trail** of a segment is its header, the signed terms of every backing that any issue or burn in its history names, and the history with, for every statement, its proof and (for an issuance) its obligor signature. Without a backing's terms the issuances naming it cannot be verified and the replay stops at the first such statement.

## 10. Import and replay

A **finalized prefix** of a segment `S` at length `n` is what a verifier holds after replaying `S`'s trail through its first `n` statements: `S`'s header; its **events**, the deduplicated closure of the events of `S`'s own imported prefixes together with `(S, 1) … (S, n)`, each event being `(segmentId, position) → statementHash` with the statement's nullifiers, output commitments and, for an issue or burn, its backing and quantity; its **roots**, the union of its imported prefixes' roots with `noteRoot_0 … noteRoot_n`; `historyHash_n`; and its directory at `n`. Only a checkpoint the record finalized makes a prefix importable (C2.10.3–5); this section fixes what a verifier does with the one it is handed.

**Evidence for an opening.** For every entry whose opening is a commitment `(o, q, ρ)`, the importer is given the commitment itself with its signature and the directory whose root it signed, the trail of the segment `S` that commitment checkpointed, and the length `n` it checkpointed. It refuses the opening unless:

- the commitment verifies strictly under `o`, its sequence is `q`, and the directory's root is `ρ`;
- `S`'s header names this configuration hash and this venue (C2.10.2), and its operator is `o`;
- replaying `S`'s trail through `n` statements, with `S`'s own openings imported by these same rules, yields a prefix whose directory equals the commitment's directory, entry for entry — every scoped backing of `S`, each at `snapshot(b)` over `S`'s identity, `historyHash_n` and its totals (C2.10.3);
- the entry's backing is in that directory.

A replay walks the supplied evidence, verifies each checkpoint once, and never discovers a global graph; an opening whose evidence is missing, whose ancestry returns to a segment still being replayed, or whose checkpoint fails a check stops the replay (C2.10.5).

**Merging the prefixes** (C2.10.6–7). One prefix per distinct opening commitment is merged into the segment's opening state:

- events are keyed by `(segmentId, position)`; two prefixes that hold one key must hold one `statementHash` for it, else the import is refused;
- across the distinct events, every nullifier is distinct and every output commitment is distinct, else the import is refused: a set union does not repair a conflict;
- the **spent set** is the accumulator holding exactly the distinct events' nullifiers, whose root does not depend on the order they were inserted; the **output set** is the distinct events' outputs; the **forest** is the union of the prefixes' roots with `z_32`;
- `issued(b)` and `burned(b)` are the sums over the distinct events' issues and burns naming `b`, and the import is refused where `issued(b) ≥ 2^64` or `burned(b) > issued(b)` for any `b`.

The segment's local tree starts empty. A local statement is then admitted by [§8](#8-admission) against this state, and the segment's own events, roots and history extend it.

**Replay.** A verifier given the configuration, its artifacts, a segment's trail and the evidence for its openings recomputes every root, the spent set, `historyHash_n`, the directory and every `outstanding(b) = issued(b) − burned(b)` by running this section and [§8](#8-admission) itself, verifying every proof and every issuance signature in the segment and in its supplied ancestry. It accepts nothing it did not recompute. A replayed segment proves itself and the ancestry it was handed; whether each opening is the backing's canonical predecessor, whether its terms were in force, and which checkpoint is current are the record's to say (Construction §C2.7, C2.10.3–4).

## 11. The spent set

The spent set is a sparse Merkle tree of height `256` over `SHA256`, keyed by the nullifier's 32 big-endian bytes, so that both membership and non-membership are provable in the clear against `spentRoot`:

```text
leaf(nf)            = SHA256("moe/pool/v2/spent/leaf" ‖ nf[32])      where nf is in the set
e_0                 = 0[32]                                            the empty leaf
node(left, right)   = SHA256("moe/pool/v2/spent/node" ‖ left[32] ‖ right[32])
e_{h+1}             = node(e_h, e_h)                                   the empty subtree of height h+1
```

The root is the node of height `256`, and it is a function of the set alone, which is what lets an imported closure be inserted in any order ([§10](#10-import-and-replay)). **Key bits and the path.** At height `h`, counting `0` at the leaf, the node on a key's path is the right child of its parent when bit `255 − h` of the key is `1`, and the left child when it is `0`; so the root's two children are told apart by bit `0`, the most significant, and the leaf's parent reads bit `255`, the least. Inserting `nf` sets its leaf to `leaf(nf)` and recomputes the 256 nodes on its path.

**A proof** for a key is its 256 siblings, the sibling at height `h` being the other child of the path node's parent at height `h + 1`, sent as 32 bytes of **map** followed by the siblings not omitted. The map is a 256-bit value whose bit `h`, numbered as every bit in this document is (bit `0` the most significant), is `1` exactly when the sibling at height `h` equals `e_h`; those siblings are omitted and every other sibling is sent in ascending height. A proof whose map is clear for a sibling equal to `e_h` is malformed, so one set of siblings has one encoding; a map set at a height where the true sibling is not `e_h` carries the path to some other root, which a verifier refuses as it refuses any wrong sibling. **Non-membership** of `nf` at `spentRoot` is a proof whose path carries `e_0` at key `nf` to `spentRoot`; **membership** is a proof carrying `leaf(nf)` there. Construction §C2b.3's snapshot redemption establishes that a claim is unspent as of the snapshot; [the recovery contract](pool-recovery.md) has each reader of force read that from the spent set it replays (C2b.3.3), publishing no proof beside the release, so this encoding is defined and unused there. The venue-side record that redemption also needs is a later version's ([§7.4](#74-redemption-and-what-this-version-does-not-carry)). Nothing in this version's bytes, circuits, keys, bounds or accumulator changes.

## 12. The proof system

Circuits are written in Noir and compiled with `nargo`/`noir_wasm` `1.0.0-beta.26` to ACIR; proofs are Barretenberg `5.2.0` UltraHonk over BN254 with the verifier target `noir-recursive`, which is the zero-knowledge mode. The backend's legacy `keccak` mode disables zero knowledge and is not this construction. A proof is the backend's proof bytes: nonzero in length, a multiple of 32 bytes, and at most `131072` bytes; a verifier that accepts an empty, longer or unaligned proof accepts something this construction did not define.

**The circuits are the pinned sources, and the prose above is the relation they satisfy.** Two implementations that each wrote a circuit from the prose would derive two verification keys and refuse each other's configuration; so `moe/pool/v2` is instantiated only by the circuit sources the reference implementation publishes for it, at a named revision, whose SHA-256s and derived `bytecode(k)` and `vk(k)` values are recorded in [§15](#15-status). An implementation compiles those sources itself and refuses a configuration whose identities do not match what it derived. The backend's structured reference string is a trust assumption of this version: an implementation records the hashes of the parameter files it verified with, and a deployment states where they came from.

## 13. Bounds, and what is refused

| Item | Bound | On excess |
|---|---|---|
| a value | `< 2^64` | malformed |
| `issued(b)` after an issue, or after an import | `< 2^64` | statement or import refused before state changes |
| `burned(b)` after an import | `≤ issued(b)` | import refused |
| a field element | `< p` | malformed |
| a limb | `< 2^128` | malformed |
| local note-tree leaves | `2^32` | statement refused before state changes |
| scope entries, and segment header entries | `1 … 2^16`, strictly ascending by backing | malformed |
| a commitment sequence, and a segment's opening sequence | `≥ 1`; an opening's `0` means genesis with zero operator and root | malformed |
| statement inputs / outputs | `2` / `2` (burn: `2` / `1`) | fixed by the circuits; a different shape is a different construction |
| public inputs | issue `9`, spend `11`, burn `13` | malformed |
| proof bytes | `> 0`, `≤ 131072`, multiple of `32` | malformed |
| spent-set proof | 32-byte map plus the unomitted siblings, ascending height | malformed if a sent sibling equals its `e_h`, the length disagrees with the map, or bytes trail |

## 14. What this replaces, and what it costs

Under Construction §C0a this document names what it retires and what it charges.

**It replaces** `moe/pool/v1`. The field, the hash and its tags, the limb encoding, the 64-bit values with widened sums, the anchor-independent nullifier, the receiver-generated secret, the depth-32 tree with child-level tagging, the two-input padded shape with per-backing conservation, the spent-set accumulator, the history chain, the evidence-binding receipt and the derived randomness are carried over unchanged. What changed, and why:

- the configuration names no pool identity and no operator, so that its hash is an immutable domain a note can keep across operators (C1.2.1); **E** names the original operator beside it, as the genesis of the replacement chain;
- the note commits to the domain where it committed to the pool identity;
- every statement names its segment and scope root, and every hidden input and output backing is proven a member of the scope in-circuit, with the padding input included, so that a proof is authority for one segment and current authority is read from the public scope rather than from a fixed key (C1.2.2);
- a spend names one anchor per input, so a statement can spend notes created in different histories (C2.10.7);
- the note tree is per segment, and anchors form a forest certified by finalized imports rather than one history's own roots;
- the history's genesis names the segment, the snapshot names the segment, and the receipt names the segment and its scope root (C2.10.8);
- a segment's opening state is imported from exact commitments named in its header, with deduplication and conflict refusal, rather than continued from one key's own log (C2.10.5–7);
- every context string is this construction's, and none is a prefix of another.

**It costs.** Everything v1 cost — two hash functions, the 256-high spent set, padding nullifiers, the fixed 2×2 shape, the full leaf-list sync, an unpriced append — and, on top: a depth-16 scope path per input in a spend and one per issue or burn, about a third more in-circuit hashing than v1's spend; four more public inputs per statement; a public scope, which discloses which backings share an operator, and a public link per entry; two anchors per spend, which disclose the source histories of the inputs; a directory that carries the whole scope, so that a commitment finalizes the scope or nothing (C2.10.3); an import closure that a verifier holds in full, since every transitive ancestor's events must be present to deduplicate and to refuse conflicts, and a spent set rebuilt from that closure at every new segment; and a wallet that keeps the leaves of every segment it holds notes in. Each is stated so a reader prices it; none is hidden in an implementation.

## 15. Status

Independent adversarial review of the reference's circuits, frames, admission, receipts and import completed on 2026-09-06. Replay now owns supplied evidence before asynchronous verification, and an opening checkpoint cannot precede its segment's first commitment sequence (§6). Regression tests cover both. The constructor's computed-prefix API is trusted local state; external histories are verified by replay. This review is not a completed deployment audit.

**Pinned circuit sources**, in the reference's `src/pool/circuits/`, as SHA-256:

| Source | SHA-256 |
|---|---|
| `notes.nr` | `0c0a6cf6c12c6adc702cb8ef95e237ce3486dae9e65df2fc3dd724e7ca93b484` |
| `issue.nr` | `fdddefd8a94eb026259794de03faeb8854e53d274e62cb392407d5c88f9dd4cb` |
| `spend.nr` | `9ea3fe7f32d32502ea13c16df38ded5f4d688cac49c24a70abca31ff65f2dec1` |
| `burn.nr` | `1194077efafc1990d3fc90e866609ffc5cc1d3b6011252ae15d127d8e111ed51` |
| `vendor/poseidon2.nr` | `44f3a3d1abe7d5fa2da5c0339e52018195d55f295c320e530d355f9cc62159d8` |

**Derived circuit identities**, under §12's pinned toolchain and `noir-recursive` verifier target:

| Circuit | Bytecode SHA-256 | Verification-key SHA-256 |
|---|---|---|
| issue | `25e9d1af26fb6003d567587f7c71879bcabd9749e0279c870bb923264ec69d22` | `f9fb1624b85cf70350e4a9d15bfee1b2814e8cae5f75285ba6c0207a8f195ca2` |
| spend | `f00b1721a8a93d70c56758bb01662250f9ce8ee0ffcd9c6823aa41703b35ff93` | `8b46a4be1b3a307c22ff541319223c6fb7d9bb721a7a20b0a6822a291b38fcbc` |
| burn | `45effac560af94f59e89c3453f7c5261bc6ed64f878842be583b1a9424d50e8c` | `cca81c12041d305acea0c28a990253c03ef79083d2236feae24ec06be8df0af5` |

The resulting configuration hash is `651a78bf0db068bb26afcb602a74f48677243669235e2b541b5bebb1da876d6c`. The reference's `npm run check:pool` recompiles these sources, derives these keys, and drives admission, replacement imports and replay with real proofs. Its execution evidence is recorded in `docs/pool-v2-verification.json` in the reference repository. Cached parameter hashes are observations, not authenticated setup provenance.

Still required before any backing names `moe/pool/v2`:

| Item | Where it lands |
|---|---|
| Reviewed setup assumptions, authenticated parameter distribution and build provenance | the release record |
| The sequencing rules over these frames: scope derivation from the record, the commit schedule, whole-scope finality, lapse, descent, restart (C2.6–C2.8, C2.10.3–4, C2.10.9) | the reference's pool sequencer |
| Measured proving and verification on the target devices | the release record |
