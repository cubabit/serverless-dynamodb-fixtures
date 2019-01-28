[![serverless](http://public.serverless.com/badges/v3.svg)](http://www.serverless.com)
[![npm version](https://badge.fury.io/js/serverless-dynamodb-fixtures.svg)](https://badge.fury.io/js/serverless-dynamodb-fixtures)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

# serverless-dynamodb-fixtures
Serverless plugin to load data on DynamoDB tables.

## This Plugin Requires
* [Serverless](https://serverless.com/framework) version > 1.0.0. It has been tested with v1.36.3.
* Native support of promises: ECMAScript6 - Node 4.9.1

## Usage

You can use this plugin in two ways:
* Via CLI: executing `sls fixtures` from your CLI.
* Via deployment hook: after deploying your service (`sls deploy`). A hook is linked in this point (after deploying).

See the configuration flag `enable`, for each fixture block, for more information.

## Configuration

You should include the plugin as dev dependency, and in your serverless.yml file. Then, as a custom variable, you should include a `fixtures` variable with the configuration of the plugin.

The accepted configuration is an array of fixtures, where each fixture has the following variables:
* **table**: (String) Name of the DynamoDB table where you want to load your fixtures.
* **enable**: (String or boolean) *Optional*. This flag allows us to enable/disable the load of these fixtures. Accepted values:
  * 'cli': enable the load of these fixtures only if the plugin was launched via CLI (`sls fixtures`). This is the *default value*.
  * 'deploy': enable the load of these fixtures only if the plugin was launched via deployment hook (`sls deploy`).
  * true: enable the load of these fixtures independently of how the plugin was launched.
  * false: disable the load of these fixtures independently of how the plugin was launched.
* **sources**: (Array) List of relative file paths where your data is, in a valid format for the function [batchWrite](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB/DocumentClient.html#batchWrite-property) of [DynamoDB.DocumentClient](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB/DocumentClient.html).
* **rawsources**: (Array) List of relative file paths where your data is, in a valid format for the function [batchWriteItem](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB.html#batchWriteItem-property) of [DynamoDB](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB.html).
* **stage**: (String) *Optional*. The name of the stage where you want to load these fixtures. If this variable is not set, the fixtures will be loaded for every stage.
* **concurrentWrites**: (Positive integer) *Optional*. The maximum number of concurrent writes sent to DynamoDB per source. DynamoDB limits to 25 the number of sent items per request, so we have to internally chunck the sources, but at the same time we can send concurrently these chuncks. With this variable you can control this aspect. The default value is 5.
* **autogeneratedId**: (String) *Optional*. Name of a key attribute in the table that this plugin should autogenate for you. Any attribute with the same name in the provided data files will be overwritten.

**Important**: ``sources`` **or** ``rawsources`` must be defined (at least one of them).

## Examples

### Serverless configuration

```
plugins:
  - serverless-dynamodb-fixtures

custom:
  fixtures:
    - table: TABLE1-${self:custom.stage}
      [enable: true]
      sources:
        - ./file1-${self:custom.stage}.yml
        - ./file2-${self:custom.stage}.json
      rawsources:
        - ./rawFormatFile1-${self:custom.stage}.yml

    - table: TABLE2-${self:custom.stage}
      [stage: test]
      [concurrentWrites: 5]
      sources:
        - ./file3-${self:custom.stage}.yml

```

#### Autogeneration of id

**`serverless.yml`**

```
plugins:
  - serverless-dynamodb-fixtures

custom:
  fixtures:
    - table: TABLE1-${self:custom.stage}
      autogeneratedId: id
      sources:
        - ./file.yml
```

**`file.yml`**

```
- name: Jack London
- name: John Doe
```

It will be loaded in DynamoDB as follow:

```
- id: 45745c60-7b1a-11e8-9c9c-2d42b21b1a3e
  name: Jack London
- id:  35625c60-8b1f-22e9-1c3c-1ae1c16a2f4e
  name: John Doe
```

### Fixtures

#### Source in yml format
```
- id: 1
  name: Jack London
- id: 2
  name: John Doe
```

#### Source in json format
```
[
    {"id":1, "name":"Jack London"},
    {"id":2, "name":"John Doe"}
]
```

# Thanks
* How to develop the seeder: https://github.com/99xt/serverless-dynamodb-local
* Original idea: https://github.com/marc-hughes/serverless-dynamodb-fixtures-plugin
