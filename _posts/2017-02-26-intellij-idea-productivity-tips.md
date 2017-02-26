---
layout: post
title:  "IDEA lesser known features"
---

I'm a JetBrains fanboy. I simply love everything they do. Their tools have
dramatically increased my productivity and enjoyment for developing software.
IDEA is elegant, fast, and smarter beyond any IDE I've tried. Every update they
do brings something cool like [ligatures](https://d3nmt5vlzunoa1.cloudfront.net/idea/files/2016/05/idea_2016_2_ligatures_1.png)
or [paramter hints](https://d3nmt5vlzunoa1.cloudfront.net/idea/files/2016/09/Screen-Shot-2016-09-27-at-10.29.15.png).
They're awesome!

I've gather a collection of features I use daily which some of my peers were
unaware of. I think it's an awesome list for juniors to quickly become productive
without listing an exhaustive cheat sheet or using an obnoxious key promoter plugin.

## Shortcuts

Instead of being an exhaustive list, I've selected only the shortcuts which I use
frequently or which have great value.

| **Navigation** | **Shortcut** |
|:-----------|-------------:|
| Search symbol | `Ctrl + Alt + Shift + N`|
| Search everywhere | `Shift + Shift`|
| Goto or create test | `Ctrl + Shift + T`|
| **Inspection** |  |
| Quick Definition | `Ctrl + Shift + I`|
| Show usages (in popup) | `Ctrl + Alt + F7`|
| **Edit** |  |
| Join lines | `Ctrl + Shift + J` |
| [Complete statement](https://www.jetbrains.com/help/idea/2016.1/completing-statements.html) | `Ctrl + Shift + Enter`|
| Surrround with... | `Ctrl + Alt + T` |
| Generate... | `Alt + Insert` |
| Override methods | `Ctrl + O` |
| **Refactor** | |
| Change Signature | `Ctrl + F6`|
| Change Signature >> Move parameter | `Alt + Up / Down`|
| **Misc** |  |
| Search Action | `Ctrl + Shift + A`|
| Hide tool windows | `Ctrl + Shift + F12`|
| **Debug** | |
| Evaluate expression | `Alt + F8` |

## Inspections

### Cyclomatic Complexity
This is a metric that quantifies the complexity of a piece of
code (mostly methods) by counting the number of branching points. The more
ifs and switches a method has, the higher the score. Such methods are
inherently harder to test and more prone to bugs. IDEA can calculate the cyclomatic
complexity of methods (and classes) by enabling:

`Java » Method metrics » Overly complex method`

This will issue a warning for methods with a complexity higher than a threshold
which defaults to 10.

### Overly coupled class
High cohesion / low coupling is a widely known principle
in OOP. IDEA can count the number of dependencies a class hass by enabling:

`Java » Class metrics » Overly coupled class`

The coupling threshold is configurable, and it excludes classes from `java` and
`javax` packages.

---

That's it for now. I'll probably update this list when I discover something new.

Develop with pleasure!
