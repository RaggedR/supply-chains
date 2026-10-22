# Lean Verification Plan — *Supply Chains as Directed Containers*

Target paper: `papers/supply-chains-containers.tex`
Existing infrastructure: `~/git/trust-boundaries/lean` (sorry-free, Lean 4 v4.32.0, no Mathlib)

## What is already verified (reuse, don't redo)

| Paper claim | Lean artifact (trust-boundaries) |
|---|---|
| Def 2 (def:dcont), directed container, laws D1–D5 | `Directed.lean` — `DirectedContainer` |
| ACU theorem: DC structure ⇔ comonad on extension | `Comonoid.lean` + `ComonadConverse.lean` |
| Ahman–Uustalu: DCs are small categories (both directions) | `DContCat.lean` — `cat C`, category → DC, round-trip bijections |
| DC-morphisms are cofunctors (`DCont ≅ Cof`) | `Cofunctor.lean` — `DContMorphism`, `Cofunctor`, both directions |
| con:bom — ℬ, product-labelled rooted trees as DC | `TrustBoundary.lean:117` — `trustRegistry T`, take `T := Products` |
| Unlabelled rose trees as DC | `ProofSearchN.lean:98` — `proofSearchN` |

## What needs to be built

Dependency chain: **N1 → {N2, N3} → N4**; N5 uses `Cofunctor.lean`; N6, N7 independent.

### N1. `Network.lean` — Construction con:network (the keystone)
- `structure Digraph` : `V : Type`, `E : Type`, `src tgt : E → V` (finiteness NOT
  assumed — the paper drops it on the graph rung anyway; add `[Fintype]` only
  where prop:tree needs it).
- Inductive `Path (G) : G.V → G.V → Type` (nil / cons), concatenation, its
  associativity and unit laws.
- `network G : DirectedContainer` with shapes `V`, `Pos v = Σ w, Path G v w`,
  sub = endpoint, root = nil, shift = concatenation.
- **Deliverable**: D1–D5. D1/D2 should be `rfl`; D3–D5 by induction on paths
  (structurally easier than the RTree recursion already done — list-append
  associativity in disguise).
- Sanity check: instantiate the 9-facility Sawasdee graph (2 vineyards … 3
  retailers) as a concrete `Digraph` and `#eval`/`decide` that
  `P(Vineyard₁)` has 9 elements and `P(Retail₁)` has 1.

### N2. `Thin.lean` — Proposition prop:tree (the paper's only proved proposition)
- Define *polytree*: the underlying undirected graph is acyclic. Suggested
  formalization: undirected walks as paths in the symmetrised graph
  (`E ⊕ E` with swapped src/tgt), acyclicity = any closed walk without
  immediate backtracking is trivial. (Design decision — pick whatever makes
  the injectivity proof cleanest; alternatives: unique undirected path
  between any two vertices.)
- Theorem `thin_of_polytree : Polytree G → ∀ v w, Subsingleton (Path G v w)`.
- Corollary: `endpoint : Pos v ≃ {w // Reachable G v w}` (the paper's
  `P(v) ≅ reachable nodes`).
- Paper's proof sketch: two distinct paths v→w yield an undirected cycle.
  Formal version: induct on the pair of paths; first edges differ ⇒ splice
  into a nontrivial closed walk. This is the hardest single proof in the
  plan; expect the polytree *definition* to need one revision.

### N3. `Tracking.lean` — Construction con:tracking (free-category universal property)
- `structure Monoid'` or reuse Lean core `Monoid`.
- `Tracking G M := ` functor `network G → BM`: a map
  `d : ∀ v, Pos v → M` with `d (root) = 1` and `d (p ⊕ q) = d p * d q`.
  (No category framework needed — state it directly on the DC, or via
  `DContCat.cat` if that reads better.)
- **Main theorem**: `Tracking G M ≃ (G.E → M)` — "a tracking is exactly a
  labelling of legs". Forward: restrict to single-edge paths. Backward: fold
  along the path. Round trips by path induction.
- Instances as definitions + small `example`s: max-temperature monoid
  `(ℝ ∪ {-∞}, max)` can be `(WithBot ℚ, max)` to stay decidable;
  cold-chain `(Bool, ∧)`; licences `(Finset L, ∪)`. One `decide` example:
  a warm last-mile leg breaks the whole route (idempotence bite).

