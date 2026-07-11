# World-Cup-2022-Latent-Profile-Analysis
Model-based clustering (tidyLPA/mclust) of 576 World Cup players on 6 game based metrics. Why use a Latent Profile Analysis? I chose it because cluster count and covariance structure are selected by fit indices (BIC, bootstrapped LRT, entropy) instead of guessed. This means that assignments come with posterior probabilities. The model recovered 5 interpretable roles without ever seeing positions.


## TL;DR
 
- Latent Profile Analysis of **576 outfield players** (≥180 minutes) at the 2022 World Cup, built from **StatsBomb event-level data** across all 64 matches, identified **five distinct player profiles**: Clinical Finishers, Pressing Engines, Ball-Playing Defenders, Limited Role Players, and Balanced Contributors.
- The model was never told a player's position, yet it rediscovered recognizable footballing roles: the finisher profile (9% of players) contains Messi, Mbappé, Giroud, and Julián Álvarez; the rarest profile (3.1%) is elite ball-playing defenders; Stones, Varane, Maguire, Aké.
- One hypothesized type failed to appear: no distinct "Creative Playmaker" profile emerged. Chance creation turned out to be a feature of several player types rather than a type of its own.
---
 
## Motivation
With a passion for soccer and people analytics, I saw an opportunity to apply methods across industry. Thie project is meant to act as a cross-walk for how an LPA might be applied in a sporting context. Given that soccer is a complex game that is often reduced to averages, I wanted to apply a more rigorous methodology to help make sense of the underlying data. 

In people analytics, variable-centered methods (regression, factor analysis) describe the *average* person: "does shooting accuracy predict goals?" However, person-centered methods like LPA ask: "**what kinds of people exist in this population?**" This mirrors a long line of IO psychology research on engagement profiles, burnout typologies, and team composition (Humphrey et al., 2007; Spurk et al., 2020). Looking at the data through this lens is also more useful question whenever teams rely on complementary types rather than interchangeable averages.
 
A World Cup is an unusually clean opportunity to deploy the method because it has a fixed elite talent pool, a shared performance context, and objective event-level measurement.
 
**Research questions**
 
1. How many distinct performance profiles exist among 2022 World Cup outfield players, and what characterizes them?
2. Do the profiles recovered by the model correspond to substantively meaningful footballing roles, and do they match what was hypothesized in advance?
---
 
## Data & Indicators
 
**Source:** StatsBomb Open DatA: This source contains every pass, shot, carry, pressure, and duel from all 64 matches. Player minutes were reconstructed from Starting XI and substitution events, capped at 90 per match (extra time excluded for opportunity consistency).
 
**Inclusion:** The analysis include players that logger ≥180 minutes played and ≥10 passes attempted. The analysis excluded goalkeepers. n = 576.
 
Six per-90 indicators spanning attack, creation, progression, defense, and technical reliability, each z-scored on the sample:
 
| Indicator | Definition |
|---|---|
| npxG/90 | Non-penalty expected goals |
| xAG/90 | Expected goals assisted |
| Progressive carries/90 | Carries advancing ≥~9 yards toward goal |
| Pressures/90 | Defensive pressing actions |
| Pass completion % | Completed ÷ attempted (set pieces excluded) |
| Aerial duels/90 | Contested aerials (see data note) |
 
> **Data note:** StatsBomb logs standalone aerial duels only as "Aerial Lost" events (wins are attributes of other events), so the aerial indicator undercounts won aerials and is best read as *aerial involvement*.
 
---
 
## Method: Model Enumeration & Selection
 
LPA solutions with 1–6 profiles were fit via `tidyLPA`/`mclust` under three variance-covariance specifications (equal variances; varying variances; varying variances + covariances), following the model-selection framework standard in organizational research (Spurk et al., 2020; Nylund et al., 2007): BIC, entropy > .80, bootstrapped LRT, minimum profile size, and theoretical interpretability.
 
| Specification | k | BIC | Entropy | Smallest profile |
|---|---|---|---|---|
| Equal variances (M1) | 5 | 9,128 | .81 | 3.1% |
| Equal variances (M1) | 6 | 8,884 | .83 | 2.6% |
| Varying variances (M2) | 2 | 8,220 | .89 | 45% |
| Varying var + cov (M6) | 2 | 8,064 | .91 | 46% |
 
**Selected: 5-profile, equal-variance solution.** The fit indices did not agree, which is the norm rather than the exception in applied LPA. The more flexible specifications achieved better BIC but failed to converge beyond k = 2, and a 2-profile solution ("attackers vs. everyone else") is theoretically uninformative. Within M1, BIC declined monotonically through k = 6 without a clean elbow; the 5-profile solution was preferred for entropy above .80, significant BLRT (p < .01), and interpretability. The selection of the 5-profile solution was selected despte the smallest profile holding only 3.1% of the sample, which is under the 5% guideline. However, I decided to retain that profile, because Ball-Playing Defenders has face validity.

![BIC by number of profiles](bic_elbow_plot.png)
 
![Entropy by number of profiles](entropy_plot.png)
 
 
---
 
