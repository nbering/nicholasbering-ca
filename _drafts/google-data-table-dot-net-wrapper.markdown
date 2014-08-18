---
title: Google DataTable .Net Wrapper
description: A Google Visualization DataTable wrapper for building google chart objects from the server.
category: dot-net
tags: [google-visualization, asp.net]
---

So far my blog has focused on using the Google Charts API in AngularJS with Angular-Google-Chart.  While this a fantastic combination for a client-side solution, so far it doesn't help us at all on the server side.  In fact, if you're trying to take full advantage of the Google Charts API, your job just got a little more complicated.

Getting data into a Google Chart is pretty easy for simple or static data, but if you're working with complex, dynamic data the json formatted tables needed are a lot of work to set up.  For starters, the structure is kind of a collection of arrays that are logically linked.  Setting something like this up on either the server or client side takes a fair bit of code.  Also, the date format used by the API is not standard (at least, its not any kind of standard I've ever seen).

In my search for a simpler solution, I came across the Google DataTable .Net Wrapper.  This helper/conversion library, available on Nuget, makes it a lot easier to get a valid Google DataTable to send off to the client.  It will even convert the standard .Net DataTable class, so there's a good chance you'll be able to adapt some of your existing data access code to new purpose.