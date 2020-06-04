# Create a Subnet

## Introduction

A **Subnet** is a set of validators. 
A Subnet validates a set of blockchains.
Each blockchain is validated by exactly one Subnet, which is specified on blockchain creation.
Subnets are a powerful primitive that allow blockchains to have custom validator sets, which means one can create permissioned blockchains.

When a Subnet is created, a threshold and a set of keys are specified.
(Actually the addresses of the keys, not the keys themselves, are specified.)
In order to add a validator to that subnet, *threshold* signatures from those keys are needed.
We call these the Subnet's **control keys** and we call a control key's signature on a transaction that adds a validator to a Subnet a **control signature.** 
The upshot is that a Subnet has control over its membership.

In this tutorial we'll create a new Subnet with 2 control keys and a threshold of 2.

## Create Users

It would somewhat defeat the purpose of having 2 control keys if one user held both, so we'll have 2 users, each of which holds one key.
Recall that a user is an identity that one can use when interacting with blockchains.
To create the users, we'll call [`keystore.createUser`](../api/keystore.md#keystorecreateuser).
Of course, if you already have 2 users on this node you can skip this step.

Create the first user:

```json
curl -X POST --data '{
     "jsonrpc": "2.0",
     "id": 1,
     "method": "keystore.createUser",
     "params": {
         "username": "USERNAME 1 GOES HERE",
         "password": "PASSWORD 1 GOES HERE"
     }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/keystore
```

The response should indicate success.

Now create another user:

```json
curl -X POST --data '{
     "jsonrpc": "2.0",
     "id": 1,
     "method": "keystore.createUser",
     "params": {
         "username": "USERNAME 2 GOES HERE",
         "password": "PASSWORD 2 GOES HERE"
     }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/keystore
```

## Generate the Control Keys

First, let's generate the 2 control keys.
To do so we call [`platform.createAccount.`](../api/platform.md#platformcreateaccount)
This generates a new private key and stores it for a user.

To generate the first key:

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.createAccount",
    "params": {
    	"username":"USERNAME 1 GOES HERE",
    	"password":"PASSWORD 1 GOES HERE"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

This gives the first control key (again, it actually gives the *address* of the first control key).
The key is held by the user we just specified.

```json
{
    "jsonrpc": "2.0",
    "result": {
        "address": "P-KNjXsaA1sZsaKCD1cd85YXauDuxshTes2"
    },
    "id": 1
}
```

To generate the second key:

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.createAccount",
    "params": {
    	"username":"USERNAME 2 GOES HERE",
    	"password":"PASSWORD 2 GOES HERE"
    },
    "id": 2
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The response contains the second control key, which is held by the user we just specified:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "address": "P-Aiz4eEt5xv9t4NCnAWaQJFNz5ABqLtJkR"
    },
    "id": 2
}
```

## Create the Unsigned Transaction

To create a Subnet we call [`platform.createSubnet`.](../api/platform.md#platformcreatesubnet)
`payerNonce` is the next unused nonce of the account that pays the transaction fee.

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.createSubnet",
    "params": {
    	"controlKeys":[
    		"P-KNjXsaA1sZsaKCD1cd85YXauDuxshTes2",
    		"P-Aiz4eEt5xv9t4NCnAWaQJFNz5ABqLtJkR"
    	],
    	"threshold":2,
    	"payerNonce":1
    },
    "id": 3
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The response gives us the unsigned transaction:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "unsignedTx": "1112LA7e8GvkGHDkxZa9Q7kszr1HN6ppLb9KCUv41BzceP3uyzYFQN1jUgZdAjchGkGPXYJn9P1N36qdxpSaGLotkCZv4UXUToksTiGHBK3nGB1J2sxfgsqjfH1TesstRqqX2QDF5k6DP2E6JWni89fZvZGZPdPWoTp5de3iePnTPNMgSn"
    },
    "id": 3
}
```

## Sign the Transaction

Now we sign the unsigned transaction with the key of the account that pays the transaction fee:

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.sign",
    "params": {
    	"tx":"1112LA7e8GvkGHDkxZa9Q7kszr1HN6ppLb9KCUv41BzceP3uyzYFQN1jUgZdAjchGkGPXYJn9P1N36qdxpSaGLotkCZv4UXUToksTiGHBK3nGB1J2sxfgsqjfH1TesstRqqX2QDF5k6DP2E6JWni89fZvZGZPdPWoTp5de3iePnTPNMgSn",
    	"signer":"P-6Y3kysjF9jnHnYkdS9yGAuoHyae2eNmeV",
    	"username":"USER THAT CONTROLS ACCOUNT PAYING FEE",
    	"password":"THEIR PASSWORD"
    },
    "id": 4
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

