---
title: PlayFab SDK V3
date: 2022-10-10
categories: [Open-source]
tags: [library, luau, roblox, devlog]
image:
  path: /assets/img/post/playfab-sdk.jpg
  width: 1280
  height: 720
  alt: PlayFab and Roblox Studio logos on top of a navy blue background.
---

![PlayFab + Roblox Logo](/assets/img/post/playfab-sdk.jpg)

Over the past few weeks, I have been working on a complete rewrite of my [Microsoft PlayFab](https://playfab.com) integration for Roblox. This rewrite brings some key new features and significant housekeeping to better support community contributions.

## Firstly, what is PlayFab?
Quoted from PlayFab themselves:
> PlayFab is a complete backend platform for live games with managed game services, real-time analytics, and LiveOps.

---

*"But, Roblox already provides managed game services for me! What value does PlayFab bring?"*

Good question, you probably asked! While you'll certainly use a smaller portion of their products as a Roblox developer, there are still a few that could be incredibly helpful. Just make sure to weigh the pros and cons of each service you consider and if it makes financial sense to use.

PlayFab has *many* products available, and I encourage you to research them all at their [website](https://playfab.com). However, I think by far the most useful to Roblox is matchmaking:

### Matchmaking
PlayFab provides real-time matchmaking with support for complex constraints and rules. This is the same matchmaking infrastructure titles like *Rainbow Six: Siege* and *Sea of Thieves* run on.

Whilst it's certainly possible (and maybe even trivial) to implement simple queue-based matchmaking using Roblox [MemoryStore](https://create.roblox.com/docs/scripting/data/memory-stores)'s, it's difficult to apply multiple rules and constraints (such as region and skill-level) to tickets. Even more challenging is decentralising it across hundreds or thousands of running game servers -- all while keeping queue times low.

PlayFab provides an out-of-the-box, proven matchmaking solution that supports multiple queues, rules and constraints, and in-depth matchmaking analytics to help optimise queue times. The engineering time saved going down this route could prove more cost-effective than rolling an in-house matchmaking solution.

If you're interested in seeing a production-ready implementation of PlayFab's matchmaking in Roblox, check out the [`example/`](https://github.com/grilme99/RobloxPlayFabSDK/tree/master/example) directory in the SDK repository.

Read more about PlayFab's matchmaking [here](https://learn.microsoft.com/en-us/gaming/playfab/features/multiplayer/matchmaking/).

Anyway, back to the SDK...

## Why the rewrite?
In preparation for future projects I want to take on, I needed to get the SDK into a more usable state with Luau. While the SDK came bundled with type annotations for [roblox-ts](https://roblox-ts.com), it pre-dated Luau, and thus people who used it with Luau didn't have proper access to API types or errors from the type checker. Additionally, it had the following other problems which helped to justify a complete rewrite:

- The SDK was *big*. PlayFab has thousands of individual APIs, each with its own type annotations for roblox-ts. These APIs added up to a lot of code, which has an excellent opportunity to be split up. I would need to work out how to make individual parts of the SDK opt-in before I could consider including Luau types.
- It's hard to use PlayFab's official generator and even harder to contribute to it. If I wanted to support community contributions to my PlayFab SDK, I would have to maintain my own fork of PlayFab's official generator, which would have been much more work than it was worth. Looking at PlayFab's [official generator](https://github.com/PlayFab/SDKGenerator), it is cumbersome to navigate and even more difficult to edit or use. Codegen is driven by EJS templates, which have to be styled in a particular, hard-to-maintain way to get the desired output. Ultimately, I decided to run the generator periodically on my local machine and not bother with community contributions. I want to change that this time around.

## What's changed?
A few things! In addition to some general ergonomics improvements, all of the problems bought up above have also been addressed:

### Luau types! 🎉
My biggest goal for the rewrite of the PlayFab SDK was to have static typing for the *entire* PlayFab API right inside Luau. I'm pleased to say that, as of V3, the request parameters and responses of every PlayFab API are available, and you'll receive type-checking errors when you do something wrong. Additionally, this SDK version no longer uses the [Promise](https://eryn.io/roblox-lua-promise/) library, meaning Luau will preserve all type information on function returns.

![Luau catching type errors](/assets/img/post/playfab-luau-types.png)
_Screenshot of Luau code missing a required request parameter and the error given by the Luau type-checker._

### Migration to the Wally package manager
[Wally](https://wally.run) is a new package manager for Roblox based on Cargo (for Rust) and NPM (for JavaScript) and is what the SDK needed to allow for opt-in PlayFab services. Each service is its own Wally package, and users only need to import the services they use. You can find a reference to the Wally packages and their current version on the SDK [README](https://github.com/grilme99/RobloxPlayFabSDK#api-reference).

Depending on people's needs and whatever direction Roblox's open-source community moves, I may also consider alternate distribution methods down the line. Ultimately, I want to support native Studio users who may not be using Rojo (and, by extension, Wally), but I think this is better done by supporting Wally inside of Studio. It's a great tool that everyone should have access to. (Hint at the next project?)

![Wally package reference](/assets/img/post/playfab-services-reference.png)
_A table of Wally packages for each PlayFab service, showing a description and version string for each. Each service links to the relevant PlayFab documentation._

### Included SDK generator
This release brings a custom Rust-based code generator to create each Wally package automatically. If you're contributing to the SDK, doing it through the generator is how.

Similar to PlayFab's official generator, this one generates code using PlayFab's Swagger API specification. However, unlike the official generator, it does not use EJS templates. Instead, it uses Rust's [Writers](https://doc.rust-lang.org/std/io/trait.Write.html) (with the help of some macros) to construct Wally packages through code. This approach is nice and gives complete control over formatting, but there are definitely economic improvements to be made.

See the [`generator/`](https://github.com/grilme99/RobloxPlayFabSDK/tree/master/generator) directory for more.

## Finally, what's next?
While the SDK is in a **much** better place than it was previously, there are still some parts I want to improve and places I want this SDK to go in the future:

### Automatic updates
Everything is in place to support automatic updates to the repository, and I just need to find the time to write the GitHub Action to do it. Hopefully soon!

### Better error handling
To put it bluntly, the error handling in this SDK kind of sucks. Error handling in Luau generally sucks, and the error handling approach from Rust has opened my eyes. I'm honestly not sure the best way to improve it in Luau, but I want to. Using pcalls sucks! Catching promises sucks!


### Better Accessibility
As I mentioned earlier in the post, this SDK currently requires Wally to install and use it as intended. However, Wally is only available to those using Rojo. In one of my future projects, I want to bring Wally to studio users so they can access the benefits of a real package manager. More on this soon!

### More Open-Source!
Open-source is great, and I want to keep creating open-source tools, libraries, and integrations for other developers. From now on, I will seriously consider releasing things to the community whenever I make something for myself. The only time this won't happen is if it's something I feel I can't maintain long-term or if there is some security, business-related, or legal reason not to.

## Closing Remarks
This blog post is my first time writing this kind of long-form content, and I think it works rather nicely. Please give me feedback on whether this kind of content is interesting or helpful, as well as on my writing style -- if it's too long, short, vague, or if I keep rambling on. I want to improve!
