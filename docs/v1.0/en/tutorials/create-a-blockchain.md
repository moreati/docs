# Create a New Blockchain

## Introduction

One of the core features of the AVA network is the ability to create new blockchains.
AVA currently supports creation of new instances of the AVM (the Virtual Machine the X-Chain runs) and the Timestamp VM.

In this tutorial we'll create a new instance of the AVM.

## Create the Subnet

Every blockchain is validated by a [Subnet.](../core-concepts/overview.md#what-are-subnets)
Before you can create a blockchain, you'll need a Subnet to validate it.

[This tutorial](./create-a-subnet.md) will guide you through creating a Subnet.

You can also use a Subnet that already exists, if you have a sufficient number of its control keys.

## Add Validators to the Subnet

The Subnet needs validators in it to, well, validate blockchains.

Make sure the Subnet that will validate your blockchain has at least `snow-sample-size` validators in it.
(Recall that `snow-sample-size` is one of the [command-line arguments](../references/command-line-interface.md) when starting a node. Its default value is 20.)

[This tutorial](./adding-validators.md) will guide you through adding validators to the Subnet, if that's necessary.

## Create the Genesis Data

Each blockchain has some genesis state when it's created.
Each Virtual Machine has a static API method named `buildGenesis` that takes in a JSON representation
of a blockchain's genesis state and returns the byte representation of that state.
(This isn't true for some VMs, like the Platform VM, because we disallow creation of new instances.)

The [AVM's documentation](../api/avm.md#avmbuildgenesis) specifies that the argument to `avm.buildGenesis` should look like this:

```json
{
"genesisData":
    {
        "assetAlias1": {               // Each object defines an asset
            "name": "human readable name",
            "symbol":"AVAL",           // Symbol is between 0 and 4 characters
            "initialState": {
                "fixedCap" : [         // Choose the asset type.
                    {                  // Can be "fixedCap", "variableCap"
                        "amount":1000, // At genesis, address A has
                        "address":"A"  // 1000 units of asset
                    },
                    {
                        "amount":5000, // At genesis, address B has
                        "address":"B"  // 1000 units of asset
                    },
                    ...                // Can have many initial holders
                ]
            }
        },
        "assetAliasCanBeAnythingUnique": { // Asset alias can be used in place of assetID in calls
            "name": "human readable name", // names need not be unique
            "symbol": "AVAL",              // symbols need not be unique
            "initialState": {
                "variableCap" : [          // No units of the asset exist at genesis
                    {
                        "minters": [       // The signature of A or B can mint more of
                            "A",           // the asset.
                            "B"
                        ],
                        "threshold":1
                    },
                    {
                        "minters": [       // The signatures of 2 of A, B and C can mint
                            "A",           // more of the asset
                            "B",
                            "C"
                        ],
                        "threshold":2
                    },
                    ...                    // Can have many minter sets
                ]
            }
        },
        ...                                // Can list more assets
    }
}
```

To create the byte representation of this genesis state, call `avm.buildGenesis`.
Your call should look like the one below.
Note that this call is made to the AVM's static API endpoint, `/ext/vm/avm`.

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "id"     : 1,
    "method" : "avm.buildGenesis",
    "params" : {
        "genesisData": {
            "asset1": {
                "name": "myFixedCapAsset",
                "symbol":"MFCA",
                "initialState": {
                    "fixedCap" : [
                        {
                            "amount":100000,
                            "address": "8UeduLccQuSmYiY3fGQEyotM9uXxoHoQQ"
                        },
                        {
                            "amount":100000,
                            "address": "AgVkHvvDShLumJrzXzkwuHa7rYpewj9Kg"
                        },
                        {
                            "amount":50000,
                            "address": "AwBDGsUwNdXgVc8XG2E8A8dL3bkoVbkL9"
                        },
                        {
                            "amount":50000,
                            "address": "AATN8YjgmFjC2jQRq45sEeGcBFXNYPcM8"
                        }
                    ]
                }
            },
            "asset2": {
                "name": "myVarCapAsset",
                "symbol":"MVCA",
                "initialState": {
                    "variableCap" : [
                        {
                            "minters": [
                                "AATN8YjgmFjC2jQRq45sEeGcBFXNYPcM8",
                                "FNqMDYafoDVYQ2o4a7Zd9maJAxcUEieQb"
                            ],
                            "threshold":1
                        },
                        {
                            "minters": [
                                "JJSiKQfha9Z2TiMxBZ8XdW9F6KFw8aKS4",
                                "7jJHY1vZL6AAbCFb97KMLY8nqMQVyd5JG",
                                "58pM5cEf1wMSncPdCwtQ8tbHs2xdMA4eo"
                            ],
                            "threshold":2
                        }
                    ]
                }
            }
        }
    }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/vm/avm
```

This returns the byte representation of your blockchain's genesis state:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "bytes": "111TNWzUtHKoSvxohjyfEwE2X228ZDGBngZ4mdMUVMnVnjtnawW1b1zbAhzyAM1v6d7ECNj6DXsT7qDmhSEf3DWgXRj7ECwBX36ZXFc9tWVB2qHURoUfdDvFsBeSRqatCmj76eZQMGZDgBFRNijRhPNKUap7bCeKpHDtuCZc4YpPkd4mR84dLL2AL1b4K46eirWKMaFVjA5btYS4DnyUx5cLpAq3d35kEdNdU5zH3rTU18S4TxYV8voMPcLCTZ3h4zRsM5jW1cUzjWVvKg7uYS2oR9qXRFcgy1gwNTFZGstySuvSF7MZeZF4zSdNgC4rbY9H94RVhqe8rW7MXqMSZB6vBTB2BpgF6tNFehmYxEXwjaKRrimX91utvZe9YjgGbDr8XHsXCnXXg4ZDCjapCy4HmmRUtUoAduGNBdGVMiwE9WvVbpMFFcNfgDXGz9NiatgSnkxQALTHvGXXm8bn4CoLFzKnAtq3KwiWqHmV3GjFYeUm3m8Zee9VDfZAvDsha51acxfto1htstxYu66DWpT36YT18WSbxibZcKXa7gZrrsCwyzid8CCWw79DbaLCUiq9u47VqofG1kgxwuuyHb8NVnTgRTkQASSbj232fyG7YeX4mAvZY7a7K7yfSyzJaXdUdR7aLeCdLP6mbFDqUMrN6YEkU2X8d4Ck3T"
    },
    "id": 1
}
```

## Create the Unsigned Transaction

Now let's create the unsigned transaction to create the new blockchain.

To do so, we call [`platform.createBlockchain`](../api/platform.md#platformcreateblockchain).

Your call should look like the one below.
Of course, you'll have to change `subnetID` to the subnet that will validate your blockchain, and `payerNonce` to the next unused nonce of the account that will pay the transaction fee for this transaction.

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.createBlockchain",
    "params" : {
    	"subnetID": "KL1e8io1Zi2kr8cTXxvi321pAzfQuUa8tmBfadqpf9K2dc2TT",
        "vmID":"avm",
        "name":"My new AVM",
        "payerNonce":8,
        "genesisData": "111TNWzUtHKoSvxohjyfEwE2X228ZDGBngZ4mdMUVMnVnjtnawW1b1zbAhzyAM1v6d7ECNj6DXsT7qDmhSEf3DWgXRj7ECwBX36ZXFc9tWVB2qHURoUfdDvFsBeSRqatCmj76eZQMGZDgBFRNijRhPNKUap7bCeKpHDtuCZc4YpPkd4mR84dLL2AL1b4K46eirWKMaFVjA5btYS4DnyUx5cLpAq3d35kEdNdU5zH3rTU18S4TxYV8voMPcLCTZ3h4zRsM5jW1cUzjWVvKg7uYS2oR9qXRFcgy1gwNTFZGstySuvSF7MZeZF4zSdNgC4rbY9H94RVhqe8rW7MXqMSZB6vBTB2BpgF6tNFehmYxEXwjaKRrimX91utvZe9YjgGbDr8XHsXCnXXg4ZDCjapCy4HmmRUtUoAduGNBdGVMiwE9WvVbpMFFcNfgDXGz9NiatgSnkxQALTHvGXXm8bn4CoLFzKnAtq3KwiWqHmV3GjFYeUm3m8Zee9VDfZAvDsha51acxfto1htstxYu66DWpT36YT18WSbxibZcKXa7gZrrsCwyzid8CCWw79DbaLCUiq9u47VqofG1kgxwuuyHb8NVnTgRTkQASSbj232fyG7YeX4mAvZY7a7K7yfSyzJaXdUdR7aLeCdLP6mbFDqUMrN6YEkU2X8d4Ck3T"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The response contains the unsigned transaction:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "unsignedTx": "111KqXN6mRBbpZt1Mzmjuk4wmnBBpyWu62DWr18HUFa69DqEHdQMqAXiWnd1pzrWcBMWy69chiLc1qFoGEDxoKdas1JQJp58r8mbFJNcUfdj4QvJpdUTMSttb5HZUH9iGB1ZpdPyWepwuxFwqJgBs2qtVGtt2mSahvY1CDBQK5FVMhh85NBskgeNHDxyhxXPxz1JwAGtHma6KdE1pQKps9NX57tP1nqVtheDtSH7VkJJtNpQ8Dj1FCsVZtWm7nMGg7MK1YNEhgm2HBMV14qF1AujEn8onJWJGb5sakbgieghGFNwpyEwiBkoKSxHDoxvD9HiGdquT7froaYNsWQeJGUMJ21EpRnwffPsNN6S4zFPCdJ3MGvaLWdmfmmCm37nj4QaVQh4kEfZb7qN3txYCS2a3vvCyJ9fMDp5YnZidDx3NGWJcaMVzBBLNMD5cP54GhXdSJzJmxC2EKCtRqp62VFSoYW7iVnnQU7bw91JMkxvyW4inMj11GqmoZQhMTwU4eeDQeJy6CQrf1fYTjNEQBcy31mrqgRujJDp3LRbfMhEfQ12CZeAoyEFXRH58Jdg2pE6Ez5RaPkRUJHfXTjNYiio71RYEiByGNZs7tAbDApxPWJZzracqsDsaRMT5JmUCd5XUh9HyaXgwyLNZ6YsJLCQ9MKm4iwj2qAdWpAT1oGDYJpif2fUFNrTzNftcwsx5P9ZLjPc3FQdMTx2abiUSEyX2VqXGN4pfXCZedoozFKe9AdZV8bMerG6nYhzvE2s7wUYPwvRz1RjXgRi1aEcAmHRkLprWhBa9FHtcbQPWBpfxD7fEwTk6qLjGWPFCkgYPgB9pGbGRYW6zkECiPfzd8e1J9qFJm4FMtm1GFT9MmHHbRcRbodareyZc28YMx4JjaU3iLQ4ivkNPWBC2R27oU1L3uz1vheUHHFpMLGZZBJzbKvoXKxkydm6e"
    },
    "id": 1
}
```

## Sign the Transaction

It would be bad if anyone could create a blockchain and make a Subnet validate it without the Subnet's permission.
To that end, the transaction must be signed with *threshold* of the Subnet's control keys.
This is the same thing we do when adding a validator to a Subnet.
(As a reminder, you can find out what a Subnet's threshold and control keys are by calling [`platform.getSubnet`.](../api/platform.md#platformgetsubnet))

Suppose that the Subnet has threshold 1 and user `bob` has control key `4sEEgYu1mrFyyEQotuTBTxrRnYJ61SAAn`.
Then to sign the transaction: 

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.sign",
    "params": {
    	"tx":"111KqXN6mRBbpZt1Mzmjuk4wmnBBpyWu62DWr18HUFa69DqEHdQMqAXiWnd1pzrWcBMWy69chiLc1qFoGEDxoKdas1JQJp58r8mbFJNcUfdj4QvJpdUTMSttb5HZUH9iGB1ZpdPyWepwuxFwqJgBs2qtVGtt2mSahvY1CDBQK5FVMhh85NBskgeNHDxyhxXPxz1JwAGtHma6KdE1pQKps9NX57tP1nqVtheDtSH7VkJJtNpQ8Dj1FCsVZtWm7nMGg7MK1YNEhgm2HBMV14qF1AujEn8onJWJGb5sakbgieghGFNwpyEwiBkoKSxHDoxvD9HiGdquT7froaYNsWQeJGUMJ21EpRnwffPsNN6S4zFPCdJ3MGvaLWdmfmmCm37nj4QaVQh4kEfZb7qN3txYCS2a3vvCyJ9fMDp5YnZidDx3NGWJcaMVzBBLNMD5cP54GhXdSJzJmxC2EKCtRqp62VFSoYW7iVnnQU7bw91JMkxvyW4inMj11GqmoZQhMTwU4eeDQeJy6CQrf1fYTjNEQBcy31mrqgRujJDp3LRbfMhEfQ12CZeAoyEFXRH58Jdg2pE6Ez5RaPkRUJHfXTjNYiio71RYEiByGNZs7tAbDApxPWJZzracqsDsaRMT5JmUCd5XUh9HyaXgwyLNZ6YsJLCQ9MKm4iwj2qAdWpAT1oGDYJpif2fUFNrTzNftcwsx5P9ZLjPc3FQdMTx2abiUSEyX2VqXGN4pfXCZedoozFKe9AdZV8bMerG6nYhzvE2s7wUYPwvRz1RjXgRi1aEcAmHRkLprWhBa9FHtcbQPWBpfxD7fEwTk6qLjGWPFCkgYPgB9pGbGRYW6zkECiPfzd8e1J9qFJm4FMtm1GFT9MmHHbRcRbodareyZc28YMx4JjaU3iLQ4ivkNPWBC2R27oU1L3uz1vheUHHFpMLGZZBJzbKvoXKxkydm6e",
    	"signer":"P-4sEEgYu1mrFyyEQotuTBTxrRnYJ61SAAn",
    	"username":"bob",
    	"password":"correct horse battery staple"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

This gives the partially signed transaction:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "tx": "1118LY8kjED4Fz83KX7r7HZ1jPPBMYBfRs7TPg32UfKktNiKw6L5m4kqDhWym4i7sNZsaEdcqdCUeYVpQYw8ydrfTKo9rJyvEHJXuiSbVJp5FGHgqfrmSB4yFkkYWLZjqFbWsj7dCxLaHUCTLYePRQqnvH6Vu8FnVk6JQzD7Sz3cNL3CPzJ2GMnzFbQtwdoQkzQL8PWXstQYmwo2VGU7LZnPaHr28zEv5q6EAo7c7dcL4aiC6pmX5XRpkvSB2mfrKSFK2XAAitmxxpCtkS5G8vLt4z36rzuq6F2T2zudL87TLPG6KqbZVPGBG4y9RkzQTMuPvaDx7BYfMXasvikWcHxQo3WCcHkeLaLCFwbwby82iBXaSHXe9Z3jvpoXjFcnmdrWMDvztp3d8c31WmEg2cudvvRqXNTaichWo7iME8KY3Kgf7xSCUWh4YjpbwdM5iYcAxPVZ2WxK53gbS6z2p9Hz1PKYoQyPjjmXQf2H65R2ZjdG4JsnXWw7a7d2WMbfKMYJQQHLzRv6hfp89LXE5368YCm2WzSJPSJxZFfVwX2rNGq34oL72Efvti6TGz6zT5aJdgqsN4QqDUBD5tNQ54B7hE3nDN66Rj1wsd3snfCBHSqpnvKLk3N9g5AZDLf6s5mRjbjrgcbvYMxiVNp4QVwW3gUTS1eEmyg9A5Ebpa4w9vKQBx8B5GuQaKmE4LV5epsM1tzh9MCJaYLUq6EGiQ9pr9Wp9VWqzULiMjuHwaQ2igyhDAcAJSgBhwN4ZtSSvvcFEnpeeSFkg7y5NxzKgsYKZG9xapKV4rkZmwccNNa7heH6yY1oFoiJn4TCtTpEmGgpt57JAQoLkQ3L2d5SahGqKsZWuwTE11EocDV2aqffptsAMTw86ax3asRXfW3BoCLaMncEKzzCeShPZJaJyKCYAF7mK5zurLz9akbNxwvML4fcNX7Bg6zqhrQoFRAiTUHzxEJ2kpubgWmzuUZHBEVpLKwZm5YY8bkVF3AXEPF4NuXdXezov2xAFcuSuy56uAMDQeEwTokYRAZzCH"
    },
    "id": 1
}
```

If the threshold of the Subnet that validates the new blockchain is 2, you have to sign with another control key.
If the threshold is 3, you have to sign 2 more times with control keys, etc.

We also have to sign the transaction with the key of the account that is paying the transaction fee.
Suppose user `bob` pays the transaction fee:

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.sign",
    "params": {
    	"tx":"1118LY8kjED4Fz83KX7r7HZ1jPPBMYBfRs7TPg32UfKktNiKw6L5m4kqDhWym4i7sNZsaEdcqdCUeYVpQYw8ydrfTKo9rJyvEHJXuiSbVJp5FGHgqfrmSB4yFkkYWLZjqFbWsj7dCxLaHUCTLYePRQqnvH6Vu8FnVk6JQzD7Sz3cNL3CPzJ2GMnzFbQtwdoQkzQL8PWXstQYmwo2VGU7LZnPaHr28zEv5q6EAo7c7dcL4aiC6pmX5XRpkvSB2mfrKSFK2XAAitmxxpCtkS5G8vLt4z36rzuq6F2T2zudL87TLPG6KqbZVPGBG4y9RkzQTMuPvaDx7BYfMXasvikWcHxQo3WCcHkeLaLCFwbwby82iBXaSHXe9Z3jvpoXjFcnmdrWMDvztp3d8c31WmEg2cudvvRqXNTaichWo7iME8KY3Kgf7xSCUWh4YjpbwdM5iYcAxPVZ2WxK53gbS6z2p9Hz1PKYoQyPjjmXQf2H65R2ZjdG4JsnXWw7a7d2WMbfKMYJQQHLzRv6hfp89LXE5368YCm2WzSJPSJxZFfVwX2rNGq34oL72Efvti6TGz6zT5aJdgqsN4QqDUBD5tNQ54B7hE3nDN66Rj1wsd3snfCBHSqpnvKLk3N9g5AZDLf6s5mRjbjrgcbvYMxiVNp4QVwW3gUTS1eEmyg9A5Ebpa4w9vKQBx8B5GuQaKmE4LV5epsM1tzh9MCJaYLUq6EGiQ9pr9Wp9VWqzULiMjuHwaQ2igyhDAcAJSgBhwN4ZtSSvvcFEnpeeSFkg7y5NxzKgsYKZG9xapKV4rkZmwccNNa7heH6yY1oFoiJn4TCtTpEmGgpt57JAQoLkQ3L2d5SahGqKsZWuwTE11EocDV2aqffptsAMTw86ax3asRXfW3BoCLaMncEKzzCeShPZJaJyKCYAF7mK5zurLz9akbNxwvML4fcNX7Bg6zqhrQoFRAiTUHzxEJ2kpubgWmzuUZHBEVpLKwZm5YY8bkVF3AXEPF4NuXdXezov2xAFcuSuy56uAMDQeEwTokYRAZzCH",
    	"signer":"P-6Y3kysjF9jnHnYkdS9yGAuoHyae2eNmeV",
    	"username":"bob",
    	"password":"correct horse battery staple"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

This gives the fully signed transaction: 

```json
{
    "jsonrpc": "2.0",
    "result": {
        "tx": "1118LY8kjED4Fz83KX7r7HZ1jPPBMYBfRs7TPg32UfKktNiKw6L5m4kqDhWym4i7sNZsaEdcqdCUeYVpQYw8ydrfTKo9rJyvEHJXuiSbVJp5FGHgqfrmSB4yFkkYWLZjqFbWsj7dCxLaHUCTLYePRQqnvH6Vu8FnVk6JQzD7Sz3cNL3CPzJ2GMnzFbQtwdoQkzQL8PWXstQYmwo2VGU7LZnPaHr28zEv5q6EAo7c7dcL4aiC6pmX5XRpkvSB2mfrKSFK2XAAitmxxpCtkS5G8vLt4z36rzuq6F2T2zudL87TLPG6KqbZVPGBG4y9RkzQTMuPvaDx7BYfMXasvikWcHxQo3WCcHkeLaLCFwbwby82iBXaSHXe9Z3jvpoXjFcnmdrWMDvztp3d8c31WmEg2cudvvRqXNTaichWo7iME8KY3Kgf7xSCUWh4YjpbwdM5iYcAxPVZ2WxK53gbS6z2p9Hz1PKYoQyPjjmXQf2H65R2ZjdG4JsnXWw7a7d2WMbfKMYJQQHLzRv6hfp89LXE5368YCm2WzSJPSJxZFfVwX2rNGq34oL72Efvti6TGz6zT5aJdgqsN4QqDUBD5tNQ54B7hE3nDN66Rj1wsd3snfCBHSqpnvKLk3N9g5AZDLf6s5mRjbjrgcbvYMxiVNp4QVwW3gUTS1eEmyg9A5Ebpa4w9vKQBx8B5GuQaKmE4LV5epsM1tzh9MCJaYLUq6EGiQ9pr9Wp9VWqzULiMjuHwaQ2igyhDAcAJSgBhwN4ZtSSvvcFEnpeeSFkg7y5NxzKgsYKZG9xapKV4rkZmwccNNa7heH6yY1oFoiJn4TCtTpEmGgpt57JAQoLkQ3L2d5Sao6GguTGEGoLFt3mKdU12akA7xWD7dE7C3dieobVYY66y9Gim5bYgK44YpHrBwcwkhxABmPGTCExZBqd8E6wdzQCnJ4LbGZnrwV9KExThGLxexyTN24RbRn9EHUNF1gmWknRNZ1rnWB9fBPA8X8kwkKF8YZGoYS7MhzAyhCKjLui9mAcs17VzWpAHAEJ8V"
    },
    "id": 1
}
```

## Issue the Transaction

To issue the transaction to the network we call `platforn.issueTx`:

```json
{
    "jsonrpc": "2.0",
    "method": "platform.issueTx",
    "params": {
    	"tx":"1118LY8kjED4Fz83KX7r7HZ1jPPBMYBfRs7TPg32UfKktNiKw6L5m4kqDhWym4i7sNZsaEdcqdCUeYVpQYw8ydrfTKo9rJyvEHJXuiSbVJp5FGHgqfrmSB4yFkkYWLZjqFbWsj7dCxLaHUCTLYePRQqnvH6Vu8FnVk6JQzD7Sz3cNL3CPzJ2GMnzFbQtwdoQkzQL8PWXstQYmwo2VGU7LZnPaHr28zEv5q6EAo7c7dcL4aiC6pmX5XRpkvSB2mfrKSFK2XAAitmxxpCtkS5G8vLt4z36rzuq6F2T2zudL87TLPG6KqbZVPGBG4y9RkzQTMuPvaDx7BYfMXasvikWcHxQo3WCcHkeLaLCFwbwby82iBXaSHXe9Z3jvpoXjFcnmdrWMDvztp3d8c31WmEg2cudvvRqXNTaichWo7iME8KY3Kgf7xSCUWh4YjpbwdM5iYcAxPVZ2WxK53gbS6z2p9Hz1PKYoQyPjjmXQf2H65R2ZjdG4JsnXWw7a7d2WMbfKMYJQQHLzRv6hfp89LXE5368YCm2WzSJPSJxZFfVwX2rNGq34oL72Efvti6TGz6zT5aJdgqsN4QqDUBD5tNQ54B7hE3nDN66Rj1wsd3snfCBHSqpnvKLk3N9g5AZDLf6s5mRjbjrgcbvYMxiVNp4QVwW3gUTS1eEmyg9A5Ebpa4w9vKQBx8B5GuQaKmE4LV5epsM1tzh9MCJaYLUq6EGiQ9pr9Wp9VWqzULiMjuHwaQ2igyhDAcAJSgBhwN4ZtSSvvcFEnpeeSFkg7y5NxzKgsYKZG9xapKV4rkZmwccNNa7heH6yY1oFoiJn4TCtTpEmGgpt57JAQoLkQ3L2d5Sao6GguTGEGoLFt3mKdU12akA7xWD7dE7C3dieobVYY66y9Gim5bYgK44YpHrBwcwkhxABmPGTCExZBqd8E6wdzQCnJ4LbGZnrwV9KExThGLxexyTN24RbRn9EHUNF1gmWknRNZ1rnWB9fBPA8X8kwkKF8YZGoYS7MhzAyhCKjLui9mAcs17VzWpAHAEJ8V"
    },
    "id": 1
}
```

The response contains the transaction's ID, which is also the ID of the new blockchain:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "txID": "zpFTwJwzPh3b9N6Ahccy4fXdJFHJJdhGah5z731J6ZspcYKpK"
    },
    "id": 1
}
```


## Verify Success

After a few seconds, the transaction to create our blockchain should have been accepted
and the blockchain should exist (assuming the request was well-formed, etc.)

To check, call `platform.getBlockchains`.
This returns a list of all blockchains that exist.

```json
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"platform.getBlockchains",
    "params" :{}
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The response confirms that the blockchain was created:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "blockchains": [
            {
                "id": "AXerNQX7voY2AABaRdTAyXcawBURBR6thPRJp43LtPpZZi6Qz",
                "name": "X-Chain",
                "subnetID": "11111111111111111111111111111111LpoYY",
                "vmID": "jvYyfQTxGMJLuGWa55kdP2p2zSUYsQ5Raupu4TW34ZAUBAbtq"
            },
            {
                "id": "tZGm6RCkeGpVETUTp11DW3UYFZmm69zfqxchpHrSF7wgy8rmw",
                "name": "C-Chain",
                "subnetID": "11111111111111111111111111111111LpoYY",
                "vmID": "mgj786NP7uDwBCcq6YwThhaN8FLyybkCa4zBWTQbNgmK6k9A6"
            },
            {
                "id": "CqhF97NNugqYLiGaQJ2xckfmkEr8uNeGG5TQbyGcgnZ5ahQwa",
                "name": "Simple DAG Payments",
                "subnetID": "11111111111111111111111111111111LpoYY",
                "vmID": "sqjdyTKUSrQs1YmKDTUbdUhdstSdtRTGRbUn8sqK8B6pkZkz1"
            },
            {
                "id": "VcqKNBJsYanhVFxGyQE5CyNVYxL3ZFD7cnKptKWeVikJKQkjv",
                "name": "Simple Chain Payments",
                "subnetID": "11111111111111111111111111111111LpoYY",
                "vmID": "sqjchUjzDqDfBPGjfQq2tXW1UCwZTyvzAWHsNzF2cb1eVHt6w"
            },
            {
                "id": "2SMYrx4Dj6QqCEA3WjnUTYEFSnpqVTwyV3GPNgQqQZbBbFgoJX",
                "name": "Simple Timestamp Server",
                "subnetID": "11111111111111111111111111111111LpoYY",
                "vmID": "tGas3T58KzdjLHhBDMnH2TvrddhqTji5iZAMZ3RXs2NLpSnhH"
            },
            {
                "id": "zpFTwJwzPh3b9N6Ahccy4fXdJFHJJdhGah5z731J6ZspcYKpK",
                "name": "My new AVM",
                "subnetID": "KL1e8io1Zi2kr8cTXxvi321pAzfQuUa8tmBfadqpf9K2dc2TT",
                "vmID": "jvYyfQTxGMJLuGWa55kdP2p2zSUYsQ5Raupu4TW34ZAUBAbtq"
            }
        ]
    },
    "id": 1
}
```

## Interact With the New Blockchain

You can interact with this new instance of the AVM almost the same way you'd interact with the X-Chain.
There are two small differences:

* The API endpoint of your blockchain is `127.0.0.1:9650/ext/bc/zpFTwJwzPh3b9N6Ahccy4fXdJFHJJdhGah5z731J6ZspcYKpK`.
* Addresses are prepended with `zpFTwJwzPh3b9N6Ahccy4fXdJFHJJdhGah5z731J6ZspcYKpK-` rather than `X-`. 

In the genesis data we specified that address `8UeduLccQuSmYiY3fGQEyotM9uXxoHoQQ` has 100,000 units of the asset with alias `asset1`.
Let's verify that:

```json
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :2,
    "method" :"avm.getBalance",
    "params" :{
        "address":"zpFTwJwzPh3b9N6Ahccy4fXdJFHJJdhGah5z731J6ZspcYKpK-8UeduLccQuSmYiY3fGQEyotM9uXxoHoQQ",
        "assetID":"asset1"
    }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/bc/zpFTwJwzPh3b9N6Ahccy4fXdJFHJJdhGah5z731J6ZspcYKpK

```json
{
    "jsonrpc": "2.0",
    "result": {
        "balance": "100000"
    },
    "id": 1
}
```