### N4. `Sourcing.lean` — Construction con:sourcing (glues ℬ to 𝒩)
- New notion needed: **covariant functor of directed containers**
  (shapes forward, positions forward, preserving root/sub/shift up to the
  transported equalities). Not in trust-boundaries — `DContMorphism` is the
  cofunctor (positions backward). Define `DContFunctor C D` mirroring the
  M0/M1/M2 pattern of `Cofunctor.lean` but covariant; or define it as a
  functor `cat C ⥤ cat D` using `DContCat.lean`. **Decision point: choose
  after N1 lands — the `op` is easier to manage on the category side.**
- `Dop` : opposite of a DC via `cat`/op/back, or directly (positions of
  `network G`ᵒᵖ at v = paths *ending* at v). Direct reversal of the graph
  (`Gᵒᵖ` swaps src/tgt) is likely simplest: `𝒩ᵒᵖ = network Gᵒᵖ`.
- `sourcing : DContFunctor (bom Products) (network Gᵒᵖ)` given data:
  site of each product + an inbound leg per BOM edge; functoriality =
  consolidation. Prove the two functor laws by occurrence induction.
- Concrete instance: the 4-node steel BOM (rebar/billet/DRI+scrap/ore)
  over the 4-facility steel network.

### N5. `Aggregation.lean` — Construction con:aggregation (routing policy)
- Reuse `DContMorphism`/`Cofunctor` from trust-boundaries verbatim.
- Aggregate network: `network G̃` for the 2-node graph Australia → Thailand.
- `routingPolicy : DContMorphism (network G_wine) (network G̃)` — region map
  on shapes, lift of the single aggregate leg to the concrete route per
  facility. Laws C0/C1/C2 by cases on the (finite) aggregate positions.
- This is a *witness*, not a theorem: the content is that the concrete
  Sawasdee policy type-checks against the existing cofunctor laws.

### N6. `Rungs.lean` — the DAG and graph rungs (negative results, cheap)
- DAG: add the Singapore transshipment edge; `example : ¬ Subsingleton
  (Path G' MelbournePort ThaiPort)` by exhibiting both routes (`decide`).
- Cycle: scrap loop rolling-mill → melt-shop; construct `laps : ℕ → Path G v v`
  injective ⇒ `Infinite (Pos v)`, and a nontrivial endomorphism.

### N7. `Inventory.lean` — Section 8 (update monad) — optional stretch
- Constant-position DCs ≅ monoid actions: `monoidActionDC (M) (S) (act)` with
  D1–D5 = action laws; conversely extract the action. (Small.)
- Update monad `T X = S → M × X`: monad laws from the action laws.
- Remark-level lemmas: abelianization forgets order (ℤ^Prod vs free monoid);
  monus clamps (`∃ s m, (s ⊖ m) ⊕ m ≠ s` over ℕ) — one-line `decide`.
- Skip: the double category of squares (sec 7.1) — real project, out of scope.

## Where the code lives — DECIDED (Robin, 2026-07-23)

**Everything goes in `~/git/trust-boundaries/lean`**, alongside the existing
containers library. New files N1–N7 sit next to `Directed.lean` and
`Cofunctor.lean`; the Sawasdee/steel instantiations live inside the same
files as `example`s/`def`s (or a single `SupplyChain.lean` if they grow).
This repo (supply-chains) keeps only the paper and this plan.

## Read these files first (next instance: start here)

Before writing any Lean, read in this order:

1. `~/git/trust-boundaries/lean/README.md` — library map, build instructions.
2. `~/git/trust-boundaries/lean/Directed.lean` — `DirectedContainer`, laws
   D1–D5, the transport idioms (D5 transports along D4). Every new file
   imports this.
3. `~/git/trust-boundaries/lean/DContCat.lean` — DC ↔ small category
   dictionary (`cat C`, `cod`, `idHom`, `after`); the round-trip-gap
   discussion in its docstring explains why not to force `rfl`-equalities.
4. `~/git/trust-boundaries/lean/Cofunctor.lean` — `DContMorphism` (M0/M1/M2)
   and `Cofunctor` (C0/C1/C2). N5 instantiates this; N4's `DContFunctor`
   should mirror its file structure and law-naming style.
5. `~/git/trust-boundaries/lean/ProofSearchN.lean` (rose-tree DC, line ~98)
   and `TrustBoundary.lean` (`trustRegistry`, line ~117) — the proof
   patterns for path-induction over trees; N1's path induction is a
   simplification of these.
