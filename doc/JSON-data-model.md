Data structure and format
=========================

<<**this is the draft version.  More discussion is needed.**>>

The IEEE1888-2014 defines the XML format to represent the IEEE1888 data model.
This memo defines another two formats.

    1) JSON format
    2) RESTful interface

## JSON format

The JSON format is used for both a request and response of the POST method.
And, it is also used for a response of the GET method.

### Unsupported functions

The JSON format is trying to support all of functions defined by the XML format.
TBD.

### Notation

### Definition

    ~~~~
    {
        "fiap" : {
            <version spec>, 
            <method spec>
        }
    }

    <method spec> :=
        <data request spec> |
        <data response spec> |
        <query request spec> |
        <query response spec>

    <data request spec> :=
        "dataRQ" : { <point spec>, ... }

    <data response spec> :=
        "dataRS" : { <response status spec> }

    <query request spec> :=
        "queryRQ" : <query spec>

    <query response spec> :=
        "queryRS" : {
            <response status message>,
            <query request spec>,
            <point spec>, ...
        }

    <point spec> :=
        "<point id>" : [ <value spec>, ... ]

    <value spec> := {
        "time" : <time spec>,
        "value" : <value>
    }

    <query spec> := {
        "type" : <query type spec>,
        "key" : [ <key spec>, ... ],
        <uuid spec>,
        <acceptableSize spec>,
        <cursor spec>,
        <ttl spec>,
        <callbackData spec>,
        <callbackControl spec>,
    }

    <query type spec> := "storage" | "stream"

    <key spec> := {
        "<point id>" : { <attribute spec>, ... }
    }

    <attribute spec> :=
        "attrName" : <attribute name>,
        <condition spec>

    <condition spec> :=
        "eq" : <value> |
        "ne" : <value> |
        "lt" : <value> |
        "gt" : <value> |
        "lteq" : <value> |
        "gteq" : <value> |
        "select" : "maximum" |
        "select" : "minimum" |
        "state" : "changed"

    <uuid spec> := "uuid" : <uuid>

    <acceptableSize spec> := "acceptableSize" : <acceptable size>,

    <cursor spec> := "cursor" : <cursor>

    <ttl spec> := "ttl" : <ttl>

    <callbackData spec> := "callbackData" : <callback data point>

    <callbackControl spec> := "callbackControl" : <callback control point>
    ~~~~

"version spec" specifies the version of the JSON format.
All components supporting this specification must set "20140401".

"method spec" specifies the method of the JSON format.
one of four entries must be specified.

"query request spec" in "query response spec" is optional.

"time spec" should conform to the ISO8601 date and time string.
A timezone indicator should be used for large scale interoperability.
For interoperability, this specification recommends to use the following format.

    ~~~~
    2014-11-21T07:54:03+0900
    ~~~~

The format of "point id" must confom to the IEEE1888 specification.

    ~~~~
    "http://fiap.example.org/test/home/light"
    ~~~~

"point spec" in "query response spec" is valid for the FETCH protocol.
    TBC.

"query type spec" must be either "storage" or "stream".

"attribute spec" is optional.

TBD: if you want to specify an "OR" condition for a single point id.
you need to specify two keys.
e.g. getting both a maximum value and a minimum value.

    ~~~~
    "key" : [
        { "a": { "attrName":"value", "select":"maximum" } },
        { "a": { "attrName":"value", "select":"minimum" } }
    ]
    ~~~~

"condition spec" must be formed by some of the entries.
each etnry must be used at once.
"state" : "changed" is usually used to initiate the TRAP protocol.

"value" is a string representing the value.
it must be quoted by a double quotation and
a double quatation is not allowed in the content of the value.

"uuid spec" is optional.

"acceptableSize spec" is optional.

"cursor spec" is optional and valid to use when the query method is used.
    Should this description be put in the IEEE1888 base specification.  TBC.
the content of "cursor" depends on the implementation.

"ttl spec", "callbackData spec" and "callbackControl spec" are valid in the TRAP protocol.

### response message

if the request is acceptable and no error is found, the reponse message must be "200 OK".
other messages may be referred to the setion 10 of RFC2616.

### consice format for GET response

The JSON format can be simplified in the case of a reponse to the GET method.

    ~~~~
    {
        "fiap" : {
            <version spec>, 
            "queryRS" : {
                <response status spec>,
                <point spec>, ...
            }
        }
    }
    ~~~~

"response status spec" should be present.

### example: query

- a query request message

    ~~~~
    {
        "fiap" : {
            "version" : "20140401",
            "queryRQ" : {
                "type" : "storage",
                "acceptableSize" : "2",
                "key" : [
                    {
                        "http://example.org/light/p02" : {
                            "attrName" : "value",
                            "gteq" : "2002",
                            "lteq"  : "2003"
                        }
                    },
                    {
                        "http://example.org/light/p03" : {
                            "attrName" : "time",
                            "select" : "maximum"
                        }
                    }
                ]
            }
        }
    }
    ~~~~

