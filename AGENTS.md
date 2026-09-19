# Money from First Principles

The paper is an argument; Construction and Extensions define the Medium of
Exchange Protocol. Keep that distinction and the reference's version pins clear.

## Start and navigate

Read `git status`, recent log and applicable instructions. For implementation
work, read the companion `reference-ts/WORK.md` when available; it owns the
active slice, acceptance, companion branches and blockers. Read only documents
relevant to that slice, not the full decision history.

| Source | Boundary |
|---|---|
| `money-from-first-principles.md` | Paper: argument, cited rather than versioned |
| `construction.md`, `extensions.md` | Normative core and optional/alternative profiles |
| `pool-v2.md`, `pool-authority.md` | Implemented layouts and authority/history contract |
| `pool-v3.md` | Successor layouts: approved artifacts and adoption remain unset; no backing may declare v3 |
| `pool-recovery.md`, `pool-fault.md` | Later-version presentation, recovery, checkpoint classification and clock |
| `pool-delivery.md`, `pool-fees.md`, `pool-spent.md` | Later-version delivery, transfer/fee and spent-set contracts |
| `pool-v1.md` | Historical layouts, superseded by v2; never reinterpret |
| [Reference decision index](https://github.com/mediumofexchange/reference-ts/blob/main/DECISIONS.md) | Durable choices; follow only the relevant entry |
| [Reference implementation status](https://github.com/mediumofexchange/reference-ts/blob/main/docs/IMPLEMENTATION_STATUS.md) | Runtime/model/experiment evidence and specification pins |

The core claim layer is the shielded pool (Construction C1). Transparent,
accumulator and Chaumian layers are Extensions profiles. Implementation profiles,
pilots and research evidence belong beside reference code. The reference README
pins its runtime revision; a normative rule alone is not implementation evidence.

## Intent and language

Build a protocol anybody can issue into and independently verify, including where
public institutions cannot be relied on. Preserve open entry, private payments,
public supply verification, immutable terms, holder authorization and
compartmentalized failure. Prefer practical, simpler and lower-cost designs
within those constraints; never reintroduce a gatekeeper for convenience.

The paper's own wording governs its purpose:
- "The aim is money creation anyone can contest." (Abstract)
- "What goes is the legal moat. Entry is free, terms are public, and anyone whose promise is better can undercut an incumbent." (§1)
- The system must work where courts are captured, currency is dying, or banks will not open an account (§2).

Use Construction's words and numbered rules, not new normative metaphors.
Say "licensed activity", "licence", "legal moat" or "franchise", not "monopoly":
banking is an oligopoly behind a licensing moat. "Decentralised" is an axis of
§17's plane, not a banner for the neutral grammar; §18 reopens that corner.
The goal is decentralised; the object can also describe fiat and bank deposits.

## Editing and decisions

Paper edits are minimal corrections for accuracy, not rewrites or new claims
smuggled into old sentences. Construction, Extensions and normative contracts
change what implementations must do; clear Construction C0a:
- One mechanism per property; generalise an existing mechanism where possible.
- Do not layer a patch over a mechanism whose rule is wrong.
- Name what is replaced, or why existing mechanisms cannot cover the need.
- Derive the rule from the object/law or a failure they cannot answer.
- State costs and tradeoffs. Deployment-specific optional needs belong in Extensions.
- Use plain, numbered, unambiguous rules. Keep retired mechanisms and their costs in Construction's Appendix.

When implementation exposes ambiguity, quote the exact rule, explain the conflict
and pause only dependent code. Compare the smallest alternatives, including reuse
or omission; explain invariants, trust/privacy, compatibility and resource/operating
costs; name a counterexample or measurement that could falsify the choice.
A fresh independent reviewer inspects the actual proposal and relevant sources.
Resolve material findings with evidence, record the neutral decision/review
disposition in the reference's decision log, then commit the specification before
dependent code. Add an immutable `**Spec change:**` link to that decision.
If the companion checkout is unavailable, put reasoning/evidence in the spec
commit body and reconcile the shared log later. Never silently work around a
specification conflict or reinterpret a pinned version.

Use one strong reviewer by default for consequential/normative choices; add another
only for a distinct risk or unresolved disagreement. Read back critical fixes and
nearby variants. Model agreement, self-review and generated audit prompts do not
replace independent inspection. If review is unavailable, keep its merge gate
and exact review owed while continuing safe work. Routine prose cleanup needs
focused self-review and link/consistency checks, not a review panel.

## Authority and delivery

Standing authorization effective 2026-09-08 covers development, protocol decisions
and merge/push after verification until superseded. A reviewed choice within intent
does not need another approval. This excludes public releases, live deployment,
real funds, destructive operations and access-control changes unless separately
authorized. Ask only for unavailable access/physical input, a departure from core
intent or an action outside authority; continue unaffected work.

Work in complete argument/protocol capabilities with concrete acceptance, evidence
limits and a real stop boundary. A slice can span several commits and repositories;
an internal milestone is not completion. Review consequential choices before code
and integrated sensitive patches at stable acceptance boundaries, not each helper.
Choose by product dependencies and uncertainty; start with the cheapest decisive
probe, including wallet, device and venue assumptions before fixing unused formats.
Delegate bounded outcomes with sources, constraints and acceptance criteria.
Writers own disjoint files/worktrees; one primary integrates.

Check affected links and cross-document consistency. With the reference checkout,
run `node scripts/check-links.mjs ../money-from-first-principles` from its root
and `npm run check:docs` when its handoff/decision index changes.
Fetch and inspect upstream, preserve unrelated work and branch protections,
satisfy required checks/reviews, merge/push and verify remote parity. Do not
bypass failed gates or report unavailable evidence as passed.

Keep instructions here and `CLAUDE.md` exactly `@AGENTS.md`. Keep one current
document per topic; history belongs in Git and decisions, active status in the
companion WORK.md. Record choices, rationale, alternatives, evidence, limits and
status neutrally; no conversation quotes or authority attributed to a person/model.
Git preserves authorship. Apply obvious low-risk workflow improvements in scope;
record larger opportunities in the companion handoff without derailing the slice.
Before stopping or compaction, update that handoff with goal/status, branches and
relevant commits, exact evidence, next action, blockers/review owed and decision links.

Use the reference's production requirements and handoff for the coarse effort
estimate. Reassess from gathered evidence after meaningful product progress or a
consequential blocker; no extra research/delegation just to estimate. Credit
reusable progress before release gates close; never infer progress from counts.
Report rounded changed/requested estimates and blockers; omit unchanged percentages
for routine work. Workflow cleanup is not product progress.

Final reports state delivered behavior, evidence/review, integration and limits.
After each completed slice, recommend staying with the instance or switching,
based on next work, context freshness and efficiency, not claimed model superiority.
