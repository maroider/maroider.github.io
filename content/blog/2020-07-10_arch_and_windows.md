+++
title = "Arch and Windows dual-boot woes"

[taxonomies]
tags = ["Arch Linux", "boot", "dual-boot", "GPT", "GRUB 2", "Linux", "MBR", "multi-boot", "Windows", "Windows 10"]
+++

I've been experimenting with Arch Linux lately, and have it installed on a separate drive from my Windows 10 installation. Unfortunately, I couldn't get `os-prober` to detect my Windows installation even with Windows being properly shut down and with `ntfs-3g` installed. I even tried various stanzas in the [dual-booting section](https://wiki.archlinux.org/index.php/GRUB#Dual-booting) found on the Arch Wiki's page on GRUB. It simply refused to work.

After faffing about inside GRUB's command-line I ended up doing `set root=(hd0,msdosX)` where `X` is a number between 1 and 3. The partition in question ended up being the partition that Windows boots from (not the partition labeled "C:"). `ls /` showed `bootmgr` as an entry. `bootmgr` is what BIOS/MBR installs of Windows use, which would be fine if I had installed Arch in BIOS/MBR mode. Thing is, that isn't what I had done. While I was installing Arch, I had completely forgotten how it was I had installed Windows since it was such a long time ago.

Given that I would like to still have Arch installed in UEFI/GPT mode, I decided to have a look at wether it was feasible to make my BIOS/MBR Windows installation into a UEFI/GPT installation. It turns out that there's a command for exactly that built into Windows 10 now. It's got the desriptive name: `mbr2gpt`. Being somewhat scared of borking my Windows installed, I ran `mbr2gpt /validate` to check if it could be done. This is also where I should have made a full-disk back-up, just to be sure, but most of my important data is on another drive so I didn't do that. After `mbr2gpt` told be that the conversion was possible, I ran `mbr2gpt /convert /allowFullOS` and crossed my fingers.

```
MBR2GPT: Attempting to convert disk 0
MBR2GPT: Retrieving layout of disk
MBR2GPT: Validating layout, disk sector size is: 512 bytes
MBR2GPT: Trying to shrink the system partition
MBR2GPT: Trying to shrink the OS partition
MBR2GPT: Creating the EFI system partition
MBR2GPT: Installing the new boot files
MBR2GPT: Performing the layout conversion
MBR2GPT: Migrating default boot entry
MBR2GPT: Adding recovery boot entry
MBR2GPT: Fixing drive letter mapping
MBR2GPT: Conversion completed successfully
Call WinReReapir to repair WinRE
MBR2GPT: Failed to update ReAgent.xml, please try to  manually disable and enable WinRE.
MBR2GPT: Before the new system can boot properly you need to switch the firmware to boot to UEFI mode!
```

At this point I was a bit worried. I had no idea what it was that truly was wrong, but `Failed to update ReAgent.xml` didn't exactly sound all that great. I found [this one forum thread](https://www.tenforums.com/general-support/143526-reagentc-failed.html) where someone else was getting this error message, but the instructions I found for fixing it weren't very helpful in the end. The one helpful thing I found there was a link to [another forum thread](https://www.tenforums.com/performance-maintenance/131248-cant-enable-windows-recovery-environment.html#post1618841) where I found `reagentc /info`, which I promptly ran.

```
PS C:\Windows\System32> reagentc /info
Windows Recovery Environment (Windows RE) and system reset configuration
Information:

    Windows RE status:         Disabled
    Windows RE location:
    Boot Configuration Data (BCD) identifier: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
    Recovery image location:
    Recovery image index:      0
    Custom image location:
    Custom image index:        0

REAGENTC.EXE: Operation Successful.
```

I've tried to replicate the command's output at the time I ran it for the first time, but some things may be incorrect. There's also a UUID that I've taken the liberty to write over with `x`. In the second forum thread, the person asking for help got a UUID which was all zeroes. I do not remember what my result was.

I tried running `reagentc /enable`, but that failed.

After faffing about some more, I decided that I could deal with things if I completely broke Windows' ability to boot and promptly rebooted my computer. The boot process happened completely without incident and I was able to log into my account. Running `reagentc /info` now showed something more reasonable.

```
Windows Recovery Environment (Windows RE) and system reset configuration
Information:

    Windows RE status:         Enabled
    Windows RE location:       \\?\GLOBALROOT\device\harddisk0\partition4\Recovery\WindowsRE
    Boot Configuration Data (BCD) identifier: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
    Recovery image location:
    Recovery image index:      0
    Custom image location:
    Custom image index:        0

REAGENTC.EXE: Operation Successful.
```

In the end, I was probably worried about nothing.

After that whole `mbr2gpt` debacle, I rebooted my computer and booted into my Arch install. After running `grub-mkconfig -o /boot/grub/grub.conf` again, I was overjoyed to see that `os-prober` __finally__ picked up my Windows install. To make sure everything worked as it should, I rebooted, selected the "Windows Boot Manager" entry in GRUB and watched as Windows booted successfully.

I can finally boot into Windows without mashing F8 to bring up the motherboard's boot manager.
