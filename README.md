[German translation of the specification. 🇩🇪](README_de.md)

# Very Easy Automation Protocol (VEAP)

In the following, a TCP/IP-based communication protocol for automation is presented that is **easy to understand** and **easy to implement**, yet also meets advanced requirements such as **modelling of data models** or **security requirements**.

The protocol can be used, for example, in the field of building automation or for the Internet of Things. It can connect devices with each other, with higher level systems (e.g. control centers, production data acquisition) or with graphical user interfaces.

For the impatient, [protocol examples can be found below](#examples).

### VEAP Implementations

The [VEAP demo server](veap-demo-server/README.md) implements the protocol as an example. It should serve as a reference implementation and for testing VEAP clients.

Another implementation is the [CCU-Jack](https://github.com/mdzio/ccu-jack) for the HomeMatic home automation system from eQ-3.

### Reason for a new protocol

 In the field of automation and the Internet of Things, there is already a multitude of manufacturer-independent protocols (including [MQTT](https://de.wikipedia.org/wiki/MQTT), [CoAP](https://de.wikipedia.org/wiki/Constrained_Application_Protocol), [OPC-UA](https://de.wikipedia.org/wiki/OPC_Unified_Architecture)). However, an investigation has shown that none of these communication protocols has all the properties required above. For example, they are functionally limited (e.g. MQTT) or complex to implement (e.g. CoAP, OPC-UA). 

**Of course, there are other requirements that are only met by one of the above protocols (MQTT, CoAP or OPC-UA) and not by VEAP. Each protocol has its own unique features.**

## Protocol Basics

In order to meet the requirements, the protocol must be based on existing and established technologies. Web technologies are particularly suitable for TCP/IP-based communication. VEAP therefore makes use of the following standards:
* Transport protocol: [Hypertext Transfer Protocol (HTTP)](https://de.wikipedia.org/wiki/Hypertext_Transfer_Protocol)
  * Differentiation of different protocol services (e.g. read/write data point) by HTTP verbs (e.g. GET, PUT)
* Addressing of objects/data points: [Uniform Resource Locator (URL)](https://de.wikipedia.org/wiki/Uniform_Resource_Locator)
* Further characteristics of [Representational State Transfer (REST)](https://de.wikipedia.org/wiki/Representational_State_Transfer)
* Mapping of object relationships: [Based on HAL (Hypertext Appplication Language)](http://stateless.co/hal_specification.html)
* Serialization (and data types): [JavaScript Object Notation (JSON)](https://de.wikipedia.org/wiki/JavaScript_Object_Notation)
* Authentication: [HTTP Authentication](https://de.wikipedia.org/wiki/HTTP-Authentifizierung)
* Transport encryption: [HTTPS](https://de.wikipedia.org/wiki/Hypertext_Transfer_Protocol_Secure)

### Advantages

Many advantages result from the above choice of basic technologies:
* Simple comprehensibility
  * The protocol is also understandable and applicable for non-programmers.
  * The protocol messages are readable by humans.
* Easy to implement
  * The use of basic services of the protocol (e.g. reading and writing data points) is possible in most programming languages with only a few lines of source code without integration of an external library.
  * The protocol services can already be used with tools that already exist for most operating systems (e.g. wget, curl).
  * No special infrastructure is required (e.g. a broker at MQTT). This reduces setup and maintenance costs.
* Security
  * Users can be authenticated without significant additional effort. In addition, state-of-the-art encryption with certificates is available.

## VEAP server

A VEAP server is a simple HTTP and/or HTTPS server on the surface. The following network ports should be used for dedicated VEAP servers:

Port | Protocol
-----|----------
2121 | HTTP
2122 | HTTPS

For HTTP/S servers that also deliver other documents (e.g. HTML pages), the VEAP root object should be found via the path `/veap`.

The `Content-Type` of the exchanged VEAP requests and responses should be `application/json`.

## Data model

In the field of automation, a protocol must be able to read measured values (e.g. temperatures) from sensors and set states of actuators (e.g. heating valves). In general, the term data point is used here for a sensor or actuator value. The data points should also be explorable. The communication partners therefore do not need any prior knowledge of which data points exist at all.

A further level of information is relationships between data points and objects (e.g. the device containing this data point) or relationships between objects (e.g. a device is connected to an automation center). Each object should be able to have any other properties (e.g. room name, measuring range start/end). A data point is itself a specialized object that offers additional services (reading and writing values).

### Objects

Each object is addressed via a unique path in the VEAP server (e.g. ```/bidcos-rf/abc012345/1/STATE```). It can have any application-dependent properties and relationships to other objects and offer various services (e.g. read/write _process value).

Address components that begin with a tilde (~) are keywords reserved for the VEAP protocol and must not be used in object addresses. This also applies to object property identifiers.

#### Special objects

The ```/~vendor``` object can be used to explore the properties of the VEAP server. The following properties are defined:
* serverName (String, optional)
* serverVersion (String, optional)
* serverDescription (String, optional)
* vendorName (String, optional)
* veapVersion (VEAP protocol version, String, currently "1", optional)

Operating data of the server (e.g. access statistics) should be displayed below the vendor object as process values. Example: ```/~vendor/statistics/requests/~pv```

## Services

Each object can offer different services. The services are mapped to an HTTP verb (e.g. GET) and/or to a special address suffix (e.g. ```/~pv```). The data structures to be transferred are encoded using the [JavaScript Object Notation (JSON)](https://de.wikipedia.org/wiki/JavaScript_Object_Notation). The default character encoding is [UTF-8](https://de.wikipedia.org/wiki/UTF-8).

### Read data point

The HTTP method GET and the address extension ```/~pv``` (Process Value) request the current process value from an object. A process value consists of up to three components: Value (v), time stamp (ts) and status (s). Timestamp and status are optional. A corresponding JSON object then looks as follows, for example:
```
{
  "v": 123.456,
  "ts": 1483228800000,
  "s": 0
}
```

The value (v) is any JSON data type. Complex data structures are permitted, but should be avoided for reasons of interoperability. 

The time stamp (ts) indicates the original read time of the sensor. It is an integer and indicates the number of milliseconds since 1.1.1970 UTC. The number 1483228800000 therefore corresponds to the time 1.1.2017 01:00:00 CET. The time stamp is optional.

The value status is an integer number from three ranges. The number used within a range is application-specific (e.g. 100=battery empty, 200=sensor defective, 201=underrange, 202=overrange). The status is optional. If the status has not been specified, it is implicitly assumed as _GOOD_ (value 0).

Range values for the status (s):

Number | Symbol | Description
------------|-------------|-------------
0 to 99 | _GOOD_ | The measured value can be trusted.
100 to 199 | _UNCERTAIN_ | The measured value is questionable.
200 to 299 | _BAD_ | The measured value is bad and should not be processed further.

Variable properties of an object are mapped as process values. Static properties are determined via [Read object properties] (#Object data-point properties-read).

### Write data point

The HTTP method PUT (a server can also accept the POST method) and the address suffix ```/~pv``` write the current process value from a data point. The process value is structured [as described above](#data point-read).

### Read object/data point properties

The HTTP method GET and the unchanged object address are used to read application-specific properties ([and also the relationships](#explore-object-relationships)) of an object. In contrast to the process value, these properties rarely change. The identifiers of the properties can be freely selected, but must not begin with a tilde (~).

The property `title` can contain a readable identifier for the object. A description can be stored in the `description` property.

Example response:
```
{
  "title": "HWR ceiling lamp",
  "description": "4xGU10,20W",
  "address": "BidCos-RF.ABC01232:3.STATE",
  "iseId": 1234
}
```

For process values, the following additional properties can be offered, depending on availability: `minimum`, `maximum` and `unit`.

Example response:
```
{
  "title": "Brightness outside",
  "description": "Brightness sensor weather station",
  "address": "BidCos-RF.ABC01232:3.LUX",
  "iseId": 1234,
  "minimum": 0,
  "maximum": 200000,
  "unit": "Lux"
}
```

### Explore object relationships

An object or data point can itself contain other objects, similar to a folder in a file system, or point to other objects, similar to a reference or link in a file system. If, as described in the [previous section](#read-objectdata-point-properties), the object properties are read, the referenced objects are also listed.

With the help of the references, a VEAP client can **customize** the complete data inventory of a VEAP server. It needs **no prior knowledge** about which data points actually exist.

Referenced objects are specified in the reserved object property ```~links```. This is a JSON array whose elements are link objects. Link objects have at least the properties ```rel``` (relation) and ```href``` (hypertext reference). ```rel``` contains the relationship type and ```href``` contains the absolute or relative address of the referenced object. Optionally, a link object can also have the ```title``` property, which contains a readable name for this reference.

The following relationship types are defined in advance:

  Type       | Type of referenced object
:-----------:|:-------------------------------
interface    | Communication interface
device       | Device
channel      | Channel
datapoint    | Data point
room         | Room
function     | Function
service      | Service; The destination address must end either in ```/~pv``` or ```/~hist```.
vendor       | Information about the server

Each VEAP server can additionally define its own relationship types and also do without the predefined ones.

Example of the object ```/interface-rf/sensor1```:
```
{
  "~links": [
    { "rel": "datapoint", "href": "temperature", "title": "Data point temperature" },
    { "rel": "datapoint", "href": "humidity", "title": "Data point humidity" },
    { "rel": "room", "href": "/rooms/kitchen", "title": "Location of the Sensor" }
    { "rel": "interface", "href": "..", "title": "Interface" }
  ]
}
```

Example of object ```/interface-rf/sensor1/temperature```:
```
{
  "~links": [
    { "rel": "service", "href": "~pv", "title": "Process Value" }
    { "rel": "service", "href": "~hist", "title": "History" }
    { "rel": "room", "href": "/rooms/kitchen", "title": "Location of the data point" }
    { "rel": "interface", "href": "../..", "title": "Interface" }
  ]
}
```

#### Hierarchy of objects

The objects should be structured hierarchically. This is mapped using the addressing path: The object ```/sensor1/temperature``` is part of the object ```/sensor1```. This is a composition.

The object types _communication interface_ → _device_ → _channel_ → _data point_ must form a hierarchy, whereby individual components are optional. 

### Signalling of errors

Service errors are signaled via the returned HTTP status. The status codes of the VEAP protocol used correspond in their meaning to the [HTTP status codes](https://de.wikipedia.org/wiki/HTTP-Statuscode).

HTTP Status| Meaning
----------:|:--------
   200 | OK
   201 | OK, object created
   400 | Invalid request (HTTP protocol error)
   401 | (User) authentication required
   403 | Access rights missing
   404 | Object/service not available
   422 | Request does not conform to VEAP protocol
   500 | Unexpected server error

For a simple VEAP client, it is sufficient to compare the HTTP status with 200. Any status not equal to 200 is then interpreted as an error.

### Read history of a data point (optional)

The history of a data point is requested via the HTTP method GET and the address suffix ```/~hist`` (History). The history shows the temporal progression of a data point value. The queried time range is specified via GET parameters.

The following GET parameters are defined:

Parameter name|meaning
-------------|---------
begin | start of time domain (milliseconds since 1.1.1970 UTC, time included)
end |end of time domain (milliseconds since 1.1.1970 UTC, time excluded)
limit |Maximum number of entries to be returned (optional)

```begin``` and ```end``` indicate the requested time range. The timestamps of the returned time series must be greater than or equal to ```begin``` and less than ```end```. If both parameters are missing, the time series of the last 24 hours is returned. 

With ```limit``` you can optionally limit the number of entries to be returned. The number of returned entries must always be less than or equal to ```limit```. If this parameter is not specified, the VEAP server decides on a possible limit.

Example request (data point **a**, 1.1.2018 to 2.1.2018 CET, max. the first 10 entries):
```
GET /a/~hist?begin=1514761200&end=1514847600&limit=10
```

The server responds with a JSON object consisting of the fields ```v``` (values), ```ts``` (timestamp) and optionally ```s``` (status). The fields are JSON arrays, they must all have the same length. The meaning of the elements is identical to [read a data point](#data point-read). The arrays must be sorted in ascending order. The entries are sorted in time before the limit is applied. 

Example response:
```
{
  "v": [123.456, 234.567, 345.678],
  "ts." [1514761200, 1514761201, 1514761202],
  "s": [0, 0, 0]
}
```

### Write history of a data point (optional)

The history of a data point is described using the HTTP method ```PUT``` (a server can also accept the POST method) and the address suffix ```/~hist```. The history is structured [as described above](#read-history-of-a-data-point-optional).

### Create and modify objects (optional)

With the HTTP method ```PUT``` on an object path the properties of an object can be changed. If an object does not exist on this path, it is created in the VEAP server. A JSON object must be passed which contains all properties to be set. If a VEAP object was successfully created, the HTTP status code ```201``` (Created) instead of ```200``` (OK) is returned.

Object relationships cannot be created or deleted with this service.

### Deleting objects (optional)

The HTTP method ```DELETE``` on an object path deletes the corresponding object. 

### Protocol extensions

The protocol version supported by a VEAP server can be retrieved from the `veapVersion` property of the `/~vendor` object. Currently only version "1" of the protocol exists.

The aim is to make protocol extensions so that they are downward compatible. Older VEAP clients should therefore be able to work with newer VEAP servers without changes.

## Examples

(In the requests and responses given below, the HTTP protocol is only shown in simplified form.)

### Minimum VEAP server with a flat hierarchy

The VEAP server described below offers the two data points A and B in a flat hierarchy.

#### Read data point

In the example, ```/a``` is the (arbitrarily structured) addressing of the data point and ``~pv`` (Process value) is the addressing of the _read/write process value_ service.

Request:
```
GET /a/~pv
```

Answer:

In the field "v" is the current value of the data point. The fields "ts" (time stamp; milliseconds since 1.1.1970 UTC) and "s" (status: 0=good) are optional.
```
HTTP 200 OK
{
  "v": 123.456,
  "ts": 1483228800000,
  "s": 0
}
```

#### Write data point

In the example ```/b``` is the addressing of the data point and ```~pv``` (Process value) is the addressing of the _read/write process value_ service. In the field "v" is the value of the data point to be set. The fields "ts" (timestamp) and "s" (status) are optional.

Request:
```
PUT /b/~pv
{
  "v": 123.456,
  "ts": 1483228800000,
  "s": 0
}
```
Answer:
```
HTTP 200 OK
```

#### Read data point properties

In the example, ```/a``` is the address of the object/data point to be queried.

Query:
```
GET /a
```

Answer:

Any device-dependent properties can be returned.
```
HTTP 200 OK
{
  "name": "HWR ceiling lamp",
  "description": "4xGU10,20W",
  "address": "BidCos-RF.ABC01232:3.STATE",
  "iseId": 1234,
  "rooms": "HWR,UG",
  "functions": "Light"
}
```

#### Read VEAP server properties

The special object ```/~vendor``` can be used to read properties of the VEAP server itself.

Request:
```
GET /~vendor
```

Answer:
```
HTTP 200 OK
{
  "serverName": "Minimal VEAP server",
  "serverVersion": "0.1.0",
  "serverDescription": "A minimal VEAP-Server for demonstration",
  "vendorName": "VEAP",
  "veapVersion": "1"
}
```

#### Explore data points/object relationships

In the example, ```/``` is the address of the parent object that can contain other objects/data points.

Query:
```
GET /
```

Answer:

The references to the child objects can be found in the property ```~links```. Optionally, properties of the queried object can also be returned (e.g. "name"). The relation type ````datapoints```` means that ``/a``` and ```/b`` are data points of the root object ```/``` (see also [object relations](#explore-object-relationships)). ```/~vendor``` is a reference to information about the VEAP server.

```
HTTP 200 OK
{
  "name": "Root directory",
  "~links": [
    { "rel": "datapoint", "href": "/a", "title": "Data point A" },
    { "rel": "datapoint", "href": "/b", "title": "data point B" },
    { "rel": "vendor", "href": "/~vendor", "title": "VEAP Server Information" }
  ]
}
```

## License

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International Public License](http://creativecommons.org/licenses/by-sa/4.0/).

[![License](https://i.creativecommons.org/l/by-sa/4.0/88x31.png)](https://creativecommons.org/licenses/by-sa/4.0/)
