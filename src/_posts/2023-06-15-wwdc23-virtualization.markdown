---
title: "WWDC 2023 Review: Virtualization"
short: "My very subjective review of WWDC 2023 in the area of virtualization."
---

# TL;DR;

During WWDC 2023 Apple announced macOS 14 Sonoma beta alongside an iterative update to the Virtualization.framework. 
I have already tested CitroVM, and I'm happy to report it is able to virtualize the new macOS.

# Foreword

This is the first in a series of posts under the common name "WWDC 2023 Review". Subject to my current focus on CitroVM,
this post is dedicated to virtualization.

# macOS 14 Sonoma beta

Following the tradition first introduced 2013 with macOS 10.9 Mavericks, Apple released the next major version of macOS 
during WWDC 2023. With macOS 14 Sonoma we again get a California-landmark inspired name. I really like how consistent 
Apple has been with both naming and schedule in the last few years.

After the keynote, I waited impatiently for the files to arrive in the developer portal. I really wanted to check if the 
beta will run inside CitroVM on top of macOS Ventura (spoiler alert: it does). I was quite disappointed when the first 
couple of attempts failed. It was already past 2AM in the morning (I'm based out of Poland), so I decided to go to 
bed. First thing in the morning, I check the release notes. I still ask myself why I didn't have this idea the night 
before? Nevertheless, I did find a know issues and a solution: install Xcode 15 beta or a stand-alone device support 
package on the Host. I checked the former, and it allowed me to install and test Sonoma inside CitroVM.

Fun fact: Although mentioned in the release notes, the stand-alone device support package wasn't released until around
Wednesday. Well, it just shows how hectic WWDC can get for Apple. 

<img src="/assets/2023-06-15-wwdc23-virtualization/device_support.png" alt="Screenshot from the Apple Developer portal showing Device Support package in the download section" style="width: 100%"/>

To sum up:
* macOS 14 Sonoma can be installed inside CitroVM
* Either Xcode 15 beta or a stand-alone device support package must be installed on the Host for the installation to 
succeed
* To download any of the above and the macOS 14 Sonoma beta IPSW you will need access to the Developer Portal

Fun fact: Due to review guidelines, I'm prohibited from mentioning any Apple beta software inside an app on the 
App Store. This is a funny situation, where every Apple-outlet out there is writing about stuff Apple just released, but 
mentioning Sonoma inside the release notes results in instant app rejection.

## A known issue or expected behavior?

After talking to some engineers in the virtualization lab, I learned that this is not a bug but rather the result of 
how the VM installation process works. Under the hood, the framework is using the same code that Apple Configurator uses 
to perform device-to-device restore and configuration operations. It doesn't really make any difference that 
the target device is virtual. What the stand-alone device support package contains is just additional definitions 
specific to macOS 14.

I must applaud just how elegant this solution is. Not only did it save Apple engineers weeks, if not months, of 
work, it will also be easier to maintain in the long term. Sadly, from a UX perspective, this means users will have to 
install additional software to work with the beta. And the same situation is expected to repeat itself every year.

# New in Virtualization.framework

While waiting for the installation files to arrive, I had plenty of time to read the updated documentation. It looks 
like the Virtualization.framework received a number of nice additions. 

## Automatic reconfiguration of display on resize

The first addition is a no-brainer really. VMs can now opt to have their display resolution automatically adjusted to
match the dimensions of the view. This is a feature that has long been present on desktop virtual machines. It's nice 
that it will soon also be available in the Virtualization.framework. One thing I didn't check yet, and which is not 
clear from the documentation, is if this will work for macOS guests older than Sonoma. 

You can expect support for this in CitroVM shortly after the official release of macOS 14. Although I think the default 
will be a fixed resolution.  

## Ability to save and restore the VM state into/from a file

New this year is also the ability to make a snapshot of a paused VM and save it into a file. This can be later used to 
recreate the VM in the same state. Of note here is that this snapshot only includes the contents of the RAM. Disk images 
need to be handled separately by the developer. 

This unlocks two very nice scenarios. The simpler version just makes use of snapshots to support suspending the VM while
the app is not running. As noted in this year's virtualization session, this aligns well with the convention for macOS 
apps to retain and restore their state between lunches. Taking it a bit further, if we would make a copy of the storage 
and the snapshot files, we could implement point-in-time recovery.

My plans for snapshot support inside CitroVM only included storage, and I assumed a full VM stop-backup-start loop would be
necessary to prevent data corruption. With the added ability to capture the RAM, I'm now able to use a much quicker 
pause-backup-resume loop.  

## New types of storage attachments

I think the new Network Block Device based storage attachment is probably the most exciting. If you haven't heard about 
NBD, it is a protocol first developed for Linux back in 1997 and refined over the years. The Virtualization team has 
embedded a NBD client directly into the framework, which allows disk I/O operations to be delegated to a network 
attached NBD server. 

<img src="/assets/2023-06-15-wwdc23-virtualization/nbd.png" alt="Network Block Device schematic demonstrating usage with Virtualization.framework. " style="width: 100%"/>

The first implication is the ability to tap into dedicated NAS storage solutions. In comparison to a typical 
Mac machine, NAS tend to be far more cost-effective per GB, while also supporting much higher capacities. 
They can also benefit from redundancy courtesy of RAID. I expect this to have a major impact on how macOS VMs are 
operated inside clusters, small and big. 

The second implication is related to the open nature of NBD. The protocol itself is relatively simple, and multiple 
compatible implementations exist. By tapping into the server, we get (for the first time in Apple VM land) access to 
low level block based I/O. And we can build all sorts of crazy things on top of it. Think continuous data protection, 
and layered file systems.

All that glitters is not gold. Piping everything through the TCP stack will have a significant impact on performance. 
And it will get worse as the network bandwidth drops and the latencies rise.

As much as I'm excited about the NBD feature, I still need to wrap my head around how and when to integrate it into 
CitroVM. If you have any thoughts on this, please drop my an email at [contact@citrovm.com](mailto:contact@citrovm.com)

# Final word

I think Apple followed suite from last year and provided a solid update to the Virtualization.framework in 2023. I can 
only hope this trend will continue into the future. Which reminds me: I still have a bunch of things on my wishlist, I 
need to fill Feedback tickets for.
