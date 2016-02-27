**Wireless Sensor Networks** kann benutzen für:

* Precision Agriculture
* Forest-Fire Detection
* Exploration of Unknown Territory
* Traffic Telematics
* Und etc.

Sensorknoten leiden unter einigen Limitierungen:

* Geringe Bandbreite
* Geringe Akkukapazität
* Geringer Speicherplatz
* Geringe Rechenleistung

WSNs sind **Sensor Nodes** (Sensorknoten) über ein **Sensor Field** verteilt. Sie kommunizieren via einer **Sink**-Einheit mit zum Beispiel dem Internet. Sie können zufällig (zB aus Flugzeug geworfen) oder geplant (von Hand) verteilt werden, oder die Sensorknoten sind mobil.

# Herausforderungen und Methoden

## Geringe Akkukapazität

![Needs more JPG: Crappige Darstellung von Moore's Law vs. Battery Capacity](/img/moore-vs-battery.jpg)

**Ampere-Stunden** $[Ah]$ sagen aus, wie viele Stunden man ein einen Strom von 1 Ampere ziehen kann.

Die Energiekapazität von Batterien ist beschränkt.

Ein Beispiel: Ein Tmote Sky Sensorknoten benötigt $0.5 mA$ für den Prozessor und $17.4 mA$ für den Transmitter. Er kann mit zwei AA Alkaline Batterie betrieben werden, welche jeweils eine Kapazität von $2200 mAh$ haben. Die Lebensdauer der Batterien lässt sich ausrechnen:

$$
lifetime = \frac{2 \cdot 2200}{17.4 + 0.5} \quad \left[ \frac{mAh}{mA} = h \right] \quad \approx 10.2 \quad [days]
$$

Um die Lebenszeit zu erhöhen kann der Transmitter nach dem Senden auf "nur noch hören" gestellt werden, wo er nur noch $0.365 mA$ braucht. Bei einer Übertragungsrate von $250 \cdot 1024 = 256000 \ bps$ entspricht die Übertragungszeit für 16 Bit einem $16 / 256000 = 1 / 16000$-stel einer Sekunde. Also wird für eine Sekunde benötigt:

$$
\frac{1}{16000} \cdot 17.9 + (1 - \frac{1}{16000}) \cdot 0.365 \approx 0.5 mA
$$

$$
lifetime = \frac{4400}{0.5 + 0.5} \approx 183 \quad [days]
$$

*Todo: Mal die Sachen von den Folien nachrechnen, da stimmt vielleicht was nicht.*

Durch das geschickte Nutzen von verschiedenen Wachheits-Phasen (transmit, receive, idle, sleep) kann man die Lebensdauer theoretisch auf mehrere Jahre erhöhen, aber es gibt auch noch andere Energieverbrauche auf einem Sensorknoten, die Sensorknoten brauchen einiges an Kommunikation für Synchonisation und Batterien verlieren an Stromstärke über Zeit.

## In-Network-Processing
Um die Menge an zu übertragenden Daten zu verringern können kann im Netzwerk bereits ein gewisses Preprocessing gemacht werden, das irrelevante Daten verwirft.

## Multihop-Kommunikation

Statt direkt zu einem weit entfernten Ziel zu senden können Nachrichten über mehrere dazwischen liegende Knoten geleitet werden, was Energie spart.


# MAC-Layer-Fallstudie IEEE 802.15.4

*Wird nicht wirklich gut erklärt und ist wahrscheinlich nicht sehr relevant.*


# Energieeffiziente MAC-Layer

## S-MAC und T-MAC

**Sensor-MAC** legt Knoten periodisch schlafen. Diese Schlafzyklen werden mit Nachbarn innerhalb virtueller Cluster synchronisiert.