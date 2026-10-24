# Research basis

Source: RL project research, 16 October 2026. Extract preserved from the report; references were not revalidated during delivery publication on 24 October 2026. Product potential is a hypothesis. The execution plan is the narrower delivery contract if timing or scope differs.

## 3. FlowForge — a multi-site fulfillment digital twin and dispatch decision service

### Identity and user problem

**One sentence:** FlowForge is a local-first digital twin for small distribution operations that tests order release, wave batching and worker/vehicle dispatch policies against automatically generated demand, capacity limits and disruptions.

The target is a small warehouse operator, logistics software integrator or operations researcher who wants to quantify service/cost trade-offs before changing scheduling rules. The product would accept a documented facility layout and demand profile, run a discrete-event simulator, compare heuristics/optimization/RL and export a decision report. The initial value proposition is **decision support and what-if analysis**, not autonomous control or guaranteed savings. Unlike single-item reorder optimization, the ambitious question is coordinating tasks, queues and constraints across a facility/network.

**Why now:** warehouse and fleet systems have many interacting discrete decisions, while simulation allows stress testing without collecting proprietary data. RL can learn dispatch policies in synthetic episodes, but operations research baselines may be stronger, cheaper and easier to explain. A production-quality product must earn its place by making the comparison, constraints and integration simple—not by assuming RL is the winning method.

### Evidence and source foundation

