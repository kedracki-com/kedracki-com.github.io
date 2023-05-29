---
title: "Introducing CitroVM"
short: "For the last couple of months I have been working on my first own app. The effect is CitroVM, a macOS-on-macOS virtualization software for developers. Read more about why I decided to creat it and why I think you should use it."
order: 1
---

<img src="/assets/citrovm_banner.png" alt="CitroVM banner" style="width: 100%"/>

# TL;DR;

For the last couple of months I have been working on my first own app. The effect is CitroVM, a macOS-on-macOS 
virtualization software for developers. Read more about why I decided to creat it and why I think you should use it.

# But why?

I decided to create CitroVM because of my disappointment with what the major players in the VM space have to offer in 
regard to macOS guests on Apple Silicon. After almost 2.5 years since the introduction of the M1, the macOS-on-macOS 
virtualization scenario is still a second class citizen. I'm not surprised why this is. Both Parallels Desktop and 
VMWare Fusion have a strong focus on running Windows. The tag line of the former is literally "Run Windows on Mac". 
The product page of the later reads "... Windows or other operating systems on the Mac". I didn't find an exact number, 
but my guess is the Windows-on-macOS scenario represents something like 95% of all. It's only natural for these companies 
to focus on that. You should put your effort where the awards are, right?

What is too small of a reward for a big company, may be just fine for an independent developer. And so I decided to 
occupy this niche. 

# Isn't writing a Virtual Machine extremely hard? 

Yes, it is. At the core of every virtualization software lies a hypervisor, and writing one is very hard. Consider 
virtual hardware also needs to be implemented and the whole task becomes not only hard but also huge. Luckily Apple is
providing the Virtualization.framework, which already covers most aspects of setting up and running virtual machines. 
As a result, we already see a number of virtualization apps popping up on the Mac App Store and elsewhere. 

# So how is CitroVM different?

Refinement and Focus are the keywords here. CitroVM already comes with a great UI. I made an effort for it to feel 
very native, as in my opinion all pro tools for macOS should. But the plan is for it to be far more than just a thin UI
wrapper around the Virtualization.framework. Quality of life features will allow developers to get more work done in
less time. In an upcoming version of CitroVM I will introduce a powerful Automation mechanism. It will handle common 
repetitive manual tasks that are part of each VM set up, like onboarding and installing Xcode.

# The plan

The first version of CitroVM is already available on the Mac App Store [Mac App Store](https://apps.apple.com/us/app/citrovm/id1669517079).
It currently only provides a very basic set of features like setting up and running VMs. As I add more PRO features I 
intend to introduce a hybrid subscription/perpetual pricing model. With it, users will have a choice between a very 
affordable monthly subscription and a slightly larger one-time payment. In both cases, the exact same features will be 
unlocked. Basic features will remain free forever.

You can find more info on the official [CitroVM](https://citrovm.com) page. 