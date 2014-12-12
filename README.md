# Logstash Dockerfile

This is a highly configurable [logstash][7] (1.4.2) image running [Elasticsearch][8] (1.1.1) and [Kibana][9] (3.0.1).

## How to use this image

To run the image, you have to first decide on one of three Elasticsearch configurations:

 * Use the embedded Elasticsearch server
 * Use a linked container running Elasticsearch
 * Use an external Elasticsearch server

### Embedded Elasticsearch server

By default, an example [logstash.conf][2] will be downloaded using `wget` and used in your container.

    $ docker run -d \
      -p 9292:9292 \
      -p 9200:9200 \
      ianblenke/docker-logstash

To use your own config file, set the `LOGSTASH_CONFIG_URL` environment variable using the `-e` flag as follows:

    $ docker run -d \
      -e LOGSTASH_CONFIG_URL=<your_logstash_config_url> \
      -p 9292:9292 \
      -p 9200:9200 \
      ianblenke/docker-logstash

### Linked container running Elasticsearch

If you want to link to container running Elasticsearch rather than use the embedded Elasticsearch server:

    $ docker run -d \
      -e LOGSTASH_CONFIG_URL=<your_logstash_config_url> \
      --link <your_es_container_name>:es \
      -p 9292:9292 \
      -p 9200:9200 \
      ianblenke/docker-logstash

To have the linked Elasticsearch container's `bind_host` and `port` automatically detected, you will need to create an `ES_HOST` and `ES_PORT` placeholder in the `elasticsearch` definition in your logstash config file. For example:

    output {
      elasticsearch {
        bind_host => "ES_HOST"
        port => "ES_PORT"
      }
    }

I have created an example [logstash_linked.conf][6] which includes the `ES_HOST` and `ES_PORT` placeholders to serve as an example.


### Alternative configuration methods

As an alternative to a `LOGSTASH_CONFIG_URL`, you may put the contenst of a `LOGSTASH_CONFIG_FILE` into the environment variable `LOGSTASH_CONFIG_CONTENTS`.

As this causes a bit of trouble with newlines given the format of the config file, it is also possible to turn a config file into a `LOGSTASH_CONFIG_ONELINE` format.

For example:

    input {
      stdin {
        codec => json
        tags => ["source:stdin"]
      }
      syslog {
        type => syslog
        port => 514
        tags => ["source:syslog"]
      }
    }
    output {
      stdout {
        codec => json
      }
      elasticsearch {
        embedded => false
        host => "172.17.42.1"
        port => 9200
        protocol => "http"
        cluster => "logstash"
        codec => "json"
        node_name => "thishost"
      }	
    }

can be represented as a series of "properties" like so:

    input.stdin.codec=json
    input.stdin.tags=["source:stdin"]
    input.syslog.type=syslog
    input.syslog.port=514
    input.syslog.tags=["source:syslog"]
    output.stdout.codec=json
    output.elasticsearch.embedded=false
    output.elasticsearch.host="172.17.42.1"
    output.elasticsearch.port=9200
    output.elasticsearch.protocol="http"
    output.elasticsearch.cluster="logstash"
    output.elasticsearch.codec="json"
    output.elasticsearch.node_name="thishost"

and then chained together with semicolons like this:

    $ docker run -d \
      -e LOGSTASH_CONFIG_ONELINE='input.stdin.codec=json;input.stdin.tags=["source:stdin"];input.syslog.type=syslog;input.syslog.port=514;input.syslog.tags=["source:syslog"];output.stdout.codec=json;output.elasticsearch.embedded=false;output.elasticsearch.host="172.17.42.1";output.elasticsearch.port=9200;output.elasticsearch.protocol="http";output.elasticsearch.cluster="logstash";output.elasticsearch.codec="json";output.elasticsearch.node_name="'`hostname`'"' \
      --link <your_es_container_name>:es \
      -p 9292:9292 \
      -p 9200:9200 \
      ianblenke/docker-logstash

Ugly? Perhaps. Functional? You bet.

### External Elasticsearch server

If you are using an external Elasticsearch server rather than the embedded server or a linked container, simply provide a configuration file with the Elasticsearch endpoints already configured:

    $ docker run -d \
      -e LOGSTASH_CONFIG_URL=<your_logstash_config_url> \
      -p 9292:9292 \
      -p 9200:9200 \
      ianblenke/docker-logstash

### Finally, verify the installation

You can now verify the logstash installation by visiting the prebuilt logstash dashboard:

    http://<your_container_ip>:9292/index.html#/dashboard/file/logstash.json

## Optional, build and run the image from source

If you prefer to build from source rather than use the [ianblenke/docker-logstash][1] trusted build published to the public Docker Registry, execute the following:

    $ git clone https://github.com/ianblenke/docker-logstash.git
    $ cd docker-logstash

If you are using Vagrant, start and provision a virtual machine using the provided Vagrantfile:

    $ vagrant up
    $ vagrant ssh
    $ cd /vagrant

From there, build and run a container using the newly created virtual machine:

    $ make build
    $ make <options> run

You can now verify the logstash installation by visiting the [prebuilt logstash dashboard][3] running in the newly created container.

## Acknowledgements

Special shoutout to @ehazlett's excellent post, [logstash and Kibana via Docker][4].

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

## License

This application is distributed under the [Apache License, Version 2.0][5].

[1]: https://registry.hub.docker.com/u/ianblenke/docker-logstash
[2]: https://gist.githubusercontent.com/ianblenke/8778567/raw/logstash.conf
[3]: http://192.168.33.10:9292/index.html#/dashboard/file/logstash.json
[4]: http://ehazlett.github.io/applications/2013/08/28/logstash-kibana/
[5]: http://www.apache.org/licenses/LICENSE-2.0
[6]: https://gist.githubusercontent.com/ianblenke/0b937485fa4a322ea9eb/raw/logstash_linked.conf
[7]: http://logstash.net
[8]: http://www.elasticsearch.org/overview/elasticsearch
[9]: http://www.elasticsearch.org/overview/kibana
