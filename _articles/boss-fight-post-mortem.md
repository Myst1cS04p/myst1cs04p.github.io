---
layout: post
title: "Boss Fight Postmortem: My First Game Jam"
date: 2025-02-14
author: Myst1cS04p
tags: [game dev, game jam, GDevelop, collaboration]
color: "#E01BFF"
excerpt: "My first game. My first team. My first submission. Years later, it's still the project I'd remake first, because I can still see exactly what it should have been."
description: "My first game. My first team. My first submission. Years later, it's still the project I'd remake first, because I can still see exactly what it should have been."
reading_time: "5 min read"
css: assets/css/articles.css
cover: /assets/img/articles/covers/boss-fight.png
title-image: /assets/img/articles/titles/boss-fight.png
---

In May 2021 I made my first real game. Something that wasn't a prototype, and wasn't a tutorial. A thing I submitted to a real jam, with [a collaborator](https://0new4y.itch.io/), that other people actually played.

It's not that good. The audio was added in the last two hours. The final boss had a phase-transition bug I nearly shipped. And the technical architecture was about as sophisticated as you'd expect from someone who had never made a game before: a simple state machine, a few sprite sheets, and a lot of prayer.

But it was one of the first projects I ever finished. And of everything I've built since, across projects, jams, essays, and communities, this is the one I'd remake first. Not purely out of nostalgia, but because I can still see exactly what it should have been.

---

## What We Built

**Boss Fight** was a one week jam entry built in GDevelop: a fast-paced arena game where you face three bosses back to back, alone, no other mechanics. Just you, a boss, and a pattern you have to learn to survive.

The concept came from the jam's theme, and honestly it was a good concept. Strip away everything except the boss fight, the part most games treat as punctuation, and make *that* the whole game. Learn the pattern. Survive. Next boss.

All three bosses shared the same structure: an attack phase where they became invulnerable and you just had to not die, and a cooldown phase right after where they went passive and you could deal damage. Very simple. Very readable.

What didn't work was the execution on the third boss.

---


## The Golem

The Golem was a big grey figure with a battle axe, and he had two attacks.

The first was a simple charge attack, where he'd lock onto you and ram across the arena. Straightforward. Readable. Felt natural for a big slow bruiser type.

The second attack, the spin, has a more specific origin. Basically, growing up I played a lot of Clash of Clans, and I never got to a high enough Town Hall to actually use the Valkyrie troop. But I always watched the attack animations. That spinning, attack always seems so strong, and I wanted that. So when it came time to design the Golem, I basically, just kinda yoinked the entire attack, but more like the cheaper DIY version. lol

The idea was that the spin should feel like genuine danger, like if you're anywhere near him while it's active, you're just going to melt. Get out or get hit.

Did it feel that way? Not really. It was a simple game, and the effect didn't fully sell. But the concept was right. A boss whose most dangerous state is just *existing in a radius*.

---

## The Spider

The Spider sat in the center of the arena and didn't move. It had a single attack: a shotgun spread of web shots firing outward from her mouth in all directions, constantly, for the duration of the attack phase. You just had to weave through the gaps.

On paper she was meant to be oppressive, and claustrophobic. In reality however, completely cheesable in at least two different ways, both of which I discovered myself *after* shipping.

**Cheese strat one:** If you walked close enough to the Spider, she'd spin trying to track you, but you could move laterally faster than she could rotate. Time it right and she'd just spin endlessly, never completing a full turn, never getting her mouth pointed at you long enough to trigger the shooting phase. You could just stand there and stab her while she chased her own tail.

**Cheese strat two:** The web shots spawned from a single point, her mouth, and then accelerated outward. There was a brief moment right at spawn, before they spread out, where all three were clustered together and could be destroyed simultaneously with a spell. If you timed it perfectly you could wipe the entire attack on spawn, skip the dodging phase completely, and get a free damage window every single cycle.

Both of these were unintended. Both are kind of funny. And both point at the same underlying issue: the Spider's threat was entirely dependent on her attacks actually landing, and there were too many ways to prevent that from ever happening.

The core idea was still good though. A stationary boss that fills the arena with projectiles and forces you to keep moving, that's a real design. She just needed more attacks, more variation, more ways to punish you for being clever.

I honestly have a thing for organically produced exploits, so, this sorta thing was a pleasent surprise for me.

---

## The Assassin

The concept on this one was pretty simple: a cloaked figure, ninja stars, teleportation. A boss that felt unpredictable and threatening, something with a bit of mystique and danger. After the brute force of the Golem and the static pressure of the Spider, the Assassin was supposed to feel like fighting something faster than you.

He had two attacks.

The first: a spell that sent ninja stars orbiting around you in a slow circle before pulling inward. The idea was a closing ring of danger, you have a moment to react, then you have to commit to a direction and move. In theory? Tense, dramatic, requiring precision. In practice? the stars were tiny, there weren't enough of them, and they moved so slowly that "walk sideways" was a complete counter. Very underwhelming :/

The second attack was a teleport. He'd cast it, wait a very long time, then appear somewhere else on the map. Cool concept. Completely harmless due to the cast time, and the overall randomness. By the time he teleported you'd already repositioned and were waiting for him at the new location.

It was *supposed* to feel like aggressive pressure. It did not. 

The result: the Assassin had no direct attacks, no way to close distance aggressively, and no attacks that threatened a player who was paying even minimal attention. He became a punching bag. You could stand still, ignore everything he did, and just hit him until he died.

Why did it end up this way? Honestly I'm not sure. It's been long enough that I can't fully reconstruct the decisions. My best guess is we ran out of time and never got to the attacks that would have made him dangerous. But whatever the reason, the gap between what he was supposed to feel like and what he actually felt like is the biggest failure in the game.

---

## What We Got Right

For a first project in a week, more went right than I expected.

Keeping scope to three bosses and one arena meant we actually finished. The pattern design on the first two bosses was genuinely solid, the Golem's charge and the Spider's spread both required real attention without feeling unfair. And the core loop of, learning the pattern, surviving, and then advancing, held up. People who played it came back for more attempts.

Working remotely with a another person for the first time over a week also went better than it had any right to. We stayed coordinated. We shipped. For a first collaboration, that's not nothing.

---

## What I See Now

Here's the thing about the Assassin: I know exactly how to fix him. I've known for a while. The ninja stars need to be faster, more of them, less predictable in their pull timing. He needs direct attacks, something that closes distance and punishes you for standing still. The teleport needs to be instant, not telegraphed, used offensively rather than just repositioning. Add a stage or two, a darker atmosphere, actual menace, and he'd be easily the best boss in the game.

The Spider could hop around the map instead of sitting still. Create spiderlings. Plant slow webs. Launch cocoon traps that root you. Basically, continuing with that theme of a suffocating oppressive force, but dialed up to a hundred. She had the right idea in her one attack but nowhere near enough of it. 

The Golem could be faster, and hit harder. Get an earth-splitting shockwave. Turn what was already working into something that escalates.

And the concept underneath all of it of just boss fights, and nothing else, is genuinely good - atleast in my opinion. Game jams, I feel, produce things like this. They sorta strips away everything except the most interesting part and ask whether that part is strong enough to carry the whole thing. Prototyping basically, but with a formal time cap to really force you to keep the scope tight.

I'm writing this and I want to open Unity right now. I don't mean that figuratively, I mean I am actively resisting the urge to close this document and start blocking out a scene. But alas, for now I have other responsibilities to attend to, and an already (completely self-inflicted) bloated scheduele to navigate. 

---

You can [play the game on itch.io](https://shotgunflamez.itch.io/boss-fight) if you found this interesting :)