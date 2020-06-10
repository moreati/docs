# Info API

This API can be used to access basic information about the node.

## Format

This API uses the `json 2.0` RPC format. For more information on making JSON RPC calls, see [here.](./issuing-api-calls.md)

## Endpoint

```http
/ext/info
```

## API Methods

### info.getNodeID
Get the ID of this node.

#### Signature 
```go
info.getNodeID() -> {nodeID: string}
```

#### Example Call
```json
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"info.getNodeID"
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/info
```

#### Example Response

```json
{
    "jsonrpc": "2.0",
    "result": {
        "nodeID": "5mb46qkSBj81k9g9e4VFjGGSbaaSLFRzD"
    },
    "id": 1
}
```

### info.peers
Get description of peer connections.

#### Signature 
```go
info.peers() -> {peers:[]{
    ip: string,
    publicIP: string,
    id: string,
    version: string,
    lastSent: string,
    lastRecevied: string
}}
```

#### Example Call
```json
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"info.peers"
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/info
```

#### Example Response

```json
{
    "jsonrpc":"2.0",
    "id"     :1,
    "result" :{
        "peers":[
          {
             "ip":"206.189.137.87:9651",
             "publicIP":"206.189.137.87:9651",
             "id":"8PYXX47kqLDe2wD4oPbvRRchcnSzMA4J4",
             "version":"avalanche/0.5.0",
             "lastSent":"2020-06-01T15:23:02Z",
             "lastReceived":"2020-06-01T15:22:57Z"
          },
          {
             "ip":"158.255.67.151:9651",
             "publicIP":"158.255.67.151:9651",
             "id":"C14fr1n8EYNKyDfYixJ3rxSAVqTY3a8BP",
             "version":"avalanche/0.5.0",
             "lastSent":"2020-06-01T15:23:02Z",
             "lastReceived":"2020-06-01T15:22:34Z"
          },
          {
             "ip":"83.42.13.44:9651",
             "publicIP":"83.42.13.44:9651",
             "id":"LPbcSMGJ4yocxYxvS2kBJ6umWeeFbctYZ",
             "version":"avalanche/0.5.0",
             "lastSent":"2020-06-01T15:23:02Z",
             "lastReceived":"2020-06-01T15:22:55Z"
          }
        ]
    }
}
```

### info.getNetworkID

Get the ID of the network this node is participating in.

#### Signature 
```go
info.getNetworkID() -> {networkID:int}
```

#### Example Call
```json
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"info.getNetworkID"
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/info
```

#### Example Response

```json
{
    "jsonrpc":"2.0",
    "id"     :1,
    "result" :{
        "networkID":"2"
    }
}
```

### info.getBlockchainID

Given a blockchain's alias, get its ID. (See `avm.aliasChain` for more context.)

#### Signature 
```go
info.getBlockchainID({alias:string}) -> {blockchainID:string}
```

#### Example Call
```json
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"info.getBlockchainID",
    "params": {
        "alias":"X"
    }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/info
```

#### Example Response

```json
{
    "jsonrpc":"2.0",
    "id"     :1,
    "result" :{
        "blockchainID":"sV6o671RtkGBcno1FiaDbVcFv2sG5aVXMZYzKdP4VQAWmJQnM"
    }
}
```