6. `~/git/trust-boundaries/lean/lakefile.lean` + `lean-toolchain` — package
   layout, Lean v4.32.0, no Mathlib (do not introduce it).
7. `~/git/supply-chains/papers/supply-chains-containers.tex` — the claims
   being verified; key lines: con:network 396, prop:tree 418, con:tracking
   523, con:bom 681, con:sourcing 723, cofunctor def 860, con:aggregation
   869, inventory §8 (~line 939).

## Stages — one instance per stage

Each stage is self-contained: an instance reads only the listed files plus its
task section above, works in `~/git/trust-boundaries/lean`, and is done when
`lake build` passes sorry-free. Stages B1–B3 and D can run as parallel
instances once Stage A has landed.

**Staging rationale**:
- The stages are the transitive reduction of the dependency DAG above; N5+N6
  are bundled (both low-effort, both depend only on N1) so no instance gets a
  trivially small task.
- The Stage-A "decision note" is the cross-instance handoff: Stage A makes the
  covariant-functor design call while `Network.lean` is fresh in context, and
  writes it into this file — the plan doubles as the inter-instance message bus.
- B3 (Thin) gets a whole instance despite being one file: it's the flagged
  likely-false-start, so a failed polytree definition burns only that
  instance's context.

### Stage A — Foundation (must go first, blocks everything) — ✅ DONE 2026-07-23
- **Build**: N1 `Network.lean`.
- **Read**: files 1, 2, 5, 6, 7 (con:network, line 396) from the reading list.
- **Also decide & record**: at the end, append a note to this file answering
  N4's decision point (covariant `DContFunctor` directly vs via `cat`/op) —
  Stage C consumes that decision.
- **Done**: D1–D5 proved; Sawasdee 9-facility `#eval` sanity check passes.

### Stage B1 — Tracking (after A)
- **Build**: N3 `Tracking.lean`.
- **Read**: 1, 2, `Network.lean`, 7 (con:tracking, line 523).
- **Done**: `Tracking G M ≃ (G.E → M)`; three monoid instances; one `decide`
  example (warm last-mile leg).

### Stage B2 — Rungs + Aggregation (after A; low difficulty, pairs well)
- **Build**: N6 `Rungs.lean` and N5 `Aggregation.lean`.
- **Read**: 1, 2, 4 (`Cofunctor.lean`), `Network.lean`, 7 (con:aggregation,
  line 869).
- **Done**: DAG non-thinness `decide`; cycle ⇒ `Infinite (Pos v)`; Sawasdee
  routing policy type-checks against C0/C1/C2.

### Stage B3 — Thin (after A; **hardest single proof**, give it a whole instance)
- **Build**: N2 `Thin.lean`.
- **Read**: 1, 2, `Network.lean`, 7 (prop:tree, line 418).
- **Warning**: expect one false start on the polytree definition; three-strike
  rule applies — if the injectivity proof stalls three times, stop and report.
- **Done**: `thin_of_polytree` and the `Pos v ≃ reachable` corollary.

### Stage C — Sourcing (after A; needs Stage A's recorded design decision)
- **Build**: N4 `DContFunctor` (+ `Dop`) and `Sourcing.lean`.
- **Read**: 1, 2, 3, 4, `Network.lean`, this file's Stage-A decision note,
  7 (con:bom 681, con:sourcing 723).
- **Done**: functor laws proved; 4-node steel BOM instance compiles.

### Stage D — Inventory (optional stretch; independent, can run any time)
- **Build**: N7 `Inventory.lean`.
- **Read**: 1, 2, 7 (§8, ~line 939). Does not need `Network.lean`.
- **Done**: monoid-action ≅ constant-position DC; update-monad laws; the two
  `decide` remarks.

**Instance briefing template**: "First, read ~/.claude/AGENT.md. Then read
`~/git/supply-chains/LEAN-PLAN.md` — you are Stage ⟨X⟩. Follow its reading
list, build only your stage's files in `~/git/trust-boundaries/lean`, keep it
sorry-free on Lean v4.32.0 with no Mathlib."

## Stage A — DONE (2026-07-23) + decision note for Stage C

**Status**: N1 `Network.lean` built, sorry-free. D1, D2, D4 are `rfl`
(positions are `⟨endpoint, path⟩` pairs, so `sub` is first projection);
D3/D5 are `append_nil`/`append_assoc` by path induction. Sanity checks:
`|P(Vineyard₁)| = 9`, `|P(Retail₁)| = 1`, both by `rfl`.

