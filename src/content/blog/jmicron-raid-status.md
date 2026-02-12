---
title: "JM Raid Status - Mediasonic HFR2-SU3S2 Raid SMART status utility"
description: "A project to retrieve individual drive SMART data from RAID devices with JMicron JMS567 USB to SATA interface"
date: 2026-02-12
ogImage: "/external-raid.png"
---

![Mediasonic HFR2-SU3S2](/external-raid.png)

##### [JM Raid Status on GitHub](https://github.com/Majamietre/jm-raid-status)

This is something I wanted for a long time. I have a couple of these Mediasonic HFR2-SU3S2 Proraid boxes. These are 4-bay hardware RAID
boxes that are cheap and reliable. They work great, but it's traditionally been difficult or impossible to get status of the array and the individual
drives except on Windows. I want to hook one of these up to my Synology NAS via USB to extend my storage, but I'm not comfortable with letting this
thing run for months without ever laying eyes on it, and no way to monitor its health.

The maker of [Hard Disk Sentinel](https://www.hdsentinel.com/) - a fantastic piece of software for Windows - does offer a much pared Linux
utility (for free, too!), out of the box it cannot detect the individual drive status. If your spelunking skills are excellent you can find an add-on
that [will actually let you get individual drive status](https://www.hdsentinel.com/forum/viewtopic.php?p=18046&sid=8fde0ef696b0f1ecc3a500bd0f9e34b3#p18046)
using his utility, but it comes with some rather scary warnings, and it's closed source. Also, it only reports SMART status, and does not provide any other
health information about the overall state of the array. Other than this, the only information I could find anywhere online that is helpful is [JMRaidCon](https://github.com/wjoe/jmraidcon), a tool that has some basics of the protocol implemented but not much else.

With a lot of help from Claude Code, I was able to whip up this utility. It can retrieve all the SMART data for the drives, get some RAID status information,
and more importantly, it's open source so it doesn't feel like a black box. I am hoping it can be improved over time with support for other devices, or
identification of already supported devices.

Basic features are:

- Can detect complete SMART status of RAID member drives
- Can detect when array is in a rebuilding state
- Cannot directly detect a degraded state, but by providing it with an expected number of drives, will report an unhealthy state if SMART status falls below thresholds or a drive is missing
- Can output in human readable or JSON format
- Includes a tool to aggregate data from other sources (right now, `smartctl`) and provide overall system health and complete drive status in a consistent format

I'm hoping to reverse engineer more data; it seems like it should be possible to obtian the actual RAID status (e.g. healthy or degraded) in a way other than counting drives,
since the hardware itself knows this and has lights that show when a drive is missing/degraded/rebuilding. This involves intentionally braking my array; I did this
once to get the data I have so far but may need to test other configurations to better understand the data.

#### Resources

Other things related to this chipset that I found:

- [JM Raid Status on Github](https://github.com/Majamietre/jm-raid-status) - this project
- [JMRaidCon](https://github.com/wjoe/jmraidcon) - original implementation of JMS576 protocol
- [Firmware flasher when the offiical one doesn't work](https://github.com/projectgus/jms567ctl)
- [Discussion - firmware supporting SMART for JMS567](https://forum.odroid.com/viewtopic.php?t=41926) - this appears to be relevant for single drive enclosures. I did flash this to one of my boxes, and it didn't work, but it was possible to reflash it to the original FW
- [Repository of various JMS567 firmwares](https://www.usbdev.ru/files/jmicron/jms567firmware/) - these appear to be extracted from different devices. FW22.01.00.07 is recognized as valid for my Mediasonic boxes. The
  factory firmare on mine was both JMS567_FW20.05.02.03. Since they both appear to use the same versioning, I updated it on mine. Works fine, can't tell any difference. I will probably try downgrading again and see if there's any performance or other difference noted.
- [Hard Disk Sentinel dev's post about HD Sentinel linux tool for JMicron and risk](https://www.hdsentinel.com/forum/viewtopic.php?p=18046&sid=8fde0ef696b0f1ecc3a500bd0f9e34b3#p18046)
