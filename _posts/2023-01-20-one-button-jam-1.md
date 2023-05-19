---
title: "Rust and Bevy Engine: One-Button GameJam (Part One)"
date: 2023-01-20
categories: [University, Devblog]
tags: [university, gamejam, rust]
image:
  path: /assets/img/post/one-button-jam-header.png
  width: 1280
  height: 720
  alt: Screenshot of the final game. Not that pretty, but not bad for two weeks of work!
---

A few months ago, I completed my first university game jam. Everybody was in teams of two to three, and the only rule was that the game could only use one button. Simple enough!

While there was only one official rule, an inherent requirement of the game jam was that the game could be distributed as a standalone binary. Unfortunately, this meant that Roblox, an engine I am already highly proficient in, was immediately off the table.

Additionally, I had already used Unity, Godot, and Unreal Engine in game projects. While these are absolutely acceptable candidates, I wanted to try something especially challenging and new.

I was the only programmer on our three-person team, so I had an opportunity to play with some technology I was new to. Before this game jam, I had already been using the Rust programming language for about a year. I had also been watching the Bevy game engine project for a similar time. Having already been wanting to use Bevy in an actual project, Rust and Bevy are the options I ultimately went with.

## Bevy Engine

*Bevy* is a modular open-source and community-maintained game engine written in Rust. At Bevy's core, it uses an Entity Component System (ECS), an architectural pattern primarily found in game development. ECS is still an emerging technology but has many advantages over your standard object-orientated component systems in engines like Unity and Unreal Engine.

Without going into heavy detail (there are many resources online!), an ECS enables the decoupling of data and logic. In an ECS, systems (logic) have no state, and components (state) have no logic. This distinct separation presents several advantages over traditional object-orientated approaches:

* *Improved performance* : ECS architectures can safely multithread systems based on the data they mutually access. Additionally, an ECS is optimised for CPU cache hits, which is always great in hot code paths.

* *Better organisation* : ECS allows developers to organise their game objects and their behaviour in a more modular and organised way. This modularity can make it easier to work on large and complex games and can also make it easier for multiple developers to work on the same project.

* *Reusability* : Because ECS separates game objects into their individual components, developers can easily reuse components across different objects. This reusability can save time and effort and help ensure consistency across the game.

* *Testability* : Because most ECS systems operate on known pieces of state with no extra side effects, they can be executed in isolation using mock state during automated testing, making unit and integration tests trivial to implement.

However, despite these advantages, there are some disadvantages to consider as well:

* *Increased complexity* : Because ECS involves organising game objects in a more modular and granular way, it can add complexity to the development process. This added complexity can make it more difficult for new developers to understand and work with the codebase and make it more challenging to debug and troubleshoot issues.

* *Reduced flexibility* : Because ECS requires a more structured and standardised approach to game object organisation, it can limit the flexibility of the game design. This reduced flexibility can make it more difficult to implement certain features or behaviours and make it harder to make changes or updates to the game.

To end this post, I will refer to a now-famous GDC presentation the  Overwatch team presented that covers their experience using an ECS architecture. In their own words, ECS reduces your flexibility to solve problems and puts you in a pit. However, that pit is a pit of success. 

# Part Two
In the next blog post, I will review the fun bits -- implementation! We'll explore how the game works behind the scenes, the decisions made, and what I learnt from using Bevy Engine for the first time.
