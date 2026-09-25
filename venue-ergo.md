# Ergo venue profile

Normative venue profile for [pool-v3 §13](pool-v3.md#13-record-range-evidence)
record-range answers read from the Ergo mainnet. Section 13 fixes the request,
the answer and the reader's rules, and leaves to a venue profile which venue
evidence establishes an answer and how. Construction C2.3.2 and C2.3.5 name a
venue with its finality rule and lag, and §13.1 adds the attribution rule to
that name. This profile fixes all three for Ergo, the verifier's evidence and
the header rules by which a reader authenticates that evidence itself.

This profile is selected for pool-v3 §13: it is the rule an Ergo venue
identity names, and each reader still selects its own verifier implementing
it (§12.1). It makes neither pool-v3 adoptable nor any backing's declaration
valid before pool-v3 §1's remaining items are approved. The v2 runtime's
Ergo venue keeps its own identity and index convention, and nothing here
reinterprets it.

The **reference node** is the Ergo node release v6.0.6 (source tag `v6.0.6`
of `ergoplatform/ergo`; release `ergo-6.0.6.jar`, SHA-256
`21b9023933b19b98b7eb4d50cb78bcb6c827a0fe65711a00ceaf1b83f8f3a323`) under
its mainnet chain settings: epochs of 128 blocks, eight epochs read by the
difficulty rule, a 120 s block interval, its initial difficulty and its
Autolykos parameters. Where a rule below defers to the reference node, it
means that release's behavior on the mainnet. Integers are unsigned
big-endian unless marked VLQ; `||` is byte concatenation; `lp(x)` is a u32
length followed by `x`.

## 1. Identity

```text
identity = SHA256(lp("moe/venue/ergo/v3") || anchor[32] || u64 depth ||
                  lp(script_1) || lp(script_2) || lp(script_3) || lp(script_4))
```

The context is literal ASCII. The deployment chooses the parameters:

- `anchor` is the id of one Ergo mainnet header at height 844,672 or above
  (§3), the last block before the venue's index space. A header id commits to
  its whole ancestry through parent ids, so the anchor names the chain up to
  itself. The all-zero id names no header and is refused. A deployment
  anchors at a block before its first record that is already final under
  its depth. An anchor the chain later orphans is the venue's failure: a
  reader sees only the anchor's descendants, so it cannot tell a branch that
  a minority extends from that anchor from the chain.
- `depth` is the finality depth, below 2^64 − 1 (§2).
- `script_k` is the exact ErgoTree that is the **location** of record kind
  `k`, in §13.1's kinds: 1 commitment, 2 replacement, 3 revocation,
  4 publication. The four are distinct, and each is exactly one tree as §5
  reads it. A deployment writes each as the reference node writes that tree:
  the node rewrites a tree it parses, so a location written otherwise carries
  nothing. Whoever can spend a location's boxes collects their value; the
  record stays in the blocks that carried it.

A different anchor, depth or location is a different venue. The identity is
hashed over the same parameter bytes that attribution compares.

## 2. Index, finality and lag

The witnessed index of an object is derived from the height of the block
whose transaction created it, never from a box's own creation height. With
the anchor at height `A`, index `i` is the block at height `A + 1 + i`, so
the anchor's child is index 0. Index `t` is witnessed once the best chain's
tip (§3) is at height `A + 1 + t + depth` or above; the venue's witnessed
index is `tip − depth − A − 1`, and nothing is witnessed before the tip
reaches `A + 1 + depth`. The lag (C2.3.5) is `depth + 1`: a transaction
signed at clock `c` is included at index `c + depth + 1` at the earliest.

Under C3.3's window a publication authorized at tip `T`, with its instant at
the latest witnessed index, has force when included at height `T + k` for
`1 <= k <= depth + 2`; proving and propagation spend the same margin. A
reorganization deeper than the depth is the venue's failure (§13.2), not a
fact an answer survives. Choosing the depth trades that margin and the
reorganization bound against a wait of one block interval (120 s target) per
unit of depth for every witnessed act. The profile sets no floor: a holder
judges a backing by the depth it declares. Over one measured mainnet day, 14
of 784 heights were reorganized, each a venue failure at depth 0, and the
share of included transactions inside the window was 42% at depth 0, 76%
at 2, 94% at 6 and 99.8% at 10 (reference
[latency probe](https://github.com/mediumofexchange/reference-ts/blob/main/docs/POOL_DEPLOYMENT_PROBES.md#inclusion-latency-on-the-mainnet)).

## 3. The header chain

The venue's chain is the Ergo mainnet's best chain above the anchor. A reader
establishes it from header bytes it obtains from suppliers it selects,
accepting only headers it verifies under this section; a supplier, including
a node the reader runs, is untrusted: a node is one supplier, and its headers
pass through this section like any other's.

**Context.** The reader starts from the anchor's context: the anchor and at
least the 1,024 headers below it, ascending, each read as below and each the
parent of the next, which is one height above it, authenticated by linkage alone
(each id the Blake2b-256 of the bytes read, the last equal to `anchor`). The
difficulty rule reads eight epochs back. The anchor is at height 844,672 or
above, so every header above it follows the EIP-37 difficulty rule; an
anchor below that height gives no chain.

**Header bytes.** The reference node checks a block's version against the
voted parameters only at the first block of a voting epoch, so any miner can
write any version byte on a block inside an epoch and the network keeps it.
A header of every version is therefore read, in the reference node's
canonical serialization for that version: version (one byte), parent id 32,
AD-proofs root 32, transactions root 32, state root 33, timestamp (VLQ, at
most 2^63 − 1), extension root 32, `nBits` (4 bytes), height (VLQ, at most
2^31 − 1), votes 3; then, where the version read as a signed byte is 2 to
127, the length of new fields (one byte: zero through version 4, and above it
followed by that many bytes, which the node keeps unread); then the
solution, ending exactly. The solution of version 1 is the miner key 33, a
one-time key `w` 33, the nonce 8 and `d` as a length byte followed by its
minimal unsigned big-endian bytes (one zero byte for zero); every other
version's is the miner key 33 and the nonce 8. VLQs are minimal. The miner
key and `w` are each a compressed secp256k1 point that decodes, or 33 zero
bytes for the identity. The header id is the Blake2b-256 of these bytes;
any other spelling is refused. A supplier copying the node's own statement
of a header writes this spelling.

**Acceptance.** A header is accepted where its parent is the anchor or an
accepted header, its height is the parent's plus one, its timestamp exceeds
the parent's, and:

1. its difficulty, `nBits` decoded as the node decodes a compact integer, equals
   the required difficulty: the parent's inside an epoch of 128 headers, and
   after a parent whose height is a multiple of 128 the EIP-37 value the
   reference node computes over the parent's own ancestors at heights
   `parent − 128·i` for `i` from 8 down to 0, the parent included; the
   required difficulty is positive and at most `q`, the
   secp256k1 group order;
2. its work holds at that difficulty, as the reference node checks it over
   the bytes before the solution: for version 1 the Autolykos v1 equation
   `w^f = g^d · pk`, with `d` below `q` divided by the difficulty and neither
   the miner key nor `w` the identity; for every other version the Autolykos
   v2 hit, over those bytes, the nonce and the height, below `q` divided by
   the difficulty.

**Best chain.** A chain's score is the sum of the required difficulties of
its headers above the anchor. The best chain is the accepted chain of
greatest score; of equal scores the one the reader accepted first keeps
precedence, so a tie is reader-local until a later header breaks it. Its
headers from the anchor's child to its tip are the verifier's header input.

**Not applied.** The node's local-clock bound on timestamps, its local
fork-depth bound, its configured checkpoint and its marking of headers whose
block failed full validation are not header rules here: the first three are
node settings, and full validity is invisible to a header reader. The chain
therefore rests on work, as a light client's does: the reader trusts that
the heaviest chain it is shown is the network's and that its blocks are
valid in full. A supplier can withhold a heavier chain but cannot
make the reader accept a header without the work; several independently
selected suppliers reduce, without removing, withholding. Without the clock
bound, a supplier can lower the required difficulty on a side branch by
stating future timestamps; such a branch cannot outscore the work of the best
chain, but each header it adds costs the reader a work check. Which suppliers
a reader consults, and the budget of headers it accepts from each, are local
policy like §13.1's budgets. A budget that runs out before a heavier chain
arrives leaves the reader where withholding would, which is the light-client
limit above, so a reader bounds what a supplier may add without refusing
the heaviest chain it can see.

## 4. Block sections

A block's **transaction section** is its transactions in block order, each
as its unsigned bytes and its witness id. The unsigned bytes are the
reference node's serialization of the transaction with every input's proof
empty; their Blake2b-256 is the transaction id. The witness id is 31 bytes:
the Blake2b-256 of the transaction's input proofs concatenated, first byte
dropped.

Index `i`'s section is supplied by a nonempty section that reproduces the
transactions root of the best chain's header at height `A + 1 + i`, whatever
that header's version: a gate on the version would let any miner deny every
range through its block. The reference node takes the root's rule from the
section's own serialization, which nothing compares with the header's
version, so the miner chooses it; a section is therefore accepted where it
reproduces the root under either rule, the ids alone or the ids followed by
the witness ids:

```text
leaves = id_1 .. id_n                                   (ids rule)
leaves = id_1 .. id_n || witnessId_1 .. witnessId_n     (witness rule)
leaf   = Blake2b-256(0x00 || leaf bytes)
node   = Blake2b-256(0x01 || left || right)
```

Leaves are paired left to right level by level; a node without a right
sibling hashes its left child alone, and a single leaf still has a node
above it. Without a Blake2b collision no two sections reproduce one root
under the two rules: leaves and nodes hash under different prefixes, and only
the witness rule has 31-byte leaves. The root binds every unsigned byte, and
under the witness rule every witness id. It binds neither how proofs divide
among inputs nor, under the ids rule, any witness id, and no rule here reads
either. A section that does not reproduce the root, belongs to another
header or duplicates an index already supplied is not that index's section,
and a reader passes over it.
Blocks may come from any supplier; the root, not the supplier, authenticates
them.

## 5. Reading a transaction

The profile reads a transaction's outputs from its unsigned bytes under this
grammar, the reference node's layout restricted to what a publisher needs.
Every VLQ is minimal and within its field's bound:

1. inputs: a VLQ count at most 65,535; each input a 32-byte box id, a VLQ
   proof length of zero, a one-byte count of at most 127 extension entries,
   and each entry a key byte and a `Coll[Byte]` constant;
2. data inputs: a VLQ count at most 65,535 and that many 32-byte box ids;
3. token ids: a VLQ count at most 2^32 − 1 and that many 32-byte ids;
4. outputs: a VLQ count at most 65,535; each output a VLQ value at most
   2^64 − 1, a tree, a VLQ creation height at most 2^32 − 1, a one-byte token
   count with each token a VLQ index at most 2^32 − 1 and a VLQ amount at most
   2^64 − 1, and a one-byte register count of at most 6 with that many
   `Coll[Byte]` constants, the registers `R4` onward in order;
5. nothing after the last output.

A `Coll[Byte]` constant is the type code `0x0e`, a VLQ length at most 65,535
and that many bytes. A tree is one of:

- a sized tree: a header byte with bit 3 (`0x08`) set and bits 5–7 clear, a
  VLQ size at most 2^32 − 1 and that many bytes;
- pay-to-public-key: exactly `0x00 0x08 0xcd` and 33 bytes;
- the miner-fee tree: exactly the reference node's fee proposition
  (`minerRewardDelay` 720), these 105 bytes:

```text
1005040004000e36100204a00b08cd0279be667ef9dcbbac55a06295ce870b07029bfcdb2dce
28d959f2815b16f81798ea02d192a39a8cc7a701730073011001020402d19683030193a38cc7
b2a57300000193c2b2a57301007473027303830108cdeeac93b1a57304
```

Pay-to-public-key and the fee tree are complete expressions, so no other
tree read here begins with either.

A transaction outside this grammar carries no record here. Its id still
enters the root like any other, so its block keeps its section: no
transaction can deny a range by being unreadable. Every reader frames the
same committed bytes alike, and a transaction outside the grammar is one its
author could have written inside it, so a publisher spends plain boxes, pays
change to pay-to-public-key and the fee to the fee tree, and writes
registers as `Coll[Byte]`.

## 6. Attribution, reassembly and order

An output is **at** location `k` when its tree equals `script_k` exactly.
It has the **shape** when its own `R4` is a `Coll[Byte]` of exactly 32
bytes, the **subject**, and its own `R5` is a `Coll[Byte]`, the **piece**;
other registers are not read. An output at no location or without the shape
is not an object here.

- **Kinds 1–3.** The object is one output with the shape at that kind's
  location; its record is the piece, and the object is omitted unless the
  piece's length is the kind's exact length (136, 233, 96).
- **Kind 4.** The object is a maximal run of adjacent outputs of one
  transaction, each with the shape at the kind-4 location and all with one
  subject, in output order; its record is the pieces concatenated, and it is
  omitted where that exceeds §13.1's 131,914 bytes. A publisher separates two
  publications of one subject in one transaction by another output, or uses
  two transactions; a run that merges two does not decode under §6 of
  pool-v3 and has no force, which is the publisher's cost.

The **ordinal** of an object is `position · 2^32 + output`, where `position`
is its transaction's index in the block's section and `output` its first
output's index, both from zero. Ordinals therefore compare as the chain
orders objects, transaction order then output order, and ordinals of
different subjects at one index are positions in one section (C2b.4.2). The
profile applies no signature, sequence, kind or content rule: an object that
nobody signed is carried under its subject and §13.3 disregards it, and
identical objects at two positions are two witnessings of one object.

## 7. Answers

A reader's verifier answers a request whose venue is this identity, whose
`toIndex` is witnessed (§2), and every index of whose range has its section
(§4), and no other. The answer's entries are, index by index, every object
§6 attributes to the request's kind and subject: for kind 4 in ordinal
order, carrying the ordinal; for kinds 1–3 with ordinal zero, in ascending
record-byte order. Every output of every transaction of every section in the
range is read, so an empty answer is proven by exhaustion (§13.2). Where
`toIndex` is not witnessed, a section is missing, the request names another
venue or a local budget is exceeded, there is no answer and the read is
unresolved.

## 8. Publishing

A kind-4 object is one transaction's run, so a publication must fit one
transaction the network relays and includes. A pool-v3 configuration is
publishable on a venue under this profile only where its largest publication
fits one such transaction as pieces at that venue's kind-4 location. §13.1's
131,914-byte bound is the frame's parser bound over pool-v2 §12's generic
proof limit, not a size the chain must carry. For a pay-to-public-key
location, under the reference node's consensus limit of 4,096 bytes a box
and its default relay policy of 98,304 bytes a transaction, a piece of 3,981
bytes fills a box and 24 pieces, 95,544 bytes of publication, fill a
transaction, which holds every publication of a configuration whose proofs
are at most 94,702 bytes; a longer location tree leaves less room in each
box. Each box carries at least the network's minimum value for its
size, which a publisher pays; objects a stranger publishes at a location
cost a reader their bytes and signature checks, priced by the same minimum.

## 9. Forks

Every rule above is the reference node's reading of the mainnet. A soft
fork, a change the reference node still follows such as a voted rise of the
block version, changes none of it: every header version is read and scored
(§3), every section is read (§4), and a transaction the reference node still
accepts is serialized as that node reads it, so the grammar reads it alike.
A hard fork, a change to the header layout, the work or difficulty rule, the
root rules or the transaction layout that the reference node would not
follow, splits the chain, and this profile follows the side the reference
node follows. Once the network has left that side, it is a branch a
minority can extend, which is the venue's failure for every backing that
declares it. The profile is part of what the venue identity names (§13.1),
so a revision for a hard fork is a new context and a new venue. C2.3.1 moves
a venue only under **E**'s replacement rule, and pool-v3 defines no record
that does so: its kind-2 record names a successor party, not a venue. Until
such a record is defined, a backing leaves a forked venue only through a
successor with new terms and a swap (C1.6). Hard forks are announced before
they activate, so that move can precede the fork, at the price of every
holder's swap.

## 10. Costs and C0a

Exhaustion costs the range's section bytes: each transaction is hashed once
and framed once in time linear in its length. The replacement chain and a
revocation are read from index zero (§13.3), the anchor's child, so the cost
is bounded by the deployment's age rather than the chain's, and a reader may
keep its answers while the finality rule stands, paying once per venue and
then per block. It also retains every header from the anchor's child, about
221 bytes each on the wire, and checks each header's work once. Measured
costs are in the reference's
[probes](https://github.com/mediumofexchange/reference-ts/blob/main/docs/POOL_DEPLOYMENT_PROBES.md#real-chain-exhaustion-cost-from-a-real-anchor).

This adds no primitive beyond the venue's own hashes and proof of work, no
signature, no authority and no party whose word stands for the record. Its
parameters cost cohesion: each anchor, depth and location set is its own
venue, and backings presented in one demand or read together (invariant 24,
C3.3, C2b.4.2) must declare one venue, so a deployment publishes one
parameter set for the backings it expects to be held together. What
it replaces for pool-v3 is the source-neutral gap §13 left: the verifier is
fixed, not assumed. Alternatives and their costs:

- **A node the reader must run.** Every reader carries a full node's storage
  and operation; the header rules here let a reader verify the chain from
  its anchor with header bytes alone, and a node remains one supplier.
- **Inclusion proofs or a node's box index.** They prove presence, not
  absence, and §13.2 requires exhaustion.
- **Decoding transactions with an Ergo library.** A library that refuses a
  transaction the node accepts leaves its block without a section and denies
  every range through it; the grammar in §5 cannot refuse a block, only
  decline to read a transaction its author could have written readably.
- **The node's JSON split of fields.** The root binds the concatenated bytes,
  not where one field ends.
- **Reading from the chain's genesis.** Reads from index zero would scan the
  whole chain; the anchor bounds them by the deployment's age.
- **A fixed depth, anchor and locations.** One parameter set would make
  every Ergo backing share one venue, but the depth is part of each venue's
  name (C2.3.2) and the anchor bounds each deployment's reads by its own age;
  a deployment chooses them, and shares them where its backings are held
  together.

The evidence behind these rules (real mainnet headers and sections, hostile
inputs, publication and reassembly on a node, latency) is in the reference's
[Ergo venue guide](https://github.com/mediumofexchange/reference-ts/blob/main/docs/ERGO_VENUE_PROFILE.md).
