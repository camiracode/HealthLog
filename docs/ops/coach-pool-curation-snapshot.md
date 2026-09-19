# Coach shared-pool curation + sleep stages (local Titan deploy snapshot)

Base: HealthLog **v1.38.8** (`MBombeck/HealthLog`).

Deployed locally as Docker tags culminating in `healthlog:coach-sleep-stages`
(`buildSha: coach-sleep-stages`).

## Motivation
Coach snapshot uses a shared newest-first **6000-row** measurement pool.
High-volume types (PULSE, then SLEEP_DURATION + walking gait) starved
cross-metric cause/effect series (glucose CGM, HRV, resting HR) for Libre
weeks W30–W34. Soft char-cap also dropped sleep stages (only on
`timeline.recent` ~14d) so weekly Libre analysis saw duration only.

## Changes (files attached)

### `src/lib/ai/coach/snapshot.ts`
- Dedicated capped reads: **glucose**, **HRV**, tiny **pulse**, **resting HR**
- Exclude from shared pool sources: glucose, hrv, pulse, sleep, resting_hr,
  walking_asymmetry, walking_double_support, walking_step_length, walking_speed,
  audio_env, audio_headphone
- Type denylist belt-and-suspenders for gait/stairs/audio/sleep/RHR/pulse/glucose/HRV
- **Keep** OXYGEN_SATURATION (spo2) in shared pool
- Soft-cap: raise char budget; retain weekly for sleep/glucose/cardio; protect aggregates
- Merge dedicated HRV + resting HR into value-series measurement rows

### `src/lib/ai/coach/snapshot-blocks/sleep-block.ts`
- Weekly sleep buckets now include mean **CORE / DEEP / REM / AWAKE** minutes
- Built from **all** reconstructed nights (not only pre-recent) so soft-cap
  shedding `timeline.recent` still leaves Libre-week stage mix on weekly
- Aggregate `meanStages` across window

### `src/lib/ai/coach/clusters.ts`
- Include if it differs from v1.38.8 (cardio/sleep priority / weekly retain wiring)

## Out of scope / follow-ups
- Multi-year monthly/yearly rollups for priority metrics (user wants multi-year analysis)
- Upstream may use slightly different path names — match this tree’s layout under
  `src/lib/ai/coach/`

## How to verify
1. Build image with these files
2. New Coach thread: Libre W30–W34 cause/effect with glucose, sleep stages,
   HRV, resting HR, SpO₂
3. Expect resting HR present for W30–W31; weekly sleep with core/deep/rem/awake;
   no reliance on daytime pulse

## PR intent
Draft PR from fork `camiracode/HealthLog` → `MBombeck/HealthLog` (or open on
fork only if upstream PR not possible). Title/body should describe the above
clearly for optional upstreaming later.
