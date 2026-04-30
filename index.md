---
title: "Neuro-Modulating Architecture Priors (NMAP) for Context-Dependent Reconfiguration of Locomotory Circuits in Caenorhabditis elegans"
abstract: |
    *Caenorhabditis elegans* switches between distinct crawling and swimming gaits via extrasynaptic dopamine and serotonin signalling, yet existing neuro-inspired architectures such as Neural Circuit Architectural Priors (NCAP) encode only the hardwired synaptic layer with fixed weights, precluding context-dependent reconfiguration. We introduce **Neuro-Modulating Architecture Priors (NMAP)**: a three-layer extension that pairs the NCAP central pattern generator with a GRU context encoder emitting a dopamine/serotonin-analogue modulatory vector and a hierarchical manager that infers substrate from proprioceptive phase-lag alone. Trained with PPO in a progressive water–land MuJoCo environment, NMAP produces a bistable amplitude–frequency separation, four discrete oscillator-period clusters tracking the curriculum, and qualitatively distinct kymographs per substrate. NCAP, under the same curriculum, collapses toward a single compromise gait. Neuromodulatory priors are a necessary architectural extension beyond hardwired connectivity for context-adaptive embodied AI.
acknowledgments: |
    This work was supported by the Neuromatch Impact Scholar Program. We thank Dr. Srikanth Ramaswamy (Neural Circuits Laboratory, Newcastle University) for mentorship, and the program sponsors and teaching assistants whose contributions do not meet the criteria of any authorship role.
---

# Description

## Background

Biological locomotion adapts across mechanical contexts through neuromodulation. In *C. elegans*, dopamine is necessary for crawling and serotonin for swimming [@vidalgadea2011]; dopamine-deficient mutants exhibit unstable locomotion rates [@omura2012]. Whereas hardwired synapses support point-to-point communication, neuromodulators act as wireless broadcasts that dynamically reconfigure entire network states [@randi2023] — a distinction that integrated neuromechanical models alone cannot capture [@boyle2012]. Neural Circuit Architectural Priors (NCAP) embed sparse connectivity, sign constraints, and intrinsic dynamics as differentiable priors for embodied control [@bhattasali2022]. While effective for single-substrate locomotion, NCAP captures only the hardwired synaptic layer with fixed weights and cannot reconfigure motor output across mechanical contexts. We close this architectural gap with **Neuro-Modulating Architecture Priors (NMAP)**, in which a learned modulatory layer sits on top of NCAP and a hierarchical manager selects gait from proprioception alone.

## Methods

**Simulation.** All experiments used the `dm_control` physics suite [@tunyasuvunakool2020] with a six-link, five-joint swimmer modelled on *C. elegans*. We used two substrate regimes: an aquatic baseline at viscosity $\mu = 0.001$, and a progressive mixed substrate where circular land zones at $\mu = 0.05$ were introduced in a four-phase curriculum (pure water, then one, two, and four islands). The agent received no explicit zone-detection signal; effective viscosity updated each step from head position relative to zone boundaries.

**NMAP architecture.** Layer 1 preserves the standard NCAP central pattern generator (CPG) unchanged. Layer 2 adds a *context encoder* — a GRU that consumes only the inter-joint phase-lag signal (consistent with the absence of viscosity sensors in *C. elegans*) and emits a two-dimensional context vector analogous to dopamine and serotonin signalling, which produces per-joint gain and bias parameters applied at each CPG step. The effective oscillator period is modulated as a sigmoid function of the context vector, enabling smooth swim-to-crawl transitions. Layer 3 introduces an HRL *manager* that observes accumulated mechanical context and issues a continuous mode signal to the encoder, implementing a timescale separation between gait selection and locomotion.

**Training.** All models were trained with PPO for $3 \times 10^{6}$ environment steps. The base reward is a tolerance function on the swimmer head's forward velocity. Experiment 3 (NMAP) added a transition bonus on confirmed substrate crossings and a mismatch penalty for applying the wrong gait, both annealed from zero over the first 30 % of training. We compared (1) NCAP on a single aquatic substrate, (2) NCAP on the progressive curriculum, and (3) NMAP on the progressive curriculum.

## Results

NMAP produces measurable bistable gait switching that matches *C. elegans* kinematics; NCAP, under identical curriculum and reward, fails to differentiate motor output by substrate across every diagnostic.

**Amplitude–frequency separation (@figure-main A,B).** NMAP is the only model showing two separated kinematic clusters with a bistable gap at 1.1–1.5 Hz. NCAP under the same progressive curriculum produces a continuous mediocre scatter with no distinct separation between swim and crawl regimes.

**Substrate-specific forward speed (@figure-main C,D).** NMAP maintains a near-constant water–land speed gap of $\sim$0.007–0.020 m/s reflecting true substrate specialisation. NCAP's gap converges toward zero through mediocrity, with both substrates approaching the same compromise speed of $\sim$0.175 m/s.

**Body-wave morphology (@figure-main E,F).** NMAP produces fast short-period waves in water ($\sim$20–25 steps) and slow long-period waves on land ($\sim$50–60 steps) across all curriculum phases. NCAP produces identical swim-like wave morphology on water and land — zero gait adaptation.

**CPG period commitment** (described here, not figured): NCAP on a single substrate locks to $\sim$59 steps; under the progressive curriculum it collapses to a broad unimodal distribution shaped by mixed terrain. NMAP shows four discrete period clusters, one per curriculum phase, reflecting graded modulation that tracks curriculum complexity.

## Ablations

We isolated the contribution of each architectural component with three ablations of NMAP under identical training conditions. **Removing neuromodulation** (gain = 1, bias = 0) contracts the land-crawl wave period toward swim frequency across all phases; switching degrades but is not eliminated, because the HRL manager retains residual modulation capacity. **Disabling the bistability regulariser** allows context vectors to adopt continuous intermediate values rather than committing to discrete attractors; discrete gait commitment collapses, the reward profile reverts to an NCAP-like pattern, and water performance drops by 16 % — making this the single most critical component. **Removing anisotropic drag** reduces substrate contrast to viscosity alone; switching persists via viscosity contrast, but the crawl cluster over-amplifies, degrading biological correspondence without eliminating gait differentiation.

```{figure} figure.png
:name: figure-main
:alt: Six-panel figure contrasting NCAP and NMAP on the progressive water-land curriculum.

\
**A.** NCAP amplitude–frequency under the progressive curriculum: continuous scatter, no swim/crawl separation.
\
**B.** NMAP amplitude–frequency: two distinct clusters with a bistable gap at 1.1–1.5 Hz (solid ellipses mark the swim and crawl regimes).
\
**C.** NCAP forward speed by curriculum phase: water–land gap collapses toward zero as phases progress.
\
**D.** NMAP forward speed by phase: a stable substrate-specific speed gap is preserved across the curriculum.
\
**E.** NCAP kymographs (final phase, water and land substrate): identical swim-like body waves on both substrates.
\
**F.** NMAP kymographs (final phase, water and land substrate): fast short-period swim waves in water; slow long-period crawl waves on land.
```

## Conclusion

NMAP demonstrates that a learned dopamine/serotonin-analogue modulatory layer, coupled with a hierarchical manager that infers substrate from proprioceptive phase-lag alone, is sufficient to produce bistable swim–crawl transitions that quantitatively match *C. elegans* kinematics. NCAP, despite an identical CPG and curriculum, cannot. Ablations identify the bistability regulariser as the single most critical component, with neuromodulation and anisotropic drag providing complementary contributions to switching precision. **Neuromodulatory priors are a necessary architectural extension beyond hardwired connectivity for context-adaptive neuro-inspired AI.**
