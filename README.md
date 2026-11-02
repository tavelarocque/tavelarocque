# Hey, I'm Tave 👋

I'm an **Engineering student at UBC** (Integrated Engineering, computer + finance) working on **quantitative research** and **AI evaluation**.

What ties my work together is that the results can be checked. Hypotheses are written down and registered before the data that decides them is touched. Every important number is computed twice, by implementations written independently. Planted bugs prove the tests can fail. Final runs reproduce bit for bit, and on the chip, the hardware is formally proven equal to a separate spec.

Right now that means a [pre-registered test of order-driven volatility](https://github.com/tavelarocque/itchbook#research-3-a-pre-registered-test-of-order-driven-volatility) on NASDAQ order-book data, a [protocol emulator chip](https://github.com/tavelarocque/protocol-emulator) for Jane Street's ASIC competition, and [exact computations](https://github.com/tavelarocque/spr-calibration-code) on an open problem in online calibration.

---

## 🔬 Research and Projects

### ⚡ [itchbook: NASDAQ order book engine and three microstructure studies](https://github.com/tavelarocque/itchbook)

A C++20 engine that rebuilds NASDAQ's full order-by-order (L3) book from raw TotalView-ITCH 5.0 data, 423 million messages a day, at **92 ns per message**. That's **3.3× faster** than CppTrader's L3 book on the same machine, and the output is **byte-identical** to an independent Python reference. The rebuilt book is the ground truth for three studies:

- **Queue position.** Across **7.5M orders**, 85% of cancelled volume comes from *behind* an order, so the L2 queue models backtests use overfill. Behind 10–24 orders, 1 in 6 of the uniform model's simulated fills never happened.
- **Forecasting.** On 7 out-of-time days, DeepLOB beats a multi-level order-flow-imbalance baseline (macro-F1 **0.53 vs 0.45**). But no model survives the spread and fees: 0.1–0.4 ticks of edge against at least 1.6 ticks of cost.
- **Order-driven volatility (pre-registered).** An out-of-sample test of Maitrier and Bouchaud (2026) on 8 NASDAQ days, [registered on OSF](https://osf.io/z4t7x) with the code frozen at a git tag before the held-out stocks were used.
  - In one Holm family of 8 tests, P-F is rejected in both tick-size bins, which is evidence against the informed-flow regime and nothing more. The other three predictions are not rejected.
  - Power studies run before the freeze showed why a single venue's data can't settle them.
  - The confirmatory run reproduces bit for bit.

`C++20` `CMake` `GoogleTest` `RapidCheck` `libFuzzer` `Python` `NumPy` `PyTorch` `Pre-registration`

[GitHub](https://github.com/tavelarocque/itchbook) · [Walkthroughs (PDF)](https://github.com/tavelarocque/itchbook/tree/main/docs) · [Pre-registration](https://osf.io/z4t7x)

---

### 🧪 [ImpossibleBench extraction study: does a scoring detail change a cheating benchmark?](https://github.com/tavelarocque/impossiblebench-extraction-study)

[ImpossibleBench](https://github.com/safety-research/impossiblebench) measures how often LLMs cheat by defeating tests that cannot honestly be passed. [PR #6](https://github.com/safety-research/impossiblebench/pull/6) changes which code block in a model's reply gets executed, and that choice can hide a cheat behind an honest attempt.

**Method:**
- The plan was pre-registered before the first model call.
- **515 Qwen3.5-9B episodes** were generated and re-scored under 4 extraction rules × 2 exit rules.
- Every run used a locked-down sandbox: no network, read-only filesystem.
- Nothing was reported until three gates passed:
  - a fixture suite;
  - 22 planted known-answer cases;
  - replay matching Inspect's own scores on 509 of 512 episodes.

**Result:** the loophole is real in principle but absent in practice for this model. 0 replies used the pattern, and the rules differ by −0.5 points (95% CI [−1.5, 0.0]).

`Python` `Inspect` `Docker` `LLM Evaluation` `Pre-registration` `McNemar`

[GitHub](https://github.com/tavelarocque/impossiblebench-extraction-study) · [Report](https://github.com/tavelarocque/impossiblebench-extraction-study/blob/main/results/qwen3.5-9b/report.md)

---

### 🔌 [protocol-emulator: a programmable protocol chip](https://github.com/tavelarocque/protocol-emulator)

An ASIC for [Jane Street's protocol emulator competition](https://blog.janestreet.com/protocol-emulator-asic-competition/), written in **Hardcaml** and being taped out through Tiny Tapeout on IHP 130nm. It's a tiny two-thread CPU modelled on the RP2040's PIO: its instructions read pins, write pins and count cycles, so UART, SPI and I2C live in firmware loaded after fabrication.

- It runs all three protocols, two at a time, including an I2C-to-SPI bridge with no host in the data path.
- It can also act as the I2C target or SPI device for outside controllers.
- Its firmware runs on the post-layout gate-level netlist.
- The whole chip, pins to pins, is **formally proven equivalent** to an independently written spec, with a SAT proof in Hardcaml_verify and Z3. Bugs planted in the spec break the proof, so the proof isn't vacuous.

`OCaml` `Hardcaml` `Z3` `cocotb` `Tiny Tapeout` `IHP 130nm`

[GitHub](https://github.com/tavelarocque/protocol-emulator)

---

### 📊 [Implied Odds: prediction markets vs. options](https://github.com/tavelarocque/implied-odds-research)

A live research system comparing Kalshi's S&P 500, Nasdaq-100 and Bitcoin contracts with the probabilities implied by option prices: Breeden–Litzenberger on arbitrage-checked SVI smiles, with Deribit for Bitcoin.

- A cloud pipeline on GitHub Actions collects Cboe, Kalshi, Polymarket and Deribit data around the clock.
- It paper-trades the gap under fixed rules, re-reading the order book after every trade to check the trade would really have filled.
- Every test is **pre-registered** before its data exists, judged on a locked holdout, and adjusted in a **multiple-testing ledger** (Holm / BHY).

**Results so far:** Kalshi's S&P contracts are well calibrated, and a Kalshi + realized-volatility blend beat Kalshi's own price on the sealed holdout.

`Python` `NumPy` `pandas` `SciPy` `Options Pricing` `Multiple Testing` `GitHub Actions`

[GitHub](https://github.com/tavelarocque/implied-odds-research)

---

### 🐫 [ocaml-contracts: a contract language for prediction markets](https://github.com/tavelarocque/ocaml-contracts)

A financial contract language in OCaml, after Peyton Jones, Eber and Seward's *Composing contracts*, that prices Kalshi's Bitcoin contracts from the distributions implied by Deribit options.

- Every contract is a tree of nine combinators.
- Contracts are valued by **three independent pricers** that check each other: density integration, Carr–Madan replication and a Rubinstein implied tree. They match Implied Odds' Python probabilities to **2.7 × 10⁻¹²** across 13,804 contracts.
- On top: bump-and-reprice Greeks, a delta-hedging simulation, an **Avellaneda–Stoikov market maker** adapted to $0/$1 payouts, and a markout system for measuring order-flow toxicity.

`OCaml` `Core` `dune` `Options Pricing` `Market Making` `Property Testing`

[GitHub](https://github.com/tavelarocque/ocaml-contracts)

---

### ♠️ [holdem-cfr: game-theory solver for no-limit hold'em](https://github.com/tavelarocque/holdem-cfr)

A counterfactual regret minimization solver for heads-up no-limit hold'em, built from scratch in Rust. It has the same pieces as Libratus and Pluribus, at laptop scale:

- a CFR engine verified against exact best responses (**4.9 × 10⁻⁶** exploitability on Leduc hold'em);
- a 7-card hand evaluator checked on all **133.8M** hands;
- a multithreaded postflop solver that solves a 293,000-node flop tree to 0.33% of pot in **31 s**;
- an abstracted strategy trained for **1.5 billion** Monte Carlo CFR iterations.

The trained bot runs in the browser.

`Rust` `rayon` `proptest` `Game Theory` `Monte Carlo CFR`

[Project page](https://tavelarocque.github.io/holdem-cfr/) · [Play the bot](https://tavelarocque.github.io/holdem-cfr/web/table/) · [GitHub](https://github.com/tavelarocque/holdem-cfr)

---

### 🎲 [spr-calibration-code: exact computations for an open problem in calibration](https://github.com/tavelarocque/spr-calibration-code)

Independent research on the optimal rate of sequential calibration error, an open problem in online prediction. The best known bounds pass through the sign-preservation-with-reuse game of Dagan et al. (STOC 2025), and this code computes that game exactly.

- **Exact game values.** A memoised minimax solver in Rust computes exact values of the full game up to 18 cells, the forced-erasure game up to 20, and the no-reuse game up to 22. It is cross-checked against brute force and an independent Python reference.
- **Certified bounds for oblivious strategies.** Each lower bound is the exact rational value of a stored strategy, checked by two independent verifiers. Each upper bound is an exact Farkas certificate.

`Rust` `Python` `Linear Programming (HiGHS)` `Exact Arithmetic` `Online Learning`

[GitHub](https://github.com/tavelarocque/spr-calibration-code)

---

## 🏗️ Outside Research

**💪 [MODUX](https://modux.ca): training app for hybrid athletes.** A native iOS app for athletes who both lift and run, built end to end and submitted to the App Store.
- Its **Interference Score** models how strength and endurance training conflict, using an 11-dimensional fatigue vector with 72-hour decay, shown as a 0–100 readiness score.
- It syncs with Strava and Apple HealthKit, and fuzzy-matches workouts across the two so none is counted twice.

`React Native` `Expo` `TypeScript` `Supabase Postgres` `HealthKit`

**🌐 [Current at UBC](https://currentatubc.com): student-to-industry event.** I co-founded it, and it's launching **April 2027**: **300+ students**, **3 keynote speakers** and **50 Vancouver industry leaders** in one room, with all net proceeds going to BC Children's Hospital Foundation. I built the whole production stack: the site, automations, CRM, DNS and email.

**Earlier projects:**
- [Stock Chart CNN](https://github.com/tavelarocque/stock-chart-cnn): candlestick-image classifier with a Streamlit app.
- [Taskboard](https://github.com/tavelarocque/kanban-app): full-stack kanban app.
- [SkyWatch](https://tavelarocque.github.io/weather-dashboard): weather dashboard.
- [Particle Physics Simulator](https://tavelarocque.github.io/particlephysics): browser physics sim.

---

## 🛠️ Toolkit

![C++](https://img.shields.io/badge/C%2B%2B20-00599C?style=flat&logo=cplusplus&logoColor=white)
![OCaml](https://img.shields.io/badge/OCaml-EC6813?style=flat&logo=ocaml&logoColor=white)
![Rust](https://img.shields.io/badge/Rust-000000?style=flat&logo=rust&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat&logo=typescript&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-4169E1?style=flat&logo=postgresql&logoColor=white)

- **Also:** Java · C · Hardcaml (RTL) · PyTorch · NumPy / pandas / SciPy · React / React Native · Node.js · Supabase · GitHub Actions
- **Verification:**
  - pre-registration and locked holdouts;
  - Holm / BHY multiple-testing corrections, bootstrap and jackknife inference;
  - property tests (RapidCheck, proptest, Quickcheck) and planted-bug (mutation) tests;
  - fuzzing under ASan / UBSan / TSan;
  - formal equivalence proofs;
  - bit-identical reproduction.
- **Quantitative methods:** SVI smiles · Breeden–Litzenberger densities · Carr–Madan replication · implied binomial trees · Black-76 · Greeks · Avellaneda–Stoikov market making · markouts and order-flow toxicity
- **Market microstructure:** L3 book reconstruction from TotalView-ITCH · queue-position models · multi-level OFI · DeepLOB · propagator models · cost-aware evaluation
- **Game theory:** CFR, CFR+, Linear / Discounted CFR, Monte Carlo CFR · exact and local best response · abstraction
- **AI evaluation:** Inspect · sandboxed execution of model-written code · re-scoring and replay gates
- **Math:** Calculus I–III · Linear Algebra · probability and statistics
- **Spoken languages:** English (native) · Japanese · French (basic) · Spanish (basic)

---

## 🎓 About Me

- 📐 Engineering student @ **University of British Columbia** (BASc, expected 2028)
- 💼 **Co-Founder & CEO** @ [Current at UBC](https://currentatubc.com)
- 📱 **Co-Founder & Lead Engineer** @ [MODUX](https://modux.ca)
- 🏦 **Indigenous Project Assistant Intern** @ Vancity: AI automations for Corporate Real Estate (May–Aug 2026)
- 📋 **Director of Philanthropy** @ KS Society. I was promoted from Recruiter in 4 months after leading External Recruitment, and I now own philanthropic strategy, partner outreach and event programming.
- 🛠️ Built microcontroller systems, CAD-designed 3D-printed devices, and a full rainwater harvesting system
- 🏔️ Former ski patrol volunteer · 💪 Certified Weight Lifting Trainer and varsity athlete

---

## 📫 Let's Connect

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat)](https://linkedin.com/in/tave-la-rocque-56967a329)
[![Email](https://img.shields.io/badge/Email-D14836?style=flat&logo=gmail&logoColor=white)](mailto:tave2000@icloud.com)

*🟢 Open to quantitative research, AI evaluation and software roles: Summer 2027 internships and full-time opportunities, in Vancouver, BC or remote.*
