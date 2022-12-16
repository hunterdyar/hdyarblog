---
title: A Few Lessons Learned While Programming for Students
Date: 2022-12-08
---

More than many other programmers, much of the code I write will be examined by students in great detail. I would like to share some of the lessons I have learned from that, and how they have affected how I write code.

## Write Self Documenting Code

Writing comments is good. Writing code that doesn't need comments is better. There are many good articles and [videos](https://www.youtube.com/watch?v=cmh5WzEqbDI) explaining various perspectives of [self documenting code](https://en.wikipedia.org/wiki/Self-documenting_code), so I don't feel the need to explain it in detail.

I almost never use single-letter variables, and tend to use quite verbose function names. While commenting code is important, we are still asking students to read the comments... which they hopefully are doing.

But with self-documenting code, at least some level of code explanation is inescapable, even to students just looking for an answer to copy and paste. The code is written in a way that a student might *accidentally* learn something!

```c#
// Use Comments.
public void WriteDescriptiveFunctionNames()
{
    
}
```

## Be Clear about Configuration and Setup

When a script needs to be set up in a very specific way, this needs to be extremely clear. Using comments, [Tooltip] attributes, and [Header] attributes to separate setup (must be a certain way) and customization (which I call "configuration", i.e.: variables for the designer to adjust).

One good example is when naming a public variable that will be used to Instantiate from. We usually Instantiate from prefabs, so if the object should be a prefab, we should be clear about that, and put "prefab" in the variable suffix.

```c#
public class Gun : MonoBehaviour 
{
	//unclear
	public GameObject bullet;
	//better!
	public GameObject bulletPrefab;
}
```

Now it's less likely the student will *accidentally* misconfigure, and have to spend time dealing with a rather silly bug.

## Code Style: Be Consistent & Don't Be Fancy

Ternary Operators, String Formatting, inline functions, oh my! 

So many fun features C# has! 

Let's not use them!  Let's be as consistent as we can. We want students to encounter **one new thing at a time.** So a random little nicely written ternary operator is probably not worth the potential confusion. 

It's devastating for a student who is trying to read code to learn A to suddenly have to take a trip to google to figure out B. It really interrupts their ability to learn. 

If I ever do use some "fancy" trick, I'll literally include a link to documentation/resources about that feature right in the code, and then they can instantly go learn this new thing on their own, which many students will opt to do without being assigned.

But, avoiding it where possible - that kind of thing can be a detriment for students who are struggling. Let's not "kick them while they are down", so to speak. 

## Write Searchable Names

Students are going to copy and paste the error messages they get into google, sure, but they're also going to try googling other things that just have the code involved.

We want them to at least have a *chance* of finding an answer, or at least finding a related answer.

## Isolate Code Systems By Types-Of-Problems 

Much of Unity game development involves a lot of small systems. As a rule of thumb, one component should do one thing. A component should have one job, one piece of functionality that it is doing.

But what we consider a single "system" can often be ambiguous. "Player Movement" might be one system that contains a large number of mechanics and systems: input handling, movement, particles, sounds, and so on.

When writing a script that will be easy for a student to parse, separating different types of problems to not overlap is a nice way to work.

In the movement example above, PlayerMovement is one script, PlayerInput is another script that calls functions on PlayerMovement. 

```c#
class PlayerInput : MonoBehaviour
{
	private PlayerMovement _movement;
	void Awake()
	{
		_movement = GetComponent<PlayerMovement>();
	}
	void Update()
	{
		_movement.Move(Input.GetAxis("Horizontal"));
	}
}
```

This much boilerplate for a simple input call feels unnecessary, but I find it extremely useful and clear to provide students with a single function or script, inside of which they could solve a single problem. As much as possible, take "other problems we aren't worrying about right now" and move them to other classes. When we are looking at movement bugs, we are looking in the movement script. When we are looking at adding input features, like adding support for gamepads or implementing delay-auto-shift, then there is already the boilerplate location where those problems can be solved.

In other words, it's easier for a student to look at a simple script and imagine complexity, and harder for a student to look at a complex script and ignore elements to just pay attention to one (simple) system. We are removing parsing and mentally extracting the systems from each other.

It *should* feel obvious. 

## Clearly Separate Patterns

So far, my advice has just been "be diligent and verbose about writing clean code". Nothing too unusual.  

