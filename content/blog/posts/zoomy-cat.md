---
title: Zoomy Cat!
draft: false
date: 2024-01-16
---

I made a video game this weekend! 

 {{< rawhtml >}}
<iframe frameborder="0" src="https://itch.io/embed/2470877" width="100%" height="167"><a href="https://blooper.itch.io/zoomy-cat">Zoomy Cat! by Hunter Dyar</a></iframe>
{{< /rawhtml >}}

I have been feeling pretty creatively burned out. I have a small number of very large projects that I have not been able to make significant (visible) progress on (because academia). It's been a while since I've actually finished anything. So, with MLK day giving me 3 days of work, and no plans other than to watch the steelers lose to the bills, I got to work!

### The Design

I started with a list of requirements, which looked something like this:

- One button
- Less than 30 second game loop
- Replayable
- No dialogue, no audio, no 'reading', no tutorial
- Readable in black/white
- Better players should do better, but a random awful or lucky run is permissible
- No tactics or strategy. Short term fast decisions and constantly revising decisions. 
- It cannot have a point. It must not mean anything. It is being made for the sake of being made.

> *The last bullet point I wrote out to specifically fight my game-dev-equivalent-of-writers-block. It worked! I tricked myself into finishing a project.*

From there, I started sketching some idea on post-it notes, sticking to the 'dodge things' scheme. I liked the idea of infinite wall-jumping, remembering some old game-boy platformers from my past, and that was that. 

I sent some screenshots to friends and asked what they thought. One person told me to make it a cat, and I instantly recalled [cat planet](https://twitter.com/sylviefluff/status/1205451485347303427), a game from a ludum dare jam made in 2009. With that amazing game fresh in my mind, I decided that I needed a high score feature, and I needed little round cats in black/white.

>  Cat planet is playable [online](https://love-game.net/catplanet/) now! Cat planet cat planet cat planet cat planet.

The rest of the design was just saying "no" to ideas, to keep scope in check. No to making the spikes wiggle, no to color splashes in the particle effect, yes to screen shake, no to more types of obstacles, etc etc.

### The Code

The source code is [available](https://github.com/hunterdyar/Zoomy-Cat), although it got messy after a while. 

The most interesting parts is the edge detection effect, a shader technique I implemented as part of the URP post processing stack. The shader code was a technique I saw pop up a number of times found while researching (checking offsets for a threshold). To implement it in Unity, I copied the method (and most of the code) of how I did my [transition effects package](https://github.com/hunterdyar/Unity-Transition-Effects). Basically renaming things and swapping the shader.

The next most interesting part, form a game programming perspective, is probably the weighted random item selection, which adjusts over time. I am quite pleased with the technique, for it's simplicity. I plan to at least write another blog post to go over it in detail. Before that's done, [here's the code](https://github.com/hunterdyar/Zoomy-Cat/tree/master/Assets/Scripts/BWeightedSelection).

That's all I have to say about it.

 Zoomy cat! Play at [blooper.itch.io/zoomy-cat](https://blooper.itch.io/zoomy-cat).