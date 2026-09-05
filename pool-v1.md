# The shielded pool, construction v1

### The core construction's layouts, bit for bit

[Construction §C1.2](construction.md#c12-the-shielded-pool) says what the shielded pool is and what a statement proves. This document fixes one construction of it, **`moe/pool/v1`**, at the level two implementations must agree on: field encodings, hash functions and domain tags, the note and its commitment and nullifier, the note tree, the three statements and their public-input order, the spent-set accumulator and its non-membership proof, the ordered history, the per-backing snapshot digest, the receipt's contents, and the proof system. **E** names this construction and a configuration hash ([§C1.3](construction.md#c13-what-e-declares-for-the-construction)); everything here is fixed by that name. A change to anything below is `moe/pool/v2`, and a backing moves to it by successor ([§5.4](#54-redemption-and-what-this-version-does-not-carry) says what that costs under v1).

This supersedes the research profile `moe-private-payment-research/v1` in `reference-ts/experiments/private-payment`, from whose measured runs it is derived; [§11](#11-what-this-replaces-and-what-it-costs) lists what changed and why. It has had one independent adversarial review; [§12](#12-status) records what is still unpinned.

Notation. `‖` is byte concatenation. `frame(…)` is the framed byte string of [§1](#1-fields-hashes-and-encodings); `SHA256(…)` is SHA-256 over such a frame. `H(…)` is the in-circuit hash. `u8`, `u32`, `u64` are unsigned big-endian integers of that width. `F` is a field element written as 32 big-endian bytes. Bits of a 256-bit value are numbered from the most significant as bit `0`.

## 1. Fields, hashes and encodings

**The field.** `p = 21888242871839275222246405745257275088548364400416034343698204186575808495617`, the BN254 scalar field. A field element is canonical only in `[0, p)`; a value at or above `p` is malformed wherever it appears. Written as bytes, a field element is 32 big-endian bytes; written as text, `0x` followed by exactly 64 lowercase hexadecimal digits.

**The in-circuit hash `H`.** Poseidon2 over BN254 with state width 4 and rate 3, the permutation being the Noir standard library's `poseidon2_permutation`, in the variable-length sponge of `noir-lang/poseidon` v0.3.0 (`src/poseidon2.nr`, SHA-256 `44f3a3d1abe7d5fa2da5c0339e52018195d55f295c320e530d355f9cc62159d8`). Precisely: to hash `n` field elements, the state starts as `[0, 0, 0, n · 2^64]`; inputs are added into the first three state elements three at a time, each full block followed by one permutation; a final partial block of one or two inputs is added and permuted once more; when `n` is a positive multiple of three no partial block exists and no extra permutation runs; the output is the first state element. Every in-circuit object is hashed with a **domain tag** as its first input:

| Tag | Value | Used for |
|---|---|---|
| `T_OWNER` | `1001` | an owner from a spend secret |
| `T_NOTE` | `1002` | a note commitment |
| `T_NULLIFIER` | `1003` | a nullifier |
| `T_NODE` | `1004` | a note-tree node |

**Frames and `SHA256`.** A frame is a context string written raw as UTF-8 with no length prefix, followed by fields in the order the definition lists them, each fixed-width (`F`, a 32-byte identifier, `u8`, `u32`, `u64`) or `u32`-length-prefixed where variable. No context string in this document is a prefix of another. Two different inputs never produce one byte string. The reference implementation's `ByteWriter` is this convention, and `SHA256(…)` is SHA-256 over the frame.

**Identifiers as field elements.** A 32-byte identifier (a pool identity, a backing name) enters a circuit as two limbs: `hi` is the first 16 bytes and `lo` the last 16, each read big-endian as an integer below `2^128`. A circuit range-checks both limbs. Reducing a 32-byte value modulo `p` is not a representation of it.

**Quantities.** A quantity is a `u64`, positive where the rule says positive. Sums are computed after widening to 128 bits; equality in the field is never a substitute for integer equality. Running totals per backing stay below `2^64`, and admission refuses what would exceed that ([§6](#6-admission)).

## 2. What E declares, and what the configuration fixes

Beside the operator, venue, interval and silence clause, **E** carries two fields for the claim layer:

- `construction`: the ASCII string `moe/pool/v1`;
- `configuration`: 32 bytes, the **configuration hash** below.

Both are inside the backing's name. A backing whose **E** names an operator and a configuration is served in that operator's pool under that configuration and no other. A backing that declares identified issuance ([Construction §C1.1](construction.md#c11-issuance-is-logged-in-the-open-and-the-recipient-is-not)) cannot be served under v1, which carries no recipient field.

**The configuration** is a document whose bytes are exactly the preimage below, and whose hash is

```text
configHash = SHA256(
  "moe/pool/v1/config" ‖ pool[32] ‖ operator[32]
  ‖ bytecode(issue)[32] ‖ vk(issue)[32]
  ‖ bytecode(spend)[32] ‖ vk(spend)[32]
  ‖ bytecode(burn)[32]  ‖ vk(burn)[32]
  ‖ helper[32] ‖ u8 noteTreeDepth ‖ u8 inputs ‖ u8 outputs
)
```

where `pool` is the pool identity, `operator` the operator's Ed25519 public key, `bytecode(k)` the SHA-256 of the bytes that circuit `k`'s compiled artifact's `bytecode` field base64-decodes to, exactly as the compiler emits them, `vk(k)` the SHA-256 of the verification-key bytes the backend's key derivation returns for that bytecode, `helper` the SHA-256 of the Poseidon2 helper source, and the three bounds are `32`, `2`, `2` in this version. The configuration does not name the backings it serves: backings name it, so that a backing's name can contain it.

**The pool identity** is `SHA256("moe/pool/v1/pool" ‖ operator[32] ‖ u64 creationIndex)`, and an operator uses one pool identity per configuration. Every note commits to its pool ([§3](#3-notes)), and every issuance signature covers `configHash` ([§5.1](#51-issue)); what stops a statement crossing from one pool or configuration to another is [§6](#6-admission)'s anchor check, whose anchors come only from this pool's own history, and the issuance signature, which authorizes one configuration. The pool field in the note is a second binding, not the first.

Before accepting a note a wallet checks that the backing's **E** names `moe/pool/v1`, that its `operator` equals the configuration's `operator`, that its `configuration` equals the hash of the configuration the wallet holds, that the note's pool is the configuration's pool, and that the configuration's circuit and key identities are the ones the wallet has pinned. A verifier never accepts a verification key supplied with a statement.

## 3. Notes

A **note opening** is `(pool, backing, value, owner, rho)`: the pool identity and the backing name as limbs, a `u64` value, and two field elements. The **spend secret** `secret` is a nonzero field element the note's receiver derives and never reveals; `rho` is a nonzero field element the note's creator derives.

```text
owner = H(T_OWNER, secret)
cm    = H(T_NOTE, poolHi, poolLo, backingHi, backingLo, value, owner, rho)
nf    = H(T_NULLIFIER, poolHi, poolLo, cm, secret)
```

`owner`, `cm` and `nf` must be nonzero. The nullifier is a function of the immutable note and its owner's secret alone: neither the anchor nor the path a note is later proved under enters it, so one note has one nullifier however it is proved (Construction §C1.2).

**The receiver generates the secret** (invariant 25) and hands the payer only `owner`; the payer builds the output note and delivers the opening `(backing, value, rho)` to the receiver privately, with the statement's `statementHash`. The receiver recomputes `cm`, requires `value > 0` — a zero-value note can never be an input — and checks `cm` among the outputs of a statement in a history the receiver has itself verified up to a witnessed commitment, or a statement whose receipt the receiver accepts as the operator's liability ([§7](#7-the-ordered-history)); an operator's word alone is neither. A receiver derives a **fresh secret per expected payment**; an `owner` reused across payments lets the payers that received it link those payments.

**Randomness is derived, not drawn** (invariant 26): a wallet derives every `rho` it creates, and the `(rho, secret)` of every padding input ([§5.2](#52-spend)), deterministically from its own root secret and the statement's real input nullifiers (for an issuance, from the obligor's root secret and the issuance's `owner`), so that a statement rebuilt after a crash is the same statement and finds its receipt. The protocol cannot check this; a wallet that draws fresh randomness on retry is refused as a double spend and has lost nothing but its own receipt.

## 4. The note tree

The pool's note tree is a binary Merkle tree of depth `32` over `H`, append-only: leaf `i` is the commitment of the `i`-th output accepted in the pool, counting from zero across every statement in acceptance order, and an unused leaf is the field element `0`. A node is hashed with the level of its **children**, `0` for the two leaves under it and `31` for the two children of the root:

```text
node(l, left, right) = H(T_NODE, l, left, right)         l is the children's level
z_0 = 0,   z_{l+1} = node(l, z_l, z_l)                     the empty subtree with its top at level l+1
```

The root is the node at level `32`, so the empty tree's root is `z_32`. A **path** for a leaf is 32 siblings, one per level from `0` to `31`, with 32 direction bits, `true` where the node on the path is the right child at that level.

An **anchor** is the root after any accepted statement, or `z_32` before the first; the pool's anchor set is exactly those roots, in acceptance order, and a spend may prove membership against any of them. A wallet proves against the latest anchor it holds, since an old anchor bounds the input's position and dates the wallet's sync. The tree holds `2^32` leaves; a statement whose outputs would not fit is refused before it changes state, and there is no pruning or rollover under one pool identity.

**A wallet computes its own paths.** It syncs the pool's full leaf list and builds paths locally; it never asks a server for one leaf's path or index, and a receiver never fetches one `statementHash`, since either tells the server which note that wallet holds and links the receipt to the spend that follows. The sync is [§11](#11-what-this-replaces-and-what-it-costs)'s cost.

## 5. Statements

A **statement** is `(kind, publicInputs, proof)`, plus `obligorSignature` for an issuance. `kind` is `1` for issue, `2` for spend, `3` for burn. `publicInputs` is the list of field elements below, in the verifier's order and of exactly the stated length. `proof` is the proof bytes of [§9](#9-the-proof-system).

The **statement bytes** frame what a statement asserts, independently of its proof, and their hash names the statement:

```text
statementBytes = frame("moe/pool/v1/statement" ‖ configHash[32] ‖ u8 kind
                       ‖ u32 n ‖ publicInputs[0] … publicInputs[n−1])      each an F
statementHash  = SHA256 over statementBytes
```

`statementHash` is the statement's identity in admission, receipts, the history and a payer's delivery to a receiver. Two statements with one `statementHash` are one statement, whatever their proof bytes.

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
witness      = for each input i in {1, 2}: note_i (backing_i, value_i, owner_i, rho_i), secret_i,
               siblings_i[32], right_i[32];  for each output j in {1, 2}: note_j
```

The circuit proves, for each input `i`:

- `secret_i ≠ 0` and `owner_i = H(T_OWNER, secret_i)`;
- `nf_i = H(T_NULLIFIER, poolHi, poolLo, cm(note_i), secret_i)` and `nf_i ≠ 0`;
- **where `value_i > 0`**, that the path `(siblings_i, right_i)` carries `cm(note_i)` to `anchor`;
- **where `value_i = 0`**, nothing about the tree: the input is **padding**, its commitment need not exist anywhere, and its nullifier is whatever the formula gives for the wallet's derived `rho_i` and `secret_i`.

And across the statement:

- every output names the backing of one of the inputs: `backing_out_j ∈ {backing_1, backing_2}` for each `j`; a padding input names the backing of the other input, so a single-backing spend has one backing throughout and a two-backing spend has two real inputs;
- **quantities conserve per backing**, in 128-bit arithmetic: for each backing `b` among the inputs, the sum of input values naming `b` equals the sum of output values naming `b`; every value is below `2^64`, and `value_1 + value_2 > 0`;
- `nf_1 ≠ nf_2`; `cm_1 = H(T_NOTE, … note_out1)`, `cm_2 = H(T_NOTE, … note_out2)`, `cm_1 ≠ cm_2`, both nonzero; every output's `owner` and `rho` nonzero.

A spend reveals the anchor, two nullifiers and two commitments, and nothing else. An output of value `0` is allowed and occupies a leaf. A two-backing spend is how a payment from two backings, or a two-backing exchange inside one pool, is one statement (Construction §C1.2, §C1.7); a payment from more than two backings is several statements. A padding input's nullifier enters the spent set like any other.

### 5.3 Burn

```text
publicInputs = [poolHi, poolLo, backingHi, backingLo, quantity, anchor, nf_1, nf_2, cm_change]   n = 9
witness      = for each input i: note_i, secret_i, siblings_i[32], right_i[32];  note_change
```

The circuit proves, in full: for each input, `secret_i ≠ 0`, `owner_i = H(T_OWNER, secret_i)`, `nf_i = H(T_NULLIFIER, poolHi, poolLo, cm(note_i), secret_i)`, `nf_i ≠ 0`, and, where `value_i > 0`, the path to `anchor`; `nf_1 ≠ nf_2`; every note, input and change, names the public `backing`; `value_1 + value_2 = quantity + value_change` in 128-bit arithmetic with every value below `2^64` and `quantity > 0`; `cm_change = H(T_NOTE, … note_change)`, `cm_change ≠ 0`, `owner_change ≠ 0`, `rho_change ≠ 0`; all four limbs below `2^128`.

A burn is destruction of claims: it lowers `outstanding(backing)` by `quantity` and proves nothing about the external payout.

### 5.4 Redemption, and what this version does not carry

Redemption is a spend whose output the backer owns, the backer having generated that output's secret; it is not a statement kind, and it leaves `outstanding` unchanged (invariant 10).

**v1 carries issue, spend and burn, witnessing and the directory (Construction §C2.3–C2.4), replacement and takeover (§C2.5–C2.8), and the no-commitment grade (§C2b.6).** The following Construction rules have no v1 object and are inoperative for a backing issued under v1, to be defined by `moe/pool/v2` together:

- presentation (§C3): the demand naming claims by `H(nullifier)`, the spent-pending lock, acceptance, release, and invariant 27's settlement;
- the non-service object (§C2b.5) — so **E**'s non-service grade is inert for a v1 backing, and its two durations bind nothing;
- snapshot redemption's venue leg and the adoption of venue-witnessed nullifiers on return from silence (§C2b.3–C2b.4) — v1 defines the non-membership proof ([§8](#8-the-spent-set)) but no record that puts a nullifier into the spent set without a statement, so the no-commitment grade opens no redemption path under v1 and replacement under **E**'s rule is the only remedy against a dark operator;
- the atomic swap (§C1.7), and with it invariant 23's demand record and pending-lock set and §C1.5's aborted-presentation rule.

A backing crosses from v1 to v2 by successor. Under v1 that crossing is not atomic: a holder spends to the backer, the backer burns, and the backer issues under v2, each step a separate statement resting on the backer's honesty, because v1 has no swap that reads the holder's signature against the backer's. [§11](#11-what-this-replaces-and-what-it-costs) prices it.

## 6. Admission

The sequencer admits a statement against **one committed view**. First, if a statement with this `statementHash` has already been accepted, it returns that statement's receipt and changes nothing (invariant 26): an exact resubmission is the same statement whatever its proof bytes. Otherwise it changes state only if every check passes:

1. `publicInputs` has the length for `kind`, every element is a canonical field element, and `poolHi ‖ poolLo` is this pool.
2. The proof verifies against the configuration's verification key for `kind` and these public inputs, and against nothing else.
3. For an issue or burn: `backingHi ‖ backingLo` names a backing whose **E** names this operator and this configuration, and whose signed terms the operator holds; for an issue, `obligorSignature` verifies under that backing's **K** over `statementBytes`, and `issued(backing) + quantity < 2^64`; for a burn, `outstanding(backing) ≥ quantity`.
4. For a spend or burn: `anchor` is in the pool's anchor set; no nullifier of the statement is in the spent set, and the statement's nullifiers are distinct.
5. Every output commitment is absent from the note tree, the statement's outputs are distinct, and they fit under `2^32` leaves.

Then, as one transition: the outputs are appended in order, the nullifiers are inserted into the spent set, `issued(backing)` or `burned(backing)` moves, the statement is appended to the history, and the receipt is signed. A statement that fails any check leaves no trace.

## 7. The ordered history

The pool's public history is the sequence of accepted statements `s_1, s_2, …`, numbered from `1`. After statement `i` the pool has a note-tree root `noteRoot_i`, a spent-set root `spentRoot_i` ([§8](#8-the-spent-set)) and per-backing totals; before any statement, `noteRoot_0 = z_32` and `spentRoot_0 = e_256`. The history is bound by a hash chain:

```text
historyHash_0 = SHA256("moe/pool/v1/genesis" ‖ configHash[32])
historyHash_i = SHA256("moe/pool/v1/history" ‖ historyHash_{i−1}[32] ‖ statementHash_i[32]
                       ‖ noteRoot_i[32] ‖ spentRoot_i[32] ‖ u64 i)
```

`historyHash_i` binds the order of every statement, every nullifier and every accepted root up to `i`; two histories with equal note roots and different spent sets have different history hashes (invariant 23).

**The receipt** for statement `i` is the operator's signature, under the sequencing layer's receipt envelope, over: `i`; `statementHash_i`; `historyHash_i`; `SHA256` of the proof bytes it verified and, for an issuance, of the `obligorSignature` it verified, so that the operator's signature attests the exact evidence it admitted and a stranger served bad bytes can tell an operator's fault from a replica's corruption; and the sequence of the commitment the operator last signed (Construction §C2b.4).

**The snapshot digest** a commitment's directory carries for a backing `b` served in this pool is

```text
snapshot(b) = SHA256("moe/pool/v1/snapshot" ‖ b[32] ‖ historyHash_n[32] ‖ u64 issued(b) ‖ u64 burned(b))
```

at the pool's current length `n`. Every backing in one pool shares the pool's history hash; a stranger verifying one backing replays the pool's whole public history, since a spend does not say which backing it moved.

**The served trail** is the history with, for every statement, its proof and (for an issuance) its obligor signature, and the signed terms of every backing that any issue or burn in the history names; without a backing's terms the issuances naming it cannot be verified and the replay stops at the first such statement.

**Replay.** A verifier given the configuration, its artifacts and the served trail from genesis recomputes every root, the spent set, `historyHash_n` and every `outstanding(b) = issued(b) − burned(b)` by running [§6](#6-admission) itself, verifying every proof and every issuance signature. It accepts nothing it did not recompute. A replayed prefix proves that prefix; which history is current is the witnessed commitment's to say (Construction §C2).

## 8. The spent set

The spent set is a sparse Merkle tree of height `256` over `SHA256`, keyed by the nullifier's 32 big-endian bytes, so that both membership and non-membership are provable in the clear against `spentRoot`:

```text
leaf(nf)            = SHA256("moe/pool/v1/spent/leaf" ‖ nf[32])      where nf is in the set
e_0                 = 0[32]                                            the empty leaf
node(left, right)   = SHA256("moe/pool/v1/spent/node" ‖ left[32] ‖ right[32])
e_{h+1}             = node(e_h, e_h)                                   the empty subtree of height h+1
```

The root is the node of height `256`. **Key bits and the path.** At height `h`, counting `0` at the leaf, the node on a key's path is the right child of its parent when bit `255 − h` of the key is `1`, and the left child when it is `0`; so the root's two children are told apart by bit `0`, the most significant, and the leaf's parent reads bit `255`, the least. Inserting `nf` sets its leaf to `leaf(nf)` and recomputes the 256 nodes on its path.

**A proof** for a key is its 256 siblings, the sibling at height `h` being the other child of the path node's parent at height `h + 1`, sent as 32 bytes of **map** followed by the siblings not omitted. The map is a 256-bit big-endian integer whose bit `h` is `1` exactly when the sibling at height `h` equals `e_h`; those siblings are omitted and every other sibling is sent in ascending height. A proof whose map is clear for a sibling equal to `e_h`, or set for one that is not, is malformed. **Non-membership** of `nf` at `spentRoot` is a proof whose path carries `e_0` at key `nf` to `spentRoot`; **membership** is a proof carrying `leaf(nf)` there. Construction §C2b.3's snapshot redemption will read the holder's non-membership proof at the last witnessed commitment's `spentRoot`; the venue-side record it also needs is v2's ([§5.4](#54-redemption-and-what-this-version-does-not-carry)).

## 9. The proof system

Circuits are written in Noir and compiled with `nargo`/`noir_wasm` `1.0.0-beta.26` to ACIR; proofs are Barretenberg `5.2.0` UltraHonk over BN254 with the verifier target `noir-recursive`, which is the zero-knowledge mode. The backend's legacy `keccak` mode disables zero knowledge and is not this construction. A proof is the backend's proof bytes: nonzero in length, a multiple of 32 bytes, and at most `131072` bytes; a verifier that accepts an empty, longer or unaligned proof accepts something this construction did not define.

**The circuits are the pinned sources, and the prose above is the relation they satisfy.** Two implementations that each wrote a circuit from the prose would derive two verification keys and refuse each other's configuration; so `moe/pool/v1` is instantiated only by the circuit sources the reference implementation publishes for it, at a named revision, whose SHA-256s and derived `bytecode(k)` and `vk(k)` values are recorded in [§12](#12-status). An implementation compiles those sources itself and refuses a configuration whose identities do not match what it derived. The backend's structured reference string is a trust assumption of this version: an implementation records the hashes of the parameter files it verified with, and a deployment states where they came from.

The research profile's one-input, depth-8 circuits proved in 1.1–1.7 s at 14,656 bytes on the recorded desktop. v1's two-input, depth-32 circuits are unmeasured until they are compiled; a phone or browser budget is a deployment's to measure before it relies on this construction there.

## 10. Bounds, and what is refused

| Item | Bound | On excess |
|---|---|---|
| a value | `< 2^64` | malformed |
| `issued(b)` after an issue | `< 2^64` | statement refused before state changes |
| a field element | `< p` | malformed |
| a limb | `< 2^128` | malformed |
| note-tree leaves | `2^32` | statement refused before state changes |
| statement inputs / outputs | `2` / `2` (burn: `2` / `1`) | fixed by the circuits; a different shape is a different construction |
| public inputs | issue `6`, spend `7`, burn `9` | malformed |
| proof bytes | `> 0`, `≤ 131072`, multiple of `32` | malformed |
| spent-set proof | 32-byte map plus the unomitted siblings, ascending height | malformed if the map and siblings disagree or are non-canonical |

## 11. What this replaces, and what it costs

Under Construction §C0a this document names what it retires and what it charges.

**It replaces** the research profile `moe-private-payment-research/v1`. The note relation, the hash and its domain tags, the limb encoding, the 64-bit values with widened sums, the anchor-independent nullifier, the receiver-generated secret, the child-level tagging of tree nodes and the direction-bit convention are carried over unchanged, since the measured runs established them. What changed, and why:

- the configuration no longer contains the backings, so that a backing's **E** can name it; it names the operator, the helper and the bounds instead, and writes the pool as 32 bytes;
- spend and burn take two inputs with padding, so a wallet can consolidate; a spend may carry two backings with per-backing conservation, so a two-backing payment or exchange is one statement as Construction requires;
- either spend output may be zero, where the research forbade a zero payment output; the payment/change distinction was the wallet's, not the protocol's;
- the spent set is an accumulator with non-membership proofs, which snapshot redemption needs;
- the history binds the spent root beside the note root, and numbers statements from one;
- a statement's identity is its `statementHash`, and randomness is derived so a retry is the same statement; the research keyed idempotency on a caller-chosen command identifier;
- the receipt binds the proof and signature bytes the operator verified;
- the per-backing snapshot digest is defined so the directory can name it;
- the note tree is `32` deep rather than `8`, and the proof bound is `131072` bytes rather than the research's `131072` base64 characters;
- every context string is this construction's, and none is a prefix of another.

The research circuits are therefore superseded and are rewritten to these layouts when they are promoted.

**It costs.** Two hash functions, since proving needs an arithmetic-friendly hash and the clear needs a standard one. A 256-high spent-set tree, whose proofs are cheap to compute and, compressed, a few hundred bytes, at the price of 256 hashes per insertion. Padding nullifiers, which enlarge the spent set by one leaf per spend that needed no second input. A fixed 2×2 shape, so paying from `k` notes takes `k − 1` consolidating statements first, each a proof on the spender's device, and a payment from more than two backings is several statements. One pool history per operator, which a stranger replays in full to verify one backing, needing the signed terms of every backing the history names. A full leaf-list sync on every wallet, up to `2^32 × 32` bytes over a pool's life, because asking a server for one path tells it what you hold. An unpriced append: any holder adds two leaves per statement, so filling the tree is a deployment's exposure and admission pricing is the operator's. And a second version for the presentation and recovery objects, which every v1 backing crosses by a non-atomic redemption, burn and reissuance under the backer's honesty; a deployment that needs those objects waits for v2 rather than issuing under v1. Each is stated so a reader prices it; none is hidden in an implementation.

## 12. Status

Independently reviewed once (2026-09-05), findings applied. Still unpinned, and required before any backing names `moe/pool/v1`:

| Item | Where it lands |
|---|---|
| Circuit sources for issue, spend and burn at these layouts, at a named revision, with their SHA-256s | `reference-ts/src/pool/circuits/`, recorded here |
| `bytecode(k)` and `vk(k)` for each, as derived by the pinned toolchain | recorded here |
| The sequencing layer's receipt and commitment envelopes over these fields | the sequencing layouts, with §C2 built over notes |
| Measured proving and verification on the target devices | the release record |

Until those are recorded this construction cannot be instantiated, and this document is the relation its circuits will satisfy.
