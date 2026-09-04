# Construction

### What you need to build it rather than to judge it

[Money from First Principles](money-from-first-principles.md) derives the object. This is the machinery. It assumes **B = (K, P, R, E)** and the law.

This document and [Extensions](extensions.md) are the **Medium of Exchange Protocol**: the normative part, and what an implementation tracks. The paper argues; this specifies. Where the two disagree, one of them is wrong and it should be said out loud rather than worked around — see [§C0](#c0-invariants) for what an implementation must never violate, [§C0a](#c0a-what-may-be-added) for the test anything added here has to pass, and [reference-ts](https://github.com/mediumofexchange/reference-ts) for one implementation of it.

- **[The card](#the-design-in-one-page)**: the object, the law, presentation and the wallet layer on one page.
- **[§C0](#c0-invariants)**: what an implementation must never violate.
- **[§C0a](#c0a-what-may-be-added)**: the test a change to this document has to pass.
- **[§C0b](#c0b-the-payout-language-and-the-publication-layer)**: the payout language, and what publishing means.
- **[§C1](#c1-claims-and-wallets)**: claims: issuance, blinding, swaps, leaks, successors.
- **[§C2](#c2-sequencing)**: ordering, witnessing, and the interval that sets most security properties.
- **[§C2b](#c2b-failure-silence-and-recovery)**: stolen and lost keys, the silence clause, and what a holder can still do.
- **[§C3](#c3-presentation-and-dishonour)**: presentation, and how a refusal gets recorded without a trusted recorder.
- **[§C4](#c4-threat-model)**: attack costs and what is left over.
- **[§C5](#c5-deploying-it)**: the build order.
- **[Appendix](#appendix-alternatives-and-what-they-cost)**: every alternative considered, and what each breaks.

---

## The design in one page

```
OBJECT
  backing   B = (K,P,R,E)   signed, public, immutable, named by the hash
                            of all four fields
     K   obligor    the verification key that owes. May be threshold.
     P   payout     what one unit of quantity pays: a claim of
                    quantity q pays q*P. Any arithmetic expression
                    over witnessed time -> quantity of a named thing;
                    declares the unit and the settlement quantum.
     R   reliance   (backing-or-chain-asset, count) pairs; counts whole,
                    in UNITS per unit of this claim. One level;
                    may be empty.
     E   evidence   who currently attests a claim unspent, the venue it
                    commits to, the witness interval, the replacement
                    rule, the claim-layer setting and its construction
                    (transparent | shielded, pooled or accumulator),
                    and the silence clause: two durations, two grades,
                    the non-service aggregate (m, W) and the
                    refusal aggregate (m', W') (C2b, C2).

  claim             bearer quantity against a backing; blind from the
                    second hop where the setting blinds (C1)
  wallet            a set of claims, plus any bare assets held
                    alongside them
  root              R = empty
  grounded          root whose payout chain ends in something that is
                    nobody's liability; transitive
  chain asset       a nameable term, not a backing: no K signs it; P
                    may pay in it, R may require it, nobody promised
                    anything

GOVERNING ASYMMETRY
  Requirements are structural, fixed, machine-verifiable; payouts are
  functional, may float, settle outside the protocol. Hence constant
  whole counts, a fixed list, AND not OR, and a payout blind to the
  presented set. Disjunction on the payout side is free.

LAW
  No act increases what a backer owes (its written maximum) except
  one signed by that backer.
  No act reduces what a holder holds except one signed by that holder.
  K names nobody but the signer. The law is bilateral, the graph is
  not, and every externality sits in that gap (paper 9).

PRESENTATION
  H is presentable at b for q  iff  H contains q of b and q*c_i of each
                                    (b_i, c_i) in R(b)

  WALLET LAYER (computed by anyone from public data unless noted)
  closure(S)   = smallest multiset containing S fully unwindable by
                 presenting at each member; a macro, expanded before
                 hashing. Counts sum where paths meet.

  the three ratios
  completability(b) = min(1, min_i outstanding(b_i)/c_i / outstanding(b))
                      min over empty set = infinity, so roots = 1;
                      nothing outstanding scores 1. load and unitload
                      are undefined over zero outstanding. min ranges
                      over required backings only; chain-asset legs
                      sit outside the ratio.
                      Net of writer-held claims: assemblability, which
                      is whether a holder can complete a set. Gross:
                      whether the writer itself is covered. Net input
                      public only under transparent.
  load(z)           = sum_b outstanding(b)*c_z(b) / outstanding(z)
                      over backings that REQUIRE z; publish the
                      closure-weighted sum as the bound.
  unitload(z)       = sum_b outstanding(b)*(units of z owed per unit)
                                                   / outstanding(z)
                      over backings whose CHAIN REACHES z. A leverage
                      multiple, not a reserve ratio: ten is a tenth
                      covered. [Extensions: evaluation convention]
  None reads as a bare scalar. Scope to accepted backings, or weight by
  the writer. The graph is discoverable downward only: exact over an
  enumerable set, lower bounds outside.

  max liability(b)  = assemblability(b) * circulating(b) * P_b
                      simultaneous exposure; cumulative payout is
                      unbounded. Exact under transparent only.

  two validity qualifiers, fixed at signing (invariant 3)
  metric quality    = weakest claim-layer setting over the reliance
                      closure AND denomination chain
  referenced(b)     = some payout in that closure or chain reads an
                      external reference
```

### Notes on the fields

**K sits inside the hashed terms**, or two keys writing identical terms would collide.

**P is constant for nearly every promise.** P = 1 kg, so the claim count counts kilos. The unit of account lives inside P, and there is no separate face-value field, so scale lives here and nowhere else. Cover is written proportional: `P_H = 0.9*P_N`, never a flat 0.9 kg. The state-reading arguments, meaning this backing's own totals, named backings and external references, are [Extensions](extensions.md).

**R counts units, never claims**, so how a quantity is packed into claims stays the holder's choice. Counts are whole and need not be one, and {N:10, M:7} is fine. A fractional ratio scales into P at signing, so the fraction rounds once at settlement instead of quantizing assembly.

**A public chain is one setting of E, not the absence of one.** Chaumian is an [Extensions](extensions.md#the-chaumian-profile-and-the-denomination-ladder) profile rather than a core setting.

## C0. Invariants

Everything else is a way of satisfying these. An implementation that violates one is a different system.

**Identity and immutability**

1. A backing's name is the hash, under a declared function, of a canonical encoding of (K, P, R, E). The obligor is inside it, so identical terms under two keys are two backings. There is no edit operation, and no way to name a backing whose terms you do not have.
2. **A backing exists only with a valid signature by K over its own name**, or anyone can publish well-formed terms naming somebody else's key as obligor.
3. Metrics are defined over the full reliance closure **and** the denomination chain, and are only as good as the weakest claim-layer setting in either. Strength of supply checking runs transparent, accumulator, pooled, Chaumian, weakest last; privacy runs roughly the other way. A requirement naming unpublished terms needs no rule against it, since unreadable promises price at zero. A second qualifier travels over the same structures: `referenced(b)` is true where some payout in the closure or chain reads an external reference, meaning the numbers rest on something the protocol does not fix. **Both qualifiers are fixed at signing**, since closure and chain name their members by hash.
4. **Every published ratio is instantaneous.** Completability, load, unitload and max liability are stocks. None of them bounds a cumulative quantity.
5. A reliance cycle would need a hash cycle. Do not write cycle detection. It cannot be built.

**Authority**

6. The law (the card states it) governs *authorised* acts. Invariant 8 states what the mechanism enforces.
7. Issuance changes the outstanding count and needs the backer's signature. Reissuance preserves the count and needs no backer signature. Never one code path for both.
8. No clawback, no reversal, no privileged party who can move claims. Invariant 6 governs authorised acts; this one forbids the path existing at all.
9. Fees are ordinary transfers alongside a swap, never a shaved reissue; fee claims are the operator's own holding, never custody. Shaving is an unrecorded burn, and it shrinks anonymity sets.

**Conservation**

10. `outstanding = issued − burned`, in claim quantity, per backing, at every published moment. Not in tokens, since a swap splits a claim without changing quantity, and not in value paid. There is no redemption term: presenting hands claims to the backer, who is then simply their holder, so presentation destroys nothing. Any result that needs the count to fall needs a burn, and a promise to burn is a covenant, so where a construction depends on it, a contract does it.
11. **Supply enforcement is a declared setting rather than a property**: transparent or shielded in the core, with Chaumian as an Extensions profile. Everything computed from outstanding totals inherits the declaration ([§C1](#c1-claims-and-wallets)). Shielded is the setting, and C1's accumulator and pooled forms are constructions of it.
12. **Per backing, at every published moment and in every core construction: inserted − nullified = issued − burned.** That is invariant 10 read at the claim layer. The accumulator construction proves it in public per-denomination arithmetic, with outstanding as Σ *denom(d)* · (insertions − nullifications). The pooled construction proves it per transaction, with a conservation proof over hidden amounts and the boundary lit: every entry carries a logged issuance or a same-backing conservation proof, and every exit is a published burn, while redemption is an internal transfer to the backer, who becomes the claims' holder (invariant 10). Which proof style **E**'s construction declares is which check a wallet runs. Both trails are public and must be served, since the commitment commits to them without containing them (invariant 23).

**Presentation**

13. A holding is presentable at *b* for *q* if and only if it contains *q* units of *b* and *q·cᵢ* units of each *(bᵢ, cᵢ)* in R(b). Units, never claims, since presentability must not depend on packing. One level, no traversal. The *for q* form carries linearity, so partial redemption needs no separate path.
14. Reliance is a conjunction over a fixed list with constant counts. No disjunction, no computed membership, no count that reads state.
15. **A claim's quantity is a whole number of units, the backing's declared smallest slice**, and counts in R are whole, which makes presentation integer arithmetic with one canonical form. Beside the named thing, **P** declares two granularities: the unit, and the quantum of the thing that settlement rounds to ([§C0b](#c0b-the-payout-language-and-the-publication-layer)). Both sit inside the hash.
16. `closure(S)` expands deterministically **before hashing** (the card defines it). Counts sum where paths meet, the stored object is flat, and multiplicities grow like a matrix power, so cap closure size.
17. An unaccompanied claim is inert, never invalid, and still transferable.
18. **Reliance names backings and chain assets only**, things a wallet can hold and a protocol can check, so presentability stays decidable and redemption can be atomic. Goods and services belong on the payout side: a promise to deliver a good is a backing that pays it, and requiring *that* is checkable.

**The payout**

19. A payout may read a quantity only where it is **published**, is **not a function of the holder's identity or of what they present**, and, where the backer can move it freely, **moving it can only raise the payout**. The one exception is a payout **declining in this backing's own outstanding count**, allowed only under the five pro-rata conditions in [Extensions](extensions.md#pro-rata-ceilings-and-workouts). [§5](money-from-first-principles.md#5-what-it-pays) works the own-count case, and the paper's [appendix](money-from-first-principles.md#appendix-a-the-arithmetic) tables both regimes.

    The exception is scoped to the backing's own count, which prices itself. It never reaches sibling totals, which cost one signature to drive up until the payout is nothing. A pooled read is allowed only where the pool is funded by issuance, or where the named backings are somebody else's. **Declining in an argument outside the backer's control is allowed without conditions**: proportional cover, maturity, demurrage. The argument list is the machine-checkable form of the criterion, and it is sound for the first three arguments, since in a restricted total language syntactic monotonicity is checkable. A backer-controlled reference reopens every channel, so the hygiene rule in [Extensions](extensions.md#external-references) applies.
20. The payout reads presentation time, never a claim's age. Blind claims carry no timestamp, and accrual per vintage shatters the anonymity set. An attribute that varies across issuances, a date or a tenor, goes in one backing per value, published before anything names it.
21. The payout resolves against witnessed totals, so the witness interval is the granularity of every state-reading term.

**Evidence**

22. Every state a sequencer asserts must prove against its latest published commitment.
23. A commitment **commits to** the issuance log, the spent set, running totals and the standing demand record, plus the pending-lock set and the construction's supply object: the accumulator root, the pooled note and nullifier trees, or the per-denomination signature count (Chaumian). The void-by-spend check reads the spent set only. It does not contain any of them, and anything checked against them has to be served ([§C0b](#c0b-the-payout-language-and-the-publication-layer)). **The spent set must support non-membership proofs**, since [§C2b](#c2b-failure-silence-and-recovery)'s recovery path proves a claim *not* spent as of the last commitment, which a bare Merkle root cannot do.
24. **One witnessed evaluation instant per presentation.** The instant is **named in the demand and agreed by the acceptance**, two signatures over one object, on one declared venue, no later than the latest witnessed index at signing. Absent an acceptance, the demand's witnessed publication fixes it. The instant is an index on one venue that every backing in the set declares; a set with no common venue cannot be presented in one demand, and presents per backing instead. Never a timestamp the holder signs alone.
25. **The receiver generates the secret.** The payee creates and blinds it, and the payer never learns it. The wrong version, where the payer generates the secret, obtains the signature, hands over the claim and can then spend it first, is the classic footgun in this family, and nothing else here catches it. The one exception is the offline class ([§C2b](#c2b-failure-silence-and-recovery)), caught instead by identity revelation.
26. **The swap is idempotent, and so is presentation.** A repeated request returns the identical prior response; under blinding, blinding factors derive deterministically from a holder secret, so a replay returns the same signature. Without this, a sequencer that marks claims spent and then crashes destroys money. The rule covers locks and releases, so partition recovery simply repeats the request. And every void record **commits to the replacement it authorises**, so a sequencer that voids and withholds the reissue cannot deny what it owed. A crash loses nothing, and the withholding is a countable non-service object ([§C2b](#c2b-failure-silence-and-recovery)).
27. Settling a published demand voids the exact claims offered, and only on the holder's release signature. A backer must never void unilaterally, or non-payment can be recorded as settlement.

## C0a. What may be added

[§C0](#c0-invariants) binds an implementation. This binds the protocol, and it is the test a change to this document has to pass.

Everything specified here is read twice: by somebody deciding whether to accept a promise, and by somebody deciding whether an implementation is honest. Both readings are part of the security model, because a protocol nobody can audit is not secure whatever its proofs say. Every mechanism added is a permanent charge on both readers, paid on every future reading, by everyone.

So an addition earns its place:

1. **One mechanism per property.** A property enforced in two places must be checked in two places and can be broken in two. Before adding a mechanism, ask which existing one should be generalised to reach the case.
2. **No patch on a patch.** A rule that exists to close the gap left by another rule says the first rule is in the wrong place. Move it rather than fencing it, even when fencing is smaller.
3. **Name what it replaces.** An addition should say which rule it retires, or say plainly why nothing could be generalised to cover it.
4. **Derive it from the object and the law**, or from a failure they cannot answer. A rule needing a special case beside them is usually answering the wrong question.
5. **State the cost.** Every step in the paper names what it gives up. A rule that appears to cost nothing has not been examined yet.
6. **If only some deployments need it, it is an Extension.** The core is what every implementation must do; [Extensions](extensions.md) is where a need that summons its own price belongs.

An addition that makes one case work and leaves every reader one more thing to check has made the system worse, however local the improvement looks. The [Appendix](#appendix-alternatives-and-what-they-cost) records alternatives refused on exactly this ground, and refusal is the ordinary outcome.

## C0b. The payout language and the publication layer

This is the largest item in the build, and [§C5](#c5-deploying-it)'s fourth step makes it tractable: ship templates, not an interpreter. Templates buy away the parser, so the canonical-encoding work is not needed, since the hash names the expression and two implementations must agree bit for bit.

Expressions evaluate in exact rational arithmetic, which arithmetic makes cheap. Rounding happens once, at settlement, in the holder's favour, to the payout thing's declared quantum. Under invariant 19's pro-rata form the residue stays in the pool instead. The residue is bounded by one quantum per settlement, so a promise whose quantum is coarse against its unit price invites fragmented presentment. The repair is declaring a finer quantum.

**A payout is an expression, and it needs a language.** The language must be **total** (no non-termination, and no failure mode but a declared fallback), **deterministic**, and **canonically encoded**, so one function has one serialisation and one name. A fallback is part of the expression, and therefore inside the name, because beside the terms it would be an edit, which invariant 1 forbids. So a silent reference still leaves one value every reader computes alike. Numerics are exact rationals in evaluation, never floating, and where the result meets the payout thing's declared quantum (invariant 15), **rounding favours the holder**.

**One form belongs to the state-reading extension rather than the core**: a named backing's payout at a stated instant, which also reaches that backing's published totals, with the two guards that keep it total ([Extensions](extensions.md#state-reading-payouts)). A bounded maximum over a stated range of witnessed indices is allowed on the same terms, which is what makes the latching trigger writable ([Extensions](extensions.md#the-trigger)).

**Every time a party asserts is a witnessed interval index**: a demand's instant, its deadline, a release's effective time, a withdrawal's effective index. Never a wall-clock timestamp, so two sequencers cannot disagree about when something happened. Invariant 21 is the same rule for state-reading terms. Time constants are unaffected.

**Published means retrievable by a stranger**: terms, logs, totals, commitments, demands, successor offers, and the construction's trail, which is insertions and nullifications under accumulator and the transaction set under pooled. Otherwise invariant 12's check cannot run. Content-addressed storage gives integrity, not availability. No institution is needed. Terms nobody can retrieve price at zero (invariant 3), backers pay for replication, and holders replicate what pays them.

## C1. Claims and wallets

Against a backing circulate **claims**: bearer units, each carrying a quantity. A **wallet** is a set of claims. There is no account and no balance record.

**Issuance is logged in the open, and whether the log names the first holder is a setting.** The count needs *(backing, quantity, time)*. Pooled, accumulator and Chaumian can blind the recipient; transparent cannot. A backing may declare identified issuance, which a lender wants. The log is the outstanding count.

**Everything after the first hop is unlinkable where the setting blinds**, per the declared setting (invariant 11).

**Pooled (the default at scale).** One pool per operator across the backings it serves, with amounts and histories both hidden, and per-backing supply proven per transaction and at the pool's lit boundary (invariant 12). So the anonymity set is the operator's clientele rather than one backing's holders ([§14](money-from-first-principles.md#14-privacy-and-disclosure)). Under it, a non-service object carries a membership proof for a real note without revealing its amount, and a demand names its claims by `H(nullifier)` with no blinding factor, so equality across demands is public at filing and the preimage appears only at spend. The per-backing accumulator below is the simpler reference construction.

**Accumulator (reference).** A claim is a leaf in a public append-only accumulator. Spending publishes a nullifier and a zero-knowledge membership proof. Supply is not the leaf count, since leaves are inserted on every swap, so what bites is invariant 12's identity, with both sides public given the trail, which somebody has to serve. Supply is *checkable given availability*: weaker than a chain's, where consensus serves the data, and much stronger than Chaumian, where no data makes the check possible.

Denomination-tagged insertions make the check per-interval arithmetic, at the price of denomination fingerprinting. Hiding the amounts instead, in a value-conserving shielded pool, makes counts prove nothing, since a large leaf can balance a small nullification, so soundness then needs every transaction's proof verified. That removes the ladder and gives one anonymity set per backing, at the price of heavier proofs. The ladder is a way of making the check cheap, and it is where claims quantize, which is what turns R's whole counts from a free normalisation into protection.

**Transparent (the cold start, and wherever supply matters more than unlinkability).** A per-token public ledger of key-controlled balances, transfer by holder signature. Double-spend prevention is free, `outstanding = issued − burned` is exact and externally checkable, and anyone can find clearing cycles. No ladder, swap, blinding or nullifier. What it gives up is unlinkability, and with it resistance to *targeted* transfer censorship, the one thing blinding genuinely provides.

**Chaumian (small circles; an Extensions profile).** Chaum's construction at every hop. It is unlinkable against a hostile signer only where one keyset serves every user, which nothing here checks. No public trail, supply unprovable, suited to circles whose operator is trusted anyway.

A claim is a container. It carries a whole number of units, and packing, meaning how many containers hold a quantity, is a per-construction choice that never touches what a requirement means. That is what lets different backings run different privacy mechanisms, ladders, shielded pools, or none, under one grammar. Where a construction uses a ladder, quantities come in denominations, each with its own signing key, and awkward amounts use a **swap**: hand claims to the sequencer, receive freshly blinded claims of the same quantity.

Who sees what, by setting where it differs:

| Party | Sees | Cannot see |
|---|---|---|
| Sequencer | blinded requests, nullifiers, timing, denominations where there is a ladder; everything, under transparent | who holds or pays whom (accumulator, pooled, Chaumian) |
| Payee | the whole presented set: backings, denominations, quantities | the payer's other holdings or history |
| Backer, at issuance | quantity and time; the recipient only under transparent or identified issuance | later holders |
| Backer, at redemption | the presenter, the specific claims, the quantity | how the claims travelled |
| Public | terms, issuance log, totals, commitments, demands | holdings and transfers, outside transparent |
| Everyone | — | the link between a claim and its replacement, under blinding |

### What still leaks, and what a wallet does about it

Blinding breaks the link between a claim and its replacement. It does not hide correlations, and five remain.

**Session correlation.** Address and timing link a void to its reissue without touching the blinding ([§14](money-from-first-principles.md#14-privacy-and-disclosure)). An anonymising transport is assumed here, not supplied. And under the accumulator, thin intervals pair them regardless of transport: the published batch matches nullifications to insertions by denomination whenever few transactions share an interval, and the crowd is thinnest exactly at [§20](money-from-first-principles.md#20-in-practice)'s village scale.

**Aborted presentations.** [§C3](#c3-presentation-and-dishonour)'s prepare publishes a spent-pending nullifier beside a signed notice, so an aborted note must be swapped before reuse.

**Cross-backing composition (accumulator).** Under the accumulator, several sequencers see one event in a window, and under [§C5](#c5-deploying-it)'s shared operator one observer joins the whole payment. The pooled construction closes this leg: the payment is one shielded transaction. Jitter does nothing, since [§C3](#c3-presentation-and-dishonour) binds the legs under one release. The remedy needs volume, so the leak is worst where [§14](money-from-first-principles.md#14-privacy-and-disclosure)'s anonymity set is thinnest.

**Set fingerprinting.** An idiosyncratic combination identifies its holder to the payee. Pay in standard combinations of widely-issued claims.

**Denomination fingerprinting.** An unusual pattern is a marker, so the ladder should be standard and swaps routine. Swap is the privacy-refresh operation.

Under transparent the issuance log also names who received credit, though not who spends.

### Successors

A backing *is* its terms, so there is no edit. New terms are a **successor** with a standing swap offer, and a swap needs the holder's signature ([§16](money-from-first-principles.md#16-what-emerges) prices the move). There is no successor-policy field, because a promise not to publish a better successor is a covenant.

What answers the squeeze is a term in the cover's own payout, `0.9 · min(1, N₀/outstanding(N))`, allowed under invariant 19: it reads a named backing's published total, and does not decline in the cover's own count. The term answers inflation. Migration, where the underlying drains into a successor, still strands the cover, is priced at acceptance, and stays [§C4](#c4-threat-model)'s residual.

### Swap

The n-party atomic exchange is first-class, and the paper quietly needs it: bilateral netting (a gift if either burns first), clearing a cycle, set-for-set reassembly trades, atomic multi-hop routing. It is presentation's machinery. Every participant locks what it gives, all sign, commit converts every lock at once, any refusal to prepare aborts, and the timeout unlocks everywhere. The fully signed exchange object is the release, publishable by any participant, and read against the same timeout predicate. Presentation is the two-party case with a backer; clearing is the n-party case with nobody.

### Retirement

Cease issuing, and claims live forever unless the payout is dated. That is the messy route, with an attester alive indefinitely. **Date the payout at signing** and retirement is arithmetic. An undated backer can publish a dated successor and offer the swap, taken only if better, so retirement is bought like everything else.

## C2. Sequencing

A **sequencer** serves one backing at a time: the declaration is per backing, and any backing can name a different operator without asking the rest. One operator can serve a thousand backings and usually will. It never holds funds. It co-signs, and refuses a second spend by declining to sign. It is sold as a service, backer-paid, per-transfer fees, or crowd-funded, which lets someone issue without buying a machine ([§18](money-from-first-principles.md#18-limits)).

**Who runs it is the backer's choice**, named in **E**, and it is a priced setting rather than a rule. The per-backing declaration assigns responsibility, not topology. It is the backer's signed promise to honour this operator's truth, and operators pool freely underneath it. Backer-run is the cold-start default: one party to hold responsible, no operator to recruit, and the dominant risk, the backer not paying, is untouched by who sequences.

Two things argue for paying an independent operator as stakes rise. A backer-run sequencer can make claims illiquid without ever refusing payment, and a stall is deniable where a dishonour is recorded ([§18](money-from-first-principles.md#18-limits)). And under Chaumian a backer running its own attester holds both the admitted issuance count and the unlinkable supply, where the public log stops helping; separating them splits accounting trust from performance trust. The silence clause in **E** is the guard either way.

**And the veto reaches upward.** A presentation aborts on any refusal to prepare, so whoever sequences a required backing blocks every presentation above it. Stalling a dated cover to expiry realises the whole exposure at no cost, so an operator serving many backings ([§C5](#c5-deploying-it) recommends that) holds a position rather than only a liveness hazard.

The answer is a term written under the state-reading extension ([Extensions](extensions.md#state-reading-payouts)). A dated cover extends its expiry, and a consenting underlying its zero-date, by the duration a witnessed lock request naming the instrument's own legs stands **unserved** ([§C3](#c3-presentation-and-dishonour)). A request is answered at the index its sequencer publishes a lock or a signed refusal naming it, and accrual runs only while it is neither. Bare silence needs no separate clause: a dark sequencer neither locks nor refuses, so a standing request accrues straight through it. Three gates stop the object being farmed. Accrual starts only past the declared non-service duration. The filer proves it holds every leg the request names. And one request per instrument accrues per interval, up to a declared maximum, cumulative over the instrument's life. Without the term, the operator waits for exercise, then stalls the guarantor into a claim that expires while it cannot present ([§17](money-from-first-principles.md#17-every-money-is-a-setting)). Systematic refusal is the remaining move, and it is countable: **E** declares an aggregate over signed refusals to prepare, a count *m′* within a window *W′* beside the non-service pair. A refusal counts only where the sequencer was obliged to serve the request, its decision venue being one the refused leg's own backing declares, and only where the filer proved it held every named leg, at most one counted refusal per instrument per interval. Firing the aggregate is an aggravated-grade condition, opening snapshot redemption, where the venue-published nullifier stands in for the sequencer's lock and the veto is bypassed.

**Witnessing** keeps sequencers honest. At the declared interval each publishes a small commitment to a widely witnessed venue, typically a public chain, committing to invariant 23's objects. Venue and attester are named in **E** and move only under its replacement rule. An obscure venue degrades load(*z*) for everyone written over the backing, since the venue is where [§11](money-from-first-principles.md#11-redemption)'s upward sums are assembled.

A venue is named together with its **finality rule**, the depth or gadget under which an index counts as witnessed there. **Its record for one key rises in sequence as it rises in index**: a venue keeps a commitment only where it strictly extends the highest it already holds for that key, so one sequence stands at one index, the record's latest is the highest it holds, and a sequence the record steps over is one it never held. A chain orders nothing on its own — anything that decodes and verifies is a record there — so without the rule a commitment anyone can copy off the chain and re-post moves the record's last, and with it every reading taken against it. That is a floor under the interval, or two sequencers answer [§C3](#c3-presentation-and-dishonour)'s release predicate differently. It also fixes the venue's **lag**: the least number of indices by which an act a party signs is witnessed after the clock it signs at — none where publication lands at the clock's own index, the depth plus one where a chain includes in its next block and reads behind itself by that depth — which the replacement rule below floors a handover's lead on, and which the venue's name must therefore fix, as it fixes the depth: two readers of one name with two lags would disagree forever about who was in force at a past index. Witnessing pins order rather than validity (invariant 22 stops divergent histories): backers replicate what exonerates them, holders what pays them. One addition is worth the bytes, which is the backer's standing bid on its own claims, signed, sized, dated ([§14](money-from-first-principles.md#14-privacy-and-disclosure)).

**A replacement is itself a witnessed object.** It is signed by whoever **E**'s rule names, the backer by default, and co-signed by the successor, states the role, the successor and the effective index, and is published always at the successor venue and at the old one while it serves. Each replacement names its predecessor, so the chain from the original terms is walkable. **The successor's own signature is what seats it.** Naming somebody is not a power over them: without their signature a stranger's publication makes a party that never agreed to serve the operator of record, and every fault that attaches to an operator attaches to a key that never touched the backing. It costs nothing to obtain, since an operator that will not sign is an operator that will not serve. Its effective index is later than the index at which it is itself witnessed by at least twice the venue's lag plus one — a record below that floor is not a replacement — twice, because an operator holds one commitment in flight and signs the next only when the record shows that one — once every lag — so reading the record buys it nothing unless the lead also covers the wait for a clock it is free to commit at and the lag that commitment then takes to land — and **it takes effect there**. The floor is what gives every party the record by the last clock at which an act it signs can still be witnessed in the incumbent's term — at the venue's own speed: an act witnessed at or past the effective index was signed no later than that index less the lag, which the floor puts strictly after the record's witnessing, so a commitment the predecessor signed before it could read the record is witnessed inside its term where the venue includes at its lag, and one it signs after is its own choice. Inclusion is bounded below by the lag and above by nothing, so a slow block can still carry a commitment signed before the record past the index; what the floor buys is one index of notice, and what a party still holds uncommitted at that index is a slow block's cost. Without it a record effective at its own witnessing — or, under a finality depth, one dated the depth ahead of the incumbent's next scheduled commitment, whose cadence **E** declares — put that commitment in no term: not the book a successor takes on, placed by nothing, every payment witnessed in it dead and re-spendable, for one record from the rule-holder and provable against nobody. So an operator hands over by committing what it holds at the first clock it can read the record, and by co-signing nothing on or under the backing from the clock at which the lag reaches the effective index, since what it co-signs from there can only be witnessed in its successor's term and dies there; a payee reads the same record, and treats a receipt given from that clock as it treats one from an operator gone dark. Both are the parties' to do and not a door's to enforce: a door that shut on a pending record was a lever — the rule-holder held it shut, one record per lag, by superseding each record before it arrived, with nothing to grade — and what a rolling record buys against a party instead is that party's own caution: one that stops on every pending record can be stopped the same way, one that reads a link re-armed as re-armed keeps serving at the price of one window of dead co-signatures should the next record be real, and the record shows which, though nothing here grades either. A door shut only by its holder's own conduct is the shape a door may have. The successor co-signs from the effective index itself, once nothing the predecessor could still land in its term is unread. It is also **strictly later than the index at which the link it replaces took force** — a record placing two operators in force at one index is void rather than a handover, since it would empty a term retroactively and move readings already read — except where it names the incumbent itself, which is not a handover but a revocation and carries whatever index the first rule allows — the floor included, since that rule is read before any link is, so a revocation dated below it is no record at all; its index is otherwise inert as a date, and a name in the same-index tie like any other's. **Where a fresh seat's walk begins is a fixed object: the backing's last commitment witnessed strictly before the effective index** — the predecessor's own wherever it has one, and past a predecessor that never committed it reaches back to the last that did, exactly as [§C2b](#c2b-failure-silence-and-recovery)'s snapshot does, so one never-committing link does not strand the backing for its next successor; that is where the descent below starts, not what it ends at. Where the record pins nothing at all — no commitment by a party then in force stands strictly before the effective index — what the successor takes on is the **empty book**: nothing was ever final, and never publishing must not read as having published. **And the pinned object is a floor, not a ceiling.** What a seat serves is always the book the record's last commitment stands on: the content of the last commitment, by a party then in force, that **carries** the backing — reached by exhibiting each later one as a state carrying nothing for it, every step the record's own answer to what was last committed in force at or before that index, with the state offered or exhibited matched to that answer by identity, so a seat chooses only how far back to walk and never what stands there — and the empty book where the record runs out. Each step must be **paid**: an exhibit is that commitment's own served state, which only a party holding it can produce, so a successor that cannot obtain every drop above the last carrying state is stranded rather than seated on a state of its own choosing — the honest bound, since withholding a state is not itself a fault — and the number of steps is the number of drops the record holds above the last carrying commitment, which no rule bounds: every one of them is a rewritten history against the key that published it, so an honest record holds none it did not mean — a deliberate stop-serve wears one knowingly — and a hostile one prices the successor's catch-up in exhibits it must obtain. A process that restarts resumes from its **own latest witnessed commitment** by that same walk, because restarting must not read as never having served, and commits nothing until the venue's lag has passed since it resumed — what it published just before it stopped is neither readable nor yet abandoned, and a commitment signed over that sequence is an equivocation against its own key with no adversary in it. A book behind the record's last commitment serves nothing until it is taken over, which is what makes a lost book, a superseded twin process, and a stale handover copy one detectable condition rather than three silent ones. **But a book its own signature put the record on is ahead of the record rather than behind it**: an operator that has signed and published a commitment the venue has not shown yet holds exactly the log it committed to, and serves it — the seat carrying what that commitment stood on as well as the commitment itself, so a record moving to anything else takes the seat at the index it always did and the three conditions stay one. Read the other way, every operator on a venue that reads behind its chain is dark for the lag after each commitment, on the same signal a superseded twin gives and with every grade reading healthy. **One commitment stands in flight at a time**: an operator signs its next only when the record shows the last, or when the venue's lag has passed without it — inclusion is bounded below by the lag and above by nothing, so a commitment the chain has not taken by then is one the operator can no longer assume, and its seat goes stale on its own, one takeover from repair rather than a book lost for ever. Its sequence is one past the highest the operator has itself signed, never merely one past the record's, or a transaction the chain declined frames the operator that signed it. Which backings a commitment carries is not readable from its root, so once the record's last commitment drops a backing, **one exhibit is as far as a door can see**: that commitment, shown to carry nothing for it, reads the same whether the record never carried the backing or the operator lost its book. A seat that can pay the whole descent is told apart there and takes the empty book at the door, on the walk above; where the descent cannot be paid, or is not made, the empty book is a claim the operator **signs in its next commitment** — naming the backing beside that last commitment exhibited as carrying nothing for it, and refused where the operator already serves the backing, is not in force for it, still holds any operation in it, where that commitment still carries it, or where the record holds no commitment at all, since the empty book is then the walk's own answer — because a book held is a book to take over rather than to open. True, it is growth from nothing; false, it is a rewritten history that any holder of the earlier state proves. Registering is holding the book only where the record pins nothing, for every seat, genesis included. Naming the state by an index rather than by whoever is latest when somebody happens to ask is what makes it one object for every reader once the index arrives — the predecessor governs until then and its commitments up to then move the target, but past it nothing anybody publishes does. The lead time between witnessing and effect is what keeps the catch-up small; a replacement leaving only the floor hands over to an operator that is late, which is graded, rather than to one that is holding nothing and cannot get it. Two replacements naming one predecessor resolve to the later, where it was witnessed strictly before the earlier one's effective index — which is how re-naming revokes a successor not yet in force; a lead at the floor on a venue with no lag leaves no such window but the record's own index, where two witnessed at one index resolve by hash, and widening it is the rule-holder's. One witnessed at or after that index names a link the chain has already left and is ignored, since force having arrived the successor is the incumbent, and replacing it is a replacement naming it. Two witnessed at one index, where witnessing pins nothing, resolve to the lesser record hash — the objects' own names, so every reader answers alike from the records and none from the order they reached it. That name is the rule-holder's to grind, and the cost is priced rather than fought: the rule-holder signed both records in any such pair, so grinding buys it an outcome this section already gives it for free by publishing one record alone. So no reading of a past index ever moves. Until the effective index the predecessor governs and goes on serving — a handover that froze the backing in between would hand any named successor a stall — the successor co-signs nothing, and accrual against the incumbent continues. From the effective index the old attester's co-signatures stop counting, which is why a wallet verifies the chain rather than the key it remembers, and accrual against it stops there. **The successor's signature is its undertaking to serve the set in full.** Whether it does is checked against the served state rather than read from the root (invariant 23) — and a successor that has served nothing has no served state, which is the ordinary opening position rather than an excuse — and one that does not serve is answered by the non-service grade like any other operator, read against the last state that did carry the backing. That is what replaces requiring it to prove service before it may hold the role, a bound that made the act conferring the role the same act that proved the fault. **What that costs** is the one thing the discarded bound bought for free: at the effective index the operator of record has consented and has had its lead time, but has not necessarily published anything carrying the backing. So **the effective index is a routing field and never a clock**. It says who answers for the backing from here; a reader that takes it for a measured fact is reading a party's own claim as a witnessed one. Silence is measured on the backing ([§C2b](#c2b-failure-silence-and-recovery)), and a handover neither opens a gap nor closes one. **A term of the chain, rather than a key, is the unit of obligation, of accrual and of fault.** Which term a committed state belongs to is read from the record — the index at which that commitment was witnessed, against the index the next link took force — and is never a field the operator asserts, which would hand the choice to the party with the motive. A key the rule-holder names twice holds two terms and answers for each separately, or a re-appointment launders everything the first term did — and it **co-signs nothing in its new term until it has committed in it**, exactly as a sequencer returning from silence commits before it serves ([§C2b](#c2b-failure-silence-and-recovery)): its receipts would otherwise name an era that ended with its old term, and a lapsed era is the excuse the operator-fault pair reads, so the window between its seat and its first commitment would mint equivocations nothing can prove. A holder reads the same rule from the other side: a receipt whose era predates the operator's current seat is stale on its face, however recently that key last committed. Failure to serve the trail on request is not itself a grade: nothing declares the object, the duration or the aggregate that would make it checkable the way [§C2b](#c2b-failure-silence-and-recovery)'s two are, so it is a reason to have replicated rather than a condition anyone can fire. The checkable remedy against an incumbent that will not serve is the non-service grade, read against the last state a holder was given — so a holder keeps one.

### The witness interval is the main dial for ordering security

Nearly every quantitative property of ordering comes down to the time between publications. Double-spend exposure is the window since the last one. Finality means witnessed rather than co-signed, and that includes a release's deadline ([§C3](#c3-presentation-and-dishonour)). Recovery runs against the last witnessed snapshot. And the minimum life of a dated instrument is a multiple of the interval: [§17](money-from-first-principles.md#17-every-money-is-a-setting)'s bill runs several witnessed indices, so short-dated cover names a fast venue and pays the fee.

Short intervals cost fees, long ones cost exposure, and there is no third option, which is why the interval is a signed field rather than operational discretion. What no interval covers is the dominant risk: the backer does not pay.

**The fee is per backing per interval, which forces batching.** A shared operator publishes one transaction over a root of its backings' commitments. That costs part of the failure-domain claim, since one outage leaves every standing lock request unserved across every backing served, moving their dated expiries and zero-dates together.

**A payment from several backings touches several sequencers.** That is the common case wherever a wallet's backings name different operators, since a wallet spends from a portfolio. If the third void succeeds and the fourth fails, the payer is short and the payee holds part of a price. Two honest answers, pick one. Extend [§C3](#c3-presentation-and-dishonour)'s prepare-decide-commit to any multi-sequencer transfer, at a round trip per payment. Or let payees accept partial-and-retry and price it, as card networks do. Either way *k* backings need *k* sequencers live at once, which is an availability hazard, and [§16](money-from-first-principles.md#16-what-emerges)'s concentration force.

## C2b. Failure, silence, and recovery

**If a backer's key is stolen**, the damage is unbounded and permanent, since K alone authorises issuance and nothing expires. So K may publish a **revocation**: witnessed, and prospective, so existing claims keep their terms and no further issuance is valid.

It is per venue, and effective for each backing at its witnessed index on that backing's declared venue, published by K to every venue its backings name. The boundary is the index rather than the signing clock, so issuance witnessed before the revocation stands, anything witnessed after is void, and a thief's unwitnessed batch dies with it. A payout reading this backing's own outstanding count (invariant 19) reads what stands rather than what was committed, or a revoked thief dilutes every holder by issuing on.

A thief's purpose is to issue, so it revokes only on the way out, destroying the name permanently, since de-revocation would carry the same K. Revocation is the one act no later signature can repair. That is the strongest argument for a threshold K, and for the right to nominate an attester under **E**. Acceptance should attach to a history under **continuous** control, and a break in that control is worth publishing.

**If the backer's key is lost**, there is no further issuance, which is mostly benign, and claims keep circulating. The exception is where the same key attests in **E**, when every claim goes illiquid and the silence clause is the only path out. So separate the keys, or name a threshold attester.

**If the attester's key is compromised**, the loss is worse. Under Chaumian whoever holds it blind-signs without limit, and a bond does not deter a thief. A threshold attester is the defence, which is what Fedimint runs.

**When sequencers go dark**, claims go illiquid rather than dead. Value discounts until they return, and after the declared silence, redemption against the last witnessed snapshot opens without co-signature, with the holder proving the claim unspent as of that snapshot. **The snapshot is the backing's rather than a key's: the last commitment witnessed from a party that was in force for this backing when it published it.** A handover does not discard it, so a successor that has committed nothing is redeemed against the backing's last, whichever term of the chain it fell in ([§C2](#c2-sequencing)) — a chain whose middle operator never committed reaches past it to the last that did. Read instead against whoever is in force now, the remedy would vanish the moment a rule-holder named anybody at all, and the rule-holder is the backer by default, which is the party that owes the money.

Snapshot redemption publishes the claim's nullifier at the witness venue as the release leg, after the backer's acceptance, and a sequencer returning from silence adopts every nullifier witnessed during the gap before co-signing again. Returning is committing: the gap runs until a commitment is witnessed and a publication inside it has force, so the sequencer rebuilds from its last commitment — what it co-signed after that commitment was never witnessed, and finality means witnessed — so a co-signed receipt names the last commitment it stood on — that commitment rather than the index the venue witnessed it at, since on a venue that reads behind its chain the operator co-signs between signing a commitment and reading it and that index does not exist yet, so a receipt naming the previous commitment is a lie about the operator's own log the moment this one lands, and one naming a predicted index is void for ever should a block run slow. A stranger resolves the name against the record, which has three answers about it: it **holds** the commitment, and the era began where that commitment was witnessed; it has **not reached** it, and nothing has ended the era, so the operation is still on its way to one however far ahead the name runs — the gap widens by one with each commitment the chain declines, so no bound on it is sound; or it has **moved past** it without ever holding it, and that commitment died and the era it opened lapsed with its tail, which is this section's own return from silence at less than the declared duration. So a stranger can read whether the position a receipt claims survived the era it was signed in — adopts the gap, commits, and serves from the index after, a publication at the commitment's own index being judged against the record strictly before it. A snapshot redemption also stands for a declared challenge window, during which anyone may publish at the venue the holder-signed transfer request that spent the named claim. On publication the redemption pays the payee the request names instead, and that payee already holds the request. Serving a claim the venue shows spent is equivocation, provable against its own commitment (invariant 22). The unspentness proof runs against the published trail, which replicas serve because publication was the point. Where nobody replicated, the claim waits, and the availability price was already on the table. Under transparent, a signed spend record published at the venue, checked against the last committed balance state, stands in for the nullifier, with the difference that it names a position in the holder's sequence rather than the claim itself. Whether a sequencer can be replaced at all is answered in **E**.

**The clause is measured on service rather than publication.** A stalling backer-run sequencer publishes on time, and the stall shows only as a spent set that stops growing. So **E** declares two durations and two grades.

**Non-service**: a signed transfer request, published where demands are, left unserved past the declared duration and counted only while still unserved. It is [§C3](#c3-presentation-and-dishonour)'s demand shape without the backer, checkable under transparent, accumulator and pooled. Under the accumulator and pooled the request proves a real note without revealing its amount; under transparent it is checked against the published balance state, which reveals both, and is the privacy that setting had already given up. It is unavailable under Chaumian, where proving non-service means exposing the claim, so the remedy there stays the [Chaumian profile](extensions.md#the-chaumian-profile-and-the-denomination-ladder)'s bond and reputation.

It fires in the aggregate, and **E declares the aggregate**: at least *m* distinct requests, each unserved past the duration, standing within a window *W*. **The count stands against the backing and the grade names the incumbent.** A handover neither resets it nor moves it: a successor inherits the standing requests and clears them by serving them, and a party whose duty has ended is never the subject of a present reading — the co-signature is what makes naming the incumbent fair, since no key holds the role without having signed for it with the standing requests in view. Scoped instead to who held the role when a request landed, the count would reset for one publication and a key the rule-holder generates itself; what the inheritance charges an honest heir is what the no-commitment clock below already charges one. Set *m* low and one scripted wallet replaces an operator; set it high and the clause never fires. The holder reads the choice before accepting. Faking a request means holding a real claim, so the count is checkable. But one holder can split a holding into *m* claims and file as many requests, and pooled hides the amounts that would weight them, so no request is fake, though one holder can supply all *m* of them. Firing makes the case for **E**'s replacement rule, which stands whether or not a grade has fired, and moves no dates. The remedy is inert wherever that rule names the backer, and absent wherever **E** names no rule at all.

**No commitment past a second declared duration**, in any setting: the aggravated grade. It opens snapshot redemption and runs from the last commitment witnessed from a party then in force for this backing until commitments resume, and **only a commitment closes it**. Where none has ever been witnessed for it the clock runs from index zero, or never publishing at all would be the way to read punctual forever. The clock is the backing's, so a handover neither starts it nor stops it, and a successor inherits the remainder of its predecessor's window rather than a fresh one. Any clock that reset on an event the rule-holder chooses would be cancellable by the rule-holder: a replacement costs one publication and a key it can generate and co-sign itself, so a party under a standing grade would otherwise clear it by hopping to a fresh key each duration and never committing at all. What that charges is an honest heir appointed to rescue a dark backing — the grade stands, and redemption can open against a backing that has just been rescued, until the heir's first commitment lands. An heir named late into a long silence inherits little of the window or none of it, and cures the grade only by committing, which it can do without having recovered anything. No reader can tell that heir from a rule-holder colluding with itself, so the rule does not try. A shared operator batches ([§C2](#c2-sequencing)), and what a venue witnesses is a root, so whether a commitment carries any particular backing is unreadable from it: the grade fires on the operator publishing nothing, not on it covering nothing. One that drops a single backing while committing the rest on time is reached by the non-service grade above instead, read against the last state that did carry it. Dated terms read neither grade; they read the standing lock request ([§C2](#c2-sequencing)). Silence is not a holder's to manufacture, and a request accrues only under §C2's gates.

A refusal aimed at one holder is out of scope for transfers, where blinding hides the target and transparent gave that up. At presentation it is in scope, since [§C3](#c3-presentation-and-dishonour)'s notice names the holder and the claims.

**Offline payment** is a distinct claim class using Chaum, Fiat and Naor's trick: the claim carries the spender's identity, sealed so that spending it more than once reveals it. It identifies a cheat rather than convicting one, and a revealed double-spender loses its key's accumulated history ([§15](money-from-first-principles.md#15-valuation)). Ordinary claims carry no identity at all.

## C3. Presentation, and dishonour

One mechanism solves two problems. **A presentation can span operators**: a void at one, a transfer at another. If one succeeds and the other fails, the holder has surrendered claims and received nothing, and no rule was broken. **And nobody paid by the backer can be trusted to record the backer's refusals**, since the backing's own sequencer is frequently the backer, and otherwise its supplier.

One protocol answers both. **Consent between the parties is demand-accept-release. Atomicity across sequencers is prepare-decide-commit.** They are two different protocols, and one with only the first leaves the hole open.

1. **Prepare.** The holder locks at *every* sequencer in the set. Under blinding, that is a nullifier marked spent-pending, reserving without consuming. A chain-asset leg locks in an escrow on the decision venue for the lock period. Any refusal aborts.
2. **Decide.** The backer publishes an acceptance, or does not. This is where the backer's side of the outcome is determined.
3. **Commit.** The holder publishes a release to the witnessed venue, effective on witnessing rather than delivery, so every sequencer evaluates one predicate against the same object: was a valid release witnessed at or before the lock timeout? It has to be durable rather than legible, and a transaction-nonce entry is enough.
4. **Abort.** The **lock timeout** declared in the prepare, itself a witnessed index, unlocks everywhere, and expired locks unlock unilaterally. **One attempt carries one timeout**, at every sequencer and for every participant, since an attempt commits or expires as a whole. It is not the demand's deadline: the timeout ends the atomic attempt, the deadline governs evidence, and a demand outlives its locks.

Failure to prepare is ordinary and costs a retry. Failure between decide and commit is what the timeout is for, resolving against the last witnessed state. Publication is not optional, since a release nobody witnessed did not happen, which is [§C2](#c2-sequencing)'s finality applied to commit. The cost is arithmetic: a release published inside the timeout's interval may be witnessed after it, so publish a full interval early, and wallets enforce the margin.

**The boundary is atomic signing.** Single-phase wherever every lock in the set can be taken in one atomically signed decision: R empty and the payout settling outside the claim layer, or the whole set and the paying leg inside one operator. Two-phase only where the set spans operators that cannot jointly sign one decision — for the swap; two-phase *presentation* is the [Extensions profile](extensions.md#cross-operator-presentation), built on the same prepare-decide-commit, and the core files a presentation where one operator serves the whole set or not at all. **A cross-operator prepare names a decision venue**: one venue among those the set's backings declare, on whose witnessed indices the lock timeout is read, so every sequencer evaluates one predicate against one clock. A sequencer unwilling to watch it refuses to prepare, which is an abort rather than a fork.

**Presentment is committed, and published when it matters.** The holder signs a notice naming the backing, the quantity, the specific claims, the evaluation instant and a deadline of their choosing. The named instant is an offer: an acceptance may agree it, and absent one, publication fixes the instant (invariant 24). Both times are witnessed indices ([§C0b](#c0b-the-payout-language-and-the-publication-layer)). Publishing turns a private commitment into evidence, done when the backer is not cooperating.

**An unanswered demand stands**: the claims are committed against payment until withdrawal or settlement. The deadline marks when non-payment becomes a public fact, and it is not the end of the commitment. The protection against stalling is unilateral withdrawal, which the backer cannot wait out. The payout is fixed at the demand's instant up to the deadline and floats after, to zero against a dated backing. So a filer without cover sets the deadline at or past the zero-date, and one relying on cover sets it early enough that the share arms first. Refiling recovers nothing.

**The commitment is enforced two ways.** Where the sequencer serves, by the lock. Where it does not, by the published void counts: the demand names specific claims, and spending them voids it, checkable under the accumulator and pooled, since nullifiers are public where amounts are not. Under transparent there are no claims to name, so the check falls back on the holder's own spend sequence, and a sequence position is the signer's to choose where a nullifier is not: the commitment rests on the lock, and [§C2b](#c2b-failure-silence-and-recovery)'s venue-published challenge binds a careless double-spender rather than a deliberate one. It is unavailable under Chaumian. A lock request left unserved is [§C2b](#c2b-failure-silence-and-recovery)'s non-service object while the sequencer was obliged to serve it: it names the declared decision venue, its attempt is uncommitted at that venue, and its timeout stands.

**The window is the holder's.** The deadline is the holder's own lock-up, so the party bearing the cost sets the term. A backer would be setting the standard by which its own failure is measured. Nothing needs adjudicating: a five-minute window is worthless evidence, thirty unanswered days damning. The deadline is strictly ahead of the index the demand is filed at: any earlier is not a term but a manufactured verdict, since no acceptance could land both at or after filing and at or before it, so the demand would read as dishonoured from the moment it was filed, for one signature, against any backer — and one at the filing index is no window at all.

**An acceptance carries its own deadline**, or the backer holds a free option: accept, keep the claims committed, wait for the payout to move. Under the trigger, an acceptance answers only with its paying leg locked and a deadline at least the answer window away ([Extensions](extensions.md#the-trigger)). The holder may decline to release against an acceptance whose terms have moved, which is the other half of why a refusal burns nothing ([§11](money-from-first-principles.md#11-redemption)).

**Settlement takes two signatures** (invariant 27): acceptance from the backer, carrying the backer's blinded outputs for the set it receives so the insertion is buildable atomically (invariant 25), and void only on the holder's **release**, since a unilateral void would record non-payment as *settled*, which is worse than silence. Claims still live past the deadline are the backer's visible failure — read, where P pays in claims, across both records: dishonour is the branch where no acceptance stood with its payout reserved through that acceptance's own deadline, and an acceptance that stood so reserved and expired unclaimed is the holder's lapse, not the backer's. In kind the release is the receipt, and a backer performing before it is extending credit, where a chain-asset leg records who paid.

**A payout paying in claims settles as a swap inside the settlement.** The acceptance names the claims, or the fresh issuance, that will pay, and the release executes as one atomic exchange, surrendered set against paying claims, co-signed by every sequencer either side needs. That is [§C1](#c1-claims-and-wallets)'s swap run at settlement, so the backer cannot take the set and not pay. Neither party can write the other's outcome. A set is one act and dies as one, so a presentable set spans backings declaring one silence duration — a return from silence rebuilds per backing, and a set across two clauses would tear in half. One commitment therefore shows the whole settlement or none of it, for the backings its operator then held the pen on; half a settlement under one such root is a history no door produced, and provable fault.

Dishonour is then not a separate mechanism. It is the branch where the acceptance never arrives. Voids appear in counts the sequencer already publishes, so a satisfied demand resolves in public with nobody reporting anything. The record is kept by everyone whose backing requires the refusing one, who lose money if it is wrong either way. That is the alignment the sequencer lacks.

**What this does not achieve.** In a chain asset the exchange can be atomic, and refusal becomes proof. In kind it cannot, since each party may assert what suits them, so a demand and its silence is evidence rather than proof, and a dishonest holder can damage a name cheaply. What restrains that is identity: presentment is signed and attributable, answerable at law and against the demander's reputation.

## C4. Threat model

| Attack | Defence | Honest residual |
|---|---|---|
| Double-spend | co-signature against the attester declared in **E** | the window since the last witnessing; for the offline class it runs to deposit, against a key that may carry no history worth losing |
| Sequencer equivocation | witnessed commitments; every asserted state proves against the latest one (invariant 22) | none, once witnessed, beyond the venue's own reorg past declared finality and its censorship |
| Hidden issuance | accumulator: invariant 12, both sides public; pooled shielded: every transaction's conservation proof; Chaumian: a bond, plus per-interval commitment to signatures issued | **Chaumian: unbounded**, since the commitment buys attribution rather than proof. **Accumulator: the check needs the trail served**, and nobody is assigned to serve it. **Pooled shielded: a proof-system break is silent**, and one break reaches every backing in the pool |
| Backer–sequencer collusion | on-chain settlement makes the chain the sequencer; elsewhere the backer is already trusted to perform | soft reserves, drainable regardless of who sequences |
| Backer over-issuing | invariant 19's pro-rata form; collapse at the ceiling as the fallback | a stolen backer key, ruinous under every payout form |
| Selective refusal; probe-and-walk | published demands, two-signature settlement, read as a standing aggregate ([Extensions](extensions.md#the-trigger)) | in-kind claims that cannot be checked; a mark that costs the filer less than the backer |
| Transfer censorship by a backer-run sequencer | an independent operator, named at signing; the silence clause in **E** | an indiscriminate stall, visible to every holder at once |
| Manufactured distress (junk backings inflating load(*z*)) | scope metrics to accepted backings; weight by writer | anyone reading a bare scalar |
| Squeezing cover via a successor; severing it outright | date the cover, and let its payout read the underlying's total ([§C1](#c1-claims-and-wallets)); forced severance is impossible, since a swap needs the holder's signature, while bought severance is the price contest with the writer as the highest bidder, unbounded where its own credit is failing and visible only under transparent or a declared lit key | undated constant-payout cover; the squeeze raises completion cost |
| Non-atomic multi-sequencer presentation | prepare-decide-commit ([§C3](#c3-presentation-and-dishonour)) | an interval of margin on the deadline; roots paying outside the claim layer use the single-phase path; a sequencer ignoring a witnessed release strands the set until the silence clause fires |
| A required backing's sequencer vetoing presentations above it | the unserved-request term and the refusal aggregate ([§C2](#c2-sequencing)) | a depth-*k* set hands *k* operators a veto; under Chaumian neither grade nor the request object can be shown, so the remedy there stays the profile's bond and reputation; a backer holding its own legs can file, self-stall, and extend up to the cumulative maximum, against a standing unserved request naming itself; a stall outlasting the underlying's zero-date defeats a stranger's recovery; a guarantor that delays filing after exercise bears the gap ([Extensions](extensions.md#reading-the-sequencers-silence)) |
| Sequencer concentration | substitutability: any backing may name a different operator | **substitutability is not diversity**: one operator serving most of a deployment is a correlated failure, and replacement runs through **E** while nothing settles |
| Third party enlarging a guarantor's live exposure | publish max liability; date the cover; a capping term is allowed and stays non-canonical, since a cap turns a guarantee into a ration | the underlying's issuance moves realised liability with no signature from the guarantor; outside transparent the netted input is the writer's own account |
| Manipulated credit-event trigger | a latching read over the witnessed demand record ([Extensions](extensions.md#the-trigger)), never a single demand; qualification by an answer window, a per-nullifier count at the proved lower bound, the root's venue, a live payout and a lockable leg, so trigger terms never arm on in-kind roots; *k* consecutive indices; an acceptance answers only with its paying leg locked, so holding the trigger off costs continuous lockup | the latch never disarms; the denominator is a declared term, since live forms dilute by self-issuance and declared forms drift as the root grows; the arming condition is writable by the root's own obligor at the price of its published dishonour; a backer-run sequencer can stretch acceptance deadlines by its own published silence, loudly; an aggregator sees demands one cadence period early and may hold cover ([Extensions](extensions.md#the-trigger)) |
| Backer covering its own root through a second key | weight by the writer's standing, not by counting ([§15](money-from-first-principles.md#15-valuation)) | apparent depth of protection is inflatable at the cost of a real position; and a payout declining in a named backing signed by the backer's own second key dilutes on the backer's signature, since the somebody-else's test is a key test and keys are free |
| Holder key theft or loss | none in the protocol: remedies run at law against the thief (and, where the claim is not negotiable, against innocent recipients too), and every protocol-level fix hands somebody else the power to move claims ([§18](money-from-first-principles.md#18-limits)) | the loss lands on whoever lost the key; wallets carry the mitigations |
| Manipulated external reference | prefer self-measuring anchors; a payout naming a reference declares it, so the exposure is priced at acceptance (invariant 3's `referenced` qualifier) | the reference sits outside the protocol, and a backing that reads one imports its manipulation surface |
| Sovereign hostility | censorship-resistant base; jurisdictional spread | real, and only mitigated |

## C5. Deploying it

Nothing needs an institution before the first transaction. Every historical attempt failed on circulation rather than issuance. The order:

1. **Unit first.** Declare the basket, name the references. Established businesses quote in it while the old currency circulates.
2. **One acceptance anchor.** The grocer, the employer, the utility, whoever collects the local equivalent of taxes. Then widen what it accepts.
3. **Roots before guarantees.** Plain root claims are the liquid objects. Cover appears where acceptance needs crossing, and not before.
4. **One template.** A single canon setting carrying nearly all volume, with `closure` used for every requirement. Exotics wait for a reason to exist, since liquidity lives on promises that need no reading.
5. **One shared sequencer**, serving every backing. Start transparent for build simplicity, or in the shielded setting wherever who borrowed is more sensitive than the cryptographic cost, paying in proofs on the spender's device. Invariant 10 holds at any size in either setting, and the trade is build cost against holder liveness, which does not reverse ([§13](money-from-first-principles.md#13-what-it-takes-to-run)).

    The setting sits in **E**, hence in the hash, so sign the *next* backing shielded when a float is large enough for unlinkability to matter, since a successor has to be bought and cover will not follow the swap ([§9](money-from-first-principles.md#9-the-law)). Starting transparent costs nothing in metric quality. Each backing can name a different operator later, and sharing concentrates the deployment's regulatory exposure into one identifiable party ([§18](money-from-first-principles.md#18-limits)).
6. **Matching from day one.** A person or bot whose only job is closing loops. The search problem [§16](money-from-first-principles.md#16-what-emerges) predicts arrives later.

**The wallet runs alongside from step 2**, scoped by the steps above: no closure, no cover, no upward sum, no stopping problem. A balance, a send button, a policy list. The terminal that [§15](money-from-first-principles.md#15-valuation) and [§16](money-from-first-principles.md#16-what-emerges) describe arrives with the exotics that need it.

**Wallets must be open source**, and the user should see none of the machinery ([§20](money-from-first-principles.md#20-in-practice) says why).

---

## Appendix: Alternatives and what they cost

Each is expressible, or nearly so, and attractive from inside an implementation.

**A payout that reads what the holder presents.** Presentability, completability and assemblability stop being definable. [§8](money-from-first-principles.md#8-what-must-come-with-it).

**Either/or in reliance.** Price stays ambiguous until somebody chooses. [§8](money-from-first-principles.md#8-what-must-come-with-it).

**Reach-through reliance.** Forbids a backer asking for less, and the closure macro recovers the safety. [§8b](money-from-first-principles.md#8b-closure-and-what-r-costs).

**Merging requirements at a diamond.** Leaves the backer holding one member with no partner. [§8b](money-from-first-principles.md#8b-closure-and-what-r-costs).

**Fewer than four fields.** Without **K**, identical promises collide. Without **P**, no promise. Without **R**, every guarantee is pre-funded, killing the elasticity that makes cover worth writing. Without **E**, nobody can tell whether a claim is unspent, or what time a payout reads. Pushing state-dependence into **R** fails separately, since a field fixed at signing cannot carry a schedule.

**Reliance without a count.** Assumes every promise in a chain shares one scale, which is false once two backers choose different payouts.

**Fractional coefficients in reliance.** Equivalent to whole counts wherever claims divide, since the scaled form costs nothing there. Wherever claims quantize, a coefficient p/q maps on-grid holdings off-grid except at multiples of q: presentation locks into blocks, and every split and change-making step that must hit the ratio rounds and sheds a residue. Whole counts keep the entitlement test exact at every quantity and park the fraction in P, where it rounds once per settlement under the declared rule, in the holder's favour.

**Separate fields for maturity, interest, demurrage, triggers, capacity.** They are names without structure. [§5](money-from-first-principles.md#5-what-it-pays).

**A payout that reads the backer's whole book.** It is state you could not have priced at acceptance. [§5](money-from-first-principles.md#5-what-it-pays).

**A payout that degrades softly past a ceiling.** Allowed under invariant 19's conditions and not otherwise, with collapse at the ceiling as the fallback. [§5](money-from-first-principles.md#5-what-it-pays).

**A separate unit field beside the payout.** One field factored two ways, inviting a par that does not exist. [§5](money-from-first-principles.md#5-what-it-pays).

**A unit-over-time field instead of a payout expression** (`U` diverging from a reference unit, claims counting `U`). For constants and time functions it is P relabelled, and a unit allowed to read state is the payout language under a new name. What it costs on its own terms: units are shared and schedules are per-backing, so fusing them mints a private unit per schedule and kills the common measure. And the state-reading forms, the credit-event trigger, pro-rata under a coverage floor, the anti-squeeze read, have no unit-over-time expression at all.

**Wrapping a fully collateralised asset in a claim.** Adds an attester where none was needed, and creates no money. [§10](money-from-first-principles.md#10-roots-grounding-and-hardness).

**Mixing custody and ordering.** Imports the weaknesses of both ends. [Appendix B](money-from-first-principles.md#on-chain-backing-and-custody).

**A global ledger for ordering.** Buys nothing the per-backing sequencer does not, and costs a metadata trail on every payment. [§13](money-from-first-principles.md#13-what-it-takes-to-run).

**Public issuance as a supply guarantee.** Bounds what a backer admits issuing, not what its attester signs. [§13](money-from-first-principles.md#13-what-it-takes-to-run).

**Classical mutual credit as a configuration.** Heavy machinery for what a plain ledger does with none, and it socialises losses. [Appendix B](money-from-first-principles.md#mutual-credit).

**An instrument as a first-class object.** A note is a set that happens to be closed, and the object reintroduces the appraisal unit the design dissolves.

**Unifying E's attester with P's named references.** They differ in kind. E's output is a predicate about one claim, a reference supplies a quantity, and only E carries a silence clause, since a missing price has a fallback while a missing attester strands transfers.

**Recasting the object as an offer** (`in = R` plus one unit of itself, `out = P`). It is elegant, and it costs the distinction between what a presenter surrenders, the claim as authorisation and R as subrogation. It also obscures the paper's strongest unification: an issuer is a guarantor with R empty.

**Naming a backing as (K, hash(P, R, E)).** Both disambiguate, and the single hash makes a name one opaque value everywhere. It is a convention rather than a necessity.
