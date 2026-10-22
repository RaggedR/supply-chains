# Handover — Supply Chains as Directed Containers

**Last updated:** 2026-07-23

## Lean verification: COMPLETE

All 8 steps of the effort table are sorry-free. See `LEAN-PLAN.md` for
full stage-by-stage notes (A through D), corrections, elaboration
lessons, and design deviations.

- **Buildable package**: `~/git/macbeth-seed/lean/Containers/` (Lean v4.30.0, no Mathlib)
- **Reference copies**: `~/git/trust-boundaries/lean/`
- **Paper**: `papers/supply-chains-containers.tex` (build with `tectonic`; open with Skim)
- **No git repo** — Robin has declined `git init` so far; offer at session start.

## Open items for the paper

### 1. Polytree caveat (flag from Stage B3)

The Lean `Polytree G` is defined as `Acyclic G` + `EdgeDisjoint G` —
the two hypotheses the induction actually needs. This is *implied by*
the paper's "underlying undirected graph acyclic" but is likely strictly
weaker, so the theorem is stronger than prop:tree. The bridge lemma
(undirected-acyclic ⇒ Polytree) is unproved. Either add that lemma or
adjust the paper's Formalization paragraph to state the hypothesis used.

### 2. Sourcing deviation (flag from Stage C)

con:sourcing as stated — data "site per product + inbound leg per BOM
edge", functor on all of ℬ — is not formalizable: ℬ contains perverse
shapes whose occurrences would need routes running backwards. Fix
chosen: the domain's shapes carry their routing choices (routed BOMs),
and sourcing is the canonical functor. Pending Robin's call: adjust
the Formalization paragraph accordingly.

### 3. One construction, three containers — ladder slogan clarification

**Robin's observation:** the ladder slogan "the formalism stands still"
is easy to over-read. DAGs and graphs ARE different directed containers
than trees — 𝒩_G depends on G; adding one edge changes the position
sets. 𝒩_tree, 𝒩_DAG, 𝒩_cyclic are pairwise non-isomorphic.

**Resolution:** what stands still is the *construction*, not the
container. con:network is a map 𝒩₍₋₎ : Digraph → DCont, and the
ladder varies the argument while the map is fixed. Literal in Lean:
`network : Digraph → DirectedContainer` defined once (`Network.lean`),
applied to four concrete graphs. D1–D5 proved once for arbitrary G.

What changes per rung is which theorems about 𝒩_G survive:

| Rung  | Container  | What holds                                         |
|-------|------------|----------------------------------------------------|
| tree  | 𝒩_polytree | thin: P(v) ≅ reachable nodes (prop:tree)            |
| DAG   | 𝒩_DAG      | finite positions, no nontrivial endos, NOT thin     |
| graph | 𝒩_cyclic   | P(v) infinite, nontrivial endomorphisms v → v       |

Compressed: **one map 𝒩₍₋₎, three arguments, three containers,
progressively fewer theorems.**

**Not yet written into the paper.** The paper gets close (§7 opens:
"Neither con:network nor con:bom used the tree property; only prop:tree
did") but never states the clean version. Candidate insertion points:
top of §7 (where the ladder first climbs) or §4's ladder announcement.
One or two sentences suffice.

## Paper state

- Formalization section written (item→file table + the two caveats above)
- Falcon Steel example fleshed out to wine-section parity (facility
  table, TikZ graph, position counts, site table, worked consolidation
  equation, cost roll-up 440 $/t, CBAM tally ≈1.4 tCO₂/t)
- Build: `tectonic papers/supply-chains-containers.tex`

## Key files to read first

1. `LEAN-PLAN.md` — full verification plan + stage completion notes
2. `papers/supply-chains-containers.tex` — the paper itself
3. `~/git/macbeth-seed/lean/Containers/Network.lean` — the keystone Lean file
4. `~/git/trust-boundaries/lean/Directed.lean` — `DirectedContainer` definition