## Results: The Five Profiles
![Latent player performance profiles — z-score means by profile](lpa_profile_plot.png)
Profile means on the original scale:
 
| Profile | n | % | npxG/90 | xAG/90 | Prog. carries/90 | Pressures/90 | Aerials/90 | Pass % |
|---|---|---|---|---|---|---|---|---|
| **Clinical Finishers** | 52 | 9.0 | **0.28** | 0.08 | 1.84 | 9.70 | **2.24** | 72.4 |
| **Pressing Engines** | 169 | 29.3 | 0.05 | 0.07 | 3.32 | **11.83** | 1.32 | 80.8 |
| **Ball-Playing Defenders** | 18 | 3.1 | 0.05 | 0.03 | **9.64** | 5.60 | 1.34 | **89.7** |
| **Limited Role Players** | 69 | 12.0 | 0.05 | 0.02 | 0.66 | 5.20 | 0.89 | 61.5 |
| **Balanced Contributors** | 268 | 46.5 | 0.02 | 0.02 | 1.13 | 4.29 | 0.47 | 83.7 |
 
**Clinical Finishers (9%).** npxG roughly 5× the sample mean, the strongest aerial presence, and offensive output purchased at the price of possession security. Members: Messi, Mbappé, Giroud, Julián Álvarez, Lautaro Martínez, En-Nesyri.
 
**Pressing Engines (29%).** The tournament's engine room: nearly 12 pressures per 90 with meaningful ball progression and almost no direct goal threat. Members: Modrić, Kovačić, De Paul, Amrabat, Enzo Fernández, Tchouaméni.
 
**Ball-Playing Defenders (3.1%).** The rarest and most distinctive profile: extreme progressive carrying (9.6/90, nearly 3× the next profile) combined with near-90% passing and modest defensive event volume — possession-dominant teams' center-backs advancing into uncontested space. Members: Stones, Varane, Maguire, Marquinhos, Aké.
 
**Limited Role Players (12%).** Low volume on every dimension with the weakest passing (61.5%); disproportionately substitutes and direct wide attackers whose game is high-risk touches in the final third (Kolo Muani, Ezzalzouli).
 
**Balanced Contributors (46.5%).** Nearly half the sample: reliable circulation, modest event volume everywhere.
 
![Profile membership distribution](profile_size_distribution.png)
 
*(Full fit table: `outputs/tables/model_fit_comparison.csv`; profile exemplars: `outputs/tables/notable_players_per_profile.csv`.)*
 
### What the model does well
 
Five profiles were hypothesized in advance (finishers, playmakers, pressers, defensive anchors, balanced contributors). Four emerged in recognizable form. The fifth, **"Creative Playmakers", never separated**: xAG never dominated a profile on its own, instead distributing across finishers, pressers, and ball-playing defenders. In a variable-centered analysis (like averages) this would be invisible; the person-centered result says creation at the elite level is not a role but an attribute carried by several roles. The positionless model also produced an informative surprise: it grouped elite center-backs by what they *do with the ball* rather than where they stand, something no position-based grouping would reveal.
 
---
 
## Limitations
 
- Profiles describe *World Cup group-and-knockout tournament behavior* per 90 minutes. This is a compressed, tactically distinctive context, and should not be read as stable player traits.
- The aerial indicator undercounts won duels (see data note).
- The 3.1% profile, while interpretable, is below conventional size guidelines and rests on 18 players.
- Indicators are per-90 rates. This means players near the 180-minute floor have noisier rates than tournament ever-presents.

---
 
## Repository Structure
 
```
wc-player-profiles/
├── 01_soccer_data.R              # StatsBomb pull + player-minutes reconstruction
├── 02_Metric Creation.R          # Event aggregation: 6 per-90 indicators, z-scoring
├── 03_lpa_model_selection.R      # 1–6 profile enumeration, 3 specifications, fit indices
├── 04_lpa_final_model.R          # Final 5-profile model, labeling, visualization
├── 05_team_composition.R         # Shows profile distribution on teams
├── data/                         # raw (StatsBomb) and processed (analysis-ready)
├── outputs/                      # figures, tables, saved model objects
└── codebook.md                   # variable definitions
```
 
Requirements: `tidyverse`, `tidyLPA`, `mclust`, `StatsBombR`, `here`. All models seeded (`set.seed(42)`).
 
---
 
## References
 
- Nylund, K. L., Asparouhov, T., & Muthén, B. O. (2007). Deciding on the number of classes in latent class analysis and growth mixture modeling. *Structural Equation Modeling*.
- Spurk, D., Hirschi, A., Wang, M., Valero, D., & Kauffeld, S. (2020). Latent profile analysis: A review and "how to" guide of its application within vocational behavior research. *Journal of Vocational Behavior*.
- Humphrey, S. E., Hollenbeck, J. R., Meyer, C. J., & Ilgen, D. R. (2007). Trait configurations in self-managed teams. *Journal of Applied Psychology*.
- Rosenberg, J. M., et al. (2018). tidyLPA: An R package to easily carry out latent profile analysis. *Journal of Open Source Software*.

## Disclosures
This project was completed with the assistance of Claude.
- StatsBomb Open Data: https://github.com/statsbomb/open-data
 
