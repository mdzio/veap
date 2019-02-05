# VEAP-Demo-Server

Der VEAP-Demo-Server implementiert beispielhaft das [Very Easy Automation Protocol](https://github.com/mdzio/veap). Er soll als Referenzimplementierung und zum Testen von VEAP-Clients dienen.

**Zurzeit befindet sich der VEAP-Demo-Server noch in der Entwicklung!**

## Download

Der VEAP-Demo-Server besteht aus nur einer ausführbaren Datei ohne Abhängigkeiten. Sie kann aus folgenden Paketen entpackt und sofort gestartet werden:

* [veap-demo-server-0.1.0-win.zip](veap-demo-server-0.1.0-win.zip)
* [veap-demo-server-0.1.0-linux.tgz](veap-demo-server-0.1.0-linux.tgz)
* [veap-demo-server-0.1.0-macos.tgz](veap-demo-server-0.1.0-macos.tgz) (ungetestet)

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

(in Arbeit)

## Besondere Merkmale

* Unterstützung für HTTP/2 und TLS 1.2
* Automatische Generierung von Zertifikaten
 
## Zurzeit nicht implementierte VEAP-Merkmale

* Nicht-hierarchische Verlinkung
* Lesen- und Schreiben von Zeitreihen

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
