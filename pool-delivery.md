# Pool delivery and restoration

Normative contract for the successor to `moe/pool/v2`, implementing
Construction C1.2, invariants 25–26 and C4. It does not reinterpret v2 notes,
bytes, keys or delivery. The successor must declare this contract and fix its
profile and statement layouts in its configuration before use. This document
selects the wallet contract; `pool-v3.md` still waits on F4 and the remaining
layout decisions. The smallest supported profile uses explicit payment
requests, constant-payout roots and no reliance graph.

## 1. Receiver-prepared outputs

**C4.1 A payment request names one exact output.** The receiver prepares the
note opening `(domain, backing, value, owner, rho)`, its commitment `cm`, and
an opaque recovery capsule. It gives the payer the opening and capsule over
the channel by which the payer authenticates the intended recipient. It
never gives the payer the spend secret or a wallet root key. `value > 0` for
a requested payment. A reusable address for arbitrary unsolicited amounts
is not supplied by this profile: the receiver chooses the backing and exact
amount before the payer constructs its statement, as it already contributes
a fresh owner for each expected payment in v2 §3.

The payer verifies the opening, commitment, supported domain and capsule
framing, agrees to the requested backing/amount and inserts that exact output
and capsule in the statement. A request is not a payment, invoice signature,
identity credential or proof that a payer owns the inputs. Substitution on
an unauthenticated request channel can redirect a payment; the cipher does
not authenticate a human recipient. A multi-output payment gives each
recipient its own exact requested output. The payer prepares its own change
and zero-value output positions by the same procedure, under its own seed.
All capsule positions have the same public length, including zero outputs.

**C4.2 The receiver derives the opening.** A wallet root seed is 32 uniformly
random secret bytes, backed up securely; a low-entropy password is not a
seed. At creation of a request, the wallet obtains a fresh 32-byte request
identifier from its cryptographic random generator. It persists that
identifier and the exact request parameters before exposing the request.
An exact retry reads that record, not another draw. A different backing or
amount requires a new request. A restored wallet uses fresh identifiers for
new requests: it neither guesses a lost counter nor enumerates unissued
addresses. Unfulfilled requests and human invoice labels are not seed-only
recoverable; finalized notes are discovered from their published capsules.

Under profile 1, the wallet derives independent `spendKey`, `rhoKey` and
`recoveryKey` using HKDF-SHA256 (RFC 5869), with the seed as input keying
material, `domain[32]` as salt, and respectively these ASCII info strings:

```text
moe/wallet/recovery/v1/spend
moe/wallet/recovery/v1/rho
moe/wallet/recovery/v1/recovery
```

Each result is 32 bytes. To derive a nonzero field element, HMAC-SHA256 under
the selected key covers the fixed-width preimage below followed by a
big-endian `u32` attempt, starting at zero. Interpret the digest as a
big-endian integer and reject zero or values at least the construction's
field modulus, incrementing the attempt. Exhaustion refuses; it never wraps.
The spend-secret preimage is `requestId[32]`; the rho preimage is
`requestId[32] || backing[32] || u64 value`. `owner`, `cm` and `nf` use the
unchanged pool note formulas (pool-v2 §3). If a derived owner, commitment or
nullifier is zero, preparation refuses and the wallet creates a new request
identifier before exposing anything. The note shape and ownership/nullifier
relations are unchanged; it is the receiver, now the output's creator, that
derives rho. No payer input, operator, segment or scope enters this opening.

## 2. Recovery capsule

**C4.3 Profile 1 is receiver-to-self authenticated encryption.** Its plaintext
is exactly `requestId[32] || backing[32] || u64 value` (72 bytes). The receiver
computes the complete opening and `cm` by C4.2 before encrypting. For this
one commitment it derives a 32-byte capsule key using HKDF-SHA256 with
`recoveryKey` as input keying material, the canonical 32-byte big-endian
encoding of `cm` as salt, and ASCII `moe/wallet/recovery/v1/capsule` followed
by `domain[32]` as info. Encryption is AEAD_AES_256_GCM (RFC 5116), with the
12-byte all-zero nonce and a 16-byte authentication tag. Its associated data
is ASCII `moe/wallet/recovery/v1/aad` followed by `u8(1) || domain[32] || cm[32]`.
The capsule is `u8(1) || ciphertext[72] || tag[16]`, exactly 89 bytes.