**Corrections to this plan** (discovered during Stage A):
- The buildable Lake package is `~/git/macbeth-seed/lean/Containers/`
  (module dir `Containers/`, toolchain **v4.30.0**, not v4.32.0).
  The files in `~/git/trust-boundaries/lean/` are **reference copies**
  (see its README). Workflow: develop + `lake build` in macbeth-seed,
  add your module to `Containers.lean`, then `cp` the file to
  `trust-boundaries/lean/` and update its README table.

**N4 decision (op / covariant functor)**: go **direct**, not via `cat`.
- `Gᵒᵖ` is just `⟨V, E, tgt, src⟩` (swap endpoints); use `𝒩ᵒᵖ := network Gᵒᵖ`.
  No general DC-opposite needed — avoids the `DContCat` round-trip gap.
- Define `DContFunctor C D` directly on DCs, mirroring `Cofunctor.lean`'s
  M0/M1/M2 file structure and law-naming, with positions covariant.
  `network`'s `sub = Sigma.fst` keeps the transported equalities `rfl`-cheap
  on the network side.

**Elaboration lessons for Stages B1–C** (concrete-graph instantiations):
- Mark concrete `src`/`tgt` maps and the concrete `Digraph` **`@[reducible]`**,
  or instance search (e.g. `HAppend` on `List (Route (Leg.src e))`) fails.
- Never ask Lean to infer an edge from its endpoint: `Path.cons` has type
  `Path G (G.src e) w`, and `G.src ?e =?= v` is a stuck match — unsolvable.
  Build concrete positions with a helper (`alongLeg : (e : Leg) →
  Route (tgt e) → Route (src e)`, with `(G := graph)` explicit) instead of
  raw `.cons` chains or explicit node arguments under an unsolved `G`.

## Stages B1–B3 — DONE (2026-07-23)

Three parallel instances; all built sorry-free; full `lake build` (23 jobs)
passes with all four new modules imported from `Containers.lean`. Files also
copied to `trust-boundaries/lean/` and its README table updated.

- **B1 `Tracking.lean`**: `Tracking G Mon` (d_root/d_shift), main bijection
  `trackingBijection G Mon : Bijection (Tracking G Mon) (G.E → Mon.M)`.
  No-Mathlib substitutions: bundled `Monoid'` (ZappaSzep style) instead of a
  typeclass; bespoke `Bijection` instead of `Equiv`; two-element `MaxTemp`
  instead of `WithBot ℚ`. Warm-last-mile example proved by `simp`.
- **B2 `Rungs.lean` + `Aggregation.lean`**: DAG non-thinness via path-length
  contradiction (`pathLen` 1 ≠ 2, `omega`); scrap loop `laps : ℕ → Path`
  injective ⇒ minimal `Infinite (Pos rollingMill)`; nontrivial endomorphism.
  `routingPolicy : DContMorphism Wine.network (network GAgg)` — M0/M1 by
  `rfl` casework, M2 26 branches. Elaboration lesson: match on the concrete
  facility, not on `region f` — dependent substitution doesn't flow back
  into the position's type.
- **B3 `Thin.lean`**: `path_unique_of_polytree`, `thin_of_polytree`,
  `endpoint_equiv` (`Pos v ≃ {w // Reachable G v w}`, bespoke `Equiv'`).
  One false start, as predicted. Technical trick: `PathView`, a non-indexed
  view type packaging constructor tags with index-equality proofs, to avoid
  stuck dependent elimination on the doubly-indexed `Path`.

**Corrections discovered during Stage B**:
- `Cofunctor.lean` and `DContCat.lean` BOTH define `DContMorphism`/`Cofunctor`
  at top level with different field names (`f`/`fSharp` vs `onShapes`/`onPos`).
  They cannot coexist in one import closure. The root already imports
  `DContCat`, so **new files must import `Containers.DContCat`, not
  `Containers.Cofunctor`**, and use field names `onShapes`/`onPos`.
  Stage C: mirror `DContCat`'s naming for `DContFunctor`.
