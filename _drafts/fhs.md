---
layout: post
title: File Hierarchy Standard (FHS)
description: "Or: How I learned to stop worrying and love /opt."
category: Linux
author: Nicholas Bering
---

The File Hierarchy Standard (FHS) is a set of guiding principles for management of files on Linux systems. The standard is maintained by a working group of the Linux Foundation.

## A Little History

The File Hierarchy Standard was first released as Filesystem Standard (FSSTND) in 1994. The newer FHS name was adopted on version 2.0 in 1997. The current version is 3.0, having been announced on June 3, 2015.

FHS is a widely-adopted standard, in use by the majority of Linux Distributions, with a few notable exceptions.

## Summary of the Structure

- `/` root of the filesystem
- `/bin` essential binaries needed in single-user mode
- `/boot` bootloader and it's dependencies
- `/dev` directory for special "device" files, representing physical or logical devices
- `/etc` configuration files
- `/home` user home directories
- `/lib` shared libraries
- `/media` mounted removable media (CDs, USB drives)
- `/mnt` temporary mount points - programs should not depend on these mounts
- `/opt` "add-on" application packages - more on this below
- `/root` root user's home directory
- `/run` run-time variable data - cleared on boot
- `/sbin` System binaries
- `/srv` service data (ftp, webserver)
- `/tmp` temporary files
- `/usr` subdivision of the filesystem used in multi-user mode
  - `/usr/bin`
  - `/usr/include` headers referenced by source files
  - `/usr/lib`
  - `/usr/libexec`
  - `/usr/local` used for "locally-installed" software - may contain a hierarchy mirroring `/usr`
  - `/usr/sbin`
  - `/usr/share` static data files and documentation
  - `/usr/src` source code
- `/var` variable data that is typically not shareable between systems
  - `/var/account` data from linux process accounting
  - `/var/cache` computed data, or fetched from other systems - should be safe to delete
  - `/var/crash` system crash dumps
  - `/var/games` variable game data like game saves
  - `/var/lib` application data
  - `/var/lock` lock files
  - `/var/log` system and application logs
  - `/var/mail` mailbox data
  - `/var/opt` data for "add-on" applications
  - `/var/run` same as `/run` - in fact this is often a symlink to `/run`
  - `/var/spool` application "spool" data (print jobs, logrotate spool)
  - `/var/tmp` temporary data that is preserved between reboots
- `/proc` virtual filesystem used by Linux kernel to expose process information
- `/sys` virtual filesystem used by Linux kernel to expose driver and device info

## Why Do We Need a Standard?

Having a standard supports an orderly software packaging system. If software packages had no guidance about where to place files, it can lead to file placement conflicts, build errors from dependencies not being where they're expected to be, etc.

A standard also promotes a software environment friendly to administrators. It is significantly easier to diagnose and solve problems with a system if the file and directory layout is familiar.

## What About Software That Wasn't Designed for FHS?

The standard has some provisions for software that wasn't built to support the standard directory layout. Software in `/opt` is "add-on" software, and is - to some degree - exempted from the layout rules.

The two examples I've most commonly seen taking advantage of `/opt` are Java and Python applications.

Java, because the application often comes with many source library files and other supporting content that was laid-out for systems that do not use FHS (ie. Windows).

I've commonly seen Python applications installed into `/opt` when they are using a Python Virtual Environment (Virtualenv), which may include it's own version of Python, the Pip package manager, and versions of libraries which may differ from those installed "globally" on the system.

Command binaries installed in `/opt` are often added to the standard search path for binaries (the `$PATH` environment variable) by adding a symbolic link to `/usr/local/bin`. I'm not certain that this is correct according to the standard, but it generally achieves the intended effect, and is a fairly common practice.
