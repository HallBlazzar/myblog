+++
title = "How Did I Choose OS For My Laptop?"
date = 2026-04-19
draft = false
categories = ["Debian", "Linux"]
+++

{{<notice tip>}}
TL;DR: If you're a developer who is struggling on selecting Linux based desktop environment for your laptop/workstation, Debian can be the most reliable solution for you. Otherwise, consider pickup and test from [DistroWatch](https://distrowatch.com/dwres.php?resource=popularity) to find most suitable one for you.
{{</notice>}}

It's a story about how I chose OS for my new laptop, which is also a 2 + 3 months journey.

Last November (2025), I spent about $1,200 on a Dell Pro 14 Laptop (64 GB RAM, AMD Ryzen 5 220) with Black Friday deal to replace my Surface Laptop 3 (32 GB RAM, i7-1065G7). The Surface Laptop 3 was pretty good for developers, but Windows 11 and recent desktop applications (plus WSL2) seems too heavy to it. It just ran slower from time to time even after re-install the whole system. As a result, by the end of last year, I decided to replace it.

The first question is, which OS should I use?

Actually, the first thought came to my mind was choosing options other than Windows. I've been using Windows for more than 2 decates. I still think it has the best user experience. Users don't need to spend too much time to understand how to use it. All controls are straightforward. Also, most modern applications I used in my daily basis were Windows based, e.g., Notion Desktop, [Groupy](https://www.stardock.com/products/groupy/) and Notepad++. However, I still decided to leave it.

The most significant reason was the system resource consumption. On the Surface Laptop 3, Windows took about 8 GB RAM to run great amount background processes which purposes were unknow to me. The situation got worse when I run applications. In my daily work, I used to run FireFox with more than 100 tabs, multiple Jetbrains-based IDE instances and WSL at the same time. Memory and CPU consumption couldn't be told from the Task Manager would be significantly increase under the situation. For instance, when I saw system CPU and memory usage had reached about almost 90%, the summarized number of system resource consumption in the Task Manager's process view were mismatched. Even if I tried close all applications and shutdown WSL, I could still see system resource consumption was higher than freshly booted.

From what I remembered, since Windows 10, there were great amount background processes added to the Windows itself for different purposes like security and system resource optimization. But since AI related features and pre-installed desktop utilities introduced to Windows 11, telling source of resource consumptions getting more difficult. There already have great amount of topics on the internet to discuss how to optimize Windows to decrease default consumption, including modifying the Windows Registry and install 3rd-party tools (which could be overridden by random Windows Update anytime). However, my most basic expectation to an operating system to run things **I need**. Any additional features from OS itself are bonuses but shouldn't be blockers, especially lacking of simple and officially supported approaches to directly turn off/remove them.

I loved Windows, but it was time for me to move forward.

Except Windows, Linux seems to be the most available options on the market. Platforms most softwares and hardware supports are still Windows, MacOS and Linux. Though there are still some open source OS like [ReactOS](https://reactos.org/), I'm still doubt they're ready for production usage. But in my personal experience, Linux based desktop environments are still not stable as Windows or MacOS. For instance, Ubuntu usually start popping error messages after using for a while (few days or few weeks), like some unknow upgrade failure or broken system components; Fedora receives updated frequently and easily cause some system configurations or software broken (from system library or kernel). However, as I already decided to pick this solution, all I could do was try my best to test different Linux distributions and desktop environments most suitable for my daily work.

My requirements were pretty simple:
- Least configuration. I expected an OS itself could run out-of-box instead of great amount of special configurations.
- No too much surprises. An OS shouldn't have some restrictions recorded in GitHub issues or some topics written by someone on Reddit or StackOverflow which could be blocker to software I run.

Based on these principals, my research was split into 2 areas, desktop environment and distribution.

# Desktop Environments

There are plenty of modern desktop environments for Linux. For example, [GNOME](https://www.gnome.org/), [KDE](https://kde.org/), [Cinnamon](https://projects.linuxmint.com/cinnamon/) and [XFCE](https://www.xfce.org/). But as I use Jetbrains based IDEs for my daily basis, GNOME and KDE became most viable option to me.

## GNOME

GNOME provides pretty smooth user experience to me. I didn't need to make too much customization to make it works like a charm. Also, GNOME owns plenty of useful extensions to help me add some extra features. For instance, showing system information on system tray ( [Vital](https://extensions.gnome.org/extension/1460/vitals/) ) and showing Caps Lock and Num Lock status ( [Lock Keys](https://extensions.gnome.org/extension/36/lock-keys/) ). 

However, I think the disadvantage also come from the design. First of all, customizable part of GNOME (from its settings) are restricted. Mostly users need to rely on extensions ([JS](https://gjs.guide/extensions/development/creating.html) based) to achieve some changes. But as extensions were mostly individual open source project, it's easy to encounter the situation like out-dated extensions which don't support latest GNOME or conflicts between extensions. Therefore, my recommendation there is keeping least extensions as possible, and making sure each of them are as simple as possible. Accepting workflow it provides is easier than making it run as you want.

## KDE

Compare with GNOME, KDE provides more customization options like number of system trays and system tray appearance. Users could do almost whatever changes to the appearance. Also, extensions of KDE is as rich as GNOME.

However, I didn't chose KDE because of the [SDDM](https://github.com/sddm/sddm). The display manager made the user experience separated. My experience was KDE was modern and smooth, but SDDM was primitive. SDDM themes and user experience provided by KDE were not compatible with KDE itself. Also, the most weired part was multiple display for SDDM. When login to a KDE environment, login window would be displayed on all monitors, and only one of them worked. There were some topic on the internet discussed possible solutions, but I didn't want to spend time on it. 

Currently, KDE provides an alternative to the SDDM, [Plasma Login Manager](https://github.com/KDE/plasma-login-manager), which seems solves these issues. But as I've already chosen GNOME before the Plasma Login Manager released, also it haven't been included in Debian, I'll still stick on GNOME and might consider KDE in the future.

## XFCE / LXDE

Simple and lightweight. But as KDE and GNOME don't cause impacts to system loading on my new laptop, I think I don't need to consider them.

## Cinnamon

Beautiful and smooth. Also provides Windows-liked user experience and UI. But extensions support is not as rich as GNOME and KDE. Additionally, if users want to get the best user experience, choices of distributions were mostly Ubuntu-based (e.g., [Mint](https://linuxmint.com/) and [Ubuntu Cinnamon](https://ubuntucinnamon.org/)), which was not I prefer.

## COSMIC

Similar to GNOME, but still buggy and at early stage.

# Distributions

I tested variety of distributions high-ranked on [DistroWatch](https://distrowatch.com/dwres.php?resource=popularity), and my understanding was, rank doesn't mean an OS is suitable for all scenarios. It's more likely to show which distributions get more attention and discussions. People still need to install and deploy their workloads to know if a distribution can satisfy their needs.

## Debian

To be honest, I originally didn't take Debian into account. Compare with other distributions, Debian is *boring*. It doesn't provide latest packages or built-in 3rd-party repositories like Arch [AUR](https://aur.archlinux.org/) or rolling update model, nor latest or customized kernel. However, users can run their software without surprise. Packages and libraries included for a version are stable and always work. After trying different distributions, I found it is the one I really need. Things run out-of-box, no special configurations I need to modify to avoid breaking things, system won't be broken by random update, and the most important of all, compatible with all software I need.

## CachyOS

I didn't hear it before until this research. I can imaging why it's popular after using it. It contains well-preconfigured user interface and almost no extra extensions needed. Also, it contains optimized Linux kernel which seems speed up system on modern hardware. Originally, it was my first tried option and the one I seriously consider to use. However, the Android Emulator in Android Studio cannot boot after few updates within a week, which looks AMD driver related issue but cannot be simply fixed.

## Fedora / EndeavourOS / PopOS / ArchLinux

Android Emulator in Android Studio suffered same issue as CachyOS on these distributions (even for fresh installed system).

## MX Linux

No GNOME based option.

## ZorinOS / Ubuntu / Ubuntu Cinnamon 

Kept receiving random error popped after few updates. Also, Ubuntu based distributions used different sources to install software, including Ubuntu APT, PPA and Snap. It made package installation ambiguous as each source seemed maintained by different owners and update states were not synced. Otherwise, user experience of them were still good enough to be my backup plan (if I couldn't find other suitable distributions).

## Linux Mint

Using its own package repository but no GNOME support.

## Bazzite And Similar Immutable Distributions

Failed Android Emulator in Android Studio. Also, running software in containers/[DistroBox](https://distrobox.it/) seemed clean but caused extra efforts on setting up, especially some desktop applications don't work well in DistroBox.

# Summary

The whole research took me about 2 month to verify compatibility and stability between software and desktop environment. Though I selected Debian in the end, I still don't think it's perfect. The situation seems applied to most of Linux desktop distributions as development of them were communities driven. Sometimes making a system stable or easy to use was just not the main goal to a community, not to mention those with commercial support (e.g., Microsoft, Ubuntu, Apple and SUSE). It seems what users can do nowadays is find one **acceptable** but **not perfect** for their daily basis. Expecting an once and for all solution looks difficult today.

By the way, if you're interested in what happened to another 3 months, you can check the [lbhelper](/posts/2026_apr_12_lbhelper/).

# Appendix

If you're interested in basic hardware spec of Dell Pro 14 Laptop and Surface Laptop 3:

|| Dell Pro 14                                                                                                  | Surface Laptop 3                          |
| --- |--------------------------------------------------------------------------------------------------------------|-------------------------------------------|
| CPU | AMD Ryzen 5 220, 6 cores 12 threads, 3.2-4.9 GHz                                                             | i7-1065G7, 4 cores 8 threads, 1.3-3.9 GHz |
| Memory | 64 GB, Crucial DDR5, 5600 MHz, 32 GB x 2                                                                     | 32 GB                                     |
| Storage | 1 TB, WD Black, SN770M                                                                                       | 1 TB                                      |
| Warranty | 3 years [Pro Support](https://www.dell.com/support/contents/en-ie/article/warranty/prosupport-suite-for-pcs) | 1 year for impersonal damage              |
| Price | About $1200 (Thanks to Black Friday and Amazon)                                                              | About $3000                               |

Another thought for me now is avoiding ultrabooks. CPU nowadays upgrade so fast, it took about 5 years to make a non-top-tiered CPU beat a top-tier one with almost 1/3 price (also RAM and CPU). Previous, I expect a device can be powerful enough to last more than 5 years. However, my expectation to hardwares now is cheap and fast enough to cover my daily work, which I can replace anytime without heavy financial burden.

By the way, if you're considering buying new laptop, Dell can be a great choice as it provides supports/on-site supports for almost all countries compare with other brands. For example, Dell is the only one provides on-site support in Ireland. Other brands require users to send their devices to their EU headquarters by themselves.
