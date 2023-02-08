---
title: Introducing my Transition Effects Unity Package
date: 2022-02-08
draft: false
---

During Global Game Jam 2023, we raised the possibility of having the camera fade out before a cutscene. I added it to the list, but never had time to implement it. I did think "Why should I have to implement that? Fading the camera should just be a thing". Well, with that spirit in mind, I created [Unity Transition Effects](https://github.com/hunterdyar/Unity-Transition-Effects/), a package that allows one to easily do camera transitions.

![Animated Gif demonstrating a few of the effects](transitionEffect.gif)

Hacking together a proof of concept is simple enough - I first made a feature that can do this back in [2021](https://github.com/hunterdyar/BloopsUnityUtility/commit/dc0ace58bcb1045023485455748c376caf11a211). The problem with that solutions is that it's annoying to use, if you're already using other camera effects like Post Processing. It doesn't work "with" Unity.

Instead of some hacky addition, Scriptable Render Features are the exact right tool for the job. I don't just want to do transitions, but make an easy-to-use tool that  does it's job well.

As far as graphics projects go, this turned out to be fairly easy to implement. If one wanted to get started writing render features, a fade out or transition effect would be an excellent starter project, since it's so basic. So how do I expose this to other systems?

To answer that question, I need to know who my users are.

Possible Users:

1. Novices who want a quick fade-out but have never seen a coroutine in their life. They just want a component they can add to projects to get the feature they want working.
2. Users who already have an additive scene control system/manager, and would like an easy way to add camera effects on top of it.
3. Users who don't need screen transitions, but do want to fade to solid colors or use the effect in other unintended ways.

This sends the code design in 3 directions:

1. Easy to use components that just sort of work magically behind the scenes. For this, I am going to create a few set-and-forget components to call a transition on Start or from code, as well as a "Fade then change scene" scene manager.
2. An easy to use API for working inside of coroutines. For this, I am going to create some static coroutines and yielded for from whatever other systems a user already has, as well as include a sample project containing a more advanced scene manager.
3. For the user who wants to mess with this tool as a post-processing effect more than changing scenes, I will make sure everything I can reasonably expose can be used well.

The only question is what to include in the package, and what to include as a Sample. I think a lot of beginners will tend to skip looking at samples, trying to jump straight to just seeing something in action, then adjust from there. So I will include the plug-n-play scripts in the base package, and the scene manager - which this package is not for - in the Samples.

That's my thinking on the design of it for now.