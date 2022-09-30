---
layout: post
title: "Chronicles : A journey into World of Warcraft lore and addon development"

author: Ciaanh
categories: warcraft
tags: warcraft addon
---

As I writing this article I have been working on this addon for almost two years. 
This might seems a lot of time and I will present here my journey into learning to make an addon for World of Warcraft.

## Every journey start

To present a bit of context I am a C# developper but I am also a long time Blizzard fan. I first began with Warcraft II and its expansion Beyond the Dark Portal in 1996, due to some issues (the game was not in the box) before playing the game I read the manual, discovered the lore of this universe and fate was sealed.
Then came Warcraft III and in 2004 World of Warcraft. And I have been playing WoW for 15 years now, exploring Azeroth, trying some roleplay and... well reading the quests.

I've been really enjoying every minute of it but I found myself asking things like "When does the War of the three hammers happenned exactly?" or "If my character was born 23 years ago did he witnessed the fall of Stormwind?".
It felt really frustrating and one day came the WoW Chronicles book series and there came my idea to have a way to view the timeline of events that happenned as a timeline inside the game.

## Diving into addons

As I am a developer I knew I have basic skills that I could use for this project but where to start ?
I had an idea of what I wanted, a simple frame with three areas:
- a timeline to select a date
- a list of events for the selected date
- a detailed view to display informations about the selected event

 ![Interface mockup](/assets/img/posts/chronicles-mockup.png)

I searched for infos and tutorial but except the book WoW Programming by By James Whitehead II and Rick Roe there is not much to start with so I began to look at existing addon.
The first one I began to tear appart were Total RP and Handy Notes

http://wowprogramming.com/

explore existing addon : total rp for interface, handy note for modules
as i'm not into xml i tested wow ace

trys and errors
- all is code and it's difficult to separate responsibility
- limit of designing with wow ace (make it nice)
- limit of designing interface timeline (make it work)

after 6 months quit the dev
come back

start over from the ground
making the interface
making the db
making the modules
publish first version (alpha) 01/2020

fix bus on timeline
add filter feature and refine timeline view

building the first set of data

publish first public version

trying to gather the community