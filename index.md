---
title: API Reference

language_tabs: # must be one of https://prismjs.com/#supported-languages
  - shell

toc_footers:
  - <a href='https://github.com/kurumiimari/gohan'>View Gohan Source</a>
  - <a href='https://github.com/kurumiimari/gohan-docs'>View Docs Source</a>
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:

search: true
code_clipboard: true
---

# Introduction

Gohan is an alternative Handshake wallet node. Its goals are the following:

1. Provide a high-performance, even on low-specced devices.
2. Expose a unified, RESTful wallet management API.
3. Securely custody names and funds without sacrificing user experience.
4. Make it easy to embed Gohan into other applications.

The Gohan binary contains both the wallet server as well as a CLI client.

# Installation

Follow these steps to install Gohan as a standalone wallet daemon:

1. Download the appropriate release for your OS/architecture.
2. Verify the download (optional, but recommended.)
3. Move Gohan onto your `$PATH`.

Gohan has no external dependencies, so you can run it right away by running `gohan start`. By default, it will try to connect to a mainnet Handshake node running on your local machine. If this isn't your setup, you can specify the following flags to configure how Gohan connects to the Handshake network:

- `--network`: Can be `main`, `simnet`, `regtest`, or `testnet`.
- `--server-url`: Full URL to a Handshake full node. Useful if your full node is running on a non-standard port. This is common for hosted nodes, which run over HTTPS on port 443.
- `--server-api-key`: API key for the Handshake full node.

<aside class="notice">
  If you are running your own <code>hsd</code> node, make sure to run it with the <code>--no-wallet</code> flag.
</aside>

# Authentication

> To start Gohan in API key mode:


```shell
gohan --api-key your-api-key-here start
```

> To talk to Gohan using cURL:

```shell
curl -H 'X-API-Key: your-api-key-here' http://localhost:12039
```

> To talk to Gohan using the Gohan CLI:

```shell
gohan --wallet-api-key your-api-key-here status
```

Gohan has two levels of authentication: API key authentication, and wallet authentication.

To specify an API key, use the `--wallet-api-key` flag when running `gohan start`. When making requests, include the API key in a header that looks like the one below:

`X-API-Key: your-api-key-here`.

If you're using the Gohan CLI to make requests, use the `--wallet-api-key` flag to include the API key.

# Node

## Get Node Status

> Via CLI:

```shell
gohan status
```

> Via cURL:

```shell
curl http://localhost:12039/api/v1/status
```

> Returns JSON structured like this:

```json
{
  "status": "OK",
  "height": 71614,
  "mem_usage": 8945720,
  "version": ""
}
```

Returns information about the wallet node's status.

### HTTP Request

`GET /api/v1/status`


# Wallets

A single Gohan instance can support multiple wallets. Each wallet is identified by a name, which the user specifies during wallet creation. By default, the CLI specifies a wallet ID of `primary` for all CLI commands. This can be changed by using the `--wallet/-w` flags.

Unlike `hsd`, Gohan does not create a default wallet on startup.

## Create Wallet

> Import a mnemonic via CLI:

```shell
gohan -w your-wallet-name import
```

> Import a mnemonic via cURL:

```shell
curl -X POST --data '{"name": "your-wallet-name", "mnemonic": "your-mnemonic-here", "password": "your-password-here"}'
```

> JSON response:

```json
{
  "name": "your-wallet-name",
  "watch_only": false
}

```

> Create a wallet via CLI:

```shell
gohan -w your-wallet-name create
```

> Create a wallet via cURL:

```shell
curl -X POST --data '{"name": "your-wallet-name", "password": "your-password-here"}'
```

> JSON response:

```json
{
  "name": "your-wallet-name",
  "mnemonic": "drum virus cave hurdle memory awful cable frog pulse avocado picnic rifle tourist job usage audit inject hill sport love merry mango uniform evil",
  "watch_only": false
}
```


Creates a wallet. Gohan supports creating wallets in three different ways:

1. By importing an existing BIP39 mnemonic.
2. By importing an existing extended public key.
3. By generating a wallet with a random seed phrase.

In all cases, the endpoint requires a password to encrypt the wallet.

The CLI commands are interactive, and will ask you to paste in your mnemonic and create a password via STDIN.

### HTTP Request

`POST /api/v1/wallets`

### JSON Parameters

