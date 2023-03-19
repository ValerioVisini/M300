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


# Theorie Wissensstand
## IaC
## Cloud
## Systemsicherheit


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

### Box Inhalt

Die Box enthaltet folgenden Inhalt

![image](https://user-images.githubusercontent.com/125886316/223152031-de5c3913-54f2-4d96-9002-c6732de1d3d8.png)

**Vagrantfile**

Das Vagrantfile wird für die Erstellung und Konfiguration der VM verwendet. 

Beispiel von ubuntu/xenial64:

```
# Front load the includes
include_vagrantfile = File.expand_path("../include/_Vagrantfile", __FILE__)
load include_vagrantfile if File.exist?(include_vagrantfile)

Vagrant.configure("2") do |config|
  config.vm.base_mac = "02BE826BCC1D"

  config.vm.provider "virtualbox" do |vb|
     vb.customize [ "modifyvm", :id, "--uart1", "0x3F8", "4" ]
     vb.customize [ "modifyvm", :id, "--uartmode1", "file", File.join(Dir.pwd, "ubuntu-xenial-16.04-cloudimg-console.log") ]
  end
end
```
Bei "Vagrant.configure" bis zu "end" wird die VM Konfiguriert. Man sieht welche MAC Addresse definiert wird und über welchen Provider es läuft.

**metadata.json**

Hier wird der Provider Notiert

```
{
  "provider": "virtualbox"
}
```

**.vmdk file**

Die Disk auf welcher die VM läuft.

**ovf file**

Das VM file



#LB2

##Konzept

![Zeichnung-LB2 drawio](https://user-images.githubusercontent.com/125886316/226170719-48f7535e-b70b-4712-8ce1-ffa7f8cb6d9a.png)

Es werden im Privaten V-Netzwerk zwei VMs erstellt für den Web und SQL Server. Die Firewall und der Reverse-Proxy laufen auf dem Web Server. Der Web Server Port 80 wird auf den Port 8080 forwarded und für den Web access wird der Localhost verwendet. 





