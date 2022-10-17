---
layout: default
title: Adavanced - APIs
parent: Advanced
grand_parent: Datagen
permalink: /datagen/advanced/api
nav_order: 1
---

# API

To interact with Datagen, it must be done through REST APIs exposed. 

A Swagger is also started to help interact with them.


## Data Generation APIs

All APIs for data generation are in **/datagen** path and ask for a POST.

Each service has its own API endpoint named like: _/datagen/servicename_ (ex: /datagen/hbase).


Two headers are required when interacting with them:

 - Content-Type: multipart/form-data
 - accept: */*

Some parameters are available foreach API call but **All parameters are optional !** and have default values.


### Common Parameters

All APIs calls for data generation have at least 5 parameters in common:

* rows = Number of rows to generate at each batch of data generation
* batches = Number of batches to launch (you will end up to have (rows x batches) total rows generated)
* threads = To speed up generation, you can multi thread this (by default it is single-threaded), it is recommended to go on 10 threads.
* model by specifiying either:
    * model_file = file path on the machine where a model is present (for example /opt/cloudera/parcels/DATAGEN/models/public_service/weather-model.json)
    * model = upload directly to the swagger your model file from your computer

There are 3 parameters related to kerberos authentication:

* kerb_auth = true or false depending on using kerberos or not
* kerb_user = kerberos user to log in with to make data generation
* kerb_keytab = path to the keytab of this user to login with (must be datagen user's readable)

By default, all these are set to the datagen's user.

There are also 2 other parameters that enables you to schedule a launch:

* scheduled = true or false
* delay_between_executions_seconds = The interval (in seconds) between two executions


### Specific parameters

Each service, will have its own parameters that can be set (and so override default ones.)

As an example, for HBase, you can specify: hbase-site.xml path, hbase zookeeper quorum, hbase zookeeper port, hbase zookeeper znode.


### Example

An example of a curl command:

```shell
curl -X POST "https://ccycloud-1.lisbon.root.hwx.site:4242/datagen/hbase" -H  "accept: */*" -H  "Content-Type: multipart/form-data" -F "batches=1" -F "model_file=@example-model.json;type=application/json" -F "rows=1"
```

Batches and rows parameters are both set to 1 and a file example-model.json is uploaded to be the model.


### Returned data

Once an API call has been made, Datagen parses the model (check for any errors), get configuration (merging default and provided ones), creates a Command Object and queue it for processing.

It returns almost instantly to the user a simple JSON object made of: 

- a UUID of the command created (useful to get later info on the command)
- a string of error: empty if no error, and describing an error if there is one (for example if model file was not parseable because malformed)

Example:
```json
{ "commandUuid": "5c5b0dfb-a1f2-4e95-b76e-750e8d07aa92" , "error": "" }
```


## Control Data Generation APIs

Once data generation has been launched, another set of APIs are helpful to follow the data generation process.

All APIs for data generation control are in **/command** path and ask for a POST.

### Follow up a command to generate data

To get the status of the command launched and basic information, use endpoint _/command/getCommandStatus_ with parameter the command UUID (received earlier).

Example:

```shell
curl -X POST "https://ccycloud-1.lisbon.root.hwx.site:4242/command/getCommandStatus?commandUuid=5c5b0dfb-a1f2-4e95-b76e-750e8d07aa92" -H  "accept: */*"
```

Answer is the JSON self-describing:

```json
{ "commandUuid": "5c5b0dfb-a1f2-4e95-b76e-750e8d07aa92" , "status": "FINISHED" , "comment": "" , "progress": "100.0" ,  "duration": "3ms" }
```

It is also possible to retrieve all configuration for a command and all related information by hitting: _/command/get_ with _commandUuid_ as a parameter.

Two other endpoints allows to retrieve more commands:

- _/command/getAll_ : No parameters, retrieves all commands (even finished commands)
- _/command/getByStatus_ : One parameter _status=_ that must equals: QUEUED, STARTING, RUNNING, FINISHED, FAILED. It will return all commands matching this status.


### Scheduled Commands

All data generation APIs (seen above) allows a user to schedule them recurrently based on scheduling parameters.

The endpoint _/command/getAllScheduled_ returns all scheduled commands.

Note that, a command is removed from being scheduled if one of its execution failed, if you search for it, it will print its error however if you look for the command, telling you to solve it before re-scheduling it.

The endpoint _/command/removeScheduled_ allows a user to remove a scheduled command using its UUID as a parameter.


## Model Tester API

An endpoint exists to allow a user to test its model.

Located at _/model/test_ it inputs either the model file path (on the machine where Datagen runs) or a direct upload of the model file.

It parses the file, returns an error if the model is not working, otherwise a row made using this model.

Hence a user can test its model before generating data.


## Metrics API

A get Endpoint located at _/metrics/all_ retrieved all metrics from the server and return it as a simple JSON. (CM Agent uses this endpoint to gather metrics to Cloudera Manager)

Output example:

```json
{ 
"HDFS_JSON_ROWS_GENERATED" : 0,
"OZONE_ORC_ROWS_GENERATED" : 0,
"HDFS_AVRO_ROWS_GENERATED" : 0,
"OZONE_JSON_FILES_GENERATED" : 0,
"OZONE_AVRO_ROWS_GENERATED" : 0,
"PARQUET_FILES_GENERATED" : 0,
"JSON_FILES_GENERATED" : 0,
"HDFS_CSV_ROWS_GENERATED" : 0,
"OZONE_CSV_FILES_GENERATED" : 0,
"KUDU_ROWS_GENERATED" : 0,
"OZONE_JSON_ROWS_GENERATED" : 0,
"OZONE_PARQUET_FILES_GENERATED" : 0,
"OZONE_CSV_ROWS_GENERATED" : 0,
"KAFKA_ROWS_GENERATED" : 0,
"OZONE_ORC_FILES_GENERATED" : 0,
"HDFS_PARQUET_ROWS_GENERATED" : 0,
"PARQUET_ROWS_GENERATED" : 0,
"HDFS_ORC_FILES_GENERATED" : 0,
"SOLR_ROWS_GENERATED" : 1000000,
"AVRO_FILES_GENERATED" : 0,
"CSV_ROWS_GENERATED" : 22,
"JSON_ROWS_GENERATED" : 0,
"ORC_ROWS_GENERATED" : 0,
"HBASE_ROWS_GENERATED" : 0,
"OZONE_AVRO_FILES_GENERATED" : 0,
"HDFS_PARQUET_FILES_GENERATED" : 0,
"HDFS_AVRO_FILES_GENERATED" : 0,
"OZONE_PARQUET_ROWS_GENERATED" : 0,
"HIVE_ROWS_GENERATED" : 100000,
"ALL_ROWS_GENERATED" : 1100401,
"AVRO_ROWS_GENERATED" : 0,
"CSV_FILES_GENERATED" : 21,
"HDFS_CSV_FILES_GENERATED" : 0,
"GENERATIONS_MADE" : 4,
"HDFS_ORC_ROWS_GENERATED" : 0,
"HDFS_JSON_FILES_GENERATED" : 0,
"ORC_FILES_GENERATED" : 0
}
```


## Health API

An endpoint _/health/status_ accept GET requests only and returns _{ "status": "OK" }_ if server is running well.
