+++
title = "Installing without Fear"
author = ["Valentin Boettcher"]
date = 2020-09-16T15:00:00+02:00
categories = ["Tricks"]
draft = false
+++

Note to self:

If you want to make sure some nice GNU/Linux installer does not touch
certain drives just run `echo 1 > /sys/block/sdX/device/delete` in a
****root**** shell and the drive will vanish from the system.

Shamelessly stolen from:
<https://askubuntu.com/questions/554398/how-do-i-permanently-disable-hard-drives>