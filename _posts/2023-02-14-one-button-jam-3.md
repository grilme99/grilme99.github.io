---
title: "Rust and Bevy Engine: One-Button GameJam (Part Three)"
date: 2023-02-14
categories: [University, Devblog]
tags: [university, gamejam, rust]
image:
  path: /assets/img/post/one-button-jam-header.png
  width: 1280
  height: 720
  alt: Screenshot of the final game. Not that pretty, but not bad for two weeks of work!
---

This is the final entry for my One-Button GameJam series of blog posts, and we'll be covering the game's core mechanic - golf balls! Everything from physics to sound.

# Physics

If I were to remake this game, with more time than two weeks, I would probably create a bespoke physics controller for the golf balls. At the time, the Bevy Engine physics scene was relatively young, and the most popular (only) physics engine integration, Rapier, had several bugs and lacked several features that came to hurt our little golf game later.

However, I only had two weeks and had to use an off-the-shelf solution. So, enter:

## Rapier

[Rapier](https://rapier.rs/) is a "Fast 2D and 3D physics engine for the Rust programming language". While the library is certainly pleasant to work with, I didn't have any choice in picking it because it's the only physics engine with an open-source Bevy integration. Writing integrations for any other physics engine would take too much time, so this is what I had to use.

However, my lack of choice isn't necessarily a bad thing. Rapier works, and it works *really* well with Bevy. The integration with Bevy ([`bevy_rapier`](https://github.com/dimforge/bevy_rapier)) is first-party and well-maintained. The API is simple to use and is well-documented. For example, here is how to add collisions to a Bevy mesh:

```rust
// Executed on startup, responsible for spawning in our map.

let scene_handle = asset_server.load("test_map.glb#Scene0");

commands
    .spawn_bundle(SceneBundle {
        scene: scene_handle.clone(),
        ..default()
    })
    .insert(Name::from("Map"))
    .insert(AsyncSceneCollider {
        handle: scene_handle,
        // `TriMesh` gives us the most accurate collisions, at the cost of
        // physics complexity.
        shape: Some(ComputedColliderShape::TriMesh),
        named_shapes: HashMap::default(),
    })
```

Simple enough! `AsyncSceneCollider` from `bevy_rapier` lets us add a high-fidelity `TriMesh` collision shape to all meshes in our scene. The collision shape is generated in a seperate thread behind the scenes and then applied to the scene when ready.

## Ball Controller

At the core of our game is the golf ball controller, a system that handles launching the ball in response to some player input. Thanks to our use of an off-the-shelf physics engine, the controller is relatively simple and consists primarily of our input handling for launching the ball.

<video src="/assets/video/one-button-jam-controller-demo.mp4" controls style="max-width: 100%"></video>

![](/assets/video/one-button-jam-controller-demo.mp4)
_Controller demonstration, showcasing the charge, launch, and respawn behavior_

As you can see in the video, we chose to give players accurate control over the power of their golf ball shots. This contrasts with many existing golf games, where the shot is based on a timer, and players must try hitting the ball at the perfect time.

Based on feedback we received at the end of the GameJam, many people were receptive to our take on the charging mechanism. Its simple nature allows players to take accurate shots without having to perfectly time their mouse release. Despite the simple nature, some said this method of control felt far more enjoyable than existing golf games.

### Charge Mechanism

At its core, the charge mechanism is straightforward in how it works. Here is what it boils down to:

1. On mouse click, set `charging` to `true` and `charge_power` to `0`.
2. On mouse release, set `charging` to `false`.
3. For each frame, increment `charge_power` by our mouse `y` delta if `charging` is `true`.

Then, we can launch the ball forward on mouse release if `charge_power` is above *some threshold*. A stripped-down version of the code for this looks like this:

```rust
// We should launch the ball if we were charging last frame and we're not
// this frame.
let should_launch = charge_state.charging_last_frame && !command.charge;

if should_launch {
    // Launch the ball!

    // Convert the forward vector of our camera into a flat vector, removing
    // the Y component. Then normalize it, so the length equals 1.
    let forward = (camera.forward() * Vec3::new(1.0, 0.0, 1.0)).normalize();
    
    // Charge ratio is a number between 0 and 1 that describes how much charge
    // to apply to the launch.
    ball_velocity.linvel +=
        forward * charge_state.charge_ratio * LAUNCH_FORCE_MULTIPLIER;
}
```

# Audio

Like everything else in our little golf game, audio could get much more sophisticated than it currently is, and there's little to talk about here. We sourced appropriately licensed sourced audio from various websites and included them in our project as `.ogg` files. We chose to use `.ogg` files because of their smaller file sizes than alternatives like `.mp4`. `.ogg` is an open-source format and uses a modern compression technique called Vorbis, which maintains higher audio quality with less storage.

Smaller file sizes were crucial for our use case because we wanted to embed our game on the web, and large files take considerable time to load. In the end, each audio file averaged 20kb in size.

By the time we finished the game, we had multiple audio variations for the following:
Club hits for when the ball is launched by the player. 3 variations, cycled between at random. Audio volume is based on how hard the ball is launched.
Wall hits for when the ball makes hard contact with another surface. 5 variations, cycled between at random. Audio volume is based on ball velocity when it hits a surface.

# Physics Bugs

Looping back to the start of this post, I stated Rapier had several drawbacks that would come back to hurt us later. I wasn't kidding!

## Ball Following Collision Mesh Lines

Rapier's collision backend, [`parry`](https://parry.rs/), has trouble handling flat triangle collision meshes. As demonstrated in the video below, the ball is heavily influenced by the lines of a collision mesh. This bug causes the ball to stray off course. This is seemingly out of nowhere to a user if they don't enable wireframes.

<video src="/assets/video/one-button-jam-collision-lines-bug.mp4" controls style="max-width: 100%"></video>

![](/assets/video/one-button-jam-collision-lines-bug.mp4)
_Collision bug demonstration, showcasing the ball following the collision lines of a ms_

## Ball Getting Stuck in Barriers

We had a bug that plagued us from the start, and we never found the cause of it. In certain situations, the golf ball would collide with a surface and become "stuck", violently glitching back and forth every frame. I spoke with the maintainer of Rapier, who could not offer any solution. The most likely culprit we came to was that the meshes used (from [Kenney](https://kenney.nl)) were malformed in some way, causing the physics engine to incorrectly place the ball.

Regardless, the bug was clearly an issue with the physics engine we chose, and we, unfortunately, didn't have the time to fix it there. This is what the bug looked like:

<video src="/assets/video/one-button-jam-ball-stuck-bug.mp4" controls style="max-width: 100%"></video>

![](/assets/video/one-button-jam-ball-stuck-bug.mp4)
_Demonstration of the above bug. The camera doesn't break in this video, for your sake._

Certainly not ideal to finish with! In addition to the visual glitch, this also caused the game audio to break, producing a loud hit sound every frame.

# Retrospective

This was the first game jam of my university course, and despite our game not having the best graphics, or the breaking bugs, it went well. Not only was it an opportunity to experiment with new tech, to finally use an engine I've wanted to for a long time, but it was also an opportunity to network with the people on my course for the first time.

I probably wouldn't change much if I did the jam again today. Bevy has already improved substantially since it took place, several release versions ahead of when the game jam took place. Using the lessons I've learnt during the jam and after, I would probably take the time to implement my own physics controller for golf balls. That, or I would create bindings for another off-the-shelf engine, such as PhsX or Bullet.

Regardless, I am pleased with how the game jam ended and what we produced. It was clearly a learning experiment, and that's okay.