- **Caveat for prop:tree (flag to Robin)**: Lean's `Polytree G` is defined as
  `Acyclic G` (no nontrivial directed self-loop path) + `EdgeDisjoint G`
  (distinct co-sourced edges reach disjoint vertex sets) — the two hypotheses
  the induction actually needs. This is *implied by* the paper's
  "underlying undirected graph acyclic" but is likely strictly weaker, so
  the theorem is stronger than prop:tree; the missing lemma
  (undirected-acyclic ⇒ Polytree) is unproved. Either add that bridge lemma
  or adjust the paper's Formalization paragraph to state the hypothesis used.

**Remaining**: optional Stage D (N7 `Inventory.lean`) only — steps 1–7 of
the effort table are now all sorry-free, so the paper's "Formalization"
paragraph can be written (subject to the two caveats: Polytree above,
routed BOMs below).

## Stage C — DONE (2026-07-23)

Two files, built sorry-free and axiom-free in macbeth-seed (25 jobs),
copied to `trust-boundaries/lean/`, README table updated.

- **`DContFunctor.lean`**: covariant functor of directed containers
  (`onShapes`/`onPos` both forward; laws F0/F1/F2 mirroring
  `DContCat`'s M0–M2 naming, F2 transporting the inner position along
  F0). Identity, composition (two `subst; rfl` transport lemmas:
  `onPos_transport`, `transport_trans`), unit + associativity laws all
  `rfl`. Imports `Containers.DContCat` per the Stage-B correction.
- **`Sourcing.lean`**: `Digraph.op` (swap `src`/`tgt`, `@[reducible]`);
  **routed BOM trees** `Bom P G v` (indexed by producing facility; each
  edge carries its inbound route in `G.op`); `bom P G` DC with D1/D2
  `rfl`, D3–D5 by occurrence induction (LTree pattern);
  `sourcing P G : DContFunctor (bom P G) (network G.op)` — F0/F1 `rfl`,
  F2 = `routeOf_shift` (consolidation), one 10-line path induction
  stated as an equation of positions (Sigma pairs) so the D4-equal
  endpoints travel with their routes. Steel instance: 4 facilities on a
  path, 5 products, 4-level rebar BOM; ore's three-leg inbound route and
  a consolidation equation both checked by `rfl`.

**DESIGN DEVIATION from N4 (flag to Robin, alongside the Polytree
caveat)**: con:sourcing as stated — data "site per product + inbound leg
per BOM edge", functor on all of ℬ — is *not formalizable*: ℬ contains
perverse shapes (ore as parent of rebar) whose occurrences would need
routes running backwards through the steel path graph; the total data
`inbound : ∀ parent child, Path …` is uninhabitable for the steel
network. Fix chosen: the domain's shapes carry their routing choices
(routed BOMs), and sourcing is the *canonical* functor, no side data.
The paper's data builds exactly one routed BOM per plain BOM of
interest. The alternative (subtype of ℬ compatible with a fixed
component relation) makes D1 propositional and reintroduces transports
everywhere. **Pending Robin's call**: adjust the paper's Formalization
paragraph to say the shapes of ℬ are equipped with their sourcing
choices when con:sourcing is applied (or state the construction on
routed BOMs directly).

**Elaboration lessons (Stage-A lesson, sharpened)**:
- The unifier will NOT assign a `@[reducible]` constant (e.g. the
  concrete `graph`) to a constructor's graph metavariable through an
  indexed family: `.node …` and even `Bom.node …` fail against expected
  `Bom Product graph v`. Fix: fix the parameters up front with a helper
  `@[reducible] def node' := @Bom.node Product graph`.
- Same for `Path.nil`/`Path.cons` in branches of a dependent `match`:
  give `(G := graph.op)` explicitly.
- Nested anonymous occurrence chains `.inside 0 (.inside 0 .here)`
  leave the outer implicit `children` as a stuck metavariable
  application `?children 0` (not a Miller pattern). Build occurrences
  bottom-up through typed `def`s, one level per definition.

## Stage D — Briefing (planned 2026-07-23; ready to run)

**Build**: N7 `Inventory.lean` (one file). Develop in
`~/git/macbeth-seed/lean/Containers/`, add to `Containers.lean`,
`lake build` (toolchain v4.30.0, no Mathlib), then copy to
`trust-boundaries/lean/` and update its README table — same workflow as
Stages A–C.

**Read**: `Directed.lean`, `Tracking.lean` (reuse its bundled `Monoid'`
and `Bijection` — do NOT redefine them), paper §8 (lines 939–1023).
Import `Containers.DContCat` if anything categorical is needed (per the
Stage-B correction); `Network.lean` is NOT needed.

