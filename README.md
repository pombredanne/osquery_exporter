# osquery_exporter
Exporter for exporting osquery (https://osquery.io) query results to prometheus

## Installation
Install osquery from https://osquery.io/downloads/

Build with go1.7
```
go get github.com/zwopir/osquery_exporter
go build
```

## Configuration
The exporter can be configured via configuration file and commandline parameters.
The configuration file is mandatory, whereas the commandline parameters are optional and have resonable default values

The configuration file (YAML) defines the queries that are run via osqueryi.

```
---
runtime:
  # osqueryi binary. Looked up in PATH if not specified as absolute path
  osquery: "osqueryi"
  # timeout for a single call to osqueryi
  timeout: 10s

metrics:
  counters:
    # a list of counter definitions
    [ - <counter definition> ... ]  
  gauges:
    # a list of gauge definitions
    [ - <gauge definition> ... ]  
  countervecs:
    # a list of countervec definitions
    [ - <countervec definition> ... ]  
  gaugevecs:
    # a list of gaugevec definitions
    [ - <gaugevec definition> ... ]  
```
There are four types of metrics, that can be exported:

### counter and gauges
Counter and gauges are defined as an osquery query that returns a single line with a single element.
Typical queries are ```select count(*) as c from <table>;```. The resulting column must me named and referenced in the metric definition:

```
# name of the metric. Directly exported to prometheus (but prefixed with osquery_exporter_).
name: history_lines_count
# metric help
help: "number of entries in the history"
# the query to be executed via osqueryi
query: "select count(*) as count from shell_history"
# reference to the name of the result column
valueidentifier: count
```

It's up to the user to decide if the osquery query result is a counter or gauge. Further information about metric types and labeling recommendations can be found on
- https://prometheus.io/docs/concepts/metric_types/
- https://prometheus.io/docs/practices/naming/

### countervecs and gaugevecs
counter- and gaugevecs are analog counters and gauges, but the query result can (and should) consist of more than one result set.
A single result set must contain label columns which are referenced in the metric definition:

```
name: users_by_shell
help: "number of users by login shell"
query:  select count(*) as count, shell from users group by shell;
valueidentifier: count
labelidentifier:
  - shell
```