- a reponse message of a query message

    ~~~~
    {
        "fiap": {
            "version": "20140401",
            "queryRS": {
                "query": {
                    "type": "storage"
                }
                "point": [
                    {
                        "http://fiap.tanu.org/test/alps01/temp": [
                            { "time": "2014-11-21T07:54:03+0900", "value": "26.0" },
                            { "time": "2014-11-21T07:55:00+0900", "value": "26.5" }
                        ],
                    },
                    {
                        "http://fiap.tanu.org/test/alps01/light": [
                            { "time": "2014-11-21T07:54:03+0900", "value": "1301" },
                            { "time": "2014-11-21T07:55:00+0900", "value": "1400" }
                        ]
                    }
                ]
            }
        }
    }
    ~~~~

### example: write

    ~~~~
    {
        "fiap" : {
            "version" : "20140401",
            "dataRQ" : {
                "http://example.org/light/p01" : [
                    { "time" : "2014-04-01T16:50:34+09:00", "value" : "1000" }
                ],
                "http://example.org/light/p02" : [
                    { "time" : "2014-04-01T16:50:34+09:00", "value" : "2001" },
                    { "time" : "2014-04-01T16:51:34+09:00", "value" : "2002" },
                    { "time" : "2014-04-01T16:52:34+09:00", "value" : "2003" },
                    { "time" : "2014-04-01T16:53:34+09:00", "value" : "2004" }
                ],
                "http://example.org/light/p03" : [
                    { "time" : "2014-04-01T16:50:34+09:00", "value" : "3001" },
                    { "time" : "2014-04-01T16:51:34+09:00", "value" : "3001" },
                    { "time" : "2014-04-01T16:52:34+09:00", "value" : "3002" }
                ]
            }
        }
    }
    ~~~~

## RESTful interface

It is not intended to support all protocols of IEEE1888.

Supported protocols are:

- FETCH protocol with multiple point IDs, but without any attributes.
- FETCH protocol with a single point IDs, multiple attributes.
- WRITE protocol with a single point IDs.

Other protocols and values should be considered later.

### element

    ~~~~
    k=<point id>
        it must be specified one or more.
    a=<0 or 1>
        0: time
        1: value
    s=<0 or 1>
        0: minimum
        1: maximum
        must be used with "a=".
    eq=<value>
        equal to.
        must be used with "a=".
    ne=<value>
        not equal to.
        must be used with "a=".
    lt=<value>
        less than.
        must be used with "a=".
    gt=<value>
        greater than.
        must be used with "a=".
    lteq=<value>
        less than or equal.
        must be used with "a=".
    gteq=<value>
        greater than or equal.
        must be used with "a=".
    i=<value>
        the maximum number of data to be acquired.
    ~~~~

the following elemets are for the write operation.

    ~~~~
    v=<value spec>
        <value spec> must be enclosed by a double quatation.  e.g. "26.5"
        the value must not allowed to include both "&" and any double quatations.
    t=<time spec>
    ~~~~

### query operation

- no conditions

It returns the latest data of each point ID.
Multiple point IDs can be specified.

    ~~~~
    k=<1st point id>[&k=<2nd point id>[...]]
    ~~~~

- with conditions

It returns the data matching with the conditions.
The conditions are applied to each key specified regardless of the order.

    ~~~~
    k=<point id>[&a=<v or t>][&s=<max or min>][&...]
    ~~~~

### write operation

It write data for the single key specified.
Only one point ID must be specified.
Multiple point IDs must not be specified.
Any condition must not be specified.

    ~~~~
    k=<point id>&v=<value spec>&t=<time spec>
    ~~~~

### example

RFC 2396 format should be used.

- fetch

    ~~~~
    GET http://server.example.org/?k=http://example.org/test/temperature&k=http://example.org/test/humidity
    ~~~~

- write

    ~~~~
    PUT http://server.example.org/?k=http://example.org/test/temperature&v="26.5"&t=2014-11-21T07:55:00+0900
    ~~~~

## method

*Right now, fiapc.py implementation does use GET for IEEE1888 WRITE protocol.*

### POST

it supports full IEEE1888 data and query method.
The format is either JSON or XML.

Content-Type must be text/json when JSON format is used.

if a requester sends a query by JSON format, the responder must respond data by JSON format or a HTTP error code with 415 (Unsupported Media Type). 

### GET

it supports simple IEEE1888 query method.
it allows to fetch a set of data of a single point id,
or to fetch a set of data of identical condition in each point ids.

a requester sends a request to the RESTful interface.
a responder responds data by JSON format.

typically, a requester uses this method to fetch a latest data against a point id.

### PUT

it supports the limited IEEE1888 write method.
it allows to write a single value for a single point id.
it dosen't allow to write multiple values for a single point id,
nor to write multiple point ids.

### Other method

TBD