Parameter | Default | Description
--------- | ------- | -----------
name | primary | Sets the wallet's name.
mnemonic | none | If set, imports the given BIP39 mnemonic. If neither xpub nor mnemonic is specified, a mnemonic will be generated for you and returned.
xpub | none | If set, imports the given extended public key. The returned wallet will be watch only.


## List Wallets

> Via CLI:

```shell
gohan wallets
```

> Via cURL:

```shell
curl http://localhost:12039/api/v1/wallets
```

> JSON response:

```json
{
  "wallets": [
    "primary"
  ]
}
```

Lists the wallets Gohan knows about.

### HTTP Request

`GET /api/v1/wallets`

## List Wallet Accounts

> Via CLI:

```shell
gohan accounts
```

> Via cURL:

```shell
curl https://localhsot:12039/api/v1/wallets/:wallet_name/accounts
```

> JSON response:

```json
{
  "accounts": [
    "default"
  ]
}
```

Lists the BIP-44 accounts this wallet knows about.

### HTTP Request

`GET /api/v1/wallets/:wallet_name/accounts`


## Unlock

> Via CLI:

```shell
gohan unlock
```

> Via cURL:

```shell
curl -X POST --data '{"password": "your-password"}' https://localhost:12039/api/v1/wallets/:wallet_name/unlock
```

> Response code:

```
204 No Content
```

Unlocks the wallet and all its associated accounts.

### HTTP Request

`POST /api/v1/wallets/:wallet_name/unlock`

## Lock

> Via CLI:

```shell
gohan lock
```

> Via cURL:

```shell
curl -X POST https://localhost:12039/api/v1/wallets/:wallet_name/lock
```

> Response code:

```
204 No Content
```

Locks the wallet and all its associated accounts.

### HTTP Request

`POST /api/v1/wallets/:wallet_name/lock`

# Accounts


## Get Info

> Via CLI:

```shell
gohan info
```

> Via cURL:

```shell
curl http://localhost:12039/api/v1/wallets/:wallet_name/accounts/:account_name
```

> JSON response:

```json
{
  "name": "default",
  "index": 0,
  "wallet_name": "primary",
  "balances": {
    "available": 11939946000,
    "immature": 0,
    "bid_locked": 0,
    "reveal_locked": 0,
    "name_locked": 0
  },
  "address_depth": {
    "receive": 2,
    "change": 4
  },
  "lookahead_depth": {
    "receive": 1001,
    "change": 1003
  },
  "receive_address": "rs1qztdtxtrcdvkyxsk5r8fhyymh0mw6ju338pe9wg",
  "change_address": "rs1qmycfs2zf4mtsx5q5y04dpe00jyx3run76mt8nn",
  "xpub": "rpubKBBUaydwRpVxLcm8YESMRikrSFRG9nsXDquhppVigpKymkS6fhoKxxJa1Ud76TgHUMMrvAvqJXyxkJKjWdmX6uSkQNYKHnuqDnDsLSVyVQnQ",
  "rescan_height": 36
}
```

Returns balance, address, and rescan info about an account.

### HTTP Request

`GET /api/v1/wallets/:wallet_name/accounts/:account_name`

### Balance Types

Determining a Handshake wallet's balance is complex, since coins can be locked up for extended periods of time in name auctions. Gohan tries to make this simpler by returning four different balance types:

Balance Type | Description
------------ | -----------
Available | Funds available to be spent.
Immature | Coinbases or airdrops that will be spendable once they mature.
Bid Locked | Funds locked in bids.
Reveal Locked | Funds locked in reveals.
Name Locked | Funds burned after winning a name auction.


## Get Transactions

> Via CLI:

```shell
gohan transactions [count] [offset]

# or, alternatively:

gohan txs [count] [offset]
```

> Via cURL:

```shell
curl http://localhost:12039/api/v1/wallets/:wallet_name/accounts/:account_name/transactions?count=<count>&offset=<offset>
```

> JSON response:

