+++
title = "Removing legacy boot from a Linux live stick."
author = ["Valentin Boettcher"]
date = 2021-11-19T13:48:00+01:00
categories = ["Hacks"]
draft = false
+++

I got my ThinkPad used for a bargain price but with locked
bios/uefi-setup. Annyingly, it defaults to legacy boot and there is no
way to change that.

My previous workaround was rather involved and is documented in the
[Arch wiki](https://wiki.archlinux.org/title/Lenovo%5FThinkPad%5FT470#UEFI%5Fboot). Today however I bricked my system at work and had to
restore it in a hurry.

It turns out that you can nuke the `MBR` of the live stick to remove
the legacy boot.

****Make sure you know what the device path of the USB stick is. You
  don't want to nuke some innocent hard drive :)!****

Over at [stack exchange](https://askubuntu.com/questions/1100086/removing-extra-option-from-boot-manager-in-legacy-mode-after-deleting-ubuntu) someone had a similar problem and one proposed
solution was to overwrite the first `446` Byte of the `MBR` with
zeros.  Find the device path of the live stick with `lsblk` and then
`dd if=/dev/zero of=/dev/sdx bs=446 count=1` as root and you're set.