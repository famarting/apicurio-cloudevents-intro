+++
title = "Cloud Events Intro"
outputs = ["Reveal"]
"reveal_hugo.theme" = "blood"
"reveal_hugo.highlight_theme" = "atom-one-dark"
+++

## [Cloud Events](https://cloudevents.io/) Introduction

---

CNCF project

Started by Serverless WG

Related to cloud services and serverless world: AWS lambda, knative,...

---

Initiative to standardize a common event format in order to achieve interoperability across services, platforms and systems.

---

The project provides specs for [events](https://github.com/cloudevents/spec/blob/v1.0/spec.md), protocol bindings and events format, as well as some SDKs.

---

#### Events what?

---

JSON formatted event
```
{
    "specversion" : "1.0",
    "type" : "com.example.someevent",
    "source" : "/mycontext",
    "id" : "A234-1234-1234",
    "datacontenttype" : "application/json",
    "dataschema": "http://example.com/registry/someevent/v2",
    "data" : {
        "text": "example"
    }
}
```

---

Binary mode message
```
POST /someresource HTTP/1.1
ce-specversion: 1.0
ce-type: com.example.someevent
ce-id: 1234-1234-1234
ce-source: /mycontext/subcontext
Content-Type: application/json; charset=utf-8

{
    "text": "example"
}
```

---

Structured mode message
```
HTTP/1.1 200 OK
Content-Type: application/cloudevents+json; charset=utf-8
Content-Length: nnnn
{
    "specversion" : "1.0",
    "type" : "com.example.someevent",
    "source" : "/mycontext",
    "id" : "A234-1234-1234",
    "time" : "2018-04-05T17:31:00Z",
    "comexampleextension1" : "value",
    "datacontenttype" : "application/json",
    "data" : {
        "text": "example"
    }
}
```

---

#### Yes, it says message not event !!!

---

#### Quick terminology: Message

---

Events are transported from a source to a destination via **messages**.

- A **structured-mode message** is one where the event is fully encoded using a stand-alone event format and stored in the message body.

- A **binary-mode message** is one where the event data is stored in the message body, and event attributes are stored as part of message meta-data.

---

### Cloud Events Specs

---

* **attributes**: key-value pairs (type, source, id, ...)
* **extensions**: concept to add more attributes related to specific features(tracing, ordered sequences, ...)

---

* **event format**: JSON or AVRO, used for structured messages

---

* **protocol bindings**: protocol bindings (http, kafka, mqtt, amqp, ...)

---

#### Required Attributes

* id
* source
* specversion
* type

---

#### Event Data

the event payload
* encoded in the format specified in attribute `datacontenttype`
* follows the schema specified in attribute `dataschema`

---

#### Dataschema

Optional Attribute
```
{
    "specversion" : "1.0",
    "type" : "com.example.someevent",
    "source" : "/mycontext",
    "id" : "A234-1234-1234",
    "time" : "2018-04-05T17:31:00Z",
    "comexampleextension1" : "value",
    "datacontenttype" : "application/json",
    "dataschema": "http://example.com/registry/someevent/v2",
    "data" : {
        "text": "example"
    }
}
```

---

#### Schema registry

Almost perfect fit for Apicurio Registry

{{< highlight java >}}
POST ​/schemagroups​/{group-id}​/schemas​/{schema-id}
GET /schemagroups/{group-id}/schemas/{schema-id}/versions/{version-number}
{{< /highlight >}}

---

#### SDKs

{{< highlight java >}}
CloudEvent ce = CloudEventBuilder.v1()
    .withId(UUID.randomUUID().toString())
    .withType("com.example.someevent")
    .withSource(URI.create("/mysource"))
    .withData("application/json", bytes)
    .build();
{{< /highlight >}}

---

#### SDKs

* Golang, Javascript, Java, ...
* easy to not use them, as spec is simple

---

### Apicurio Registry and CloudEvents

---

Event-driven systems need a schema registry

---

It doesn't matter if they use kafka or if it's a serverless system using cloud events over http

---

#### Proofs

* CloudEvents `dataschema` attribute
* CloudEvents Schema registry API spec
* Knative Event Registry `schema` field

---

```
apiVersion: eventing.knative.dev/v1
kind: EventType
metadata:
  name: dev.knative.source.github.push-34cnb
  namespace: default
spec:
  type: dev.knative.source.github.push
  source: https://github.com/knative/eventing
  schema: http://example.com/registry/dev.knative.source.github.push/v4
  description:
  broker: default
```

---

#### Extending Apicurio Registry usecases with CloudEvents

---

#### Implement CloudEvents schema registry API

---

* Missing schemagroup concept
* schema-id -> apicurio's artifactId
* version-number -> apicurio's artifact version

---

#### Create serdes libraries for CloudEvents

---

Maybe not serdes but schema validation tools

---

Using the SDKs create libraries to easily validate and serialize/deserialize CloudEvents **data**

---

Use CloudEvents attributes such as `type` or `dataschema` to query the schema

---

`type` attribute can indicate versioning

```
com.example.someevent/v1beta1
```

this will be helpful to query schemas when `dataschema` is not available

---

Integration with quarkus quarkus-funqy-knative-events

Will require modifications on quarkus funqy extension

---

Or also implement the library allowing it to be used explicitly in user code

---

### Go to the code

---

### Open Questions?

---

is this worth it? does it make sense?

---

would make sense to implement something is this way but generic enough to be used for whatever protocol?

---

Implement registry clients and serdes utilities in other languages?

---

if implemented, should this share some codebase with existing kafka-serde library?

---