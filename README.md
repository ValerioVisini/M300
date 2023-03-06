# M300 Dokumentation
### Inhaltsverzeichnis

* Theorie Wissensstand
  * IaC
  * Cloud
  * Systemsicherheit 
* Virtual Box
* Vagrant
  * Vagrant VM erstellen
  * Vagrant Boxen


# 

# VirtualBox
VirtualBox ist ein Hypervisor Typ 2 von Oracle. 

Mit diesem Hypervisor können Virtuelle Maschienen und Netzwerke erstellt werden.

![image](https://user-images.githubusercontent.com/125886316/223142724-e002532b-e87f-465a-a683-9348230b6580.png)

# Vagrant
Vagrant ist eine IaC lösung mit der man VMs mit Vorlagen Automatisiert erstellen kann ohne, dass man ein GUI braucht.
Vagrant ist mit VirtualBox kompatibel weshalb wir darum auch mit VirtualBox arbeiten.

## Vagrant VM erstellen
Nachdem ich VirtualBox und Vagrant installiert habe konnte ich eine Test VM installieren.

Vagrant kann nun genutz werden um VMs automatisiert zu erstellen
Folgende Commands habe ich für die erstellung eingegeben
```
vagrant init ubuntu/xenial64
vagrant up --provider virtualbox
```
Der VM erstellungsprozess wird gestartet und kann schlussendlich in VirtualBox gesehen werden.

![image](https://user-images.githubusercontent.com/125886316/223126175-f47f51b7-c675-4b6f-afc2-386e0dca97f0.png)
![image](https://user-images.githubusercontent.com/125886316/223127055-7e4ac42f-1a55-46fd-974a-ab8af3143aa9.png)

Die VM kann man mit ```vagrant destroy -f``` gelösch werden 


## Vagrant Boxen
Vagrant Boxen sind Vorlagen für das erstellen von VMs.
Vagrant hat bereits Vorlagen wie die "ubuntu/xenial64" Box welche ich z.B. für das installieren von einer Ubuntu VM verwendet habe


