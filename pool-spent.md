# Pool spent-set root

Normative layout contract for the successor to `moe/pool/v2`, implementing
Construction C1.2, invariant 23 and the recovery contract's C2b.3.3.
`pool-v3.md` must adopt this contract with the final statements and
configuration before runtime use. No v2 bytes, roots, circuits, keys or
configuration are reinterpreted.

## 1. The root is determined by the set

**C1.2.8 The successor spent root commits to a canonical compressed binary
tree of the nullifiers.** A key is exactly 32 bytes, read most significant
bit first; bit positions are absolute integers from 0 through 255. For a
finite set S of distinct keys define R(S) recursively:

- If S is empty, R(S) = SHA256(UTF8("moe/pool/v3/spent/empty")).
- If S contains only k, R(S) = SHA256(UTF8("moe/pool/v3/spent/leaf") || k).
- Otherwise, let b be the first bit position at which keys in S differ.
  Partition S into nonempty S0 and S1 according to bit b. Then
  R(S) = SHA256(UTF8("moe/pool/v3/spent/node") || u16be(b) || R(S0) || R(S1)).

Here `||` is byte concatenation, each child root is 32 bytes and `u16be`
is exactly two unsigned big-endian bytes. The fixed UTF-8 domains have no
length prefix or terminator. No unary node, empty child, insertion position,
balancing choice or uncommitted key prefix participates in the tree.
Every leaf binds its full key, including skipped prefix bits. Every branch
binds its absolute split position and the ordered roots of both nonempty
children. Split positions strictly increase down a path. This definition
selects one tree for a set; it does not accept an arbitrary supplied topology
that merely hashes to a claimed root.

The root covers the same nullifiers as pool-v2 §§9–11, including zero-input
padding nullifiers and the validated transitive import closure. Imports
deduplicate the same ancestral event, not conflicting spends of the same
nullifier. Distinct events spending the same nullifier remain invalid.
Closure validation precedes use of its set; treating input as a mathematical
set cannot silently erase conflicting history. The tree accepts every
32-byte key, including zero and all-one keys; statement relations retain
their own nonzero and field-range restrictions.

**C1.2.9 Every statement commits the spent root of its replayed prefix.**
Import order and insertion order cannot change R(S). The opening's spent
root is R of its validated imported set, or R(empty) when there are no
imported nullifiers. Replay validates a statement and its nullifiers before
inserting all of them, and binds the resulting root in that statement's
history hash as pool-v2 §9 does. Invalid statements cannot partly update
the accepted set. An exact statement retry reads the previous result before
attempting new insertions; it is not another spend. Statements that add no
nullifier retain the previous spent root. No deletion, spent-set reset or
privileged insertion is introduced here.

## 2. Verification and cost

Readers of force still establish absence from complete validated replay
under [pool recovery](pool-recovery.md) C2b.3.3. This contract introduces no
published membership/non-membership proof or proof parser. A root by itself
does not establish evidence availability, canonical finality or absence.
Missing evidence is unresolved, never an empty set. The spent root remains
outside the private note relations; this change gives no new party secret
keys or authority and exposes no data beyond already-public nullifiers.

The tree retains invariant 23's non-membership capability without replay.
Against a root already established as a canonical set commitment, a path
witness gives the full terminal key and, for each traversed branch, its
absolute split position and sibling hash. Positions must be in 0..255 and
strictly increasing from root to leaf, with at most 256 branches. The query's
bit selects the traversed child at every branch; the terminal key must have
those same bits. Starting with the terminal key's leaf hash, fold the path
backward with the specified branch frames and require the given root.
A terminal key equal to the query proves membership; an unequal key proves
absence. For the empty root the witness is empty and proves absence only.
This is a capability argument, not a new accepted wire object: no path
encoding, publication or runtime verifier is selected here. A later use must
pin those bytes and validation, and cannot treat an unvalidated topology as
a canonical set. Current recovery still requires complete replay.

A cached implementation stores N leaves and N−1 branches for N > 0, with
one cached hash per node. An insertion hashes one leaf, one new branch when
nonempty, and the changed ancestors; it never hashes skipped empty levels
or the whole set at each statement. There are at most 256 branch levels on
any path, even for adversarial keys. Approximately logarithmic paths for
uniform keys are a performance expectation, not an admission assumption.
Lookup follows branch bits and compares the full terminal key. Roots are
constant-size reads once updated. The reference records reproducible replay
measurements; memory in bytes and device budgets remain implementation costs.

For this successor, this replaces pool-v2 §11's 256-high sparse-tree root
and its unused published-proof encoding, retaining its set and per-statement
commitment meaning. The v2 rule remains fixed for v2. The full sorted-set
hash was rejected because it hashes O(N) keys at every statement; an indexed
tree assigned by insertion order was rejected because the same imported set
can have different roots. Committing compressed branch positions removes
empty-level hashing while retaining SHA-256, fixed-width frames and a single
set-determined commitment mechanism (Construction C0a).