And we get the signed transaction:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "Tx": "1112LA7e8GvkGHDkxZa9Q7kszr1HN6ppLb9KCUv41BzceP3uyzYFQN1jUgZdAjchGkGPXYJn9P1N36qdxpSaYA31WmW6WQAa9KP5WQaToxwZoYazubE5jEVj3K2ZsrqZmEFVLRhUds4LEyngRpXGnf3x91Sf7ZKxp32yyoBRueWbRrc6Yr"
    },
    "id": 4
}
```

## Issue the Transaction

To issue the transaction we call [`platform.issueTx`](../api/platform.md#platformissuetx):

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.issueTx",
    "params": {
    	"tx":"1112LA7e8GvkGHDkxZa9Q7kszr1HN6ppLb9KCUv41BzceP3uyzYFQN1jUgZdAjchGkGPXYJn9P1N36qdxpSaYA31WmW6WQAa9KP5WQaToxwZoYazubE5jEVj3K2ZsrqZmEFVLRhUds4LEyngRpXGnf3x91Sf7ZKxp32yyoBRueWbRrc6Yr"
    },
    "id": 5
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The transaction ID is the ID of the new Subnet:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "txID": "hW8Ma7dLMA7o4xmJf3AXBbo17bXzE7xnThUd3ypM4VAWo1sNJ"
    },
    "id": 5
}
```

## Verifying Success

We can call [`platform.getSubnets`](../api/platform.md#platformgetsubnets) to get all Subnets that exist:

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.getSubnets",
    "params": {},
    "id": 6
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The response confirms that our Subnet was created:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "subnets": [
            {
                "id": "hW8Ma7dLMA7o4xmJf3AXBbo17bXzE7xnThUd3ypM4VAWo1sNJ",
                "controlKeys": [
                    "P-KNjXsaA1sZsaKCD1cd85YXauDuxshTes2",
                    "P-Aiz4eEt5xv9t4NCnAWaQJFNz5ABqLtJkR"
                ],
                "threshold": "2"
            }
        ]
    },
    "id": 6
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

## Export a User

The 2 control keys that are needed to add validators to this Subnet are held by users on this node.
Since users are local to a node, that means validators can only be added to the new Subnet via transactions made by this node.
What if we want to use a different node to issue a transaction to add a validator?

