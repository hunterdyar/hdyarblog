---
title: Text Parsing Basics with C#
draft: true
---

This is the first of a series of posts on importing text files for Unity game data. This first post explores the basics of text in Unity, and some functions and features of C# that can help turn a string into some userfable data.

## What kind of data should we use text for?
Plain text is a versitale file format. It's not really a file format, it's... just text. You can open it on any computer, and you don't need any libraries, licenses, or overhead to do so.

Unity's inspector is great, but it's not a good writing environment.

First, text files are great for word! Sentences! English.
Simple dialogues, item descriptions, flavor text, and more may benefit from being authored in plain text files.

Next, data that you can author or generate from other tools, like basic python scripts, or text editors.

For basic tabular data, you can export from excel to [CSV](https://en.wikipedia.org/wiki/Comma-separated_values) (which is just plain text), and write an importer manually pretty easily.
For more complex 'database' style data, I would recommend [JSON](https://docs.unity3d.com/ScriptReference/JsonUtility.html) instead.

A future blog post will discuss turning these kinds of files into usable game data automatically. 

## TextAsset
If you are using Unity, we have to get the text data into our Unity Project.

You could add the [TextArea](https://docs.unity3d.com/ScriptReference/TextAreaAttribute.html) to a string to make it easier to type in the inspector.

Let's use a file instead. Make a public variable of type TextAsset, and reference any text file in your project.

As you can read in the [manual](https://docs.unity3d.com/ScriptReference/TextAsset.html), we can access the .text property for text data, and .bytes for a raw array of bytes.

## Line Breaks, Carriage Returns, and New Lines

We haven't talked about encoding yet. For plain text, it's easier than you might think. Just use [UTF-8=(https://en.wikipedia.org/wiki/UTF-8. But you want to write your decoder to be able to handle different operating systems and encoding types. 

The first place you are likely to encounter a discrepency is in the line breaks. When you read your strings in the debugger, you are going to notice "\n" and "\r". \n is what we use for a newline character (or LF, Line Feed). The backslash is an escape key "I mean something special, not the letter n". r is for return, or carriage return (CR). On windows, text files end in '\r\n' - both symbols. The order wouldn't matter on a teletypes, they would execute both moves at the same time. Read the 

> Windows is obsessed with backwards compatibility, so this quirk of still usiung \r is not going away anytime soon. At least we can open files from the 1970's! 

For operating systems based on Unix, \n is all that is needed.

Read the [History](https://en.wikipedia.org/wiki/Newline#History) section of the newline page on Wikipedia. No, really.

#### So What Does That Mean?

We want to treat the \n as the newline character, and ignore or remove the carraige return \r character in our text files. Replacing "\r\n" with "\n" will allow most of our files to work the same on mac/linux as they do on Windows.

Compatability between encoding standards and operating systems is *not* this easy, but that rule of thumb can at least get us started. 

## Splitting Up A String

The one feature we are likely to use the most is [Split](https://learn.microsoft.com/en-us/dotnet/api/system.string.split?view=net-8.0). This function will take a string and a 'delimiter' character. It will return an array of strings made up as the elements in between the delimiter character.

> example with comma seperated liet

## Cleaning the Whitespace

The easiest thing to do now is Trim. [Trim](https://learn.microsoft.com/en-us/dotnet/api/system.string.trim?view=net-8.0) will remove whitespace characters from the beginning and end of the string. This includes extra \r characters if we split the file by the \n character.



## Custom Structs
