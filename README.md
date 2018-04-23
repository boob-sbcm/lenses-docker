# Lenses™ for Apache Kafka

This is the official image of Landoop’s Lenses for Apache Kafka software.

Lenses is a Streaming Data Management Platform. It enhances Kafka with a web user interface and vital enterprise capabilities that enable engineering and data teams to query real time data, create and monitor Kafka topologies with rich integrations to other systems and gain operational awareness of their clusters.

Please visit our [website](https://www.landoop.com/) or the [documentation pages](https://lenses.stream/install_setup/docker/index.html) to learn more.


## The Docker Image

**Please check out
the [docker image documentation at lenses.stream](https://lenses.stream/install_setup/docker/index.html)
for the most recent docs and the complete set of features, settings and tweak
knobs.**


This image is aimed for our enterprise clients, though anyone with a free developer license may use it. Visit our [download page](https://www.landoop.com/downloads/) to get a free developer license or an enterprise trial.
Only Lenses is included in this docker. Our development environment image, which additionally includes Kafka, Connect, Schema Registry and our open-source Stream Reactor collection of connectors can be found as `landoop/kafka-lenses-dev`.

This image has to be run alongside a Kafka cluster.


## How to run

In the current iteration `landoop/lenses` uses environment variables as the primary means for configuration and alternatively configuration files.

### Setup with environment variables

For any lenses configuration option, set an environment variable by converting the option name to uppercase and with dots replaced by underscores. As an example `lenses.port` should be converted to `LENSES_PORT`. Optionally settings may be mount as volumes under `/mnt/settings` or `/mnt/secrets`. As an example you could set —file— volume `/mnt/settings/LENSES_PORT` with the port number as the content of the file.

A brief example of a docker-compose file to setup Lenses, would be:

```yaml
version: '2'
services:
  lenses:
    image: landoop/lenses:2.0
    environment:
      LENSES_PORT: 9991
      LENSES_KAFKA_BROKERS: "PLAINTEXT://broker.1.url:9092,PLAINTEXT://broker.2.url:9092"
      LENSES_ZOOKEEPER_HOSTS: |
        [
          {url:"zookeeper.1.url:2181", jmx:"zookeeper.1.url:9585"},
          {url:"zookeeper.2.url:2181", jmx:"zookeeper.2.url:9585"}
        ]
      LENSES_SCHEMA_REGISTRY_URLS: |
        [
          {url:"http://schema.registry.1.url:8081",jmx:"schema.registry.1.url:9582"},
          {url:"http://schema.registry.2.url:8081",jmx:"schema.registry.2.url:9582"}
        ]
      LENSES_CONNECT_CLUSTERS: |
        [
          {
            name:"data_science",
            urls: [
              {url:"http://connect.worker.1.url:8083",jmx:"connect.worker.1.url:9584"},
              {url:"http://connect.worker.2.url:8083",jmx:"connect.worker.2.url:9584"}
            ],
            statuses:"connect-statuses-cluster-a",
            configs:"connect-configs-cluster-a",
            offsets:"connect-offsets-cluster-a"
          }
        ]
      LENSES_SECURITY_MODE: BASIC
      # Secrets can also be passed as files. Check _examples/
      LENSES_SECURITY_GROUPS: |
        [
          {"name": "adminGroup", "roles": ["admin", "write", "read"]},
          {"name": "readGroup",  "roles": ["read"]}
        ]
      LENSES_SECURITY_USERS: |
        [
          {"username": "admin", "password": "admin", "displayname": "Lenses Admin", "groups": ["adminGroup"]},
          {"username": "read", "password": "read", "displayname": "Read Only", "groups": ["readGroup"]}
        ]
    ports:
      - 9991:9991
      - 9102:9102
    volumes:
      - ./license.json:/license.json
    network_mode: host
```

The docker image has two volumes where data are saved: `/data/log` for logs and `/data/kafka-streams-state` for storing the state of Lenses SQL processors. Depending on your queries and the topics volume, the latter can become pretty large. You should monitor space and plan for adequate capacity. Maintaining the streams state directory between Lenses restarts is mandatory for the SQL processors to be able to continue from where they left.

The container starts with root privileges and drops to `nobody:nogroup` (`65534:65534`) before running Lenses. If you start the image as a custom `user:group`, it falls under your responsibility to make sure that the two volumes are writeable by the custom `user:group`.

### Setup with configuration files

Lenses software configuration is driven by two files: `lenses.conf` and `security.conf`. In the docker image we create them automatically from environment variables
but it is possible to set directly these files instead.

Create your configuration files according to the [documentation](http://lenses.stream/install_setup/configuration/lenses-config.html) and mount them
under `/mnt/settings` and `/mnt/secrets` respectively —i.e `/mnt/settings/lenses.conf` and `/mnt/secrets/security.conf`. You can set either one or both together. Please for `lenses.conf`
omit the settings `lenses.secret.file` and `lenses.license.file`. If by any chance you set them, you have to make sure lenses can find the files
described in these settings.

### The license file

Lenses require a license file in order to start. It may be passed to the container via three methods:

- As a file, mounted at /license.json or /mnt/secrets/license.json (e.g `-v /path/to/license.json:/license.json`)
- As the contents of the environment variable LICENSE (e.g `-e LICENSE="$(cat license.json)"`)
- As a downloadable URL via LICENSE_URL (e.g `-e LICENSE_URL="https://license.url/"`)

---

For more information, please visit our [documentation](https://www.landoop.com/docs/lenses/). Enterprise customers may use the support channels made available to them. Developer Edition users are encouraged to visit our [gitter chat](https://gitter.im/Landoop/support). We are always happy to help and hear from you.

With respect,

The Lenses Team.

---

Copyright 2017-2018, Landoop LTD