### D.1 Monoid actions ⇔ constant-position DCs

Design decision (made now, from the marie-claire constant-Pos note):
**do not state "constant position" as a type equality** `∀ s, C.Pos s = M`
— that buries everything in transports. Instead:

- `structure MonoidAction` : `Mon : Monoid'`, `S : Type`, `act : S → Mon.M → S`,
  `act_one`, `act_mul` (right action).
- `def MonoidAction.toDC : MonoidAction → DirectedContainer` with
  `Pos := fun _ => Mon.M`, `sub := act`, `root := 1`, `shift := mul`.
  D1/D2 = action laws, D3/D4 = unit laws, D5 = associativity. Because
  `Pos` is literally constant, the D4/D5 transports are along `rfl` —
  expect every law to be the corresponding monoid/action law verbatim.
- Converse: `structure ConstDC` — a DC restated with `Pos` a fixed `M`
  (fields `S M sub root shift` + the five laws with no transports).
  `def ConstDC.toAction : ConstDC → MonoidAction` — the monoid structure
  on `M` is `root`/`shift`: with constant positions, shift genuinely IS
  a binary operation on one set (this is the *sound* case of the
  "monoid that wasn't" — say so in the docstring; the dream journal
  2026-06-20 records the unsound general case).
- Round trips `MonoidAction ↔ ConstDC` by `rfl`-level structure eta;
  package as `Bijection` (from `Tracking.lean`) if it's free, otherwise
  two `theorem ... : toAction (toConst A) = A` statements suffice.
  Also `def ConstDC.toDirected : ConstDC → DirectedContainer` (the
  trivial embedding) so `toDC = toDirected ∘ toConst` connects to the
  real `DirectedContainer`.

### D.2 Update monad

- `def Update (A : MonoidAction) (X : Type) := A.S → A.Mon.M × X`.
- `pure x := fun _ => (1, x)`;
  `bind t k := fun s => let (m, x) := t s; let (m', y) := k x (A.act s m); (m * m', y)`.
- Three monad laws as plain theorems (`update_pure_bind`,
  `update_bind_pure`, `update_bind_assoc`), each `funext s` then the
  corresponding monoid/action law. No `Monad` typeclass instance —
  universe/typeclass ceremony buys nothing here; bespoke structures are
  the house style (B1 precedent).

### D.3 The two `decide` remarks

- **Abelianization forgets order** (paper Remark, line 995): two
  movement words with equal net effect but unequal as lists.
  `Movement := receipt q | issue q` over `Prod := Fin 1` (keep it tiny),
  net : List Movement → ℤ. `example : net w₁ = net w₂ ∧ w₁ ≠ w₂ := by decide`.
- **Monus clamps** (Remark, line 1001):
  `example : ∃ (s m : ℕ), (s - m) + m ≠ s := ⟨0, 1, by decide⟩` —
  and a companion showing ℤ repairs it: `∀ s m : ℤ, (s - m) + m = s`
  (by `omega` or ring lemmas from core — check what v4.30 core offers;
  if awkward, state for the concrete instance only).