We can export a user and all of their data (including their keys) by calling [`keystore.exportUser`](../api/keystore.md#keystoreexportuser)

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "keystore.exportUser",
    "params":{
    	"username":"USERNAME 1 GOES HERE",
    	"password":"PASSWORD 1 GOES HERE"
    },
    "id": 7
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/keystore
```

This gives the user's (encrypted) data:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "user": "3xnxQ4r7a49tcALqnod6L9jLjQ5KEiZucQfAYZGmfQ67b62TXYsx7iKeBh8YSNu4mdkL1tDKtnSCHGQaZtJdirbyfV991yLdeyioKVcK76VzDdTcha63rQ5qXJtKGktjNL7fMdSxkYzwKhgEz2wuQiWcnoLUrDRnrj71a8VJ6m36eoZ4fspGeyncktYuRoJp8ppVzUKiFf5T4BWpwet4Y6gbkc44v9rYo4Qo7XcDuPHMJ7ZWKX7ZTHtAvEhkpUP1jVvKCxhrFqF4ftomrhKmBQdxyRKevXA6ToFV77xicWuNkPbQ2hsGJpuN8xeWsKp514a5QDXPbuBVNuBJgHmsTSuHSsAwatjDPL2QDpvB1TZnU81XuXRs68Zcs6fFjwCGumbT29jcSBZsoSkL9uC1sK9krDLWKJGhSNWvFTYTRkJb8iw9J48xX7uf5GKT1CNKf1uLTPkFB8guBhb1obJF17TVNKyxix8LbvK9kLq7yr1zF8LzEgDoCF5Yw3yYvYJc7MFiTA3TiZcm4LKjDGhmR2zmH19fQ1SyEacSRtMkZomLrnrSQpNomrebQsnGm2Mv9es88SahdjDFHADqcxqsd5aigydCQCm2mj8beXQNqK5jRL6oYyd8rxD8WdbbkVRed5X2aMXqkWFXc97sa5ToyqyBpWXrfDg3yPjYyo29ywCd2Untt7M2TpYyUGwGZo9tYzEiS5ozGgWFN3HkZurBENjScdJgGJG3S1iwXheYgMjqhYyRTh3FgM7GU5XM56aDKWbX6HH8Xzf71mUWEH1PFvPsTvMgrYzKK9oeHbAJeh87WrrNFFJp5wmYCgyxxMtyFxHw7W3b7EoTKYw9GoaCvHRSqM1UBchy94uWKp812d7iB5e734NDzad9mikwtPwJasP8Y5i89HpPJG8NQW2Sbq2HbBvtVe4yj3FrATLNqRvGZmDjQtBFjkss2KVaRYQH3pCd4mWD7gTnDhJmpzk3j1GQXUZ7xUBgsV8QNymx5CtsVx1JRT63FuKnafr8FL3GncZLXsmQvxYaDoo9ufXXEbEuSyVDGyHGjosrabGAGuQJ7CVWQPaKbSjdSfiuXhjECjd8a688aMJZtLjh4nMorsuTW4squwo3rVSUvLqGW5eUb4gBvBhoK9RTL2F35QYAVSwEbWhut8uQ2QKLSLz8gRT2eBT3Grz8FWXpM3hKRk7xcw2ZuXLiPfsRftVsoyneAqsEQnqvN61B7wzChSzvYETjZPgEN6AqFqd1xoCXQiD7XvBnvjvVFiWt3TL6szX9Hyo3qVQH41qx82fQbxq3jjwCbrAowhd1aF1txCnxcXr5s8MNFefG2XhdPSqckTGTCRUUVAJ7m4p"
    },
    "id": 7
}
```

We can import the user's data to another node by calling [`keystore.importUser`.](../api/keystore.md#keystoreimportuser) 
Note that we're sending this API call to a different node (one listening on port `9652`.)

```json
curl -X POST --data '
{
    "jsonrpc": "2.0",
    "method": "keystore.importUser",
    "params":{
    	"username":"USERNAME 1 GOES HERE",
    	"password":"PASSWORD 1 GOES HERE",
    	"user":"3xnxQ4r7a49tcALqnod6L9jLjQ5KEiZucQfAYZGmfQ67b62TXYsx7iKeBh8YSNu4mdkL1tDKtnSCHGQaZtJdirbyfV991yLdeyioKVcK76VzDdTcha63rQ5qXJtKGktjNL7fMdSxkYzwKhgEz2wuQiWcnoLUrDRnrj71a8VJ6m36eoZ4fspGeyncktYuRoJp8ppVzUKiFf5T4BWpwet4Y6gbkc44v9rYo4Qo7XcDuPHMJ7ZWKX7ZTHtAvEhkpUP1jVvKCxhrFqF4ftomrhKmBQdxyRKevXA6ToFV77xicWuNkPbQ2hsGJpuN8xeWsKp514a5QDXPbuBVNuBJgHmsTSuHSsAwatjDPL2QDpvB1TZnU81XuXRs68Zcs6fFjwCGumbT29jcSBZsoSkL9uC1sK9krDLWKJGhSNWvFTYTRkJb8iw9J48xX7uf5GKT1CNKf1uLTPkFB8guBhb1obJF17TVNKyxix8LbvK9kLq7yr1zF8LzEgDoCF5Yw3yYvYJc7MFiTA3TiZcm4LKjDGhmR2zmH19fQ1SyEacSRtMkZomLrnrSQpNomrebQsnGm2Mv9es88SahdjDFHADqcxqsd5aigydCQCm2mj8beXQNqK5jRL6oYyd8rxD8WdbbkVRed5X2aMXqkWFXc97sa5ToyqyBpWXrfDg3yPjYyo29ywCd2Untt7M2TpYyUGwGZo9tYzEiS5ozGgWFN3HkZurBENjScdJgGJG3S1iwXheYgMjqhYyRTh3FgM7GU5XM56aDKWbX6HH8Xzf71mUWEH1PFvPsTvMgrYzKK9oeHbAJeh87WrrNFFJp5wmYCgyxxMtyFxHw7W3b7EoTKYw9GoaCvHRSqM1UBchy94uWKp812d7iB5e734NDzad9mikwtPwJasP8Y5i89HpPJG8NQW2Sbq2HbBvtVe4yj3FrATLNqRvGZmDjQtBFjkss2KVaRYQH3pCd4mWD7gTnDhJmpzk3j1GQXUZ7xUBgsV8QNymx5CtsVx1JRT63FuKnafr8FL3GncZLXsmQvxYaDoo9ufXXEbEuSyVDGyHGjosrabGAGuQJ7CVWQPaKbSjdSfiuXhjECjd8a688aMJZtLjh4nMorsuTW4squwo3rVSUvLqGW5eUb4gBvBhoK9RTL2F35QYAVSwEbWhut8uQ2QKLSLz8gRT2eBT3Grz8FWXpM3hKRk7xcw2ZuXLiPfsRftVsoyneAqsEQnqvN61B7wzChSzvYETjZPgEN6AqFqd1xoCXQiD7XvBnvjvVFiWt3TL6szX9Hyo3qVQH41qx82fQbxq3jjwCbrAowhd1aF1txCnxcXr5s8MNFefG2XhdPSqckTGTCRUUVAJ7m4p"
    },
    "id": 8
}' -H 'content-type:application/json;' 127.0.0.1:9652/ext/keystore
```

The response should indicate success.
Now we can call [`platform.listAccounts`](../api/platform.md#platformlistaccounts) and see that the user was successfully imported:

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "result": {
        "accounts": [
            {
                "address": "P-KNjXsaA1sZsaKCD1cd85YXauDuxshTes2",
                "nonce": "0",
                "balance": "0"
            }
        ]
    },
    "id": 8
}
}' -H 'content-type:application/json;' 127.0.0.1:9652/ext/P
```

Now we can use this node to provide `KNjXsaA1sZsaKCD1cd85YXauDuxshTes2`'s signature when add validators to the new Subnet.

## Add Validators to the Subnet

[This tutorial](../tutorials/adding-validators.md) will show you how to add validators to a Subnet.

