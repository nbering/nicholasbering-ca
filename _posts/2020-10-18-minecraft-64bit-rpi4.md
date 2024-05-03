---
layout: post
title: "Minecraft on 64-bit Raspberry Pi"
description: >
  After considerable hunting and troubleshooting, I managed to get the (nearly)
  latest version of Minecraft installed on a Raspberry Pi 4, running a 64-bit
  build of Ubuntu MATE.
category: raspberry-pi
tags: [Minecraft, Gaming, arm64, Java, Ubuntu]
author: Nicholas Bering
---

My daughters have Raspberry Pi 4 (8GB) desktop kits, which they use for online
learning, during these times of pandemic. But they also use them as a personal
computer for games, programming with Scratch, etc.

Here, I'll cover a bit about how I set them up with Minecraft. I'm going to
first cover a bit of the state of Minecraft on the Raspberry Pi... as much as I
can gather.

If you want, you can skip ahead to the [Feature Presentation](#feature-presentation). The next couple of sections help to set up context for anyone in the future
who needs to reverse-engineering my process or make improvements.

## State of Minecraft on RPi

Some time ago, I set my daughters up with Minecraft 1.12.2. I followed along with
[How to setup Minecraft 1.12.1 on Raspberry Pi 3](https://www.raspberrypi.org/forums/viewtopic.php?t=186547),
or perhaps another post like it.

Here's kind of how this setup process works, in a nutshell:

1. Set up Raspberry Pi graphics and driver dependency
2. Install an old version of the Minecraft Launcher, from before Mojang updated
    to a version that no longer runs in Java
3. Run the launcher to download the game files
4. Try to run the game (get errors)
5. Download some drivers from Dropbox, OptiFine mod, and a run script that glues
    this all together
8. Edit run script with Mojang account, username and password
9. Run Minecraft (at last)

### LWJGL on ARM

Step 5 is the interesting part here. I've later learned that the driver being
downloaded from dropbox is for an armhf build of the native code dependencies for
LWJGL.

Minecraft is written entirely in cross-platform Java, so it can theoretically
run on any machine. But 3D graphics acceleration, sound, and input devices are
all a little different on different hardware and operating systems. LWJGL is a
collection of Open Source libraries that allow Minecraft to remain cross-platform
without handling all of those differences themselves.

The trouble is, LWJGL did not support the ARM CPU architecture, used by the
Raspberry Pi, until the most recent version at the time of this writing.

So how did they get this to work on the Raspberry Pi 2, 3, and 4? It's this magic
file that is downloaded from Dropbox during the above procedure. It's a build of
a now-older version of LWJGL, but as far as I've been able to find so far... no
one seems to know who built it or how they managed to do it.

### Minecraft 32-bit Support (discontinued)

The other thing to note here, is that Minecraft 1.12.2 was the last version of
Minecraft to support 32-bit operating systems. So if you were following this old
set of instructions to set up Minecraft on the Raspberry Pi... you wouldn't have
any of the multitude of features added to the game since September 2017.

The Raspberry Pi 2's Cortex A7 was a 32-bit CPU, so on that hardware there was
no other choice. But starting with Raspberry Pi hardware has been using a 64-bit
Broadcom chip since the Raspberry Pi 3.

Unfortunately, upgrading a Linux distribution from 32-bit to 64-bit is no small
task. And we didn't see a 64-bit base image of the official Raspberry Pi OS until
earlier this year. When I last checked in on it, this version was still Beta, with
a long list of known issues - ome of them pretty serious. I might have tried this
version myself, but it wasn't something I'd put my daughters through.

There are other Linux distributions now available for the Raspberry Pi, though...
and a few of them are ahead of the Raspberry Pi Foundation for 64-bit support.

## Getting to 64-bit Minecraft on RPi

### Selecting the Base Image

Once I installed Minecraft 1.12.2 on the Raspberry Pi 2 for my daughters, I
basically just set it up and left it alone. Installing it initially was enough
work and I didn't want to risk breaking it.

So the lack of updates never really hit me, until we got the Raspberry Pi 4.
When I followed along on instructions for the RPi 4, I found they were all still
installing 1.12.2, despite the latest version of Minecraft being 1.16.x.

I wanted to fix this, but with 64-bit support being beta for Raspberry Pi OS, I
had to pick another Linux distribution.

My choice of Ubuntu MATE was driven entirely by a different requirement, coming
from the devices use for online learning...

Our school board was using Microsoft Teams for classes... and the only way I
could get it to work on the RPi was to update to the latest version of Open Source
Chromium. Almost all of the Linux distributions I checked were a major version or
more behind the official Chromium version used to build Google Chrome. Only
Canonical's builds of Chromium for Ubuntu seemed to be tracking close enough to
the latest version to support Microsoft Teams.

### LWJGL for Arm64

I'm going to skip over some troubleshooting here, and jump to LWJGL. In-between
selecting the base image and here, I backed-up and tried installing Minecraft
with some variations on the tried-and-tested instructions. These did not work
because the ARM build of the LWJGL binary from Dropbox was out-of-date compared
to the latest LWJGL used by newer versions of Minecraft.

Then I backed-up again, and tried to install Minecraft the way you would on any
other Linux machine. That didn't work because the newer Launcher was no longer a
Java program, and wasn't built for ARM. In theory, you can run the Java game
without the Launcher, but there's no reasonable way to get all the files needed
to run the game without it.

While trying to combine the two approaches, my Google Keywords hit upon [this AskUbuntu post](https://askubuntu.com/q/1211821/1138343).

This connected the dots to MultiMC - an Open Source implementation of the
Minecraft Launcher. This eventually solved both the Launcher
problem... and through a [fork of the metadata](https://github.com/JJTech0130/meta-multimc/blob/da0b924eecab8dbaec8e058ddd79ae3e36e4e396/org.lwjgl3/3.2.3.json)
for MultiMC it also gave a solution for downloading and using the arm64 binaries
for the latest LWJGL version.

## Feature Presentation

Here's the actual instructions for repeating what I did to get Minecraft 1.16.1
working on the Raspberry Pi 4, with Ubuntu MATE. Thanks for hanging in, for
anyone who read all of the above background. If you didn't... that's ok, too.
These instructions should be pretty self-contained.

This tutorial is starting from a freshly-imaged SD card, from the Ubuntu MATE
20.04.1-beta2 image, with the first boot setup completed, and all package
updates applied.

I won't get into the instructions for installation here. You can find them on
the [Ubuntu MATE for Raspberry Pi](https://ubuntu-mate.org/ports/raspberry-pi/)
project website.

### Install Dependencies

The first few steps will be done from the Terminal. On a Ubuntu Desktop, you can
bring it up any time with the shortcut keys `Ctrl + Alt + T`.

First... install the package dependencies. This list includes everything for
building the MultiMC launcher, and running Minecraft.

```
sudo apt-get install \
  build-essential \
  cmake \
  qt5-default \
  qtbase5-dev \
  zlib1g-dev \
  libgl1-mesa-dev \
  openjdk-8-jdk \
  git
```

### Build MultiMC from Source

Following along with the official [MultiMC5 Build Instructions](https://github.com/MultiMC/MultiMC5/blob/develop/BUILD.md), here's what we're going to run from the
terminal. I'm going to do things _slightly_ out of order to make it easier to
follow in my breakdown for those who want to know what's going on here.
```
mkdir ~/MultiMC
cd ~/MultiMC
git clone --recursive https://github.com/MultiMC/MultiMC5.git src
mkdir build
mkdir install
cd build
cmake -DCMAKE_INSTALL_PREFIX=../install \
  -DMultiMC_META_URL:STRING="https://jjtech0130.github.io/meta-multimc/" \
  ../src
make -j4
make install
```

If you just ran all that and don't care for a description, just skip ahead to
[running MultiMC](#Run-MultiMC-and-First-Time-Setup).

1. Create a new directory at `/home/username/MultiMC`, and set it as our working
directory.
    ```
    mkdir ~/MultiMC
    cd ~/MultiMC
    ```
2. Download the source code for the MultiMC launcher project into
`/home/username/MultiMC/src`. We use `--recursive` because MultiMC uses git
submodules, which are basically repositories within repositories. After cloning
MultiMC, it does the same for all submodules.
    ```
    git clone --recursive https://github.com/MultiMC/MultiMC5.git src
    ```
3. Create a `build` directory for the all the files generated for the build
process, and an `install` directory where the final program will be installed,
and where we will run MultiMC from.
    ```
    mkdir build
    mkdir install
    cd build
    ```
4. Now we use cmake to generate all the build configuration files. This basically
looks for all the dependencies we installed earlier and wires them up for the
compiler step that comes afterward. There's one notable addition here:


    `-DMultiMC_META_URL:STRING="https://jjtech0130.github.io/meta-multimc/"`.
    
    This overrides the URL where MultiMC will fetch information about what
    versions of Minecraft exist, and where to download the files to install them.
    GitHub user JJTech0130 has done the heavy lifting of setting up the overrides
    here to install LWJGL that were mentioned in the background above, so that
    we get LWJGL 3.2.3 instead of 3.2.2, with the ARM builds for Linux.

    ```
    cmake -DCMAKE_INSTALL_PREFIX=../install \
      -DMultiMC_META_URL:STRING="https://jjtech0130.github.io/meta-multimc/" \
      ../src
    ```
5. Finally, we compile MultiMC, and install it to `/home/username/MultiMC/install`.
    ```
    make -j4
    make install
    ```

## Run MultiMC and First-Time Setup

Now that we've built MultiMC from source, we should have a working copy that we
can run on our Raspberry Pi's ARM processor. You can now run it on the command
line, or create a menu item for it. If you've followed along above, and there
were no errors, there will be an executable at:

```
/home/username/MultiMC/install/MultiMC
```

There's lots of screenshots coming up here, following along the first-time
setup of MultiMC.

### 1. Language Selection

Multi-MC has a wide-spread international community that has contributed
many translations of the interface text. Choose your preferred language here.

It probably goes without saying that I've chosen English and the screenshots
will reflect that.

<p class="image-frame">
    <img src="{{ site.baseurl }}/images/MultiMC-01_Language_Selection.png"
        alt="MultiMC language selection dialog, showing a large collection of languages and a translation completion percentage.">
</p>

### 2. Java Options

The second screen MultiMC will show on first run is for some Java options. If
you've installed the `openjdk-8-jdk` package during the dependencies step while
we were building MultiMC, then all of the defaults should work for you here.

<p class="image-frame">
    <img src="{{ site.baseurl }}/images/MultiMC-02_Java_Options.png"
        alt="MultiMC java options dialog. Provides settings for the path to the Java executable, and the min and max memory allocation values for the JVM heap.">
</p>

### 3. Analytics Disclosure

As any good data steward of analytics data should, the MultiMC developers are
disclosing here what data they are collecting, and giving you a chance to opt-out.

Sending them device information like - _ahem_ - the fact that you're running on
an ARM processor, will help them prioritize development and bugfix resources to
the most commonly-used platforms.

<p class="image-frame">
    <img src="{{ site.baseurl }}/images/MultiMC-03_Analytics.png"
        alt="MultiMC's analytics disclosure and opt-out screen.">
</p>

### 4. Main Page

Yay! We're done the first-launch wizard steps. Now we can start setting things
up. The first thing I suggest you do is configure your Mojang account. You'll
need this for MultiMC to set up a session using your purchased Minecraft license.

<p class="image-frame">
    <img src="{{ site.baseurl }}/images/MultiMC-04_Main_Page.png"
        alt="MultiMC's main page, with the Profiles menu open to Add Account.">
</p>

### 5. Add Account

The main page's profiles menu will take you to the Accounts section of the
MultiMC settings window. You can use this page to manage multiple Mojang accounts.

<p class="image-frame">
    <img src="{{ site.baseurl }}/images/MultiMC-05_Accounts_Empty.png"
        alt="MultiMC's Account settings page, where you can manage your Mojang account info.">
</p>

If you haven't already purchased Minecraft, go do that. This tutorial is _not_ a
guide on how to pirate Minecraft, nor would I condone doing so. Your money helps
to pay for new Minecraft features and bugfixes. And while I'm not holding my
breath... maybe someday we'll get an official Arm64 build of Minecraft if enough
paying customers are already running it.

Click "Add" on the right to get the dialog where you can enter your email and
password.

<p class="image-frame">
    <img src="{{ site.baseurl }}/images/MultiMC-05_Add_Account.png"
        alt="Add Account dialog, with email and password fields for Mojang login credentials.">
</p>

Once you've entered your account info, we're done in this settings window, and
you may close it.

### 6. Add Instance

As the name suggest, MultiMC is designed to run multiple copies - or _instances_
of Minecraft. We're not necessarily using it for that purpose, but we'll need to
set up at least one.

Click the "Add Instance" button on the top-left of the main page.

<p class="image-frame">
    <img src="{{ site.baseurl }}/images/MultiMC-06_Button_Add_Instance.png"
        alt="The Add Instance button on the top-left of the main page will open the Add Instance dialog window.">
</p>

This will open a dialog where we can choose from some options for our game
installation.

<p class="image-frame">
    <img src="{{ site.baseurl }}/images/MultiMC-06_New_Instance_Form.png"
        alt="The Add Instance dialog without any changes made to the defaults.">
</p>

The "name" field defaults to the version number, and is also the directory name -
and what you'll call it from the command line, if you want to launch directly
into Minecraft without opening the MultiMC GUI.

I personally called my instance "minecraft", since I'm only running one.

You can also pick a specific version of Minecraft to run, here. I have only
tested this with version 1.16.1. Versions that depend on a version of LWJGL other
than the one that JJTech1301 configured for us in the forked Meta repository may
not work correctly at this time.

Click "Ok" to create the instance.

> **Note**: At the time I'm writing this, the latest version of Minecraft is 1.16.3. So you
may notice that the latest version here is a little out-of-date. That's because
we're using a _fork_ - or copy, with changes - of the official MultiMC Minecraft
release meta-data in order to get our arm64 build of LWJGL for the Raspberry Pi.
>
> Unfortunately, these meta-data files are getting a little behind on that fork.
I'm personally hoping to spend some time on getting these updated, or getting the
LWJGL changes made available on the main branch of MultiMC, but at the moment we
will need to accept being a _little_ behind. At least we're not on 1.12.2 anymore
like we were on Raspberry Pi OS.

### 7. Launch Minecraft

Back on the main page again, we should see an icon for our new instance of
Minecraft. With it selected, click "Launch" from the menu on the right-hand
side of this main page.

<p class="image-frame">
    <img src="{{ site.baseurl }}/images/MultiMC-07_Launch_Button.png"
        alt="The main page again, this time populated with our new instance of Minecraft. The right-hand menu now has some options enabled for working with our instance, including the Launch button to start a game.">
</p>

A series of progress bars should pop up, as MultiMC downloads content from
Mojang's servers. This is _the main thing_ that we needed MultiMC to do. This
is what Mojang's Launcher needed to do for us, but they don't provide an arm64
build.

<p class="image-frame">
    <img src="{{ site.baseurl }}/images/MultiMC-07_Launcher_Assets.png"
        alt="A progress bar showing MultiMC downloading the Minecraft asset files.">
</p>

If everything goes well, the progress bars should be followed by this beautiful
Mojang-branded loading screen.

<p class="image-frame">
    <img src="{{ site.baseurl }}/images/MultiMC-07_Mojang_Loading.png"
        alt="A progress bar showing MultiMC downloading the Minecraft asset files.">
</p>

If they did not, you will probably get a notice that suggests you look at the
logs to see why Minecraft exited suddenly. Unfortunately, I don't have the
capacity in writing those article to cover troubleshooting. If you encounter any
errors up to this point that you cannot troubleshoot yourself, head over to the
MultiMC discord community. There is a link for that on the [MultiMC project website](https://multimc.org/).

They're a helpful bunch of volunteers, and helped me get as far as I did to write
this article. Consider donating while you're there. They really are dedicating a
lot of time and technical resources to keeping this project running, and
no small part of that goes to supporting and troubleshooting for users.

### 8. Minecraft Video Settings

We should not have Minecraft running, and open to the main menu. From here,
click "Options...".

<p class="image-frame">
    <img src="{{ site.baseurl }}/images/MultiMC-08_Minecraft_Menu.png"
        alt="The Minecraft main menu screen.">
</p>

Next, select "Video Settings" from the left column.

<p class="image-frame">
    <img src="{{ site.baseurl }}/images/MultiMC-08_Minecraft_Options.png"
        alt="The Minecraft options menu. The video options button is in the left column.">
</p>

Before starting a game, you will need to update the video settings. Even with
the new 8GB models of the Raspberry Pi 4 Model B, the hardware is severely
under-powered for the default video settings.

<p class="image-frame">
    <img src="{{ site.baseurl }}/images/MultiMC-08_Video_Settings.png"
        alt="The Minecraft video options screen with all of the options turned down to minimum quality, in order to improve performance on the Raspberry Pi's weak hardware.">
</p>

The image below shows the settings I've used. Basically... just turn everything
down to the lowest setting. The one other setting I use is "Fullscreen: ON", but
it was difficult to take a screen capture in full-screen.

### 9. Performance Expectations

The Raspberry Pi is _absolutely not_ a high performance game system. It _will_
run Minecraft Java Edition... but not at a very high resolution or frame rate.

Below is a screen capture of the "F3" debug menu running with Minecraft in a
pretty small window.

<p class="image-frame">
    <img src="{{ site.baseurl }}/images/MultiMC-09_Minecraft_Debug.png"
        alt="The Minecraft debug text overlay, showing Minecraft running at 24 frames per second.">
</p>

I've found that 20 FPS is fairly normal, once the game is loaded and running...
but when it first starts up, 5-10 FPS is pretty common. Entities that take a
little more performance to render, like Animals and Monsters will also eat at
the performance a little while on-screen.

Using the [OptiFine](https://optifine.net/home) mod may help improve performance.
MultiMC has a [wiki page about OptiFine](https://github.com/MultiMC/MultiMC5/wiki/MultiMC-and-OptiFine),
but this seems to be out of date. Some information I was reading in their GitHub
issues suggests that recent versions have conflicting requirements compared to
the Wiki instructions.

## Thank You

Thank you for taking the time to read this. At the time I wrote this, I could
not find comprehensive instructions on how to run Minecraft versions above 1.12.2
on the Raspberry Pi 4... so I hope this helps some people get more out of their
Raspberry Pi.

I'd also like to thank the MultiMC team for help troubleshooting my installation,
and JJTech0130 on GitHub for doing the heavy lifting of getting LWJGL arm64
builds to work with MultiMC.
