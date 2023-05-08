# LB3 Dokumentation

### Inhaltsverzeichnis
* [Umgebung](#Umgebung)
* [Konzept](#Konzept)
* [Testfälle](#Testfälle)
* [Reflexion&Wissenszuwachs](#Reflexion&Wissenszuwachs)

## Umgebung
Für diese Aufgabe habe ich Docker Desktop verwendet. Dieser ermöglicht mir Docker ohne eine Linux VM zu verwenden. Zusätzlich habe ich ein Grafisches Interface welches eine bessere übersicht ermöglicht.

![image](https://user-images.githubusercontent.com/125886316/236646923-5abdfc3c-733d-472c-924b-e9319d5e3d08.png)


## Konzept
Im Docker wird ein MySQL Container erstell, welcher von Prometheus überwacht wird. Damit dies Funktioniert muss noch ein zusätzlicher Container erstellt werden nämlich der MySQL Exporter. Dieser ermöglicht schlussendlicht die Überwachung und leitet die Informationen an Prometheus weiter. Prometheus kann über den Localhost accessed werden.

![image](https://user-images.githubusercontent.com/125886316/236646790-76e9a88f-98ee-4e93-968d-0795157b70d1.png)

## Setup
Zuerst den Ordner erstellen in welchem alle Files abgelegt werden. 
In meinem fall: ``` C:\Users\vvtv2\Desktop\Docker-LB3 ```

### docker-compose

Das docker-compose.yml file erstellen für die Container Konfigurationen:

```
#version von Docker Compose
version: '3' 

# Definizionen von den Services
services:
  # MySQL Container
  db:
    # image für MySQL
    image: mysql:latest
    
    # MySQL Port
    ports:
      - "3306:3306"
      
    # MYSQL user und database Informationen  
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: mydb
      MYSQL_USER: myuser
      MYSQL_PASSWORD: mypass
      
      
  # Prometheus Container
  prometheus:
    # image für Prometheus
    image: prom/prometheus:latest
    
    #Prometheus Port
    ports:
      - "9090:9090"
      
    volumes:
      # Konfigurationsdatei für Prometheus
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
  
  
  # MySQL Exporter Container
  mysqlexporter:
    image: prom/mysqld-exporter:latest
    
    # Exporter Port
    ports:
      - "9104:9104"
      
    environment:
      # Datenquelle für den MySQL Exporter
      DATA_SOURCE_NAME: root:secret@(db:3306)/mydb
      
    volumes:
      # Exporter Konfigurationsdatei
      - ./mysql-exporter.yml:/etc/mysql-exporter/config.yml
      
    command:
      #Konfiguration des Exporter
      - "--config.my-cnf=/etc/mysql-exporter/config.cnf"
      # Port freigabe
      - "--web.listen-address=0.0.0.0:9104"

```

### prometheus.yml

prometheus.yml Konfigurationen damit Prometheus weiss von welchen Quellen die Daaten gesammelt werden sollen:

```

global:
  # Aktuallisierungrate
  scrape_interval: 15s

scrape_configs:
  - job_name: prometheus
    static_configs:
      # Prometheus Container target
      - targets: ['localhost:9090']

  - job_name: mysql
    static_configs:
      # MySQL Exporter Container target
      - targets: ['mysqlexporter:9104']

```

### mysql-exporter.yml

mysql-exporter.yml Konfigurationen damit der Exporter die daten von dem MySQL Container sammeln kann:

```

---
config:
  username: root
  password: secret
  datasource: "tcp(db:3306)/mydb"
  collect.info_schema.INNODB_METRICS: "ON"
  collect.global_status: "ON"
  collect.global_variables: "ON"

```

Danach können die Container mit folgendem Command (in meinem fall im CMD im richtigen Ordner) erstellt werden:

```
docker-compose up -d
```

Damit sollten alle Container aktiv sein.


## Testfälle
Alle Tests wurden am 06.05.2023 durchgeführt.

### Test 1 Docker Compose up
#### Beschreibung
docker-compose up command durchführen und dadurch die Container erstellen.
#### Soll
command wird ohne Fehler durchgefürt und die Container sind erstellt.
#### Ist
Die Container konnten erstellt werden:

![image](https://user-images.githubusercontent.com/125886316/236648228-de065909-4da7-48b6-882e-7cd713ec13b7.png)

```docker-compose ps``` Ergebnis

![image](https://user-images.githubusercontent.com/125886316/236648260-c11dd0dc-dbc3-4efc-8be9-abd328afddf5.png)


### Test 2 MySQL Login
#### Beschreibung
Einloggen in den MySQL Container.
#### Soll
Mann soll sich mit den angegeben docker-compose login Daten einloggen können.
#### Ist
Login ist möglich:

![image](https://user-images.githubusercontent.com/125886316/236648464-8703bb5d-0f0a-4982-9b83-c42b3390e505.png)




### Test 3 Prometheus Webinterface
#### Beschreibung
Webinterface von dem Prometheus Container.
#### Soll
Zugriff auf Prometheus Webinterface über http://localhost:9090/ möglich.
#### Ist
Man kann auf das Webinterface zugreifen:

![image](https://user-images.githubusercontent.com/125886316/236648533-d5ad4ad5-876c-475a-b2d0-f650454c48a8.png)



### Test 4 Prometheus Monitoring
#### Beschreibung
Prometheus monitored MySQL.
#### Soll
Prometheus hat eine Verbindung zum MySQL Container
#### Ist
Der MySQL Container wird monitored:

![image](https://user-images.githubusercontent.com/125886316/236648613-b0b5a6a7-9170-4e63-b36f-1b471125aee8.png)

![image](https://user-images.githubusercontent.com/125886316/236817760-8848b346-2024-4ef1-9275-647bdf234553.png)


## Reflexion&Wissenszuwachs
### Neues Wissen
* Docker Desktop Umgebung kennengelernt
* Docker Commands
* Container
* Konfiguration von Containern 

### Reflexion
Anfangs wusste ich noch nicht richtig was ich genau als Projekt machen soll und habe noch nicht richtig verstanden, was ich genau mit dieser Aufgabe erreichen soll. 