```json
{
  "transactions": [
    {
      "hash": "2840c896226bb6135a04d6b4103eb71ea443f7b1ba075090bd24dc02f243222a",
      "height": 25,
      "block": "55400661261f207e36b746300831aa32d8d16ea3fd969cc7f6df6fa7d8841280",
      "time": 1623358754,
      "index": 0,
      "version": 0,
      "inputs": [
        {
          "prevout": {
            "hash": "0000000000000000000000000000000000000000000000000000000000000000",
            "index": 4294967295
          },
          "witness": [
            "6d696e656420627920687364",
            "f0ba89cd353becd5",
            "0000000000000000"
          ],
          "sequence": 450522216,
          "coin": null
        }
      ],
      "outputs": [
        {
          "value": 2000000000,
          "address": {
            "address": "rs1qedtqrtu8eavsl7sepgy3fp336966pqxmyhnquc",
            "derivation": "m/0/0",
            "own": true
          },
          "covenant": {
            "type": 0,
            "action": "NONE",
            "items": []
          }
        }
      ],
      "hex": "00000000010000000000000000000000000000000000000000000000000000000000000000ffffffff686cda1a0100943577000000000014cb5601af87cf590ffa190a09148631d175a080db000019000000030c6d696e65642062792068736408f0ba89cd353becd5080000000000000000",
      "fee": 0
    }
  ],
  "count": 123
}
```

Lists the transactions involving this account.


### HTTP Request

`GET /api/v1/wallets/:wallet_name/accounts/:account_name/transactions`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
count | 50 | Number of transactions to return.
offest | 0 | Number of transactions to skip.


## Get Names

> Via CLI:

```shell
gohan names [count] [offset]
```

> Via cURL:

```shell
curl http://localhost:12039/api/v1/wallets/:wallet_name/accounts/:account_name/names?count=<count>&offset=<offset>
```

> JSON response:

```json
{
  "names": [
    {
      "name": "superdope",
      "hash": "61e4969faa9903e88ac3be9dfbd744d48864fca10eae31cefecb16bb4b24d922",
      "status": "UNOWNED"
    },
    {
      "name": "chips",
      "hash": "f11bfeae0c2007312ec8071fc66500cded6faeefeb1378237e23211d90049643",
      "status": "TRANSFERRED"
    },
    {
      "name": "shakedex",
      "hash": "e1dfddd001a6ff65e8a3aa565cc5014f6506ef60243f9155a44f0a472314bf8f",
      "status": "OWNED"
    },
    {
      "name": "silverhand",
      "hash": "ea175debd94bf73540e6a17c8f398ba36076507dde79dba6cc4dad5d532680a4",
      "status": "TRANSFERRED"
    }
  ],
  "count": 123
}
```

Lists names involving this account. This includes any name the account has opened, bid on, or won.

### HTTP Request

`GET /api/v1/wallets/:wallet_name/accounts/:account_name/names`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
count | 50 | Number of transactions to return.
offest | 0 | Number of transactions to skip.


## Get Name History

> Via CLI:

```shell
gohan name-history <name>
```

> Via cURL:

```shell
curl http://localhost:12039/api/v1/wallets/:wallet_name/accounts/:account_name/:name
```

> JSON response:

```json
{
  "history": [
    {
      "name": "whncsjjgtc",
      "type": "FINALIZE_OUT",
      "out_idx": 0,
      "parent_tx_hash": "05e52af86197be24b2a5d87d00209ecc18dff4af8ade10fb9cc3150bcd657730",
      "parent_out_idx": 0,
      "transaction": {}
    },
    {
      "name": "whncsjjgtc",
      "type": "TRANSFER",
      "out_idx": 0,
      "parent_tx_hash": null,
      "parent_out_idx": null,
      "transaction": {}
    },
    {
      "name": "whncsjjgtc",
      "type": "REGISTER",
      "out_idx": 0,
      "parent_tx_hash": null,
      "parent_out_idx": null,
      "transaction": {}
    },
    {
      "name": "whncsjjgtc",
      "type": "REDEEM",
      "out_idx": 0,
      "parent_tx_hash": "c83ffe91abd049a2f0852ab2f43f9b46771ca3e0efe773a74d079952cecf3ce4",
      "parent_out_idx": 1,
      "transaction": {}
    },
    {
      "name": "whncsjjgtc",
      "type": "REVEAL",
      "out_idx": 0,
      "parent_tx_hash": "17fbc4892fba7f9d15aea9089366c97e770f4b0e0b8187b881b2739007c255ee",
      "parent_out_idx": 0,
      "transaction": {}
    },
    {
      "name": "whncsjjgtc",
      "type": "REVEAL",
      "out_idx": 1,
      "parent_tx_hash": "4a4f5fdcacc50b5cfa15ecdaea7d6e67d4be8a99e16c1ddb9bd53f996f13607f",
      "parent_out_idx": 0,
      "transaction": {}
    },
    {
      "name": "whncsjjgtc",
      "type": "BID",
      "out_idx": 0,
      "bid_amount": 123,
      "parent_tx_hash": null,
      "parent_out_idx": null,
      "transaction": {}
    },
    {
      "name": "whncsjjgtc",
      "type": "BID",
      "out_idx": 0,
      "bid_amount": 234,
      "parent_tx_hash": null,
      "parent_out_idx": null,
      "transaction": {}
    },
    {
      "name": "whncsjjgtc",
      "type": "OPEN",
      "out_idx": 0,
      "parent_tx_hash": null,
      "parent_out_idx": null,
      "transaction": {}
    }
  ]
}
```

