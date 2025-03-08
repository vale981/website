+++
title = "Fixing Linux Dualboot: Reinstalling the Windows EFI Bootloader Files"
author = ["Valentin Boettcher"]
date = 2020-07-11T15:00:00-04:00
categories = ["Tricks"]
draft = false
+++

Note to my future self :).

Reloading my Linux install after a pretty radical 'nuke and pave' I
had to get my Windows dualboot back to work.  There are a thousand
guides on how to do that, but I'll add another one in case your setup
is similar to mine.

I have installed windows on a separate drive and Linux on my main
drive, along with the efi partition.

Don't follow this guide blindly. Think about every step you take,
because you can seriously mess up your system :).

With that out of the way, the things you have to do are:

1.  Boot a windows install medium.
2.  Choose your language and enter the 'repair options'.
3.  Go to advanced and select 'command line'.
4.  To mount the efi partition type diskpart and in diskpart then type
    list volume. A list of volumes will be printed and one of them the
    efi partition (usually around 500mb ). Select this partition
    (select volume `[number]`) and assign a drive letter (`X` is the
    drive letter you assign).
5.  Check where your windows partition is mounted. The diskpart list
    volume output will probably include it. I will assume that it is
    volume `C`. Exit diskart with `exit`.
6.  To finally install the boot files type the command `bcdboot
           c:\windows /s x:`. This will generate boot files based on
    `c:\windows` and install them on the partition with the letter
    `X`.

Thats it, you can reboot now.  You may have to reconfigure grub (or
whatever loader you use). On arch-linux, make sure you have os-prober
installed :).
