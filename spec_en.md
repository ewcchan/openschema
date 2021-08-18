<H2> I. Overview

This topic describes OpenSchema metadata and interaction modes.




## II. Compatibility Design

| Compatibility Settings | Description |
| ------------------- | ------------------------------------------------------------ |
| BACKWARD (default) | The consumer can use the new schema to read data sent by the producer using the latest schema.|
| BACKWARD_TRANSITIVE | The consumer can use the new schema to read data sent by the producer using all previously registered schemas.|
| Forward | A consumer can use the latest schema to read data sent by the producer that uses the latest schema.|
|forward_TRANSITIVE | The consumer can use all registered schemas to read data sent by the producer using the latest schema.|
| FULL | The new schema is backward and forward compatible with the newly registered schema.|
| FULL_TRANSITIVE | The new schema is backward and forward compatible with all previously registered schemas |
| NONE | Disables mode compatibility checking.|



## III. Content-Types

The OpenSchema REST server communicates using HTTP+JSON.

The request SHOULD specify the most specific format and version information via the HTTP Accept header, and MAY include several weighting preferences:

> Accept:application/vnd.openschema.v1+json



## IV. ErrorCode

The HTTP response of all requests is consistent with the HTTP standard. The detailed error code is determined by the returned JSON character string. The format is as follows:

```json
{
"error_code": 422,
"error_message": "schema info cannot be empty"
}
```



## V. Schema Format

### 5.1 Meta Information

Meta-information is called subject.

|MetaInfo|Meaning|Example|
| ------------- | ------------------------ | ------------------------------- |
| tenant | Tenant | org/apache/rocketmq/mybank |
| namespace | Namespace | Cluster name, for example, rocketmq-cluster |
| subject | Metadata name | For example, use the topic name as the metadata name. |
| app | Application deployment unit of the service provider | |
| description | Description | Provided by the applicant |
| status | Metadata status | For example, released or obsolete |
| Compatibility | Compatibility policy | None, forward compatibility, backward compatibility, and full compatibility|
| Coordinate | Maven coordinate | Maven coordinate of the JAR of the message payload|
| schema | Data format | Associated data format description. For details, see the following table. |

### 5.2 Schema Definition

The Payload Schema is used to describe the payload data of a message.

|MetaInfo|Meaning|Instance|
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| name | Payload name, which can be null. For example, the payload of a message does not need a name. | |
| id | Globally unique identifier, which is used to identify the schema | |
| comment | Payload comment | |
| Serialization | Serialization mode: Hissian, JSON, PB, AVRO, and user-defined | |
| schemaType | Enumeration of schema types: NONE, JSON, PB, AVRO, USER-DEFINED, Int, Long, String, and Map | If no schema is provided in a message, the schema type is NONE. You can also add a schema to the current message. For example, you can use PB to describe the format of the data transmitted by the RocketMQ.
| schemaDefinition | Schema content, which is used to describe the data format. | NONE: none PB: PB description file AVRO: AVRO schema content USER-DEFINED: user-defined information Basic content type: none |
| Validator | Value validator | Validates the values of objects described in the schema.|
| version | Schema version | For example, the payload may change. In this case, the version is required to identify different schemas. |

Example:

```json
{
"subject": "test-topic",
"namespace": "org.apache.rocketmq",
"tenant": "messaging/rocketmq",
"app": "rocketmq",
"description": "rocketmq user infomation",
"compatibility": "NONE",
"validator": "a.groovy",
"comment": "Rocketmq user infomation",
"schemaType": "AVRO",
"schemaDefinition": [{
"name": "id",
"type": "string"
},
{
"name": "age",
"type": "short"
}
]
}
```



## VI. Mapping Between Subjects and Topics

### 6.1 Relationship Between Subjects and Topics

- Message system


Subject Name The default value is a topic name, which defines the format of the message body. The value can be extended by suffix ${topic}-${suffix}. For example, in Kafka, Kafka-Key is generally used to define the data format of the key in Kafka messages.

- Other systems


The user is responsible for the interpretation.

### 6.2 Relationship Between Subjects and Schemas

- A subject contains schemas of multiple versions, and the relationship is 1:N.

- Compatibility settings are provided at the subject level. Multiple schemas evolve based on the settings.

- The globally unique ID is defined in a schema. You can find a unique schema based on the ID. The relationship is 1:1.

- Subject+version can also uniquely locate a schema.




## VII. REST Interface Definition

- **Common Parameter**