<aside class="notice">
  The returned transaction data is identical to the get transactions endpoint. Transaction data has been removed from the JSON response for brevity.
</aside>

Returns the history between this account and the provided name. The following events are tracked:

Event | Description
----- | -----------
OPEN | This wallet opened the name.
BID | This wallet bid on the name.
REVEAL | This wallet revealed a bid on the name.
REDEEM | This wallet redeemed a bid on the name.
REGISTER | This wallet registered the name.
TRANSFER | This wallet initiated a transfer of the name.
FINALIZE_OUT | This wallet finalized the transfer of the name to someone else.
FINALIZE_IN | This wallet received this name after someone else finalized a transfer.
REVOKE | This wallet revoked the name.

### HTTP Request

`GET /api/v1/wallets/:wallet_name/accounts/:account_name/names/:name`

## Get Unspent Bids

> Via CLI:

```shell
gohan unspent-bids
```

> Via cURL:

```shell
curl http://localhost:12037/api/v1/wallets/:wallet_name/accounts/:account_name/unspent_bids
```

> JSON response:

```json
{
  "unspent_bids": [
    {
      "name": "foobar",
      "block_height": 1234,
      "lockup": 10000,
      "bid_value": 5000,
      "tx_hash": "61e4969faa9903e88ac3be9dfbd744d48864fca10eae31cefecb16bb4b24d922",
      "out_idx": 1,
      "revealable": true,
      "revealable_block": 1230
    }
  ]
}
```

Lists all bids that haven't been revealed yet.

### HTTP Request

`GET /api/v1/wallets/:wallet_name/accounts/:account_name/unspent_bids`

## Get Unspent Reveals

> Via CLI:

```shell
gohan unspent-reveals
```

> Via cURL:

```shell
curl http://localhost:12037/api/v1/wallets/:wallet_name/accounts/:account_name/unspent_reveals
```

> JSON response:

```json
{
  "unspent_reveals": [
    {
      "name": "foobar",
      "block_height": 1234,
      "lockup": 10000,
      "bid_value": 5000,
      "tx_hash": "61e4969faa9903e88ac3be9dfbd744d48864fca10eae31cefecb16bb4b24d922",
      "out_idx": 1,
      "redeemable": true,
      "redeemable_block": 1230
    }
  ]
}
```

Lists all reveals that haven't been redeemed yet.

### HTTP Request

`GET /api/v1/wallets/:wallet_name/accounts/:account_name/unspent_reveals`

## Rescan

> Via CLI:

```shell
gohan rescan <height>
```

> Via cURL:

```shell
curl -X POST --data '{"height": 1234}' http://localhost:12037/api/v1/wallets/:wallet_name/accounts/:account_name/rescan
```

> Return code:

```
204 No Content
```

Rolls the account back to the specified block, then rescans transactions.

### HTTP Request

`POST /api/v1/wallets/:wallet_name/accounts/:account_name/rescan`

### JSON Parameters

Parameter | Default | Description
--------- | ------- | -----------
height | 1 | The height to roll back to.

## Zap

> Via CLI:

```shell
gohan zap
```

> Via cURL:

```shell
curl -X POST http://localhost:12037/api/v1/wallets/:wallet_name/accounts/:account_name/zap
```

> Return code:

```
204 No Content
```

Removes pending transactions. Useful is you have a transaction that is "stuck" in the mempool.

### HTTP Request

`POST /api/v1/wallets/:wallet_name/accounts/:account_name/zap`

## Sign Message

> Via CLI:

```shell
gohan sign-message <address> <message>
```

> Via cURL:

```shell
curl -X POST --data '{"address": "rs1qedtqrtu8eavsl7sepgy3fp336966pqxmyhnquc", "message": "falafels"}' http://localhost:12037/api/v1/wallets/:wallet_name/accounts/:account_name/sign_message
```

> JSON response:

