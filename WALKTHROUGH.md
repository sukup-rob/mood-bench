



# functions

## tick()

### decayMood(s)
- affect decays towards baseline.

### applyBehaviour(s, def)
- 





# constants

## ST
state/behaviour table.
for example:
```js
foraging: {
  col: '#c9772b',          // colour on the mood-plane / timeline
  hunger: 1.8,             // how fast hunger burns while in this state
  push: { a: .002 },       // steady nudge on arousal each tick
  // (no tug here)
  want: c => Math.max(...),// how much the agent WANTS to enter this state (decision fn)
  surroundAgeRate: 1       // how fast nearby objects despawn
}
```

---

# TODO
- fix asleep affect drift

| State | Valence drift target | Valence drift rate | Arousal drift target | Arousal drift rate |
|---|---|---|---|---|
| Awake | 0 (neutral) | **Fast** ("mean of recent history" is a fine proxy) | Moderate baseline (~0.15–0.25) | Moderate |
| REM | 0 (neutral) | **Slow** (~30–50% of wake rate) | Slightly elevated | Low (nearly stable) |
| NREM (SWS) | 0 (neutral) | **Intermediate** (~60–80% of wake rate) | Low (~0.05–0.1) | Moderate (pulls arousal down) |

- `surroundAgeRate` CANNOT stay a `ST` thing. it needs to be archetype-specific.
- save custom events OR delete the custom event section.


---

# TODO (less important edition)
- ACTUAL LONGTIME BASELINE ISNT 0,0 BUT DEPENDS FROM INVIDIUAL TO INVIDIUAL. CONNECT WITH TEMPERAMENT MAYBE?
Long-term set point (weeks/months)
    ↑ nudged by sustained affect + major events
    ↓ pulls adaptation level

## temperament implementation

| Axis | Controls | Mechanism |
|---|---|---|
| **Shy ↔ Bold** | Reactivity gain to novelty + baseline arousal level | Kagan's BI / Gray's BIS: high BIS → exaggerated arousal response to the unfamiliar, slower return to baseline |
| **Avoidant ↔ Exploratory** | *Exposure rate* to novel stimuli → how fast familiarity builds | Approach-avoidance motivation: avoidant agents get less exposure, so desensitization is starved of input |
| **Inactive ↔ Active** | Baseline arousal level + event rate (how often the agent is stimulated) | General activation: active agents encounter more stimuli per unit time, so affect drifts faster simply because more is happening |
| **Docile ↔ Aggressive** | Threshold for negative valence in social/provocative contexts + arousal reactivity to threat | Low threshold = smaller provocation needed to push valence negative; high reactivity = bigger spike per provocation |
| **Solitary ↔ Sociable** | Valence *polarity* of social stimuli specifically | Social stimuli are a positive input for sociable agents, a neutral-or-negative input for solitary agents |

long term...
```pseudocode
# Baseline is a function of temperament at initialization
baseline_arousal = 0.35 + 0.15 * activity_level - 0.05 * shyness
baseline_valence = 0.10 + 0.05 * sociability - 0.05 * aggressiveness
# Range constraint: baseline can drift (as we discussed), but temperament
# pulls it back if it strays too far. Think of it as a wider "home zone."
baseline_pull_rate = 0.001 + 0.002 * (1.0 - reactivity)  # high-reactivity agents have less stable baselines  
```

adaptation level...
```pseudocode
# Reactivity scales the tracking rate
# High-reactivity agents' reference shifts faster (they're more "stuck" on what just happened)
tracking_rate = base_tracking_rate * (0.5 + reactivity)
# Avoidant agents get less social data → their reference is less influenced by social events
# (not a rate change, but a *filter* on what enters the recent-mean)
if stimulus.is_social and agent.avoidance > 0.5:
    stimulus_weight *= (1.0 - agent.avoidance)  # they barely register it   
```

stimulus impact...
```pseudocode
def stimulus_impact(base_valence, base_arousal, stimulus, agent):
    v, a = base_valence, base_arousal

    # Shy/Bold: novelty amplifies arousal for shy agents
    if stimulus.is_novel:
        a *= (1.0 + agent.shyness * 0.8)       # shy: +80% arousal for novelty
        v -= agent.shyness * 0.2 * stimulus.novelty  # shy: novelty is slightly aversive

    # Docile/Aggressive: provocation threshold
    if stimulus.is_provocative:
        threshold = 0.3 + agent.docility * 0.4  # docile: need bigger provocation
        if stimulus.intensity > threshold:
            a *= (1.0 + agent.aggressiveness * 1.0)  # aggressive: big spike
            v -= agent.aggressiveness * 0.4          # aggressive: big valence drop

    # Solitary/Sociable: social stimuli polarity
    if stimulus.is_social:
        social_valence = (agent.sociability - 0.5) * 0.6  # -0.3 to +0.3
        v += social_valence

    # Active/Inactive: general arousal gain
    a *= (0.8 + agent.activity_level * 0.4)  # active agents feel everything more

    return v, a   
```

learning...
```
Avoidant agent encounters novel stimulus
  → High arousal spike (shy × novelty gain)
  → Avoids it (avoidance behavior)
  → No further exposure → confidence stays low, familiarity stays low
  → Next encounter: same big spike (no desensitization happened)
  → Short-term reference shifts toward "novel things are threatening"
  → Baseline arousal creeps up (sustained high-arousal state)
  → Now the agent is *more* reactive to novelty (baseline is higher)
  → Positive feedback loop → anxiety-like trajectory   
```


---

# extra sources

Helson (1964)
- adaptation level
- basically "we experience things as derivated from our Normal"
Frederick & Loewenstein (1999)
- also about adaptation level
- ADAPTATION LEVEL SHIFT vs DESENTISATION
- adaptation level shift = AL moves
- desentization = event stops being interesting. (prolly put down as too low novelty to be cared about?)
