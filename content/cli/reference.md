---
layout: bt_wiki
title: CLI Command Reference
category: Command-Line Interface
draft: false
abstract: Reference for CFY commands
weight: 200
---

usage: cfy [-h] [--version]
           {status,blueprints,bootstrap,teardown,workflows,recover,node-instances,snapshots,deployments,init,agents,dev,use,plugins,nodes,executions,local,events,ssh}
           ...

Manages Cloudify in different Cloud Environments

positional arguments:
  {status,blueprints,bootstrap,teardown,workflows,recover,node-instances,snapshots,deployments,init,agents,dev,use,plugins,nodes,executions,local,events,ssh}
    status              Show a management server's status
    blueprints          Manages Cloudify's Blueprints
    bootstrap           Bootstrap a Cloudify management environment
    teardown            Teardown Cloudify
    workflows           Manages Deployment Workflows
    recover             Performs recovery of the management machine and all
                        its contained nodes.
    node-instances      Manage node instances
    snapshots           Manages Cloudify's Snapshots
    deployments         Manages and Executes Cloudify's Deployments
    init                Initialize cfy work environment
    agents              Manages Cloudify's Agents
    dev                 Executes fabric tasks on the management machine
    use                 Use/switch to the specified management server
    plugins             Manages Cloudify's plugins
    nodes               Manage nodes
    executions          Manages Cloudify's Executions
    local               Execute workflows locally
    events              Manages Cloudify's events
    ssh                 SSH to management server

optional arguments:
  -h, --help            show this help message and exit
  --version             show version information and exit