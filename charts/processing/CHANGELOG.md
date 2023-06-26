# Changelog

## v0.4.0 [2023-06-27]

### Changes

#### Connection Pools
`poolsize` **values in MongoDB and Redis are shared over all pipelines again. Please make sure that you increase the poolsizes to a reasonable value for the Amount of Pipelines and the load you have.** This has been changed to be in better control of the amount of connections that processing is creating to the databases.

#### Logging
Logging and Tracing has been changed. Logging and tracing can now be setup seperately. Logging is used to log system events, which do not have any hierarchical or temporal relation to each other. Tracing is used to trace a message through the system using hierarchical and temporal structures. Tracing can also connect to external systems such as Jaeger or OpenTelemetry (e.g. Elastic APM). If no `TRACER` is set, and `LOG_LEVEL` is set to "DEBUG" or "TRACE, traces will be printed to the log output. The following ENV Variables have been renamed: 

```ini
TRACER=...
# split into (see below)
LOGGER=... and 
TRACER=...

RUST_LOG=...
# renamed to
LOG_LEVEL=...
```

This means the follwing are now available:

```ini
# controls the log print output type
LOGGER=COMPACT # COMPACT is standard. ALternatives: "PRETTY" or "JSON"
# controls the log print level of LOGGER
LOG_LEVEL=INFO # or "TRACE" or "DEBUG" or "WARN" or "ERROR"

# controls the tracing output
TRACER=LOG # or "JAEGER" or "OTLP"
# controls trace level of TRACER
TRACE_LEVEL=DEBUG

# in case of OLTP use the following ENV vars to connect to the right collector
# more here: https://opentelemetry.io/docs/concepts/sdk-configuration/otlp-exporter-configuration/

OTEL_RESOURCE_ATTRIBUTES=service.name=processing
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:8200

# in case you want to use Elastic APM, you use TRACER=OTLP, and:
OTEL_RESOURCE_ATTRIBUTES=service.name=processing
OTEL_EXPORTER_OTLP_ENDPOINT=https://apm_server_url:8200
OTEL_EXPORTER_OTLP_HEADERS="Authorization=Bearer an_apm_secret_token"
OTEL_METRICS_EXPORTER="otlp"
OTEL_LOGS_EXPORTER="otlp"
# (taken from: https://www.elastic.co/guide/en/apm/guide/current/open-telemetry-direct.html#instrument-apps-otel)

# in case of JAEGER, more: https://opentelemetry-python.readthedocs.io/en/latest/exporter/jaeger/jaeger.html 
OTEL_EXPORTER_JAEGER_USER=...
OTEL_EXPORTER_JAEGER_PASSWORD=...
OTEL_EXPORTER_JAEGER_ENDPOINT=...
OTEL_EXPORTER_JAEGER_AGENT_PORT=...
OTEL_EXPORTER_JAEGER_AGENT_HOST=...
OTEL_EXPORTER_JAEGER_AGENT_SPLIT_OVERSIZED_BATCHES=...
OTEL_EXPORTER_JAEGER_TIMEOUT=...
```

### Fixed
- In logs, the printed pre-calculated ObjectID of a Message for Mongo was wrong.
- Freshdesk #28: In case a Assignment fails, the following assignments will still be computed (important for Formulas, Presets and Rule Conclusions)
- Freshdesk #34: disabled Rules will not be evaluated anymore 


## v0.3.1 [2023-06-12] HOTFIX

### Fixed
- on high load, the new tracing logging mechanism can produce redundant traces including different traceIds. This has been fixed


## v0.3.0 [2023-06-12]

### Added

- The behaviour on SLAs has been redesigned. Instead of a single `timeout` value which immediately interrupts the message computation and returns the result "until timout", there are two timeouts.
  - `timeout` will NOT interrupt the message computation, but it will return early from computation with the results "until timeout", including the SLAs. The message computation instead will still continue. A log on level WARN will be written.
  - `hardTimeout` will interrupt message computation immediately and write an log on level ERROR. With the boolean option `"hardTimeoutPrintMessage": true`, processing will write state of the message "until timeout" to the log. Please make sure that you will not write sensitive data to the log if you use this option
