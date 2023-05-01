---
title: Methods for Handling Custom Packages in Unity
draft: true
---

## Custom Asset Pack

From unity projects, export a .unitypackage and archive it somewhere accessible, like a NAS or FTP server; or for yourself. 

Import as needed. Make changes and adjustments while working on a current project, and re-export/replace the package as you go, if making updates.

Pros: Simple, understandable, no software dependencies.
Cons: Version management, hard to make update to one package from multiple projects.

## Custom Package - Local

Like the above solution, but after the annoyance of setting up the package, it's easy to deal with versions, and switch versions. 

No clean way to propagate changes back to the original package files. You have to make changes, change the version number, and re-export/replace the files. The simplicity, that is the pro of just an assetpackage, is lost when also dealing with manually handling version numbers/syncing.

## ## Custom Package - Git

Add by git URL. This is the best solution (IMHO) when you have a package that you don't update that often, and use in a lot of projects, like utilities scripts. I've used this for [transition effects](https://github.com/hunterdyar/Unity-Transition-Effects) and the like.

Pros: Part of the package cache.

Cons: Part of the package cache. The cons is updating the package. Often we add a feature or fix a bug specific to a project. Any changes you make to local files in the project will be lost, you have to go to a proper git clone of the asset and push changes to the remote, then go back to the project and push update. You don't have to constantly increment the version number, you can just always hit 'update' to, effectively, do a pull.

Git needs to be installed to add by git url.

## Custom Package - Git Submodule

Instead of adding the package by a git url, you skip the package manager, and add a git [submodule](https://www.atlassian.com/git/tutorials/git-submodule). 

Pros: You can edit the files and manage it as a 'prope' remote package. 
Cons: You must use git for your base project in order to use the submodule. Also, git submodules are very annoying, and can cause plenty of git headaches. *(Note: I like [Fork](https://git-fork.com/), a git UI that makes some of the more tedious git things, like rebasing or submodules, much easier).* 

First, set up the package just like the above, as a proper unity package and host it on github, or whatever git remote you are using. You **could** add the asset by git url in the package manager at this point. 

Now, go to git and add a git submodule with the remote url, to a "Packages/com.package.packagename/" folder.

From Unity's point of view, this is an [embedded package](https://docs.unity3d.com/Manual/CustomPackages.html#EmbedMe). It will see it, locally, and display it in the package manager just as if it were added by git url.

The difference is that you can make changes to the files, then go into the submodule and commit/push those changes to the packages remote, then go back to your project and commit and push it's separate changes there.

You can set up multiple packages as submodules.