# Transparent pilot v0

This experimental implementation profile exercises the simplest payment path
in Construction. It does not change B=(K,P,R,E) or the authorization law.

## Scope

One operator serves one signed root backing: R is empty and P is a positive
constant quantity of an external named thing. E names the operator and one
explicit local witness with immediate finality. No replacement, non-service
aggregate or silence clause is enabled in this initial profile. Issuance,
transfer, burn, demand, acceptance, release and withdrawal keep Construction's
signatures, nonces and conservation rules. Redemption in external goods is
acknowledged by the holder's release, not proven by the computer.

The local witness is a development trust assumption. The operator service
hosts its record; it is not an independent censorship-resistant venue. A
receipt has local finality only after the durable witness record includes it
and a receiver validates the lawful committed history. Privacy, independent
chain witnessing and recovery against a hostile operator remain outside this
profile. An outage is recovered from durable local history; permanent loss of
that history remains loss. Software must identify these limits to users.

Registration and advancing the local witness require operator administration
authority. Wallet transport credentials permit signed submissions and reads;
they confer no issuance or spending authority. The root must be registered
before the first witness record. The pilot does not add new roots later.

## Durable execution

The implementation preserves the ordered canonical commands and exact
responses in a transactional journal. Replaying it must reconstruct the same
ledger, nonces, receipts, commitment sequences and witness indices. A command
is exposed to a caller only after its transaction commits durably. A repeated
command identifier with identical content returns its original response;
different content under that identifier is rejected.

The database serializes competing writers. The signing key is bound to the
journal; a different key or incompatible journal version must be refused.
Replay validates the commands and compares their deterministic responses to
the stored responses before the service answers. No partially reconstructed
state may be served. A failed transaction discards its speculative in-memory
state. Local witness publication is part of the same transaction, so replay
never broadcasts historical records to an external venue.

This replaces process memory as the pilot's recovery source. It does not make
replaying all history an acceptable long-term scaling strategy. The format is
versioned and the journal is bounded for this pilot; growth limits must fail
explicitly rather than silently omitting history. External publication will
require a durable outbox and signer reservation before broadcast; the local
transaction cannot be assumed to make a remote write atomic.

## State transitions and evidence

| Event | Preconditions | Durable effect | Receiver evidence |
| --- | --- | --- | --- |
| Register | Canonical root terms; valid obligor signature; declared operator and venue | Retain signed terms | Canonical terms and signature |
| Submit | Supported signed operation; correct nonce and authorization; lawful transition | Append operation and exact operator receipt | Receipt proves acceptance, not finality |
| Witness | Operator state is current; next venue index | Advance local index and retain the signed commitment | Directory, relevant log, receipt and witness record |
| Retry | Same identifier and canonical content | No new operation | Identical stored response |
| Restart | Compatible journal and signing key | Replay and compare every committed response | Same witnessed state and outstanding receipts |
| Failed transaction | Storage or validation failure | No accepted new command | No success response |

The verifier must bind the payment to the expected backing, recipient, amount
and signed request; verify the obligor signature, operator receipt, commitment
and directory; check the relevant history's authorization and conservation;
and establish inclusion at a witnessed index from the configured venue view.
A signed but unwitnessed acceptance is pending. Missing or malformed evidence
must not be reported as final. Final acceptance is not a proof of the
recipient's current spendable balance.

A payment checker reports inclusion of a transfer, not freshness for a new
purchase. A receiving application must pin acceptable terms independently and
durably associate each accepted transfer with one invoice or delivery before
fulfilling it. Reusing valid historical evidence must not buy a second delivery.

## Acceptance milestone

Run distinct wallet and service processes over a local transport. Issue,
transfer, witness and independently verify; interrupt execution before and
after durable commit, retry lost responses, restart with the same journal,
then redeem. Also exercise competing writers, mismatched command identifiers,
wrong signing keys, malformed requests and altered payment evidence.

Use Node.js 24 for the pilot's SQLite store; the core reference library retains
its separately documented runtime support. Pin the companion specification
revision and commitment format in the implementation profile documentation.
