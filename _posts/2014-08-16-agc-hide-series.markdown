---
layout: post
title: Hide a Series with Angular-Google-Chart
category: AngularJS
tags: [AngularJS, Google-Charts-API, Google-Visualization, Angular-Google-Chart]
description: In my previous post - Google Charts Directive for AngularJS - I introduced the Angular-Google-Chart Project.  In this post I'm going to skip some of the basics and go straight to something advanced that I found to be in demand for both this directive and the Google Charts API in general.
date: 2014-08-16 15:53:00
---
In my previous post - [Google Charts Directive for AngularJS][1] - I introduced
the [Angular-Google-Chart Project][2].  In this post I'm going to skip some of
the basics and go straight to something advanced that I found to be in demand
for both this directive and the [Google Charts API][3] in general.

Sometimes when you're displaying data in charts, you just have too much data to
show at once.  The chart gets very cluttered and difficult to read (especially
in line charts, because the lines will usually overlap), but you just *have* to
show the weekly, four-week, and twelve-week rolling averages.  How can your user
possibly make a decision without considering all these figures, right?  Well,
they probably don't need them all at once so it might be nice to hide one or
more lines so that they can focus on what is important to them right now.

Fortunately, the directive we're using exposes a way to work with the On Select
event provided by the Google Charts API.  We pass a function call to the
directive's on-select property and it will give us the row and column id's of
the selection.  If there is no row id, then the user selected the legend.

In this application, all we care about is when the user selects the legend.
When they do, we will replace the view of the column with a calculated column,
and assign it's colour a grey tone to make it clear that the column is hidden.
If the legend item is clicked again, we reverse the change and display the data
again.

Here's the code:

{% gist Balrog30/b0de4dfcc36944bdf8e8 index.html %}

{% gist Balrog30/b0de4dfcc36944bdf8e8 app.js %}

[View the example on Plunker.][4]

[1]: {% post_url 2014-08-14-angular-google-chart %}
[2]: https://github.com/bouil/angular-google-chart/
[3]: https://developers.google.com/chart/
[4]: http://http://embed.plnkr.co/lOXTg5XRggwdctUedvfl/preview