- `traceidDataField` has been introduced on pipeline level to define a DataField where a traceId is transmitted with the message. This traceId will be used to be shown in the logs to identify log entries with specific messages. If no traceidDataField is set, the MongoDB ObjectID of the record will be used.
- Logs now contain information about the current pipeline (pipelineName), stage (stageName) and compute (computeName), as well as a traceId.
- `output` Fields in Rule-Conclusions, Presets, Formulas etc. now support nested values with dot-notation. This means that `"output": "Response.SomeNumeric"` is equal to the combination of `outputDataField: "Response" / output: "SomeNumeric"`. Please keep this in mind, because `outputDataField: "Response" / output: "Response.SomeNumeric"` would write the value to `Response.Response.SomeNumeric`.
- function `timestamp_with_nanos_to_isostr(seconds, nanos)` has been added to get a ISO Datetime string from seconds-based UNIX-timestamp together with nanoseconds value.
- function `timestamp_ms_to_isostr` has been added to get a ISO Datetime string from a milliseconds UNIX-Timestamp.
- function `str_replace(source, from, to)` has been added to provide a replace function which can be used in assignments. `str_replace("ym string", "ym", "my") => "my string"`
- function `str_trim(source)` has been added to provide a trim function which can be used in assignments. `str_trim(" aa   ") => "aa"`

