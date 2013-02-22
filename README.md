Echo Service for Stackato
=========================

An example system service for Stackato which echoes service requests and
provides a port credential. This sample, along with the following
instructions, show how to add a system service to a Stackato micro
cloud.

This sample is based on [cloudfoundry/vcap-services/echo](https://github.com/cloudfoundry/vcap-services/tree/master/echo)
with some additional configuration (e.g for `kato` and `supervisord`)
and other minor differences (e.g. the Gemfile). The instructions
here are for [Stackato
2.6](http://www.activestate.com/stackato/get_stackato).

## Copying/Cloning the Service to Stackato

Log in to the Stackato VM (micro cloud or service node) as the
'stackato' user and clone this repository directly into a
vcap/services/echo directory:

    $ git clone git://github.com/ActiveState/stackato-echoservice.git /s/vcap/services/echo

Alternatively, copy a local checkout to Stackato using SCP:

    $ scp -r stackato-echoservice stackato@stackato-vm.local:~/stackato/vcap/services/echo

## Install the service gems

On the VM, go to the 'echo' directory and run 'bundle update':

    $ cd /s/vcap/services/echo
    $ bundle update

## Edit the config files

Some settings in the default files in the config/ directory will need to be modified. This may include:

* `cloud_controller_uri`: This needs to match the API endpoint of your
  system (e.g. api.stackato-wxyz.local)
* `token`: can be any string, but we will need to add this auth token
  to the cloud_controller in a later step
* `mbus`: This should match the setting for other services. You can check
  the correct setting using `kato config get redis_node mbus`

## Install to supervisord

Supervisord monitors, starts, and stops all Stackato processes, and will
need to have configuration files for the 'echo_gateway' and 'echo_node'
processes. These supervisord config files can be found in the
'stackato-conf' directory.

First, stop kato and supervisord:

    $ kato stop
    ...
    $ stop-supervisord
  
Copy the supervisord config files:

    $ cp stackato-conf/echo_*  /s/etc/supervisord.conf.d/
  

## Install to Kato

The 'kato' administrative tool will also need configuration to recognize
the new service. This can be done by appending the contents of
process-snippet.yml and roles-snippet.yml to their respective
kato config files:

    $ cat stackato-conf/processes-snippet.yml >> /s/etc/kato/processes.yml
    $ cat stackato-conf/roles-snippet.yml >> /s/etc/kato/roles.yml

Note that 'echo_node' should always be specified before 'echo-gateway'.

Optionally, you can add echo to the "data-services" group in
role_groups.yml or create a new group. These groupings enable subsequent
easy enabling/disabling of logical groups of services.

## Loading the config into Doozer

Doozer is the centralized configuration management component in
Stackato, including the service configuration we have just added. To
load the settings from the YAML files in 'echo/config/':

Change to the /s/ directory (symlink of /home/stackato/stackato/), then
start supervisord:

    $ start-supervisord

Run the following two commands:

    RUBYLIB=kato/lib ruby -e 'require "yaml"; require "kato/doozer"; Kato::Doozer.set_component_config("echo_node", YAML.load_file("/s/vcap/services/echo/config/echo_node.yml"))'
  
    RUBYLIB=kato/lib ruby -e 'require "yaml"; require "kato/doozer"; Kato::Doozer.set_component_config("echo_gateway", YAML.load_file("/s/vcap/services/echo/config/echo_gateway.yml"))'
  
These commands must be run after any change in the YAML config files.


## Add the service AUTH token to the cloud controller

The auth token used must match between the service and cloud controller
nodes so we must set them accordingly:

    $ kato config set cloud_controller builtin_services/echo '{"token": "<echo_gateway.yml auth token>"}' --json

Replace the <echo_gateway.yml auth token> string above with the auth
token you setup up earlier in config/echo_gateway.yml

## Enable echo and start

    $ kato role add echo
    starting echo_node...               ok
    starting echo_gateway...            ok
    starting logyard...                 ok
    starting cloudevents...             ok
    starting systail...                 ok

Finally, start all other stackato processes:
    
    $ kato start

## Verify the service

Once the echo service has been enabled and started in kato, clients
targeting the system should be able to see it listed in the System
Services output:

    $ stackato services
  
    ============== System Services ==============
   
    +------------+---------+------------------------------------------+
    | Service    | Version | Description                              |
    +------------+---------+------------------------------------------+
    | echo       | 1.0     | Echo service                             |
    | filesystem | 1.0     | Persistent filesystem service            |
    | memcached  | 1.4     | Memcached in-memory object cache service |
    | mongodb    | 2.0     | MongoDB NoSQL store                      |
    | postgresql | 9.1     | PostgreSQL database service              |
    | rabbitmq   | 2.4     | RabbitMQ message queue                   |
    | redis      | 2.4     | Redis key-value store service            |
    +------------+---------+------------------------------------------+
    
To create a new service:

    $ stackato create-service echo
    Creating Service [echo-503db]: OK
