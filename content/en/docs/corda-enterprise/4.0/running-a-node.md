---
aliases:
- /releases/4.0/running-a-node.html
date: '2020-01-08T09:59:25Z'
menu:
  corda-enterprise-4-0:
    parent: corda-enterprise-4-0-corda-nodes
tags:
- running
- node
title: Running nodes locally
---


# Running nodes locally


{{< note >}}
You should already have generated your node(s) with their CorDapps installed by following the instructions in
[Creating nodes locally](generating-a-node.md).

{{< /note >}}
There are several ways to run a Corda node locally for testing purposes.


## Starting a Corda node using DemoBench

See the instructions in [DemoBench](demobench.md).



## Starting a Corda node from the command line

Run a node by opening a terminal window in the node’s folder and running:

```shell
java -jar corda.jar
```

By default, the node will look for a configuration file called `node.conf` and a CorDapps folder called `cordapps`
in the current working directory. You can override the configuration file and workspace paths on the command line (e.g.
`./corda.jar --config-file=test.conf --base-directory=/opt/corda/nodes/test`).

Optionally run the node’s webserver as well by opening a terminal window in the node’s folder and running:

```shell
java -jar corda-webserver.jar
```


{{< warning >}}
The node webserver is for testing purposes only and will be removed soon.

{{< /warning >}}




### Setting JVM arguments

There are several ways of setting JVM arguments for the node process (particularly the garbage collector and the memory settings).
They are listed here in order of increasing priority, i.e. if the same flag is set in a way later in this list, it will override
anything set earlier.


* **Default arguments in capsule**: 
The capsuled corda node has default flags set to `-Xmx512m -XX:+UseG1GC` - this gives the node (a relatively
low) 512 MB of heap space and turns on the G1 garbage collector, ensuring low pause times for garbage collection.


* **Node configuration**: 
The node configuration file can specify custom default JVM arguments by adding a section like:

```kotlin
custom = {
   jvmArgs: [ '-Xmx1G', '-XX:+UseG1GC' ]
}
```

Note that this will completely replace any defaults set by capsule above, not just the flags that are set here, so if you use this
to set e.g. the memory, you also need to set the garbage collector, or it will revert to whatever default your JVM is using.


* **Capsule specific system property**: 
You can use a special system property that Capsule understands to set JVM arguments only for the Corda
process, not the launcher that actually starts it:

```kotlin
java -Dcapsule.jvm.args="-Xmx:1G" corda.jar
```

Setting a property like this will override any value for this property, but not interfere with any other JVM arguments that are configured
in any way mentioned above. In this example, it would reset the maximum heap memory to `-Xmx1G` but not touch the garbage collector settings.
This is particarly useful for either setting large memory allowances that you don’t want to give to the launcher or for setting values that
can only be set on one process at a time, e.g. a debug port.


* **Command line flag**: 
You can set JVM args on the command line that apply to the launcher process and the node process as in the example
above. This will override any value for the same flag set any other way, but will leave any other JVM arguments alone.




### Command-line options

The node can optionally be started with the following command-line options:


* `--base-directory`, `-b`: The node working directory where all the files are kept (default: `.`).
* `--config-file`, `-f`: The path to the config file. Defaults to `node.conf`.
* `--dev-mode`, `-d`: Runs the node in development mode. Unsafe in production. Defaults to true on MacOS and desktop versions of Windows. False otherwise.
* `--no-local-shell`, `-n`: Do not start the embedded shell locally.
* `--on-unknown-config-keys <[FAIL,INFO]>`: How to behave on unknown node configuration. Defaults to FAIL.
* `--sshd`: Enables SSH server for node administration.
* `--sshd-port`: Sets the port for the SSH server. If not supplied and SSH server is enabled, the port defaults to 2222.
* `--verbose`, `--log-to-console`, `-v`: If set, prints logging to the console as well as to a file.
* `--logging-level=<loggingLevel>`: Enable logging at this level and higher. Possible values: ERROR, WARN, INFO, DEBUG, TRACE. Default: INFO.
* `--help`, `-h`: Show this help message and exit.
* `--version`, `-V`: Print version information and exit.


#### Sub-commands

`clear-network-cache`: Clears local copy of network map, on node startup it will be restored from server or file system.

`initial-registration`: Starts initial node registration with the compatibility zone to obtain a certificate from the Doorman.

Parameters:


* `--network-root-truststore`, `-t` **required**: Network root trust store obtained from network operator.
* `--network-root-truststore-password`, `-p`: Network root trust store password obtained from network operator.