| | Parameter name | Parameter type | Mandatory or not | Parameter description |
| ------------ | ------------- | -------- | -------- | -------- |
| Request public parameter | tenant | string | Optional | Tenant |
| | namespace | string | Optional | Namespace |
| Return public parameter | error_code | int | Mandatory | Error code |
| | error_message | string | Mandatory | Error explanation |

- **Version Rules**

By default, the schema version number increases in ascending order. You can use the latest version number to obtain the latest schema version. However, a new schema version may be generated when the latest version number is obtained.

For example, the following request is used to obtain the latest schema definition under test-value:

```sh
curl -X GET http://localhost:8081/subjects/test-value/versions/latest/schema
```



### 7.1 Schema-related APIs

#### 7.1. 1 Obtaining Schema Details by ID

- URL


GET /schemas/ids/{string: id}

- Request parameters


|Parameter name|Parameter type|Mandatory or not|Parameter description|
| -------- | -------- | -------- | ---------------- |
| id | string | Unique ID of a | schema |

- Response parameters


|Parameter name|Parameter type|Mandatory or not|Parameter description|
| -------- | -------- | -------- | -------------------- |
| schema | JSON | No | Return the specific schema definition |

- Error code.

401:

40101 - Unauthorized Error

404:

40401: The corresponding schema information does not exist.

500:

50001 - Storage Service Error

- Sample request


```shell
curl -X GET http://localhost:8081/schema/ids/1
```

- Response Example


```json
{
"version": 1,
"id": "20",
"serialization": "PB",
"schemaType": "AVRO",
"schemaDefinition": [{
"name": "id",
"type": "string"
},
{
"name": "age",
"type": "short"
}
],
"validator": "a.groovy",
"comment": "user information"
}
```



#### 7.1. 2 Obtaining the Subject and Version Number Based on the ID

- URL


​ GET /schemas/ids/{string: id}/subjects

- Request parameters

|Parameter name|Parameter type|Mandatory or not|Parameter description|
| -------- | -------- | -------- | ---------------- |
| id | string | Unique ID of a | schema |

- Response parameters


| Parameter Name| Parameter Type| Parameter Description|
| -------- | -------- | -------- |
| subject | string | Subject name |
| version | int | Version |

- Error code.

401:

40101 - Unauthorized Error

404:

40401: The corresponding schema information does not exist.

500:

50001 - Storage Service Error

- Sample request


```shell
curl -X GET http://localhost:8081/schemas/ids/1/versions
```

- Response Example


```json
[{"subject":"test-topic","version":1}]
```



### 7.2 Subject-related Interfaces

#### 7.2. 1 Obtaining All Subjects

- URL


GET /subjects

- Request parameters

None

- Response parameters


| Parameter Name| Parameter Type| Parameter Description|
| -------- | --------- | --------------- |
| name | JsonArray | Subject name list |

- Error code.

401:

40101 - Unauthorized Error

500:

50001 - Storage Service Error

- Sample request


```shell
curl -X GET http://localhost:8081/subjects
```

- Response Example


```json
["subject1", "subject2"]
```



#### 7.2. 2 Obtaining All Versions of a Subject

- URL


GET /subjects/(string: subject)/versions

- Request parameters

|Parameter name|Parameter type|Mandatory or not|Parameter description|
| -------- | -------- | -------- | ----------- |
| subject | string | Mandatory | Subject name |

- Response parameters


| Parameter Name| Parameter Type| Parameter Description|
| -------- | -------- | -------- |
| version | int | Version number |

- Error code.

401:

40101 - Unauthorized Error

404:

40401: The corresponding openschema information does not exist.

500:

50001 - Storage Service Error

- Sample request


```shell
curl -X GET http://localhost:8081/subjects/test-value/versions
```

- Response Example


```json
[1, 2, 3, 4]
```



#### 7.2. 3 Delete the subject and its compatibility settings.

- URL


DELETE /subjects/(string: subject)

- Request parameters

|Parameter name|Parameter type|Mandatory or not|Parameter description|
| -------- | -------- | -------- | ----------- |
| subject | string | Mandatory | Subject name |

- Response parameters


| Parameter Name| Parameter Type| Parameter Description|
| -------- | -------- | -------- |
| version | int | Version number |

- Error code.

401:

40101 - Unauthorized Error

404:

40401: The corresponding openschema information does not exist.

500:

50001 - Storage Service Error

- Sample request


```shell
curl -X DELETE http://localhost:8081/subjects/test-value
```

- Response Example


```json
[1, 2, 3, 4]
```



#### 7.2. 4 Obtaining Subject Definitions

- URL


GET /subjects/(string: subject)

- Request parameters

