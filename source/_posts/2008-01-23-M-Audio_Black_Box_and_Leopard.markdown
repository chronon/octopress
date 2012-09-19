---
layout: post
title: M-Audio Black Box and Leopard
date: 2008-01-23
categories: [os-x, audio] 
---

Update Jan. 27: Two days after I wrote this, M-Audio released a 10.5.1 driver. Ha!

Possibly M-Audio is discontinuing or replacing their Black Box "guitar performance recording system" as it is no longer findable on their website. No Mac 10.5 Leopard drivers are available, and my guess is they will never be. Just for fun, I installed the 10.4.10 Tiger drivers, rebooted, and found upon (every) reboot BootstrapDaemonPerUser crashed. I ran the driver uninstall app (included with the driver) only to find BootstrapDaemonPerUser was still crashing upon reboot, and the driver was not actually uninstalled.

Manually uninstalling the driver was easy enough, looking at the uninstall.ss shell script inside the uninstallation app:

``` sh
$ rm -rf /System/Library/Extensions/M-AudioBlackBoxBoot.kext
$ rm -rf /System/Library/Extensions/M-AudioBlackBoxJag.kext
$ rm -rf /Library/StartupItems/com.m-audio.startupitem.blackbox
$ rm -f /etc/mach_init_per_user.d/com.m-audio.blackbox.helper.plist
$ rm -rf /Library/Audio/MIDI Devices/M-Audio
$ rm -rf /Library/PreferencePanes/M-AudioBlackBox.prefPane
$ rm -rf /Library/Receipts/M-Audio\ Black\ Box.pkg
$ rm -f ~/Library/Preferences/com.m-audio.prefpane.blackbox.plist
```

Upon rebooting, all traces of the driver were gone. Guess it was not the best idea to install it.

The Black Box seems to work fine with no driver installed, though Logic 8 only sees two inputs (there should be four).
