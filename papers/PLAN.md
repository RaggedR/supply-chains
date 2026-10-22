# PLAN: Restructure supply-chains-containers.tex as a ladder — tree → DAG → graph

**Status: DONE — executed 2026-07-23.** All edits applied, plus a new §1 Introduction (map of the paper, labels `sec:setup`/`sec:glossary`/`sec:dcont` added). Compiles clean with tectonic, no undefined refs. Kept for the record; safe to delete.

## Style rules (hard-won — do not violate)

- **Compile with `tectonic supply-chains-containers.tex`** from `papers/`. No pdflatex on this machine. Open PDF with Skim (`open -a Skim`).
- **Never invent fiction.** Sawasdee Cellars is Robin's *real* venture (`~/wine/`): a Melbourne export *agency* — no warehouse, no bottling, holds no stock, just takes title and markup. Past inventions (gift cases, box mills, warehouses, "Competitor" nodes) were all angrily stripped. Steel facts must be real metallurgy (gas-based DRI/EAF is genuinely standard in the UAE; home scrap is real).
- **One idea at a time** — the whole point of this restructure.
- No SKU/MRP-style jargon unexplained; the glossary is wine-native; manufacturing vocabulary is defined in situ when steel arrives.
- Unicode math in chat with Robin; LaTeX only in the .tex.

## Current state of the file (all compiling clean as of last session)

Section order now: §1 Setup (wine) · §2 Glossary · §3 Directed containers · §4 Network + tracking (`sec:network`, `sec:tracking`) · §5 Cofunctors (`sec:cofunctors`) · §6 Inventory (`sec:inventory`) · §7 Steel (`sec:steel` with subsections `sec:bom`, `sec:sourcing`, `sec:rollups`) · §8 Outlook.

The ladder currently lives in two remarks that this plan dissolves:
- `rem:tree` ("Tree, DAG, graph") in §4 — three-rung ladder, Sawasdee-as-waist, Singapore transshipment, endomorphisms.
- `rem:scrap` ("The scrap loop") in §7 steel — home scrap cycle, BOM well-foundedness as the accountant's cut. Also a "ℬ is not thin" paragraph (two DRI-edges) just before it.
- `rem:scrap` is referenced only from `rem:tree`; both dissolve together.

## New section order (formalism first, then instance, at every rung)

1. **Setup** — rewrite closing note (after shipping graph) to announce the four-rung plan: wine is a tree; steel adds a second tree (the BOM); alternative routings make a DAG; recycling makes a graph with cycles.
2. **Glossary** — unchanged; labels below keep refs valid.
3. **Containers and directed containers** — unchanged.
4. **Trees I: the shipping network** (`sec:network`)
   - Formalism: keep Construction `con:network` (free category on any G) + NEW Proposition `prop:tree`: G a tree ⇒ 𝒩 thin = reachability poset; positions ≅ destinations; "the directed container of a finite tree".
   - Instance: existing wine prose (P(Vineyard₁), deep-sea leg, last mile, comonad, Slogan `slo:coherence`). Fold in from `rem:tree`: no marked root needed (local roots ο_f); Sawasdee (3 in, 1 out) is the *waist*, not a root.
   - `sec:tracking` stays as subsection, verbatim (monoid table, markups, Aff, `rem:noncommutative`, `rem:dutypoint`) + one added line: on a tree, a tracking is an edge-labelling composed along unique paths.
   - DELETE `rem:tree` (content redistributed here, §6, §7).
5. **Trees II: bills of materials — steel** (`sec:steel`) — MOVE the whole steel section here from after inventory.
   - Intro: pipeline prose + in-situ BOM/subassembly definition + tree figure. **Simplify to a genuine tree: ONE DRI edge** ("a billet needing two charges of DRI has two DRI-edges" sentence and the "ℬ is not thin" paragraph move to §6). Steel network is a path (berth → reduction plant → melt shop → rolling mill → port) — still rung 1.
   - 5.1 `sec:bom`: Construction `con:bom` (single-occurrence version), explosion = comultiplication slogan.
   - 5.2 `sec:sourcing`: `con:sourcing`, site : ℬ → 𝒩ᵒᵖ, unchanged.
   - 5.3 `sec:rollups`: roll-ups + CBAM, unchanged EXCEPT delete the sentence "The ontology table of Section sec:inventory gains its third row..." (inventory moves after; its table gets three rows natively).
   - Keep "What the second example buys" remark; last sentence now points *forward* to `sec:cofunctors` (sourcing squares) instead of backward.
6. **NEW §: DAGs — alternative routes and repeated components** (`sec:dag`)
   - Formalism: free category on a DAG: P(f) still finite, no nontrivial endomorphisms, but no longer thin — several routes per endpoint pair; positions are routes, not destinations.
   - Instance (a), wine: Singapore transshipment alongside the direct sailing — same endpoints, different peak temp/freight/duty; the tracking functors force positions = routes (text from old `rem:tree`).
   - Instance (b), steel: two DRI-edges = two occurrences with the same subshape, possibly different suppliers/mill certs/embedded carbon — ℬ non-thin for a manufacturing reason (text currently in §7 steel).
7. **NEW §: Graphs — cycles and the scrap loop** (`sec:graph`)
   - Formalism: free category on an arbitrary graph: loops ⇒ endomorphisms f → f, P(f) infinite (one route per lap); Definition `def:dcont` assumes neither finiteness nor acyclicity. Closing line: "Acyclicity, like thinness, is a property of examples, never a hypothesis of the theory."
   - Instance: home scrap (from `rem:scrap`): rolling mill → melt shop closes a loop; tracking prices n remelt passes (yield ×, energy +); BOM stays a tree because *the parts list cuts the cycle* — well-foundedness is an accounting decision that keeps explosion terminating.
   - Forward pointer: inventory (§9) lives on this rung too — receipt-then-issue is an endomorphism of a stock state.
8. **Cofunctors** (`sec:cofunctors`) — moves after the ladder. In "The two flows do not merge": remove "(Sourcing, once it enters in Section~\ref{sec:steel}, supplies a second family...)" and instead state the sourcing square directly alongside the tracking square (sourcing now precedes this section).
9. **Inventory** (`sec:inventory`) — after cofunctors. Restore table intro to "The note's three directed containers render three different ontologies uniformly:" and re-add row `bills $\B$ & BOM trees & component occurrences \\`. The melt-order issue example becomes a backward ref (fine). Add one line: this container sits on the graph rung of §7.
10. **Outlook** — unchanged (already references both a replenishment order and a steel works order).

## Other edits

- **Abstract**: rewrite around the ladder (wine tree → steel BOM tree + sourcing functor → DAG → cyclic graph; formalism unchanged at every rung; keep the functor/cofunctor double-category sentence and "expository note" closer).
- **Labels**: keep every existing label attached to its moved section. New: `sec:dag`, `sec:graph`, `prop:tree`. Deleted: `rem:tree`, `rem:scrap` — grep for refs to both and fix (only cross-ref is rem:tree → rem:scrap).
- **Glossary refs**: auto-heal via kept labels; verify with a final grep + compile.

## Verification

1. `tectonic supply-chains-containers.tex` — clean; `grep -iE "undefined|warning"` on output empty.
2. Grep the .tex: no `rem:tree`, no `rem:scrap`; `sec:dag`/`sec:graph` referenced from §1 note.
3. Open in Skim; check each rung reads formalism-then-instance; check section numbering and the three-row ontology table.
4. Update memory `~/.claude/projects/-Users-robin-git-supply-chains/memory/project_sawasde_cellars.md` (spine changed to ladder structure) and MEMORY.md hook.
