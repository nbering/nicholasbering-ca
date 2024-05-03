---
layout: post
title: "Exploring Concepts: Writing Garbage Code on Purpose"
description: "Sometimes we go off in the weeds, down a path that doesn't lead to some poorly written software. It's ok to go back to where you started. That's why we have version control."
category: rambling
tags: [Code-Quality, Rambling, Development-Process]
author: Nicholas Bering
date: 2016-10-23
---

I'm going to start this post with a quote from one of my favourite fiction authors of all time.

> “Why do you go away? So that you can come back. So that you can see the place you came from with new eyes and extra colors. And the people there see you differently, too. Coming back to where you started is not the same as never leaving.” --Terry Pratchett, A Hat Full of Sky

Sometimes I find myself repeating that quote to myself in my mind, usually as I'm deleting some
un-merged git branch or reverting an un-committed file.

## The Journey

Software development is a journey. As we write code we're exploring concepts and making discoveries.
Sometimes we discover that we had a bad idea. But that's okay! The important thing is learning to
pick up on the signs that we've made a wrong turn and get back on track.

Studying algorithms, design patterns and anti-patterns can definitely help you understand what is 
good and bad about the code you write. Sometimes it's architectural, other times you just realize
half way through that you gave a concept the wrong name and realizing the right name for it 
completely changes your understanding of the problem.

## Travel Safely

In the amateur stages of my career, when I was writing code for myself and didn't know I could get
version control software for free, rolling back could be painful. Sometimes I honestly didn't know
what I had changed and I couldn't get back to where I started down the wrong path.

Developing software without source control is like
driving a car around without a spare tire. You'll probably be fine most of the time, but if
something happens and there's no one around to help you may be stuck on the side of the road for
a long time. So use source control. Git is free and available everywhere - you don't even need to
store your code online if that's what's stopping you. Git can be distributed or just local as needed.

## Scout for Safer Paths

When you're not sure how to go about implementing a complex feature, sometimes it's helpful to start
a proof-of-concept branch. I've also heard people call these "spikes". The idea is to start a new
branch (sometimes I do this in a whole separate repository if it's a really fundamental concept),
knowing that you're going to throw away this code. Make some quick changes and see what it does to
the project.

Do you run into errors you weren't expecting? Did you break tests you were expecting to pass?
How does the code look? Would it make more sense if you did it a little different? Now throw the
code away and do it again back from where you started.

You'll probably write it better the second time. That's because you've learned from the experience
of the first attempt.

Sometimes this is a little like playing "Code Golf." The goal isn't really to write as few lines as
possible though, it's just to figure out where all the water hazards and sand traps are and avoid
them next time.

## Don't Do Groundhog Day

At some point you might feel like Phil in the movie Groundhog Day, doing the same day over and over
again until you get it just right. If that's the case you've probably gone too far off track. Just
pick something and move on.

Unless you're writing software for NASA's interplanetary missions, it's
probably not critical that you get it right this time around. Take on a little technical debt and
pay it back later. Once you've moved far enough down the path to look back, you'll see if there was
a better way. Even if you never fix it you've learned from the mistake.
