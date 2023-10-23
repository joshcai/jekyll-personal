---
layout: post
title:  "Coroutine Simplicity"
date:   2023-10-22 06:13:34 +0000
categories: tech
shortlink: coroutine-simplicity
---

When I was a TA for a high school game development course, I ended up learning Unity alongside the students. I ended up learning about coroutines there and how much simpler they can make your code when trying to work with timing logic.

I regularly host informal knowledge sharing sessions with some of my coworkers (originally to go over topics in [aosabook.org](https://aosabook.org)), and I decided I wanted to share how simple the code can be.

I used this [Replit post](https://replit.com/talk/learn/A-Starter-Guide-to-Pygame/11741) to create a simple "game" loop in Python with Pygame to make it easier to demonstrate since most of my coworkers weren't familiar with C#.

### The Task

I had a very simple contrived task - have an "animation" that moves an image right for 2 seconds and then move left for 1 second. For example:

![Example animation](/assets/img/coroutine_simplicity/dvd_moving.gif)

### Setup Code

First, we have some initialization code to get the scene set up:

```python
pygame.init()
width, height = 800, 600
backgroundColor = 0, 0, 0

screen = pygame.display.set_mode((width, height))

dvdLogo = pygame.image.load("dvd-logo-white.png")
dvdLogoRect = dvdLogo.get_rect()
```

### Coroutine-less approach

Without coroutines, one way to approach this would be to keep track of each state: 1) moving right 2) moving left 3) staying still and the time each state starts. When the current time is enough time after the previous state, we can move onto the next state.

For example:

```python
dvdLogoSpeed = [0, 0]
moveRightStart = time.time()
moveLeftStart = time.time()
movingRight = False
movingLeft = False

while True:
  screen.fill(backgroundColor)

  screen.blit(dvdLogo, dvdLogoRect)
  dvdLogoRect = dvdLogoRect.move(dvdLogoSpeed)
  for event in pygame.event.get():
    if event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE:
      dvdLogoSpeed[0] = 1
      moveRightStart = time.time()
      movingRight = True

  if movingRight and time.time() - moveRightStart > 2:
    dvdLogoSpeed[0] = -1
    movingRight = False
    moveLeftStart = time.time()
    movingLeft = True

  if movingLeft and time.time() - moveLeftStart > 1:
    dvdLogoSpeed[0] = 0
    movingLeft = False

  pygame.display.flip()
  time.sleep(10 / 1000)
```

### Coroutine approach

With coroutines, we have the `await` keyword that suspends execution of a particular async function until a later point in time - this makes it a ton simpler to express that logic.

For example:

```python
dvdLogoSpeed = [0, 0]

async def change_speed():
  dvdLogoSpeed[0] = 1
  await asyncio.sleep(2)
  dvdLogoSpeed[0] = -1
  await asyncio.sleep(1)
  dvdLogoSpeed[0] = 0

async def main():
  global dvdLogoRect
  while True:
      screen.fill(backgroundColor)
  
      screen.blit(dvdLogo, dvdLogoRect)
      dvdLogoRect = dvdLogoRect.move(dvdLogoSpeed)
      for event in pygame.event.get():
        if event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE:
          asyncio.create_task(change_speed())
  
      pygame.display.flip()
      await asyncio.sleep(10 / 1000)

asyncio.run(main())
```

This is much simpler for a couple of reasons:

- We no longer need to keep track of the time that certain states start - so we no longer need to keep track of the variables of when state changes start, e.g. `moveLeftStart` and `moveRightStart`
- We no longer need to keep track of what state we're in, so we no longer need to keep track `movingRight` and `movingLeft`
- We can encapsulate all of the logic into a function and easily follow the sequencing of transitions
- If we wanted to have more transitions, e.g. start moving right again for 3 seconds, it would be a lot easier to express - just 2 more lines!

Coroutines can be hard to wrap your head around initially, but once you understand them, it can really make your code much simpler.