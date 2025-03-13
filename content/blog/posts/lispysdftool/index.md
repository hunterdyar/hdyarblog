---
title: In Search of a Language for 2D Images
date: 2025-03-11
draft: true
---

I have a dream of parametric 2D images. I would like to define a set of rules that allow an image to be generated, in an easily sharable, human-readable, and authorable file.

# Background

I don't need to create my own programming language from scratch (although, spoilers... I will). There are a number of existing projects worth mentioning. Let's start with the lay of the land, then I will lay out my vision.

## Processing
Processing, p5.js, p5py, and other versions of the same [core software](https://processing.org/). It's a language, and environment, and community. It's widely loved and supported, with a plethora of learning resources. It has high accessibility!

For "Write code, create images", Processing is the first name worth mentioning. But, similar creating-coding packages have been created in other languages: [OpenFrameworks](https://openframeworks.cc/), [Cinder](https://libcinder.org/), working in a [Game Engine](https://godotengine.org/), and [more](https://github.com/terkelg/awesome-creative-coding). It feels like I've played with them all.

They are not quite what I want for a few reasons: They are often not portable. To share an image, my target generally needs to have the same framework installed. (P5.js is nice because it's on the web!)

For creating interactive widgets (such as this [DOF visualizer](https://explainers.hdyar.com/dof/index.html), I generally use a javascript graphics library that outputs an SVG or Canvas element. Whichever one I use, they aren't dissimilar to processing.

## Shaders and ShaderToy
"Write Code, Make Images" you say? The answer is shaders! Just write code for the GPU. Simple!

[ShaderToy](https://www.shadertoy.com/) is a community for sharing such projects, and it's wonderful! The downside is that shader languages like GSL are... not very accessible. Hard to use in practice. Reading through the featured shaders that have lots of comments is a great way to learn tricks; but that's what learning [shaders](https://thebookofshaders.com/) feels like to me: a collection of tricks over a set of foundational, transferrable, knowledge.

Also see: [Kodelife](https://hexler.net/kodelife)

## Bauble
[Bauble](https://bauble.studio/) is a major inspiration. I discovered it when I was months into writing my own language and tool, and it was a revlation. "Someone else get's it!". I basically want to create Bauble but for 2D and not on the web.

## Geometry nodes
Blender's [Geometry Nodes](https://docs.blender.org/manual/en/latest/modeling/geometry_nodes/introduction.html), using a visual graph representation of a procedural (i.e.: parametric) definition. One of my favorite tools for working in 3D.

I'm not a huge fan of node-based workflows, but not for lack of trying. I've used em all! From [Blueprints](https://dev.epicgames.com/documentation/en-us/unreal-engine/blueprints-visual-scripting-in-unreal-engine) to [Max](https://cycling74.com/products/max) to [vvv](https://vvvv.org/) to [TouchDesigner](https://derivative.ca/)

## Penrose
[Penrose](https://penrose.cs.cmu.edu/), a clever constraint-solver based approach to generating diagrams. I love this thing. It's not really designed for creative coding - and you need to have a clear understanding of your goals - but constraint solvers are a fascinating approach.

Working with a constraint solver feels like a conversation with a computer, and the non-deterministic nature should be a negative, but it feels like a positive. At least, it's a refreshing workflow. It's doing what you are telling it matters, and it's *not* doing what you *aren't* telling it. I am extremely interested in constraint solver tools applied to creative workflows, although that's not my current take.

## SVG
Scalable Vector Graphics. Did you know you can just... write them? Like HTML! 
SVG's are an XML-based markup language for describing 2D images! That's literally exactly what I want!

In fact, I have deeply considered, and even [prototyped](https://github.com/hunterdyar/vector-nodes-desktop), writing languages or tools that compile *to SVG*. Doing so has a lot of advantages: the portability and wide usage of SVG, primarily. I think this would be a good idea... generally.

The disadvantages? No SDF support (I would need bitmaps. Read on for that), and they're just kind of clumsy to write or edit in. No languages, loops, or other tools for describing images procedurally. With SVG's, you're describing them directly, one vector component at a time.

# So... 2D SDF's?
SDF's are cool! Ian Henry, the creater of Bauble, explains their thinking in [this blog post](https://ianthehenry.com/posts/bauble/building-bauble/). The short answer is that [with tools, that we hope to create] they provide an intuitive way to think about how one constructs shapes, and translate those thoughts into action.

SDF's are Signed Distance Fields, or Signed Distance Functions. (Solving the function for an area is a field). You write some math function that is positive or negative for any given pixel. (Give it x and y coordinates, it gives you some number). The sign of that number determines if the pixel is inside or outside the shape we are drawing.

I think best understood through [example](https://iquilezles.org/articles/distfunctions/) than reasoned about.

The biggest advantage over this approach is it permits "Constructive Solid Geometry". Think "going wild with boolean modifiers in blender". Building up shapes by just smashing together primitives, including with shapes that represent holes or cutouts.

Check out [MagicaCSG](https://ephtracy.github.io/index.html?page=magicacsg). A nondestructive editor for 3D objects using signed distance functions as the base-building blocks. [Clayxels](https://assetstore.unity.com/packages/tools/game-toolkits/clayxels-165312?srsltid=AfmBOopxLVGfIh4RwKGAXOpauscW-znEXnPCjOpwXqjVDJ3JwQBqPRe3) works this way, as does my own [Clayze](https://clayze.hdyar.com/) project.

### Parametric Design
I am pretty deeply comitted to non-destructive editing practices. This is just a generally true sentiment that I have when I to approach any digital work. When I edit a photo, I prefer Lightroom over Photoshop. With Photoshop, I'm all for adjustment layers over applying image effects. Same thoughts towards editing audio, 3D modeling (Blender's Geometry nodes my beloved), etc.

Recently I started learning [Fusion](https://www.autodesk.com/products/fusion-360/overview), software for CAD; I've been designing my own models to print with a 3D printer. In Fusion, you edit visually, but you are *actually* recording a timeline of actions that the software will execute. You can "go back in time" and change any parameter, and all the resulting actions wil re-execute and update.

It's exactly like Blender's GeoNodes, among other tools, only it hides the timeline-view (a small bar on the bottom) of what you are working on in favor of a view of... the thing itself. I find this confusing, but I've slowly started to figure it out.

If you've never done parametric or procedural design before, you may not know the power of the "ah-ha" moment when you make a non-linear change, and everything just updates perfectly. It completely re-wires your brain for the better. I want to bring this kind of workflow with me everywhere I go

### 2D Images
I was recently in a Unity project and I needed to define a rounded rectangle. I had to open photoshop and tell it a width and height... but that's the thing I know last, not first! It felt all wrong.

So, I paused my project and spend a week [prototyping a unity tool for generating images from SDF](https://github.com/hunterdyar/SDF-Sprite-Graph)'s. I defined a rectangle, rounded it, and clicked generate. I used percentages of the width or height instead of direct pixel units, so I can render the exact size I want later.

It's viable but slow. I like this prototype and wish this tool was finished and complete in Unity... but I don't want to be stuck inside of the Unity editor when working on this dream project of mine.

### Pipeline Programming


## Thistle



### A Toy with LISP
I was looking for a project to learn the nim programming language with, over my spring break. I found "Make a Lisp", and once I had one working; I quickly decided to turn it into a clone of my Thistle project, generating SDF's with it.

It's ... really nice? Although 
