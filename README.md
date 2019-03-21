# Message Queuing

Download MQTT.fx: https://mqttfx.jensd.de/index.php/download

## Ausarbeitung des Referats

# MQTT (Message Queue Telemetry Transport)

MQTT ist ein leichtgewichtiges publish/subscribe - messaging Protokoll, welches vernetzten Geräten eine Möglichkeit bietet, möglichst bandbreiten- und batterieschonend zu kommunizieren. Um dies zu erreichen, baut MQTT auf einem Konzept auf, welches einerseits die Bandbreite minimiert und zusätzlich eine garantierte Zustellung der Daten ermöglicht. Daher eignet sich das Protokoll gut für IoT (Internet of Things) und M2M (machine-to-machine) Kommunikation im Bereich der vernetzten Geräte. 

### Publish-Subscribe-Pattern

Das MQTT-Protokoll basiert auf einer Publish-Subscribe- Kommunikation, d.h die Clients kennen sich untereinander nicht und für die Kommunikation wird ein zentraler Verteiler, ein sog. Broker verwendet.

- Broker

  Der Broker stellt die zentrale Vermittlungsplattform des MQTT Nachrichtenaustauschs dar indem er den kompletten Datenverkehr verwaltet - d.h ohne Broker könnten die Clients nicht untereinander kommunizieren. Zu den Aufgaben des Brokers zählen die Speicherung, Verwaltung und Verteilung aller Informationen zu sog. Topics. 

- Clients

  - Publisher (z.B Temperatursensor)
    - veröffentlichen ("publishen") Nachrichten
  - Subscriber (z.B ESP32 Mikrocontroller)
    - abbonieren ("subscriben") Nachrichten

## Topics

Topics sind frei wählbare Zeichenketten, welche mit '/' aneinandergereiht werden können und dienens als Grundlage für das publishen und subscriben von Informationen. 
Ein Topic ist vereinfacht gesagt mit einem schwarzen Brett vergleichbar, wobei sich alle anmelden müssen, die Informationen auf dem Brett veröffentlichen wollen, ebenso diejenigen, die Nachrichten lesen wollen.

Beispiel:

Ein Temperatur- und ein Luftfeuchtigkeitssensor publishen ihre Werte an die topics "home/temerature" bzw. "home/humidity". Dabei ist "home" das root-topic und "temperature" bzw. "humidity" sind untergeordnete topics. Daraufhin erhalten alle Clients, welche diese topic subscribed haben, diese Werte durch den Broker übermittelt. Dabei wäre es auch möglich, alle untergeordneten topics des root topics ("home") auf einmal zu subscriben - und zwar mittels der Wildcard "home/#" (Mechanismus, mit dem sich das Ordnersystem der Topics gruppieren lässt).

### Kommunikation Broker-Clients

Beim Verbindungsaufbau vom Client zum Broker (=Subscribe), teilt dieser dem Broker mit, welche Topics er abbonieren möchte - d.h für welchen Topics er Benachrichtigungen erhalten möchte.
Ähnliches gilt für Clients, die Nachrichten versenden möchten (=Publish). Dabei muss beim Senden an den Broker die Information enthalten sein, welche Topics diese Nachricht erhalten sollen. 

Durch diese Lösung ist es möglich, tausende Clients über den Broker zu vernetzen, ohne Ahängigkeiten untereinander zu haben. Der große Vorteil dieses Protokolls im Vergleich zu HTTP ist, dass ein Client automatisch vom Broker benachrichtigt wird, wenn neue Informationen vorhanden sind, und diese nicht extra mittels GET-Requests abfragen muss. Ein tolles Feature des Protokolls ist, dass Nachrichten, die nicht zugestellt werden können (z.B wenn der Broker oder ein Client nicht erreichbar ist), persistiert werden und bei erneuter Verbindung sofort zugestellt werden.

![mqtt](/images/mqtt.png)



### Mosquitto (http://mosquitto.org)

Mosquitto ist eine kostenlose und leichtgewichtige open-source MQTT-Broker-Lösung. Sie ist für einige Plattformen verfügbar und findet nicht ihre Anwendung von Geräten mit wenig "Power" bis zu High End Server.

Mosquitto in Docker starten: docker run -ti -p 1883:1883 -p 9001:9001 toke/mosquitto

### Abgrenzung zu OpenHAB (https://www.openhab.org/docs/)

OpenHAB (**open H**ome  **A**utomation  **B**us) ist eine technologieunabhängige Home Automation Plattform, welche das Zentrum eines Smarthomes darstellt. Sie stellt benutzerdefinierte Tools zur Verfügung um mit den Geräten des Users zu interagieren und kann mit über 200 verschiedenen Technologien und Systemen kommunizieren.

Stärken: 

- Integration anderer Geräte und Systeme und Vereinigung in **EINE** Lösung
- einheitliches User Interface und Regeln für alle angeknüpften Systeme
- Flexibilität für viele verschieden Home Automation Konstellationen

# AMQP(Advanced Message Queuing Protocol)

