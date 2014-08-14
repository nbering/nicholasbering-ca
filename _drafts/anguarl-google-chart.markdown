---
layout: post
title: "Google Charts Directive for AngularJS"
categories: [AngularJS, Google Charts API, Chart, Data Visualization]
---
# Angular Google Chart Directive
### Background

There's no doubt about it, if you're working on a project that involves
the display of data, you're probably going to need or want a chart.  It's no
coincidence that just about every spreadsheet package ever has had some sort of
charting feature.  Unfortunately, if your display medium is the web browser,
there's nothing built in to solve this problem.

I did a lot of digging around to find a simple solution to this problem, and
despite the extremely high demand for such capabilities, there were not many
options available in my price range (about $0).  The packages I did find were
either very complicated to set up, or lacking features most of the features I'd
come to expect from using commercial charts on other sites.

Then I found the [Google Charts API][1].  It seemed to be the solution to all my
problems.  It was a full-featured JavaScript chart library, available for free,
with some caveats.  Specifically, it is free but not open source.  So the source
code is not available in any easily human-readable format.  Also, the code
itself is included not with a script tag directly.  You add a loader script
which you then call, providing this loader function with a callback function to
be run on completion.

This last part added some considerable complexity for a new AngularJS developer.
I did some hunting around and found a couple work arounds where you manually
bootstrap your Angular app in the loader's callback function.  This worked but
seemed unnecessary when the chart would not be displayed in every page of the
app.  Making my own directive wasn't exactly a piece of cake either.

### Enter the Directive

Then I stumbled upon the [Angular-Google-Chart][2] directive by
[Nicolas Bouil][3].  This simple directive takes much of the work out of the
initial setup of a basic Google Chart.  The directive comes bundled with a
service that handles the API Loader call asynchronously using the AngularJS
Promise API, so you don't have to worry about fetching the library and passing
it a callback function, the included code handles it for you.

For a very simple application you can just setup an object with your chart data
and options, and give that object to the directive from the markup side of
things.

```
//Insert Example Code Here.
```

This directive works pretty well for basic scenarios, but does not necessarily
support 100% of the features provided by the [Google Charts API][1].  To be more
precise, it does not support [Controls and Dashboards][4], as these
features require binding the Chart to a Dashboard and passes responsibility for
drawing the chart to the Dashboard.  It's a little complicated to explain but
basically the workflow is a little different and the directive isn't setup to
support it.

[1]: https://developers.google.com/chart/
[2]: https://github.com/bouil/angular-google-chart/
[3]: https://github.com/bouil/
[4]: https://developers.google.com/chart/interactive/docs/gallery/controls