- Concrete inventory instance: `S := ℤ × ℤ` (two products, backorders
  admitted per the paper's remark), `M := ℤ × ℤ` under `+`, act = `+`.
  A replenishment policy example as a Kleisli map
  `policy : Unit → Update A Unit` reading stock and emitting a reorder.

### D.4 Elaboration warnings (inherited from Stages A–C)

- No indexed families in this file — the v4.30 unifier traps (stuck
  `?G` through indexed families, non-Miller `?children 0`) shouldn't
  arise. If a concrete inductive is used for `Movement`, still mark
  concrete defs `@[reducible]` before `decide` examples.
- Read compiler errors from the top (`grep -A8 "error:"`), never `tail`.
- Skip entirely: §7.1 double category, §9 Poly dynamics — out of scope
  per N7.

**Done when**: `lake build` sorry-free with `Inventory.lean` imported
from `Containers.lean`; file copied to trust-boundaries; README updated;
a completion note appended to this file.

**Instance briefing**: "First, read ~/.claude/AGENT.md. Then read
`~/git/supply-chains/LEAN-PLAN.md` — you are Stage D; follow the
'Stage D — Briefing' section exactly. Work in
`~/git/macbeth-seed/lean/Containers/`, keep it sorry-free on Lean
v4.30.0 with no Mathlib."

**Confidence**: high (≈85%) — every law is a renamed monoid/action law,
no transports, no indexed families; the one risk is core-library gaps
around ℤ lemmas without Mathlib (mitigation in D.3).

## Stage D — DONE (2026-07-23)

**Status**: N7 `Inventory.lean` built, sorry-free, axiom-clean (only
`Quot.sound`/`propext` via `funext`/`omega`); full `lake build` 26 jobs.
Copied to `trust-boundaries/lean/`, README table updated. All eight steps
of the effort table are now sorry-free — the stretch goal is closed.

- **D.1**: `MonoidAction` (bundled right action on `Monoid'` from
  `Tracking.lean`), `ConstDC` (five transport-free laws c1–c5,
  shape-independent `root`/`shift`), dictionaries `ConstDC.toAction` /
  `MonoidAction.toConst` with **both round trips `rfl`** (structure eta +
  proof irrelevance), and `ConstDC.toDirected` into the real
  `DirectedContainer`; `MonoidAction.toDC := toDirected ∘ toConst`,
  connection theorem `rfl`. As predicted, every D-law is the matching
  monoid/action law; the only content is one helper `transport_const`
  absorbing the D2/D5 casts. `Bijection` packaging skipped as anticipated:
  `MonoidAction : Type 1`, `Bijection` is `Type`-only — two `rfl` theorems
  instead.
- **D.2**: `Update A X := A.S → A.Mon.M × X`, `pure`/`bind` written with
  `.1`/`.2` projections (no pattern-matching `let` — keeps goals
  rewritable). Laws: `pure_bind` (act_one + one_mul), `bind_pure`
  (mul_one), `bind_assoc` (`simp only [bind]; rw [← A.act_mul,
  A.Mon.mul_assoc]`). All three: `funext s` + `show`/`rw`, closed by
  Prod eta. Trivial difficulty, as forecast.
- **D.3**: abelianization example (`Movement`, `net : List Movement → ℤ`,
  equal net / unequal lists, `decide`); monus clamp `⟨0, 1, by decide⟩` +
  ℤ repair by `omega` (present in v4.30 core — no gap); `Warehouse`
  namespace: `intPairMonoid` (ℤ×ℤ, +), `inventory` action (backorders),
  `warehouseDC := inventory.toDC`, `receive` and order-up-to `policy`
  Kleisli maps, three `decide` examples incl. a `bind` showing the policy
  reads post-receipt stock.

**Elaboration lesson (new)**: the `▸` notation **refuses to elaborate a
cast whose motive is constant** ("equality does not contain the expected
result type on either side") — you cannot even *state* `(h ▸ q : M) = q`
for `h : s = t` with `M` unrelated. State the helper over a raw
`@Eq.rec S s (fun _ _ => M) q t h` instead; it then unifies fine (up to
defeq) with the transports the `DirectedContainer` fields produce when
`Pos := fun _ => M`. Corollary: don't restate D2/D5 goals with `show`
containing `▸` — write the proofs as terms (`Eq.trans`, `congrArg`).

**Deviations from briefing**: none of substance. Law names are
namespaced (`Update.pure_bind` etc.) rather than `update_pure_bind`;
`ConstDC` keeps shape-independent `root`/`shift` per the briefing (the
docstring notes this is the sound case of the "monoid that wasn't").

## Order of work & effort

| Step | File | Difficulty | Depends on |
|---|---|---|---|
| 1 | Network.lean | low (paths + append laws) | — |
| 2 | Tracking.lean | low-medium (UP of free cat) | 1 |
| 3 | Rungs.lean | low (decidable witnesses) | 1 |
| 4 | Thin.lean | **high** (polytree defn + cycle argument) | 1 |
| 5 | DContFunctor (+ op) | medium (design, then mechanical) | 1 |
| 6 | Sourcing.lean | medium | 5, bom |
| 7 | Aggregation.lean | low (instantiate Cofunctor) | 1 |
| 8 | Inventory.lean | medium (monad laws) | — |

Definition of done for "the paper is Lean-verified": steps 1–7 sorry-free,
building with `lake build` on v4.32.0; paper gains a short "Formalization"
paragraph citing the repo. Step 8 stretch.

Confidence: high for 1–3, 7 (patterns already proved for RTree; these are
easier). Medium for 4 (the polytree definition is the one place a false
start is likely) and 5–6 (new design, but small).
