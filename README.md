# M300 Dokumentation
### Inhaltsverzeichnis

* Virtual Box
* Vagrant
** Vagrant VM erstellen
** Vagrant Boxen


# Vagrant VM erstellen
Nachdem ich VirtualBox und Vagrant installiert habe konnte ich eine Test VM installieren.

Vagrant kann nun genutz werden um VMs automatisiert zu erstellen ohne GUI
Folgende Commands habe ich für die erstellung eingegeben
```
vagrant init ubuntu/xenial64
vagrant up --provider virtualbox
```
Der VM erstellungsprozess wird gestartet und kann schlussendlich in VirtualBox gesehen werden.

![image](https://user-images.githubusercontent.com/125886316/223126175-f47f51b7-c675-4b6f-afc2-386e0dca97f0.png)
![image](https://user-images.githubusercontent.com/125886316/223127055-7e4ac42f-1a55-46fd-974a-ab8af3143aa9.png)

Die VM kann man mit ```vagrant destroy -f``` gelösch werden 


# Vagrant Boxen
Vagrant Boxen sind Vorlagen für das erstellen von VMs.
Vagrant hat bereits Vorlagen wie die "ubuntu/xenial64" Box welche ich z.B. für das installieren von einer Ubuntu VM verwendet habe


