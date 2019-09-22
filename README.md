# Fork
This repo was forked from https://github.com/deviantony/docker-elk.
The repo was simplified to manage only Elasticsearch and Kibana

## Requirements

### Host setup

* [Docker Engine](https://docs.docker.com/install/) version **17.05+**
* [Docker Compose](https://docs.docker.com/compose/install/) version **1.12.0+**
* 1.5 GB of RAM

By default, the stack exposes the following ports:
* 9200: Elasticsearch HTTP
* 9300: Elasticsearch TCP transport
* 5601: Kibana

> :information_source: Elasticsearch's [bootstrap checks][booststap-checks] were purposely disabled to facilitate the
> setup of the Elastic stack in development environments. For production setups, we recommend users to set up their host
> according to the instructions from the Elasticsearch documentation: [Important System Configuration][es-sys-config].

### SELinux

On distributions which have SELinux enabled out-of-the-box you will need to either re-context the files or set SELinux
into Permissive mode in order for docker-elk to start properly. For example on Redhat and CentOS, the following will
apply the proper context:

```console
$ chcon -R system_u:object_r:admin_home_t:s0 docker-elk/
```

## Initial setup

### Setting up user authentication

> :information_source: Refer to [How to disable paid features](#how-to-disable-paid-features) to disable authentication.

The stack is pre-configured with the following **privileged** bootstrap user:

* user: *elastic*
* password: *changeme*

Although all stack components work out-of-the-box with this user, we strongly recommend using the unprivileged [built-in
users][builtin-users] instead for increased security. Passwords for these users must be initialized:

```console
$ docker-compose exec -T elasticsearch bin/elasticsearch-setup-passwords auto --batch
```

Passwords for all 6 built-in users will be randomly generated. Take note of them and replace the `elastic` username with
`kibana` inside the Kibana configuration files respectively. See the
[Configuration](#configuration) section below.

Restart Kibana to apply the passwords you just wrote to the configuration files.

```console
$ docker-compose restart kibana
```

> :information_source: Learn more about the security of the Elastic stack at [Tutorial: Getting started with
> security][sec-tutorial].

## Extensibility

### How to add plugins

To add plugins to any ELK component you have to:

1. Add a `RUN` statement to the corresponding `Dockerfile` (eg. `RUN logstash-plugin install logstash-filter-json`)
2. Add the associated plugin code configuration to the service configuration (eg. Logstash input/output)
3. Rebuild the images using the `docker-compose build` command

### Swarm mode

Experimental support for Docker [Swarm mode][swarm-mode] is provided in the form of a `docker-stack.yml` file, which can
be deployed in an existing Swarm cluster using the following command:

```console
$ docker stack deploy -c docker-stack.yml elk
```

If all components get deployed without any error, the following command will show 3 running services:

```console
$ docker stack services elk
```

> :information_source: To scale Elasticsearch in Swarm mode, configure *zen* to use the DNS name `tasks.elasticsearch`
instead of `elasticsearch`.

## Head plugin


> github: https://github.com/mobz/elasticsearch-head
> chrome extension: https://chrome.google.com/webstore/detail/elasticsearch-head/ffmkiejjmecolpfloofpjologoblkegm

## for more information see:

https://github.com/deviantony/docker-elk/blob/master/README.md