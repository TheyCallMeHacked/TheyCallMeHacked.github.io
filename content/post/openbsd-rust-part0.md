+++
title="Rewrite it in Rust: OpenBSD base package edition - Part 0"
date = 2023-09-15

[taxonomies]
categories = ["Programming"]
tags = ["rust", "openbsd"]
+++

If you've been around the Rust community, you have probably heard the
trope "rewrite it in Rust". That's what we're gonna be doing today with
OpenBSD's base system package.

<!-- more -->

# Notice:

Just a note before we start: I am affiliated in no way with either the
OpenBSD nor the Rust Projects or Foundations. I'm just a student who
wants to have fun. Especially, don't harass OpenBSD developers to
include any of my code: It's trash and they probably wont want to
include it anyway (see Theo de Raadt's [view on Rust](https://marc.info/?l=openbsd-misc&m=151233345723889&w=2)).

This isn't a fork of OpenBSD either, but simply a way for me to toy
around. Again, please treat everyone with respect.

# Motivation

My motivations are simple and mainly twofold:

1. Learn Rust
2. Learn OpenBSD

That's it. Now to get a bit more into the details: I want to get better
at Rust. I have been fully convinced, that Rust, while far from a
perfect language, is the direction modern Programming should go. And
while I believe its "no breaking changes" policy is doomed to make it a
bloated mess in about as much time as it took C++ to become said bloated
mess, I still think Rust is a language to take inspiration from. And
therefore I want to deepen my experience with Rust's paradigm. And I
want to better understand OpenBSD. Why OpenBSD? Because GNU has become
too complex for little me to handle, BusyBox is a single executable
making it harder to convert piece by piece, and I simply love the idea
of the OpenBSD project. My long term goal would be to maybe contribute
to OpenBSD, but I would first like to familiarise myself with its code
base. And what better way to do it than to rewrite it in an other
language. Hell I might discover bugs this way, that I may report
upstream (unlikely considering I barely understand what an allocator is
and does).

Also note I will probably only "maintain" (if at all) the amd64 version.
It may or may not be possible to build any part of what I'm doing on
other architectures, especially i386, but I will neither guarantee nor
even test it.

# Objectives

The main goal of this endeavour is to have functional clones of at least
some of the binaries in OpenBSD's base package. Considering that it
contains 775 executables (not counting shared libraries) as of writing
this, I can obviously not rewrite every single one of them. I will try
my best efforts to clone the most useful ones first.

Note I know the existence of [uutils coreutils](https://lib.rs/crates/coreutils)
but
1. they implement the GNU coreutils, not OpenBSD base system
2. I still want to build my own thing out of fun.

As for a road map, let's try to follow this:
- [ ] rebuild all of `/bin` except `sh`, `csh` and (`r`)`ksh`
    - [ ] [
    - [ ] cat
    - [ ] chgrp
    - [ ] chio
    - [ ] chmod
    - [ ] cksum
    - [ ] cp
    - [ ] cpio
    - [ ] csh
    - [ ] date
    - [ ] dd
    - [ ] df
    - [ ] domainname
    - [ ] echo
    - [ ] ed
    - [ ] eject
    - [ ] expr
    - [ ] hostname
    - [ ] kill
    - [ ] ln
    - [ ] ls
    - [ ] md5
    - [ ] mkdir
    - [ ] mt
    - [ ] mv
    - [ ] pax
    - [ ] ps
    - [ ] pwd
    - [ ] rm
    - [ ] rmdir
    - [ ] sha1
    - [ ] sha256
    - [ ] sha512
    - [ ] sleep
    - [ ] stty
    - [ ] sync
    - [ ] tar
    - [ ] test
- [ ] rebuild all of `/sbin`
    - [ ] atactl
    - [ ] badsect
    - [ ] bioctl
    - [ ] chown
    - [ ] clri
    - [ ] dhclient
    - [ ] dhcpleased
    - [ ] disklabel
    - [ ] dmesg
    - [ ] dump
    - [ ] dumpfs
    - [ ] fdisk
    - [ ] fsck
    - [ ] fsck_ext2fs
    - [ ] fsck_ffs
    - [ ] fsck_msdos
    - [ ] fsdb
    - [ ] fsirand
    - [ ] growfs
    - [ ] halt
    - [ ] ifconfig
    - [ ] iked
    - [ ] init
    - [ ] ipsecctl
    - [ ] isakmpd
    - [ ] kbd
    - [ ] ldattach
    - [ ] ldconfig
    - [ ] mkfifo
    - [ ] mknod
    - [ ] mount
    - [ ] mount_cd9660
    - [ ] mountd
    - [ ] mount_ext2fs
    - [ ] mount_ffs
    - [ ] mount_mfs
    - [ ] mount_msdos
    - [ ] mount_nfs
    - [ ] mount_ntfs
    - [ ] mount_tmpfs
    - [ ] mount_udf
    - [ ] mount_vnd
    - [ ] ncheck
    - [ ] ncheck_ffs
    - [ ] newfs
    - [ ] newfs_ext2fs
    - [ ] newfs_msdos
    - [ ] nfsd
    - [ ] nologin
    - [ ] pfctl
    - [ ] pflogd
    - [ ] ping
    - [ ] ping6
    - [ ] quotacheck
    - [ ] rdump
    - [ ] reboot
    - [ ] resolvd
    - [ ] restore
    - [ ] route
    - [ ] rrestore
    - [ ] savecore
    - [ ] scan_ffs
    - [ ] scsi
    - [ ] shutdown
    - [ ] slaacd
    - [ ] swapctl
    - [ ] swapon
    - [ ] sysctl
    - [ ] ttyflags
    - [ ] tunefs
    - [ ] umount
    - [ ] unwind
    - [ ] vnconfig
    - [ ] wsconsctl
This road map will be update as I progress in my endeavour. Some of
these programs are pretty large, so I may decide to remove some from
this list as the project progresses. 

# Development environment

I will developing this on (GNU/)Linux on an x86_64 machine. I will try
to test building it with OpenBSD's port of the Rust toolchain, but
considering that OpenBSD is only a Tier 3 target, I will not guarantee
that this will always be possible, and it will definitively not be able
to be my main developing environment (missing tooling etc...).

EDIT: It turns out, cross compiling for OpenBSD is a pain in the arse.
So change of plans: I will still be developing on Linux, but building
and testing on OpenBSD.

# Source code

The source code of the project will be available on [my Codeberg repository](https://codeberg.org/TheyCallMeHacked/openbsd_base_rs).

# Note on contributions

I welcome bug reports and bug corrections, but will not accept full
implementations. Considering that I do this to learn, it would
completely defeat the purpose of the project to accept implementation
submissions.

# What's next?

In the next part, I will be writing `cat`
