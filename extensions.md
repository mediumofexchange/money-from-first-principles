# Extensions

### What need builds on the core, and what each costs

The core keeps the payout to arithmetic over witnessed time, the supply to two checkable settings, and the record of a refusal to a published demand. Nothing here changes that object.

Every extension below is ordinary published terms, priced when somebody accepts them, like any other promise. What makes it an extension is the machinery a wallet or a sequencer has to carry to work it out. So each entry states the need that summons it, the mechanism, and the price. Complexity arrives when somebody pays for it. The build mechanics live in [Construction](construction.md).

## State-reading payouts

**The need.** Some promises are about the present rather than the calendar: pay less as my outstanding grows, pay on this pool, pay on that index.

**The mechanism.** The payout's argument list widens from time alone to:

> **P : (time, this backing's published totals, a named fixed list of backings with their payouts and totals, named external references) → quantity of a named thing**

The first three arguments are safe because the protocol can prove them itself. Outside the pro-rata conditions below, reads of this backing's own totals must not fall as the count rises, or issuing to yourself becomes a dial on the payout.

One form has to be added to the language: a named backing's payout at a stated instant. A live read of a dated backing after its maturity returns zero exactly when you need the number. That form can be worked out at acceptance only where the payout is fixed by data that cannot change.

Two guards keep evaluation total. A forward reference must sit under a time guard. And a reference past a backing's expiry resolves to its last value. The guard reaches references whose record has stopped, a retired backing or a dark sequencer. It never rewrites a dated payout's own arithmetic: a payout past its zero-date reads zero by arithmetic, and the trigger's fifth condition reads that arithmetic.

**The price.** Evaluation instants, monotonicity checking and guard rules, which is most of the payout-language machinery the core deleted.

## External references

**The need.** Parametric insurance, escrow release, a stablecoin reading its declared reserve. Promises about the world need a way to read it.

**The mechanism.** A payout may name external references, under one hygiene rule: **references must be independent of the backer, redundant, and with declared fallbacks.** A backer running its own reference measures with its own ruler, and reopens every channel the fixed argument list closes.

**The price.** An oracle surface. Manipulating a named reference is a real attack. The exposure is declared and priced when somebody accepts, and cover written over a root that reads references inherits reference risk it did not write. So retrofit cover concentrates on the simplest roots, which is one more force toward standard templates.

## The trigger

**The need.** The core's cover can be exercised whenever it is in the money, so it is priced as an option and costs more than the historical bill of exchange ([paper §17](money-from-first-principles.md#the-bill-of-exchange-with-recourse)). A guarantee should pay on failure, not on fear.

**The mechanism.** A payout reads the refusal record. Three parts: the read, what counts as a demand, and the latch.

**The read.** Each cover declares a denominator *D*: the root's outstanding at the cover's signing, or a constant the writer states. The **unanswered share** at an index is the quantity committed under qualifying demands standing unanswered at that index, divided by *D*. Two rules fix the numerator. A demand stands unanswered at an index only where the index is at or after its deadline and no acceptance for it stands witnessed at or before it. And a nullifier named by more than one standing demand counts once, at the largest proved lower bound.

The payout pays zero until the share has stood **at or above** a declared level *s* through *k* consecutive witnessed indices at or after a stated time *T*, with *k* a declared term and one its floor. It reads a level, never a crossing: it asks whether the share has stood at or above *s*, not whether it rose through *s* while anyone watched, or cover written after the distress began would never arm while still reading as valid.

One thing no term removes: the arming condition is writable by the root's own obligor, which can issue to a key it controls, file, and refuse, at the price of locking that quantity through the window and publishing its own dishonour. The fee prices an option the root itself can exercise, the way [paper §18](money-from-first-principles.md#18-limits) prices the credit event itself.

**What qualifies.** A demand qualifies on five conditions. Its deadline stood at least a declared **answer window** *w* after filing, the filing's witnessed index being direct publication or the aggregator commitment that includes it. It names its claims by nullifier hash (`H(nullifier)`, [Construction §C1](construction.md#c1-claims-and-wallets)), so duplicates over one note are visible, and, since a membership proof shows a real note but not its size, the quantity it commits is opened or range-proved and counts at the proved lower bound. It is witnessed at the venue named in the root's **E**, and a demand witnessed anywhere else does not qualify; trigger cover names that venue as its own, so *T*, the scan and the cover's dates read one clock. The root pays in a claim or a chain asset, never in kind: an in-kind acceptance has no leg to lock, so the lapse rule would have nothing to read, and plain dated cover is the in-kind instrument. And the root's payout at the demand's deadline is not zero, read as arithmetic, since claims left outstanding past a zero-date cost nothing to buy and a retired backer is not watching.

An acceptance answers a qualifying demand only where it agrees the demand's instant, covers its full named quantity, and carries a deadline at least *w* after its publication. Its paying leg is locked where it lives when the acceptance is published: an escrow for a chain asset, a witnessed lock at its sequencer for claims. Answering costs lockup in every setting. The **lapse rule** then reads the leg. An acceptance whose paying leg does not stand at its deadline lapsed on the backer's side, and the demand returns to unanswered at that index. One that lapses because the holder never released stays answered, since one party's inaction must not write the other's failure. Where the root's sequencer stood at the aggravated grade when the leg failed, the demand does not return, and the acceptance's deadline extends by the silence.

**The latch.** Once armed, the read stays armed, or the trigger races cover holders and pays whoever withdraws and presents first. The semantics is a bounded maximum over the witnessed indices at or after *T*, a form the payout language allows ([Construction §C0b](construction.md#c0b-the-payout-language-and-the-publication-layer)). The payout resolves against demands witnessed at the venue named in **E**, filed directly or through an aggregator's witnessed commitment. Acceptances, releases and withdrawals are witnessed where demands are, and a demand counts as unanswered at an index only if no acceptance for it stands witnessed at or before that index. A withdrawn or voided demand counts at the indices it actually stood unanswered, and prospectively no longer. Each cover computes its own share from that record, with its own declared denominator, *T* and *w*, and takes the maximum over indices at or after its own *T*. A sequencer-published running mark is an evaluation aid a wallet checks against the witnessed record, never authoritative. The latch is also what satisfies invariant 19, since no move on the writer's own signature lowers a maximum over history.

**Why the denominator is declared.** A live denominator can be diluted, since a backer facing a rising share can issue to itself and push the share back under *s* at no cost. A declared one drifts instead. It means less of the float as the root grows, so it becomes a hair trigger, which is the writer's loss, and it gets cheaper to arm by self-filing every year the root grows. Dating the cover bounds both.

And the denominator is taken from outstanding, never from circulating, since a backer's disclosure of its own holdings must never be able to suppress a trigger. In a chain asset, a backer answers a demand by posting a full-value escrow that stands at each index, so holding the trigger off costs continuous lockup. In kind, an unanswered demand is evidence rather than proof, as notarial protest was: proof of presentment and refusal, never of ability to pay.

**The price.** Five terms carry the judgement a court would otherwise supply. The writer picks them, the buyer prices them, and different values over one root are a term structure rather than a conflict.

| Term | What it is | What it stops |
|---|---|---|
| *s* | the share of the declared denominator that must stand unanswered | too low and a slow-but-solvent root converts the cover; too high and a real failure never arms it |
| *w*, the answer window | the minimum gap between filing and deadline | a demand due next index, which no backer can answer, arming the trigger against a solvent root |
| the denominator | outstanding at signing, or a stated constant | the backer diluting the share by issuing to itself |
| the lapse rule | which lapsed acceptances still count as answers | a holder's own inaction being recorded as the backer's failure |
| *k* | consecutive indices the share must stand at or above *s* | a coordinated filer synthesizing one index and withdrawing |

*s* is a level ever reached, never a level sustained. The latch does not disarm, so the share standing at or above it through *k* indices converts the cover for good, and it will pay on presentation, even where the root then pays in full.

Procyclicality is real, since the trigger rewards filing demands during stress. And without *w*, a holder of *s* of the denominator arms the trigger against a solvent root, withdraws, and holds option-grade cover bought at guarantee price. With it, arming needs the backer's actual silence for *w*.

The trigger also puts work on the reader. Evaluation is a scan of the witnessed demand record since the earliest relevant *T*, and a backing no trigger is written over pays none of that.

Demands have to be witnessed to count, which costs most exactly when the trigger matters. A neutral **aggregator** answers that: one witnessed commitment covering many demands, each provable by inclusion, so filing stays cheap when the backer's own sequencer is the one stalling. It originates nothing and holds nothing, so it is a service rather than a fourth role beside [paper §9](money-from-first-principles.md#9-the-law)'s three.

It has its own price. The aggregator returns a signed, indexed receipt on filing, and commits at a fixed cadence, publishing an empty commitment when there is nothing, so delay is itself an object. A filer holding a receipt and a non-inclusion proof against that index proves omission, which turns inclusion-proves-inclusion into a completeness check. What remains is position: the aggregator still sees every demand before anyone else, for one cadence period, and nothing stops it holding cover. Filers pick aggregators the way they pick venues.

### Reading the sequencer's silence

Non-service and missed commitments are published objects ([Construction §C2b](construction.md#c2b-failure-silence-and-recovery)), and they belong to the same family as the refusal record: witnessed protocol facts rather than outside references. So the hygiene rule above does not bar a payout from reading a backer-run sequencer's own record. The record runs against the sequencer, the holder publishes it, and faking one means holding a real claim.

That matters for cover written after the fact, over claims already circulating, by a stranger the underlying never agreed with ([paper §21](money-from-first-principles.md#21-what-is-new-here-and-what-is-not)). [Construction §C2](construction.md#c2-sequencing) writes one term against a stalling sequencer: a dated instrument extends its dates by the duration a witnessed lock request naming its own legs stands unserved, and a consenting underlying extends its zero-date the same way. A stranger cannot ask the underlying for consent. What it can do is write the same term on itself, under §C2's gates: the object stands through a full silence, since a dark sequencer neither locks nor refuses, and no holder can manufacture it against a serving one. And the recovery leg protects itself part of the way: the guarantor's own demand at the underlying, filed on receipt of the surrendered claims with a deadline at or past the zero-date, fixes the payout at its instant ([Construction §C3](construction.md#c3-presentation-and-dishonour)). That holds against any stall shorter than the run to the zero-date. A stall that outlasts the zero-date defeats a stranger's recovery, since no term of the stranger's can move *Z*; only the underlying's own consent term does, and retrofit cover prices that residual ([Construction §C4](construction.md#c4-threat-model)). A guarantor that delays filing bears the gap.

## Pro-rata, ceilings, and workouts

**The need.** The core's only answer to a run is visibility. A backer who wants to sell run-proofness needs a payout that shares out what is left instead of racing for it.

**The mechanism.** A payout that declines in this backing's own outstanding count, allowed only on strict conditions:

> The count is **provable rather than admitted**, the numerator is a pool held in a contract the backer can neither mint nor withdraw from, the pool **depletes as it pays**, the redemption leg **burns what it takes in**, and the backing's issuance is **capped by a declared ceiling that the pool's contract or operator enforces at the boundary, refusing entries past it.**

Merely admitted, a declining payout is the appearance of a ceiling with the backer picking the arithmetic.

**Depletion is what makes pro-rata fair.** Present batches of 50 out of 200 claims against a pool of 100, at nominal 1:

| | pool **depletes** as it pays | pool a **fixed declared ceiling** |
|---|---|---|
| Rate per claim, per batch | 0.5, 0.5, 0.5, 0.5 | 0.5, 0.67, 1.00, 1.00 |
| Owed over four batches | 100 | **158.3, against a declared ceiling of 100** |
| Behaviour | racing gains nothing | stalling pays: a reverse run on what the pool can fund |

The left column is the theorem. Pay *q* claims at *A/n* and the pool becomes *A − qA/n* against *n − q* claims, which is *A/n* again. Both columns assume the redemption leg burns what it takes in, and the ceiling column's rate is capped at nominal.

**What it buys.** An orderly workout you can write in advance and price at acceptance. And a ceiling that does not need the collapse-to-zero trick, which destroys every honest holder the moment a key is stolen.

**The price.** Checkable counts and contract-held pools, so this extension exists only over checkable settings and chain-held numerators. A ceiling means nothing where the count is merely asserted.

## Pooled cover, and the named list

**The need.** One pool standing behind several backings, and protection against a third party enlarging your exposure.

**The mechanism, and its one channel.** A payout reading a named fixed list of backings is the one channel through which a backer can degrade a claim you already hold. Fixing the list at signing stops anyone *adding* members to a pool. It does not stop the members already named from inflating. So a pooled payout is allowed where the pool is funded by issuance, or where the named backings belong to somebody else.

The same read, run defensively, is the anti-squeeze term: a payout declining in a named backing's published total caps what a stranger's issuance can do to a guarantor's live exposure.

**The price.** Every named total is a channel that moves on somebody else's signature. Declared and priced, never closed.

## Unitload

**The need.** Requiring *z* is not the only way to rest on it. Three roots, A paying a gram of gold, B paying one A claim and C paying one B claim, read flat on completability and load while being three layers on one gram. Leverage through denomination needs its own sum.

**The mechanism.**

> **unitload(z)** = ( Σ_b outstanding(b) · units of *z* owed per unit of *b* ) ÷ outstanding(z), over every *b* whose **denomination chain reaches** *z*

Units owed multiply along the chain: a backing paying one *y*, where *y* pays one *z*, owes one *z* through *y*. Each link is evaluated at the current witnessed instant, where a payout past its expiry resolves to its last value under the guard above, so two honest wallets compute one number without the object needing a maturity field.

Chains end, because payouts name backings by hash and a hash cycle cannot be built. Wallets cap chain length as they cap closures. This is a leverage multiple rather than a reserve ratio, so ten reads as a tenth covered. Grounding passes along the chain, and where the chain ends in a trustless asset, consensus proves the denominator, with no served record to trust. Where the chain is grounded, then, one side of the fraction is proved rather than asserted. A conventional leverage ratio gets that on neither side.

**The price.** The evaluation convention, and the same scoping caveat as every graph number, since junk backings move it and summaries are scoped to what a policy accepts. The sum is gross, since a backing's own holdings of what it owes sit on both sides, which is why completability is computed twice. And where the chain ends outside the protocol, in a commodity, the denominator is asserted rather than proved.

## The Chaumian profile, and the denomination ladder

**The need.** Compatibility with mints that already exist, Cashu and Fedimint, and the cheapest build for a small circle whose operator is trusted anyway.

**The mechanism.** Blind signatures with one signing key per denomination rung. Quantities are packed into rungs, and awkward amounts use a swap. Bound the signer by bonding it, rotating denomination keys each epoch, and committing every interval to how many signatures it issued per denomination.

**The price.** Supply becomes the signer's, or its federation's, assertion. Nobody outside it can count valid signatures, and the commitments give you attribution against an over-signing operator rather than proof.

Padding the void list with fabricated nullifiers is free, so non-service cannot be shown either. Trigger cover over a Chaumian root cannot be exercised against a hostile operator, while still reading as valid. The ladder adds denomination fingerprinting, one rung-pattern per amount. And metric quality spreads, so one Chaumian backing inside a closure poisons the numbers above it. The whole-counts rule in R is unaffected either way, since its canonical-naming argument never depended on the ladder.

## Cross-operator presentation

**The need.** A presentable set can span operators: a void at one and a transfer at another. If one leg succeeds while the other fails, the holder has surrendered claims for nothing.

**The mechanism.** Two-phase presentation, prepare-decide-commit across the named sequencers, with witnessed timeouts. The protocol is [Construction §C3](construction.md#c3-presentation-and-dishonour). The core mostly routes around the need: with operators pooling many backings, the common presentation completes inside one pool, and a wallet can swap into one pool, or sell to a dealer, before presenting.

**The price.** Timeout semantics tied to the witness interval, and a liveness dependency on every operator in the set for as long as the protocol runs.
