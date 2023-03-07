---
title: "Rust and Bevy Engine: One-Button GameJam (Part Two)"
date: 2023-01-26
categories: [University]
tags: [university, gamejam, rust]
image:
  path: /assets/img/post/one-button-jam-header.png
  width: 1280
  height: 720
  alt: Screenshot of the final game. Not that pretty, but not bad for two weeks of work!
---

Given that we only had two weeks to complete this game jam, our group's concept had to be simple to execute. After exploring various ideas, we ultimately chose a minigolf-style game that used only the mouse as our one button.

Considering this is a programming blog, I will not delve into details about the game design. However, this is what the control scheme looked like:

```
Mouse Movement = Pan Camera
Press Left Click = Enable Charging
Release Left Click = Release Charging
Move Y Movement = Change Charge Power
```

As you can see, this is not *technically* single-input. However, it is one-button (left click)! Technicalities 😌

# Implementation

Now, onto the fun stuff! As mentioned above, this game was implemented with Bevy and the Rust programming language. In this blog post, I will cover the high-level system design decisions and how they could have been implemented differently.

## Commands

At the core of this game are *commands*. A command is generated at the start of every frame and describes the actions the player wants to take based on their input. Commands can then be queried in other systems that run later in the frame.

```rust
struct Command {
    camera_zoom: f32,
    mouse_delta: Vec2,

    charge: bool,
    release_charge: bool,

    toggle_camera_lock: bool,
}

/// This code is very Bevy-specific, but an Action is essentially just an input, using the Leafwing Input Manager library.
///
/// See https://github.com/Leafwing-Studios/leafwing-input-manager for more info.
fn drive_command_gen(
    action_query: Query<&ActionState<Action>, With<LocalPlayer>>,
    mut command_query: Query<&mut Command, With<LocalPlayer>>,
) {
    let action_state = action_query.single();
    let mut old_command = command_query.single_mut();

    let delta = match action_state.axis_pair(Action::MouseDelta) {
        Some(delta) => Vec2::new(delta.x(), delta.y()),
        None => Vec2::ZERO,
    };

    // Update the old command to match new data
    old_command.camera_zoom = action_state.clamped_value(Action::CameraZoom);
    old_command.mouse_delta = delta;

    old_command.charge = action_state.pressed(Action::Charge);
    old_command.toggle_camera_lock = action_state.pressed(Action::ToggleCameraLock);
}
```

It is not easy to improve upon this system. Commands decouple gameplay systems from processing raw player input and provide an appropriate level of abstraction for player actions. The advantage of this decoupling is that gameplay systems are unaffected if input mappings change (which they probably will!). Processing keycodes and other input methods is done in one function, making later maintenance significantly more manageable.

## Camera Controller

Another core component of any game is its camera controller, and our little golf game is no different! Given our short timeframe for this jam, I opted to use an open-source camera manipulation library called [`Dolly`](https://github.com/h3r2tic/dolly), which abstracts away much of the math and quaternion logic that comes with handling cameras.

The camera system builds off of commands, utilising `mouse_delta` to drive camera rotation around the golf ball. In addition, it uses the command structs `camera_zoom` to drive the zoom in/out logic.

Zooming the camera in and out took much tuning to get right. In addition to adjusting the camera's distance to the golf ball, the camera's vertical *height* is also adjusted based on how far the camera is zoomed out. This effect is implemented with various easings, allowing players to view their surroundings more intuitively.

<video src="/assets/video/one-button-jam-camera-demo.mp4" controls style="max-width: 100%"></video>

![](/assets/video/one-button-jam-camera-demo.mp4)
_Camera controller demonstration, showcasing panning and zooming in/out behavior_

Thanks to the use of Dolly, the camera implementation remained relatively straightforward. With some of the query boilerplate omitted, this is what the camera system looks like:

```rust
fn drive_camera(
    q0: Query<(&GlobalTransform, &LocalCharacter)>,
    q1: Query<&Command, With<LocalPlayer>>,
    mut q2: Query<(&mut Rig, &mut CameraState)>,
    mut q3: Query<&mut Projection, With<CurrentCamera>>,
    mut windows: ResMut<Windows>,
) {
    if let Ok((ball_transform, character)) = q0.get_single() {
        let command = q1.single(); // SAFETY: Commands will always be generating if we have a character
        let (mut rig, mut camera_state) = q2.single_mut();
        let mut projection = q3.single_mut();

        // Set the camera position to the golf ball's position
        let ball_translation = ball_transform.translation();
        rig.driver_mut::<Position>().position = ball_translation + (Vec3::Y * 0.5);

        // Zoom the camera in and out
        let new_zoom =
            (camera_state.zoom - (command.camera_zoom * CAMERA_ZOOM_SENSITIVITY)).clamp(0.0, 1.0);

        // Persist the zoom in the state for future frames to access
        camera_state.zoom = new_zoom;

        // Ease the horizontal offset along a curve, mapping the 0-1 value to the min and max horizontal offsets
        let horizontal_offset = map_01(
            ease_out_quad(new_zoom),
            MIN_CAMERA_Z_OFFSET,
            MAX_CAMERA_Z_OFFSET,
        );

        // Ease the vertical offset along a curve, mapping the 0-1 value to the min and max vertical offsets
        let vertical_offset = map_01(
            ease_in_out_cubic(new_zoom),
            MIN_CAMERA_Y_OFFSET,
            MAX_CAMERA_Y_OFFSET,
        );

        // Apply the offsets
        let arm = rig.driver_mut::<Arm>();
        arm.offset = Vec3::new(0.0, vertical_offset, horizontal_offset);

        // Update the FOV of the camera, based on whether or not it is moving
        // TODO: Add easing to camera FOV
        if let Projection::Perspective(projection) = projection.deref_mut() {
            projection.fov = if character.should_allow_control {
                CAMERA_FOV_STILL
            } else {
                CAMERA_FOV_MOVING
            }
        }

        // Rotate the camera around the golf ball based on mouse delta
        if !command.charge && camera_state.camera_locked {
            rig.driver_mut::<YawPitch>().rotate_yaw_pitch(
                -CAMERA_MOVE_SENSITIVITY * command.mouse_delta.x,
                -CAMERA_MOVE_SENSITIVITY * command.mouse_delta.y,
            );
        }

        // A debug key can pressed to give mouse control back
        if command.toggle_camera_lock {
            camera_state.camera_locked = !camera_state.camera_locked;
            for window in windows.iter_mut() {
                window.set_cursor_lock_mode(camera_state.camera_locked);
                window.set_cursor_visibility(!camera_state.camera_locked);
            }
        }
    }
}   
```

## Part Three

The final part of this mini-series, Part Three, will cover the physics and ball controller implementations. We'll explore how it was implemented and how it could be improved.
