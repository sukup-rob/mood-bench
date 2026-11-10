



# functions

## misc
`push: { a: .002 }` = add 0.002 to arousal every tick
`tug: { a: [-.5, .02] }` = pull arousal toward -5, closing 2% of the gap each tick


## tick()

### decayMood(s)
- affect decays towards baseline.

### applyBehaviour(s, def)
- applies the current behaviours tugs and pushs
- TO GET RID OFF LATER AND REPLACE WITH SLEEP ARCHITECTURE AND BETTER OBJECT SUMMONING

### applyNeeds(s, def)
- messy one/two letter variables. fix at some point?
- `need` is like the on-switch for making the "be uneasy and bad when youre hungry/thirsty". odd name, but makes sense.
- temperature

### applyHealth(s, def)
- this just deals with illness.

### fireEvent(s, def)
- Q is a queue of events about to fire.
- has a little force-sleep thing...
- `if (e.fromApproach)` ---> "stuff fires if object gets approached"?

### feel(s)
- ages all pulses
- adds them to the valence

### moveBaseline(s, def, m)
- uses output of feel to tweak baseline slightly.
- tonic = baseline viewed as a resting state instead of a computation anchor. theyre slightly different in real neuroscience though, so thats a gap to work on later.
- the malaise section is a bit messy. 
- also handles the sleep baseline repair.
- basically everything that moves baseline lives here.

### stepWorld()
- moves all that cycles thru the 24 hour cycle. thats currently sun+temperature only.
- i think that sun going down/up and temperature limits are a big gap in the user-set parameters. TODO
- deriveWorld() is its own function so you can tweak all those derived things on their own? but right now only reset and init and stepWorld call it im pretty sure. underused!!

### stepSurrounding()
- right i forgot spawn rates are saved in behaviour. bad bad bad they need to be saved in the archetypes themselves and also they need to have like. a better way to put them in because current choosing system does it relatively, to know how often does a wolf spawn you also need to know how often does everything else spawn and that sucks as hell
- archetype = a type of object. "big-red-animal + if approached you get attacked" would be an archetype. they have names.
- `entries.reduce((sum, [, r]) => sum + r, 0)` --> sum up all the odds 
- `if (totalRate > 0 && Math.random() < totalRate)` --> pick first that gives you 0
- (this choosing system WILL get killed after the archetype spawnrate thing is fixed.)
- for every object in the surrounding, make sure youre not running away from it or towards it. once we have a Speed variable, this will need tweaks. ...also, in general it needs tweaks haha. right now when you have a predator and food next to each other and start running from predator, the food stays in surrounding, which isnt the expacted behaviour of the simulation lol. (also there should be objects that are hard/easy to run from. a predator might follow you!)
- eyes() give previewPulse() a target. previewPulse() decide if a Preview Pulse is even necessary. ...we could put temperament here in the "do we get preview" gates. as well as other stuff. DISTANT TODO

### stepBodyClock(s, m)
- handles the C sleep process
- also handles the wake/fall-asleep tresholds
- also handles the offset that makes circadian rhytm do All That with arousal

### stepSleepPressure(s, noon)
- handles the S sleep process
- rises while awake, falls while asleep
- a little confused about what does malaise do here... make sleep deprivation worse, maybe?

### stepSleepWake(s, body)
- process C and S meet here through `body.st` and `body.ct`
- also handles not being able to fall asleep due to arousal or other body state
- if currently asleep, handles ultradian cycle - changing behavior, but only between REM and NREM.

### stepApproach(s)
- if an object is being approached, handles the approach timer.
- this will also eventually need to be redone - instead of approach timer we should be dealing with distance + speed. this is more or less cosmetic afaik, but right now it really feels like approached are happening TO the agent.
- if approach timer is 0, pushes event connected to the archetype into Q for fireEvents() to get to next tick.
- consumes the target. eventually, this should depend on the archetype, (you dont consume a wolf,) but for now its probably not worth the headache of giving archetypes ANOTHER thing to remember. DISTANT TODO
- i KNOW theres some messy stuff in this function or in related, because the agent never really seems to learn anything proper from approaches, but i just dont see it.

### chooseBehaviourAndRandomEvents(s, m, ea)
- my sweet jesus that REALLY shouldnt be a function name.
- actually uses `ea` as the arousal...
- if not asleep: wraps up stuff needed for decision into `c` and calls `decide(c)`

### decide(c)
- `desper` is a bool and shouldnt be. also bad name kinda.
- generally the idea of overrides is odd to me. they shouldnt be boolean. i mean its probably fine for now but they REALLY shouldnt be boolean. DISTANT TODO

### record(s, m, body)
- `body` gets passed here because `body.ea` and `body.offset` need to be drawn.
- this pushes everything that the screen needs to draw into `H`, which is where the html takes its data, like, directly.



# variables

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

- `surroundAgeRate` CANNOT stay a `ST` thing. it needs to be archetype-specific. the rest will be handled by the eventual speed variable.
- energy meter. (same category as sleep debt - meters that arent directly bodily, but very much affect stuff).
- remove hunger from behaviour definitions.
- metabolism rate. affected by speed, heat, health, ...? should have a number it doesnt go under.
- Catabolism rate. higher catabolism = higher energy bar filling + higher hunger bar emptying. affected by stress, behaviour... should have a number it never goes under.
- speed meter. affected by behaviour, temperament, health, ...?. higher speed = objects spawn more + metabolism rate goes up. is closer to "behaviour" than to "hunger", categorically.
- rename `K.deprivedBurn` to `K.sleepDeprivedBurn`
- rewrite the comments around malaise to be clearer
- The code only does instrumental ("I chose to approach this, and it felt X"), never Pavlovian ("this was present when X happened to me"). FIX  THAT
- remove sunlight checkbox and all related to it
- add seasonal/diurnal tuning. when sun rises and disappears should be sliders, as well as temperature.
- split chooseBehaviourAndRandomEvents(s, m, ea) into chooseBehaviour (wrapper for decide()) thatd be gated from sleeping agents from tick() directly (???) and chooseRandomEvents()



---

# TODO (less important edition)
- ACTUAL LONGTIME BASELINE ISNT 0,0 BUT DEPENDS FROM INVIDIUAL TO INVIDIUAL. CONNECT WITH TEMPERAMENT MAYBE?
Long-term set point (weeks/months)
    ↑ nudged by sustained affect + major events
    ↓ pulls adaptation level
- save custom events

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
```?
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