|Parameter name|Parameter type|Mandatory or not|Parameter description|
| -------- | -------- | -------- | ----------- |
| subject | string | Mandatory | Subject name |

- Response parameters


| Parameter Name| Parameter Type| Parameter Description|
| ------------- | -------- | ---------------------- |
| subject | string | subject name Subject name |
| namespace | string | Namespace |
| tenant | string | Tenant |
| app | string | Application |
| compatibility | string | Compatibility setting |
| coordinate | string | coordinate |
| status | string | Status |
| description | string | description |

- Error code.

401:

40101 - Unauthorized Error

404:

40401: The corresponding openschema information does not exist.

500:

50001 - Storage Service Error

- Sample request


```shell
curl -X GET http://localhost:8081/subjects/test-value
```

- Response Example


```json
{
"subject": "test-topic",
"namespace": "org.apache.rocketmq",
"tenant": "messaging/rocketmq",
"app": "rocketmq",
"description": "JSON",
"compatibility": "NONE"
}
```
#### 7.2. 5 Obtaining Schema Definitions Based on Subject and Schema Version

- URL


​ GET /subjects/(string: subject)/versions/(version: version)/schema

- Request parameters

|Parameter name|Parameter type|Mandatory or not|Parameter description|
| -------- | -------- | -------- | ------------ |
| subject | string | Mandatory | Subject name |
| version | int | Mandatory | Schema version number |

- Response parameters


| Parameter Name| Parameter Type| Parameter Description|
| ------------- | -------- | ---------------------- |
| subject | string | subject name Subject name |
| namespace | string | Namespace |
| tenant | string | Tenant |
| app | string | Application |
| compatibility | string | Compatibility setting |
| coordinate | string | coordinate |
| status | string | Status |
| description | string | description |

- Error code.

401:

40101 - Unauthorized Error

404:

40401: The corresponding openschema information does not exist.

40402 - The version does not exist.

500:

50001 - Storage Service Error

- Sample request


```shell
curl -X GET http://localhost:8081/subjects/test-value/versions/1/schema
```

- Response Example


```json
{
"subject": "test-topic",
"namespace": "org.apache.rocketmq",
"tenant": "messaging/rocketmq",
"app": "rocketmq",
"description": "rocketmq user information",
"compatibility": "NONE",
"schema": {
"version": 1,
"id": "20",
"serialization": "PB",
"schemaType": "AVRO",
"schemaDefinition": [{
"name": "id",
"type": "string"
}, {
"name": "amount",
"type": "double"
}],
"validator": "a.groovy",
"comment": "rocketmq user information"
}
}
```



#### 7.2. 6 Checking and Registering Schemas

If the same definition exists, the original ID is returned.

If no, check the compatibility settings, create a new schema, and return the new ID.

- URL


POST /subjects/(string: subject)/versions

- Request parameters

|Parameter name|Parameter type|Mandatory or not|Parameter description|
| -------- | -------- | -------- | -------------- |
| subject | string | Mandatory | Subject name |
| schema | JSON | Mandatory | Refer to the schema definition |

- Response parameters


| Parameter Name| Parameter Type| Parameter Description|
| -------- | -------- | --------- |
| id | string | schema ID |

- Error code.

401:

40101 - Unauthorized Error

409:

40901 - Compatibility Error

422:

42201 - Incorrect format

500:

50001 - Storage Service Error

50002 - Timeout

- Sample request


```shell
curl -X POST -H "Content-Type: application/vnd.openschema.v1+json" \
http://localhost:8081/subjects/test-value/versions --data'
{
"serialization": "PB",
"schemaType": "AVRO",
"schemaDefinition": [{
"name": "id",
"type": "string"
}, {
"name": "amount",
"type": "double"
}]
}'
```

- Response Example


```json
{id":"10"}
```



#### 7.2. 7 Adding and Modifying a Subject

If no related subject exists, add a subject.

If yes, modify the related attributes.

- URL


POST /subjects/(string: subject)/

- Request parameters

|Parameter name|Parameter type|Mandatory or not|Parameter description|
| ------------- | -------- | -------- | ----------- |
| tenant | string | Mandatory | Tenant |
| namespace | string | Mandatory | Namespace |
| subject | string | Mandatory | Subject name |
| app | string | | Home app |
| description | string | | description |
| status | string | Mandatory | Status |
| compatibility | string | | Compatibility policy |
| coordinate | string | | Maven coordinate |

- Response parameters

| Parameter Name| Parameter Type| Parameter Description|
| ------------- | -------- | ----------- |
| tenant | string | Tenant |
| namespace | string | Namespace |
| subject | string | subject name |
| app | string | Home app |
| description | string | description |
| status | string | Status |
| compatibility | string | Compatibility policy |
| coordinate | string | Maven coordinate |

