# VEAP-Demo-Server

Der VEAP-Demo-Server implementiert beispielhaft das [Very Easy Automation Protocol](https://github.com/mdzio/veap). Er soll als Referenzimplementierung und zum Testen von VEAP-Clients dienen.

## Besondere Merkmale

* Unterstützung für HTTP/2 und TLS 1.2
* Automatische Generierung von Zertifikaten
 
## Download

Der VEAP-Demo-Server besteht aus nur einer ausführbaren Datei ohne Abhängigkeiten. Sie kann aus folgenden Paketen entpackt und sofort gestartet werden:

* [veap-demo-server-1.0.1-win.zip](https://github.com/mdzio/veap/raw/master/veap-demo-server/veap-demo-server-1.0.1-win.zip)
* [veap-demo-server-1.0.1-linux.tgz](https://github.com/mdzio/veap/raw/master/veap-demo-server/veap-demo-server-1.0.1-linux.tgz)
* [veap-demo-server-1.0.1-darwin.tgz](https://github.com/mdzio/veap/raw/master/veap-demo-server/veap-demo-server-1.0.1-darwin.tgz) (ungetestet)

### Änderungen

* 1.0.0
  * Schreiben von Zeitreihen (Pfad /whist)
* 0.3.0
  * Nicht-hierarchische Verlinkung (Pfad /linked)
* 0.2.0
  * Lesen von Zeitreihen (Pfad /hist)
  * Dynamisches Erstellen und Löschen von Objekten (Pfad /dyn) 
* 0.1.0
  * Erste veröffentlichte Version

## Kommandozeilenoptionen

Die Standardeinstellungen können durch Kommandozeilenoptionen überschrieben werden. Bei Verwendung der Kommandozeilenoption `-h` wird folgender Hilfetext ausgegeben:

```
usage of veap-demo-server:
  -host name
        name or IP address of the host running this server (normally autodetected)
  -log severity
        specifies the minimum severity of printed log messages: off, error, warning, info, debug or trace (default INFO)
  -password password
        password for HTTP Basic Authentication, q.v. -user
  -port port
        port for serving HTTP (default 2121)
  -porttls port
        port for serving HTTPS (default 2122)
  -user name
        user name for HTTP Basic Authentication (disabled by default)
```

## Beispielabfragen

Folgende Beispielabfragen können mit einem einfachen Web-Browser durchgeführt werden:

### Erkundung des Wurzelobjekts

Anfrage: \
HTTP-GET auf http://localhost:2121

Antwort:
```
{
    "description": "Root object of the VEAP object tree",
    "identifier": "root",
    "title": "Root",
    "~links": [
        {
            "rel": "component",
            "href": "const",
            "title": "Constant values"
        },
        {
            "rel": "component",
            "href": "states",
            "title": "State values"
        },
        {
            "rel": "component",
            "href": "gen",
            "title": "Generators"
        },
        {
            "rel": "component",
            "href": "vars",
            "title": "Variables"
        },
        {
            "rel": "component",
            "href": "types",
            "title": "Data Types"
        },
        {
            "rel": "component",
            "href": "ts",
            "title": "Timestamps"
        },
        {
            "rel": "component",
            "href": "tree",
            "title": "Tree"
        },
        {
            "rel": "component",
            "href": "~vendor",
            "title": "Vendor Information"
        }
    ]
}
```

### Erkundung von /gen

Anfrage: \
HTTP-GET auf http://localhost:2121/gen

Antwort:
```
{
    "description": "Various generators. Variable x is set to the number of seconds since 01.01.1970 00:00 UTC (unix epoch).",
    "identifier": "gen",
    "title": "Generators",
    "~links": [
        {
            "rel": "generator",
            "href": "rand",
            "title": "rand"
        },
        {
            "rel": "generator",
            "href": "x",
            "title": "x"
        },
        {
            "rel": "generator",
            "href": "sin_2pix_10",
            "title": "sin(2*pi*x/10)"
        },
        {
            "rel": "parent",
            "href": "..",
            "title": "Root"
        }
    ]
}
```

### Erkundung von /gen/rand

Anfrage: \
HTTP-GET auf http://localhost:2121/gen/rand

Antwort:
```
{
    "description": "Random number in [0, 1.0).",
    "identifier": "rand",
    "title": "rand",
    "~links": [
        {
            "rel": "parent",
            "href": "..",
            "title": "Generators"
        },
        {
            "rel": "service",
            "href": "~pv",
            "title": "PV Service"
        }
    ]
}
```

### Prozesswert von /gen/rand lesen

Anfrage: \
HTTP-GET auf http://localhost:2121/gen/rand/~pv

Antwort:
```
{
    "ts": 1594594800000,
    "v": 0.2065826619136986,
    "s": 0
}
```

### Historie von /hist/sin_2pix_10/1s lesen

Zeitbereichsbegin: 1.1.2019 00:00:00 UTC \
Zeitbereichsende: 1.1.2019 00:00:20 UTC

Anfrage: \
HTTP-GET auf http://localhost:2121/hist/sin_2pix_10/1s/~hist?begin=1546300800000&end=1546300820000

Antwort:
```
{
    "ts": [
        1546300800000, 1546300801000, 1546300802000, 1546300803000, 1546300804000,
        1546300805000, 1546300806000, 1546300807000, 1546300808000, 1546300809000,
        1546300810000, 1546300811000, 1546300812000, 1546300813000, 1546300814000,
        1546300815000, 1546300816000, 1546300817000, 1546300818000, 1546300819000
    ],
    "v": [
        -1.0120201733657937e-7, 0.5877851845636222, 0.9510565326657553, 0.9510565681969049, 0.5877852775854129,
        1.3779237332854878e-8, -0.5877851588477938, -0.951056486005484, -0.9510565043440965, -0.5877853033012385,
        -4.5565746879911025e-8, 0.5877852295743122, 0.9510564761829089, 0.9510565141666689, 0.5877852325747247,
        7.735225642696754e-8, -0.5877852038584851, -0.9510565400357351, -0.951056560826927, -0.5877852582905514
    ],
    "s": [
        0, 0, 0, 0, 0,
        0, 0, 0, 0, 0,
        0, 0, 0, 0, 0,
        0, 0, 0, 0, 0
    ]
}
```

## Sicherer Zugriff über HTTPS

Der VEAP-Demo-Server ermöglicht einen verschlüsselten Zugriff über HTTPS, sodass auch über unsichere Netzwerke (z.B. Internet) Daten sicher ausgetauscht werden könnan. Über den Port 2122 (änderbar mit der Kommandozeilenoption `-porttls`) kann eine HTTPS-Verbindung aufgebaut werden. Die dafür benötigten Zertifikate können vorgegeben werden oder werden beim ersten Start vom VEAP-Demo-Server automatisch generiert.

Benötigte Zertifikatsdateien für den Server:

Dateiname   | Funktion
------------|---------
svrcert.pem | Zertifikat des Servers
svrcert.key | Privater Schlüssel des Servers (Dieser ist geheim zu halten.)

Falls die oben genannten Zertifikatsdateien im Arbeitsverzeichnis des VEAP-Demo-Servers nicht vorhanden sind, so werden automatisch zwei Zertifikate erstellt. Die Gültigkeit ist auf 10 Jahre eingestellt:

Dateiname   | Funktion
------------|---------
cacert.pem  | Zertifikat der Zertifizierungsstelle (CA)
cacert.key  | Privater Schlüssel der Zertifizierungsstelle (Dieser ist geheim zu halten.)
svrcert.pem | Zertifikat des Servers
svrcert.key | Privater Schlüssel des Servers (Dieser ist geheim zu halten.)

Für den sicheren Zugriff muss lediglich das generierte Zertifikat der Zertifizierungsstelle (`cacert.pem`) den HTTPS-Clients *über einen sicheren Kanal* bekannt gemacht werden. Das Zertifikat kann z.B. im Betriebssystem oder im Web-Browser installiert werden. Die privaten Schlüssel dürfen nie verteilt werden.

Über verschiedene Programmiersprachen kann auch verschlüsselt zugegriffen werden.

### Curl

```bash
curl --cacert path/to/cacert.pem https://hostname:2122
```

### Python

```python
import requests
r = requests.get("https://hostname:2122", verify='path/to/cacert.pem')
print(r.status_code)
```

### Go

```go
caCert, err := ioutil.ReadFile("path/to/cacert.pem")
if err != nil {
    log.Fatal(err)
}
caCerts := x509.NewCertPool()
ok := caCerts.AppendCertsFromPEM(caCert)
if !ok {
    log.Fatal("Failed to parse certificate")
}
con, err := tls.Dial("tcp", "hostname:2122", &tls.Config{RootCAs: caCerts})
if err != nil {
    log.Fatal(err)
}
defer con.Close()
```


### Javascript

```javascript
var fs = require('fs');
var https = require('https');

var get = https.request({
  path: '/', hostname: 'hostname', port: 2122,
  ca: fs.readFileSync('path/to/cacert.pem'),
  agent: false,
  rejectUnauthorized: true,
}, function(response) {
  response.on('data', (d) => {
    process.stdout.write(d);
  });
});
get.on('error', function(e) {
  console.error(e)
});
get.end();
```