The per-commitment key is essential: an implementation MUST NOT reuse one
AES-GCM key with that nonce across commitments, nor expose an encryption API
that accepts an arbitrary commitment unrelated to its plaintext. The only
message encrypted under one derived key is the canonical C4.2 request that
produced that commitment. Repeating it produces identical bytes. Altering
its identifier, backing or value changes the opening and commitment, hence
the key, absent a hash/KDF collision. The same-key/different-message GCM
failure is not made safe by a retry rule alone. Rejection sampling, context
framing, key separation and commitment association are part of this profile.

The capsule reveals neither a stable receiving public key nor a scan tag.
Different requests are unlinkable by those fields under the declared
cryptographic assumptions; reuse of an exact request is linkable by its
identical owner, commitment and capsule. Payers still know their recipient's
opening. Transport, timing, scope, invoice metadata and tiny anonymity sets
retain Construction C1.4–5's leaks. Possession of `recoveryKey` reveals request
identifiers, backings and values but does not derive the spend key. Seed
compromise exposes past and future derived keys; forward secrecy is not a
claim of this recovery design.

**C4.4 The statement binds the complete capsule vector.** Every issue, spend
and burn carries one capsule for every output position, in output order.
Its public inputs additionally carry a `deliveryHash[32]` as two `u128`
limbs. The proof must constrain both limbs below `2^128`; the configuration
pins the amended relation and key. The hash is SHA256 over:

```text
ASCII("moe/pool/v3/delivery") || domain[32] || u32 outputCount ||
    for each position: cm[32] || capsule[89]
```

The host checks the supported profile byte, exact count, widths, output
association and hash at admission and replay. No cipher runs in the circuit.
A wrong hash is invalid evidence; missing capsule bytes leave the required
trail unavailable, not an empty output or a valid shorter vector. `pool-v3.md`
fixes the public-input order, record field and profile identity in configHash.
This changes issue/spend/burn relations and their configuration; it is not an
extension to v2's record. The ordered vector and output count prevent
reordering, omission and cross-output substitution. The statement identity
already enters semantic history, exact evidence and receipts, so this adds
no second history commitment or signature scheme. Proof variants preserve
the same capsule vector. A reproof after a lapse may change segment/anchors
but retains outputs and capsules.

A public verifier cannot prove that another wallet can decrypt a capsule.
A malicious payer may prove a valid payment to an output with useless
ciphertext, and a cipher cannot make that payment safe to accept. The
receiver checks its exact requested capsule along with the exact output
before accepting. Such a malformed delivery creates no privileged debit or
reversal path; unrecoverable outputs do not justify changing supply.

## 3. Acceptance, replication and restoration

**C4.5 A received payment is checked against its request and evidence.** The
receiver verifies the statement and its output position, its capsule hash and
exact requested opening/capsule. For final acceptance it independently checks
inclusion in a canonical finalized prefix and current spentness. A receipt
may instead be accepted only as explicitly pending operator liability under
C2.10.9; it is not a finalized note or a seed-restoration guarantee. Replaying
an exact request or statement cannot satisfy a second invoice: the wallet
persists acceptance against the request and commitment before reporting it.
Local invoice labels and accounting acknowledgements require their own backup.

