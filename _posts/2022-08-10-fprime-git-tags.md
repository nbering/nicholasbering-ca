---
layout: post
title: F-Prime Release Tags (Hidden Dependency)
description: >
    While I was working through the Install Guide, I ran into a small issue.
    It was pretty easy to work around, but here’s what it was and how to fix it,
    in case someone else runs into the same problem.
category: fprime
tags: [F-Prime, Robotics, NASA]
---

While I was working through the [Install Guide][install], I ran into a small issue.
It was pretty easy to work around, but here's what it was and how to fix it, in
case someone else runs into the same problem.

## Symptom
At the [Checking Your Installation][check] step in the install guide, it has
readers run the groundstation command:

```
fprime-gds -g html -r <path to fprime checkout>/Ref/build-artifacts
```

I was getting an error, and then the flask application would exit.

```
[ERROR] Dictionary version  is not in supported range: 0.0.0-9.9.9. Please upgrade fprime-gds.
```

## Root Cause

Before I began, I decided to fork the F-prime GitHub repository, in case I
needed to make any file changes while doing the tutorials.

<p class="image-frame"><a href="{{site.baseurl}}/images/fprime-git-tags-1_big.png"><img src="{{ site.baseurl }}/images/fprime-git-tags-1_small.png" alt="GitHub fork repository form, with an arrow highlighting the 'Copy the master branch only' checkbox."></a>GitHub fork repository form. Not the "Copy the master branch only" checkbox, selected by default.</p>

This is where I made an error. By forking with only the master branch, I did not
receive the release tags in my fork.

After running generate and build...

```
fprime-util generate
fprime-util build
```

...I ended up with `Ref/build-artifacts/Darwin/dict/RefTopologyAppDictionary.xml`
as would be expected, but on line one we have a problem:

```xml
<dictionary topology="Ref" framework_version="7cb9feed0" project_version="7cb9feed0">
```

`fprime-gds` wants to compare the `framework_version` property of this dictionary
file to make sure it is compatible with this version of gds, but it doesn't know
how to compare a git commit SHA to the semantic version release tag.

A correct dictionary file would look like this, where 3.1.0 is the latest
release of F-prime:

```xml
<dictionary topology="Ref" framework_version="3.1.0-9-g7cb9feed0" project_version="3.1.0-9-g7cb9feed0">
```

## Work-around / Fix
All I needed to fix this was to pull down the git tags from the upstream repo.

```
$ git remote add upstream https://github.com/NASA/fprime/
$ git fetch upstream 
remote: Enumerating objects: 22769, done.
remote: Counting objects: 100% (16900/16900), done.
remote: Compressing objects: 100% (163/163), done.
remote: Total 22769 (delta 16776), reused 16778 (delta 16735), pack-reused 5869
Receiving objects: 100% (22769/22769), 18.89 MiB | 10.20 MiB/s, done.
Resolving deltas: 100% (21179/21179), completed with 734 local objects.
From https://github.com/NASA/fprime
 * [new branch]          devel                             -> upstream/devel
 * [new branch]          master                            -> upstream/master
 * [new branch]          release/documentation             -> upstream/release/documentation
 * [new branch]          revert-1553-feat/mermaid-deframer -> upstream/revert-1553-feat/mermaid-deframer
 * [new branch]          update/add-ref-ini                -> upstream/update/add-ref-ini
 * [new branch]          update/autodocs-null-commit-fix   -> upstream/update/autodocs-null-commit-fix
 * [new branch]          update/fix-tutorial-cmd-seq       -> upstream/update/fix-tutorial-cmd-seq
 * [new branch]          update/lgtm-fix                   -> upstream/update/lgtm-fix
 * [new branch]          update/linux-uart-refactor        -> upstream/update/linux-uart-refactor
 * [new branch]          update/sqls                       -> upstream/update/sqls
 * [new branch]          update/update-readme              -> upstream/update/update-readme
 * [new branch]          workaround/cycle-fpp              -> upstream/workaround/cycle-fpp
 * [new tag]             NASA-v1.00                        -> NASA-v1.00
 * [new tag]             NASA-v1.01                        -> NASA-v1.01
 * [new tag]             NASA-v1.10                        -> NASA-v1.10
 * [new tag]             NASA-v1.2                         -> NASA-v1.2
 * [new tag]             NASA-v1.3                         -> NASA-v1.3
 * [new tag]             NASA-v1.3.1                       -> NASA-v1.3.1
 * [new tag]             NASA-v1.4                         -> NASA-v1.4
 * [new tag]             NASA-v1.5.0                       -> NASA-v1.5.0
 * [new tag]             NASA-v1.5.1                       -> NASA-v1.5.1
 * [new tag]             NASA-v1.5.2                       -> NASA-v1.5.2
 * [new tag]             NASA-v1.5.3                       -> NASA-v1.5.3
 * [new tag]             v2.0.0                            -> v2.0.0
 * [new tag]             v2.0.1                            -> v2.0.1
 * [new tag]             v2.1.0                            -> v2.1.0
 * [new tag]             v3.0.0                            -> v3.0.0
 * [new tag]             v3.0.0-RC1                        -> v3.0.0-RC1
 * [new tag]             v3.0.0-RC2                        -> v3.0.0-RC2
 * [new tag]             v3.1.0                            -> v3.1.0
 ```

After running `fprime-util build` again, the new version of the dictionary has the
correct tags.

To make this fix permanent for the forked GitHub repository, you'd still need to
upload the tags to github.

```
git push origin [tag]
```

## Troubleshooting

How did I arrive at the fix?

Again, here's the error message:

```
[ERROR] Dictionary version  is not in supported range: 0.0.0-9.9.9. Please upgrade fprime-gds.
```

Because it talks about upgrading, I checked the repository's `requirements.txt`
to see what version of `fprime-gds` was in use.

```
$ grep fprime-gds ./requirements.txt
fprime-gds==3.1.1
```

At the time, this was a patch version behind the latest `fprime-gds`, but that
didn't seem to account for the error. The version was a match to the declared
dependencies.

So I searched `fprime-gds` repository for the error message and found the [line][msg]
where it was declared.

The one place it was used didn't make a lot of sense to me, so I did a `git blame`
on the error message to find the [commit][commit] where it was introduced.

The commit message says something about "dictionaries". I googled around a bit
for fprime and dictionaries and the "Ref" app, and eventually found the filename
of the dictionary. I'm missing some of my search history here for writing this
post because I was using multiple computers.

Since the framework version was on the top line, and I'd already seen the code in
GDS that was validating the version number, it immediately stood out that it was
not the right format, and I recognized that it was getting invalid output from `git`
because it didn't have tags.

## Bug?

There may be an actual bug here since `fprime-utils` can generate build output
that is not recognizable to `fprime-gds`. Or maybe it's just user error since I
accidentally removed the release tags from my source control history.

I may report this once I get a better sense of what repository would "own" this
bug. In the meantime, this is my bug report. ;)

[install]: https://nasa.github.io/fprime/INSTALL.html
[check]: https://nasa.github.io/fprime/INSTALL.html#checking-your-f-installation
[msg]: https://github.com/fprime-community/fprime-gds/blob/48efee990481e3c152c1b0fb5a50821a07a1ca55/src/fprime_gds/common/loaders/xml_loader.py#L416
[commit]: https://github.com/fprime-community/fprime-gds/commit/a7dab5fba3bd99ba18ef9fe0df387d49256c39e5
