# Notes on using Cassandra

These are my notes from the DataStax training on Cassandra.
Each file corresponds to one module:

* **Installing and configuring cassandra**:
  - installation and key configuration files
  - starting/stopping cassandra
  - cassandra cluster manager (CCM)
  - introduction to CQL
  - tools: nodetool, cassandra stress


* **Understanding the cassandra architecture**:
  - request coordination
  - partitionning process
  - replication strategy
  - consistency levels
  - repair operations
  - internodes communications

* **Learning the Cassandra write path**
  - basic write path concepts
  - Commitlog
  - Memtables
  - SStables and compaction
  - data files
  - tombstones and compaction
