# Changelog

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