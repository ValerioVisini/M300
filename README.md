# M300 Dokumentation Valerio Visini

### Inhaltsverzeichnis

* Virtual Box
* Vagrant
  * Vagrant VM erstellen
  * Vagrant Boxen
* LB2
  * Konzept MySQL und Web Server
  * Vagrant File 
  * Testfälle und Ergebnis
  * Reflexion und Wissenszuwachs


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


**.vmdk file**

Die Disk auf welcher die VM läuft.

**ovf file**

Das VM file



# LB2

## Konzept MySQL und Web Server

![Zeichnung-LB2 drawio](https://user-images.githubusercontent.com/125886316/226170719-48f7535e-b70b-4712-8ce1-ffa7f8cb6d9a.png)

Es werden im Privaten V-Netzwerk zwei VMs erstellt für den Web und SQL Server. Die Firewall und der Reverse-Proxy laufen auf dem Web Server. Der Web Server Port 80 wird auf den Port 8080 forwarded und für den Web access wird der Localhost verwendet. 

## Vagrant File
Das Vagrant File mit allen Konfigurationen für den Web Server (mit Kommentaren):

```

 VAGRANTFILE_API_VERSION = "2"

 #Alle Vagrant Konfiguratiionen werden hier gemacht
 Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

   #Every Vagrant virtual environment requires a box to build off of.
   #Erstellung des SQL Servers 
   config.vm.define "database" do |db|
  #Ubuntu Box wird verwendet
     db.vm.box = "ubuntu/xenial64"
  #VM wird in Virtualbox erstellt
  db.vm.provider "virtualbox" do |vb|
  #Arbeitspeicher grosse	
    vb.memory = "512"  
  end
  #Definierter Hostname
     db.vm.hostname = "db01"
  #Netzwerk Konfigurationen Netzwerk und IP
     db.vm.network "private_network", ip: "192.168.10.20"
  #MySQL Konfigurationen in einem separaten File
    db.vm.provision "shell", path: "mysql.sh"
   end

   #Erstellung des Web Servers
   config.vm.define "web" do |web|
     web.vm.box = "ubuntu/xenial64"
     web.vm.hostname = "web01"
     web.vm.network "private_network", ip:"192.168.10.10"
  #Port forwarding von 80 auf 8080 
  web.vm.network "forwarded_port", guest:80, host:8080, auto_correct: true
  #web.vm.network "forwarded_port", guest: 443, host: 8443
  web.vm.provider "virtualbox" do |vb|
    vb.memory = "512"  
  end

  #Web Server Konfigurationen

web.vm.synced_folder ".", "/var/www/html"
  #shell des Servers wird geoffnet  
  web.vm.provision "shell", inline: <<-SHELL
      # Debug ON!!!
         set -o xtrace	
   sudo apt-get update
   #apache Web Server wird installiert
   sudo apt-get -y install debconf-utils apache2 nmap
   #root Passwort wird gesetzt
   sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password password admin'
   sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password admin'
   #PHP Installation
   sudo apt-get -y install php libapache2-mod-php php-curl php-cli php-mysql php-gd mysql-client  
   #Admininer Installation
   sudo mkdir /usr/share/adminer
   sudo wget "http://www.adminer.org/latest.php" -O /usr/share/adminer/latest.php
   sudo ln -s /usr/share/adminer/latest.php /usr/share/adminer/adminer.php
   echo "Alias /adminer.php /usr/share/adminer/adminer.php" | sudo tee /etc/apache2/conf-available/adminer.conf
   sudo a2enconf adminer.conf 

   #Firewall installieren und die Ports 22, 80 freigeben.
   sudo ufw allow 22
   sudo ufw allow 80
   #Firewall aktivieren
   yes | sudo ufw enable

   #Web Server Restart
   sudo systemctl restart apache2.service

   #Installation Reverse Proxy
   apt-get -y install libapache2-mod-proxy-html libxml2-dev
   a2enmod proxy
   a2enmod proxy_http
   a2enmod proxy_html
   a2enmod xml2enc
   service apache2 restart

   SHELL
    end  
    end

```

Zusätzliches .sh File für die MySQL Konfigurationen:

```

#!/bin/bash

#MySQL Datenbank installieren und Konfigurieren


#Debug ON
set -o xtrace
        
#root Password setzen, damit kein Dialog erscheint und die Installation haengt!
sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password password S3cr3tp4ssw0rd'
sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password S3cr3tp4ssw0rd'

#MySQL Installation
sudo apt-get install -y mysql-server

#MySQL Port oeffnen
sudo sed -i -e"s/^bind-address\s*=\s*127.0.0.1/bind-address = 0.0.0.0/" /etc/mysql/mysql.conf.d/mysqld.cnf

#User fuer Remote Zugriff einrichten - aber nur fuer Host web 192.168.10.10
mysql -uroot -pS3cr3tp4ssw0rd <<%EOF%
	CREATE USER 'root'@'192.168.10.10' IDENTIFIED BY 'admin';
	GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.10.10';
	FLUSH PRIVILEGES;
%EOF%

#MySQL Restart fuer Konfigurationsaenderung
sudo service mysql restart

#Test ob MySQL Server laueft
curl -f http://localhost:3306 >/dev/null 2>&1 && { echo "MySQL up"; } || { echo "Error: MySQL down"; exit 1; }


```

### Start

Erstellung der Umgebung mit "vagrant up". In dem Pfad "C:\Users\valer\VirtualBox VMs" ist das Vagrant File abgelegt.

![image](https://user-images.githubusercontent.com/125886316/226337395-7c83411b-03a7-416a-a5dc-5b90e6e12e61.png)

## Testfälle und Ergebnis

### Test 1 Vagrant up
#### Beschreibung
Vagrant up durchführen mit dem erstellten Vagrant File, damit die Umgebung erstellt wird.
#### Soll
Vagrant up ohne error.Zwei VMs installiert (Database, Web).
#### Ist
Vagrant up wurde erfolgreich beendet und die VMs wurden installiert.

![image](https://user-images.githubusercontent.com/125886316/226338564-3940c597-4d11-493f-85ee-39f79af77414.png)

![image](https://user-images.githubusercontent.com/125886316/226344998-1cfe5f35-eb5f-4d65-9516-b24f711c7e48.png)



### Test 2 Webseite
#### Beschreibung
Die Webseite welche mit dem Web Server erstellt wurde.
#### Soll
Die Standart Apache2 Seite soll über den "Localhost:8080" angezeigt werden.
#### Ist
Die Webseite wird über den Localhost mit Port 8080 angezeigt.

![image](https://user-images.githubusercontent.com/125886316/226346942-c82b3b93-6016-48cf-8add-431b5768de66.png)


### Test 3 MySQL
#### Beschreibung
Mit Adminer Zugriff auf die MySQL Datenbank zugreifen.
#### Soll
Die Adminer Seite ist über "http://localhost:8080/adminer.php" erreichbar. Mit dem Root User und dem Passwort "admin" kann man sich auf den MySQL Server einloggen.
#### Ist
Die Adminer Seite ist erreichbar. Das einloggen funktioniert.

![image](https://user-images.githubusercontent.com/125886316/226349238-8f3ab334-f6e8-42c5-b633-5b0add399936.png)

![image](https://user-images.githubusercontent.com/125886316/226349461-d8ab7cd2-2cc0-4551-91f0-8b856e96e2bc.png)


### Test 4 SSH
#### Beschreibung
SSH Verbindung auf die VMs.
#### Soll
Mann kann sich über SSH auf die Beiden Server (Database und Web) verbinden.
#### Ist
Eine SSH verbindung ist auf beide VMs möglich.

![image](https://user-images.githubusercontent.com/125886316/226349918-c6b34229-a95f-4f83-a60e-0e6a868e445c.png)

![image](https://user-images.githubusercontent.com/125886316/226350490-65005200-843a-4e2a-945d-1b2c1764bc26.png)

### Test 5 Firewall
#### Beschreibung
Installierte Firewall für den Port zugriff.
#### Soll
ufw Firewall ist auf der Web Server VM installiert und aktiv. Die Ports 22 und 80 sind geöffnet. 
#### Ist
Firewall ist aktiv und hat die Ports 22 und 80 geöffnet.

![image](https://user-images.githubusercontent.com/125886316/226353360-9c8891d2-e183-4d4c-96fa-172f96e6ce9a.png)


### Test 6 Reverse-Proxy
#### Beschreibung
Reverse-Proxy zwischen dem Web Server und den Webseiten Benutzern.
#### Soll
Reverse-Proxy ist installiert und verbessert die sicherheit des Web Servers.
#### Ist
Der Reverse-Proxy ist installiert, aber es wurden keine Konfigurationen für die verbesserung der Sicherheit vorgenommen.

## Reflexion und Wissenszuwachs

Anfangs hatte ich etwas Schwierigkeiten, weil ich Vagrant nicht ganz verstanden habe und mir war nicht ganz klar, was ich bei der LB2 genau machen muss. Nachdem ich nachgefragt habe konnte, ich es besser verstehen, was genau das Ziel ist und ich konnte anfangen verschiedene Dinge mit Vagrant auszuprobieren. Die Umsetzung dieser Umgebung hat an sich gut funktioniert, jedoch konnte ich alle Ideen umsetzen. Bei dem Proxy habe ich nicht genau herausgefunden, wie ich ihn genau in der Umgebung umsetzen kann, darum habe ich ihn nur installiert. Weitere Ideen wie HTTPS und SSH Tunnel habe ich ausprobiert, aber ich konnte es nicht funktionstüchtig implementieren und habe mich entschieden es nicht zu machen. Grundsätzlich habe ich gelernt, wie Vagrant funktioniert und wie man Umgebungen mit Code erstellen kann. Viele der Theorieteile wie Cloud oder Sicherheitsaspekte habe ich schon gekannt, das Praktische umsetzen dieser Themen konnte ich davor noch nicht. Bei der Dokumentation bin ich mir nicht ganz sicher, ob sie den Anforderungen entspricht, aber ich denke, dass ich alles Wichtige notiert habe.
