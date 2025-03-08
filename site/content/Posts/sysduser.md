+++
title = "How to use the Systemd userspace DBus API on Traivis-CI"
author = ["Valentin Boettcher"]
date = 2020-07-11T14:00:00-04:00
tags = ["DBUS", "CI"]
categories = ["Tricks"]
draft = false
+++

I am currently working on a project which involves talking to the
`systemd` userspace session via the session `dbus` instance.

After some fiddling around and enabling debug mode on travis via the
excellent user support, I came up with the following.

Travis uses VMs that run `ubuntu` which comes with `systemd`.  To
enable the userspace `dbus` session, one has to install the
`dbus-user-session` package. After the installation, it has to be
activated through `systemctl --user start dbus`. Furthermore one has
to set the `DBUS_SESSION_BUS_ADDRESS` environment variable through
`export DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/$(id -u)/bus`.

TL;DR

```yaml
script:
  - sudo apt update
  - sudo apt install dbus-user-session
  - systemctl --user start dbus
  - export DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/$(id -u)/bus
```
