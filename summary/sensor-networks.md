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

## S-MAC

**Sensor Media Access Control** legt Knoten periodisch schlafen. Diese Schlafzyklen müssen über das ganze Netzwerk synchron sein, weil sonst die Knoten Nachrichten verschlafen.

Problem: [Clock-Drift macht die Hitreg kaputt](https://www.reddit.com/r/GlobalOffensive/comments/3zsmxd/clock_drift_issue_making_client_and_server_out_of/) und die Schlafzyklen desynchronisieren.

Lösung: Synchronisationsnachrichten während jedes Wachzyklus verschicken. Dabei muss ein sinnvolles Führer/Nachfolger-Verhältnis aufgebaut werden und „Inseln” vermieden werden.

## T-MAC

![T-MAC-Problem und -Lösung](/img/tmac.png)

**Timeout Media Access Control** ist eine Erweiterung von S-MAC. Statt dass die Knoten während des ganzen Wachzyklus auf Nachrichten warten, legen sie sich nach einem gehörten Clear To Send wieder für die erwartete Übertragungszeit schlafen, da sie eh nichts tun können während die andere Übertragung läuft. Falls gar kein Request To Send kommt, legen sie sich ebenfalls schlafen, und zwar bis zum nächsten Wachzyklus.

Problem: Knoten $s$ will an $f_3$ senden, hört aber ein CTS von $f_2$. $f_3$ hört das nicht, weil er zu weit weg ist. Also legt sich $f_3$ nach der Contention Period bis zum nächsten Wachzyklus schlafen, $s$ aber nur bis $f_2$ fertig empfangen hat. $f_3$ verschläft dann die Nachricht von $s$.

Lösung: Es wird ein **Future Request To Send (FTRS)** verschuckt, das $f_3$ hört und wieder aufwachen lässt. Nach $CTS$ gibt es eine Verzögerung bis die Daten verschickt werden, damit ein etwaiger FTRS nicht mit den Daten kollidiert.

## B-MAC

![B-MAC inklusive Problemen](/img/bmac.png)

**Berkeley Media Access Control** synchronisiert Knoten nicht. Stattdessen schicken Knoten, die etwas zu senden haben, eine Präambel, die lang genug ist, dass der Empfängerknoten aufwachen und bemerken kann, dass die Nachricht für ihn ist. Nach der Präambel werden die Daten verschickt, kein CTS findet statt.

Probleme siehe Bild.

## X-MAC

Ich glaube das X steht für eXtended preamble, aber das wird [aus dem Paper nicht klar](http://web.stanford.edu/class/cs244e/papers/xmac.pdf). Es ist eine Erweiterung von B-MAC.

Statt die ganze Zeit zu präamblen, werden periodisch Präambel-„Strobes” verschickt. Wenn der Empfängerknoten einen solchen hört, schickt er ein ACK und das Datensenden beginnt. Andere Knoten legen sich durch die Strobes wieder schlafen.

Problem: im Prinzip wäre ja nur ein Strobe notwendig, kurz bevor der Empfänger aufwacht.

## Wise-MAC

Beim **Wireless Sensor MAC** sendet der Empfänger nach dem Empfang von Daten noch wie lange er sich schlafen legen wird. Falls der gleiche Sender dann nochmal Daten für ihn hat, weiß er genau, wann er den Strobe schicken muss, damit der Empfänger ihn hört. Es wird also ganz minimal synchronisiert.

# WSN-Programmierung

Operating Systems auf microcontrollers können aus Effizienzgründen nicht die üblichen Funktionen wie memory management und process management anbieten. Da aber meistens eh nur ein einzelnes Programm auf einer WSN-Node läuft, reicht es eine Operating System-artige Runtime zur Verfügung zu stellen.

**TinyOS** benutzt **nesC** als Programmiersprache, ist zwar nicht so einfach zu benutzen und nicht so portabel, braucht dafür aber nicht viel Arbeitsspeicher.

Um **Interrupts** von Sensoren behandeln zu können, werden "minimale Interrupts" (**preemption**) benutzt: Das Programm läuft normal und springt bei einem Interrupt sofort in dessen Handler, speichert aber nur die Daten für spätere Behandlung und macht dann normal weiter. Wenn später Zeit ist, werden die Daten fertig behandlet. Die Interrupts müssen so kurz wie möglich sein, da weitere Interrupts in dieser Zeit nicht behandelt werden können.

**Components** (im Sinne von TinyOS) sind eine Alternative zu Prozessen. Jede Komponente erfüllt eine einzelne Funktionalität. Alle Komponenten teilen sich einen Adressraum. Komplexere Komponenten können mit Hilfe wohldefinierter Schnittstellen aus einfachen Komponenten "zusammengebaut" werden.

**Networking** wird wie vieles Event-based gemacht, da Funktionen die blockieren nicht so cool sind.

**Split-Phased Programming** bezeichnet die Praxis, eine Funktion einer anderen Komponente anzustoßen und dann auf ein Ergebnisevent zu warten.