**C4.6 Restoration is scoped to authenticated public evidence.** The reader
chooses a construction domain, declared venue, backings and witnessed judging
index. A complete package contains signed backing terms and configuration,
complete authenticated record ranges and order, replacement/authority data,
checkpoint directories and snapshot preimages, and the full trails and
recursive finalized imports required by descent, finality, fault classification
and recovery. Trails include proofs, authorizations and all bound capsules.
The package covers every scoped backing through that index, not merely a
convenient older checkpoint; adopted recovery events and spent nullifiers are
included under their original judging rules. The seed alone cannot discover
which venues/backings have missing histories. Venue/backing discovery and
package completeness must be established separately, never asserted by a
replica or inferred from successful decryption. A wallet reports unresolved
coverage explicitly, not a globally complete zero balance.

The wallet obtains whole relevant trails in bulk, verifies them independently,
and tries capsule decryption locally at every output commitment. Failure to
decrypt is ordinary for someone else's capsule. On success it derives the
opening and secret by C4.2, recomputes `cm` and the immutable nullifier, and
requires the exact output association. It returns only positive notes unspent
in the fully replayed current state; spent and zero outputs are not holdings.
It constructs paths locally from the verified forest. It neither fetches a
single identified output nor asks a server to test a recipient key. No trusted
restore service, spending reset or original payer/operator is required.

The payer and original operator may disappear only after independent copies
of this package survive. Backers replicate public trails and holders keep
what pays them (C0b); acceptance requires obtaining and arranging independent
retention of the full evidence, not merely obtaining ciphertexts. This is an
availability obligation, not a proof of permanent availability: if every copy
is lost, restore is unresolved. Binding a capsule proves its bytes, not that
anyone will continue serving them. A restore of an older complete package is
historical only, never current permission to spend.

**C4.7 Lit settlement uses the already public opening.** C3.5 publishes the
backing, quantity, owner, rho and output commitment. For an acceptance the
backer derives its nonzero owner secret under a separate 32-byte key:
HKDF-SHA256(seed, salt=domain, info=ASCII("moe/wallet/recovery/v1/settlement")),
then the C4.2 rejection method with preimage
`demandStatementHash[32] || u64 acceptanceDeadline`. It signs the resulting
owner in C3.4's acceptance. Restoration scans public settlements and their
exact authorizations, derives that secret and verifies the owner/commitment
before checking spentness. A zero owner refuses acceptance before signing.
As in C4.6, the output must actually have been created: the settlement is in
canonical finalized history or its release has force under C2b.3.2. A valid
proof or acceptance alone creates no holding; an expired or conflicting
release creates none. A venue-created output awaiting adoption is reported
as such, not as an ordinary spendable note with an invented certified anchor.
Ordinary spending requires its canonical adoption and certified path.
No capsule or deliveryHash is needed for settle;
demand, withdrawal and request create no output. Reusing the same acceptance
reuses its owner, which is already public and linked to that demand. The
backer never relies on the presenting holder to deliver a spend secret.

## 4. Cost and adoption

**C4.8 This replaces private payer delivery for the successor.** It preserves
receiver-only authority and the note/nullifier formulas; it requires an exact
request per expected payment, a new wallet encryption profile, additional
public inputs in three relations, and 89 bytes per output plus 64 bytes for
the two public field encodings per statement (before record framing): 242
bytes for two outputs, 331 for three. Every reader checks one SHA256 vector;
a restoring wallet performs one HKDF/AES-GCM trial per capsule-bearing output
and computes note hashes only for its own successful decryptions. For each
public settlement inspected it derives a candidate secret and owner, then
the commitment/nullifier on a match. Costs of full history,
proof verification and authenticated range reads are additional. No mobile
latency or retention budget is inferred from these byte counts.

A later construction adopts this contract explicitly; earlier notes keep
their original delivery and backup requirements. Construction C4's lost-opening
residual becomes conditional for this profile: loss of local openings alone
is recoverable with the seed and complete surviving public evidence; losing
the seed or all copies has no protocol remedy. This contract does not supply
an unsolicited-payment address, pending-operation backup, a witness write
adapter, a permanent storage guarantee or completed v3 runtime recovery.

References: [HKDF](https://www.rfc-editor.org/rfc/rfc5869.html),
[AEAD and AES-GCM](https://www.rfc-editor.org/rfc/rfc5116.html).
