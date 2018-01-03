# Very Easy Automation Protocol (VEAP)

Im Folgenden wird ein TCP/IP-basiertes Kommunikationsprotokoll für den Bereich Automatisierung vorgestellt, das **einfach verständlich** und **leicht zu implementieren** ist und dennoch auch weitergehende Anforderungen, wie die **Modellierung von Datenmodellen** oder **Sicherheitsanforderungen**, erfüllt.

Das Protokoll kann beispielsweise im Bereich der Gebäudeautomatisierung oder für das Internet-der-Dinge eingesetzt werden. Es kann Geräte untereinander, mit überlagerten Systemen (z.B. Zentralen, Betriebsdatenerfassung) oder mit grafischen Benutzerschnittstellen verbinden.

Für die Ungeduldigen sind [Protokollbeispiele weiter unten](#beispiele) zu finden.

Das VEAP-Protokoll befindet sich zurzeit noch in der Entwicklung!

### Begründung für ein neues Protokoll

 Bereich der Automatisierung und dem Internet-der-Dinge existiert bereits eine Vielzahl an herstellerunabhängigen Protokollen (u.a. [MQTT](https://de.wikipedia.org/wiki/MQTT), [CoAP](https://de.wikipedia.org/wiki/Constrained_Application_Protocol), [OPC-UA](https://de.wikipedia.org/wiki/OPC_Unified_Architecture)). Eine Untersuchung hat allerdings ergeben, dass keines dieser Kommunikationsprotokolle alle oben geforderten Eigenschaften besitzt. Sie sind beispielsweise funktional eingeschränkt (z.B. MQTT) oder aufwändig zu implementieren (z.B. CoAP, OPC-UA). 

**Es gibt natürlich andere Anforderungen, die nur von einem der o.g. Protokolle (MQTT, CoAP oder OPC-UA) erfüllt werden und nicht von VEAP. Jedes Protokoll besitzt eigene Herausstellungsmerkmale.**

## Protokollgrundlagen

Um die Anforderungen zu erfüllen, muss das Protokoll zwingend auf bereits vorhandene und etablierte Technologien aufbauen. Gerade im Bereich von TCP/IP-basierter Kommunikation bietet sich der Einsatz von Web-Technologien an. Daher macht VEAP Gebrauch von folgenden Standards:
* Transportprotokoll: [Hypertext Transfer Protocol (HTTP)](https://de.wikipedia.org/wiki/Hypertext_Transfer_Protocol)
  * Unterscheidung von verschiedenen Protokoll-Diensten (z.B. Datenpunkt lesen/schreiben) durch HTTP-Verben (z.B. GET, PUT)
* Adressierung von Objekten/Datenpunkten: [Uniform Resource Locator (URL)](https://de.wikipedia.org/wiki/Uniform_Resource_Locator)
* Weitere Eigenschafte von [Representational State Transfer (REST)](https://de.wikipedia.org/wiki/Representational_State_Transfer)
* Abbildung von Objektbeziehungen: [HTTP Link Header (RFC 5988)](https://tools.ietf.org/html/rfc5988)
* Serialisierung (und Datentypen): [JavaScript Object Notation (JSON)](https://de.wikipedia.org/wiki/JavaScript_Object_Notation)
* Authentifizierung: [HTTP-Authentifizierung](https://de.wikipedia.org/wiki/HTTP-Authentifizierung)
* Transportverschlüsselung: [HTTPS](https://de.wikipedia.org/wiki/Hypertext_Transfer_Protocol_Secure)

### Vorteile

Aus der obigen Wahl der Basis-Technologien ergeben sich viele Vorteile:
* Einfache Verständlichkeit
  * Das Protokoll ist auch für Nicht-Programmierer verständlich und anwendbar.
  * Die Protokollnachrichten sind für Menschen lesbar.
* Leichte Implementierbarkeit
  * Die Nutzung grundlegender Dienste des Protokolls (z.B. das Lesen und Schreiben von Datenpunkten) ist in den meisten Programmiersprachen mit wenigen Zeilen Quelltext ohne Einbindung einer externen Bibliothek möglich.
  * Mit Werkzeugen, die für die meisten Betriebssysteme bereits existieren, sind die Protokolldienste bereits verwendbar (z.B. wget, curl).
  * Es wird keine besondere Infrastruktur vorausgesetzt (wie z.B. ein Broker bei MQTT). Dadurch ist der Inbetriebnahme- und Wartungsaufwand geringer.
* Sicherheit
  * Ohne nennenswerten Mehraufwand können Benutzer authentifiziert werden. Zudem ist eine auf dem aktuellen Stand der Technik basierende Verschlüsselung mit Zertifikaten verfügbar.

## Datenmodell

Im Bereich der Automatisierung muss ein Protokoll Messwerte (z.B. Temperaturen) von Sensoren lesen und Zustände von Aktoren (z.B. Heizventile) setzen können. Allgemein wird hier für einen Sensor- oder Aktorwert der Begriff Datenpunkt verwendet. Des Weiteren sollen die Datenpunkte erkundet werden können. Die Kommunikationspartner brauchen also kein Vorwissen darüber, welche Datenpunkte überhaupt existieren.

Eine weitere Informationsebene sind Beziehungen von Datenpunkten zu Objekten (z.B. dem Gerät, das diesen Datenpunkt enthält) oder Beziehungen zwischen Objekten (z.B. ein Gerät ist mit einer Automatisierungszentrale verbunden). Jedes Objekt soll beliebige weitere Eigenschaften besitzen können (z.B. Raumname, Messbereichsanfang/-ende). Ein Datenpunkt ist selber ein spezialisiertes Objekt, das zusätzliche Dienste anbieten (Wert lesen und schreiben).

### Objekte

Jedes Objekt wird über einen eindeutigen Pfad im VEAP-Server adressiert (z.B. ```/bidcos-rf/abc012345/1/STATE```). Es kann beliebige applikationsabhängige Eigenschaften und Beziehungen (Relationen) zu anderen Objekten besitzen und verschiedene Dienste (z.B. _Prozesswert lesen/schreiben_) anbieten.

Adressbestandteile, die mit einer Tilde (~) beginnen, sind für das VEAP-Protokoll reservierte Schlüsselwörter und dürfen nicht in Adressen von Objekten verwendet werden.

#### Besondere Objekte

Über das Objekt ```/~vendor``` können Eigenschaften des VEAP-Servers erkundet werden. Folgende Eigenschaften sind definiert:
* serverName (optional)
* serverVersion (optional)
* serverDescription (optional)
* vendorName (optional)

## Dienste

Jedes Objekt kann verschiedene Dienste anbieten. Die Dienste werden auf ein HTTP-Verb (z.B. GET) und/oder auf ein spezielles Adressende (z.B. ```/~pv```) abgebildet. Die zu übertragenden Datenstrukturen werden mit Hilfe der [JavaScript Object Notation (JSON)](https://de.wikipedia.org/wiki/JavaScript_Object_Notation) kodiert. Die Standardzeichenkodierung ist [UTF-8](https://de.wikipedia.org/wiki/UTF-8).

### Datenpunkt lesen

Über die HTTP-Methode GET und der Adressendung ```/~pv``` wird von einem Objekt der aktuelle Prozesswert angefragt. Ein Prozesswert besteht aus bis zu drei Komponenten: Wert (v), Zeitstempel  (ts) und Status (s). Zeitstempel und Status sind optional. Ein entsprechendes JSON-Objekt sieht dann beispielsweise wie folgt aus:
```
{
  "v": 123.456,
  "ts": 1483228800000,
  "s": 0
}
```

Der Wert (v) ist ein beliebiger JSON-Datentyp. Komplexe Datenstrukturen sind zulässig, sollten aber aus Interoperabilitätsgründen vermieden werden. 

Der Zeitstempel (ts) gibt den ursprünglichen Lesezeitpunkt des Sensors an. Er ist ganzzahlig und gibt die Anzahl der Millisekunden seit 1.1.1970 UTC an. Die Zahl 1483228800000 entspricht also dem Zeitpunkt 1.1.2017 01:00:00 Uhr MEZ. Die Zeitstempel ist optional.

Der Messwertstatus ist eine ganzzahlige Nummer aus drei Bereichen. Die verwendete Nummer innerhalb eines Bereichs ist applikationsspezifisch (z.B. 100=Batterie leer, 200=Sensor defekt, 201=Messbereichsunterschreitung, 202=Messbereichsüberschreitung). Der Status ist optional. Wenn der Status nicht angegeben worden ist, so wird er implizit als _GOOD_ (Wert 0) angenommen.

Bereichswerte für den Status (s):

Nummer      | Symbol      | Beschreibung
------------|-------------|-------------
0 bis 99    | _GOOD_      | Dem Messwert kann vertraut werden.
100 bis 199 | _UNCERTAIN_ | Der Messwert ist fragwürdig.
200 bis 299 | _BAD_       | Der Messwert ist schlecht und sollte nicht weiter verarbeitet werden.

### Datenpunkt schreiben

Über die HTTP-Methode PUT (ein Server kann zusätzlich auch die POST-Methode akzeptieren) und der Adressendung ```/~pv``` wird von einem Datenpunkt der aktuelle Prozesswert beschrieben. Der Prozesswert ist [wie oben beschrieben](#datenpunkt-lesen) aufgebaut.

### Objekt-/Datenpunkteigenschaften lesen

Über die HTTP-Methode GET und der unveränderten Objektadresse werden applikationsspezifische Eigenschaften ([und auch die Beziehungen](#objektbeziehungen-erkunden)) eines Objekts ausgelesen. Diese Eigenschaften verändern sich im Gegensatz zum Prozesswert selten. 

Beispiel-Antwort:
```
{
  "name": "HWR Deckenlampe",
  "description": "4xGU10,20W",
  "address": "BidCos-RF.ABC01232:3.STATE",
  "iseId": 1234,
  "rooms": "HWR,UG",
  "functions": "Licht"
}
```

### Objektbeziehungen erkunden

Ein Objekt bzw. Datenpunkt kann selber andere Objekte enthalten, ähnlich wie ein Ordner in einem Dateisystem, oder auf andere Objekte verweisen, ähnlich wie ein Verweis bzw. Link in einem Dateisystem. Wenn, wie im [vorigen Abschnitt](#objekt-datenpunkteigenschaften-lesen) beschrieben, die Objekteigenschaften gelesen werden, so werden zusätzlich die referenzierten Objekte aufgelistet.

Dies geschieht über Link-Einträge im HTTP-Header der Form ```Link: </a/b/c>; rel="typ"```. Bei ```/a/b/c``` handelt sich um die absolute oder relative Adresse des referenzierten Objektes. Mit ```rel="typ"``` wird der Typ der Relation angegeben. Es können mehrere HTTP-Link-Header existieren, oder ein HTTP-Link-Header besitzt mehrere Einträge, die durch Kommas separiert sind.

Beispiel-Header:
```
Link: </a>; rel="/~rel/comp",
  </b>; rel="/~rel/comp"
```

Es sind folgende Beziehungstypen definiert:
* /~rel/comp \
  Das referenzierte Objekt ist im übergeordneten Objekt enthalten (wie in einem Dateisystemordner).
* /~rel/link \
  Auf das referenzierte Objekt wird verwiesen (wie ein Dateisystemverweis). Die Verwendung dieses Typs ist optional.
* /~rel/svc \
  Es wird auf ein Dienst-Objekt verwiesen. Die Verwendung dieses Typs ist optional.
  * /~rel/svc/pv (Datenpunkt lesen/schreiben)
  * /~rel/svc/hist (Historie lesen)

Mit Hilfe der Relation ```/~rel/comp``` können Datenpunkte wie in einem Verzeichnisbaum einsortiert werden. 

Beziehungstypen können weiter verfeinert werden. So kann auf einen Kanal von einem Gerät wie folgt verwiesen werden: ```rel="/~rel/comp/channel"```. Mit ```rel="/~rel/link/room"``` wird auf das zugehörige Raumobjekt verwiesen.

Eine Tiefensuche darf nur den Relationen vom Typ ```rel="/~rel/comp"``` folgen. Dazu zählen auch verfeinerte Angaben (z.B. ```/~rel/comp/channel```).

Über ```/~rel/svc``` können die angebotenen Dienste eines Objektes erkundet werden (z.B. Datenpunkt lesen/schreiben).

### Signalisierung von Fehlern

Dienst-Fehler werden über den zurückgegebenen HTTP-Status signalisiert. Die verwendeten Statuscodes des VEAP-Protokolls entprechen in ihrer Bedeutung den [HTTP-Statuscodes](https://de.wikipedia.org/wiki/HTTP-Statuscode).

HTTP-Status|Bedeutung
----------:|:--------
   200     | OK
   400     | Ungültige Anfrage (HTTP-Protokollfehler)
   401     | (Benutzer-)Authentifizierung nötig
   403     | Zugriffsrechte fehlen
   404     | Objekt/Dienst nicht vorhanden
   422     | Anfrage entspricht nicht dem VEAP-Protokoll
   500     | Unerwarteter Fehler im Server

Für einen einfachen VEAP-Client reicht es aus, den HTTP-Status mit 200 zu vergleichen. Jeder Status ungleich 200 wird dann als Fehler interpretiert.

### Historie eines Datenpunktes lesen (Optional)

Über die HTTP-Methode GET und der Adressendung ```/~hist``` wird die Historie eines Datenpunktes angefragt. Die Historie gibt den zeitlichen Verlaufs eines Datenpunktwertes wieder. Der abgefragte Zeitbereich wird über GET-Parameter angegeben.

Folgende GET-Parameter sind definiert:

Parametername|Bedeutung
-------------|---------
begin        |Beginn des Zeitbereichs (Millisekunden seit 1.1.1970 UTC, Zeitpunkt eingeschlossen)
end          |Ende des Zeitbereichs (Millisekunden seit 1.1.1970 UTC, Zeitpunkt ausgeschlossen)
limit        |Maximale Anzahl an zurückzugebenden Einträgen (optional)

```begin``` und ```end``` geben den angefragten Zeitbereich an. Die Zeitstempel der zurückgegebenen Zeitreihe müssen größer oder gleich ```begin``` und kleiner als ```end``` sein. Wenn beide Parameter fehlen, so wird die Zeitreihe der letzten 24 Stunden zurückgegeben. 

Mit ```limit``` kann optional die Anzahl der zurückzugebenden Einträge beschränkt werden. Die Anzahl der zurückgegebenen Einträge muss immer kleiner oder gleich ```limit``` sein. Wenn dieser Parameter nicht angegeben wurde, entscheidet der VEAP-Server über eine mögliche Begrenzung.

Beispiel-Anfrage (Datenpunkt **a**, 1.1.2018 bis 2.1.2018 MEZ, max. die ersten 10 Einträge):
```
GET /a/~hist?begin=1514761200&end=1514847600&limit=10
```

Der Server antwortet mit einem JSON-Objekt, dass aus den Feldern ```v``` (Werte), ```ts``` (Zeitstempel) und optional ```s``` (Status) besteht. Die Felder sind jeweils JSON-Arrays, sie müssen alle die gleiche Länge besitzen. Die Bedeutung der Elemente ist identisch wie beim [Lesen eines Datenpunktes](#datenpunkt-lesen). Die Arrays müssen zeitlich aufsteigend sortiert sein. Die zeitliche Sortierung der Einträge erfolgt vor Anwendung des Limits. 

Beispiel-Antwort:
```
{
  "v": [123.456, 234.567, 345.678],
  "ts": [1514761200, 1514761201, 1514761202],
  "s": [0, 0, 0]
}
```

### Historie eines Datenpunktes schreiben (Optional)

Über die HTTP-Methode PUT (ein Server kann zusätzlich auch die POST-Methode akzeptieren) und der Adressendung ```/~hist``` wird die Historie eines Datenpunktes beschrieben. Die Historie ist [wie oben beschrieben](#historie-eines-datenpunktes-lesen-optional) aufgebaut.

## Beispiele

(In den unten angegebenen Anfragen und Antworten wird das HTTP-Protkoll nur vereinfacht dargestellt.)

### Minimaler VEAP-Server mit einer flachen Hierarchie

Der im Folgenden beschriebene VEAP-Server bietet die zwei Datenpunkte A und B in einer flachen Hierarchie an.

#### Datenpunkt lesen

Im Beispiel ist  ```/a``` die  (beliebig aufgebaute) Adressierug des Datenpunktes und ```~pv``` (Process value) die Adressierung des Dienstes _Prozesswert lesen/schreiben_.

Anfrage:
```
GET /a/~pv
```

Antwort:

Im Feld "v" ist der aktuelle Wert des Datenpunktes. Die Felder "ts" (Zeitstempel; Millisekunden seit 1.1.1970 UTC) und "s" (Status: 0=good) sind optional.
```
HTTP 200 OK
{
  "v": 123.456,
  "ts": 1483228800000,
  "s": 0
}
```

#### Datenpunkt schreiben

Im Beispiel ist ```/b``` die Adressierug des Datenpunktes und ```~pv``` (Process value) die Adressierung des Dienstes _Prozesswert lesen/schreiben_. Im Feld "v" ist der zu setzende Wert des Datenpunktes. Die Felder "ts" (Zeitstempel) und "s" (Status) sind optional.

Anfrage:
```
PUT /b/~pv
{
  "v": 123.456,
  "ts": 1483228800000,
  "s": 0
}
```
Antwort:
```
HTTP 200 OK
```

#### Datenpunkteigenschaften lesen

Im Beispiel ist ```/a``` die Adresse des abzufragenden Objektes/Datenpunktes.

Anfrage:
```
GET /a
```

Antwort:

Es können beliebige geräteabhängige Eigenschaften zurück geliefert werden.
```
HTTP 200 OK
{
  "name": "HWR Deckenlampe",
  "description": "4xGU10,20W",
  "address": "BidCos-RF.ABC01232:3.STATE",
  "iseId": 1234,
  "rooms": "HWR,UG",
  "functions": "Licht"
}
```

#### Eigenschaften des VEAP-Servers lesen

Über das besondere Objekt ```/~vendor``` können Eigenschaften des VEAP-Servers an sich gelesen werden.

Anfrage:
```
GET /~vendor
```

Antwort:
```
HTTP 200 OK
{
  "serverName": "Minimal VEAP-Server",
  "serverVersion": "0.1.0",
  "serverDescription": "A minimal VEAP-Server for demonstration",
  "vendorName": "VEAP"
}
```

#### Datenpunkte/Objektbeziehungen erkunden

Im Beispiel ist ```/``` die Adresse des übergeordneten Objektes, das andere Objekte/Datenpunkte enthalten kann.

Anfrage:
```
GET /
```

Antwort:

Die untergeordneten Objekte werden über HTTP-Link-Header zurück gegeben. Optional können auch Eigenschaften des abgefragten Objektes zurück geliefert werden (z.B. "name"). Der Relationstyp ```/~rel/comp``` besagt, dass ```/a```, ```/b``` und ```/~vendor``` Unterobjekte von ```/``` sind (s.a. [Objektbeziehungen](#objektbeziehungen-erkunden)).

```
HTTP 200 OK
Link: </a>; rel="/~rel/comp",
  </b>; rel="/~rel/comp",
  </~vendor>; rel="/~rel/comp",

{
  "name": "Wurzelverzeichnis"
}
```

## Geplante Erweiterungen

### Ereignisorientierte Abfrage von Datenpunkten (Optional)

Wenn ein VEAP-Server diesen Dienst anbietet, hat ein Client die Möglichkeit auf Datenpunktänderungen sofort reagieren zu können.

_Dieser Dienst muss noch spezifiziert werden._

## Konzeptionelle Entscheidungen

In diesem Abschnitt sind einige grundlegende Entscheidungen zum Protokoll-Design erklärt.

### Mehrere Datenpunkte im Paket lesen oder schreiben

Das Protokoll besitzt keinen Dienst um mehrere Datenpunkte in einem Paket zu lesen oder zu schreiben. Dieser Dienst hätte folgende Vorteile:

* Reduzierung von Netzwerkanfragen
* Atomares Lesen oder Schreiben von mehreren Datenpunkten

Die zwei Vorteile wiegen aber nicht die dadurch erhöhte Komplexität des Protokolls aus folgenden Gründen auf:

* Durch HTTP-Pipelining (HTTP/1.1) bzw. -Multiplexing (HTTP/2) kann bereits durch das unterlagerte Protokoll die Anzahl der Netzwerkanfragen reduziert werden.
* Für das atomare Lesen oder Schreiben von mehreren Datenpunkten existieren keine relevanten Anwendungsfälle, die nicht durch andere Möglichkeiten des Protokolls gelöst werden können. 

## Lizenz

Dieses Werk ist lizenziert unter einer [Creative Commons Namensnennung - Weitergabe unter gleichen Bedingungen 4.0 International Lizenz](http://creativecommons.org/licenses/by-sa/4.0/).

[![Lizenz](https://i.creativecommons.org/l/by-sa/4.0/88x31.png)](https://creativecommons.org/licenses/by-sa/4.0/)

