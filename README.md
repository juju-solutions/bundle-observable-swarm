# Production-grade Docker Swarm Cluster with Logging and Monitoring

## Overview

This is a Docker Swarm bundle that also includes logging and monitoring. It is comprised of the following components and features:

- Swarm (clustering engine)
  - Two node starter cluster, supports up to 20 nodes  
  - TLS used for communication between nodes for security
  - ZFS used as a docker datastore for resiliance and performance
- Consul (key-value and DNS)
  - Three node cluster for reliability
- Elastic stack
   - Two nodes for ElasticSearch
   - One node for a Kibana dashboard
   - Beats on every Swarm and Consul node:
     - Filebeat for forwarding logs to ElasticSearch
     - Topbeat for insterting server monitoring data to ElasticSearch

# Usage

    juju deploy observable-swarm

Any of the services provided can be scaled out post-deployment. The charms support workload status, so it is recommended to run `watch juju status` to monitor the cluster coming up. After it is deployed you need to grab the credentials from the lead swarm node to control the cluster:

    juju scp swarm/0:swarm_credentials.tar .
    tar xvf swarm_credentials.tar
    cd swarm_credentials
    source enable.sh

This sets the proper environment variables to control the cluster, you can now check the state of the cluster:

*Note, before running docker commands be sure [docker](https://docs.docker.com/engine/installation) is installed on the same machine as where you are running juju client commands from.*

    docker status

And then run a hello-world to launch a container in the cluster, and then checking to ensure that the container is running on the remote cluster:

    docker run hello-world
    docker ps -a

List all networks in the cluster:

    docker network ls

List all storage in the cluster:

    docker storage ls



## Access Kibana dashboard

The Kibana dashboard can display real time graphs and charts on the details of
the cluster. The Beats charms are sending metrics to the Elasticsearch and
Kibana displays the data with graphs and charts.

Get the charm's public address from the `juju status` command.

* Access Kibana by browser:  http://KIBANA_IP_ADDRESS/
* Select the index pattern that you want as default from the left menu.
  * Click the green star button to make this index a default.
* Select "Dashboard" from the Kibana header.
  * Click the open folder icon to Load a Saved Dashboard.
* Select the "Topbeat Dashboard" from the left menu.

![Setup Kibana](http://i.imgur.com/tgYFSjM.gif)


## Scale out Usage

By default any docker container you run will automatically be spread throughout the number of swarm nodes you have deployed. We recommend no more than 30 docker containers per host if you are just using default constraints.

### Scaling Swarm

To add more swarm nodes to host containers:

    juju add-unit swarm

or specify machine constraints:

    juju add-unit swarm --constraints "cpu-cores=8 mem=32G"

Refer to the [machine constraints documentation](https://jujucharms.com/docs/stable/charms-constraints) for other machine constraints that might be useful for the swarm nodes.

### Scaling Consul

Consul is used for service discovery, as a key-value store, and DNS resolution for the cluster. For reliability the cluster defaults to three instances out of the box.

For more scalability, we recommend bumping up to 5 _total_ consul nodes per model. You can add more nodes with `juju add-unit consul`. The consul documentation [recommends 3 to 5 nodes](https://www.consul.io/docs/internals/architecture.html) to strike a balance between availability and performance.

### Scaling elasticsearch

ElasticSearch is used to hold all the log data and server information logged by Beats. You can add more ES nodes by simply adding more units:

    juju add-unit elasticsearch

## Known Limitations and Issues

 The following issues still need to be resolved with this solution and are being worked on:

 - Killing the the swam master will result in loss of cluster PKI.
 - Consul nodes are not using TLS yet.
 - No easy way to find where the docker containers got deployed without sshing into each swarm node.
 - Add status to show which consul node is leader, and who are followers, just like swarm.
 - Consul isn't using ZFS.

# Contact Information

  Though this will be listed in the charm store itself don't assume a user will
  know that, so include that information here:

## Upstream Project Name

    - [Homepage](https://github.com/juju-solutions/bundle-observable-swarm)
    - [Bug tracker](https://github.com/juju-solutions/bundle-observable-swarm/issues)
