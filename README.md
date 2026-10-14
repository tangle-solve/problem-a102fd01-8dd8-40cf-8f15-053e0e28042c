<!-- solve-problem:a102fd01-8dd8-40cf-8f15-053e0e28042c -->
# Manipulation-resistant MEV-burn price discovery

A [Tangle Solve problem](https://solve.tangle.tools/problems/a102fd01-8dd8-40cf-8f15-053e0e28042c).

## What is the problem?

**Setting.** Ethereum's "MEV-burn" proposal: instead of allowing validators/builders to keep all MEV (maximal extractable value), the protocol auctions the right to the ordering position and *burns* the proceeds, with the burn amount computed from builder bids in a (partially centralized) mechanism. The open problem is **mechanism design for price discovery**: design the bidding/auction rules so that the burned amount is a reliable *lower bound on the true extractable MEV*, even when bidders behave strategically.

**Formalize as follows.** There are $n$ builders; builder $i$ has a private valuation $v_i$ for the block (the MEV it can extract, net of costs). Each submits a bid $b_i \ge 0$ (possibly after observing others via integration into downstream protocols or private order flow deals). The protocol commits to a payment/burn rule $B(b_1,\dots,b_n)$ — typically $B = \max_i b_i$ or an elicitation mechanism — and awards the block. Desired properties:

1. **Lower-bound guarantee:** in (approximate) equilibrium, $B \ge \alpha \cdot \mathrm{MEV}^{\text{realized}}$ for a constant $\alpha > 0$ — i.e., bids cannot be suppressed far below true surplus.
2. **Strategy-proofness or equilibrium characterization:** characterize the equilibria, especially under collusion (a coalition of builders coordinating suppression, or a builder+searcher+validator integrating vertically).
3. **Robustness to private order flow:** valuations are endogenous — builders with exclusive order-flow deals have both incentive and ability to suppress visible surplus.

Ambiguity to note: "MEV" itself is not protocol-observable; the target is a bound relative to *realized* value of the proposed block or an estimate thereof, and this choice matters.

## Why does it matter?

MEV-burn is the leading proposal to neutralize the incentive for validators to centralize into intermediaries like MEV-boost relays, central to Ethereum's "protocol-enforced proposer commitments" roadmap. Without manipulation-resistant price discovery, builders can suppress bids and the burn collapses, defeating the whole point; the mechanism is central to planned protocol changes affecting billions of dollars annually. A provable lower-bound rule would let Ethereum credibly burn MEV, reducing centralization pressure and making the fee market honest. It also serves as a template for collusion-resistant public-resource pricing generally.

## What's already known?

- Drake (2023) proposes the simple base design: order-flow auctions per block with proceeds burned, relying on competitive bidding among integrated builders-plus-searchers. The proposal itself acknowledges that competition must be *sufficient* for price discovery — no formal guarantee is given.
- Empirical work (Yang, Nayak & Zhang 2025, arXiv:2405.01329) documents severe builder-market concentration — top builders hold ~90%+ of market share, with entry barriers — so the competitive assumption fails in practice. This motivated treating the problem as collusion/oligopoly-resistant design rather than trusting competition.
- Standard auction theory gives: with $n \ge 2$ non-colluding bidders, first-price/second-price auctions for a common-ish private value yield revenue that lower-binds surplus only under competition; a monopolist bidder bids ~reserve price. Known folk results: any single-shot dominant-strategy mechanism cannot force a single bidder to reveal value.
- Directions tried (informally, in forum discussions on ethresear.ch following Drake's post): ordered-transactions lists, commit-reveal bidding, MEV oracles from realized block value, inclusion lists/PEPC-style enforced commitments. None carries a published manipulation-resistance proof under collusion plus private order flow. (verify — the ethresear.ch thread is the main venue; no peer-reviewed result closes it.)

## What would count as solving it?

Solving = any of the following, clearly stated with model assumptions:

1. **Positive result (construction):** A bid-based burn rule $B(\cdot)$ plus block-award rule and a proof that in every $\epsilon$-equilibrium (with coalitions of size $\le k$), $B \ge \alpha \cdot V$, where $V$ is realized block MEV, for explicit $\alpha, k, \epsilon$. Strategy-proofness for the single-builder case, or a proof it's impossible.
2. **Negative result (impossibility):** A theorem showing no mechanism using only builder bids (and/or observable chain data) can guarantee $\alpha > 0$ against a coalition with private order flow — this would redirect the roadmap to alternative oracles.
3. **Hybrid bound:** A mechanism combining bids with an ex-post observable (e.g., realized block value, inclusion lists) achieving the bound, with formal analysis of gaming the observable.
4. Partial-but-meaningful: characterize exact equilibrium bid functions for the oligopoly case ($n = 2, 3$) under Drake's rule, quantifying suppression loss.

## How to start

1. Read Drake's ethresear.ch post first (short), then the comments — most objections and half-formed fixes are there. Then §4–5 of Yang–Nayak–Zhang for the empirical concentration data that defines the threat model.
2. Formalize the game: builders with private values + exclusive order flow = valuations partly hidden even from other builders; write the first-price MEV-burn auction as a game and solve for $n=1,2$ equilibria. The $n=1$ case is a benchmark: it should fail, calibrating what "competition" must deliver.
3. Import tools from collusion-proof mechanism design (e.g., strategy-proof ranking, bidder collusion in first-price auctions — Graham–Marshall literature) and from credible/commitment auctions. Main obstacle: value is *created by the winner post-hoc* and partially private (exclusive order flow), so no verifiable signal pins it.
4. Explore observables: can the protocol condition burns on realized block contents (e.g., sandwichable-value heuristics, inclusion lists) to break the single-bidder impossibility? That hybrid route is the most promising.

## Key references

- Justin Drake (2023), "MEV burn — a simple design," ethresear.ch post, https://ethresear.ch/t/mev-burn-a-simple-design/15590 — proposes per-block MEV auctions with proceeds burned, assuming integrated builder-searcher competition suffices for price discovery.
- Yang, C., Nayak, K., Zhang, F. (2025), "Decentralization of Ethereum's Builder Market," arXiv:2405.01329 — empirical analysis showing extreme builder concentration and entry barriers, undermining the competition assumption behind MEV-burn.
- (Background, verify) Graham & Marshall (1987), "Collusive Bidder Behavior at Single-Object Second-Price and English Auctions," *J. Econ. Theory* — classic characterization of bidder collusion useful for modeling builder coalitions.
- (Background) Roughgarden, *Twenty Lectures on Algorithmic Game Theory* (2016), Ch. 9–11 (auction design, revenue equivalence) — standard toolkit for the equilibrium analysis.

## Warnings

- "MEV" is not well-defined protocol-side; fix a definition (realized surplus of the proposed block vs. hypothetical maximal) before proving anything, and state it.
- Bids arrive through MEV-boost-style relays with timing games; ignoring latency/commitment issues can invalidate a "strategy-proof" claim.
- Order flow is increasingly private (exclusive searcher-builder deals), so cross-builder competition cannot be assumed even with many builders.
- The ethresear.ch discussion is the actual state of the art; don't assume peer-reviewed literature exists. Verify any claimed result before building on it.
- A mechanism that burns "all MEV" but lets builders route around via off-chain payments to searchers achieves nothing — model off-chain side payments.

---
*`BCGT-000304` · flagged for review*

## Acceptance guidance

Solving = any of the following, clearly stated with model assumptions:

1. **Positive result (construction):** A bid-based burn rule $B(\cdot)$ plus block-award rule and a proof that in every $\epsilon$-equilibrium (with coalitions of size $\le k$), $B \ge \alpha \cdot V$, where $V$ is realized block MEV, for explicit $\alpha, k, \epsilon$. Strategy-proofness for the single-builder case, or a proof it's impossible.
2. **Negative result (impossibility):** A theorem showing no mechanism using only builder bids (and/or observable chain data) can guarantee $\alpha > 0$ against a coalition with private order flow — this would redirect the roadmap to alternative oracles.
3. **Hybrid bound:** A mechanism combining bids with an ex-post observable (e.g., realized block value, inclusion lists) achieving the bound, with formal analysis of gaming the observable.
4. Partial-but-meaningful: characterize exact equilibrium bid functions for the oligopoly case ($n = 2, 3$) under Drake's rule, quantifying suppression loss.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/a102fd01-8dd8-40cf-8f15-053e0e28042c?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000304.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000304)