### Fixed
- A fix for [https://fraudaverse.freshdesk.com/a/tickets/33](Freshdesk Ticket #33) has been implemented. In Rule-Conclusions, Presets, Formulas etc, outputs of prior assignments are now available in the expression fields of the next assignments. Please keep in mind that you have to give the full path. This means that if a Ruleset has the `outputDataField: "Response"` and the assignment `output: "SomeNumeric"`, in the expression of the next assignment you have to reference it by `"expression": "Response.SomeNumeric + 1000"`.


## v0.2.4

### Changed

- The Environment Variables `USE_CFG_WORKER` and `CFG_WORKER_THREADS` are not functional anymore. Instead, if a pipeline is supposed to use it's own worker threads, you can specify `"threads": 16` accordingly to `asyncThreads` value in the pipelines JSON to define 16 seperate worker threads for a specific pipeline. If you do not specify this key, the pipeline will not have any seperate worker threads. If threads is <= 0, processing will not use any seperate runtime for this pipeline.

### Fixes

- A default connectTimeout for MongoDB of 5ms caused the MongoDB driver to constantly reconnect and cause latency spikes. This has been set back to drivers default value 10000.
- Default Values for MongoDB Client have changed back to libraries defaults: connectTimeoutMS=10000, heartrateFreqMS=10000, directConnection=false, retryWrites=true
- The fix for [https://fraudaverse.freshdesk.com/a/tickets/33](Freshdesk Ticket #33) in v0.2.2 caused performance issues, it has been rolled back. This implies that in an array of assignments in presets, conclusions etc. the output of a prior assignment is not available in the following assignment as an input.

## v0.2.3
### Changed

- directConnections default value is "false" now.
- When connecting via "mongodb://"-string without username/password, you can still specify them as "username" and "password" fields in the datastore JSON representation
### Fixes

- Defined Risk Lists were not evaluating correctly for string types. This has been fixed.
## v0.2.2

### Changed
- Updated Rust to Toolchain version 1.69
- When connection settings in MongoDB URI Strings or Datastore Definitions are not specified, default values are used. The following values are default:
  - maxPoolSize=5 
  - minPoolSize=5 (in datastores json definition, we only define 'poolsize' because we recommend to set a fixed one)
  - maxIdleTime=0
  - heartrateFreq=360000
  - directConnection=true
  - retryWrites=false

### Fixed
- When using MongoDB URI Connection Strings (`mongodb://...`), poolsizes from JSON definitions in datastores have not been evaluated also, default application connection settings have not been set. This is the case now. The priority of values set are 'MongoDB Connection String' > 'Datastores JSON' > 'Default Values'
- [https://fraudaverse.freshdesk.com/a/tickets/15](Freshdesk Ticket #15): When using `precalculateMessageId` together with an `inputDataField` of a MongoInsert compute, the id was recalculated and did not match the precalculated one. For this, another Option `messageIdDataField` has been introduced on Pipeline level, which should be the same value as the `inputDataField` of the relevant MongoInsert compute.
- [https://fraudaverse.freshdesk.com/a/tickets/33](Freshdesk Ticket #33): In an assignment set (e.g. conclusions, presets), when using the output of a prior assignment as an input in the next assignment, it was not evaluated correctly. This has been fixed.

## v0.2.1

### Changed

* Debug output for mongo inserts is single line now
* Added debug outputs for defined risk list timing statistics

## v0.2.0

### Added

* Defined risk lists now support multiple same values with different conditions and validity periods

## v0.1.10

### Changed

* The "debug" field in pipelines is not available anymore. Instead, please use the environment variable `PIPELINE_DEBUG=1`

### Fixed

* Pipeline References in Async Stages are working now

## v0.1.9 (processing v0.1.9)

### Added

* Added a logmessage for message timeouts on loglevel `WARN`
* Added logmessages for starting computation on computes and stages, loglevel `DEBUG`
* Json Serializer for Handlebars: if you used handlebar syntax `My Object: {{some.key}}` and some.key was a object, it was converted to a string `My Object: [object]`. You can now serialize to json representation by using `json` helper with `My Object: {{json some.key}}`. This would be serialized to a string in json format: `My Object: {"inner_key_of_some_key": 5}`. Keep in mind, this will also add quotes to the serialization of a String. This means that `\"{{mystring}}\"` is equal to `{{json mystring}}`

### Changed

* On Timeout, the Response will contain **latencies** and **SystemTimestamp** as well as all computes that have been successfully computed until timeout 

### Fixed

* `computeLatenciesMicros` for stages included the microseconds of prior stages, so the user had to calculate the difference. This has been fixed and the values represent the time for each single stage.
## v0.1.8 (processing v.0.1.8)

### Added

* `timestamp_to_isostr` function to convert a unix timestamp into a DateTime string in ISO3339 format
* Group Compute supports `computeMode: Loop`. This computeMode additionally requires a key called `loopSettings`, which is an object. You can find an example in Documentation.
* `MongoUpdate` **Compute**: A compute which is able to run Updates on a MongoDB Datastore. Example in Documentation.
* `MongoQuery` **Compute**: A compute which is able to run queries on MongoDB Datastore. Example in Documentation.
* `DataFieldOperation` **Compute**: A compute which is able to effectively Create and **Delete** fields during computation. Also, there is an operation called `ObjectInsert` which can insert Values into an Object by Path or Value.
* Handlebars Notation: For some String fields in the configuration in processing, you can use handlebars notation to insert values from the current Message context, before processing will actually use the field. One example is the MongoQuery `query` and `replace` fields. `"query": "{\"Amount\": {{trx.amt}},\"Account\": {{trx.acc}}}"` will substitute `{{trx.acc}}` and `{{trx.amt}}` with the fitting values from the current Message. Currently, the following fields are supported:
    * MongoQuery: query field
    * MongoUpdate: query, update, replace
    * DataFieldOperation: Create.createPath, ObjectInsert.objectPath, ObjectInsert.insertValuePath

### Changed

* For more clarity about what they do, the following functions were renamed:
  * `date_parse_from_str` to `date_str_to_timestamp`
  * `datetime_parse_from_str` to `datetime_str_to_timestamp`
  * `datetime_ms_parse_from_str` to `datetime_str_to_timestamp_ms`
  * `date_ms_parse_from_str` to `date_str_to_timestamp_ms`

### Fixed

* System Timestamp available in Presets.
* In assignments arrays, Presets, Formulas, Rulesets, prior outputs are now available in the next assignment in the array. This has been fixed.
* Special Values support nested json objects
* If a preset fails, message computation is not aborted

## v0.1.7 (processing v0.1.7)

### Changed

* The option `saveInsertId` was removed from `MongoInsert`. Instead, use the `outputDataField` to retrieve the object ID of the inserted document. Please keep also in mind that the outputDataField will be filled with a String value. So if you want to save it as `_id` in `Response`, provide `"outputDataField": "Response._id"`, not just `Response`.
* Filters (allowlists and blocklists) have been removed
* Message computation sequence has changed. Presets are computed before special attributes such as timestamp and amount are parsed from the message:

```

                ┌─────────┐      ┌──────┐    ┌─────────┐
                │         │      │insert│    │calculate│
                │ request ├─────►│ path ├───►│ presets │
                └─────────┘      └──────┘    └────┬────┘
                                                  │
                                                  │
 ┌─────────┐    ┌──────────┐     ┌──────┐    ┌────▼──────────┐
 │ async   │    │  insert  │◄────┤stages◄────┤ parse special │
 │ stages  │◄───┤latencies │     │      │    │   attributes  │
 └─────────┘    └─────┬────┘     └──────┘    └───────────────┘
                      │
                ┌─────▼────┐
                │          │
                │ response │
                └──────────┘
```

### Added

* The `MongoInsert` compute now supports the `inputDataField` option to specify what part of the current message is stored. If not provided, the full document will be stored.
* `datetime_ms_parse_from_str(string, format)` and `date_ms_parse_from_str(string, format)` have been added to get the *milliseconds* timestamp from a string with a specific format. They work accordingly to their siblings without the `_ms_` in the name
* Environment variable `TRACER=JSON` will cause processing to log in json format. 

### Fixed

* runtime fix for crash if processing is run single threaded
* groups don't cancel if one compute goes wrong
* SystemTimestamp now available during message computation
* Locations for System Datafields (Latencies and Systemtimestamp) can now be defined freely in `systemTimestampDataField` and `latenciesDataField`, instead of implicitly using `responseDataField`


## v0.1.6 (processing v0.1.6)

### Changed

#### Nested JSON Paths

You can now use nested json paths. To do that, all computes now support a field called `outputDataField` which is a String representing a json path. All features or outputs in Formulas/Rulesets will be placed inside of this `outputDataField`. Accordingly, a pipeline has a `requestDataField` which defines where the HTTP request will be stored, as well as a `responseDataField` which enables the user to define the DataField which is used to be send as a HTTP response.

Very simplified example:
```
pipelines: [
    {
        "name": "Example Pipeline",
        "requestDataField": "Request",
        stages: [
            {
                "computes" [
                    {
                        "compute": "Formula",
                        "name": "LogarithmicAmount",
                        "outputDataField": "Response",
                        "formula": [
                            {
                                "output": "AmountCopyFromRequest",
                                "expression": "Request.Amount"
                            }
                        ]
                    },
                ]
            }
        ]
        "responseDataField": "Response"
    }
]
```

This pipeline stores the request JSON to `Request`, then uses a formula to copy the Value of `Amount` to `Response.AmountCopyFromRequest`. Then, after the message was computed, the API response is taken from the Field `Response`. For an incoming message `{"Amount": 5000}`, after computation the internal structure would look like this:

```
{
    "Request": {
        "Amount": 5000
    },
    "Response": {
        "AmountCopyFromRequest": 5000
    }
}
```

The API Response in the end would look like this:
```
{
    "AmountCopyFromRequest": 5000
    "trxLatencyMillis": 5,
    "computeLatencies": [...],
    "SystemTimestamp": ""
}
```




### Fixed

- request will be stored again (if specified with json paths)

### Changed

## v0.1.5 (processing v0.1.5)

### Changed

#### **Mongo Insert Compute**
To replace the `records` struct inside of a pipeline, we added a MongoInsert compute which now has to be manually placed into a Stage. To replicate default behaviour, this will be an seperate stage at the end of the pipeline. If you want to save the message into a database after the response is send, you need to specify MongoInsert in `asyncStages`, see point 3.

`saveInsertId` describes the key in the response message, where the insertion ID should be stored. Usually, this is `_id`.

**ATTENTION** if you use MongoInsert to store the transaction in `asyncStages`, please set the key `"precalculateMessageId": true` on pipeline-level (sibling to `asyncStages`) to `true`!

Example:
```
{
    "enabled": true,
    "compute": "MongoInsert",
    "name": "Store Payment",
    "saveInsertId": "_id",
    "datastore": {
        "database": "Payments"
    },
    "process": true
}
```

#### **Debug Compute**
To make development with a config easier, we implemented a compute called "Debug" which is able to print contents of the current Message during computation on the standard output of the process. You need to define `RUST_LOG=DEBUG` as an environment variable to see those Debug messages. A Debug compute to print `my.inner.datafield`'s value is defined like this:

```
{
    "enabled": true,
    "compute": "Debug",
    "name": "Print ",
    "process": true
    "data_field": "my.inner.datafield",
}
```

#### **Mongo Datastore Definitions**
... can how have the following collection keys to control collection handling. collectionHandling "Name" will use "collection" value to specify the collection that will be used. collectionHandling "Sharding" will use the Method defined in "collectionSharding" to specify the collection that will be used. If this is the case, the compute "MongoInsert" can specify the timestamp which is used to calculate the collection shard. Otherwise the Messages default Timestamp (if defined) or SystemTimestamp is used.

```json
{
    "name": "Datastore",
    "collection": "CollectionName",
    "collectionHandling": "Name/Sharding",
    "collectionSharding": "Daily/Monthly",
}
```

#### **Async Stages**

If you want to run computes asynchronously after the response has been send to the sending system, you can define a key `asyncStages` as a sibling to `stages`. The contents of both stages have the same structure, but asyncStages will be run after the response of the message has been returned.

**ATTENTION**: If you specify asyncStages, please also define the key `asyncThreads` on the same level to a value greater than 0 to avoid that async jobs can drain the API jobs threads.

#### **Removed elements Structure**

In pipeline, `stages`, `request` and `response` have been stored in a structure called `elements`. This was redundant and has been removed. All three keys therefore move up one level into the pipelines level.

Pipeline before:
```
{
    "name": "Pipe",
    ...
    "elements": {
        "stages": [],
        "request": [],
        "response": []
    }
}
```

Pipeline now:
```
{
    "name": "Pipe",
    ...
    "stages": [],
    "request": [],
    "response": []
}
```

### Fixed

- async processing of pipeline and groups has been optimized

## v0.1.4 (processing v0.1.4)

### Changed

- Special characters like dots, spaces, etc are not replaced anymore when stored as `Value`s in `StoredValue` computes

### Fixed

- Error when calculating the `...Since` compute methods of `StoredValue`s

### Deprecated

- Support for StoredValue in MongoDB (currently not maintained anymore)

## v0.1.3 (processing v0.1.3)

### Added

- The reference value for a `DistinctValue` compute can also be of type integer

## v0.1.2 (processing v0.1.2)

- entity and reference fields do not require to access flat JSON anymore, but can be accessed with dictionary or dot notation in nested structures, e.g. `"entity": "card['id']"` or `"entity": "card.id"`
- RuleSets do not cancel computation when a referenced field is not found in the transaction. If this is the case, the related Rule will evaluate to false and the other rules will be evaluated
- RuleSets were not maintaining internal state of values (e.g. Rule 1 increases Score and Rule 2 Decreases Score, the prior increase will be ignored).
- maxValues are now working for Distinct Values when using Redis
- key namings in redis fixed, there was a bug for the `:` placements.
- "database" and "collection" field for mongo datastores now support environment variable expansion