## Description

This repository contains the current work in progress towards an official
Logstash image from Elastic Inc.

Experimentation and feedback are warmly encouraged, but please don't use this
image in a production context. It's still under heavy development, and could
change significantly before official release.

## Quickstart
Clone this repository and run `docker-compose up`. You will then have an
example instance of Logstash running, with supporting containers running
[Elasticsearch][elasticsearch-docker] for storing events, and
[Kibana][kibana-docker] for visualizing them.

The example container creates Logstash [heartbeat][heartbeat-input]
events every few seconds.  To see them in Kibana, point a browser at
`http://localhost:5601`, and log in as the `elastic` user, with the
password `changeme`.

[elasticsearch-docker]: https://github.com/elastic/elasticsearch-docker
[kibana-docker]: https://github.com/elastic/kibana-docker
[heartbeat-input]: https://www.elastic.co/guide/en/logstash/5.0/plugins-inputs-heartbeat.html

## Image tags and hosting

The image is hosted in Elastic's own docker registry: `docker.elastic.co/logstash`

Available tags:

- 5.0.0-beta1
- 5.0.0-rc1
- latest -> 5.0.0-rc1

## Using the image

To save some keystrokes, first set:

``` shell
export ELASTIC_REG=docker.elastic.co/logstash
export LOGSTASH_VERSION=5.0.0-rc1
export LOGSTASH_IMAGE=$ELASTIC_REG/logstash:$LOGSTASH_VERSION
```

### Configuration

Logstash differentiates between two types of configuration:
['Settings' and 'Pipeline Configuration'][conf-types].

#### Pipeline Configuration

It's essential to place your pipeline configuration where it can be
found by Logstash. By default, the container will look in
`/logstash/pipeline/` for pipeline config files.

In this example we use a bind-mounted volume to provide the configs:

``` shell
docker run -it -v /my/logstash/configs/:/logstash/pipeline/ $LOGSTASH_IMAGE
```

Every file in the host directory `/my/logstash/configs/` will then be parsed
by Logstash as pipeline config.

If you don't provide configuration to Logstash, it will run with a minimal
config that simply echoes `stdin` to `stdout`, through the `rubydebug`
codec. Like this:

```
$ docker run -it $ELASTIC_REG/logstash:5.0.0-rc1
The stdin plugin is now waiting for input:
[2016-10-05T05:05:20,297][INFO ][logstash.pipeline        ] Starting pipeline {"id"=>"main", "pipeline.workers"=>8, "pipeline.batch.size"=>125, "pipeline.batch.delay"=>5, "pipeline.max_inflight"=>1000}
[2016-10-05T05:05:20,301][INFO ][logstash.pipeline        ] Pipeline main started
[2016-10-05T05:05:20,325][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9600}
Hi! I am typing this.
{
    "@timestamp" => 2016-10-05T05:07:01.141Z,
      "@version" => "1",
          "host" => "2f773d5a7f29",
       "message" => "Hi! I am typing this."
}
```
This configuration is baked into the image at `/logstash/pipeline/logstash.conf`.
If this is the behaviour that you are observing, ensure that your
pipeline configuration is getting picked up correctly, and that you are replacing
either `logstash.conf` or the entire `pipeline` directory.


#### Settings Files

We can provide settings files through bind-mounts too. Logstash expects to
find them at `/logstash/config/`.

It's possible to provide an entire directory, containing all needed files:

```
docker run -it -v /my/logstash/settings/:/logstash/config/ $LOGSTASH_IMAGE
```

...or just a single file

```
docker run -it -v ~/logstash.yml:/logstash/config/logstash.yml $LOGSTASH_IMAGE
```

[conf-types]: https://www.elastic.co/guide/en/logstash/5.0/config-setting-files.html


### Logging

By default, Logstash logs go to standard output.

### Operational notes

1. Use the env var `LS_JAVA_OPTS` to set heap size, e.g. to use 16GB
   use `-e LS_JAVA_OPTS="-Xms16g -Xmx16g"` with `docker run`. It is
   also recommended to set a memory limit for the container.

2. It is recommended to pin your deployments to a specific version of
   the Logstash Docker image, especially if you are using an
   orchestration framework like Kubernetes, Amazon ECS or Docker
   Swarm.

## Supported Docker versions

The images have been tested on Docker 1.12.1.

## Contributing, issues and testing

This image is built on top of [elasticsearch-alpine-base][es-base]
which is derived from [alpine:latest][alpine].

[es-base]: https://github.com/elastic/elasticsearch-alpine-base
[alpine]: https://hub.docker.com/_/alpine/
