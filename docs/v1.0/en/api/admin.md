# Admin API

This API can be used for measuring node health and debugging.

## Format

This API uses the `json 2.0` RPC format. For more information on making JSON RPC calls, see [here.](./issuing-api-calls.md)

## Endpoint

```http
/ext/admin
```

### admin.alias

Assign an API an alias, a different endpoint for the API.
The original endpoint will still work.
This change only affects this node; other nodes will not know about this alias.

#### Signature 
```go
admin.alias(endpoint:string, alias:string) -> {success:bool}
```

* `endpoint` is the original endpoint of the API. `endpoint` should only include the part of the endpoint after `/ext/`.
* The API being aliased can now be called at `ext/alias`.

#### Example Call
```json
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"admin.alias",
    "params": {
        "alias":"myAlias",
        "endpoint":"bc/x"
    }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/admin
```

#### Example Response

```json
{
    "jsonrpc":"2.0",
    "id"     :1,
    "result" :{
        "success":true
    }
}
```

Now, calls to the X-Chain can be made to either `/ext/bc/X` or, equivalently, to `/ext/myAlias`.

### admin.aliasChain

Give a blockchain an alias, a different name that can be used any place the blockchain's ID is used.

#### Signature 
```go
admin.aliasChain(
    {
        chain:string,
        alias:string
    }
) -> {success:bool}
```

* `chain` is the blockchain's ID.
* `alias` can now be used in place of the blockchain's ID (in API endpoints, for example.)

#### Example Call
```json
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"admin.aliasChain",
    "params": {
        "chain":"sV6o671RtkGBcno1FiaDbVcFv2sG5aVXMZYzKdP4VQAWmJQnM",
        "alias":"myBlockchainAlias"
    }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/admin
```

#### Example Response

```json
{
    "jsonrpc":"2.0",
    "id"     :1,
    "result" :{
        "success":true
    }
}
```

Now, instead of interacting with the blockchain whose ID is `sV6o671RtkGBcno1FiaDbVcFv2sG5aVXMZYzKdP4VQAWmJQnM` by making API calls to `/ext/bc/sV6o671RtkGBcno1FiaDbVcFv2sG5aVXMZYzKdP4VQAWmJQnM`, one can also make calls to `ext/bc/myBlockchainAlias`.

### admin.startCPUProfiler

Start profiling the CPU utilization of the node. Will write the profile to the specified file on stop.

#### Signature
```go
admin.startCPUProfiler({fileName:string}) -> {success:bool}
```

where `fileName` is the name of the file to write the profile to.

#### Example Call

```json
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"admin.startCPUProfiler",
    "params" :{
        "fileName":"cpu.profile"
    }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/admin
```

#### Example Response
```json
{
    "jsonrpc":"2.0",
    "id"     :1,
    "result" :{
        "success":true
    }
}
```

### admin.stopCPUProfiler
Stop the CPU profile that was previously started.

#### Signature
```go
admin.stopCPUProfiler() -> {success:bool}
```

#### Example Call
```json
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"admin.stopCPUProfiler"
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/admin
```

#### Example Response
```json
{
    "jsonrpc":"2.0",
    "id"     :1,
    "result" :{
        "success":true
    }
}
```

### admin.memoryProfile
Dump the current memory footprint of the node to the specified file.

#### Signature
```go
admin.memoryProfile({fileName:string}) -> {success:bool}
```

where `fileName` is the name of the file to dump the information into.

#### Example Call

```json
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"admin.memoryProfile",
    "params" :{
        "fileName":"mem.profile"
    }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/admin
```

#### Example Response
```json
{
    "jsonrpc":"2.0",
    "id"     :1,
    "result" :{
        "success":true
    }
}
```

### admin.lockProfile
Dump the mutex statistics of the node to the specified file.

#### Signature
```go
admin.lockProfile({fileName:string}) -> {success:bool}
```

where `fileName` is the name of the file to dump the information into.


#### Example Call

```json
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"admin.lockProfile",
    "params" :{
        "fileName":"lock.profile"
    }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/admin
```

#### Example Response

```json
{
    "jsonrpc":"2.0",
    "id"     :1,
    "result" :{
        "success":true
    }
}
```
