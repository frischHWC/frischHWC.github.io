---
layout: default
title: Datagen
nav_order: 1
has_children: true
permalink: datagen
---

# Datagen

Datagen is a project that aims at providing a user-friendly, customizable interface to generate data into the various Cloudera Data Platform services. (or even outside of the platform).


## What is it ?

It is a web server exposing APIs to generate data. 

Data generated is shaped in what is called a model. It comes with prebuilt models but anyone can defined its own model and provide it to generate data in any services.

Data can be generated into HDFS (CSV, Avro, Parquet, JSON, ORC), HBase, Hive, Solr, Kudu, Kafka, Ozone (CSV, Avro, Parquet, JSON, ORC) and in local files (CSV, Avro, Parquet, JSON, ORC).

Data generation can also be scheduled to run on a periodic basis.

## Requirements

-  JDK 11

Datagen is designed to run natively on CDP, so a CDP platform is usually needed:
- CDP 7.1.7+ Platform with access to Cloudera Manager

However, you can always run the application as a standalone web server, but you'll need to do all configuration by yourself.

If you plan to build it from source:

- Maven 3.6+ 
- Ansible 2.10+
- Jmespath


## Repository

Datagen code is publicly available here: [https://github.com/frischHWC/datagen](https://github.com/frischHWC/datagen)

The README of this repository gives you all details on how to create a model, how to launch it and also configurations and internals.


## How to install it ?

See: [Installation Section](installation/0-index.md)


## How to generate data ?

See: [Data Generation Section](data-generation/0-index.md)


## How to configure the project ?

See: [Configuration Section](configuration/0-configuration.md)