AMQP ist ein messaging protocol, welches Kommunikation von Applikationen mit messaging Brokern ermöglicht. Dabei werden Nachrichten an sog. Exchanges gesendet und dann regelbasiert in Queues an die Clients verteilt. Die Umsetzung des Exchanges und der Queues erfolgt innerhalb eines Brokers. Damit Nachrichten den richtigen Empfänger erreichen, verwendet man **Routing Keys** (vergleichbar mit Adresse des Empfängers), wobei der Versender jeweils einen Key bei der Nachricht angibt. Der Exchange erkennt dann anhand des Schlüssels, wohin die Nachricht weitergeleitet werden muss.

Die Nachrichtenübermittlung an den Empfänger kann entweder durch sofortige Zustellung in einer Queue oder durch eine explizite Abholung erfolgen. Ein toller Vorteil dieser Queue ist, dass sie die Nachrichten so lange behält, bis sie erfolgreich beim Empfänger angekommen ist und der Sender wird ebenfalls darüber in Kenntniss gesetzt. - d.h sollte es zu Verbindungsabbrüchen kommen, bekommt der Sender eine Nachricht und es kann davon ausgegangen werden, dass der Empfänger die Nachrichten trotzdem erhalten wird.

Zwischen Exchange und Queue besteht ein sogenanntes **Binding**, über welches jede einzelne Queue mit dem Exchange verbunden ist. Das Binding definiert, nach welchen Kriterien eine Nachricht weitergeleitet werden soll. Es gibt insgesamt 4 Arten für den Datenaustausch:

- Direct –> Nachrichten werden sofort an Queue zugestellt (Routing key = Binding key)

- Fanout –> für Broadcast-Nachrichten an alle Queues

- Topic –> wie bei Publish/Subscribe-Kommunikation. Die Nachrichten werden abhängig vom Topic an eine oder mehrere Warteschlangen ausgeliefert.

- Headers –> Die Zustellung der Nachricht zur Warteschlange erfolgt hier über den Nachrichten-Header.

![amqp](/images/amqp.png)

AMQP selbst ausprobieren: http://tryrabbitmq.com

# MQTT vs AMQP

Die beiden Protokolle unterscheiden sich hauptsächlich in der Art der Nachrichtenzustellung, wobei MQTT nur Publish/Subscribe und AMQP mehrere Varianten von Zustellung bietet. Aufgrund dieses großen Umfangs von AMQP ist MQTT jedoch schneller zu implementieren. Zudem unterscheidet sich die Größe der Pakete sehr stark, denn ein leichtgewichtiges MQTT Paket benötigt im Gegensatz von >60byte bei AMQP nur mindestens 2 byte, was bei großen Mengen von Nachrichten von Vorteil ist.

### rabbitMQ

RabbitMQ ist ein Open-Source-Messaging Broker und basiert auf AMQP, kann jedoch mittlerweile auch schon für MQTT verwendet werden. Der Vorteil von RabbitMQ liegt darin, dass der Versender der Nachricht den Versand nicht selbst übernehmen muss, da dies vom Broker, welcher die Exchanges und Queues beinhaltet, übernommen wird. Außerdem muss der Sender nicht warten, bis der Empfänger die Nachricht empfangen hat, da diese von rabbitMQ in der Warteschlange zwischengespeichert und vom Konsumenten abgeholt werden kann. Es handelt sich also um ein **asynchrones Verfahren**: Sender und Empfänger müssen nicht im gleichen Rhythmus agieren.

Zudem können Nutzer RabbitMQ über ein GUI verwalten und haben den Überblick über Nachrichten in Warteschleifen bzw. können Statistiken einsehen. Außerdem ist es möglich, mehrere Broker-Instanzen miteinander zu verbinden um Lasten besser zu verteilen bzw. Ausfallsicherheit zu gewährleisten.

Die Kommunikation funktioniert über TCP, weshalb RabbitMQ **Ports** benötigt. Diese dürfen nicht geschlossen oder durch andere Anwendungen blockiert werden.



# JMS - Java Messaging Service (aus JEE)

JMS ist eine API, welche es Java EE Anwendungen ermöglicht, Nachrichten erstellen, verschicken und empfangen zu können. 

Grundsätzlich gibt es 2 Arten, Nachrichten zu verschicken:

- **Point-to-Point (PTP)** 
  - Realisierung durch Queues (siehe AMQP)
- **Publish-Subscribe (Pub/Sub)**
  - Verwendung von Topics (siehe MQTT)

Nachrichten können auf zwei Wege empfangen werden: **synchron** und **asynchron**. Der synchrone Empfang erfolgt über eine „receive“-Methode und beim asychronen Empfang muss sich der Empfänger an einen „message listener“ registrieren. Kommt dann eine Nachricht an, wird automatisch beim Empfänger eine „onMessage“-Methode aufgerufen.

JMS Architektur:

![JMS](/images/JMS.png)





# Quellen

- https://www.it-talents.de/blog/it-talents/was-ist-mqtt
- https://blog.doubleslash.de/mqtt-fuer-dummies/
- http://mosquitto.org
- https://www.openhab.org/docs/
- https://rabbitmq.com
- https://www.sic-software.com/iot-protokolle-mqtt-vs-amqp/
- https://www.rabbitmq.com/tutorials/amqp-concepts.html
