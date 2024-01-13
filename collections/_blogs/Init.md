---
layout: blog
title: Init
author: Xabier Gabiña Barañano
date: 2023-12-1
---

## Introduccion

## Inits

### SysV

### systemd

### Runit

### OpenRC

### Upstart

### launchd

### Otros

* minit
* nosh
* s6
* BusyBox init
* Perp
* Epoch
* Serinit
* Initng
* Finit

## Cheatseat de comandos

| Accion | SysV | systemd | Runit | OpenRC |
|-|-|-|-|-|
| Start | service \<nombre> start | systemctl start \<nombre> | sv start \<nombre> | rc-service \<nombre> start |
| Stop | service \<nombre> stop | systemctl stop \<nombre> | sv stop \<nombre> | rc-service \<nombre> stop |
| Restart | service \<nombre> restart | systemctl restart \<nombre> | sv restart \<nombre> | rc-service \<nombre> restart |
| Reload | service \<nombre> reload | systemctl reload \<nombre> | sv reload \<nombre> | rc-service \<nombre> reload |
| Status | service \<nombre> status | systemctl status \<nombre> | sv status \<nombre> | rc-service \<nombre> status |
| Enable | chkconfig \<nombre> on | systemctl enable \<nombre> | ln -s /etc/sv/\<nombre> /var/service/\<nombre> | rc-update add \<nombre> |
| Disable | chkconfig \<nombre> off | systemctl disable \<nombre> | rm /var/service/\<nombre> | rc-update del \<nombre> |
| List | chkconfig --list | systemctl list-unit-files | ls /etc/service | rc-update show |
