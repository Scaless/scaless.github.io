---
layout: post
title:  "Curious Case of the Carrier"
date:   2021-06-16 12:00:00 -0500
categories: halo
---

## 1. Getting Carried Away

If you've played a lot of the Combat Evolved level The Library on the Master Chief Collection, you may have noticed something odd about Flood Carriers.

When a Flood Carrier explodes, it will deal a varying amount of damage to you depending on if you were the one to kill it or if it died of natural causes.

This is not just a minor discrepency. On Easy difficulty it is the difference between losing a modest chunk of shields vs. having your entire shield bar obliterated, with some health gone too. On Legendary, the effect is reversed! We take far *less* damage by shooting the carrier!

What's the deal with that?

<video muted autoplay controls loop style="width:100%">
    <source src="/assets/CarrierCollection.mp4" type="video/mp4">
</video>

## 2. That's A Lotta Damage

**Before we get to talking about the Carrier issue specifically, it would be helpful to go over exactly how damage is calculated in Halo 1. This section goes quite in-depth, so you can skip forward to Section 3 if you just want to get to the point.**

Halo 1's damage calculations are ... interesting. There is a surprising amount of detail to determine how much damage you (and your enemies) can deal. 

Let's get some terminology out of the way first:

|Unit|A character within the world. This can be a spartan, elite, carrier flood, popcorn, grunt, jackal, hunter, etc. If it moves on its own and can die, it's probably a unit.|
|Body Vitality|Maximum total health that a unit can have. Value usually between `0.01` to `250`.|
|Shield Vitality|Maximum total shields that a unit can have. Value usually between `0` to `200`.|
|Health|A floating point value representing the ratio of the unit's current health vs maximum health. Usually between `0.0` to `1.0`. Can go negative (AKA dead).|
|Shields|A floating point value representing the ratio of the unit's current shields vs. maximum shields. Usually between `0.0` to `1.0`, but can go higher in some cases (ex: Overshield). Cannot be reduced below `0.0` by normal means.|
|Team|In multiplayer we have Red vs. Blue. In campaign we have a couple more options. There are teams for Players, Humans, Covenant, Flood, and Sentinels. Teams can be either friendly or hostile to each other, and units assigned to a team will react appropriately. For example, the Players team is friendly to the Humans team and both are enemies to the Covenant team. Betraying marines can temporarily change the relationship between Players and Humans to hostile.|

# Scenario 1: Fragging Yourself on Easy Difficulty

<video muted autoplay controls loop style="width:100%">
    <source src="/assets/EasySelfNade.mp4" type="video/mp4">
</video>

This is one of the simpler examples and gets us thinking about how damage scaling works. As you can see above, the grenade completely removes our shield and takes us down to 1 pip of healthbar.

When a frag grenade explodes, it deals damage within a certain area of effect. Inside the core of the explosion (shown in red) is a region where the maximum amount of damage is always dealt. Within the outside radius (shown in orange), the damage taken will be scaled based on distance from the core.

![frag_explosion](/assets/explosion_radius_visualized.jpg)

Within the inner core of a frag grenade explosion, the grenade deals 120 base damage.

To find out what this number means in the context of damaging our player unit, there's a few steps we need to go through.

---

#### Global Team Damage Scaling

If a unit on one team attacks a unit on a different team, the damage they deal will be affected by a multiplier depending on two factors:

* Are the teams friends or enemies?
* What difficulty are we on? 

![team_damage_scaling](/assets/team_damage_scaling.png)

<center><sup>(Hard and Impossible are the internal names for Heroic and Legendary, respectively.)</sup></center>

In the self-fragging example we are not actually attacking a friendly or enemy team, we are attacking the *same* team. We threw the grenade so the grenade damage is associated with the Players team and is damaging a unit (you!) that is also on the Players team. Since the teams are the same, there is no additional scaling applied and this is effectively a 1.0 multiplier.

---

#### Damage Effect Modifiers

A Damage Effect is a structure that describes how damage should be dealt. Every bullet, explosion, and effect in the game that can cause damage has its own Damage Effect. Damage Effects have properties such as the area of effect radius, minimum and maximum damage ranges, physics side effects, and more.

For now we are just interested in a table of multipliers in the Damage Effect that determines an additional amount to scale the damage by depending on the surface that is hit by the effect.

Here is the modifier table for the grenade explosion effect:

![grenade_damage_effect_modifiers](/assets/grenade_damage_effect_modifiers.png)

