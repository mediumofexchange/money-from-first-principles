# Money from First Principles

### One promise, and every money built from it

**Abstract.** Almost all money is somebody's promise. This paper derives the smallest object that can carry one. It is a signed promise with four fields: who owes, what it pays, what must be handed over with it, and who says a claim against it has not already been spent. One law governs who may sign what. Each step repairs a failure in the one before. Out of the object and the law come rules that merchant law took five centuries to find, banks and market makers as roles rather than licences, and a scale of hardness with trustless assets at one end. The aim is money creation anyone can contest.

---

# Part I: Why

## 1. What is actually wrong

Money creation today is a licensed activity. Most new money is created as bank credit, and the licence to write it is valuable and politically defended. Four consequences follow.

**Distribution.** New money enters the economy next to the act of creation. Cantillon's argument, made in the 1730s, is that whoever stands nearest benefits most, and wage earners receive it last, at higher prices.

**Exclusion.** Well over a billion people sit outside the system, unbanked. Their trust networks carry real credit every day, and most of it never travels beyond the two people who agreed it.

**Fragility of the unit.** When a national currency fails, it takes the economy's credit with it, because the debt was written in that currency. The trust between the parties outlives the unit.

**Socialised failure.** The losses of the licensed system land on people who signed nothing, through bailouts and inflation. And holding the unit is not optional.

There are two usual answers. Fix the institution, which keeps the franchise. Or escape it with money fully backed by scarcity, which builds something real and stays small, because it creates almost no money.

This paper takes a third route and designs a **grammar** instead of a money. One object, general enough for fiat, a bank deposit, a bill of exchange, a stablecoin and a neighbour's word. Then it asks what happens when writing in it needs no permission.

