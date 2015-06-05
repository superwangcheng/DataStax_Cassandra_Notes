# Installing & configuring cassandra

## Installation and key files

Prerequisites :
* Java (Oracle JDK, see [here](http://idroot.net/tutorials/how-to-install-apache-cassandra-on-ubuntu-14-04/) for instruction on Unbuntu 14.04)
* Python (at least 2.5)

Download Cassandra (I used the Tarball version from PlanetCassandre [here](http://planetcassandra.org/cassandra/), v2.1.5).

Download the [student-files](https://academy.datastax.com/system/files/assets/student-files.zip) from DataStax (contains files for tutorial and exercises).

**Key files and folders** (we work with the tarball so everything is in one location) :
* `bin/`: executables
* `conf/`: configuration files
	- `cassandra.yaml`: primary config file for each instance (e.g. data directory location...)
	- `cassandra-env.sh`: Jave environment config
	- `log4j-server.properties`: log level and location settings (**note** this has been switched to `logback` in recent versions)
	- `cassandra-rackdc.properties`: config to set Rak and Data Center to which this node belongs
	- `cassandra-topology.properties`: config IP addressing for Racks and Data Centers
* `javadoc/`: C* source documentation
* `pylib/`: python libraries
* `tools/`:Additional tools e.g. Cassandra stress

All documentation is available on DataStax (see also getting started guide).

## Configuring a cassandra node

A cassandra cluster is a p2p set of nodes with no single point of failure.

* **node**: one Cassandra instance
* **Partition**: one unit of data on a node (unit by which data is replicated)
* **Rack**: logical set of nodes
* **Data Center**: logical set of racks (nodes < racks < data centers)
* **Cluster**: the full set of nodes which map to a single complete **token ring**

Key properties set in `cassandra.yaml` configuration file:

* *cluster_name* (all nodes in cluster must have the same value)
* *listen_address* (IP address other nodes use to connect to this node)
* *commitlog_directory* (best mount on a separate disk unless SSD)
* *data_file_directory*: storage directory for SSTables
* *saved_caches_directory*: storage directory for key caches and row caches
* *rpc_address / rpc_port*
* *native_ransport_port*: listen address for native CQL Driver binary protocol

Key properties set in `cassandra-env.sh`:

* JVM Heap Size settings

Key properties in `log4j-server.properties` (**note** now `logback`):

* Cassandra system.log location
* Cassandra logging level

Example: configure a Cassandra node :
set the following in `cassandra.yaml` (assuming cassandra folder is on Desktop)
```
cluster_name: 'demo_1node'
data_file_directories:
     - /home/username/Desktop/cassandra/data
commitlog_directory: /home/username/Desktop/cassandra/commitlog
saved_caches_directory: /home/username/Desktop/cassandra/saved_caches
```

and in `cassandra-env.sh` (because we just work on a modest laptop...) :
```
MAX_HEAP_SIZE="1024M"
HEAP_NEWSIZE="256M"
```

and in `logback.xml` (replacing `log4j-server.properties`) set:
```
<fileNamePattern>$/home/renaud/Desktop/cassandra/system.log.%i.zip</fileNamePattern>
```

## Manually starting and stopping a Cassandra node

Launch a server instance using the cassandra utility.
Some options are:
* `-f` (start in foreground),
* `-p <filename>` (Log process IF in named file)
* `-v` (print version)
* `-D <parameter>` (pass startup parameter).

note: for package install cassandra can be started as a service

To close cassandra:
* if foreground: close terminal window
* if background: determine process PID and use `kill <PID>`
* `sudo service cassandra stop` (for package install only)

example:
```
bin/cassandra -p PID
```
This starts cassandra and creates the directories set in configuration files. When we see:
```
INFO  12:59:09 Node localhost/127.0.0.1 state jump to normal
```
That means cassandra is up and running.

To stop use `kill <PID>` (check `system.log` for output)

## Using Cassandra Cluster Manager (ccm)

Creates and manage multinode clusters on a local machine
* **for configuring development and test clusters only**
* **communicates with localhost only**
* **not used for configuring production cluster**
* Open Source, in Python
* Installation instructions on the DataStax Dev Blog

CCM can target a **cluster** or **its nodes**.

* one cluster is always the current default for cluster commands : `ccm [cluster command] [options]`
* nodes are automatically named node1, node2, etc. `ccm [node name] [node command] [option]`
* use `ccm -help` to list all commands and `ccm [command] -help` for a specific command
* support **over 40 commands** including:
	- *create*: create new cluster
	- *list*: display list of local clusters managed by CCM
	- *populate*: add n nodes to current empty cluster
	- *add*: add node to current cluster
	- *start*: start all nodes in current cluster
	- *status*: display up/down status for each node in specified cluster
	- and many more (some relying on underlying tools)

Example: create a cluster named *demo_1node* with a previous version of cassandra, 1 node, start it and print debug infos:
```
ccm create demo_1node -v binary:2.0.1 -n 1 -s -d
ccm list
ccm status # target the default cluster (node is currently UP)
ccm stop
ccm create demo_3node -v binary:2.1.5 # create another cluster with different cassandra version
ccm status # no nodes yet
ccm populate -n 2
ccm node2 show  # show node info
ccm add node3 -i 127.0.0.3 -j 7300  # add a node specifying address
ccm start
ccm node2 stop
```

Note that after doing this, demo_1node folter is created in the /home/username/.ccm folder (inside is node1 and inside the typical cassandra folders). There is also a repository folder (the latter contains versions of cassandra available).

**Remark**: I was not able to start the *demo_3node* cluster from the code above. Got the following error:

```
ccmlib.common.UnavailableSocketError: Inet address 127.0.0.1:9042 is not available:
[Errno 98] Address already in use
```
Seems this has to do with loopback activation but I was not able to fix this. I removed the cluster ans created it again with `ccm create demo_3node -v 2.1.5` and `ccm populate -n 3` and it worked. Maybe it has to do with adding nodes manually, will have to check this issue...

## Cassandra Query Language (CQL) Shell

This is the **primary interface for interacting with the Cassandra database**
(using command line, cqlsh, or DevCenter of drivers)

* modeled after SQL for familiarity (+ additional commands)
* Familiar data types including *blob, boolean, decimal, double, float, int, text, varchar*
* Cassandra also support timestamp types : *uuid, timeuuid, counter*
* There is **no Table joins**

cql shell is an interactive command line utility for excecuting CQL statements.
* It interacts with its local Cassandra instance by default (if no host specified)
* supports tab completion
* located in /bin/cqlsh- lauch with `cqlsh [options] [host [port]]`
* some options are `-k [keyspace]` (interact with specified keyspace), `-f [file_name]` (excecutes commands from file then exit)
* uses **Thrift protocole** to interact with Cassandra nodes

cqlsh supports:
* CQL DDL qnd DML commands for schema definition and data manipulation
* CQL shell commands to support CQL use
CQL is a Cassandra capability, not restricted to cqlsh
CQL shell commands may only be used in cqlsh

Example (continued - assume demo_3node cluster with all nodes up)
```
bin/cqlsh  # run cql shell
cqlsh> EXIT;
```
we can run cql shell directly on a given node using ccm:
```
ccm node2 cqlsh
```

## Copying external data into tables, for development

use COPY command (check `cqlsg> HELP COPY;`)

Example to import data from the file *album.csv* :
```
cqlsh:musicdb> COPY album (title, year, performer, genre, tracks)
               ... FROM 'album.csv'
               ... WITH HEADER = true;
```
Remember paths are relative (here we started cqlsh on node2 from the student-files/ directory)
then we can check the first entry with with `SELECT * FROM album LIMIT 1;`
Note that 'tracks' is a map consisting ok key:value pairs.

## Using DataStax DevCenter

DevCenter is a GUIfor developpers and adminisrators wanting to work with IDE.
* supports CQL language statements
* **does not** support CQL shell command (like SOURCE, COPY, SHOW, DESCRIBE,...)
Enables to manage / save / excecute scripts, set limit for number of outputs...

## NodeTool

A command line cluster management utility. Located in bin/nodetool.
Commands are issued to a specified host and port number.
can be run directly from ccm e.g. `ccm node3 nodetool info`.
Support over 40 commands, example :
* status: display summary information about a cluster: `nodetool -h [host] -p [port] status [keyspace]`.
* info: display settings and data for the specified node `nodetool -h [host] -p [port] info`.
status and info display similar but not identical informations.
* ring: displays a summary state for nodes in target node's token ring `nodetool -h [host] -p [port]`. Aids in comparing load balance and finding downed nodes.

## Cassandra stress

A command line benchmark and load-testing utility
Primary source of statistic to tune cassandra.

* performs inserts and reads to a test keyspace to measure performance
* located in tools/bin/cassandra-stress: `cassandra-stress [options] [-o [operation name]]`
	- `-o` (--operation): INSERT, READ, etc.
	- `-t` (--threads): processors threads to use for operation
	- `-k` (--keep-going): ignore errors during insert and read
	- `-n` (--num-keys): number of records to insert
* Additional options enable configuration of:
	- column number, column size, data cardinality
	- compaction strategy, consistency level
	- replication strategy, replication factor
* for more details see the course: Apache Cassandre: Operation and Performance Tuning

Cassandra stress output:
* each line reports for the -i period (default 10 seconds)
	- total: total operation since start of test
	- interval_op_rate: number of operations per second this interval
	- interval_op_rate: number of keys/rows written per second this interval
	- latency: average latency in ms for each operation this interval
	- percentiles
	- elapsed: elapsed seconds since start of test

There was a source code change since the video (see [here](http://fossies.org/diffs/apache-cassandra/2.1.2-src_vs_2.1.3-src/tools/stress/README.txt-diff.html)). The code shown in the video does not work anymore. See some more recent documentation for info.

* Create a new Keyspace with a series of tables which have been used for testing.

## Additional tools

Each of these tools are distributed with Cassandra:
* `sstableload`: bolk load external data into a cluster
* `sstablescrub`: used with nodetool repair to fix corrupted tables
* `sstable2json`, `json2sstable`: export/import tools for data inspection
* `sstableupgrade`: upgrade SSTables to current Cassandra version
