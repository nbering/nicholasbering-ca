---
layout: post
title: AngularJS Style Guide
description: Short article highlighting a coding style guide by John Papa.  The helpful open-source guide is hosted on github.
category: AngularJS
tags: [AngularJS, style-guide]
last_modified_at: 2016-05-20 00:14:22 2016 -0400
author: Nicholas Bering
---

The project I've been working on lately has grown considerably since I first
started it. That's a pretty good sign that I'm being productive and producing
features and content... but it is at the same time a little bit scary.

This project happens to be my first web app written with AngularJS, and the
largest and most complex JavaScript code base I've ever written. I think its
pretty understandable that some of my code is a mess.

Messy code is inevitable for inexperienced programmers. You need to make a mess
before you can understand what a mess is and how you got there. The real
problem is when you make a mess, know you have one, but have no idea what clean
code looks like or how to write it.

### Enter the Style Guide

My messy code was just reaching the point where it was noticeably affecting my
productivity when I found the [Adventures in Angular] podcast. In [one episode][AiA018],
the panel introduced an [AngularJS-specific coding style guid][AngularJS Styleguide]
posted on GitHub by [John Papa].

It wasn't some magic wand tool that I could wave and my code would be tidy.
It was better than that.  It introduced certain concepts and patterns that would
keep my code readable, understandable, and easy to maintain.  Every style rule
also include examples of the pattern to be avoided, what could/should look like,
and detailed explanations of why its better that way.  It helped me clean up my
code and avoid writing any more messy code.

Since reading this guide, I start every new code file by following some of the
basic rules from this guide. I still have some messy code lying around. So,
whenever I have to touch some of my older code I take a few minutes to refactor
the existing code to follow the rules.  I try to make a habit of putting that
refactoring step into it's own git commit, so that looking back I can see what
changes are new code, and what was refactoring.

I had to screw up a little first in order to see the value of this guide, and to
even understand the concepts in it.  I wouldn't say that this guide is for
developers that are brand new to AngularJS. However, if you've been using Angular
long enough to have a firm grasp of the basics, reading this guide will give you
a push in the direction of cleaner code.

### TL;DR

Read John Papa's [AngularJS Style Guide][AngularJS Styleguide]. It teaches you
to write cleaner AngularJS code.

### Links

* [AngularJS Style Guide][AngularJS Styleguide]
* [Adventures in Angular Podcast][Adventures in Angular]
  * [Episode 18 - Style Guides][AiA018]

[Adventures in Angular]: <https://devchat.tv/adv-in-angular/>
[AiA018]: <https://devchat.tv/adv-in-angular/018-aia-style-guides/>
[AngularJS Styleguide]: <https://github.com/johnpapa/angularjs-styleguide>
[John Papa]: <https://github.com/johnpapa>
