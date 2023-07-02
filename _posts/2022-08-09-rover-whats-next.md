---
layout: post
title: "Star Rover: What's Next?"
description: >
    Almost 2 years ago now, I set out to build a clone of the Sawppy Rover project.
    As a robotics chassis inspired by the Curiosity Mars Rover, it's proven to be
    fairly capable as a platform.

    The base robot chassis has been finished construction for some time now. And it's
    time to set some new project goals to expand on what I learned from the base
    project.
category: Robotics
tags: [Robotics, Rover, 3D-Printing, Sawppy]
---

Over 2 years ago now, I set out to [build a clone of the Sawppy Rover project][intro-post].
As a robotics chassis inspired by the Curiosity Mars Rover, it's proven to be
fairly capable.

The base robot chassis has been finished construction for some time now. And it's
time to set some new project goals to expand on what I learned from the base
project.

I have a things I can improve on:
- Physical Reliability
- Control Software
- Sensors and Vision
- Remote Manipulator

I have at least some background research tasks going along for each of these
areas, and for some I've even acquired components to experiment with.

## Physical Reliability
### Stronger Drive Components
The biggest weakness in the base Sawppy design is the use of 3d-printed components
in the "drive train". The original Sawppy rover was quickly upgraded to PETG for
stronger components.

I may experiment with that myself, but have so far stuck to PLA because it's
simpler to work with. PETG has a higher melting point, tends to be stringy,
absorbs moisture from the air very quickly, and can stick to the print bed so hard
that it damages the special coating on it.

That said, the PLA components have been remarkably reliable - except for a few
weak points.

The screws in the servo couplers, and wheel hubs, need to be pretty tight since
they hold onto the detent in the drive shaft to transfer the motor torque for
drive and steering. Eventually, the tight screw starts to push the brass insert
out of the plastic.

I have some things I'd like to try in order to improve on this:

- Removing inserts from drive component designs
  - Could a platic thread be strong enough on it's own?
  - Is the weak material bonding between the insert and actually PLA making it less reliable?
- Improving the heat-set insert installation procedure
  - The variation between 3D printers causing tolerance problems for heat set
    inserts seems to be a [known issue](https://newscrewdriver.com/2020/11/25/sawppy-issue-heat-set-insert-shaft-coupling/)
  - Using a hotter iron during installation might help produce a stronger bond
- Upgrade to PETG
  - This may work, but I have to configure my printer correctly for the new material
  - I'm also worried about moisture levels quickly degrading PETG quality
- Machined Components
  - Failing all else, switch materials and use traditional manufacturing methods
  - If the detent cuts in the drive shaft were very precise, I might be able to
    use precision-cut D-shaped hole in the couplers, so that the set screw is just
    holding the part from falling off, rather than also supporting rotational force

## Control Software

The Sawppy project included a control sytem built as a Python web app. It's a
great demonstration of the working chassis but has some problems and limitations:

- Uses an outdated version of Python
- Has no built-in support for sensors or camera feeds
- Tends to jerk the steering motors a lot, causing joints to come loose

In choosing a framework to build my own improved software controls on, I'm looking
at two options:
- [ROS2 (Robotics Operating Sysstem)][ros2]
- [NASA's f'][f-prime]

### ROS2
I did a little bit of and evaluation of ROS2. I definitely haven't done enough
with it to provide a proper review, but here as some points on my first
impressions:

- Linux installation included an incredible number of packages, so it's a bit heavy
- Very modular design:
  - Components seem to be languge-agnostic and communicate over RPC, which
  - This is incredibly powerfull, but also adds to complexity for understanding
    even simply applications
  - Individual components seem to be able to run on low-end hardware, but the core
    system needs a fair bit of compute and network
- There is a diverse range of existing software components
- There seems to be a large community providing documentation and support
- Not all components live up to the same documentation and portability standards
- There are existing community software configurations for Sawppy on ROS/ROS2

### NASA f' (f-prime)
F' is a real, flight-tested framework. It powers the Ingenuity flight demo on the
Mars 2020 mission, as well as a number of smallsat projects.

I've looked at a number of NASA Open Soource projects... and honestly, most are
terribly-documented. This one seems to be the exception. It's the most
approachable and well-documented NASA Open Source project I've looked at to date.
It has really great getting-started docs, and even some walkthrough tutorials
based on the Raspberry Pi - which is nice, since the rover controls are already
based on the Raspberry Pi.

Unlike ROS, it's language-specific. It's built on modern C++, with a lot of python
build utilities for code generation and a simple ground-station software suitee.
This improves developer productivity and helps to build a modular and testable
framework.

### Picking One
Both framworks are pretty complex, so as a hobby project I need to pick one. I
simply don't have thee time for a deep-dive on two frameworks.

If it was purely about making the rover work - which was the goal I set for the
chassis build - I would definitely choose ROS2. Th community support for ROS2
and existing Sawppy projects would be a big boost.

However, I did not find a lot of joy in the "getting started" with ROS2. Whereas,
in going over the NASA f-prime documentation... I felt sort of connected to the
space industry. Getting a little peek into what matteres to NASA's developers.

So I think I'm going to deep-dive into f' and see if I can make it work.
I'd also like to try connecting it to thee open-mct mission control framework,
if that's even possible.

## Sensors and Vision
The original Sawppy design used a webcam or Xbox 360 Kinect for video telemetry.
It didn't have a self-driving functionality, or computer vision functions.

Since there was no integrated support for sensors or vision in the original
Sawppy software suite, it would necessarily involve some of the software
improvements above, to integrate new hardware inputs.

I'd like to add visual telemetry for sure, but also some typical sensor inputs
for things like temperature, pressuree, humidity, etc. Just for scientific
completeness.

## Remote Manipulator
Lastly, I'd like to add an arm to my rover. I've started on this somewhat, but
it turns out to be a rather complex and potentially expensive component to build.

I would want something that resembles at least the basic characteristics of the
Curiosity and Perseverance rover arms, but with 5 degrees of freedom, that's a
lot of motors.

If I use the same servo motors as the drive and steering, I can avoid adding a
lot of extra sensors. But I have my doubts about their ability to handle the load
needed for operating the whole arm.

I'm slowly forming ideas on how to build the arm, but have some more research and
experimentation to do.

## So What's Next?
At the moment, I'm actively working on all of the above in parallel. Most of it
is big dreams, and maybe I'll never finish any of it. But it's still a fulfilling
hobby, so I'll keep at it as long as it remains one.

For this week I'm primarily focused on replacing and improving failed structural
or drive components, and playing with the getting-started guide for f-prime.

[intro-post]: {% post_url 2020-08-01-new-rover-project %}
[ros2]: https://docs.ros.org/en/foxy/index.html
[f-prime]: https://nasa.github.io/fprime/
