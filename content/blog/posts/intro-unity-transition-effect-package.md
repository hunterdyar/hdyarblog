---
title: Introducing my Transition Effects Unity Package
date: 2022-02-07
draft: true
---

During Global Game Jam 2023, we raised the possibility of having the camera fade out before a cutscene. I added it to the list, but never had time to implement it.

Later during the post-mort, I realized I really should be able to just "fade out then change scenes" and "fade in on start". At the same time, I remembered that I wanted to explore how Unity's Scriptable Render Features work.

I already knew the basic trick - a Blit with a shader on the camera texture, and it turned out to be fairly easy to implement. If one wanted to get started writing render features, a fade out or transition effect would be an excellent starter project.

Being able to write the shader code is just one challenge. Hooking it into the rendering system elegantly is the other primary challenge... so I'm done? For a proof of concept, sure. But It isn't, it's a tool for others to use. 

That's the real work. Getting away from proof of concept and to a proper, usable, package. Time to do some design research.

Users:

1. Novices who want a quick fade-out but have never seen a coroutine in their life. They just want a component they can add to projects to get the feature they want working.
2. Users who already have an additive scene control system/manager, and would like an easy way to add camera effects on top of it.
3. Users who don't need screen transitions, but do want to fade to solid colors or use the effect in other unintended ways.