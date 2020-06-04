# Adding a Validator to a Subnet

## Introduction

In this tutorial we'll add a node to the Default Subnet and to a non-default Subnet on the AVA Public Testnet.

The [Platform Chain (P-Chain)](../core-concepts/overview.md#the-p-chain) manages metadata about the AVA network.
This includes tracking which nodes are in which Subnets, which blockchains exist and which Subnets are validating which blockchains.
To add validators to Subnets we'll issue transactions to the P-Chain.

The P-Chain uses an account model.
Each account has an address that uniquely identifies it, a balance of AVA tokens, and a nonce.
The nonce is incremented each time the account sends a transaction.

## Requirements

We assume that you've already done the [quickstart guide](../quickstart/ava-getting-started.md) and [subnet creation tutorial](../tutorials/create-a-subnet.md), and are familiar with the [AVA Network's architecture.](../core-concepts/overview.md)

## Connect to the Network

First, the node you're adding will need to be connected to the AVA Public Testnet.

To start your node and connect to the AVA Public Testnet, follow the same instructions as in the quickstart guide.

## Add a Validator to the Default Subnet

The [Default Subnet](../core-concepts/overview.md#what-are-subnets) is inherent to the AVA network and validates AVA's [built-in blockchains.](../core-concepts/overview.md#built-in-blockchains)

Let's get the values needed to make the transaction that adds a node to the Default Subnet.

### Node ID

Validators identify one another by their node IDs.

To get the ID of your node, call [`admin.getNodeID`:](../api/admin.md#admingetnodeid)

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "admin.getNodeID",
    "params":{},
    "id": 84
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/admin
```

The response has your node's ID:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "nodeID": "ARCLrphAHZ28xZEBfUL7SVAmzkTZNe1LK"
    },
    "id": 84
}
```

### Nonce

In order to add a validator to the Default Subnet, you'll need an account with a sufficient balance of AVA tokens to provide the stake and to pay the transaction fee.

Follow [this tutorial](./atomic-swap.md) to fund a P-Chain account.

To get an account's nonce, call [`platform.getAccount.`](../api/platform.md#platformgetaccount)

```json
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"platform.getAccount",
    "params" :{
        "address": "P-6Y3kysjF9jnHnYkdS9yGAuoHyae2eNmeV"
    }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

Since we haven't done anything with this account its nonce is 0:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "address": "P-6Y3kysjF9jnHnYkdS9yGAuoHyae2eNmeV",
        "nonce": "0",
        "balance": "20000000000000"
    },
    "id": 1
}
```

### Stake Amount

In order to validate the Default Subnet one must stake AVA tokens.
The minimum amount that one can stake is 10 $\mu$AVA.

### Start and End Time

When one issues a transaction to join the Default Subnet they specify the time they will enter (start validating) and leave (stop validating.)
The minimum duration that one can validate the Default Subnet is 24 hours, and the maximum duration is one year.
One can re-enter the Default Subnet after leaving, it's just that the maximum *continuous* duration is one year.

The start time must be in the future relative to the time the transaction is issued.

### Destination

When a validator leaves the Default Subnet, the staked AVA tokens are sent to an account that is specified in the transaction that added the validator to the Default Subnet.
We call this the *destination* account.

A validator earns a *validation reward* if they are sufficiently responsive and correct while they validate the Default Subnet.
The validation reward is also sent to the same account as the staked AVA.

A validator's stake is never slashed, regardless of their behavior; they will always receive their stake back when their validation period is over.

### Delegation Fee

The final parameter, `delegationFeeRate`, is the percent fee this validator charges when others delegate stake to them, multiplied by 10,000.

For example, suppose a validator has `delegationFeeRate` 300,000 and someone delegates to that validator.
When the delegation period is over, if the delegator is entitled to a reward, 30% of the reward (300,000 / 10,000) goes to the validator and 70% goes to the delegator.

### Create the Unsigned Transaction

To add a node the Default Subnet, call [`platform.addDefaultSubnetValidator`](../api/platform.md#platformadddefaultsubnetvalidator).

The API call should look like the one below. 

`startTime` **must be in the future relative to the current Unix time** (which you can lookup [here](https://www.unixtimestamp.com/)) and `endTime` must be
between one day and one year ahead of `startTime.`
Below, we use the shell command `date` to compute the Unix time 10 minutes and 2 days in the future to use as the values of `startTime` and `endTime`, respectively.

(Note: If you're on a Mac, replace  `$(date` with `$(gdate`. If you don't have `gdate` installed, do `brew install coreutils`.)

```sh
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.addDefaultSubnetValidator",
    "params": {
    	"id":"ARCLrphAHZ28xZEBfUL7SVAmzkTZNe1LK",
    	"payerNonce":1,
    	"destination":"P-Q4MzFZZDPHRPAHFeDs3NiyyaZDvxHKivf",
    	"startTime":'$(date --date="10 minutes" +%s)',
    	"endTime":'$(date --date="2 days" +%s)',
    	"stakeAmount":1000000,
    	"delegationFeeRate":100000
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The response has the unsigned transaction:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "unsignedTx": "111fRKBNoBhBfeGvBzvz6r9dZUKbEnUypM6tjiSyYrWM4ojSTuL2Syxv8cFLphtYaxdM1EA3Aj4yej8ABSfmjb9NMrtxQac9cnWwCER7GHSzFULB25hAtzGtJ8XhsrKcvtpAM8FwjRzg3Bg1q6V8GTKGMC219bYMETS48GMFGh4nts1Jsf246rjZ26r1Vyok8MdnoaxjQWR6cKq"
    },
    "id": 1
}
```

### Sign the Transaction

We need to sign this transaction with the key of the account that is paying the transaction fee and providing the staked tokens.

To do so, call `platform.Sign`.
Replace `signer` with the address of the account providing the staked AVA and the fee.
The provided user must control that account; provide your username and password to authenticate.

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.sign",
    "params": {
    	"tx":"111fRKBNoBhBfeGvBzvz6r9dZUKbEnUypM6tjiSyYrWM4ojSTuL2Syxv8cFLphtYaxdM1EA3Aj4yej8ABSfmjb9NMrtxQac9cnWwCER7GHSzFULB25hAtzGtJ8XhsrKcvtpAM8FwjRzg3Bg1q6V8GTKGMC219bYMETS48GMFGh4nts1Jsf246rjZ26r1Vyok8MdnoaxjQWR6cKq",
    	"signer":"P-6Y3kysjF9jnHnYkdS9yGAuoHyae2eNmeV",
    	"username":"YOUR USERNAME",
    	"password":"YOUR PASSWORD"
    },
    "id": 2
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

This returns the signed transaction:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "Tx": "111fRKBNoBhBfeGvBzvz6r9dZUKbEnUypM6tjiSyYrWM4ojSTuL2Syxv8cFLphtYaxdM1EA3Aj4yej8ABSfmjb9NMrtxQac9cnWwCER7GHSzFULB4RoAjStfe26qQxhS91KvCCX3WBLmpyvNXHzgWk3uJP45cPv15RHGymFboPUcxNTwGij1NgQpKPcL4YxcDnKvNjrcQzKiXAz"
    },
    "id": 2
}
```

### Issue the Transaction

Finally, we issue the transaction to the network by calling [`platform.issueTx`](../api/platform.md#platformissuetx)

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.issueTx",
    "params": {
    	"tx":"111fRKBNoBhBfeGvBzvz6r9dZUKbEnUypM6tjiSyYrWM4ojSTuL2Syxv8cFLphtYaxdM1EA3Aj4yej8ABSfmjb9NMrtxQac9cnWwCER7GHSzFULB4RoAjStfe26qQxhS91KvCCX3WBLmpyvNXHzgWk3uJP45cPv15RHGymFboPUcxNTwGij1NgQpKPcL4YxcDnKvNjrcQzKiXAz"
    },
    "id": 3
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

### Verify Success

Now we can call [`platform.getPendingValidators`](../api/platform.md#platformgetpendingvalidators) to verify that the node has been added to the pending validator set of the Default Subnet.
A Subnet's pending validator set contains the nodes that are slated to start validating in the future.

If we don't provide any arguments, this method returns the pending validator set of the Default Subnet:

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.getPendingValidators",
    "params": {},
    "id": 4
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The response should confirm that the node with ID `ARCLrphAHZ28xZEBfUL7SVAmzkTZNe1LK` will start validating the Default Subnet in the future.

```json
{
    "jsonrpc": "2.0",
    "result": {
        "validators": [
            {
                "id": "ARCLrphAHZ28xZEBfUL7SVAmzkTZNe1LK",
                "startTime": "1584021450",
                "endtime": "1584121156",
                "stakeAmount": "1000000",
            }
        ]
    },
    "id": 4
}
```

When the time reaches `1584021450`, this node will start validating the Default Subnet.

## Add a Validator to a Non-default Subnet

Now let's add the same node to a non-default Subnet (that is, any Subnet other than the Default Subnet.)
The following will make more sense if you've already done this tutorial on creating a Subnet.

Suppose that the Subnet has ID `nTd2Q2nTLp8M9qv2VKHMdvYhtNWX7aTPa4SMEK7x7yJHbcWvr`, threshold 2, and its control keys belong to addresses `98vMGrh2nWNr8oDNKVK9jdxN1bwkeg4Jd` and `6UGRmWANxejv1uM5T8BiRR2VPFSk1aFWA`.

### Create the Unsigned Transaction

The period that a node validates a non-default Subnet must be a subset of the time they validate the Default Subnet.
That is, they can only be a member of a non-default Subnet if they are a member of the Default Subnet.
The same minimum and maximum staking periods (one day and one year, respectively) that apply to the Default Subnet apply to all subnbets. 

Note that we increment `payerNonce` since we're going to use the same account as before to pay the transaction fee.

The API call to create the unsigned transaction looks very similar to the one we used to add a node to the Default Subnet:

(Note: If you're on a Mac, replace  `$(date` with `$(gdate`. If you don't have `gdate` installed, do `brew install coreutils`.)

```sh
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.addNonDefaultSubnetValidator",
    "params": {
    	"id":"ARCLrphAHZ28xZEBfUL7SVAmzkTZNe1LK",
		"subnetID":"nTd2Q2nTLp8M9qv2VKHMdvYhtNWX7aTPa4SMEK7x7yJHbcWvr",
    	"startTime":'$(date --date="10 minutes" +%s)',
    	"endTime":'$(date --date="2 days" +%s)',
    	"weight":1,
    	"payerNonce":3
    },
    "id": 5
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The response has the unsigned transaction:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "unsignedTx": "1115PJEszaVtWiXtNfyeSvUX385ih3x7V1AyiWFqYKUHp5orbz1oJ9CizuU2dK8pKJpoFtFD1MQMmvi6p4uC1GkoiGg54ujbVDnMwidhkinAhbeVPKsX6Ekdg3hLk5SWHkjDGnaNSavu4K86Z4fu1tB1xgfk9vyACzDjCU4w5rksPfqEZA48zskrh6cuK1QW1ruXPje8iWaNfzjr7cYdFa6EaPC4BcTS"
    },
    "id": 5
}
```

### Sign the Transaction

We need to sign this transaction three times.
It needs a signature from both of the Subnet's control keys and from the key of the account paying the transaction fee.

You might be wondering, "how does `sign` know whether a key should sign as a control signature or as the transaction fee payer?"
If `signer` is a control key and that has not yet provided a control signature, `signer`'s key provides a control signature.
If `signer` is a control key that has already provided a control signature, `signer`'s key provides the payer signature.
If `signer` is not a control key, `signer` provides the payer signature.  

Let's sign with the control keys first:

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.sign",
    "params": {
    	"tx":"1115PJEszaVtWiXtNfyeSvUX385ih3x7V1AyiWFqYKUHp5orbz1oJ9CizuU2dK8pKJpoFtFD1MQMmvi6p4uC1GkoiGg54ujbVDnMwidhkinAhbeVPKsX6Ekdg3hLk5SWHkjDGnaNSavu4K86Z4fu1tB1xgfk9vyACzDjCU4w5rksPfqEZA48zskrh6cuK1QW1ruXPje8iWaNfzjr7cYdFa6EaPC4BcTS",
    	"signer":"P-98vMGrh2nWNr8oDNKVK9jdxN1bwkeg4Jd",
    	"username":"USER THAT CONTROL THE SIGNER'S KEY",
    	"password":"THAT USER'S PASSWORD"
    },
    "id": 6
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The partially signed transaction:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "Tx": "1112i3pBsJBwde3PTEVXNpSpd6HKymaFvF9ejGjbuQR3omM4oa6CR9d51maB57ri6PxphfHYANkTQGpotbdRr6c2AzpVotzBfHSfx5Zdt29xcYZQFvAmY3KT9yXA1JEJguX9D6BHvCMFW9n9uymMP2Y7iPwAmSQLJLe53c8LFFzor3bsARJjA16EPnX5gwsXoG1ePJQSSFb6oNkxtSi2dHwNK1zdRvr5V8iUTTbvvzvaFFHxafNAhbzwr9eBKT9iAis2gGC6U1ZokWqsevRSwruyZGYWoLx7VEfCECtKSfU5VAzdw7H1DcXPj"
    },
    "id": 6
}
```

Sign with the other control key:

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.sign",
    "params": {
    	"tx":"1112i3pBsJBwde3PTEVXNpSpd6HKymaFvF9ejGjbuQR3omM4oa6CR9d51maB57ri6PxphfHYANkTQGpotbdRr6c2AzpVotzBfHSfx5Zdt29xcYZQFvAmY3KT9yXA1JEJguX9D6BHvCMFW9n9uymMP2Y7iPwAmSQLJLe53c8LFFzor3bsARJjA16EPnX5gwsXoG1ePJQSSFb6oNkxtSi2dHwNK1zdRvr5V8iUTTbvvzvaFFHxafNAhbzwr9eBKT9iAis2gGC6U1ZokWqsevRSwruyZGYWoLx7VEfCECtKSfU5VAzdw7H1DcXPj",
    	"signer":"P-6UGRmWANxejv1uM5T8BiRR2VPFSk1aFWA",
    	"username":"USER THAT CONTROL THE SIGNER'S KEY",
    	"password":"THAT USER'S PASSWORD"
    },
    "id": 7
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The partially signed transaction is now:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "Tx": "1112i3pBsJBwde3PTEVXNpSpd6HKymaFvF9ejGjbuQR3omM4oa6CR9d51maB57ri6PxphfHYANkTQGpotbdRr6c2AzpVotzBfHSfx5Zdt29xcYZQFvAmY3KT9yXA1JEJgtvLy65ptUR1iUPFWA8mu2AhrxSQHsNhcz4tXedaBHnjSYhHhWhJq5brsyR2M7fBAPWxNtNSZVdzG129sJYG5BWmqucNxoCWqSuarmACupejamG8HPgmyr9jZzkUnnzmBfYAjJ3DSRdFoYK7AhEP8mLzBATzxzdfi3BHwgmApRPLT1neAxLm2HRu3"
    },
    "id": 7
}
```

Finally, sign with the key that pays the transaction fee:

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.sign",
    "params": {
    	"tx":"111fRKBNoBhBfeGvBzvz6r9dZUKbEnUypM6tjiSyYrWM4ojSTuL2Syxv8cFLsFAf7GaCbLiBWEfaDfEaQ7L1qMgeimPbewtBPphXSnHy6mx86YZvBKFTyE659CYAb6kKcj3L8osr4Kf8Qd3zJCAFPbSy8sxowA27p1SJuueVw5kHdWannWzfFmPNN7DBV8wpymmktGT3gbgq7ZV",
    	"signer":"P-6Y3kysjF9jnHnYkdS9yGAuoHyae2eNmeV",
    	"username":"USER THAT CONTROLS THE SIGNER'S KEY",
    	"password":"THAT USER'S PASSWORD"
    },
    "id": 8
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

This gives the signed transaction:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "Tx": "1112i3pBsJBwde3PTEVXNpSpd6HKymaFvF9ejGjbuQR3omM4oa6CR9d51maB57ri6PxphfHYANkTQGpotbdRr6c2AzpVotzBfHSfx5Zdt29xcYZQFvAmY3KT9yXA1JEJgtvLy65ptUR1iUPFWA8mu2AhrxSQHsNhcz4tXedaBHnjSYhHhWhJq5brsyR2M7fBAPWxNtNSZVdzG129sJYG5BWmqucqH1UdeYfaUKVzEXxt53ta6SGe5RTKrAYjr3x8NbfDB3ZXwU4udttVZrVDCNMWo4HJMJH9E5rdFeuovaD3L8nrQR4PCBpHm"
    },
    "id": 8
}
```

### Issue the Transaction

Now that we've provided the necessary control signature and payer signature we can issue the transaction:

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.issueTx",
    "params": {
    	"tx":"1112i3pBsJBwde3PTEVXNpSpd6HKymaFvF9ejGjbuQR3omM4oa6CR9d51maB57ri6PxphfHYANkTQGpotbdRr6c2AzpVotzBfHSfx5Zdt29xcYZQFvAmY3KT9yXA1JEJgtvLy65ptUR1iUPFWA8mu2AhrxSQHsNhcz4tXedaBHnjSYhHhWhJq5brsyR2M7fBAPWxNtNSZVdzG129sJYG5BWmqucqH1UdeYfaUKVzEXxt53ta6SGe5RTKrAYjr3x8NbfDB3ZXwU4udttVZrVDCNMWo4HJMJH9E5rdFeuovaD3L8nrQR4PCBpHm"
    },
    "id": 9
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

### Verify Success

Now we can call [`platform.getPendingValidators`](../api/platform.md#platformgetpendingvalidators) and see that the node has been added to the Subnet's pending validator set.

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.getPendingValidators",
    "params": {
        "subnetID":"nTd2Q2nTLp8M9qv2VKHMdvYhtNWX7aTPa4SMEK7x7yJHbcWvr"
    },
    "id": 10
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The response confirms the node has been added:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "validators": [
            {
                "id": "ARCLrphAHZ28xZEBfUL7SVAmzkTZNe1LK",
                "startTime":1584042912,
    	        "endTime":1584121156,
                "weight": "1"
            }
        ]
    },
    "id": 10
}
```

When the time reaches `1584042912`, this node will start validating the Subnet.