In addition to isolating separate *literal* systems, I also like to isolate *conceptual* systems.

Much of the process of understanding code involves, not parsing the syntax, but understanding how the code fits into larger systems. Often the term "architecture" is used for high-level strategies, and "patterns" used for medium-level strategies. When employing a pattern, it's important to label that pattern, so students are able to look it up and understand the various parts of, say, the state machine. 

More importantly, and less obviously, however, is how helpful it can be to isolate different patterns. This is something I do when writing code that I probably *wouldn't* bother doing when working in a more traditional code environment. 

> I like to write my code so that I can talk about different code patterns without ever having to point to a pile code and say "okay this line is part of that other thing, don't worry about that right now". 

The need is for a student to be able to understand what a piece of code is doing, and if a code block is participating in multiple systems and patterns, it's going to be a lot harder for the student to parse which part of the code is doing what.

So, to deal with this, I tend to separate patterns.  Usually just by extracting logic into their own functions, and giving them clear names. That way, if a student pointed to any line of code and asked "What is this a part of?", there's a single clear answer. 

```c#
void OnExitState()
{
	ReturnToObjectPool();
	//other state machine code separated from object pooler code.
	poofParticles.Play();
}

void ReturnToObjectPool()
{
	//code for object pooling pattern separated from state machine pattern. 
	gameObject.SetActive(false);
	pooler.Return(this);
}

```

## Avoid Obvious Comments

Avoid self-explanatory of obvious comments. Instead, rely on self-documenting code practices to provide clarity of execution, and wherever possible, use comments to add *new* or *non-obvious* information to the script.

If a student is reading the code and the comments, it shouldn't be repetative.

```c#
//Get The Rigidbody Component
_rigidbody = GetComponent<Rigidbody>();
```

Obvious comments run a few risks.

- Train students to ignore comments
- Be condescending in tone
- Be frustrating to read ("I know, but why!?")

## Avoid Multiline Comments

You never want to have more than one "large" comment block in a line of code. For me, usually an overview at the top of what the script does - that might be perfectly reasonable, but I will still try to avoid it. If students copy/paste the code, then they might skip copying the large comment parts. There should still be enough comments that are with the code to survive copy and pasting, to continue providing the students context and information as they attempt to use and parse the scripts.

That's the main reason, but there's another less obvious reason I avoid large comment blocks: there's a counter-intuitive fear of heavily commented code.

> Confusing code should be commented, so commented code must be confusing. 

It's not an unreasonable belief. Students can spend hours trying to parse a single comment, and doing that has them learn a certain amount of "effort-per-comment line" relationship. So when opening a script with *tons* of comments, they incorrectly predict that it could take an extremely long time to parse.

 When a student opens a file and it has a *page* of comments or two, they tend to go "ah, no, I don't have time to read this" and close it. Oh no! That's the opposite of what we want!

Instead of lots of explanatory comments, we can **link to other resources** right inside the comments. This isn't the time for them to learn research filtering skills, let's set them up for success and link them right inline to a useful resource, should they need it.

I like this approach because, while not teaching the researching directly, I am demonstrating what kinds of resources are useful. The links I will choose are likely to be official documentation, like the Unity Scripting API or the MSDN C# docs. It's good for students to get used to reading and navigating these resources.

## Make Clear Notes of Exceptions

If I am ever taking a shortcut, using a quick fancy piece of syntax, or just staging something the sake of an example, it needs to be clearly delineated. 

"This is the part of the code you learn from" and "This is the part of the code that's just here to make the example work."

Countless times I've seen students pick up strange legacy code in their project that they copied from some explainer video - having not quite known what it was there for or what it was doing.

The exceptions and shortcuts are a **great place** to include little notes about what a proper solution might be. Seeing the shortcuts is a nice way for students to understand the point of the non-shortcut solutions.

```c#
//Get The Rigidbody Component
void Awake()
{
	//Find the manager for now. FindObjectOfType is slow, so you might want to replace this with the singleton pattern, or dependency injection.
	GameManager manager = GameObject.FindObjectOfType<GameManager>();
}
```

## Conclusion
Writing code for students is writing code that balances the following:

- Readable at the appropriate skill level
- Clearly understandable patterns or code architecture in use
- Survives being copied and pasted around