`generate-node-info`: Performs the node start-up tasks necessary to generate the nodeInfo file, saves it to disk, then exits.

`generate-rpc-ssl-settings`: Generates the SSL keystore and truststore for a secure RPC connection.

`install-shell-extensions`: Install `corda` alias and auto completion for bash and zsh. See [Shell extensions for CLI Applications](cli-application-shell-extensions.md) for more info.

`validate-configuration`: Validates the actual configuration without starting the node.



### Enabling remote debugging

To enable remote debugging of the node, run the node with the following JVM arguments:

`java -Dcapsule.jvm.args="-agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=5005" -jar corda.jar`

This will allow you to attach a debugger to your node on port 5005.


## Starting a node with JMX monitoring enabled

To enable export of JMX metrics over HTTP via [Jolokia](https://jolokia.org/), run the following from the terminal window:

`java -Dcapsule.jvm.args="-javaagent:drivers/jolokia-jvm-1.3.7-agent.jar=port=7005" -jar corda.jar`

This command line will start the node with JMX metrics accessible via HTTP on port 7005.

See Monitoring your node for further details.


## Starting all nodes at once on a local machine from the command line



### Native

If you created your nodes using `deployNodes`, a `runnodes` shell script (or batch file on Windows) will have been
generated to allow you to quickly start up all nodes and their webservers. `runnodes` should only be used for testing
purposes.

Start the nodes with `runnodes` by running the following command from the root of the project:


* Linux/macOS: `build/nodes/runnodes`
* Windows: `call build\nodes\runnodes.bat`


{{< warning >}}
On macOS, do not click/change focus until all the node terminal windows have opened, or some processes may
fail to start.

{{< /warning >}}


If you receive an `OutOfMemoryError` exception when interacting with the nodes, you need to increase the amount of
Java heap memory available to them, which you can do when running them individually. See
[Starting a Corda node from the command line](#starting-an-individual-corda-node).


### docker-compose

If you created your nodes using `Dockerform`, the `docker-compose.yml` file and corresponding `Dockerfile` for
nodes has been created and configured appropriately. Navigate to `build/nodes` directory and run `docker-compose up`
command. This will startup nodes inside new, internal network.
After the nodes are started up, you can use `docker ps` command to see how the ports are mapped.


{{< warning >}}
You need both `Docker` and `docker-compose` installed and enabled to use this method. Docker CE
(Community Edition) is enough. Please refer to [Docker CE documentation](https://www.docker.com/community-edition)
and [Docker Compose documentation](https://docs.docker.com/compose/install/) for installation instructions for all
major operating systems.

{{< /warning >}}



## Starting all nodes at once on a remote machine from the command line

By default, `Cordform` expects the nodes it generates to be run on the same machine where they were generated.
In order to run the nodes remotely, the nodes can be deployed locally and then copied to a remote server.
If after copying the nodes to the remote machine you encounter errors related to `localhost` resolution, you will additionally need to follow the steps below.

To create nodes locally and run on a remote machine perform the following steps:


* Configure Cordform task and deploy the nodes locally as described in [Creating nodes locally](generating-a-node.md).
* Copy the generated directory structure to a remote machine using e.g. Secure Copy.
* Optionally, add database configuration settings if they weren’t specified in the first step.This step needs to be performed if the local machine doesn’t have access to the remote database (a database couldn’t be configured in the first step).
In each top level `[NODE NAME]_node.conf` configuration file add the database settings and copy the JDBC driver JAR (if required).
Edit the top level `[NODE NAME]_node.conf` files only and not the files (`node.conf`) inside the node subdirectories.
* Optionally, bootstrap the network on the remote machine.This is optional step when a remote machine doesn’t accept `localhost` addresses, or the generated nodes are configured to run on another host’s IP address.If required change host addresses in top level configuration files `[NODE NAME]_node.conf` for entries `p2pAddress` , `rpcSettings.address` and  `rpcSettings.adminAddress`.Run the network bootstrapper tool to regenerate the nodes network map (see for more explanation [Network Bootstrapper](network-bootstrapper.md)):`java -jar corda-tools-network-bootstrapper-Master.jar --dir <nodes-root-dir>`
* Run nodes on the remote machine using [runnodes command](#starting-all-nodes-at-once).

The above steps create a test deployment as `deployNodes` Gradle task would do on a local machine.


## Database migrations

Depending on the versions of Corda and of the CorDapps used, database migration scripts might need to run before a node is able to start.
For more information refer to [Database management](database-management.md).