- Mao et al., [Resource Management with Deep Reinforcement Learning](https://people.csail.mit.edu/alizadeh/papers/deeprm-hotnets16.pdf), 2016: primary paper on DeepRM scheduling, showing resource allocation as a learned sequential decision problem in a simulated cluster. It is methodological inspiration; warehouse operations have different constraints and require new validation.
- [OR-Tools official documentation](https://developers.google.com/optimization), current docs accessed 16 October 2026: provides routing, linear/integer optimization and constraint programming references; use it as a serious non-RL baseline. Record the version in the environment lockfile.
- [SimPy documentation](https://simpy.readthedocs.io/en/latest/), current docs accessed 16 October 2026: process-based discrete-event simulation; appropriate for CPU-local synthetic facility scenarios.
- [Warehouse and Retail Research (WARs) benchmark / RWARE](https://github.com/semitable/robotic-warehouse): open multi-agent warehouse environment and benchmark code (project repository; inspect its license and paper citation before adoption). It is a useful comparison environment, not a validated customer digital twin.
- If using public sales demand, the [M5 Forecasting competition page](https://www.kaggle.com/competitions/m5-forecasting-accuracy) provides Walmart item/department sales data subject to competition site terms and registration. Do not bundle the downloaded files without checking current terms. The MVP can instead generate demand automatically with configurable seasonality, promotions, correlation and outages, thus avoiding data access as a dependency.
- Traffic management is a credible alternative domain if the user later prefers it: SUMO’s official [`randomTrips.py` documentation](https://sumo.dlr.de/docs/Tools/Trip.html), last edited 12 October 2026, describes seeded, repeatable synthetic trips and configurable arrival rates; [TraCI](https://sumo.dlr.de/docs/TraCI/) exposes online control of a running simulation. A traffic-signal controller could therefore avoid hand-curated traffic data too. It is not included as a fourth project because local deployment to a city is a long public-sector sales/integration path, while the supply-chain concept provides a clearer solo product workflow.
- [Gymnasium](https://gymnasium.farama.org/) can expose a well-defined episodic interface; the actual simulator should remain independent from the RL library so exact scheduling baselines can use the same event engine.

### Product concept and architecture

**Initial workflow:** select one canonical warehouse template; set SKU/order arrival distribution, storage zones, pick/pack capacity, service deadlines, labor or fleet constraints and disruption schedule; run baseline policies; compare an RL policy; view costs, throughput, missed deadlines, queue tails and resource use; export scenario and result bundle. The input can start as a local YAML file or guided form. Avoid scraping maps or making users manually prepare a dataset: use parameterized scenario generation and example config templates.

**Architecture:**

1. Event engine (SimPy or custom heap-based discrete event loop) owns time, queues, resources and fulfillment state.
2. Scenario generator creates facility topology, demand streams, processing-time distributions, shift calendars and disturbances from seeded configs.
3. Policy API supports heuristic dispatch, OR-Tools optimization over short horizons, and RL (PPO or discrete-action value learning; offline learning only if logged data exists).
4. Constraint layer checks capacity, deadlines, resource availability and service priorities. The simulator rejects invalid actions and records rejection reason.
5. Evaluation runner executes paired scenarios, seeds and sensitivity sweeps; result store captures raw episodes and manifests.
6. Local FastAPI/CLI with HTML report. Add interactive facility visualization only after numerical results and config workflow work.

For the 4 GB card, keep state vectors small and train CPU-first; a small MLP is sufficient. Avoid LLMs, image input and large multi-agent networks. Add one warehouse area and one decision point at first. Extend to multi-site coordination only after a stable baseline engine exists.

**Data and governance:** generated event streams mean no manual datasets. Use a seedable generative demand model with explicit assumptions; optionally add M5 as an import adapter after terms review. Do not claim generated demand resembles any specific company. If user data is ever added, use local-only storage, schema validation, redacted logs and clear retention/export controls. Keep example scenarios entirely synthetic and licensed under the repository’s terms. For borrowed simulator code, note attribution and license.

**Evaluation:** baselines should include FIFO/EDD, nearest-task or zone heuristics, a known dispatch rule, and a rolling-horizon OR-Tools solution on small instances. Compare at common seeds and identical environment conditions. Evaluate throughput, on-time fill, lateness distribution, travel/waiting time, overtime/energy proxy, starvation/fairness, constraint failures, inference cost and solve time. Test perturbations: demand surges, absences, blocked aisles, machine downtime, processing-time shifts and forecast error. Report mean and tail (e.g. 95th percentile delay) plus variability. Use ablations for state/reward components. RL has product relevance only if it improves a meaningful cost/service metric at a defensible latency/compute cost or offers flexibility where optimization fails. Be prepared for the result that a simple rule wins.

### Execution and feasibility

**Difficulty:** high (discrete-event correctness, competing objectives, policy evaluation and product UX). **Dependencies:** Python and local simulator only. **Bottleneck:** credible simulator assumptions and a comparison customers trust. One person can deliver a generated-data demonstration. A real product later needs operator workflow validation, integration expertise and permission to test on representative operations; no interviews are needed to begin, but they are needed before a serious product bet.

- **Months 1–2:** write simulator specification and data contract; generate orders and disruptions; implement FIFO/EDD baselines and conservation/invariant checks; create first scorecard.
- **Months 3–4:** public alpha with one facility, one dispatch decision, synthetic scenarios, tests and recorded demo. First result is a usable simulator and report, not a claim of RL advantage.
- **Months 5–7:** add rolling-horizon OR baseline and one RL algorithm; paired evaluation, distribution shifts and detailed reproducibility; profile runtime on target laptop.
- **Months 8–10:** add configuration UX, import/export schemas, scenario comparison and policy recommendation explanation; test failure recovery and data validation.
- **Months 11–12:** release v1.0; publish an example with synthetic inputs and a clean-clone run; perform a small, voluntary workflow review with operators or integrators if accessible. Only after evidence should pricing, managed deployment or integrations be scoped.

**Stop/reduce gate:** if event semantics or scenario generator remain unstable by month 3, freeze to a single decision point and smaller layout. If OR-Tools consistently matches or beats RL with simpler operations, pivot the product to a simulator/benchmark and keep RL as an optional plugin. If no user group can be reached by month 12, keep it as a strong portfolio tool rather than claim product-market fit.

### CV, product and risk

Artifacts: tested discrete-event simulator, schema-driven scenario generator, baseline and RL policy adapters, OR-Tools comparison, local API/report, runtime benchmarks, generated scenario packs, recorded demo, deployment guide and an honest decision memo. CV template: “Built a local-first fulfillment digital twin generating **[N]** reproducible scenarios; compared **[heuristics/OR/RL]** over **[K]** seeds and **[S]** demand shifts, measuring **[service metric]**, **[cost proxy]** and **[runtime]**.”

**Product potential: moderate, unvalidated.** Potential segments are small warehouses, 3PLs, fulfillment software vendors, and logistics consultancies. Their pain is evaluating operational changes with interacting constraints; value depends on faster scenario analysis and better service/cost tradeoffs. Commercial paths: open-source core plus paid integrations/support; per-site desktop or on-prem license; consulting to map an operator’s process; a hosted offering only if compute/data governance supports it. Keep simulation core open if trust and extensibility drive adoption. Monetize facility templates, reporting, integrations and support only after users ask for them. No market-size number is asserted because this research did not verify one.

Risks: a toy simulation produces confident but wrong advice; reward tuning encodes unfair service priorities; generated distributions do not capture actual operations; RL may underperform transparent heuristics; local UX may not fit WMS workflows; and integration/security requirements can overwhelm a solo product. Mitigate with assumptions-first configs, robust sensitivity tests, transparent baselines, constraint checks, immutable run provenance and explicit “simulation only” labels. **Verdict:** the strongest first project for this user’s constraints and product goals, provided the initial promise is a credible operations simulator rather than autonomous warehouse control.
