# The shielded pool, construction v3

## 1. Status and scope

This document fixes the successor's six proof relations and their public-input
orders. It implements the selected [recovery](pool-recovery.md),
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
statement and authorization records, publication and signed-object bytes,
evidence chains and snapshot commitments, replay/import rules and resource
bounds. The [fault](pool-fault.md) and [spent-set](pool-spent.md) contracts
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

Withdrawal, kind 5, has no proof relation or verification key. Its eventual
statement layout and authorization must implement pool-recovery C3.6.

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
