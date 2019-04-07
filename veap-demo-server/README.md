# VEAP demo server

The VEAP demo server implements the [Very Easy Automation Protocol](https://github.com/mdzio/veap) as an example. It shall serve as a reference implementation and for testing VEAP clients.

## Special features

* Support for HTTP/2 and TLS 1.2
* Automatic generation of certificates
 
## Download

The VEAP demo server consists of only one executable file without dependencies. It can be unpacked from the following packages and started immediately:

* [veap-demo-server-1.0.0-win.zip](https://github.com/mdzio/veap/raw/master/veap-demo-server/veap-demo-server-1.0.0-win.zip)
* [veap-demo-server-1.0.0-linux.tgz](https://github.com/mdzio/veap/raw/master/veap-demo-server/veap-demo-server-1.0.0-linux.tgz)
* [veap-demo-server-1.0.0-darwin.tgz](https://github.com/mdzio/veap/raw/master/veap-demo-server/veap-demo-server-1.0.0-darwin.tgz) (ungetestet)

### Changes

* 1.0.0
  * Write history (path /whist)
* 0.3.0
  * Non-hierarchical linking (path /linked)
* 0.2.0
  * Read history (path /hist)
  * Dynamic creation and deletion of objects (path /dyn) 
* 0.1.0
  * First published version

## Command line options

The default settings can be overwritten by command line options. When using the `-h` command line option, the following help text is output:

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

## Sample queries

The following sample queries can be performed with a simple web browser:

### Exploring the root object

Request: \
HTTP-GET on http://localhost:2121

Response:
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

### Exploring /gen

Request: \
HTTP-GET on http://localhost:2121/gen

Response:
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

### Exploring /gen/rand

Request: \
HTTP-GET on http://localhost:2121/gen/rand

Response:
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

### Read process value /gen/rand

Request: \
HTTP-GET on http://localhost:2121/gen/rand/~pv

Response:
```
{
    "ts": 1594594800000,
    "v": 0.2065826619136986,
    "s": 0
}
```

### Read history /hist/sin_2pix_10/1s

Time range begin: 1.1.2019 00:00:00 UTC \
Time range end: 1.1.2019 00:00:20 UTC

Request: \
HTTP-GET on http://localhost:2121/hist/sin_2pix_10/1s/~hist?begin=1546300800000&end=1546300820000

Response:
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

## Secure access via HTTPS

The VEAP demo server enables encrypted access via HTTPS, so that data can also be securely exchanged via insecure networks (e.g. Internet). A HTTPS connection can be established via port 2122 (changeable with the command line option `-porttls`). The required certificates can be specified or automatically generated by the VEAP demo server at the first start.

Required certificate files for the server:

File name   | Function
------------|---------
svrcert.pem | Certificate of the server
svrcert.key | Private key of the server (This must be kept secret.)

If the above mentioned certificate files do not exist in the working directory of the VEAP demo server, two certificates will be created automatically. The validity is set to 10 years:

File name   | Function
------------|---------
cacert.pem  | Certificate of the Certification Authority (CA)
cacert.key  | Private key of the certification authority (This must be kept secret.)
svrcert.pem | Certificate of the server
svrcert.key | Private key of the server (This must be kept secret.)

For secure access, only the generated certificate of the certification authority (`cacert.pem`) must be made known to the HTTPS clients *via a secure channel*. The certificate can be installed e.g. in the operating system or in the web browser. The private keys must never be distributed.

Encrypted access is also possible using different programming languages.

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
