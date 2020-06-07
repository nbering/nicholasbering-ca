---
layout: post
title: TypeScript for Small Projects, Too!
description: At my previous job working with Back IO, I was working almost exclusively with TypeScript. It was certainly an eye-opening experience. In this article, I provide a bit of an update on the TypeScript developer experience.
category: Jekyll
tags: [TypeScript]
author: Nicholas Bering
---

At my previous job working with [Back IO][1], I was working almost
exclusively with [TypeScript][2]. It was certainly an eye-opening experience.

## Early Adopters

At the time we were tracking closely with all the changes, sometimes even running
with the nightly builds to get that one more feature we needed. Honestly, it's
probably the closest to the bleeding edge I've ever been, and sometimes we did
get cut a little.

During my work on that team, tracked along changes from 1.6 to 1.7, and on to
1.8. We even worked with some of the preview builds for 2.0.

## Not For Small Projects?

During that time, it was my opinion that TypeScript was great for larger
projects, like your company's main product. I found that pre-2.0, it was very
cumbersome to set up new TypeScript projects. Especially with NodeJS.

When I found myself making a quick one-off scripts for a proof-of-concept or
a small sub-project, I would avoid using TypeScript unless the scale of the
project was worth the setup overhead.

What made setup so bad?

### Build Script Setup

This one's not entirely solved today, but it's improve *a lot*. Setting up
your build scripts was not always easy. For example, when we were depending
on TypeScript 1.8 features, a lot of the supporting tooling like grunt and
gulp build tasks would have hard dependencies on version 1.6. TypeScript
was moving very fast, and not everyone was so invested as to provide tooling
support for the latest versions.

Some tooling provided good support for loading settings from tsconfig.json,
others ignored it. This became an issue for the split versioning I mentioned
earlier. Some tooling did not support new build options I was using, even
when I forced them to use the correct typescript version.

### Debugging

Debugging support wasn't always great either. Post-processing tools like
[UglifyJS][4] didn't always have good support for input source-maps, if
they had support for source maps at all.

To provide mappings for stack traces in node, [source-map-support][5] needed
to be loaded as one of the first things in the application. This was just yet
another dependency when starting a new project.

[Visual Studio Code][6] was and still is the best debugging experience I've had
for source-mapped javascript. I was an early adopter of VSCode, and while I
can't say the TypeScript debugging support was always flawless... it was
generally better than the alternatives at the time.

### Type Definitions

TypeScript in a vacuum would provide a marginal benefit for a small project,
especially with language support engines like Salsa that give much of the same
experience without inline type annotations. In fact, even TypeScript itself
can understand JSDoc comments, so there's no need to go all in with TypeScript
if you don't want.

Where TypeScript really shines is when you include the type definitions for
libraries you consume.

In the pre-2.0 days, it managing type definitions was a pain. There was a huge
community of developers contributing definitions to [DefinitelyTyped], but
because they were third-party contributions they were not always up-to-date,
and being manually assembled there was often quite a lot of style difference
from one contributor to another.

In addition, there were multiple tools available to manage types. VSCode would
offer to download types for a library you were using even just in JavaScript,
which was awesome. Before that there was [tsd][8], which was deprecated in
favour of [typings][9]. Typings did solve a lot of fundamental issues from
tsd like version pinning.

Ultimately, in my opinion, the TypeScript language project itself solved
the definition [publishing][10] problem in the most elegant ways.

Today, the only tools I use to import types in npm. To install the type definitions
to NodeJS, all I need is:

```
npm install --save-dev @types/node
```

Looking for types for momentjs?

```
npm install --save-dev @types/moment
```

It's that simpe. This works because TypeScript 2.x added support for some
really good default lookup locations for types. No more wiring up references
with triple-slash comments and xml tags.

JavaScript projects can also provide a typings attribtute to their `package.json` file,
giving TypeScript a more direct hint to where to find the typings without downloading
any additional packages or files.

## So I Changed My Mind

So, I've changed my tune. Now I love TypeScript for everything. This is definitely
a case of good design, and developers breaking down barriers to use. For me, new
TypeScript project setup time has gone from over a day (and that's for my 3rd and 4th
projects), to about 15 minutes.

So if anyone asks me if TypeScript is worth the effort, I'd simply say "yes". No more
caveats.

[1]: http://back.io/
[2]: https://www.typescriptlang.org/
[3]: https://www.npmjs.com/package/ts-node
[4]: https://github.com/mishoo/UglifyJS
[5]: https://www.npmjs.com/package/source-map-support
[6]: https://code.visualstudio.com/
[7]: https://github.com/DefinitelyTyped/DefinitelyTyped
[8]: https://github.com/DefinitelyTyped/tsd
[9]: https://github.com/typings/typings
[10]: https://www.typescriptlang.org/docs/handbook/declaration-files/publishing.html
