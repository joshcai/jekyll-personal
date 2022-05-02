---
layout: post
title:  "Jico Park in Unity with Mirror"
date:   2022-04-24 06:13:34 +0000
categories: tech
shortlink: jico-park
---

During the pandemic, virtual birthday parties over Discord became commonplace within my friend group. For my birthday party this year, I wanted to create a personal copy of a game that we all played together: [Pico Park](http://picoparkgame.com/en/). We all really enjoyed it! Well, I guess except for the times where people kept dying unnecessary...which was actually pretty much all the time.

If you haven't tried it and can gather a group of friends willing to try it, I'd highly recommend it! I think one of the best parts is that it supports up to 8 players - most online co-op games are usually 4 or 5, which is usually an awkward number for us.

### Networking Framework

While I had previous experience with creating simple Unity games, I have never tried doing multiplayer with Unity - learning how to do multiplayer was the main reason I wanted to try creating Pico Park in Unity.

As of April 2022, Unity itself doesn't have a great story for multiplayer - on their website it says `Important: UNet is a deprecated solution, and a new Multiplayer and Networking
 Solution (Netcode for GameObjects) is under development.` I decided to try a third-party solution and was mostly debating between [PUN](https://www.photonengine.com/pun) and [Mirror](https://mirror-networking.com/). I ended up choosing Mirror since it was fully open source and the server-oriented (vs. P2P of PUN) meant it could scale to more concurrent users if I ever needed that in the future.

Overall, I found that Mirror works pretty well and was surprised at how easy it was to get things setup, e.g. not having to host a server and allowing one user to act as the host simplified deployment significantly for me. That turned out to be crucial since I coded most of this in 4.5 days right before my birthday and definitely had no time to mess with setting up a server.

My main issue when learning Mirror was that it can be confusing sometimes which things are synchronized and which are not. For example, when I first replaced the sprites from a box to the actual Pico Park cat, I changed the sprite renderer so that it flipped horizontally when the user moved to the left with something like this:

```csharp
void Update() {
  ...
  float dirX = Input.GetAxisRaw("Horizontal");
  if (dirX < 0f) {
    spriteRenderer.flipX = true;
  } else if (dirX > 0f) {
    spriteRenderer.flipX = false;
  }
  ...
}
```

In initial tests, this worked perfectly fine - until I tried with two players, and I realized the sprite flipping doesn't propagate at all, leading to scenarios like this:

![Jico Park - sprite renderer flip bug](/assets/img/jico_park/jico_flip_bug.gif)

Before trying to fix something like this yourself, you should check to see if there's a `Network` version of the component you're trying to sync already, e.g. `Network Transform` automatically syncs the transform variables for you. 

If there's no existing component, the fix is pretty straightforward once you get used to the Mirror paradigm:

```csharp
// Create a variable that is synced with the server.
[SyncVar(hook = nameof(OnFacingRightChanged))]
public bool facingRight;

// Create a Command that allows updating the synced variable.
[Command]
public void CmdFlipSprite(bool facing) {
  facingRight = facing;
}

// Callback that executes whenever the value of the synced variable changes.
void OnFacingRightChanged(bool oldValue, bool newValue) {
  spriteRenderer.flipX = !facingRight;
}

void Update() {
  ...
  float dirX = Input.GetAxisRaw("Horizontal");
  if (dirX < 0f) {
    // Call command instead of updating sprite renderer directly.
    // This causes all clients to end up executing the sprite renderer flip.
    CmdFlipSprite(false);
  } else if (dirX > 0f) {
    CmdFlipSprite(true);
  }
  ...
}
```

### Importance of Playtesting

This was more or less the first "complete" game that I have made - although "complete" is definitely an exaggeration as it was missing a lot of polish, e.g. it no sound effects whatsoever. 

One of the most important things I learned from this project was just how important playtesting is - my friends were able to "break" my game in ways I really didn't expect.

The most notable of these was due to how dashes reset when the user is touching ground - however, my raycast was clearly not working properly, since the dash reset whenever you touched **any** wall, regardless from which side.

This led to interesting "speed-running" techniques where they could dash repeatedly into a wall from the bottom side of the platform, which also allowed them to skip most of the obstacles. 

Here's a video of it in action:

![Jico Park - speedrunning](/assets/img/jico_park/jico_speedrunning.gif)