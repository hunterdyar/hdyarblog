---
title: Words Rules - GMTK Jam 2023
date: 2023-07-11

---

For this years [Game Makers' Toolkit Game Jam](https://itch.io/jam/gmtk-2023), I created a word puzzle game.

{{< rawhtml >}}

<iframe src="https://itch.io/embed/2154123?bg_color=EFE1D1&amp;fg_color=331D2C&amp;link_color=3F2E3E&amp;border_color=c1b3a3" width="100%" height="167" frameborder="0"><a href="https://blooper.itch.io/words-rules">Words Rules by Hunter Dyar</a></iframe>

{{< /rawhtml >}}

---

## Concept

The game plays as a sort of 'reverse wordle'. Instead of being given some list of hints/rules/conditions, and finding the words that fit the criteria, you are given the words. You must find the rules that most specifically fit these words.

I honestly did not begin thinking I would finish, and I was pleasantly surprised that, after I got the first prototype working, that it was actually kind of fun. So... well, then I *had* to try to finish it.

The game revolves around a nice puzzle. You are trying to determine what is more common in the English language, between some arbitrary conditions. It forces you to think laterally about word construction, and once you have started, it forces you to consider how different conditions work together.

> If a word ends in Y, is it common or uncommon for the second to last letter to be a vowel?

The game prompts you with a series of these interesting little thought exercises, before the player eventually figures out the patterns ('scrabble score' is overpowered, you should always choose it).

Luckily, "breaking" the game by figuring out those patterns is a fun part of the gameplay as well. All and all, I am happy with it.

## Missing Features

I scoped as small as I could, so I didn't end up missing much that I wanted to do. The most obvious shortcoming is that I couldn't figure out a nice way to determine if y was a vowel.

The next missing feature is better scoring. I need to calculate how all the possible scores are distributed, and then grade you based on how you did relative to that distribution. That is *technically* how it currently works, but an [inverse] linear interpolation between the worse and best possible scores is... naeve.

I also really wanted to add a dictionary lookup. Completely unnecessary for the game. I think the kind of "this isn't gameplay but it is important" feature would be a satisfying nod to the types of players who enjoy word puzzle games.

## Future Steps

I plan on remaking the game in Javascript as a standalone website. I don't like programming in Javascript, but I do feel that a mobile webpage (a la wordle) is the right form for this kind of game. 

The most serious next step would be to develop it in such a way to get a lot esoteric rules. "Is a noun", "Is an anagram", and "Shares a latin root" are all interesting rules, but not the kind I could do without more complex dictionary lookups. So, I'll start tinkering with that soon.

## Development

I recorded the following video as a walkthrough/trailer, and poked inside the Unity project. 

{{< youtube uI3TE4cMsoA>}}
