# APIs

Clients interact with AVA through APIs calls to nodes.

Numeric parameters in API calls may be given as strings (e.g. `"5"` or `5` are both ok for an integer argument).
Numeric return values are always given as strings (e.g. `"5"` rather than `5`).

In examples, API calls are made to a node listening for HTTP traffic on `127.0.0.1:9650`.

## [AVM (X-Chain) API](./avm.md)

The X-Chain, AVA's native platform for creating and trading assets, is an instance of the AVA Virtual Machine (AVM).
This API allows clients to create and trade assets, including AVA, on the X-Chain as well as other instances of the AVM.

## [EVM (C-Chain) API](./evm.md)

Allows clients to interact with the C-Chain, AVA's main EVM instance, as well as other EVM instances.

## [P-Chain API](./platform.md)

Allows clients to interact with the P-Chain (Platform Chain), which maintains AVA's validator set and handles blockchain creation.

## [Keystore API](./keystore.md) 

Allows clients to use an AVA node's built-in keystore.

## [Health API](./health.md)

Allows client to check a node's health.

## [Metrics API](./metrics.md) 

Allows clients to collect Prometheus metrics about a node.

## [Admin API](./admin.md)

Allows clients to examine a node's internal state.

## [Info API](./info.md)

Allows clients to examine basic information about a node.

## [Timestamp API](./timestamp.md)

Allows clients to interact with the Timestamp Chain, a simple timestamp server.