```json
{
  "signature": "KJfZtbcgAKzlBKvo5rvQ0Mf9nJY1HtwVnNaGDId2BnF7SDoK7Myu/UGE859tO69hEA4NWQBeFx2ifi0oDIX5eA=="
}
```

Signs a message using the private key associated with the specified address.

### HTTP Request

`GET /api/v1/wallets/:wallet_name/accounts/:account_name/sign_message`

### JSON Parameters

Parameter | Default | Description
--------- | ------- | -----------
address | none | The address whose private key will be used to sign the message.
message | none | The message to sign.

## Sign Message With Name

> Via CLI:

```shell
gohan sign-message-with-name <name> <message>
```

> Via cURL:

```shell
curl -X POST --data '{"name": "foobar", "message": "falafels"}' http://localhost:12037/api/v1/wallets/:wallet_name/accounts/:account_name/sign_message_with_name
```

> JSON response:

```json
{
  "signature": "KJfZtbcgAKzlBKvo5rvQ0Mf9nJY1HtwVnNaGDId2BnF7SDoK7Myu/UGE859tO69hEA4NWQBeFx2ifi0oDIX5eA=="
}
```

Signs a message using the private key associated with the address that owns the specified name.

### HTTP Request

`GET /api/v1/wallets/:wallet_name/accounts/:account_name/sign_message_with_name`

### JSON Parameters

Parameter | Default | Description
--------- | ------- | -----------
name | none | The name whose owner's private key will be used to sign the message.
message | none | The message to sign.


# Sending Transactions

The following endpoints create and send transactions. Gohan must be unlocked before these endpoints will work.

All of the below endpoints optionally accept two additional JSON request parameters:

Parameter | Default | Description
--------- | ------- | -----------
fee_rate | HSD smart fee | Sets the fee rate for the transaction.
create_only | false | Create and fund the transaction, but do not send it.

All transaction endpoints return a transaction object formatted like those returned by the get transactions endpoint.

<aside class="warning">
  Just like in HSD, the CLI expresses monetary values in <em>whole HNS</em>. The API, on the other hand, expresses them in subunits.
</aside>

## Send Funds

> Via CLI:

```shell
gohan send <recipient-address> <amount-whole-hns> [override-fee-rate-subunits]
```

> Via cURL:

```shell
curl -X POST --data '{"value": 123, "address": "rs1qedtqrtu8eavsl7sepgy3fp336966pqxmyhnquc"}' http://localhost:12037/api/v1/wallets/:wallet_name/accounts/:account_name/sends
```

Sends funds to the specified address.

### HTTP Request

`POST /api/v1/wallets/:wallet_name/accounts/:account_name/sends`

### JSON Parameters:

Parameter | Default | Description
--------- | ------- | -----------
value | none | The amount to send.
address | none | The address to send funds to.
fee_rate | HSD smart fee | Sets the fee rate for the transaction.
create_only | false | Create and fund the transaction, but do not send it.

## Send Open

> Via CLI:

```shell
gohan open <name> [override-fee-rate-subunits]
```

> Via cURL:

```shell
curl -X POST --data '{"name": "foobar"}' http://localhost:12037/api/v1/wallets/:wallet_name/accounts/:account_name/opens
```

Opens a name for bidding.

### HTTP Request

`POST /api/v1/wallets/:wallet_name/accounts/:account_name/opens`

### JSON Parameters:

Parameter | Default | Description
--------- | ------- | -----------
name | none | The name to open for bidding.
fee_rate | HSD smart fee | Sets the fee rate for the transaction.
create_only | false | Create and fund the transaction, but do not send it.

## Send Bid

> Via CLI:

```shell
gohan bid <name> <bid-amount-whole-hns> <blind-amount-whole-hns> [override-fee-rate-subunits]
```

> Via cURL:

```shell
curl -X POST --data '{"name": "foobar", "value": 1234, "lockup": 4321}' http://localhost:12037/api/v1/wallets/:wallet_name/accounts/:account_name/bids
```

Places a bid on a name.

### HTTP Request

`POST /api/v1/wallets/:wallet_name/accounts/:account_name/bids`

### JSON Parameters:

Parameter | Default | Description
--------- | ------- | -----------
name | none | The name to bid on.
value | none | The real amount of the bid.
lockup | none | The "blinded" amount of the bid that will be broadcast on chain.
fee_rate | HSD smart fee | Sets the fee rate for the transaction.
create_only | false | Create and fund the transaction, but do not send it.


## Send Reveal

> Via CLI:

