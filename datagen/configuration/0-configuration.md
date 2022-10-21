---
layout: default
title: Configuration
parent: Datagen
nav_order: 3
---

# Configuration of Datagen

To let know Datagen how to conenct to the various services where data will be generated, it requires some input configuration.



## Application.properties file

Basic file for configuration is the application.properties, which is in use when running locally, and is in fact injected when running on CDP.

This file has following permissions when deployed in CDP: `400 datagen:datagen` and is located in running directory: _/var/run/cloudera-scm-agent/process/DATAGEN-XXXXXXX/service.properties/_

There is one section foreach type of service where to load data, they can be empty with no risks.


## Ranger

When running on CDP, it is not required, but you can specify a ranger user and its password with ADMIN role.

This is only used by the _Initialize service dirs and policies_ command to create required default policies in Ranger.

if this is not set, you will have to create required policies by yourself.

_N.B.: If you generate data to some other paths/databases/topics than default ones of datagen, you will need to grant privileges for user datagen, unless you specified another user_



## Settings in Cloudera Manager

Cloudera Manager makes a bunch of auto-configuration for us, such as kerberos or auto-TLS.

In Cloudera Manager, under _Datagen > Configuration_, you can see all properties, you can see that some are automatically set and others not.

Most important ones, are the auto-discovery ones (described below).

All properties are injected into the application file and passed to the web server before starting it.


## Auto-Discovery

When starting up, Datagen is loading the _application.properties_ in-memory and proceeds to what is called an auto-discovery.

When setup in CDP, Datagen (if _cm.autodiscovery_ is true) will make an auto-discovery of **ALL** services using CM API. (User does not need to specify anything).

It also possible rely on fields described as **AUTO-DISCOVERY**, that are by defaults are filled in. 

These fields, referenced properties file such as _hdfs-site.xml, hive-site.xml_ etc... 
With this, Datagen parsed these files and automatically configure all possible other fields.

Once started, you can see such logs in Datagen Web Server:


If using CM auto-discovery:

```shell
9:35:04.701 AM INFO PropertiesLoader [main] Going to auto-discover hbase.zookeeper.quorum with CM API
9:35:04.701 AM 	INFO PropertiesLoader [main] Going to auto-discover hbase.zookeeper.port with CM API
```

If not using CM auto-discovery but configuration files auto-discovery:

```shell
2022-10-13 13:33:49,220 INFO  [main] com.cloudera.frisch.randomdatagen.config.PropertiesLoader: Going to auto-discover hbase.zookeeper.quorum
2022-10-13 13:33:49,222 DEBUG [main] com.cloudera.frisch.randomdatagen.Utils: Return value: server_zk1,server_zk_2,server_zk_3 from file: dev-support/test_files/hbase-site.xml for property: hbase.zookeeper.quorum
2022-10-13 13:33:49,222 INFO  [main] com.cloudera.frisch.randomdatagen.config.PropertiesLoader: Going to auto-discover hbase.zookeeper.port
2022-10-13 13:33:49,223 DEBUG [main] com.cloudera.frisch.randomdatagen.Utils: Return value: 2181 from file: dev-support/test_files/hbase-site.xml for property: hbase.zookeeper.property.clientPort
```

## Settings in APIs

Each time a command is created to generates data, first thing it does, is take the in-memory map of configurations and make a copy.

Then a user, can provide any configurations in its API call, it will override existing ones *ONLY* for this job and have no impact on others. 
Moreover, a user can define nothing and it will default to actual default configurations from startup.

This also means, you can theoretically push data to any outside service that respects the API used by Datagen (example _kafka-client-2.5_ for an external kafka).

You can also using APIs to provide different users (through keytab and principal) that will be used by datagen to generate data, avoiding to create specific policies for user datagen.