<center><sup>(The full list of modifiers is larger. I've trimmed it down here to just the most relevant entries.)</sup></center>

We can see here that Cyborgs (the internal name for the player unit) have a damage modifier of `1.0` for both Health and Shields so the damage is not scaled further here. Hunters take reduced damage from frag grenades, while Flood would take massively increased damage.

In this list you may notice the Engineer entry. Yes, this is the same Engineer that appears in ODST. Bungie had originally designed the creature all the way back in Halo 1 and it was even featured in some promotional material. It was eventually cut from the game, but some leftover references to it still exist within the game files.

---

#### Global Vitality

Let's take a look at a selection of base Vitality values for some units:

|Unit|Base Body Vitality|Base Shield Vitality|
|---|---|
|Spartans (you!)|75|75|
|Elite Minor (blue)|100|100|
|Elite Major (red)|100|150|
|Elite SpecOps (black)|125|200|
|Hunter|225|0|
|Flood Infection Form (popcorn)|0.01|0|

Elites gain progressively better health and shields as they rank up. Hunters are beefy health tanks but lack shields. Infection Forms have a bare minimum amount of health such that a light breeze will kill them.

Next, we need to go back to the global difficulty table. In it we can find entries for Friend and Enemy Vitality modifiers. These multipliers scale a unit's base Vitality depending on the difficulty.

![global_vitality](/assets/global_vitality.png)

Using these modifiers we can determine the effective Health and Shields that our player unit has on each difficulty:

|Difficulty|Effective Player Health|Effective Player Shield|
|-|-|-|
|Easy|60|60|
|Normal|75|75|
|Heroic|90|90|
|Legendary|105|105|

It might seem strange that you have less health and shields when on easier difficulties. In order to compensate for this, the amount of enemy health and damage is also reduced. It's a bit odd but works well enough in the final game.

---

#### Recap

With this information, we can now go through the series of steps to get to our final grenade damage calculation:

1. Our player unit starts at `1.0` Health and `1.0` Shields.
1. We're standing in core of the grenade explosion, it deals `120` base damage.
1. The damage is coming from the same team, so there is no friendly/enemy multiplier applied.
1. The grenade explosion effect has a `1.0` multiplier against Cyborg health and shields.
1. 120 points of damage is applied against our Vitality.
	1. Our unit's base Body and Shield Vitality is multiplied by `0.8` since we are on Easy difficulty, giving us an effective `60` points of Body Vitality and `60` points of Shield Vitality.
	1. Our shields are the first to take damage, depleting all `60` points of Shield Vitality.
	1. The remaining `60` damage is applied against Body Vitality.
1. The damage is converted to a ratio. 
	1. `60 Damage / 60 Shield Vitality = 1.0 Shield Damage`
	1. `60 Damage / 60 Body Vitality = 1.0 Health Damage`
1. We subtract this damage against our current health and shields.
	1. `1.0 Shields - 1.0 Shield Damage = 0.0 Shields`
	1. `1.0 Health - 1.0 Health Damage = 0.0 Health`
1. Our player unit now has exactly `0.0` Health and `0.0` Shields.

But if we have `0.0` Health, why aren't we dead? 

In order to actually die, a unit's health needs to be reduced *below* `0.0`. There are a few cases where the math works out to cause the exact amount of damage needed to put the unit on the brink of death.

Let's look at one more example.

# Scenario 2: Carrier Explosion on Easy Difficulty

1. We stand on top of a Carrier when it explodes, it deals `80` base damage
1. The damage is coming from an enemy team on Easy difficulty.
	1. There is a `0.3` multiplier applied, resulting in `24` remaining damage.
1. The Carrier explosion effect has a `1.0` multiplier against Cyborg health and shields.
1. 24 points of damage is applied against our Vitality.
	1. Our unit's base Body and Shield Vitality is multiplied by `0.8` since we are on Easy difficulty, giving us an effective `60` points of Body Vitality and `60` points of Shield Vitality.
	1. Our shields are the first to take damage, depleting `24` points of Shield Vitality.
1. The damage is converted to a ratio. 
	1. `24 Damage / 60 Shield Vitality = 0.4 Shield Damage`
1. We subtract this damage against our current shields.
	1. `1.0 Shields - 0.4 Shield Damage = 0.6 Shields`
1. After taking this damage our player unit now has `1.0` Health and `0.6` Shields.

## 3. Whose Side Are You On?

Back to topic at hand. Why is the Carrier dealing the incorrect amount of damage when we are the one to kill it?

This behavior does not occur on the original Xbox release, the Gearbox PC port, or the standalone Combat Evolved: Anniversary Edition. It is exclusive to the Master Chief Collection. But why? 

As we'll find out in a bit, the root cause is the [Medals and Scoring system added in the Master Chief Collection.](https://www.ign.com/articles/2014/10/07/medal-system-added-to-halo-ce-and-halo-2-in-halo-the-master-chief-collection)

![343_development_process](/assets/343_development_process.jpg)

*What?* 

How could *double kills* and *triple kills* possibly affect the damage a Carrier explosion does? Stick with me. It'll make sense soon, I promise.

![spaghetti](/assets/spaghetti.png)

Grab your forks and get ready to dig into some spaghetti.

My initial thought process for tracking down this anomaly was pretty simple: Damage a Carrier and set a breakpoint in the function where our player unit takes damage and work backwards from there.

After finding the appropriate functions that handled damage, looking at the surrounding code in IDA revealed something interesting:

![ida_carrier_1](/assets/ida_carrier_1.png)

This immediately stands out as worth digging into. A direct reference to the exact unit we know there is an issue with, embedded in the code that calculates damage? Pure gold. We can translate this into more readable pseudocode that looks like this:

{% highlight cpp %}
// I will be referring to these tables collectively as the "Attribution Tables"
// They exist at a static location in the DLL
UnitId Killers[8]; // Which unit has credit for killing the carrier
UnitId Carriers[8]; // Which carrier unit was killed
int NextKilledCarrierIndex; // Next index into attribution tables

void unit_damaged_function(/* ... */) {
	// ...
	if(unit_name.contains("floodcarrier")){
		Killers[NextKilledCarrierIndex] = killer_unit;
		Carriers[NextKilledCarrierIndex] = carrier_unit;
		NextKilledCarrierIndex = (NextKilledCarrierIndex + 1) % 8;
	}
	// ...
}
{% endhighlight %}

This on it's own doesn't give us much direct info, but it's a great lead to build off of. We've got a pair of tables with 8 entries each and we're storing units in them, specifically Carriers and who killed them. We can analyze the functions that read/write to these tables and find out what other systems touch them. Luckily, there are only a small number of references that we need to investigate. The most promising is this one:

![ida_carrier_2](/assets/ida_carrier_2.png)

We can roughly translate this to:

{% highlight cpp %}

void another_damage_function(/* ... */) {
// ...
UnitId damage_source_unit = damage_effect.source_unit;
TeamId damage_source_team = damage_effect.source_unit.team;

if(damage_source_unit.name.contains("floodcarrier")){
  UnitId killer = Carriers.GetKiller(damage_source_unit);
  if(killer != null){
    // Necessary to propagate the history of who the original unit was that
    // killed the carrier. Imagine a chain reaction where the player shoots
    // a Carrier which blows up, killing another Carrier that blows up,
    // killing a third Carrier. In the Medals and Scoring system the player
    // receives credit for all three kills, even though they only directly
    // killed the first Carrier. This is ok.
    damage_source_unit = killer;
    // THIS, however, is NOT ok. We are overriding the team that the damage 
    // is coming from to be the killer's team. In 99% of cases, the killer
    // will be a player unit, so this will be converting the damage's
    // team from Flood to Players.
    damage_source_team = killer.team;
  }
}
// ...
}
{% endhighlight %}

This is our smoking gun. If you followed along through the section where we broke down the elements that contribute to final damage calculations, you'll know why this is bad. If you skipped that section, I'll spell it out plainly here:

Damage is multiplied by a modifier based on the relationship between the team that the damage was caused by and the team of the unit that the damage is being applied to. Changing one of the teams in the equation can result in a different modifier being applied to the damage. In this case, the team of the damage itself is being altered from the Flood team to the Players team.

When this damage is applied to the player, the overridden team will cause us to take incorrect damage based on this new relationship (Player -> Player vs. Flood -> Player).

On Easy difficulty, the `0.3` enemy damage multiplier is replaced with a `1.0` multiplier. This results in the player taking over 3x more damage than they should have.

On Legendary difficulty, the `1.8` enemy damage multiplier is replaced with a `1.0` multiplier. This results in the player taking almost half the damage than they should have.

### Summary

To properly give players kill credit and assign Medals when Carriers explode and kill *other* enemies, 343 implemented a pair of attribution tables to propagate which Carriers the player killed. In the implementation of this system, the Team property of the carrier explosion damage effect is improperly changed from Flood to Player, resulting in an incorrect scaling of damage done to the player.

## 4. Resolution

To the geeksters of 343 Industries who may be reading:

1. First and foremost, this is a bug. It is different behavior than all other versions of the game and was introduced in the Master Chief Collection port of the game. It is also inconsistent with itself:
1. The attribution tables are NOT included in game state that is saved in checkpoints/core saves, which means this bug affects determinism. Loading a save where a Carrier has been damaged but not yet exploded will result in a different amount of damage applied versus the first time the section was played. 

I am suggesting the following solution, but you have more context than I do to deal with the issue:
* Just remove the code that overrides the team of the damage effect. From my limited testing this does not seem to affect the PGCR report as the attribution is still propagated appropriately, and this restores consistency with previous versions.

| Game Version | MCC 2282 , Steam |
| Attribution Tables Location | halo1.dll+2B6BCD0 |
| Adding Table Entry | halo1.dll+C004DC |
| Checking/Removing Table Entry | halo1.dll+B65DF8 |

## -- Credits 

* Big thanks to [doubl3h3lix](https://www.twitch.tv/doubl3h3lix) for digging up his dusty copy of the standalone Combat Evolved: Anniversary to test some things on console!

