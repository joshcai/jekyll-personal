---
layout: post
title:  "Creating Team Quiz Battles"
date:   2023-05-28 06:13:34 +0000
categories: tech
shortlink: team-quizzes
---

Another year has passed, and so my annual tradition of creating a "birthday game" for my friends continued. 

This year, I was busier than I expected (ended up coding the majority of it in a single day), so I tried to keep it simple by sticking to a web-based text-centric game, since I'm more familiar with the tech stack and knew I could hack together something quickly.

I eventually decided to base this year's game on one we play frequently when just chatting online - [Sporcle quizzes](https://sporcle.com). Sporcle quzizes take many different forms, but a common format is to name a list of items from a category off the top of your head, e.g. the 50 states of America. We would often try to get 100% on a single quiz together, which would honestly take us a lot longer than we expected each time.

I had an idea in the back of my head for a competitive team-based version of Sporcle. I think Sporcle has a competitive version where you can go head-to-head against another person, but with a group of friends hanging out together, I thought it would be a lot more fun for us to do it team vs team.

### Tech Stack

I branched off one of my old web-based games to get the boilerplate, and that stack was based off of:

- NodeJS Express for the backend
- Vue / Vuex for the frontend
- SocketIO for communication

I chose this stack mostly to have front-end / back-end in the same language, in case I needed to share any code in between the two. I found I actually don't share much code between the two anyways, so maybe in the future I'll opt for a Python Flask server, which is usually a bit simpler for me to work in...

### Verifying Right Answers

Normally, in a Sporcle game, you would type part of the answer and once it matches enough of the word, it'll count it as an answer. You can usually make slight typing mistakes and it'll still count it as the answer - originally, I was interested in writing an algorithm to get close to this logic.

Unfortunately, due to lack of implementation time (again, only had a single day to cobble everything together), I ended up with a straightforward equality check instead. What I ended up throwing together was:

```javascript
checkGuess() {
  // Cleaning a word entails lowering the case of all leters and removing all
  // non-alphanumeric characters.
  const cleanedGuess = this.guess.toLowerCase().replace(/\W/g, "");
  // this.$store.state.cleanedWords contains all possible answers which have
  // been cleaned.
  const index = this.$store.state.cleanedWords.indexOf(cleanedGuess);
  if (index !== -1) {
    this.$socket.client.emit('guess', index);
    this.guess = '';
  }
}
```

This code is pretty naive and basically requires that the user type in the word exactly right, ignoring case and special characters. Unlike Sporcle, there is no typo detection or early submission if matching the majority of the letters.

I thought this code would work for all of my words, but I didn't realize until my friends started playing the game that there were two special cases that I had included where it didn't work so well.

#### Substrings

One use case I had completely overlooked was cases where one answer to guess was a substring of the other answer.

For example, one of the categories was League of Legends champions - two of the ones being `Vi` and `Viego`. 

When trying to type in `Viego`, the user would type in `Vi`, which would trigger the check and submit the guess for `Vi`, which then cleared out the guess. 

Luckily, my friends were pretty smart, and they found a workaround quickly - by typing some random characters in the beginning, it wouldn't match, e.g. they could type in `aViego` first, and then delete the `a` in the beginning, which would then check `Viego` and submit the guess that way.

#### Special Characters

One case where I didn't get so lucky is when answers themselves actually contained special characters themselves. I only had two words like this, but I totally didn't inspect my datasets carefully to notice them.

For this one, the theme was names of the original generation of Pokemon. There's two separate Pokemon named Nidoran♀ and Nidoran♂ for the female / male version of Nidoran. Note: the special characters are officialy part of the Pokemon names.

In this case, my friends would type in Nidoran, and it would actually match Nidoran♀ to their surprise. Since I also cleaned the answers, both actually would get cleaned to Nidoran - and then the first one would get selected by the `indexOf()` call.

What that meant was that there was no possible way for my friends to get the other Nidoran♂. My friends actually thought they could do the same trick as the previous section - copy in the symbol directly, but that couldn't help them either.

### Making HTML Elements Draggable

One other thing I learned while making the game was how easy it was to make elements draggable - after the Sporcle quiz, I had a puzzle where I wanted to let each player drag the clues around, so they could re-order in any way they wanted to.

I had no previous experience with making draggable HTML elements, and I expected to have to spend a lot of time figuring out support.

To my surprise, good old jQuery has super easy support to do so, all I had to do was: 

1. Import jQuery and jQuery UI:
   
   ```html
   <script src="https://code.jquery.com/jquery-3.6.0.js"></script>
   <script src="https://code.jquery.com/ui/1.13.2/jquery-ui.js"></script>
   ```
   
2. Tell jQuery to make the cards draggable

   ```javascript
   $(function() {
     $('.card').draggable();
   });
   ```

And that was basically it - I was honestly very surprised at how little work that part took. 
