# The shielded pool, construction v1

### The core construction's layouts, bit for bit

[Construction §C1.2](construction.md#c12-the-shielded-pool) says what the shielded pool is and what a statement proves. This document fixes one construction of it, **`moe/pool/v1`**, at the level two implementations must agree on: field encodings, hash functions and domain tags, the note and its commitment and nullifier, the note tree, the three statements and their public-input order, the spent-set accumulator and its non-membership proof, the ordered history, the per-backing snapshot digest, and the proof system. **E** names this construction and a configuration hash ([§C1.3](construction.md#c13-what-e-declares-for-the-construction)); everything here is fixed by that name. A change to anything below is `moe/pool/v2`, and a backing moves to it by successor and swap.

This supersedes the research profile `moe-private-payment-research/v1` in `reference-ts/experiments/private-payment`, from whose measured runs it is derived; [§11](#11-what-this-replaces-and-what-it-costs) lists what changed and why.

Notation. `‖` is byte concatenation. `SHA256(…)` is over the framed bytes described in [§1](#1-fields-hashes-and-encodings). `H(…)` is the in-circuit hash. `u8`, `u32`, `u64` are unsigned big-endian integers of that width. `F` is a field element written as 32 big-endian bytes.

## 1. Fields, hashes and encodings

**The field.** `p = 21888242871839275222246405745257275088548364400416034343698204186575808495617`, the BN254 scalar field. A field element is canonical only in `[0, p)`; a value at or above `p` is malformed wherever it appears. Written as bytes, a field element is 32 big-endian bytes; written as text, `0x` followed by exactly 64 lowercase hexadecimal digits.

**The in-circuit hash `H`.** Poseidon2 over BN254 with state width 4 and rate 3, as the Noir standard library's `poseidon2_permutation` implements the permutation, in the variable-length sponge of `noir-lang/poseidon` v0.3.0 (`src/poseidon2.nr`, SHA-256 `44f3a3d1abe7d5fa2da5c0339e52018195d55f295c320e530d355f9cc62159d8`): the message length is the sponge's initial value, and `H(x_1, …, x_n)` is that sponge's output over exactly `n` field elements. The configuration ([§2](#2-what-e-declares-and-what-the-configuration-fixes)) pins the helper's source hash. Every in-circuit object is hashed with a **domain tag** as its first input:

| Tag | Value | Used for |
|---|---|---|
| `T_OWNER` | `1001` | an owner from a spend secret |
| `T_NOTE` | `1002` | a note commitment |
| `T_NULLIFIER` | `1003` | a nullifier |
| `T_NODE` | `1004` | a note-tree node |

**The out-of-circuit hash `SHA256`.** SHA-256 over *framed bytes*: a context string written raw as UTF-8 with no length prefix, then fields in the order the definition lists them, each fixed-width (`F`, 32-byte identifiers, `u8`, `u32`, `u64`) or `u32`-length-prefixed where variable. Two different inputs never produce one byte string. The reference implementation's `ByteWriter` is this convention.

**Identifiers as field elements.** A 32-byte identifier (a pool identity, a backing name) enters a circuit as two limbs: `hi` is the first 16 bytes and `lo` the last 16, each read big-endian as an integer below `2^128`. A circuit range-checks both limbs. Reducing a 32-byte value modulo `p` is not a representation of it.

**Quantities.** A quantity is a `u64`, positive where the rule says positive. Sums are computed after widening to 128 bits; equality in the field is never a substitute for integer equality.

## 2. What E declares, and what the configuration fixes

Beside the operator, venue, interval and silence clause, **E** carries two fields for the claim layer:

- `construction`: the ASCII string `moe/pool/v1`;
- `configuration`: 32 bytes, the **configuration hash** below.

Both are inside the backing's name. A backing whose **E** names an operator and a configuration is served in that operator's pool under that configuration and no other.

**The configuration** is a published document with these fields, and its hash is

```text
configHash = SHA256(
  "moe/pool/v1/config" ‖ pool[32] ‖ operator[32]
  ‖ bytecode(issue)[32] ‖ vk(issue)[32]
  ‖ bytecode(spend)[32] ‖ vk(spend)[32]
  ‖ bytecode(burn)[32]  ‖ vk(burn)[32]
  ‖ helper[32] ‖ u8 noteTreeDepth ‖ u8 inputs ‖ u8 outputs
)
```

where `pool` is the pool identity, `operator` the operator's Ed25519 public key, `bytecode(k)` the SHA-256 of circuit `k`'s compiled ACIR bytecode, `vk(k)` the SHA-256 of its verification key bytes, `helper` the SHA-256 of the Poseidon2 helper source, and the three bounds are `32`, `2`, `2` in this version. The configuration does not name the backings it serves: backings name it, so that a backing's name can contain it.

**The pool identity** is 32 bytes chosen by the operator, unique across every pool the operator has ever run; the recommended derivation is `SHA256("moe/pool/v1/pool" ‖ operator[32] ‖ u64 creationIndex)`. Two pools never share an identity, and every note commits to its pool ([§3](#3-notes)), so a note cannot be replayed into another pool.

A wallet verifies, before accepting a note, that the note's pool is the one named by the configuration that the backing's **E** names, and that the configuration's circuit and key identities are the ones it has pinned. A verifier never accepts a verification key supplied with a statement.

## 3. Notes

A **note opening** is `(pool, backing, value, owner, rho)`: the pool identity and the backing name as limbs, a `u64` value, and two field elements. The **spend secret** `secret` is a nonzero field element the note's receiver draws at random and never reveals; `rho` is a nonzero field element the note's creator draws at random.

```text
owner = H(T_OWNER, secret)
cm    = H(T_NOTE, poolHi, poolLo, backingHi, backingLo, value, owner, rho)
nf    = H(T_NULLIFIER, poolHi, poolLo, cm, secret)
```

`owner`, `cm` and `nf` must be nonzero. The nullifier is a function of the immutable note and its owner's secret alone: neither the anchor nor the path a note is later proved under enters it, so one note has one nullifier however it is proved (Construction §C1.2).

**The receiver generates the secret** (invariant 25) and hands the payer only `owner`; the payer builds the output note and delivers the opening `(backing, value, rho)` to the receiver privately, with the statement's identity. The receiver recomputes `cm` and checks it among the outputs of an accepted statement before treating the note as received. A receiver draws a **fresh secret per expected payment**; an `owner` reused across payments lets the payers that received it link those payments.

## 4. The note tree

The pool's note tree is a binary Merkle tree of depth `32` over `H`, append-only: leaf `i` is the commitment of the `i`-th output accepted in the pool, counting from zero across every statement in acceptance order, and an unused leaf is the field element `0`. Nodes are hashed with their level, counting from `0` at the leaves:

```text
node(level, left, right) = H(T_NODE, level, left, right)
z_0 = 0,   z_{l+1} = node(l, z_l, z_l)          the empty subtree at level l+1
```

The root is the node at level `32`. A **path** for a leaf is 32 siblings, one per level from `0` to `31`, with 32 direction bits, `true` where the node on the path is the right child at that level.

An **anchor** is the root after any accepted statement, or the root of the empty tree; the pool's anchor set is exactly those roots, in acceptance order, and a spend may prove membership against any of them. The tree holds `2^32` leaves; a statement whose outputs would not fit is refused before it changes state, and there is no pruning or rollover under one pool identity.

## 5. Statements

A **statement** is `(kind, publicInputs, proof)`, plus `obligorSignature` for an issuance. `kind` is `1` for issue, `2` for spend, `3` for burn. `publicInputs` is the list of field elements below, in the verifier's order and of exactly the stated length. `proof` is the proof bytes of [§9](#9-the-proof-system).

The **statement bytes** frame what a statement asserts, independently of its proof:

```text
statementBytes = SHA256-framed(
  "moe/pool/v1/statement" ‖ configHash[32] ‖ u8 kind
  ‖ u32 n ‖ publicInputs[0] … publicInputs[n−1]      each an F
)
statementHash  = SHA256(statementBytes)
```

`statementHash` names the statement in receipts, in the history ([§7](#7-the-ordered-history)) and in a payer's delivery to a receiver.

### 5.1 Issue

```text
publicInputs = [poolHi, poolLo, backingHi, backingLo, quantity, cm]        n = 6
witness      = owner, rho
```

The circuit proves `quantity > 0`, `owner ≠ 0`, `rho ≠ 0`, `cm ≠ 0`, and `cm = H(T_NOTE, poolHi, poolLo, backingHi, backingLo, quantity, owner, rho)`, with all four limbs below `2^128` and `quantity` below `2^64`.

`obligorSignature` is the backing's obligor **K** signing `statementBytes` with the strict Ed25519 signature the reference uses for every other signed object. A signature over the backing's terms, or over anything but these exact bytes, authorizes nothing.

### 5.2 Spend

```text
publicInputs = [poolHi, poolLo, anchor, nf_1, nf_2, cm_1, cm_2]              n = 7
witness      = for each input i in {1, 2}: note_i (backing, value, owner, rho), secret_i,
               siblings_i[32], right_i[32];  for each output j in {1, 2}: note_j
```

The circuit proves, for each input `i`:

- `owner_i = H(T_OWNER, secret_i)` and `secret_i ≠ 0`;
- `nf_i = H(T_NULLIFIER, poolHi, poolLo, cm(note_i), secret_i)` and `nf_i ≠ 0`;
- **where `value_i > 0`**, that the path `(siblings_i, right_i)` carries `cm(note_i)` to `anchor`;
- **where `value_i = 0`**, nothing about the tree: the input is **padding**, its commitment need not exist anywhere, and its nullifier is whatever the formula gives for the wallet's random `rho` and `secret`.

And across the statement:

- all four notes name one backing, `backing_1 = backing_2 = backing_out1 = backing_out2`;
- `value_1 + value_2 = value_out1 + value_out2` in 128-bit arithmetic, every value below `2^64`, and `value_1 + value_2 > 0`;
- `nf_1 ≠ nf_2`; `cm_1 = H(T_NOTE, … note_out1)`, `cm_2 = H(T_NOTE, … note_out2)`, `cm_1 ≠ cm_2`, both nonzero; every output's `owner` and `rho` nonzero.

A spend reveals the anchor, two nullifiers and two commitments, and nothing else. An output of value `0` is allowed and occupies a leaf. A padding input's nullifier enters the spent set like any other; a wallet draws fresh randomness for it, since a repeated padding nullifier is refused as spent.

### 5.3 Burn

```text
publicInputs = [poolHi, poolLo, backingHi, backingLo, quantity, anchor, nf_1, nf_2, cm_change]   n = 9
witness      = as for spend, with one output note_change
```

The circuit proves everything §5.2 proves of the two inputs, that every note names the public `backing`, that `value_1 + value_2 = quantity + value_change` in 128-bit arithmetic with `quantity > 0`, and `cm_change = H(T_NOTE, … note_change)` with `owner_change` and `rho_change` nonzero.

A burn is destruction of claims: it lowers `outstanding(backing)` by `quantity` and proves nothing about the external payout.

### 5.4 Redemption, and what this version does not carry

Redemption is a spend whose output the backer owns, the backer having generated that output's secret; it is not a statement kind, and it leaves `outstanding` unchanged (invariant 10). The presentation objects of Construction §C3 over notes — the demand naming claims by `H(nullifier)`, the spent-pending lock, acceptance and release — are not in this version. They are `moe/pool/v2`'s to define with the sequencing rules built over notes, and a backing issued under v1 moves to v2 by successor.

## 6. Admission

The sequencer admits a statement against **one committed view** and changes state only if every check passes:

1. `publicInputs` has the length for `kind`, every element is a canonical field element, and `poolHi ‖ poolLo` is this pool.
2. The proof verifies against the configuration's verification key for `kind` and these public inputs, and against nothing else.
3. For an issue or burn: `backingHi ‖ backingLo` names a backing whose **E** names this operator and this configuration, and whose signed terms the operator holds; for an issue, `obligorSignature` verifies under that backing's **K** over `statementBytes`; for a burn, `outstanding(backing) ≥ quantity`.
4. For a spend or burn: `anchor` is in the pool's anchor set; no nullifier of the statement is in the spent set, and a statement's nullifiers are distinct.
5. Every output commitment is absent from the note tree, the statement's outputs are distinct, and they fit under `2^32` leaves.

Then, as one transition: the outputs are appended in order, the nullifiers are inserted into the spent set, `issued(backing)` or `burned(backing)` moves, the statement is appended to the history, and the receipt is signed. A statement that fails any check leaves no trace. An exact resubmission of an accepted statement returns the same receipt; a different statement under the same identity is refused (invariant 26).

## 7. The ordered history

The pool's public history is the sequence of accepted statements `s_1, s_2, …`. After statement `i` the pool has a note-tree root `noteRoot_i`, a spent-set root `spentRoot_i` ([§6](#6-admission), [§8](#8-the-spent-set)), and per-backing totals. The history is bound by a hash chain:

```text
historyHash_0 = SHA256("moe/pool/v1/history/genesis" ‖ configHash[32])
historyHash_i = SHA256("moe/pool/v1/history" ‖ historyHash_{i−1}[32] ‖ statementHash_i[32]
                       ‖ noteRoot_i[32] ‖ spentRoot_i[32] ‖ u64 i)
```

`historyHash_i` binds the order of every statement, every nullifier and every accepted root up to `i`; two histories with equal note roots and different spent sets have different history hashes (invariant 23). A receipt for statement `i` names `i`, `statementHash_i` and `historyHash_i`, under the sequencing layer's receipt envelope.

**The snapshot digest** a commitment's directory carries for a backing `b` served in this pool is

```text
snapshot(b) = SHA256("moe/pool/v1/snapshot" ‖ b[32] ‖ historyHash_n[32] ‖ u64 issued(b) ‖ u64 burned(b))
```

at the pool's current length `n`. Every backing in one pool shares the pool's history hash; a stranger verifying one backing replays the pool's whole public history, since a spend does not say which backing it moved. That is the pool's price for one anonymity set per operator, and it is what the directory's `(name, digest)` pair names.

**Replay.** A verifier given the configuration, its artifacts and the history from genesis recomputes every root, the spent set, `historyHash_n` and every `outstanding(b) = issued(b) − burned(b)` by running [§6](#6-admission) itself, verifying every proof and every issuance signature. It accepts nothing it did not recompute. A replayed prefix proves that prefix; which history is current is the witnessed commitment's to say (Construction §C2).

## 8. The spent set

The spent set is a sparse Merkle tree of depth `256` over `SHA256`, keyed by the nullifier's 32 big-endian bytes, so that both membership and non-membership are provable in the clear against `spentRoot`:

```text
leaf(nf)            = SHA256("moe/pool/v1/spent/leaf" ‖ nf[32])      where nf is in the set
e_0                 = 0[32]                                            the empty leaf
node(left, right)   = SHA256("moe/pool/v1/spent/node" ‖ left[32] ‖ right[32])
e_{l+1}             = node(e_l, e_l)                                   the empty subtree at height l+1
```

The root is the node at height `256`. From the root, the `i`-th most significant bit of the key selects the child at depth `i`, `0` for left. Inserting `nf` sets its leaf to `leaf(nf)` and recomputes the 256 nodes on its path.

A **proof** for a key is its 256 siblings from the leaf upward, transmitted as a 256-bit map marking which siblings are the empty subtree hash at their height, followed by only the siblings that are not. **Non-membership** of `nf` at `spentRoot` is a proof whose path carries `e_0` at key `nf` to `spentRoot`; **membership** is a proof carrying `leaf(nf)` there. Construction §C2b.3's snapshot redemption is the holder's non-membership proof for its note's nullifier at the last witnessed commitment's `spentRoot`.

## 9. The proof system

Circuits are written in Noir and compiled with `nargo`/`noir_wasm` `1.0.0-beta.26` to ACIR; proofs are Barretenberg `5.2.0` UltraHonk over BN254 with the verifier target `noir-recursive`, which is the zero-knowledge mode. The backend's legacy `keccak` mode disables zero knowledge and is not this construction. A proof is the backend's proof bytes, a multiple of 32 bytes and at most `131072` bytes; a verifier that accepts a longer or unaligned proof accepts something this construction did not define.

The configuration pins, per circuit, the SHA-256 of the compiled bytecode and of the verification key the backend derives from it; an implementation compiles the circuits itself and refuses a configuration whose identities do not match its own. The backend's structured reference string is a trust assumption of this version: an implementation records the hashes of the parameter files it verified with, and a deployment states where they came from. Proving a spend on the recorded desktop took 1.1–1.7 s at 14,656-byte proofs; a phone or browser budget is a deployment's to measure before it relies on this construction there.

## 10. Bounds, and what is refused

| Item | Bound | On excess |
|---|---|---|
| a value | `< 2^64` | malformed |
| a field element | `< p` | malformed |
| a limb | `< 2^128` | malformed |
| note-tree leaves | `2^32` | statement refused before state changes |
| statement inputs / outputs | `2` / `2` (burn: `2` / `1`) | fixed by the circuits; a different shape is a different construction |
| public inputs | issue `6`, spend `7`, burn `9` | malformed |
| proof bytes | `≤ 131072`, multiple of `32` | malformed |
| spent-set proof | `256` siblings, compressed | malformed if the map and siblings disagree |

## 11. What this replaces, and what it costs

Under Construction §C0a this document names what it retires and what it charges.

**It replaces** the research profile `moe-private-payment-research/v1`. The relation, the hash, the domain tags, the limb encoding, the 64-bit values with widened sums, the anchor-independent nullifier and the receiver-generated secret are carried over unchanged, since the measured runs established them. What changed: the configuration no longer contains the backings, so that a backing's **E** can name it; spend and burn take two inputs with padding, so a wallet can consolidate; the spent set is an accumulator with non-membership proofs, which snapshot redemption needs; the history binds the spent root beside the note root; the per-backing snapshot digest is defined so the directory can name it; the note tree is `32` deep rather than `8`; and every context string is this construction's. The research circuits are therefore superseded, and are rewritten to these layouts when they are promoted.

**It costs.** Two hash functions, since proving needs an arithmetic-friendly hash and the clear needs a standard one. A 256-deep spent-set tree, whose proofs are cheap to compute and, compressed, a few hundred bytes, at the price of 256 hashes per insertion. Padding nullifiers, which enlarge the spent set by one leaf per spend that needed no second input. A fixed 2×2 shape, so paying from `k` notes takes `k − 1` consolidating statements first, each a proof on the spender's device. One pool history per operator, which a stranger replays in full to verify one backing. And a second version for the presentation objects, which every v1 backing crosses by successor. Each is stated so a reader prices it; none is hidden in an implementation.