Concentration will not disappear, and [§16](#16-what-emerges) argues it is overdetermined. What goes is the legal moat. Entry is free, terms are public, and anyone whose promise is better can undercut an incumbent. A village, a supply chain or a trade association can have its own money alongside the established one.

## 2. What we are designing

Money is usually defined by three functions, and two of them come apart under inspection.

A **unit of account** is a unit and nothing more. Nothing ties it to the thing being passed around, and prices have often been quoted in one thing and settled in another. Brazil ended hyperinflation in 1994 with the URV, a pure unit of account, quoted for months before any matching cash existed. [§12](#12-units-of-account-and-prices) says where a shared unit comes from.

A **store of value** is a property of whatever a holder decides to keep. A system that lets people choose what they hold has done the job. And historically the medium itself was rarely a good long-term store of value. What a medium does need is to hold value between receipt and spending.

That leaves a **medium of exchange that is widely accepted**: something a receiver can verify cheaply, that a holder can pass on at a cost small against the transactions it serves, that divides and survives well enough for them, and whose units are either comparable, or cheap to tell apart and to put a price on.

One further requirement is a judgement call, and it rules out most of what we call money today. The system should work for ordinary people where public infrastructure and institutions cannot be relied on: where courts are captured, where the currency is dying, where the bank will not open an account. That means access without permission, resistance to censorship, and privacy for spenders. It is an expensive choice, and [§14](#14-privacy-and-disclosure) states the cost.

The design works out one asymmetry. The person who owes must be known, since their promise is what everyone else has to price. The person who holds need not be, since holding obliges them to nobody.

> **You can spend anonymously. You cannot owe unprovably.**

---

# Part II: The derivation

Each step below starts from a failure in the one before, adds the smallest repair, and states its cost. What was chosen rather than forced is said out loud. Nearly every term introduced along the way is also listed in the [Glossary](#glossary).

## 3. A promise between two people

Nadia is a carpenter. She needs timber now and will be paid in six weeks. Oskar runs the grocery. Nadia gives Oskar a signed note: *I owe you the value of forty kilos of flour, redeemable against my work or my goods.* Oskar accepts it because he knows and trusts Nadia.

The note settles a debt the way money does, by transferring a claim instead of a thing. It is the oldest form we have evidence for. Under [§2](#2-what-we-are-designing)'s definition it is not money yet, since only one person takes it. Innes argued in 1913 that credit comes before coin, and Graeber's survey a century later found the pattern almost everywhere.

**Cost:** a signature.

**What comes free.** Mutual obligations cancel down to the overlap, with no settlement, no asset and no third party. And creation costs nothing but credibility. Nadia's signature made purchasing power out of her reputation, which is what a bank does with a licence.

**What fails.** The arrangement stops at two people. Oskar cannot spend Nadia's note.

## 4. Letting it pass

So let the note pass. **A promise becomes money the moment its holder values it for what it can buy rather than what it redeems for.** Two things follow.

The terms must be **published**, so that someone who has never met the backer can look them up. Terms nobody can read are terms nobody can price.

And the claim must be **bearer** rather than registered, which is the choice [§2](#2-what-we-are-designing) already made. Registered claims circulate perfectly well. Bank deposits are registered and circulate at civilisational scale, and registration solves double-spending for free, because the obligor's ledger says who holds what. What bearer buys is transfer that needs nobody's permission, and a holder nobody has to identify. It costs four things: an online uniqueness service ([§13](#13-what-it-takes-to-run)'s sequencer), no clawback, key loss as total loss, and, where no public trail records the count, a supply nobody outside can check ([§13](#13-what-it-takes-to-run)).

**What fails.** A stranger still cannot price the promise.

> When Ireland's main banks closed for six and a half months in 1970, the economy ran on personal cheques and shopkeepers became the credit assessors. But a cheque travelled only where its drawer was known. A worker paid by cheque could rarely pass it on, so they wrote their own.

## 5. What it pays

A stranger holding Nadia's note needs published terms he can compute. Call the published terms a **backing**. Two fields come first: who owes, and what one claim pays.

> **K**, the *obligor*: the verification key that owes. It sits inside the signed terms, and therefore inside the name, or two keys writing identical terms would collide. K may be a threshold key.
>
> **P**, the *payout*: what one unit pays, a quantity of a named thing. In the core it is a function of witnessed time alone; [Extensions](extensions.md) widens the argument list.

Most promises need no function. "One kilo of flour" is complete. The field is a function because nearly every mechanism people treat as separate is a shape of it:

| Usually called | The payout is |
|---|---|
| Ordinary backing | constant |
| A dated bill | zero after a date |
| A loan, deferred payment | zero before a date ([§17](#17-every-money-is-a-setting) works it) |
| Interest, demurrage (interest's mirror: a fee for holding) | rises on a schedule, or discounts a fixed payout |
| Partial cover | a fixed fraction of another backing's schedule, copied at signing |
| Conditional backing (a CDS) | zero until a declared condition |
| Reserve policy; ceiling, workout | reads declared reserve coverage, or its own outstanding where provable |
| Escrow release, parametric insurance | reads a named external reference |
| Pooled cover | reads a named list of the backer's other backings |

The first five rows are arithmetic over time. The rest read live state, and are built in [Extensions](extensions.md).

In the core, the payout reads one thing:

> **P : witnessed time → quantity of a named thing**

That is an arithmetic expression over the witnessed clock and nothing else. Payouts that read live state are real and useful: the backer's own outstanding, other backings' totals, external references. Every one of them drags in machinery the core does not need, such as evaluation instants, reference hygiene and monotonicity checks. [Extensions](extensions.md) adds them one need at a time, without touching the object.

The time-only core is larger than it looks, because a guarantee can track the promise it stands behind without reading any state at all. [§8b](#8b-closure-and-what-r-costs) shows how, once there is a guarantee to write.

**Time is read at presentation, never from a claim's age.** A claim carries no timestamp. Accruing from an issue date would need a separate denomination for every issue batch, and that shatters the anonymity set. So interest, where written, is global to the backing.

**The named thing must be something a holder can take away.** Whether a promise rests on somebody's liability depends on what it pays in ([§10](#10-roots-grounding-and-hardness)), and a unit of account is defined by whoever redeems in it ([§12](#12-units-of-account-and-prices)). Scale is arbitrary: multiply the thing by λ and divide the schedule by λ, and nothing real has changed. So any measure of manipulability has to survive rescaling.

**There is no par**, no face value separate from what the claim pays. Nine-tenths of a kilo is the promise, not a shortfall of one-tenth. Scale lives in the payout alone, because requirements count whole units ([§8](#8-what-must-come-with-it)).

**No payout shape prevents over-issuance**, which is what justifies keeping the core this small. Say the pool holds 100 kilos and the backer has written 200 claims at a kilo each. It can pay 100, because 100 is all there is, whatever the schedule says. Past that ceiling every shape pays out the same total.

What a shape does decide is who gets paid in a default. A constant payout pays the fastest presenters in full and the slow nothing, which is the definition of a run. The [appendix](#appendix-a-the-arithmetic) carries the arithmetic. Shapes that act on this, pro-rata workouts and declining ceilings, need the payout to read a count that can be trusted, and [Extensions](extensions.md) states the exact conditions. So what the core defers is a choice about allocation, priced in advance.

**What fails.** The merchant can now read the promise. He still cannot tell whether the claim in his hand was already spent.

## 6. Proving it has not been spent

A signature proves a claim authentic. It does not prove it unspent, because bytes can be copied. Somebody has to put transfers in order and mark claims consumed.

**The smallest answer is to ask the obligor.** The receiver already trusts the backer to pay, so trusting the backer to say "unspent" adds no new dependency. Two things do have to be added. A stranger must know in advance who to ask, and the backer must be bound to its own answers. So the rule travels inside the promise.

> **E**, the *evidence*: who attests that a claim is unspent.

E grows more parts later: an interval, a venue, a replacement rule, a silence clause, a claim-layer setting and its construction. Each answers a failure that has not yet appeared in this derivation, so [§13](#13-what-it-takes-to-run) adds each one where it is earned.

**E is also the clock.** The payout reads time, and the only time everyone agrees on is the witnessed one. So what a promise pays depends on the venue.

**Evidence prices double-spending; it does not prevent it.** A payee may accept a claim without fresh evidence, at a discount, the way a shopkeeper takes a cheque. That is what Ireland ran on. The claim is not locked out afterwards. Nothing moved as far as the attester is concerned, so the payee fetches fresh evidence whenever convenient, and the claim circulates normally from there. The discount prices the window until then, in which the payer could still spend it first. Evidence is a service whose absence has a price, and that keeps hard availability off the critical path.

**What fails.** The merchant can read the promise and check the claim. But he does not know Nadia, and will not take her paper at any fair price.

## 7. Getting a stranger to take it

Two things get a promise past someone who does not know the promiser: a sink on the path, and a guarantee.

**A sink on the path** needs nothing new. Somebody further along will take the claim in settlement of something already owed to them: the employer, the utility, the tax collector.

> **A sink on the path is sufficient for a claim to circulate, and it is the only route open to a promise with no guarantor.**

Redemption at the backer is the universal sink. An **acceptance anchor** is a backer whose sink is wide, and widening it is the highest-return action any deployment can take. Prices get quoted in whatever settles rent, wages and taxes, so the widest sink also ends up setting the unit of account ([§16](#16-what-emerges) seats the same force).

**A guarantee** is older. Hale & Co, a wholesaler everyone deals with, writes their own promise beside Nadia's: *bring me this note and I will pay you, and afterwards I will collect from Nadia.* Now the merchant will take it, because he trusts Hale. This is endorsement, and it carried world trade from the seventeenth century to the nineteenth. A guarantee written over somebody else's promise is called **cover** from here on, and whoever writes it is the **guarantor**.

Hale has two ways to do it, and only one needs anything added to the object.

**Funded:** Hale buys Nadia's claims and issues their own against them. No new field. Hale is a root ([§10](#10-roots-grounding-and-hardness)) holding a portfolio, which is a bank, at the cost of a reserve.

**Unfunded:** Hale promises without funding it first, and gets their claim on Nadia only at the moment they pay. This is the cheap, elastic, historically dominant form, and it is what the third field buys.

Two features are easy to lose when you work top down. Nobody assigns Hale: it chooses to guarantee, for a fee. And Hale's promise names something Nadia's does not, which is what must be handed over along with it.

## 8. What must come with it

Take the unfunded case. Hale pays the merchant, and must end up holding a claim against Nadia, or they have simply given money away. The protocol cannot mint that claim, because minting it would increase Nadia's liability without her signature ([§9](#9-the-law)). And Hale cannot count on the merchant happening to hold one. So Hale's promise says up front what must be handed over.

> **R**, the *reliance*: a fixed list of pairs (backing or chain asset, count), naming how many **units** of each must accompany one unit of this claim at presentation. Counts are whole numbers. The list may be empty.

R is the holder's burden. To be paid you hand over your claim and the listed units, and the backer keeps the set.

The count is in units because packing a quantity into claims is the holder's choice, and a requirement should not depend on it. Claims split and merge without moving quantity, so a requirement counted in claims could be met by splitting, and would measure nothing.

A unit here is the backing's own count: the quantity that issuance creates and burning destroys ([§9](#9-the-law)). A claim of quantity *q* holds *q* units and pays *q·P*. It is not the unit of account the backing pays in. Those are two different measures, and P is the bridge between them. A unit whose payout has fallen to zero still exists, and still satisfies the count.

Whole counts keep presentation in integer arithmetic and give every bargain one canonical form. To write a fractional ratio, rescale this backing's own payout at signing. To require half a Nadia unit per unit, double the payout, require one whole unit, and issue half as many units. The bargain is unchanged in aggregate. Fractional coefficients add nothing this rescaling does not, and they cost exact assembly wherever claims quantize ([Construction appendix](construction.md#appendix-alternatives-and-what-they-cost)).

**A backing with a non-empty reliance is a standing bid for a bundle**: bring me these things together and I pay you this much. Compare the free version. An open offer to buy Nadia claims at 0.9 kg is real cover, but it is free to every holder, earns no fee, and is rationed first-come. **R turns the open offer into a rationed, transferable, priced ticket.** The Hale claim is the right of access to Hale's bid. Subrogation, the old rule that whoever pays a guaranteed debt takes over the claim, is the same fact from the guarantor's side: the payer takes the set.

Presentability is then one line:

> A holding **H** is **presentable at *b* for *q*** if it contains *q* units of *b* and *q·cᵢ* units of each *(bᵢ, cᵢ)* in **R(b)**.

The field is scoped tightly, and each restriction prevents a specific failure.

**Reliance names backings and chain assets only**, things a wallet can hold and a protocol can check. So presentability stays decidable and the exchange can be atomic. A promise to deliver grain enters as a backing that pays grain. Nothing is lost but the pretence that the protocol was reasoning about grain.

**The list is fixed at signing.** Assembly takes time and presentation is atomic. If the requirement could move while you gathered the set, you could arrive complete as of yesterday and short as of today. Freezing terms at the moment of demand would also remove that race, and [§13](#13-what-it-takes-to-run) does exactly that for refusals, so this is a choice. It buys presentment against terms fixed in advance, and a requirement anyone can work out beforehand. The general rule the design keeps returning to:

> **Requirements are structural, fixed, and machine-verifiable. Payouts are functional, may float, and settle outside the protocol.**

The same split is why **the payout may not read what the holder presents**. "0.9 if accompanied by a Nadia claim, else 0" would fold both mechanisms into one, and the price is decidability: presentability computed by an arbitrary function cannot be worked out before you acquire anything. The blindness runs the other way too, and the requirement never reads what anything pays. One N unit satisfies the count whatever N pays that day, including nothing, so a promise outliving its reliance's zero-date has quietly become unconditional.

**The list is a conjunction.** An either/or requirement is not fixed until somebody chooses, which turns presentability into a search and price into a guess. Little is lost. Where a requirement genuinely must branch, a capped root that pays nothing builds the branch, one backing per option. A cap is a declared issuance ceiling, checked rather than enforced: issuance is logged in both core settings, so a wallet reads the log at acceptance and prices claims past the cap at zero. The cap is a norm everyone can check against the log, a pattern [§9](#9-the-law) returns to. Worked small: to stand behind a hundred sets against N or M, issue a hundred G, then write one cover requiring {G:1, N:1} and another requiring {G:1, M:1}, neither capped itself. The G cap bounds the pair together, so the holder picks the branch and the writer's total stays a hundred.

**Reliance reaches one level**: a requirement names backings, never the requirements of those backings in turn. Nothing is hidden by stopping there, since terms are immutable and hash-named, so anyone can compute the full chain at signing. And stopping is deliberate. A backer who wants to require the whole chain writes it out, while one who means to sell the set it takes in wants less, which automatic reach-through would forbid.

Two words carry the next steps. A **hash** is a short fingerprint computed from a document: nobody can work backwards from fingerprint to document, and no document can contain its own fingerprint. A **chain** is a public record that many strangers keep in agreement, so no one party can rewrite it.

Cycles need no rule against them. For A to require B while B requires A, each backing is named by the hash of its own terms, so each name would have to contain the other, and such a pair cannot be constructed. And a requirement may only name backings whose terms are published, because an unreadable promise cannot be priced.

> The object is complete: **B = (K, P, R, E)**, signed, public, immutable, named by the hash of its four fields.

## 8b. Closure, and what R costs

The full chain under a requirement is its **closure**. Closures are written with a `closure(S)` macro, expanded before hashing ([Construction §C0](construction.md#c0-invariants)), and counts add up where paths meet. If *b* relies on *x* and *y*, and each of those relies on *z* at 1, then closure({x:1, y:1}) = {x:1, y:1, z:2}. Two units of *z*, because presenting at *x* alone would leave *y* without the *z* it needs. Multiplicities grow like a matrix power, so wallets cap closure size.

Anyone can compute **closed(b)**, whether R(b) equals its own closure. An unclosed requirement is readable: the backer takes in a set it cannot fully unwind, usually because it means to sell it.

Three consequences follow. **Redeemability is a property of a wallet rather than an instrument**, and what commerce called a note is a set that happens to be closed. **Fragments cannot inflate**, since an unaccompanied claim is inert, and splitting a set distributes claims without creating any. **Failure is granular**: a failed backer's claims reprice, whatever required them reprices with them, and the rest of the graph is untouched.

**Cover can track what it covers without reading any state.** What a backing pays and what it takes in at redemption are independent fields, so a guarantor may promise flat, or on any schedule it likes. One template is the useful default. The underlying's terms are immutable and named by hash, so the writer copies its schedule into their own expression at signing. Hale's cover on Nadia copies hers: `P_H = 0.9 · P_N`, still plain arithmetic over time.

That is a copy made once, and not a live link. After signing, nothing that happens to Nadia moves it. A flat 0.9 kg would drift from any schedule that moves, and then the writer owes the excess with no underlying to cover it, or the holder's cover thins. Copying keeps the difference between the two promises a matter of credit alone. Which part to copy is a choice. Copy the whole curve, and the cover goes to zero when the underlying's schedule does. Copy one point, as [§17](#17-every-money-is-a-setting)'s bill does with the number owed at maturity, and the cover picks its own expiry.

**Read the bill beside the purchase.** R buys unfunded cover. It also buys fragments, closure, the completability machinery ([§11](#11-redemption)), the stranding externality ([§16](#16-what-emerges)), and a liquidity tax, since standard closed sets carry nearly all volume. A backer with R empty has none of those costs, which is why [Construction §C5](construction.md#c5-deploying-it) ships roots first.

The field is worth its price for two reasons. Unfunded cover is what bills of exchange did for five centuries. And it is how the design reaches the excluded: a promise need not circulate itself, it only needs to be the reliance of one that does. The village needs one member whose paper travels, not a village of them.

**A ring of mutual acceptance** is [§7](#7-getting-a-stranger-to-take-it)'s sink route written down, with members taking each other's paper. It cannot be written directly, because each member's terms would name the other's backing, so each hash would have to contain the other. Instead each member signs a root Aᵢ, plus one **acceptance backing** per counterparty, paying what Aᵢ pays and requiring one Aⱼ. The number issued is the size of the line.

Acceptance written into reliance is pre-sized, and the difference matters. A **sink** is revocable, unbounded, revolving, and worth nothing to strangers. A **reliance** is irrevocable, finite, non-revolving, and it travels. Re-extending an exhausted line is a fresh issuance decision, and transferability is what the grammar buys with that.

The mesh takes *n(n−1)* acceptance backings, because each one names who is accepted. A hub makes it *2n*, on top of each member's root, so **O(n²) becomes O(n)**. That arithmetic is what produced correspondent banking, and under a hub the hub's claims become the circle's settlement asset.

## 9. The law

Two keys can act; a third can only get in the way.

> **No act increases what a backer owes (meaning its written maximum) except one signed by that backer.**
> **No act reduces what a holder holds except one signed by that holder.**

The second line is phrased over holdings because there is no "move" operation. A transfer is void-and-reissue, and the replacement claim is controlled by a key that is not the old holder's. One further rule makes the first line enforceable: **a backing exists only with a valid signature by K over its own name.** Otherwise anyone could publish terms naming your key as the obligor.

Three roles fall out.

- A **backer** key signs backings and issues claims. Issuing is the only act that increases what is outstanding.
- A **holder** key transfers, burns and presents. None of these increases anything.
- A **sequencer** key originates nothing. Its only unilateral choices are refusing, or lying about order.

Everything usually written as a fairness rule follows from the two lines and the hash-named terms.

**Nobody can be made a backer.** Only the backer's key writes the backer's fields, so a new promise anywhere in the graph costs existing backers nothing, and handing someone a guarantee is as ordinary as handing them a coin.

**Nobody's terms change underneath them.** A requirement names a backing by hash, so what you accepted is what you hold. A backer who wants new terms publishes a **successor**: a different backing with a standing offer to swap. Note what this costs. A cover holder refuses even a strictly better successor, because their cover names the old hash, and swapping the underlying would strand it ([§16](#16-what-emerges)). The same mechanism that stops terms changing underneath you is the one that stops improvements reaching everyone. **Protocol upgrades are successors too, bought rather than imposed.** No flag day.

**Sequencers cannot issue.** A sequencer voids a claim and reissues an equivalent at the holder's request. If it could create claims, it would be enlarging the backer's liability without the backer's signature. Issuance moves the outstanding count and needs the backer. Reissuance preserves the count and needs only the holder.

**Valid claims cannot be clawed back.** A stolen key is a real injury, and the loss lands on whoever lost it, with remedies running against the thief at law, and, where the claim is not negotiable, at law against innocent recipients too. Money you can reclaim from innocent recipients is not money, because every acceptance would then mean investigating the whole provenance.

**Conservation.** One key increases and every other act preserves or decreases, so at any published moment:

> **outstanding = issued − burned**

Counted in claim quantity, never in tokens and never in value paid. Burning is a holder destroying a claim, for good. Redemption has no separate term, because presenting hands the claims to the backer, who may burn or respend them. The identity is exact in quantity and only a bound on liability, since lost keys and backer-held claims are outstanding but owed to nobody.

**And it is a norm before it is a fact.** Under blinding, only the attesting signer can count valid signatures. So whether the identity is enforced or merely asserted is a setting each backing declares ([§13](#13-what-it-takes-to-run)), and everything computed from outstanding totals inherits the answer. A sharper number is available whenever a backer wants it. Circulating is outstanding minus what the backer holds. Outside transparent the netted figure is a disclosure rather than a proof ([§16](#16-what-emerges)).

One principle generates most of the above:

> **Any exposure a holder cannot compute from published data at the moment of acceptance is worth zero to them.**

It is why the payout's argument list is fixed, why reliance cannot move, why evidence declares its machinery in advance, and why there is no edit operation.

**The law binds two parties at a time, and the graph ties strangers together.** Every externality in this design lives in that gap. P may name backings and references, R backings and chain assets, E an attester and a venue. So a holder's redemption strands cover holders, a successor squeezes what cover requires, and a payout reading a named total moves when that backer signs. No rule is broken, because both parties to each claim signed everything. [§16](#16-what-emerges) works through the consequences; the cause is here.

**What this protects is terms, not value.** The law forbids loss you could not have read in advance, and nothing else. Bad promises stay legal. They are only made visibly bad.

## 10. Roots, grounding and hardness

Reliance may be empty, and that case is where value comes to rest.

> A backing with **R = ∅** requires nothing alongside it. Call it a **root**.

Empty reliance is the only difference between an issuer and a guarantor. Both sign a promise and create money in the act. One of them takes something in at presentation.

A root is not necessarily the bottom. Reliance is about what accompanies a claim, not what it is denominated in, and a dollar-denominated root still rests on the Fed.

> A root pays in something held up by an issuer, a compelled sink, or convention. Call a root **grounded** where following what it pays in, down the chain, ends in something that is nobody's liability.

Some things are nobody's promise: a chain's native asset held on its own chain, a commodity in your hand. Both change hands directly, and when they do, nobody owes anybody anything. Such a thing is **not a backing at all**, because a backing exists only with its obligor's signature, and there is nobody to sign. It is a **nameable term**: a payout may pay in it, a reliance may require it, and hardness is measured against it. There is no successor, since there are no terms a signature could replace. And wrapping it is pointless, because there is no backing to wrap, only a custodian's promise standing where the asset was.

The hard corner of the monetary field is where every source of trust has been driven to zero. Six coordinates, computable from public data wherever the money is issued inside this grammar, fix what hard means:

| | Coordinate | Chain asset | Fiat | Bank deposit | Stablecoin | A neighbour's word |
|---|---|---|---|---|---|---|
| **Beneath it** | 1. Reliance depth, layers of requirement under one claim (a root is 0) | n/a | 0 | 0 | 0 | 0 |
| | 2. What the unit is | itself | itself | the state's | someone else's | whatever they say |
| | 3. What holds it up | convention | a compelled sink (taxes) | the unit's issuer | the unit's issuer | the speaker |
| **Can still move** | 4. What the payout reads | n/a | undeclared | undeclared | declared reserve | undeclared |
| | 5. Supply enforceable as issued, which needs a chain ([§13](#13-what-it-takes-to-run)) | yes | no | no | no | no |
| | 6. History pinned outside the attester | yes | yes\* | yes\* | yes\* | yes\* |

\* if issued inside this grammar; otherwise unmeasurable.

Rows 2 and 3 catch what depth misses. Fiat and a chain asset both pay in themselves, and they are still not the same object. Fiat rests on a sink somebody is compelled to honour, and a chain asset on convention nobody can be made to hold. Row 4 is the channel [§5](#5-what-it-pays) closes, row 5 the one this design pays most for, and the whole hard-money argument lives in rows 3 to 5.

Row 6 asks whether whoever attests unspentness could also rewrite the record. A history witnessed to a venue outside the attester ([§13](#13-what-it-takes-to-run)) is one it cannot edit without proving itself false, and an informal promise outside the grammar has no record at all.

The coordinates are not summed into one hardness score. A sum needs a weight on each, and weights are a judgement anyone can dispute. So hardness stays six numbers rather than one.

The grammar covers the trustless case, which is why hardness is one comparison. A chain already does for its native asset what a sequencer does for a claim: consensus orders transfers and proves them unspent. Put an operator in front of it anyway, a custodian issuing wrapped claims, and you add an interval in which unspentness rests on somebody's word, and you create no money. Hold such assets directly, in the same wallet. When trust contracts, value moves toward the corner: a flight to quality inside the system instead of a run out of it.

## 11. Redemption

To redeem, a holder presents claims together with everything the terms require alongside. The backer pays and takes the whole set: its own claims to burn or respend, and the remainder to keep as ordinary claims against what it required. Recovery walks down the chain like this and stops at a root, which requires nothing.

Numbers make it concrete. Nadia is a root paying one kilo of flour. Hale pays 0.9 and requires one Nadia claim per unit. You hold 40 of each. Redeeming at Hale hands over 40 H and 40 N for 36 kg, and Hale keeps the N to collect from Nadia. Redeeming at Nadia instead yields 40 kg, and leaves the H inert until more N arrives.

**A refusal burns nothing.** [§13](#13-what-it-takes-to-run) settles a presentment on two signatures, so claims are voided only when the holder signs the release. Present at Nadia, be refused, and you still hold everything.

That turns redemption into a sequential strategy, and probing the root first wins for every guarantee whose payout is a haircut on the underlying's ([appendix](#appendix-a-the-arithmetic)). Probing costs little: ask, be refused, walk away holding everything, at worst a swap to refresh an aborted note under blinding. The cost arrives with the demand, filed where the refusal must go on record ([§13](#13-what-it-takes-to-run)). A demand commits the claims it names until withdrawal or settlement, and cover is inert without the claims it requires. So a standing demand at the root freezes the whole position while cover decays and other holders draw the pool down. Hence:

> **Go to the root first, unless what decays during the window you must lock costs more than the chance the root simply pays.**

Under the trigger extension ([Extensions](extensions.md)), a guarantee that pays nothing until unanswered demands reach a declared share of the root's outstanding at the cover's signing is armed by the refusal record rather than by the holder's own timing. A holder whose claims fall short of that share still faces this section's ordering problem.

Everything one would write as a rule here is a consequence. Recovery terminates, because roots require nothing. Release is free, since dropping a claim is burning it. Cover follows its underlying, because a redeeming holder hands over the set. Mutual claims cancel with no clearing house, since each party burns what it holds.

That last one needs two things: atomicity, because burning first is a gift, and payouts you can compare, because netting flour against hours needs a rate. Which is the same limit redemption itself has. **The protocol cannot make an exchange atomic where payment is in kind.** It does not enforce payment. It makes refusal legible, and [§13](#13-what-it-takes-to-run) is how.

**Two public ratios price the graph.** Two formulas, three published numbers: completability, computed gross and net, and load; unitload waits for [Extensions](extensions.md). Neither formula is a field. Both are computed from published outstanding totals, and each answers one question a stranger has.

The first: of the claims this backer wrote, what fraction could be presented at all right now? For a backing *b* requiring *(bᵢ, cᵢ)*:

> **completability(b)** = min( 1, ( minᵢ outstanding(bᵢ)/cᵢ ) ÷ outstanding(b) ),  min over ∅ = ∞; a backing with nothing outstanding scores 1

That is the fraction of *b*'s claims that can presently be assembled into presentable sets. The cap and the empty-set convention keep it bounded and score roots at 1. The minimum ranges over required backings only: chain-asset legs sit outside the ratio, so a backing requiring only chain assets scores 1, and the ratio is silent about them. A guarantor who writes more cover than its underlying supports falls below 1 in public.

Compute it twice. Net of the writer's own holdings it is **assemblability**, which tells a holder whether they can complete a set. Gross, it tells you whether the writer is covered, since holding your own requirement is what makes a guarantor good ([Construction](construction.md#the-design-in-one-page) carries the netting caveats). The ratio treats each backing as if it were first in line for what it requires. Whether others are competing for the same claims is what the next number measures.

The second: how much has been written over a given thing? For any backing *z*:

> **load(z)** = ( Σ_b outstanding(b) · c_z(b) ) ÷ outstanding(z), over every *b* that requires *z*

That is the *z* units all promises over *z* would consume, against how many exist. Above 1, more has been written over *z* than *z* carries. The simple sum understates, because contingent demand runs through the whole closure, so the closure-weighted sum is the published bound. Load and unitload are undefined over zero outstanding.

Requiring *z* is not the only way to rest on it. Three roots, A paying a gram of gold, B paying one A claim, C paying one B claim, read flat on both numbers while being three layers on one gram. The chain of what each backing pays in, the denomination chain, records that layering. A third sum, **unitload**, walks the chain and measures it, so three layers on one gram read as three. It needs an evaluation convention for floating payouts, so [Extensions](extensions.md) defines it.

Both are ceilings rather than forecasts, and neither reads as a bare scalar. Issuance needs no permission, so a stranger can publish a junk backing requiring a lot of *z* and drive the numbers up. Summaries must be scoped to backings a policy accepts, and wallets compete on the scoping. The upward sums are assembled by a reader watching witness venues ([§13](#13-what-it-takes-to-run)), exact over what it can enumerate and a floor beyond it. What escapes is a backer spread across venues nobody indexes ([Construction §C4](construction.md#c4-threat-model)).

## 12. Units of account and prices

Every backing declares its own unit of account: dollars, gold, kilograms of rice, hours, or the backer's own product. The last is often the best available. The waterworks can promise cubic metres at the meter, and no price feed is needed where redemption is the definition. **A unit of account is defined wherever some backer redeems in kind at a fixed quantity.** Price feeds are prosthetics that extend a unit's reach beyond the anchor that defines it.

Where a machine must act, a feed is unavoidable, manipulating it is a real attack, and [Extensions](extensions.md#external-references)'s hygiene rule applies. Everywhere else, backer and redeemer settle at whatever price both consider honest, and a backer who settles at a self-serving price reprices all of its own claims.

Baskets tracking local prices are the neutral default. A unit that inflates against them shortchanges the lender, and one that deflates buries the borrower. This matters most where national units are dying, because credit dies with the unit it was written in. Decoupling lets a trust network keep lending in something stable, so the trust outlives the currency. The Besançon fairs cleared much of Europe in the *écu de marc*, a unit the bankers defined themselves, with no state behind it.

## 13. What it takes to run

Three problems stand between the object and a working money, and none needs an institution.

**Transfers need ordering.** A **sequencer** serves a backing, and one operator typically runs many. It holds no funds. It co-signs transfers the holder authorised, and refuses a second spend by declining to sign, so a fresh co-signature attests a claim unspent now. Holding no funds is what lets a sequencer be shared or crowd-funded, so issuing needs nobody's own machine. A global ledger buys nothing extra, because what matters is **substitutability**: any backing can name a different operator without asking the rest ([Construction §C2](construction.md#c2-sequencing)).

Each sequencer publishes a commitment at a declared **witness interval**, to a venue named in **E** beside the attester. That way it cannot show different histories to different people, and a stranger knows where to look. The venue is where [§11](#11-redemption)'s upward sums get assembled, and venue and attester move only under **E**'s replacement rule, declared in advance.

The sequencer failing earns a **silence clause** with two grades. First, transfer requests left unserved past a declared duration, measured on service, since a staller publishes on time. That grade fires on a count of requests, which **E** declares. Second, commitments stopping altogether, after which redemption against the last witnessed snapshot opens without co-signature. [Construction §C2b](construction.md#c2b-failure-silence-and-recovery) sets the durations and the checks.

**Supply is not enforceable outside a chain.** Two words are doing different work here. Supply is **enforceable** where the protocol itself prevents overissuance, which needs a chain, whether the asset is native or a contract's issuance ([§10](#10-roots-grounding-and-hardness)'s row 5). It is **checkable** where anyone can verify the count after the fact. Outside a chain the core cannot offer the first, and both settings below offer the second.

The core answers with two honest settings, declared in **E**, inside the hash, so a holder knows the setting before accepting. Which one, and which construction of it, is each backing's priced choice; [Construction §C5](construction.md#c5-deploying-it) carries the defaults.

**Transparent: exact supply, no unlinkability.** A public ledger of key-controlled balances. Transfer still needs nobody's permission, has no clawback, and identifies a key rather than a person, so it is bearer in [§4](#4-letting-it-pass)'s sense. It gives up unlinkability, and with it resistance to a refusal aimed at one person, a resistance blinding itself supplies only together with an anonymising transport. This is the honest default at the scale [§20](#20-in-practice) starts at, and the setting a lender wants.

**Shielded: unlinkable, checkable at the boundary.** Claims live in a shielded pool or accumulator, with histories hidden and, under the pooled construction, amounts too. Under pooled, spending publishes a nullifier, a one-time tag marking the claim spent without revealing which claim it was, and a zero-knowledge proof that value is conserved; the accumulator counts per denomination instead. Supply stays checkable because every spend carries that proof and the pool's boundary is lit: every entry a logged issuance, every exit a published burn, and redemption an internal transfer to the backer. It costs a proof on the spender's device, no spending without resync, and verification standing in for the arithmetic a transparent ledger gives away free.

A third profile, Chaumian blind signatures with denomination keys, is what existing mints run. It trades checkable supply for the signer's assertion, and [Extensions](extensions.md) carries it as a compatibility profile.

**And the setting propagates.** A metric is only as good as the weakest setting in what it reads, so a backing's metric quality is the minimum over its reliance closure and denomination chain, fixed at signing since every member is named by hash. The honest claim is modest: where every member of a closure sits in a checkable setting, the numbers over it are real, and a wallet can tell you which holdings qualify. The grammar itself does not notice the setting, and object, law, presentability and closure are identical across all of them. The whole difference fits inside one field.

**Refusals need recording**, and nobody paid by the backer can be trusted to record them. So a holder publishes a **demand** naming the backing, the quantity, the claims offered, the instant its payout is read at, and a deadline. It stands, with the claims committed against payment, until withdrawal or settlement. Settlement takes two signatures, acceptance and release, so neither party can write the other's outcome, and claims still live past the deadline are the backer's visible failure. Nobody reports anything. Whoever's backing requires the refusing one is next in line, and watching. [Construction §C3](construction.md#c3-presentation-and-dishonour) carries the mechanics.

A demand and its silence is evidence rather than proof, and notarial protest proved only presentment and refusal, never ability to pay. A false demand is restrained by presentment being signed and attributable. Payouts that *read* the refusal record, turning an unanswered share of demands into a credit event a guarantee can pay on, are the trigger extension, and the latching and denominator rules that make them safe live with it ([Extensions](extensions.md)).

## 14. Privacy and disclosure

One principle governs identity: **it attaches where someone must be findable afterwards, to be collected from or to be paid, and nowhere else.** Signing a backing and presenting for payment are identified acts. Holding and spending are net-zero and stay dark. An identity is a key with a public history, and binding it to a legal person is optional and priced like everything else.

Claims are bearer, and where the setting blinds, every transfer is void-and-reissue with a fresh secret, so from the second hop a claim cannot be linked cryptographically to its past. That part is solved in the cryptography rather than in the network. A void and its reissue are one round trip to one operator, so address and timing relink them unless the transport hides it.

What remains is metadata, and metadata is where the whole battle is, because the design's own virtues generate it. Heterogeneous money means the crowd you hide in is the holders of this backing, not the whole system. A payment from a portfolio spends specific backings in specific proportions, and the combination fingerprints the payer. The endpoints are lit by design: issuance is logged, and redemption identifies you, because the moment you ask to be paid you are someone. And a published demand names a backing, a quantity and specific claims, so recourse is bought with privacy.

The claim layer is built against that list, not only against linkability ([§13](#13-what-it-takes-to-run), [Construction §C1](construction.md#c1-claims-and-wallets)). Under the pooled construction amounts are hidden too, so quantities leave no denomination trail. And under the pooled construction the anonymity set is **per operator**: one operator serves many backings in one shielded pool, per-backing supply is still proven at the pool's lit boundary, so the crowd is the operator's whole clientele, and a multi-backing payment is one shielded transaction instead of several correlated ones.

The dealers [§16](#16-what-emerges) grows are also a privacy service nobody has to build. Sell to a dealer unlinkably at bid, the dealer presents in bulk under its own name, and a retail holder never touches the lit end at all.

Issuance, the deliberate exception, publishes less than it appears to need. The count requires the **quantity**, not the recipient, so the log entry is *(backing, quantity, time)* and the first holder is blinded by default. Only the transparent setting cannot omit the name, and a backer lending to a named borrower can declare identified issuance. Publishing who received credit would harm exactly the population [§20](#20-in-practice) targets, and the conservation argument does not need the name.

**The honest ceiling.** Privacy has increasing returns, and the cold start has none of it. This begins in a village, where the crowd to hide in is so small that everyone is identifiable, and no construction helps. At scale the floor is set by what must stay lit: identified issuance where a lender wants it, identification at final redemption, and the demand you publish when you exercise recourse. Everything between those points can be dark, and the design should be judged on whether it keeps it dark, not on the endpoints it deliberately lights.

**On the lit side, a disclosure is worth what it puts at risk.** Zero-knowledge proofs let a backer prove coverage above a ratio without revealing a single item, so coverage over the committed items is proven while the books stay dark. The strongest disclosure is not a statement at all but a **standing bid on the backer's own claims**, signed, sized and dated, which cannot be inflated without being taken up. Disclosure becomes a priced spectrum, with an executable bid at the top, itemised books next, and silence near zero.

## 15. Valuation

Appraising unfamiliar promises by hand means reading them, asking around and guessing. It works, it is miserable, and it has been fatal.

> In the American free-banking era, merchants consulted printed *banknote reporters* listing the going discount on hundreds of issuers' notes. Gorton found new banks discounted more heavily than known ones, with precision improving as the telegraph and railroad spread. What ended the era was a federal tax, not wildcatting.

Appraisal has two halves, and software does different things to each. The **structural** half is what the terms say, what they require, how deep they sit, how much has been written over them. It is removed outright, because every input is public and a wallet computes it in milliseconds. The **credit** half is the probability the backer pays, and it stays what it always was: local, expensive, a judgement.

What changes is visibility. Writing a promise that takes in Nadia claims is a position nobody can hold without publishing. The graph publishes the exposure, and never the price at which it was taken: who stands behind Nadia, and for how much, is public, while the fee that changed hands is not. So the graph does not value Nadia. What it prices is everyone above her, since whoever holds a writer's promises can read what that writer loses if she fails.

**A policy is a wallet's acceptance and pricing rules**, written by the holder or bought ready-made. It says which obligors and backings the wallet will take, what discount to apply for dishonour where no maker's bid supplies one, which backers it treats as failing together, and what shape of portfolio to hold. Policies do the reading, and verification is **per claim**, so acceptance degrades gracefully. An unverifiable claim counts for zero to the policy that cannot check it, and the set is worth whatever the checkable remainder supports.

What a wallet computes is not simply what it could collect. A holding's value is the best of three options: redeem, carrying both outcomes since a refusal burns nothing; sell at bid; or hold, which is the discounted value of deciding again later. Surrender the set unconditionally and the refusal branch drops out ([appendix](#appendix-a-the-arithmetic)).

This is an optimal stopping problem, and the hold branch is the interesting one, because a claim can be worth more than anything it will ever pay you, for what it lets someone else complete. Wallets approximate it, and compete on how well.

One identification falls out. An option is the right, bought for a fee, to demand something later at terms fixed now, worth nothing until the thing moves your way. A guarantor's payout is normally the underlying's minus a fee, so **a guarantee is an exchange option on the root's impairment**: worth little while the root is sound, armed as it decays, with the fee setting the moneyness. A wallet, in turn, is a credit portfolio, and value is a property of the portfolio before it is a property of any claim. Two promises that fail together are worth less side by side than either suggests alone, completion value pushes the other way, and pricing both is the wallet's job. Backers who share a locality fail together, and credit information is local, so portfolios concentrate exactly where correlations are highest. The goal is failures that stay local, since failures that never happen are not on offer.

**Imported opinions need pricing too, and the regress ends in a price.** The only rating you can trust mechanically is a standing bid on the claims themselves: signed, size-limited, dated, from a maker with a record of honouring them. Everything short of that is talk, since an opinion nobody paid to hold costs nothing to be wrong about. Market makers emerge to supply the bid, and most holders run a default policy and glance at the result.

Above the bid sits the reliance graph, which is exposure data rather than opinion, on three conditions. The backing sits in a checkable setting, transparent or shielded ([§13](#13-what-it-takes-to-run)), since under the Chaumian profile the totals are the signer's assertion. The obligor signs with one key, which nothing compels and only acceptance rewards. And a reader maintains an index from obligors to backings, scoped to what a policy accepts.

Where all three hold, exposure is public **while it accumulates**: who has sold protection, and for how much. It is a credit register rather than a rating agency. A backer who does not pay is information. Their claims reprice, distressed claims travel toward whoever collects most cheaply, and anyone may write fresh cover over them, which is credit insurance from the same primitive.

## What was chosen

Each choice with what it buys and forecloses. Everything else in the derivation was forced, and the argument said why.

| Choice | Buys | Forecloses |
|---|---|---|
| Bearer, fungible-quantity claims ([§4](#4-letting-it-pass)) | permissionless transfer, unidentified holders, appraisal per obligor | clawback, per-vintage terms, par |
| R fixed at signing; conjunction, whole counts, one level ([§8](#8-what-must-come-with-it)) | presentment against pre-fixed terms; unambiguous price, integer arithmetic, requiring less than the closure | terms that freeze at demand; disjunctive recourse, fractional coefficients, protocol-computed recourse |
| Payout blind to the presented set ([§8](#8-what-must-come-with-it)) | presentability decidable in advance | one mechanism doing two jobs |
| Per-backing sequencer ([§13](#13-what-it-takes-to-run)) | substitutability (no operator is mandatory), no global metadata trail | shared liveness, single-round payments across operators, a guaranteed system-wide anonymity set |
| Time-only payouts in the core ([§5](#5-what-it-pays)) | a payout language that is plain arithmetic | state-reading terms, until Extensions |

---

# Part III: What follows

The claims carry different weights. [§16](#16-what-emerges) predicts what a system not yet built would grow. [§17](#17-every-money-is-a-setting) and [Appendix B](#appendix-b-the-field-worked) show the object covers the field, checkable today. And [§18](#18-limits)'s merchant-law table is retrodiction.

## 16. What emerges

None of this is designed in. The rows carry four different weights, and the table says which. **Forced** means it follows once anyone at all acts on the posted terms. **Premise** means it needs a stated premise about who keeps acting. **Conjecture** means the timing claim is argued below and not established. **Computed** means the object makes something visible that was always there, which is coverage rather than emergence.

### What appears

| What appears | Forced by | Grade | Its name today |
|---|---|---|---|
| Root claims carry a **distress premium** | fragments need root claims | forced | convenience yield, flight to quality |
| **Cover crystallises together** | one public exercise condition | conjecture | monoline failure |
| A **reassembly market**, then a **bank** | inventory plus a two-way quote is a spread | premise | market making, banking |
| **Increasing returns to sameness** | liquidity lives on promises that need no reading | premise | the note; the eligible-asset list; the regulator, as a role |
| **Payment routing** | claims circulate where sinks lie; hub arithmetic ([§8b](#8b-closure-and-what-r-costs)) | forced | correspondent banking |
| **Credit insurance** | backward separability: cover written later, by a stranger, over circulating claims ([§21](#21-what-is-new-here-and-what-is-not)) | premise | the CDS market |
| **Leverage visible in three places** | reliance in the graph, denomination in the chain, reserve in the reference | computed | shadow banking, fractional reserve |
| **Backings concentrate** | anonymity sets, templates, hubs, adverse selection in cover-writing | premise | — |
| A **private lender of last resort** | backward separability, wide acceptance | premise | the lender of last resort |

Four rows stand on the middle column and are not worked further: increasing returns to sameness, payment routing, credit insurance, backings concentrating, and leverage visible in three places. The rest are below.

### The run on completability

**Completability is a rival resource, and the run on it stops at a public ceiling.** It is rival because every act that consumes root claims, or writes more cover over them, cuts everyone else's odds of completing a set. No intent is needed. Stranding, cover decay, over-issuance dilution and rival bidders for the same *z* are one fact seen from four sides. **load(z) above 1 is the oversubscription measure.**

Who gains differs across the four, and that decides which of them stop by themselves.

| The side | Who it pays | Does it stop by itself? |
|---|---|---|
| Stranding | nobody; the writer, where it buys its own requirement | ordinary redemption pays nobody and keeps producing it; a writer's buy-back is paid its own liability and shows only in assemblability |
| Cover decay | the writer, its fee | yes, at expiry |
| Over-issuance dilution | the writer, at issuance | no |
| The price contest for *z* | whoever sells the root claim | yes, when cover goes out of the money |

As *z* deteriorates, cover holders bid for *z* claims to complete their sets, and none pays more than their cover's payout times its credit, divided by its requirement. One bidder outbids them all: the writer itself, whose ceiling is the full payout it avoids owing, payable in its own fresh paper, and whose purchase extinguishes its own cover. So the ceiling is nominal, quoted in the writer's own paper, largest exactly when the writer's credit is worst; the market's discount of that paper is the only bound, and it is thinnest in a panic. Nothing moves but assemblability, and only transparent proves the netted input, though a writer can declare a lit inventory key, making the netted figure proved rather than disclosed. So one public condition, cover in the money, opens the run and caps it. What lifts the cap above the underlying's own payout is cover paying **more** per unit of requirement than the underlying: a leveraged bet on impairment, and no structural ceiling bounds the multiple.

Timing matters too. Cover holders exercise at different times only because they disagree about two probabilities, the root's failure and the writer's. Narrow that disagreement and every in-the-money holder exercises in the same week, against inventory the writer does not hold, which is the monoline pattern. Dispersion of belief is the one damper this design generates rather than declares. Whether it converges away as public numbers come to dominate is a conjecture, and nothing here establishes it.

### How the rest grow

**A root's issuance moves its guarantors' books.** [§9](#9-the-law)'s successor squeeze is one direction. The sharper one: Hale writes a thousand units of cover over a Nadia with a hundred outstanding, Nadia issues nine hundred more, and every unit goes live. Liability moved tenfold on Nadia's signature, and no rule was broken.

The default is visibility. A guarantor's **max liability = assemblability · circulating · payout** is published beside its terms. How much of that number is provable depends on the setting. Under transparent it is exact, since the writer's own holdings are public. Elsewhere the netted input is the writer's own account of what it holds, so the figure is a disclosure rather than a proof. Cumulative payout is unbounded either way, since a promise to burn is a covenant, and over-issuing cover is the mirror image [§5](#5-what-it-pays)'s payout analysis never sees ([Construction §C4](construction.md#c4-threat-model)).

**The reassembly market grows into a bank.** Inventory and a two-way quote are a dealer. Fund the inventory with its own undated root and it is a bank, [§7](#7-getting-a-stranger-to-take-it)'s funded guarantor, costing nothing but credibility. That is maturity transformation, and its run is [§5](#5-what-it-pays)'s racing analysis with an undeclared pool. It is visible here because a refused holder's demand is a public fact every wallet prices.

**A money hierarchy is derived rather than assumed.** Cover requires root claims, so completing anything consumes them. The root is the settlement asset of everything written over it, which is why load(z) reads both as leverage borne and as breadth of settlement demand. The denomination chain records the same hierarchy where load reads zero, and [§17](#17-every-money-is-a-setting)'s loan is that case worked. The seat is contestable and never empty, decided by acceptance rather than charter, and published.

**Concentration produces a lender of last resort, on one premise the rest do not need: the rescuer stays accepted.** Fresh cover over a distressed backing is money to the extent the writer's own promises are accepted. A wide sink is what a panic does not immediately revoke, since the rent, wages and taxes it settles still fall due. So a wide-sink backer can stand behind failing promises, funding it by issuance, while its own published numbers move against it. Whether acceptance outruns visibly rising liability is the crisis question, and nothing here answers it. It is bounded as no central bank is: the rescued are named by hash, and the rescuer's exposure moves in public.

**The concentration forces would operate under perfectly distributed trust**, and adding a money beside the incumbent survives them. The widest sink concentrates the **unit of account**, and a second money need not be a second unit. Denominated in the incumbent's, a local money adds credit without contesting the quoting convention, and its niche is the local credit knowledge an incumbent cannot price. WIR has held that configuration since 1934.

## 17. Every money is a setting

Any money that is somebody's promise is a setting: a root, an acceptance anchor, a unit, terms. What is not somebody's promise is [§10](#10-roots-grounding-and-hardness)'s limit case, held directly in the same wallet.

### The plane

Two questions sort the monetary field. Does reputation or collateral stand behind the promise? And does one hand run it, or many?

```
                 REPUTATION                  COLLATERAL
              ┌───────────────────────────┬───────────────────────────┐
 CENTRALISED  │ fiat, bank deposits       │ currency boards,          │
              │                           │ custodial stablecoins     │
              │ elastic and universal;    │                           │
              │ Cantillon effects, opaque │ cannot be debased,        │
              │ books, socialised losses  │ cannot breathe            │
              ├───────────────────────────┼───────────────────────────┤
 DECENTRAL-   │ individual credit,        │ heavily collateralised    │
 ISED         │ enterprise money, bills   │ promises, DeFi lending    │
              │                           │                           │
              │ contracts in downturns,   │ nearly trustless and      │
              │ transmits contagion       │ censorship-proof, but     │
              │                           │ creates almost no money,  │
              │                           │ liquidates into crashes   │
              └───────────────────────────┴───────────────────────────┘
```

The grammar writes every cell. Decentralised reputation is the corner it makes buildable, and a deployment sits wherever its holders choose. A backing's fields place it anywhere on the plane, and each move trades one failure mode for another. Move toward collateral and you buy trustlessness and pay in elasticity and liquidity. Move toward reputation and you buy money creation and pay in cyclicality. Move toward a single operator and you buy convenience and pay in that operator's failure modes.

**The wallet is what makes the plane usable.** A wallet is a set of claims and any bare assets held alongside them, so it is a portfolio: it holds positions at several points at once, and it moves between them as trust grows in one place and dies in another. The holder decides what they value and where the risk sits. The grammar's job is to keep every position priced.

### The settings table

| Money | Root | Anchor | Unit of account | Trust |
|---|---|---|---|---|
| Fiat | the state | taxes | its own | centralised, reputation |
| Bank deposit | a licensed bank | legal tender, insurance | the state's | centralised, backstopped |
| Stablecoin, custodial | a company | redemption promise | someone else's | centralised, collateral |
| Stablecoin, algorithmic | a contract | arbitrage against a reference | a reference | reflexive: why they fail |
| Bill of exchange | the acceptor | settlement at maturity | the trade currency | decentralised, reputation |
| Enterprise money | a business | its own shelves | national, or its product | decentralised, reputation |
| Portfolio money | custodians, one per asset | redemption at each provider | the holder's own portfolio | centralised, collateral |
| Individual credit | a person | their word and goods | any | far decentralised |
| Chain asset | n/a | none needed | itself | none: the limit case |
| On-chain backing | a contract | redemption from the contract | anything it can verify | code, inelastic for that reason |

A gift card is enterprise money with the terms left revocable. The mutual credit circle has no row, because only its outward edge is a setting and the ledger inside it is not; [Appendix B](#mutual-credit) works through why. That appendix also works fiat, enterprise money, on-chain backing and portfolio money, and adds real-world assets and the monies nobody has run. Two templates stay here, because the rest of the argument leans on them.

### The loan

The loan comes first, because it is the case [§1](#1-what-is-actually-wrong) is about.

> Nadia publishes a root **N1** paying one claim of anchor A, zero before month six, expiring at month nine. Oskar buys 100 units for 96 A claims. At month six whoever holds N1 presents, and Nadia delivers A claims she has since earned.

As written, that is intermediation: existing claims moved, and the medium did not grow. The variant that answers [§1](#1-what-is-actually-wrong) is Nadia paying the timber merchant in N1 directly, at a discount the merchant sets. The newly signed liability *is* the medium. Money is lent into existence by a signature, exactly where [§7](#7-getting-a-stranger-to-take-it)'s acceptance problem bites. Interest is the discount, and A is untouched if Nadia fails.

Note what the metrics see. N1's reliance is empty, so it scores flat on every reliance metric, while it **pays in** A. That is fractional reserve banking, promising in something you do not hold in full, written with R left empty and recorded in the denomination chain.

### The bill of exchange, with recourse

The second template needs nothing but time:

```
root    N        dated: pays 1 kg from maturity T until zero-date Z
                 (T, when it starts paying; Z, after which it pays nothing)
cover   H₁…Hₙ    each requires 1 N and pays fᵢ · (N's payout at T, a
                 constant inlined from N's published terms), from T
                 until its own date D, with T < D < Z
```

The cover reads nothing at runtime. N's terms cannot change and are hashed, so each writer copies the number N owes at maturity into its own schedule at signing. Dating the cover is structural: D sits before Z so a writer who has paid, and now holds the surrendered N, still has time to present it, and the dates form a chain [Construction §C2](construction.md#c2-sequencing) sets out.

Play it through. The holder presents at N at maturity. On refusal they publish the demand, and once its deadline passes unanswered, withdraw it and present at the best H, surrendering one N. H takes the N and presents before Z. Presentment, protest, recourse and subrogation, in three fields, with the refusal on public record ([§13](#13-what-it-takes-to-run)).

One honest cost. Written this way, the cover can be exercised whenever it is in the money, so it is priced as an option and costs more than the historical bill. The trigger extension, cover that pays only once unanswered demands on the root reach a declared share, is what converts the option back into an affordable guarantee, and it is the first extension scale will summon. Its one open flank is priced in its fee: the root's own obligor can write the credit event against itself ([Extensions](extensions.md)).

**The cover is parallel, not chained, so the holder chooses freely.** Each Hᵢ requires the same single N unit, and presenting at one makes the others inert. Written in series, each requiring the one before, you get the opposite instrument: recourse down the chain, no choice. The historical bill welded both, every signer severally liable to the holder and each payer recovering up the chain, and it armed only on dishonour, with notice, which is what made endorsement cheap. Here the menu and the chain are two different ways to write R, and the arming is the trigger extension. Contribution among co-sureties is inapposite rather than lost: no two writers here ever answer for one debt.

### It reaches the instruments too

Anything that is a promise with declared terms, optional collateral, and consent on both sides can be written here, which covers nearly all of finance.

**The system is already an options market.** A promise at a partial rate with a maturity is a put on what it requires, written by a defaultable writer. The strike is the payout, and the premium is what the cover claim costs. [§15](#15-valuation) prices the same thing from the holder's side. Backing always was option-writing, and [§18](#18-limits)'s regulatory reading follows from it, since writing puts at scale, to the public, for fees, is derivatives dealing in most countries.

The rest are exercises in the same fields: calls, credit default swaps, factoring, repo, syndicates and tranches, securitisation waterfalls, FX forwards. Each is a choice of payout and reliance, with collateral where the parties want it, and the conditional ones use the state-reading payouts of [Extensions](extensions.md). Nothing needs an instrument type, an indenture, or a venue.

## 18. Limits

### What the grammar cannot say

A grammar that expresses anything explains nothing. Six things do not fit.

- **Anything reading private state.** Indemnity reads *your* loss; equity reads the issuer's residual. A promise only the payer can compute is one nobody else can price. Only parametric proxies fit.
- **Covenants.** A backing promises to pay. Obligations to behave have no expression, except that an issuance ceiling is a payout. Anything addressed to a court rather than to the payout, a forum clause, a waiver, a promise to write cover later, is a covenant. The mutual credit circle breaks on exactly this ([Appendix B](#mutual-credit)).
- **Transfer restrictions.** Claims are bearer, so lock-ups and holder limits have no form, which is deliberate, per [§2](#2-what-we-are-designing).
- **Priority across strangers.** Nothing orders different backers' creditors, so insolvency is per-backing inside the protocol. That is what makes disjunctive recourse unwritable here, which [§21](#21-what-is-new-here-and-what-is-not) works through against ChainCash.
- **Authority moving a position.** Bail-in by decree, porting, retroactive repricing: *unilateral* discretion after the fact. Delegated discretion can be written, because a payout may read a named arbitrator's published determination, priced by every holder before accepting under the hygiene rule in [Extensions](extensions.md#external-references).
- **An objective determination of default.** The nearest thing, a published demand unanswered past its deadline, is checkable by anyone and still a weak proxy, since a backer may be sound and slow, or refuse one holder and pay the rest.

The nearest substitute is a parameter rather than a judgement. The trigger extension lets cover pay once unanswered demands reach a declared share, past a declared answer window, with both terms the writer picks and the buyer prices ([Extensions](extensions.md)). Real markets determine credit events with committees, which is discretion after the fact, the thing this design excludes. **This is the clearest case of the design paying for its own central commitment rather than getting it free.**

### The law we did not write

For five centuries, merchants with no common sovereign developed the *lex mercatoria*, later absorbed into English and American commercial law. Most of it is about **what a court will do**. This design determines only **what terms can say**, so each old rule lands somewhere different. One sentence governs where: **this design substitutes pricing for enforcement**, making promises legible and continuously repriced while doing nothing to make them kept.

| Rule | At law | Here |
|---|---|---|
| No transaction-specific defences against a holder | holder in due course, UCC §§3-302, 3-305, minus §3-305(a)(1)'s real defences; §3-104 gates Article 3 on a tangible writing and a fixed amount of money, so most of this object fails it twice over | **Unavailable**: the protocol does not enforce payment, so the defence is raised outside it: at law, where a court sees an assigned contract right, and the curing waiver is a covenant |
| A guarantor who pays inherits the claim | subrogation | **By agreement**: reliance buys subrogation's transfer of the claim, without the surety's reimbursement or contribution; the payer takes the set |
| Bearer title passes by delivery; an electronic record is possessable under singular control | negotiability; MLETR, UK ETDA 2023 | **By construction**, and stronger: a holding changes only by the holder's key, with no good-faith test, since the grammar cannot express a state of mind; void-and-reissue is singular control almost verbatim. The control regimes are gateways, open only where the backing is already a qualifying instrument, such as a note under BEA s.83; UCC §12-105, adopted state by state, names the cryptographic key outright, and the ETDA's reliable-system test is technology-neutral enough to accept one |

The rules explained away are as informative as those recovered. *Present to the acceptor before the endorsers* is not a duty here but optimal play ([§11](#11-redemption)). *Present promptly or lose recourse* has no analogue, since the clock goes in the payout and a dated guarantee expires by arithmetic. And the excluded ingredient is discretion after the fact, not loss allocation: a payout running pro-rata below a declared ratio *is* a bail-in, writable because every holder priced it before accepting.

### What the design cannot fix

**This design reopens the decentralised-reputation quadrant** of [§17](#17-every-money-is-a-setting)'s plane, and its worst property is the **contraction**: money becomes least available when it is most needed, panic erodes trust, and trust is the collateral. load(z) is the cycle's state variable.

Its twin is **contagion**. When the house of de Neufville failed in 1763, liability cascaded through every chain of endorsed bills it had signed, toppling merchants who had never dealt with it. That is exactly this design's characteristic failure. What differs is the shape: exposure is confined to named backings and repriced continuously, and visibility is the whole defence, since supply is not capped.

The collateral column fails its own way. In March 2023, USDC's transparency accelerated its own run, and [§5](#5-what-it-pays)'s pro-rata answer defeats the race where the pool depletes as it pays, and still does not bind the issuer. On Black Thursday 2020, MakerDAO showed chain assets falling exactly when trust contracts, and its liquidation path failing with them. So the trust layer must carry the elastic load. And **no damper here is demonstrably counter-cyclical**. The nearest exception is [§16](#16-what-emerges)'s belief dispersion, which thins the onset and may converge away by the time the crisis arrives.

Three costs are structural.

**The freedom to build bad money** is real: legible and priced, which is better than invisible, and still not prevention.

**The key is the balance.** Loss, theft and death are one problem in bearer money, mitigated by the wallet and untouchable by the protocol, since every fix hands someone else the power to move your claims.

**The obligor holds a veto over transfers** wherever it runs its own sequencer. A wholesale stall makes claims illiquid while the backer never refuses a payment ([Construction §C2b](construction.md#c2b-failure-silence-and-recovery)). The remedies are partial. The silence clause fires only after its declared durations, and an independent operator is a term of the backing, named at signing and priced at acceptance, never something a holder can impose later. A backer who wants a future keeps its own sequencer honest, and the veto bites exactly where that incentive has died. Under Chaumian a stall cannot even be shown.

### The political risk dominates the rest

A signed promise for value is a contract almost everywhere, so these promises bind by default. Binding whom is the harder question. A downstream holder may have to rely on the published terms as an offer to holders, unless the claim is negotiable, which UCC §3-104 mostly denies it, or validly assigned. In England an assignee takes subject to prior equities either way; s.136 governs only suing in one's own name, and wants written notice a bearer claim rarely supplies.

The real exposure is regulatory, and it is graduated by what an activity looks like. Holding is barely regulated, except that, where cover reads as a swap, a retail holder of it is a prohibited counterparty in the US. A root redeemed in its issuer's own goods is a prepayment at worst, stored value where the claims transfer and strangers accept them. A dated root paying in someone else's claims, sold at a discount, may be a security, past the short-paper exemptions. An undated root funding a portfolio out of received funds, which [§16](#16-what-emerges) derives rather than permits, is deposit-taking or e-money issuance, the most tightly licensed activities there are, and in the US the GENIUS Act now reserves payment-stablecoin issuance to permitted issuers, a bar rather than a licence, biting by January 2027 at the latest. Cover at scale is derivatives dealing. In the EU a claim that promises stability against a basket is an asset-referenced token under MiCA, and a claim merely denominated in [§12](#12-units-of-account-and-prices)'s default unit sits at that line. Running a sequencer risks transfer-service rules, most under Chaumian, though an operator that never touches value has a real exemption argument. Jurisdictions differ on every line. In the US, issuing and redeeming transferable value is money transmission before it is anything else, the hook, beside money-laundering charges, that ended e-gold.

And all of it assumes institutions that tolerate competing money, which historically they often have not. e-gold processed billions before prosecution ended it, and Wörgl was shut down while it was working. So censorship resistance is load-bearing rather than ideological, and the legal and survival strategies are one: serve people the incumbent is not serving, keep the base layer seizure-resistant, spread across jurisdictions.

## 19. Scoring the four charges

[§1](#1-what-is-actually-wrong) laid four charges.

**Fragility of the unit: answered**, where a self-measuring anchor exists. The trust outlives the currency.

**Exclusion: answered**, by the smaller claim in [§8b](#8b-closure-and-what-r-costs): a promise need not circulate, only be relied on by one that does. The limit is that the credit half of appraisal stays local.

**Socialised failure: mostly answered.** Failure is compartmentalised and priced before the fact. Contagion is bounded rather than absent. And a widely accepted anchor's failure is socialised in practice whatever its terms say.

**Distribution: traded, not solved.** New money still enters next to somebody, now whoever is trusted rather than whoever is licensed. The rent, on an undated root, is the claims never presented plus their interest-free use. Whether free entry competes that down to the credit spread needs the rent and the credit spread told apart, and no published data can separate them. So it is a different distribution, not the absence of one, and not one the design can check.

---

# Part IV: Putting it in the world

## 20. In practice

**Nothing needs an institution before the first transaction**: two parties and one promise are a valid system. None of this has been built, and everything below is a build order for a system that does not yet exist. Launch as digital IOUs and let complexity arrive with demand. The on-ramp is a promise denominated in the national unit, backed by someone with a bank account.

**It starts in a village rather than a market.** The regress of [§15](#15-valuation) ends only in each holder's own knowledge, so the cold start is social: a trade association, a supply chain, a diaspora corridor, somewhere a ring of trust exists and is unusable as money. That is a limitation, and also a targeting instruction, since it points at the population the licensed system serves last.

**Circulation does not arrive by itself**, and this is where deployments die. Widening what the anchor accepts comes first. Then **matching**, since at this size loops do not exist yet, and someone must notice that what one member buys outside, another sells. Wörgl closed its loops through the town's own taxes, Sardex through paid brokers. Deployments with neither mostly failed.

**The user sees none of this.** The wallet is where the design becomes money: one balance in a chosen unit, a portfolio underneath, payments routed through the acceptance graph, cover solicited when a payment needs a threshold crossed. And **wallets have to be open source**, since the dominant wallet's defaults are the canon, and a canon nobody can read is the discretionary authority this design exists to remove. Until the wallet shows a number, this is a settlement system for specialists.

One week makes it concrete. A nurse is paid Friday in her hospital's root, which the pharmacy and her landlord accept. Rent goes out Monday in the anchor claims the landlord prefers, and the wallet swaps at the going discount. On Wednesday a roofer she has never met will not take hospital paper, so the wallet solicits cover from a wholesaler both of them know, pays the fee, and the roofer accepts the covered set. On Thursday a backing from an old employer reprices after a public demand goes unanswered, and the wallet moves it out of the spending set and offers it to a collector at bid. She saw one balance move four times.

[Construction §C5](construction.md#c5-deploying-it) gives the launch order, and its appendix lists every alternative refused.

## 21. What is new here, and what is not

Almost every layer has been built before. Credit as money's base case is Innes and Graeber. Trust-graph money is Fugger's original Ripple, routing payments through bilateral credit lines; this design makes promises bearer and composable. The claim layer is Chaumian e-cash. Competing private issue is Hayek and the free-banking literature. A haircut cover is subordination, so the securitisation reading of [§17](#17-every-money-is-a-setting) is a translation. The leveraged shape [§16](#16-what-emerges) names is not, since it loses on a fully performing underlying, which no tranche does.

No single project is the nearest neighbour.

| Axis | Closest | Shares |
|---|---|---|
| The object | Ricardian contracts (Grigg, 1996) | signed, hash-named, human- and machine-readable terms |
| Multi-issuer bearer claims | Open Transactions | many issuers, blinded claims |
| Trust-graph composition | Ripple (Fugger, 2004) | bilateral credit lines as a configuration |
| Permissionless personal issuance | Circles | anyone issues; acceptance is a graph |
| Composition of backings | ChainCash and Basis (on Ergo) | implicit reliance order by signature position; on-chain reserves, off-chain debt |

A ChainCash note accumulates its holders' signatures, and redemption picks the earliest reserve with enough coverage. That is disjunctive recourse resolved by seniority, welded to the note, and it cannot be written here, where reliance is conjunction only and priority across strangers has no form. A holder gets the same choice only by holding the whole closure, which is the liquidity tax [§8b](#8b-closure-and-what-r-costs) priced.

Basis, its off-chain sibling, is the design nearest to [§13](#13-what-it-takes-to-run)'s service layer: a tracker that orders debt records and co-signs them while holding no funds, state committed to the chain at intervals, double redemption blocked by the reserve contract, and redemption against the on-chain reserve open when the tracker goes silent. That is the sequencer, the witness venue, and the snapshot path. What it does not carry is this paper's object. Basis records name a creditor and move by novation, with the debtor consenting to each transfer, where a claim here is bearer and moves on the holder's signature alone, and there is no published terms object. The gap is the claim layer rather than the plumbing, which makes it the natural place to build this.

The separability that matters is **backward**: cover can be written over claims **already circulating, by a stranger, without the underlying's consent, at a price set after the fact**. The aval came close, a stranger's guarantee written on a circulating bill, but it had to be written on the instrument itself; here the cover never touches the underlying and needs no possession of it. It makes a distressed backing insurable while its claims sit where they are, since whoever thinks the fear is overpriced writes cover and collects the fee. And it makes the reliance graph the credit register, since a rating that costs something to be wrong about is exactly a retrofitted claim. Its limit: retrofit cover inherits any reference risk in the root, so it concentrates on the simplest roots, another force toward standard templates.

**What is new is the grammar.** Redeemability belongs to a wallet rather than an instrument, which lets one primitive cover the field. Issuer and guarantor are one object, differing only in whether reliance is empty. Requirements are structural while payouts are functional. Hardness is measurable, since the trustless asset is the limit case. And a rating is a position rather than a publication.

The bill of exchange proved for five centuries that decentralised credit money works at civilisational scale. It lost to banks on appraisal cost, risk opacity and clearing latency. Software takes the structural half of appraisal to zero, makes written exposure public though never what a backer can pay, and turns clearing latency into a disclosure problem. What that reopens is free entry into banking, priced at the crisis toolkit: credit still contracts in panics, and promises still transmit failure. In exchange, a credit system sees its own leverage while it accumulates, and a trust network keeps working when its currency stops. A village, a supply chain, a city with a dying currency, or a single trusted grocer can make money that works, priced honestly, failing in compartments, and answerable to everyone who holds it.

---

# Appendix A: the arithmetic

**A1. Capacity, and the shape of a default** ([§5](#5-what-it-pays)). Pool *A* the backer cannot mint, nominal payout *P₀*, *n* claims written:

| | pro-rata `min(P₀, A/n)` | constant `P₀` | rising `P(n)` |
|---|---|---|---|
| Realised liability past the ceiling | `A` | `A` | `A` |
| Mean value per claim, over presentment order | `A/n` | `A/n` | `A/n` |
| **Allocation in default** | every holder gets `A/n` | the first `A/P₀` presented get `P₀`, the rest nothing | the first presented get `P(n)`, the rest nothing |

The apparent argument against own-count payouts computes liability from the written payout. But a backer who has written *n·P₀* against pool *A* can pay *A*, which is all it has, so no shape binds past capacity. What a shape decides is allocation. The constant payout has the same mean as the pro-rata one, paying the fastest in full, which makes racing rational.

Whether the pool depletes as it pays decides whether pro-rata defeats the race or rewards stalling. The worked batches, and the conditions a safe declining payout must meet, are in [Extensions](extensions.md) with the shapes that use them.

**A2. Sequential presentment** ([§11](#11-redemption)). Nadia paying with probability 0.5, Hale with 0.99 conditional on her refusal, at 0.9, 40 units held of each. Each branch carries the other backer as a fallback:

```
straight to Hale:   0.99 × 36 + 0.01 × (0.5 × 40)   = 35.84
try Nadia first:    0.5 × (40 + p) + 0.5 × 35.64    = 37.82 + 0.5p
```

where *p* is what the Hale claims fetch as a fragment if Nadia pays. Probing wins by 1.98 plus half the fragment value. Root-first beats cover-first whenever *P_N + p/(q·π_H) > P_H*. Where cover is a haircut, *P_H = f · P_N* with *f* below one, so absent carry cost root-first strictly dominates. Surrendering the set unconditionally at the root, dropping the refusal branch, returns 20 on the same numbers against 37.82.

---

# Appendix B: the field, worked

[§17](#17-every-money-is-a-setting) locates each money on the plane and works the loan and the bill. The rest are here, on the same parts. Each entry names the setting and says what the grammar adds to the form we already have.

## Fiat

**Fiat gains one thing inside this grammar: outstanding claims become public and priced continuously.** An audited printing press. Its unit ends at the tax office, where redemption is in kind and needs no price reference. Taxes are the strongest acceptance anchor there is, because one unit cancels one unit of a debt everybody has. On [§10](#10-roots-grounding-and-hardness)'s coordinates, fiat is a root that is not grounded. Add mandatory transparent holding and you have a CBDC.

## Enterprise money

A business as root, with its own shelves as the anchor. Gift cards and airline miles are the limiting cases: not transferable, and rewritten whenever the issuer likes. Here the terms lock and the claims transfer. What remains of the company-town trap is a discount the worker carries until a second place accepts the paper.

**Defining the unit in the product is better still.** A utility's kilowatt-hour promise, redeemed at the meter, measures itself, and for the utility's own customers it beats the national unit. The grocer, the employer and the utility that anchor any local economy are ready-made roots. E. C. Riegel made this case in the 1940s and never got it built.

## Mutual credit

A LETS circle keeps one ledger. When Nadia builds Oskar a counter, her balance rises and his falls, and each member promises to keep accepting members' work while their own balance stays inside a band.

Most of that fits here. A pre-sized acceptance line is an ordinary backing ([§8](#8-what-must-come-with-it)). The one part that does not is the promise to *keep accepting*. That is a promise about future behaviour, a covenant, and [§18](#18-limits) leaves covenants out. No field can hold it.

There is a substitute you can write, and it fails for a different reason. Let the circle issue a capped root G that pays nothing, and let each member write a backing requiring one G per unit ([§8](#8-what-must-come-with-it)'s branch construction, worked small there). Spending works. But after accepting, Nadia holds a used ration ticket rather than a claim on Oskar, so nobody in particular owes her anything.

**That difference is the real divide.** A shared-balance ledger socialises losses, since Oskar's negative balance is a claim on the whole membership. The grammar keeps losses in compartments, and Nadia's exposure to Oskar is capped at what Nadia chose to issue against him.

**A circle that wants shared balances should not force them into the object, because a plain ledger builds them simpler**: one balance per member, one tracker keeping it, which is Basis's design ([§21](#21-what-is-new-here-and-what-is-not)).

The grammar earns its cost at the circle's edge instead. The circle faces outward through one threshold key, as a single identified backer, and the claims issued under that key are ordinary backings, priced and traded with strangers and with other circles. The operator that keeps the internal ledger is the natural sequencer for those backings too, and issuance stays with the threshold key, since a sequencer never issues ([§9](#9-the-law)). A ledger inside, a backer outside: the grammar's edge runs through the circle, and to mutual credit the four-field object is a boundary object, met exactly where the grammar ends.

> WIR began as such a circle in 1934 and took a banking licence within two years. Its usage rose in recessions, when bank credit dried up.

## On-chain backing, and custody

On-chain backing takes the evidence field to its far end. The payout is code. What stands behind it is whatever claims and assets the contract holds. Redemption runs on presentation, and nobody's willingness enters anywhere. Every row in [§17](#17-every-money-is-a-setting)'s table has such a variant, written in the same four fields, so DeFi sits inside the grammar rather than beside it.

The cost comes in two currencies. Elasticity, since a contract pays only what was funded into it and no signature can create more. And privacy, since blinding needs a signer that cannot link what it signs, while a chain links everything.

**Custody decides which end a promise belongs on. Ask who can fail to pay, because that party is the thing being trusted.** A promise settled from a hard reserve is only as good as nobody being able to stand in front of that reserve, and what guarantees that is a contract on a chain. Once the reserve lives there, the chain may as well order the transfers too. A promise settled on somebody's word already trusts that person. Putting it on a chain buys certainty about transfer ordering and leaves the one thing that can fail, the person, untouched, at full price in lost privacy.

Each pure end is coherent. The middle usually combines the trust requirements of one with the stiffness of the other. The one exception is the shielded setting ([§13](#13-what-it-takes-to-run)), which buys a checkable supply without putting settlement on a chain.

## Real-world assets

**Real-world assets need no extension.** A signed promise for value is a contract wherever contracts are enforced. So a backing may pay in, or require, a deed, an invoice, a warehouse receipt or a bale of cotton, on the same terms as a kilo of flour, and issuance backed by collateral is then an ordinary requirement.

Real estate is the familiar case. A landlord's promise paying housing by the month, or issuance secured by a registered lien on the building held for the holders by a collateral agent, makes rent, mortgages and fractional property ordinary tuples, with the deed in escrow buying hardness through the legal layer. Attaching a legal instrument buys enforceability, and only where courts work, which is what [§2](#2-what-we-are-designing) declined to rely on. It is a hardness upgrade for those who have it, rather than a foundation.

## Portfolio money

David Chaum's *Better Than Money* (2023) proposes it: bearer claims on the holder's own portfolio of asset claims, one custodian per asset. The money's value is the portfolio's, holders stay invested, and a holder whose medium is losing value has somewhere to go without leaving money. Payment there liquidates the payer's proportions through a momentary issuer unit and rebuys the payee's. Here each asset claim is an ordinary backing and the wallet already is the portfolio, so the same money runs with no issuer in the middle.

## Monies with no precedent

The same freedom produces monies nobody has run. Each is a tuple rather than a protocol change, so naming one is close to building it. One example carries the point: aid anchored in food by construction, an agency root redeemed in staple rations, transparent, with the shops that take it as sinks. The rest of the empty plane is reached the same way.

---

# Glossary

| Term | Means |
|---|---|
| **acceptance anchor** | a backer whose sink is wide: many people will take its claims in settlement of something they already owe |
| **acceptance backing** | a backing requiring one unit of a named counterparty's root, which is how a circle writes mutual acceptance ([§8b](#8b-closure-and-what-r-costs)) |
| **anonymity set** | the crowd a transaction hides in |
| **assemblability** | completability computed net of the writer's own holdings: can a holder complete a set |
| **backer** | the key that signs a backing and issues its claims. The only key that can increase what is outstanding |
| **backing** | the object: **B = (K, P, R, E)**, signed, public, immutable, named by the hash of its four fields |
| **bearer** | whoever holds a claim owns it. No register, no permission to transfer, no clawback |
| **blinding** | the cryptography that unlinks a claim from its replacement at reissue, so the attester signs without seeing what it signs |
| **burning** | a holder destroys a claim, for good. The only way outstanding falls |
| **chain** | a public record many strangers keep in agreement, so no one party can rewrite it |
| **chain asset** | a chain's native asset held on its own chain. Nobody's promise, so not a backing. A payout may pay in it and a reliance may require it |
| **checkable supply** | anyone can verify the outstanding count after the fact. What both core settings buy ([§13](#13-what-it-takes-to-run)) |
| **circulating** | outstanding minus what the backer itself holds. Provable under transparent, a disclosure elsewhere |
| **claim** | a bearer quantity held against a backing |
| **closed(b)** | true where R(b) already equals its own closure, so the backer takes in a set it can fully unwind |
| **closure** | the full chain of backings under a requirement, with counts summed where paths meet |
| **completability** | the fraction of a backing's claims that can presently be assembled into presentable sets ([§11](#11-redemption)) |
| **cover** | a guarantee written over somebody else's promise. An ordinary backing whose reliance names the underlying |
| **demand** | a published presentment naming the backing, quantity, claims offered, evaluation instant and deadline. Unanswered past the deadline, it is the backer's visible failure |
| **denomination chain** | the chain of what each backing pays in, followed link by link until it ends in something paying in itself or in a nameable term |
| **enforceable supply** | the protocol itself prevents overissuance, which needs a chain, whether the asset is native or a contract's issuance ([§10](#10-roots-grounding-and-hardness), row 5) |
| **evidence (E)** | who attests that a claim is unspent, plus the venue, the witness interval, the replacement rule, the silence clause, and the claim-layer setting and its construction |
| **grounded** | a root whose payout chain ends in something that is nobody's liability |
| **guarantor** | whoever writes cover |
| **haircut** | a payout set below the underlying's, the usual shape of cover |
| **hash** | a short fingerprint computed from a document. It cannot be reversed, and no document can contain its own |
| **holder** | the key that transfers, burns and presents claims |
| **in the money** | a cover currently worth exercising |
| **load(z)** | the *z* units all promises requiring *z* would consume, divided by how many exist. Above 1, more has been written over *z* than *z* carries |
| **max liability** | assemblability · circulating · payout: a guarantor's largest simultaneous exposure, published beside its terms |
| **metric quality** | the weakest claim-layer setting over a backing's reliance closure and denomination chain. Fixed at signing |
| **moneyness** | how far in or out of the money a cover sits |
| **monoline** | a guarantor whose covers share one failure condition, so they crystallise together |
| **nameable term** | something a payout may pay in and a reliance may require, but which nobody signed: a chain asset, a commodity |
| **nullifier** | a one-time tag published at spending, marking a claim spent without revealing which |
| **obligor (K)** | the verification key that owes |
| **option** | the right, bought for a fee, to demand something later at terms fixed now |
| **outstanding** | issued minus burned, in claim quantity, per backing |
| **payout (P)** | what one unit pays, as a function of public state. In the core it reads witnessed time only and is plain arithmetic; [Extensions](extensions.md) widens the argument list |
| **policy** | a wallet's acceptance and pricing rules: which obligors it takes, what discount it applies, which backers it treats as failing together, what portfolio to hold ([§15](#15-valuation)) |
| **presentable** | a holding is presentable at *b* for *q* if it contains *q* units of *b* and *q·cᵢ* of each *(bᵢ, cᵢ)* in R(b) |
| **published** | retrievable by a stranger, and priced at zero where it is not |
| **reliance (R)** | the fixed list of (backing or chain asset, count) pairs that must accompany a claim at presentation. May be empty |
| **root** | a backing with empty reliance. Requires nothing alongside it |
| **sequencer** | the service that orders transfers and co-signs them, proving a claim unspent. Holds no funds and cannot issue |
| **shielded** | the core setting hiding histories, and under the pooled construction amounts too. Checkable at the boundary |
| **silence clause** | the terms in **E** for a failing sequencer: two durations, two grades, the non-service aggregate (*m*, *W*) and the refusal aggregate (*m′*, *W′*) |
| **sink** | somebody who will take a claim in settlement of something already owed to them |
| **substitutability** | any backing can name a different operator without asking the rest, which is why no operator is mandatory |
| **successor** | new terms published as a separate backing with a standing swap offer, since a backing has no edit operation |
| **transparent** | the core setting: a public ledger of key-controlled balances. Exact supply, no unlinkability |
| **unit** | a backing's own count: the quantity issuance creates and burning destroys. Not the unit of account it pays in |
| **unit of account** | the thing prices are quoted in. Defined wherever some backer redeems in kind at a fixed quantity ([§12](#12-units-of-account-and-prices)) |
| **unitload(z)** | leverage through denomination: the *z* owed by every backing whose denomination chain reaches *z*, divided by outstanding *z* ([Extensions](extensions.md)) |
| **wallet** | a set of claims, and any bare assets held alongside them |
| **witnessed time** | the clock formed by the witness venue's finalized indices, read at the declared interval. Commitments land on it and do not define it. The only time everyone agrees on, and the one a core payout reads |
| **witness interval** | how often a sequencer publishes a commitment to an outside venue. The granularity of every dated and state-reading term |
| **zero-date** | the date after which a dated payout reads zero. Distinct from maturity, when it starts paying |

---

## References and further reading

**Credit theory and history.** A. Mitchell Innes, *What is Money?* (1913), *The Credit Theory of Money* (1914). David Graeber, *Debt: The First 5,000 Years* (2011). Richard Cantillon, *Essai sur la Nature du Commerce en Général* (c. 1730).

**Private and competing money.** F. A. Hayek, *Denationalisation of Money* (1976). Selgin and White on Scottish free banking. E. C. Riegel, *Private Enterprise Money*. Gary Gorton on free bank note pricing (1996, 1999). Stodder on WIR.

**Mechanism.** David Chaum, *Blind Signatures for Untraceable Payments* (1982), *Better Than Money* (2023). Ryan Fugger, the original Ripple (2004). Open Transactions; Cashu; Fedimint. Alexander Chepurnoy (kushti), *ChainCash* and *Basis*.

**Law.** Uniform Commercial Code Articles 3 and 12, especially §§3-104, 3-302, 3-305, 12-105. Bills of Exchange Act 1882 (UK). UNCITRAL, *Model Law on Electronic Transferable Records* (2017); UK *Electronic Trade Documents Act 2023*.
