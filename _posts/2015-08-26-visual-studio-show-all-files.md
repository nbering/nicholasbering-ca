---
layout: post
title: "Visual Studio Quick Tip: Show All Files"
description: A little feature of solution explorer that I let go unnoticed for years, that I couldn't live without.
category: visual-studio
date: 2015-08-26
last_modified_at: 2016-05-20 00:14:22 2016 -0400
author: Nicholas Bering
---

Last night, before going home, I made a quick commit to save my work in progress so I could pull my changes in at home and keep working. Sometimes I run into a little issue if I'm not paying attention, and Visual Studio has made changes to the project file, but hasn't saved them to disk.

I had added a new folder to my project, with a few code files in them.  When I pulled the changes at home, the folder was added to my working directory, but wasn't included in my project.  I couldn't create the folder again, because it already existed on disk. This had my stumped.

## Solution: Solution Explorer

<p class="image-frame"><img src="{{ site.baseurl }}/images/vs2015-show-all-files.png" alt="Solution Explorer show all files option">"Show All Files" option found on the top of solution explorer.</p>

Honestly, I hadn't even noticed before that solution explorer has a "Show All Files" option. Turning this on show everything in your solution folder that isn't part of the project. I have caught myself digging through my project with windows explorer to view files in my node_modules or bower_components folder before. I don't like to add these folders to my Visual Studio Projects because they change dynamically outside the editor on a regular basis. Had I known the option existed, I would have been using it all along!

<p class="image-frame"><img src="{{ site.baseurl }}/images/vs2015-only-project-files.png" alt="Solution Explorer with only project files"> Solution Explorer with only project files show.</p>

If you set "Show All Files", you can select an existing folder that is not part of the project, and "Include In Project". This solved my issue, but it may also come in handy in the future when manually adding third-party libraries, or just viewing the node_modules folder.

<p class="image-frame"><img src="{{ site.baseurl }}/images/vs2015-all-files-shown.png" alt="Solution Explorer with all files shown"> Solution Explorer with all files displayed.</p>