- Error code.

401:

40101 - Unauthorized Error

422:

42201 - Incorrect format

500:

50001 - Storage Service Error

50002 - Timeout

- Sample request


```shell
curl -X POST -H "Content-Type: application/vnd.openschema.v1+json" \
http://localhost:8081/subjects/test-value/ --data'
{
"subject": "test-topic",
"namespace": "org.apache.rocketmq",
"tenant": "messaging/rocketmq",
"app": "rocketmq",
"description": "rocketmq user information",
"compatibility": "NONE",
"status": "deprecated"
}
'
```

- Response Example


```json
{
"subject": "test-topic",
"namespace": "org.apache.rocketmq",
"tenant": "messaging/rocketmq",
"app": "rocketmq",
"description": "rocketmq user information",
"compatibility": "NONE",
"status": "deprecated"
}
```



#### 7.2. 8 Deleting a Schema of a Specified Subject of a Specified Version

- URL


​ DELETE /subjects/(string: subject)/versions/(version: version)

- Request parameters

|Parameter name|Parameter type|Mandatory or not|Parameter description|
| -------- | -------- | -------- | ----------- |
| subject | string | Mandatory | Subject name |
| version | int | Mandatory | Version number |

- Response parameters

| Parameter Name| Parameter Type| Parameter Description|
| -------- | -------- | -------- |
| version | int | Version number |

- Error code.

401:

40101 - Unauthorized Error

404:

40401: The subject does not exist.

40402-The version does not exist.

500:

50001 - Storage Service Error

- Sample request


```shell
curl -X DELETE http://localhost:8081/subjects/test-value/versions/1
```

- Response Example


```json
1
```



### 7.3 Compatibility-Related Interfaces

#### 7.3. 1 Testing Compatibility

- URL


​ POST /compatibility/subjects/(string: subject)/versions/(version: version)

- Request parameters

|Parameter name|Parameter type|Mandatory or not|Parameter description|
| -------- | -------- | -------- | ----------- |
| subject | string | Mandatory | Subject name |
| version | int | Mandatory | Version number |
| schema | json | mandatory | |

- Response parameters

| Parameter Name| Parameter Type| Parameter Description|
| ------------- | -------- | -------- |
| is_compatible | boolean | Compatible |

- Error code.

401:

40101 - Unauthorized Error

404:

40401: The subject does not exist.

40402-The version does not exist.

422: The format is incorrect.

42201: Schema format error

42202: The version format is incorrect.

500:

50001 - Storage Service Error

- Sample request


```shell
curl -X POST -H "Content-Type: application/vnd.openschema.v1+json" \
--data'{"schema": "{"type": "string"}"}'\
http://localhost:8081/compatibility/subjects/test-value/versions/latest
```

- Response Example


```json
{"is_compatible": true}
```



#### 7.3. 2 Obtaining Compatibility Configurations

- URL


GET /config/(string: subject)

- Request parameters

|Parameter name|Parameter type|Mandatory or not|Parameter description|
| -------- | -------- | -------- | ----------- |
| subject | string | Mandatory | Subject name |

- Response parameters

| Parameter Name| Parameter Type| Parameter Description|
| ------------- | -------- | -------- |
| Compatibility | string | Compatibility |

- Error code.

401:

40101 - Unauthorized Error

404:

40401: The subject does not exist.

500:

50001 - Storage Service Error

- Sample request


```shell
curl -X GET -H "Content-Type: application/vnd.openschema.v1+json" \
http://localhost:8081/config/test-value
```

- Response Example


```json
{"compatibility": "FULL"}
```



#### 7.3.3 Compatibility Configuration Update

- URL


PUT /config/(string: subject)

- Request parameters

|Parameter name|Parameter type|Mandatory or not|Parameter description|
| ------------- | -------- | -------- | ----------- |
| subject | string | Mandatory | Subject name |
| compatibility | string | | Compatibility |

- Response parameters

| Parameter Name| Parameter Type| Parameter Description|
| ------------- | -------- | -------- |
| compatibility | string | compatibility |

- Error code.

401:

40101 - Unauthorized Error

404:

40401: The subject does not exist.

422:

42201 - Compatibility format error

500:

50001 - Storage Service Error

- Sample request


```shell
curl -X PUT -H "Content-Type: application/vnd.openschema.v1+json" \
--data '{"compatibility": "NONE"}'\
http://localhost:8081/config/test-value
```

- Response Example


```json
{"compatibility": "NONE"}
```
