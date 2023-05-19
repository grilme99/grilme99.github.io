---
title: "Springs for Animation in Unity"
date: 2023-03-24
categories: [University, Devblog]
tags: [university, c#, unity]
image:
  path: /assets/img/post/springs-thumbnail.webp
  width: 1000
  height: 500
  alt: Demo of React Spring, a Spring animation library used heavily in the web development world.
---

I come from a Roblox and web background, where Springs see widespread use in all facets of animation. However, this tool seems somewhat less recognized outside of these spaces, especially in the broader game development scene. This blog post aims to shed light on the concept of Springs, their usage, and their benefits in game development, along with demonstrating how I've integrated them into my own Unity projects.

## What are Springs?

In the context of video games, a Spring is a mathematical model used to simulate and animate natural, smooth, and dynamic motion. Instead of fixed durations and curves (as with traditional animation), Springs use physical properties like mass and tension to enable fluid and natural movement.

The advantage of Springs is that they simulate natural motion. Unlike traditional tweens and easings, which depend on fixed durations and curves, a Spring is a continuous motion that constantly moves to a target point. This constant fluid motion allows you to have a continuously changing target without breaking the fluidity of the animation.

Additionally, springs are incredibly adaptable and versatile, making them well-suited for complex animations and interactions. Because their behaviour is determined by dynamic parameters, they can be adjusted on the fly to create various effects. This contrasts traditional animations that must be pre-calculated and cannot easily change once running without breaking fluidity.

## Unity Spring Implementation

[Here](https://gist.github.com/grilme99/6742672cbb3547d99cccee026af4588d) is an example Spring implementation in Unity C#. This code is based on a Spring implementation used widely in Roblox, which can be found [here](https://github.com/Quenty/NevermoreEngine/blob/2ad8cea7dd3ad79a39afd7d7b785b489b90553fd/src/spring/src/Shared/Spring.lua).

This specific implementation only supports Vector3s, but extending it to support other datatypes should be trivial.

Here's an example implementation of my Spring class, where we move a gameobject to the mouse position in the world every frame. Hopefully, this illustrates how Springs can capture fluid and natural motion with continuously changing values.

{% include embed/youtube.html id='kPGJfWNHDRA' %}

```c#
class MoveToMouse : MonoBehaviour
{
    // Adjust speed and damper as needed
    public float speed = 5f;
    public float damper = 0.5f;

    private Spring _spring;

    private void Start()
    {
        _spring = new Spring(transform.position);
        _spring.Speed = speed;
        _spring.Damper = damper;
    }

    private void Update()
    {
        var ray = Camera.main.ScreenPointToRay(Input.mousePosition);
        if (Physics.Raycast(ray, out var hit))
        {
            // Update spring target with mouse world position. The spring will move
            // to the current target in a fluid motion every frame.
            _spring.Target = hit.point;
        }

        // Position is computed lazily every time it is fetched, there is no need for
        // us to pass any deltaTime.
        transform.position = _spring.Position;
    }
}
```

## Real-World Example

*Note: This section was added in May 2023 after completing a university assignment artefact.*

As already explored, Springs are great for modelling continuously changing fluid motion. As such, they are a perfect fit for first-person games that need to animate objects the character is holding, and this is the exact use case I had when creating a dodgeball game.

{% include embed/youtube.html id='j5PDc1FD-Ts' %}

As the video demonstrates, the dodgeball on-screen continuously reacts to the character's motion with natural and realistic results. This result is achieved entirely with multiple Springs that have been layered together.

In this specific case, four Springs were used:

- `SwaySpring`: Impulsed by two things:
  - Horizontal and vertical changes in camera angle and;
  - Sudden changes in vertical velocity (i.e. falling from a height and landing on the ground).
- `WalkingSpring`: Target is a sine wave whose amplitude changes based on the current horizontal speed of the character.
- `ChargeSpring`: Pulls the dodgeball backwards based on the current charge of the throw during each frame.
- `ThrowSpring`: Position is set behind the camera every time the ball is thrown, causing the ball to instantly disappear and quickly return to the frame for a realistic throw effect.

These Springs are layered by adding their positions together during each frame and using the result to place the dodgeball in the world. Overall, these layered Springs create a realistic and fluid motion that would be extremely difficult to model without. This entire system was programmed in less than 120 lines of code and uses no external animations.
