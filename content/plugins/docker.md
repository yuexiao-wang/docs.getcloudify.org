---
layout: bt_wiki
title: Docker Plugin
category: Plugins
draft: false
abstract: "Docker Plugin Documentation"
weight: 1400
---

# Description

The Docker Plugin pulls Docker images from Docker Hub and runs Docker containers on a host.


# Compatibility

 * Tested with Docker Engine version 1.0.
 * Docker-Py version 1.2.3.


# Plugin Requirements

* Python Versions:
  * 2.7.x

{{% gsNote title="Notes on Docker installation" %}}
Docker Engine needs to already be on your host.
  * The Docker plugin will not install Docker on your host.
  * You need to either use a host with Docker already installed, or you need to install Docker on it.
  * As part of the Docker installation, you should make sure that the user agent, such as ubuntu, is added to the docker group.
{{% /gsNote %}}


# Basic Usage

## Create a container

* Creates a container that can be started.
* Here, the plugin pulls images from the Docker Hub Registry, a private registry, or it may import an image from a tarball.
* This operations adds the container_id to the instance runtime_properties.

{{< gsHighlight  yaml  >}}

  some_container:
    type: cloudify.docker.Container
    properties:
      name: some_name
      image:
        repository: dockeruser/dockerrepo
    interfaces:
      cloudify.interfaces.lifecycle:
        create:
          implementation: docker.docker_plugin.tasks.create_container
          inputs:
            params:
              ports:
                - 8080
              stdin_open: true
              tty: true
              command: /bin/sleep 20
        start:
          implementation: docker.docker_plugin.tasks.start
          inputs:
            params:
              port_bindings:
                8080: 8080

{{< /gsHighlight >}}


# Features

The plugin is designed to follow the Docker Py Docker Python API library and not the Docker CLI. And so, it also differs from the Docker CLI in some respects. For example, `docker run` is split into `create` and `start`.

Many of the options exposed in the Docker-Py Python Docker API are available through the Cloudify Docker Plugin. Supplemental documentation is [here](https://github.com/docker/docker-py).

If there is a lack of description of certain parameters, more details can be found [here](https://docs.docker.com/reference/commandline/cli/).


# Types

## Node Types

### cloudify.docker.Container

**Derived From:** [cloudify.nodes.Root]({{< relref "blueprints/built-in-types.md" >}})

**Properties:**

* `image` A dictionary. It must have the `repository` key or the `src` key, or both. It may additionally have the `tag` key.
  * If `src` is provided, then it must point to a file or URL where the image's tarball is imported from.
    * If `repository` is also provided, then its value will be used as the name of the repository once the image is downloaded.
    * Otherwise, the plugin will name the repository after the Cloudify instance ID.
    * See [docker import](https://docs.docker.com/reference/commandline/cli/#import)
* Otherwise, just `repository` must be provided, and contain the name of the Docker image to pull.
  * If you pull an image from a Docker hub, `repository` is required. If you are importing an image, you leave it blank. The plugin will name the repository by the Cloudify instance ID.
  * See [docker pull](https://docs.docker.com/reference/commandline/cli/#pull).
* `name` the name of the container.

**Example**

Here is an example of importing from an URL.

{{< gsHighlight  yaml  >}}

  cloudify_manager:
    type: cloudify.docker.Container
    properties:
      name: cloudify-manager
      image:
        src: http://gigaspaces-repository-eu.s3.amazonaws.com/org/cloudify3/3.2.0/m6-RELEASE/cloudify-docker_3.2.0-m6-b176.tar
        tag: 3.2.0

{{< /gsHighlight >}}

**Lifecycle Operations**

* `Install`
    * `cloudify.interfaces.lifecycle.create` creates the container.
      * **Inputs:**
        * `params` Any parameters exposed by the Docker Py library to the create_container operation.
      * **Sets these runtime-properties**
        * `image_id` The ID of the repository/tag pulled or imported.
        * `container_id` The ID of the container in the Docker Server.
    * `cloudify.interfaces.lifecycle.start` starts the container.
      * **Inputs:**
        * `params` Any parameters exposed by the Docker Py library to the start operation.
        * `processes_to_wait_for` A list of processes to wait for before finishing the start operation.
        * `retry_interval` Before the start operation finishes, Cloudify confirms that the container is started. This is the number of seconds between checking. Defaults to 1.
      * **Sets these runtime-properties**
        * `ports` The ports as shown in the container inspect output.
        * `network_settings` The network_settings dict in the inspect output.
* `Uninstall`
    * `cloudify.interfaces.lifecycle.stop` stops the container.
      * **Inputs:**
        * `params` Any parameters exposed by the Docker Py library to the stop operation.
        * `retry_interval` Before the stop operation finishes, Cloudify confirms that the container is stopped. This is the number of seconds between checking. Defaults to 10.
    * `cloudify.interfaces.lifecycle.delete` deletes the container.
      * **Inputs:**
        * `params` Any parameters exposed by the Docker Py library to the remove_container operation.
        * `retry_interval` Before the delete operation finishes, Cloudify confirms that the container is removed. This is the number of seconds between checking. Defaults to 10.
      * **Unsets these runtime-properties**
        * `container_id`


# Examples

The following examples show how to specify additional parameters via the inputs.

### Create Inputs Example

Maps to the create_container function. You can add any of the parameters available to the create_container function in Docker-Py

{{< gsHighlight  yaml  >}}

  create:
    implementation: docker.docker_plugin.tasks.create_container
    inputs:
      params:
        ports:
          - 27017
          - 28017
        stdin_open: true
        tty: true
        command: mongod --rest --httpinterface --smallfiles

{{< /gsHighlight >}}

### Start Inputs Example

Maps to the start function. You can add any of the parameters available to the start function in Docker-Py

{{< gsHighlight  yaml  >}}

  start:
    implementation: docker.docker_plugin.tasks.start
    inputs:
      params:
        port_bindings:
          27017: 27017
          28017: 28017

{{< /gsHighlight >}}

### Stop Inputs Example

Maps to the stop function. You can add any of the parameters available to the stop function in Docker-Py

{{< gsHighlight  yaml  >}}

  stop:
    implementation: docker.docker_plugin.tasks.stop
    inputs:
      params:
        timeout: 30

{{< /gsHighlight >}}

### Delete Inputs Example

Maps to the remove_container function. You can add any of the parameters available to the remove_container function in Docker-Py

{{< gsHighlight  yaml  >}}

  delete:
    implementation: docker.docker_plugin.tasks.remove_container
    inputs:
      params:
        force: true

{{< /gsHighlight >}}

## Install Docker

While Docker installation is not supported, one way of installing Docker might be like this:

{{< gsHighlight  yaml  >}}

  vm_with_docker:
    derived_from: cloudify.openstack.nodes.Server
    properties:
      cloudify_agent:
        default:
          user: { get_input: agent_user }
          home_dir: /home/ubuntu
      server:
        default:
          image: { get_input: image }
          flavor: { get_input: flavor }
          userdata: |
            #!/bin/bash
            sudo service ssh stop
            curl -o install.sh -sSL https://get.docker.com/
            sudo sh install.sh
            sudo groupadd docker
            sudo gpasswd -a ubuntu docker
            sudo service docker restart
            sudo service ssh start

{{< /gsHighlight >}}
