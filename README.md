gofluent
========
[![wercker status](https://app.wercker.com/status/1908429c90a6d7bd437a63210fb1a97f/s "wercker status")](https://app.wercker.com/project/bykey/1908429c90a6d7bd437a63210fb1a97f)

This program is something acting like fluentd rewritten in Go.

Table of Contents
=================

* [Introduction](#introduction)
* [Architecture](#architecture)
* [Implementation](#implementation)
	* [Overview](#overview)
	* [Data flow](#data-flow)
* [Plugins](#plugins)
	* [Tail Input Plugin](#tail-input-plugin)
	* [Httpsqs Output Plugin](#httpsqs-output-plugin)
	* [Stdout Output Plugin](#stdout-output-plugin)
	* [Mongodb Output Plugin](#mongodb-output-plugin)

Introduction
============

Fluentd is originally written in CRuby, which has too many dependecies.

I hope fluentd to be simpler and cleaner, as its main feature is simplicity and rubostness.

Architecture
============

```
    +---------+     +---------+     +---------+     +---------+
    | server1 |     | server2 |     | server3 |     | serverN |
    |---------|     |---------|     |---------|     |---------|
    |         |     |         |     |         |     |         |
    |---------|     |---------|     |---------|     |---------|
    |gofluent |     |gofluent |     |gofluent |     |gofluent |
    +---------+     +---------+     +---------+     +---------+
        |               |               |               |
         -----------------------------------------------
                                |
                                | HTTP POST
                                V
                        +-----------------+
                        |                 |
                        |      Httpmq     |
                        |                 | 
                        +-----------------+
                                |
                                | HTTP GET
                                V 
                        +-----------------+                 +-----------------+
                        |                 |                 |                 |
                        |   Preprocessor  | --------------> |     Storage     |
                        |                 |                 |                 | 
                        +-----------------+                 +-----------------+
```

Implementation
==============
Overview
--------

```
Input -> Router -> Output
```
Data flow
---------

```
                        -------<-------- 
                        |               |
                        V               | generate pool
       InputRunner.inputRecycleChan     | recycling
            |           |               |     ^   
            |            ------->-------       \ 
            |               ^           ^       \
    InputRunner.inChan      |           |        \
            |               |           |         \
            |               |           |          \
    consume |               |           |           \
            V               |           |            \
          Input(Router.inChan) ---->  Router ----> (Router.outChan)Output.inChan

```

Plugins
=======

Tail Input Plugin
-----------------
The in_tail input plugin allows gofluent to read events from the tail of text files. Its behavior is similar to the tail -F command.

Example Configuration

in_tail is included in gofluent’s core. No additional installation process is required.
```
<source>
  type tail
  path /var/log/httpd-access.log
  pos_file /var/log/httpd-access.log.pos
  tag apache.access
  format /^(?P<host>[^ ]*) [^ ]* (?P<user>[^ ]*) \[(?P<time>[^\]]*)\] "(?P<method>\S+)(?: +(?P<path>[^ ]*) +\S*)?" (?P<code>[^ ]*) (?P<size>[^ ]*)(?: "(?P<referer>[^\"]*)" "(?P<agent>[^\"]*)")?$/
</source>
```
*type (required)*
The value must be tail.

*tag (required)*
The tag of the event.

*path (required)*
The paths to read.

*format (required)*
The format of the log. It is the name of a template or regexp surrounded by ‘/’.
The regexp must have at least one named capture (?P\<NAME\>PATTERN).

The following templates are supported:
- regexp
- json
One JSON map, per line. This is the most straight forward format :).
```
format json
```
*pos_file (highly recommended)*
This parameter is highly recommended. gofluent will record the position it last read into this file.
```
pos_file /var/log/access.log.pos
```

*sync_interval*
The sync interval of pos file, default is 2s.

Httpsqs Output Plugin
---------------------
The out_httpsqs output plugin allows gofluent to send data to httpsqs mq.

Example Configuration

out_httpsqs is included in gofluent’s core. No additional installation process is required.
```
<match httpsqs.**>
  type httpsqs
  host localhost
  port 1218
  flush_interval 10
</match>
```
*type (required)*
The value must be httpsqs.

*host (required)*
The output target host ip.

*port (required)*
The output target host port.

*auth (highly recommended)*
The auth password for httpsqs.

*flush_interval*
The flush interval for sending data to httpsqs.

*gzip*
The gzip switch, default is on.

Forward Output Plugin
---------------------
The out_forward output plugin allows gofluent to forward events to another gofluent.

Example Configuration

out_forward is included in gofluent’s core. No additional installation process is required.
```
<match forward.**>
  type forward
  host localhost
  port 1218
  flush_interval 10
</match>
```
*type (required)*
The value must be forward.

*host (required)*
The output target host ip.

*port (required)*
The output target host port.

*flush_interval*
The flush interval for sending data to httpsqs.

*connect_timeout*
The connect timeout value.

*sync_interval*
The sync interval of metadata file, default is 2s.

*buffer_path(required)*
The disk buffer path for output plugin, default is /tmp/test.

*buffer_queue_limit*
The queue limit of disk buffer, default is 64M.

*buffer_chunk_limit*
The chunk limit of disk buffer to forward, default is 8M.

Stdout Output Plugin
--------------------
The out_stdout output plugin allows gofluent to print events to stdout.

Example Configuration

out_stdout is included in gofluent’s core. No additional installation process is required.
```
<match stdout.**>
  type stdout
</match>
```
*type (required)*
The value must be stdout.

Mongodb Output Plugin
---------------------
The out_mongodb output plugin allows gofluent to send message to mongodb.

Example Configuration

out_mongodb is included in gofluent's core. No additional installation process is required.
```
<match mongodb.**>
  type mongodb
  host localhost
  port 27017
  capped on
  capped_size 1024
  database test
  collection test
  user test
  password test
</match>
```
*type(required)* 
The value must be mongodb.

*host (required)*
The output target host ip.

*port (required)*
The output target host port.

*capped (highly recommended)*
To create a capped collection.

*capped_size*
Specify the actual size(MB) of the capped collection.

*database(required)*
The name of the database to be created or used.

*collection(required)*
The name of the collection to be created or used.

*user(highly recommended)*
The user name to login the database.

*password(highly recommended)*
The password to login the database.
