



# functions (VERY stale)

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




---

# TODO
- REFRESH ACTUAL WALKTHROUGH
- FIX SLEEP DEBT DOING SOMETHING WEIRD
During wakefulness, adenosine accumulates in the brain, driving Process S upward along an exponential curve. During sleep, Process S decays exponentially. If your simulated agent's sleep is cut short, Process S doesn't reach its normal minimum — that residual carries into the next day's baseline, which then climbs from a higher starting point. The debt is the cumulative gap between the Process S nadir that should have been reached and the one that was. Practically: daily_debt = sleep_need − actual_sleep, summed across nights. The arithmetic is linear; the biological consequences are not. 1. Asymmetric: Surplus sleep does not bank. Sleeping 9h when you need 8h doesn't create a 1h credit that offsets the next night's deficit. Debt accumulates; surplus is wasted (beyond a point, extra sleep just shifts circadian phase). 2. Non-linear recovery: The first hours of sleep clear Process S much faster than later hours (steeper exponential decay early on). So a 10h recovery night doesn't "repay" 2h of debt the same way the first 2h of a normal night would. 3. Subjective plateau: Your agent's felt sleepiness (a proxy for Process S) levels off after ~1–2 weeks of restriction, while objective performance (attention, reaction time) keeps degrading. This is a well-documented decoupling. 4. No known ceiling: Debt keeps compounding with no asymptote in the data. 5. In a Borbély-based sim, you don't really need a separate "sleep debt" variable — it is the state of Process S. The "debt" is just the difference between the current Process S value and the value it would have if the agent had slept a full need-duration. If you want a scalar "debt" stat for game logic (UI, difficulty scaling, etc.), compute it as the time-integral of (ProcessS_current − ProcessS_baseline_nadir) over recent days, or simply track the cumulative hour-deficit as a running sum.
- ENTIRE ENVIRONMENT FRAMEWORK. CLEAN CUT BETWEEN "WORLD" and "AGENT"
- energy meter. (same category as sleep debt - meters that arent directly bodily, but very much affect stuff).
- remove hunger from behaviour definitions.
- catabolism rate. higher catabolism = higher energy bar filling + higher hunger bar emptying. affected by stress, behaviour... should have a number it never goes under.
- speed meter. affected by behaviour, temperament, health, ...?. higher speed = objects spawn more + metabolism rate goes up. is closer to "behaviour" than to "hunger", categorically.
- NEEDS as a part of memory entries
- CUSTOM NEEDS AS A PART OF THE GRAPH


# TODO (less important edition)
- ACTUAL LONGTIME BASELINE ISNT 0,0 BUT DEPENDS FROM INVIDIUAL TO INVIDIUAL. CONNECT WITH TEMPERAMENT MAYBE?
Long-term set point (weeks/months)
    ↑ nudged by sustained affect + major events
    ↓ pulls adaptation level
- add seasonal/diurnal tuning. when sun rises and disappears should be sliders, as well as temperature.
- add a section (similar to sleep) for "hurt" and maybe "feed" and "water". this will be a kind of "artificial magic messing with the agent" section i think. maybe another drawer?
- The code only does instrumental ("I chose to approach this, and it felt X"), never Pavlovian ("this was present when X happened to me"). FIX  THAT
- (after Energy Meter is implemented) metabolism rate. affected by speed, heat, health, ...? should have a number it doesnt go under.
- make default events tweakable and editable

## temperament implementation

| Axis | Controls | Mechanism |
|---|---|---|
| **Shy ↔ Bold** | Reactivity gain to novelty + baseline arousal level | Kagan's BI / Gray's BIS: high BIS → exaggerated arousal response to the unfamiliar, slower return to baseline |
| **Avoidant ↔ Exploratory** | *Exposure rate* to novel stimuli → how fast familiarity builds | Approach-avoidance motivation: avoidant agents get less exposure, so desensitization is starved of input |
| **Inactive ↔ Active** | Baseline arousal level + event rate (how often the agent is stimulated) | General activation: active agents encounter more stimuli per unit time, so affect drifts faster simply because more is happening |
| **Docile ↔ Aggressive** | Threshold for negative valence in social/provocative contexts + arousal reactivity to threat | Low threshold = smaller provocation needed to push valence negative; high reactivity = bigger spike per provocation |
| **Solitary ↔ Sociable** | Valence *polarity* of social stimuli specifically | Social stimuli are a positive input for sociable agents, a neutral-or-negative input for solitary agents |


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




# parked

- needs vs wants
- Long Term Baseline
```pseudocode
# Baseline is a function of temperament at initialization
baseline_arousal = 0.35 + 0.15 * activity_level - 0.05 * shyness
baseline_valence = 0.10 + 0.05 * sociability - 0.05 * aggressiveness
# Range constraint: baseline can drift, but temperament pulls it back if it strays too far. Think of it as a wider "home zone."
baseline_pull_rate = 0.001 + 0.002 * (1.0 - reactivity)  # high-reactivity agents have less stable baselines  
```
- remove dominance?
- learned urgency. i rarely get to fulfil this need = need gets urgent sooner.

---

# changelog (kept here so i have it in front of me)

1.1
- got rid of a clamping issue that broke learning
- you can now tweak spawn rate of each object (depending on behaviour) (flee/approach ARE behaviours but hidden in UI because tbh i will be getting rid of those anyways)
- you can now tweak rate of random events. theyre currently not dependant on anything at all which just isnt good lol
- got rid of the huge button wall of random events. you can still trigger them manually from the same section where you tweak their rates. for now you cant see their values - thatll get fixed lated.
- broke up malaise into small self-documenting variables (got rid of the "general malaise" slider too) - now got unwellValance, unwellLethargy, unwellDominance, sleepDebtAnhedonia instead.

1.2 (Big Refactor Of Needs)
- needs now have a little table instead of having to be added at like 7 places in the code
- you can now add needs! (not showing up in the graph yet.)
- behaviour didnt change much (unless you add a need) but my function walkthrough is now SEVERELY stale now lol
- misc: UI update. side menu for parameters and custom events.
- custom events now show up in custom objects
- got rid of default water need
- got rid of one of the event composer (why the hell did we have two of those, anyways)

1.2.1
- you can now add custom needs! (next on my plate: need-specific desperation)
- hunger is now treated as just another custom need (except for where it isnt) - is now removable
- UI: collapsable recent events section now

1.2.2 (The Pre-Environment Update)
- memory entries now remember which need they fulfil.

1.3 (Rough Environment Update)
- added environments
- added environment section to drawers
- removed spawn rates from behaviour table - now handled by the world
- cleaned up CSS (allegedly)




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
