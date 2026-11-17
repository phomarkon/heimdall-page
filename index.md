# Heimdall

> Markdown version of the project page at https://phomarkon.github.io/heimdall-page/. Interactive figures are on the web page. Every number here also appears there and comes from the paper or its stored results.

Only the safe shall pass. A conformal verifier between autonomous bidders and the Nordic electricity balancing market

Language-model agents are starting to bid in electricity markets, and none of them can say how often they are wrong. We put a small conformal gate between the bidders and the market. The gate keeps every accepted bid above the operator's loss limit at the coverage the operator asked for, whether or not the model reasoned correctly.

Bachelor thesis in Software Engineering · University of Southern Denmark

[Phongsakon Mark Konrad](https://phomarkon.github.io/)* and [Tim Lukas Adam](https://timadam03.github.io/)* (* equal contribution)

Supervised by Serkan Ayvaz, University of Southern Denmark, and Morten Borch, Danfoss A/S · Sønderborg, June 2026

[Markdown version](index.md) · [Code](https://github.com/phomarkon/heimdall-src) · [Weights](https://huggingface.co/Phongsakon/heimdall) · Cite this thesis

Point at any day after the rule change. Press a calibrator to hide or show it.

Figure 1. Each line is the share of the last 96 quarters (24 hours) whose realised DK1 imbalance price fell inside the calibrator's 90% interval, over the 5,568 quarters from 4 March to 30 April 2025. Split-CP was calibrated before the break. Small marks along the top are the 19 resets of the change-point detector. Stored per-quarter results of the thesis.

### Summary notes

- Over the 5,568 quarters after the 4 March 2025 rule change, a split conformal interval calibrated before the break covered 0.739 of realised prices at a 0.90 target. Online adaptive calibration covered 0.892, and its change-point variant 0.896.

- On the held-out year from May 2025 to April 2026, the focal forecaster F8 with online calibration covered 89.81% ± 0.02% of prices at the 90% target, over five seeds.

- Language-model proposers made 33,597 proposals across 390 runs. The gate accepted and the market filled 6,541 of them, and none realised a loss. For the deterministic chooser, 882 of 3,702 were accepted and filled, also with no loss.

- The language model never beat the deterministic chooser on profit. On the 2 April stress window it matched the chooser's focal profit to the euro, 2,289 EUR, and earned less in total, 24,694 against 31,015 EUR.

- Grounded in its tools, the agent cited unfounded evidence in 0% of its rationales, against 100% when ungrounded, on every frozen seed.

### Abstract

The balancing market is where the Nordic grid settles, every quarter hour, the gap between plan and reality. On 4 March 2025 its rules changed to 15-minute settlement, and two days later the DK1 imbalance price reached 36,013.90 DKK/MWh in a single quarter. We ask what it takes to let a language model bid into this market without putting the operator at risk.

First we wrap a zoo of ten probabilistic forecasters in conformal prediction and prove two theorems that carry coverage of the price over to coverage of realised profit. One is exact in finite samples when past and future are exchangeable. The other holds in the long run on any data stream.

Second we build a two-stage gate that checks physical feasibility and then the worst-case profit over the conformal interval. Through the rule change, online calibration keeps coverage near its 90% target, while a split interval calibrated before the break falls to 74%.

Finally we put three kinds of proposer behind the gate, a deterministic chooser, a single language model and a society of language-model personas. No accepted and filled bid lost money for any of them. The language model does not beat the deterministic chooser on profit. Its value lies in a rationale that can be checked against its own tool calls, and in behaviour that follows plain-language instructions.

### Introduction

On 6 March 2025, two days after the new rules went live, the DK1 imbalance price reached 36,013.90 DKK/MWh in the quarter from 15:00 UTC. That is more than 360 times the day-ahead price in the same hour. A 10 MWh bid placed against a point forecast of 100 DKK/MWh would have lost roughly 360,000 DKK, about 48,000 EUR, in fifteen minutes. A forecaster that had widened its band around that quarter would have told the gate to stand down.

Language models are now proposed as trading agents in exactly this kind of market. They read heterogeneous evidence and explain themselves in prose. They also hallucinate, and their stated confidence is not a calibrated probability. Danfoss, our industrial partner, runs power-to-heat units that turn surplus electricity into district heat and can sell that flexibility to the grid. An operator like Danfoss cannot put such a model on the bid path without a bound on what it can lose.1

In Norse myth Heimdall watches the bridge to Asgard and decides who may cross. Our gate plays that part. It never has to follow the model's reasoning. It checks the consequence of the bid, and that check carries a formal guarantee. The path of one bid through the system is drawn below.

Press a stage to read what happens there.

Figure 2. Solid arrows carry one bid from left to right. The dashed arrow returns the realised price to the calibrator after every quarter, which is what lets online calibration retune itself.

### What would have sunk it

Three results would have refuted the thesis. The interval could have lost its coverage at the rule change, and a split interval calibrated before the break does lose it, while online calibration holds (Sheet 03). The gate could have let a losing bid through, and among 6,541 accepted and filled language-model bids and 882 deterministic ones, none lost money (Sheet 04). The gate could also have been safe only because it refused everything. On a stream of 1,000 aggressive buy bids it accepts all of them at a threshold of 100 EUR, 498 at 250 EUR and none at 500 EUR (Sheet 04). That stream is synthetic, which makes it the weakest of the three tests.2

### Definitions

**mFRR.** Manual frequency restoration reserve, the slowest of the Nordic balancing products. The grid operator activates bids by hand to restore 50 Hz, cheapest first, and pays the marginal price to every activated bid.

**Imbalance price.** The price at which the grid operator settles the gap between a party's plan and its real flow. In western Denmark (DK1) it is set every 15 minutes since 4 March 2025.

**The rule change.** On 4 March 2025 the mFRR Energy Activation Market went live and settlement moved from hourly to 15-minute periods. The thesis treats it as a hard structural break.

**Coverage.** The share of quarters whose realised price falls inside the forecast interval. A 90% interval should cover 0.90 of them.

**Split conformal prediction (split-CP).** Widens an interval by the size of the forecaster's misses on a held-out calibration set. Its guarantee is exact when the future is exchangeable with the calibration days, that is, when no ordering of them is more likely than another.

**Online adaptive conformal inference (ACI).** Retunes the interval after every quarter, wider after a miss and narrower after a hit, so that the long-run miss rate returns to its target on any stream.

**Worst-case profit.** The profit of a bid at the least favourable price inside the interval. Profit moves one way with price, so this is the lower end for a sell bid and the upper end for a buy bid.

**The gate.** The two-stage verifier. It rejects a bid that is physically infeasible for the asset, and then any bid whose worst-case profit is below the operator's threshold *τ*.

**Proposer.** Whatever suggests the bid. In the thesis that is a deterministic chooser, a single language-model agent, or a society of language-model agents with personas.

## A forecast has to say how often it is wrong

A point forecast returns one price. An operator who commits money against it needs to know how far the price may stray, since the cost of being wrong runs mostly one way. So every model in the zoo returns three quantiles per horizon for 1 to 16 quarters ahead. These are the price the market should fall below one quarter in ten, the middle, and the price it should top one quarter in ten.

The zoo has ten entries. They run from a seasonal autoregressive baseline (F0) through gradient-boosted trees (F1), a Bayesian linear model (F2), a deep ensemble and a Monte Carlo dropout network built on patch Transformers (F3, F4), two conformal-wrapped patch Transformers (F7, F8), two zero-shot foundation models (F9, F10) and a domain-shaped surrogate (F11). It is a zoo on purpose. A claim that the conformal wrapper restores calibration for any base forecaster means little unless the bases differ.

Raw quantile heads do not keep their promise. On the validation window the raw band of F7 covers about 0.67 of prices against a nominal 0.80. After conformal wrapping, every entry sits within 0.003 of its target at *α* = 0.1, where *α* is the share of quarters the interval is allowed to miss.

Choose a target. Each dot is one model and one seed.

Figure 3. Horizontal position is empirical coverage on the validation window. The vertical rule is the target 1 − *α*. F12 is an explainable boosting surrogate that the thesis keeps outside the main leaderboard. Stored multi-level results of the thesis.

Accuracy and calibration are different races. Pinball loss scores a quantile by charging more for a miss on the side it promised to guard, and lower is better. On validation the five-seed deep ensemble F3 wins. On the held-out test year, scored once per configuration, the gradient-boosted F1 wins at 182.85 DKK/MWh, against 226.80 for the patch Transformers. F8 is neither, and it remains the focal default because its online calibration is the one that holds through the break.

Choose a window.

Figure 4. Values from the thesis leaderboards. On the test year F7, F8 and F11 share one patch Transformer base and so one point forecast, which is why they tie. Only these four entries have a test pinball in the thesis table.

## Price coverage becomes profit coverage at one endpoint

A covered price is not yet a safe bid. The bridge between the two is monotonicity. Realised profit moves in one direction as the clearing price moves, so over an interval its worst value sits at one end. For a sell bid that is the lower end and for a buy bid the upper end. The gate computes this single number in closed form and compares it with the operator's threshold *τ*.4 Before that, it checks that the bid fits the asset's ramp, capacity, energy balance and the regulatory price cap.

Choose a side, then drag the interval or the threshold.

Figure 5. **Illustration, not a result.** The axes are unitless. The line is the profit of one bid as a function of the clearing price, rising for a sell bid and falling for a buy bid. Only the endpoint marked with a dot is checked against *τ*. In the thesis, a buy bid that would lose 240 EUR at the upper endpoint is rejected this way.

Two theorems carry the price guarantee over to profit. Each proof is short once the endpoint fact is in place.

If calibration and test prices are exchangeable, a split conformal interval contains the next price with probability at least 1 − *α*. Every bid the gate accepts then clears *τ* with probability at least 1 − *α*, jointly over the calibration draw and the price.

Under the online ACI update, with no assumption on the price stream, the long-run share of decision steps at which an accepted bid falls below *τ* is at most *α*, almost surely.

The two statements count different things, and a third rate is easy to confuse with both. Theorem 1a is a probability for one test point. Theorem 1b is a frequency over all decision steps, rejected ones included, since a rejected bid is never placed and cannot lose. The loss rate among the bids that were accepted carries the accepted count in its denominator. It is bounded only by *α* divided by the acceptance rate, so the thesis reports it as a measurement (Sheet 04) and never claims it at *α*.

## The guarantee survives the rule change

On 4 March 2025 the mFRR Energy Activation Market went live across the Nordic zones, and settlement moved from hourly to 15 minutes. Exchangeability fails at such a break by construction. A price after the break could not have appeared before it, so the old calibration days stop standing in for the new ones. Theorem 1a no longer applies there, which is why the gate ships with online ACI.

The hero at the top of this page is the real break. A split interval calibrated before 4 March covers 0.739 of the next 5,568 quarters, and its 24-hour coverage dips as low as 0.385.3 Online ACI covers 0.892. BOCPD-ACI adds a Bayesian change-point detector that flushes the calibration buffer once it is confident the regime has moved, and it covers 0.896. The detector fired 19 times on this window, first on 6 March at 08:30 UTC. Whether BOCPD-ACI carries a formal guarantee of its own is still open.

A synthetic stress test points the same way. On an autoregressive price stream whose gas cost is multiplied by up to 20, split-CP slides from 0.8939 to 0.8826, while ACI stays between 0.8977 and 0.8985.

Choose a gas-cost multiplier.

Figure 6. The bars start at 0.87 so that the drift is visible. The gas cost enters a synthetic autoregressive price stream through the PyPSA-Eur-Sec technology cost stack. Stored stress-test results of the thesis.

The step size *γ* sets how hard each miss moves the calibrator. On the held-out year every setting we stored keeps all four calibrated forecasters within 0.002 of the target. The deployed *γ* = 0.05 sits a little lower, at 0.8981 for F8, and in exchange recovers fastest after a break.

Choose a step size. Each dot is one seed.

Figure 7. The axis spans 0.897 to 0.901 around the target, so the spread between seeds is visible. Stored test-year coverage results of the thesis.

## Nothing the gate accepted lost money

We count the outcome where it matters, on bids that were accepted and then filled by the market. Across every scored run, language-model proposers made 33,597 proposals in 390 runs. The gate and the market let 6,541 of them through, and none realised a loss. On the deterministic side, 3,702 proposals in 43 runs led to 882 accepted and filled bids, and none of those lost either.

Press a proposer to read its counts.

Figure 8. Bar length is the number of proposals, the filled part is the accepted and filled bids, and the marker at the end of the filled part counts those that lost money. Values from the thesis table.

That zero is a measurement on a finite post-break window under a conservative threshold. It sits inside every version of the bound only because its numerator is zero. The theorems own a weaker and more general statement, a long-run breach frequency of at most *α* per decision step, whoever proposes.

A gate that rejects everything is also safe, so we check that this one discriminates. On a synthetic stream of 1,000 aggressive buy bids at *α* = 0.01, it accepts every bid up to *τ* = 100 EUR, 498 at 250 EUR, and none from 500 EUR on. The stream yields uniformly profitable forecasts at low thresholds, which is why the gate only starts to bite near 200 EUR.

Choose an operator threshold *τ*.

Figure 9. Each square is one bid. Filled squares were accepted, outlined squares were rejected on worst-case profit. The counts are stored results, and the order of the squares carries no meaning.

## The language model does not earn more, and it can be audited

Behind the gate we put three kinds of proposer. A deterministic chooser takes the best-ranked candidate bid that the simulator and the gate accept. A single language model, Qwen3-32B served locally, sees the same candidates through narrow tools and picks one, or marks the quarter to watch, or abstains. A society runs six to twenty such agents with personas for a power-to-heat unit, wind, other renewables, a conventional generator, a retailer and an EV aggregator, joined in larger societies by information agents that only advise. Every tool call, rationale and verdict is written to a trace.

Figure 10. Figure 1 of the thesis. The operator dashboard replays one DK1 trading day of the gated society. Nodes are agents coloured by archetype, edges are the links along which they exchange candidate bids, and every accepted and rejected bid can be opened from this one surface.

On profit the deterministic chooser wins every window. The reason is structural, and the thesis calls it the selector trap. Candidates arrive ranked by profit and filtered by the gate, so the best any selector can do is take the top entry, which the deterministic rule always does. Any other pick costs money. On the two stress windows the language model matched the chooser's focal power-to-heat profit to the euro and earned less in total. Averaged over three seeds, the retrieval-augmented society lands within 1 to 3% of the chooser.

Choose a stress window.

Figure 11. Total is the realised profit of the whole society, focal is the power-to-heat agent alone. The two adaptive rows are language-model choosers given more freedom over the menu. Values from the thesis table.

The language model adds two things the chooser lacks. Its rationales can be checked claim by claim against its own tool traces. Grounded in its tools, it cited unfounded evidence in none of them, and ungrounded in all of them. It also follows plain-language policy. Told to mind delivery risk, it cut 28 of 30 variable bids from 2.0 to 1.0 MWh and halved the worst-case delivery tail, a volume control that the price-only gate does not have. The pairs below are verbatim decisions from the same window, agent and step, made once by each chooser.

Choose a pair, or step through them with Previous and Next.

It also fails where one would expect. Its self-reported confidence does not rank realised outcomes, with an AUC below 0.5, while the deterministic control ranks them correctly. Letting it retry rejected bids did not recover profit, and disagreement between agents did not flag wrong-side risk. Safety has to rest on the gate.

Filter by verdict.

Figure 13. Each claim is tested against the deterministic or matched control that could refute it. Positive and negative verdicts are reported alike. Values from the thesis scorecard.

## What the thesis shows, and what it does not

- Coverage of a price interval carries over to coverage of realised profit, exactly under exchangeability and in the long run on any stream (Sheet 02).

- Online ACI held its 90% target through the 2025 rule change in DK1, where a split interval calibrated before the break did not (Sheet 00).

- No accepted and filled bid lost money in the scored corpus, for every class of proposer (Sheet 04).

- A language model behind the gate adds rationales that can be audited and behaviour that follows instructions (Sheet 05).

- Anything about a fleet. One asset is physically grounded, the Danfoss power-to-heat unit, in one zone (DK1) and one product (mFRR). The other archetypes run on stylised envelopes.

- Anything after April 2026. The test year runs from 1 May 2025 to 30 April 2026.

- Live operation. The gate meets its 100 ms p99 latency budget in the test harness, and there is no production Leanheat integration, no Energinet certification and no third-party audit of the worst-case calculation.

- Realistic market profits. Profit figures come from a simplified replay, and several society results rest on a single seed.5

### Next steps

The first step is a one-week shadow pilot on a single district-heating installation, with the gate in advisory mode and a human operator approving every bid. After that come the faster aFRR and intraday products, where the calibrator has to update within a tighter latency budget, and a formal coverage statement for the change-point variant.

### Reproduce

The code is an Apache-2.0 workspace with nine services and seven shared packages, released with the forecaster checkpoints and a data bundle under CC-BY-4.0 that rebuilds from public sources. The test set sits behind a ledger that refuses a second evaluation of any configuration.

### Notes

- The thesis grounds one such asset physically, a Danfoss power-to-heat unit with 50 MW of electrical capacity and a 100 MWh thermal buffer. Its opportunity cost is the gas-boiler price divided by the conversion efficiency. ↩

- The thesis also replays the gate against injected hallucinated bids over the full six-year record, reported in its appendix. ↩

- Before the rule change Energinet published the DK1 imbalance price hourly, so the pre-break target is forward-filled onto the 15-minute grid and is piecewise constant within each hour. The thesis flags this as a limitation. ↩

- Prices are in DKK/MWh, the settlement unit. Bid prices, profit and the threshold are in EUR at the fixed rate of 7.46 DKK per EUR. ↩

- Key configurations were repeated over seeds 13, 42 and 137 and showed no material change, so the thesis treats single-seed extensions as indicative. ↩

## Cite this thesis

Konrad, P. M. and Adam, T. L., 2026. Heimdall: Only the Safe Shall Pass. A Conformal Verifier Between Autonomous Bidders and the Nordic Electricity Balancing Market. Bachelor's thesis, University of Southern Denmark.

```bibtex
@thesis{konrad2026heimdall,
  title   = {Heimdall: Only the Safe Shall Pass. A Conformal Verifier
             Between Autonomous Bidders and the Nordic Electricity
             Balancing Market},
  author  = {Konrad, Phongsakon Mark and Adam, Tim Lukas},
  type    = {Bachelor's thesis},
  school  = {University of Southern Denmark},
  address = {S{\o}nderborg, Denmark},
  year    = {2026},
  url     = {https://phomarkon.github.io/heimdall-page/}
}
```

Every plotted value comes from the thesis or from the stored results in its public code release. The one exception is the gate figure on Sheet 02, which is labelled as an illustration. Bachelor thesis in Software Engineering at the Mærsk Mc-Kinney Møller Institute, University of Southern Denmark, Sønderborg, with Danfoss A/S as industrial partner.

Supervised within the [Applied and Interpretable Machine Learning Research Group](https://aiml-sdu.github.io) at the University of Southern Denmark.
