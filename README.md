# Overview

This interface layer handles the communication between the Kafka and its clients.
The provider end of the relation provides the Kafka service.
The other end requires the existence of the provider to function.


# Usage

## Provides

Charms providing the Apache Kafka service can make use of the provides interface.

This interface layer will set the following states, as appropriate:

  * `{relation_name}.connected`   The relation to a client has been
    established, though the service may not be available yet. At this point the
    provider should broadcast the connection properties using:
      * `send_configuration(self, port)`

  * `{relation_name}.available`   The connection to the client is now available and correctly setup.


As soon an client get connected the Apache Kafka charm provides the connection details (port):

```python
@when('kafka.connected')
@when_not('kafka.available')
def waiting_availuable_kafka_client(kafka):
    kafka.send_configuration(hookenv.config()['source_port'])
    hookenv.status_set('waiting', 'Waiting for a client to become available')
```

## Requires

A client makes use of the requires part of the interface to connect to Apache Kafka.

This interface layer will set the following states, as appropriate:

  * `{relation_name}.connected` The charm has connected to the Kafka. 
    At this point the requires intrface waits for connection details (port).

  * `{relation_name}.available` The connection has been established, and the client charm
    can get the connection details via the following calls:
      * `get_kafka_ip()`
      * `get_kafka_port()`
    In case of an error a generic exception is thrown.

Example:

```python
@when('kafka.connected')
@when_not('kafka.available')
def waiting_for_kafka_available(kafka):
    hookenv.status_set('waiting', 'Waiting for availability of Kafka')


@when('kafka.available')
@when_not('service.started')
def configure_kafka(kafka):
    port = kafka.get_kafka_port()
    ip = kafka.get_kafka_ip()
```


# Contact Information

- <bigdata@lists.ubuntu.com>