```shell
gohan reveal <name> [override-fee-rate-subunits]
```

> Via cURL:

```shell
curl -X POST --data '{"name": "foobar"}' http://localhost:12037/api/v1/wallets/:wallet_name/accounts/:account_name/reveals
```

Reveals all bids placed on a name.

### HTTP Request

`POST /api/v1/wallets/:wallet_name/accounts/:account_name/reveals`

### JSON Parameters:

Parameter | Default | Description
--------- | ------- | -----------
name | none | The name to reveal bids for.
fee_rate | HSD smart fee | Sets the fee rate for the transaction.
create_only | false | Create and fund the transaction, but do not send it.

## Send Redeem

> Via CLI:

```shell
gohan redeem <name> [override-fee-rate-subunits]
```

> Via cURL:

```shell
curl -X POST --data '{"name": "foobar"}' http://localhost:12037/api/v1/wallets/:wallet_name/accounts/:account_name/redeems
```

Redeems all losing reveals for a name. Redeeming a reveal makes its value available to be spent again.

### HTTP Request

`POST /api/v1/wallets/:wallet_name/accounts/:account_name/redeems`

### JSON Parameters:

Parameter | Default | Description
--------- | ------- | -----------
name | none | The name to redeem reveals for.
fee_rate | HSD smart fee | Sets the fee rate for the transaction.
create_only | false | Create and fund the transaction, but do not send it.

## Send Update

> Via CLI:

```shell
gohan update <name> <resource> [override-fee-rate-subunits]
```

> Via cURL:

```shell
curl -X POST --data '{"name": "foobar", "resource": {"records": [{"type": "TXT", "txt": ["hello"]}]}}' http://localhost:12037/api/v1/wallets/:wallet_name/accounts/:account_name/updates
```

Updates a name's DNS records. The format of the `resource` parameter is identical to the parameter in HSD.

### HTTP Request

`POST /api/v1/wallets/:wallet_name/accounts/:account_name/updates`

### JSON Parameters:

Parameter | Default | Description
--------- | ------- | -----------
name | none | The name to redeem reveals for.
resource | none | The name's DNS records.
fee_rate | HSD smart fee | Sets the fee rate for the transaction.
create_only | false | Create and fund the transaction, but do not send it.

## Send Transfer

> Via CLI:

```shell
gohan transfer <name> <recipient> [override-fee-rate-subunits]
```

> Via cURL:

```shell
curl -X POST --data '{"name": "foobar", "address": "rs1qedtqrtu8eavsl7sepgy3fp336966pqxmyhnquc"}' http://localhost:12037/api/v1/wallets/:wallet_name/accounts/:account_name/transfers
```

Transfers a name to a different address.

### HTTP Request

`POST /api/v1/wallets/:wallet_name/accounts/:account_name/transfers`

### JSON Parameters:

Parameter | Default | Description
--------- | ------- | -----------
name | none | The name to transfer.
address | none | The address of the recipient.
fee_rate | HSD smart fee | Sets the fee rate for the transaction.
create_only | false | Create and fund the transaction, but do not send it.

## Send Finalize

> Via CLI:

```shell
gohan finalize <name> [override-fee-rate-subunits]
```

> Via cURL:

```shell
curl -X POST --data '{"name": "foobar"}' http://localhost:12037/api/v1/wallets/:wallet_name/accounts/:account_name/finalizes
```

Finalizes a name transfer.

### HTTP Request

`POST /api/v1/wallets/:wallet_name/accounts/:account_name/finalizes`

### JSON Parameters:

Parameter | Default | Description
--------- | ------- | -----------
name | none | The name to finalize.
fee_rate | HSD smart fee | Sets the fee rate for the transaction.
create_only | false | Create and fund the transaction, but do not send it.

## Send Revoke

> Via CLI:

```shell
gohan revoke <name> [override-fee-rate-subunits]
```

> Via cURL:

```shell
curl -X POST --data '{"name": "foobar"}' http://localhost:12037/api/v1/wallets/:wallet_name/accounts/:account_name/revokes
```

Revokes a name, releasing it back to the protocol for auction.

### HTTP Request

`POST /api/v1/wallets/:wallet_name/accounts/:account_name/revokes`

### JSON Parameters:

Parameter | Default | Description
--------- | ------- | -----------
name | none | The name to revoke.
fee_rate | HSD smart fee | Sets the fee rate for the transaction.
create_only | false | Create and fund the transaction, but do not send it.
