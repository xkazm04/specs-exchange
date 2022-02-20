e---
tags: [exchange, ledger, tatum]
---

# How to build a crypto exchange

*Build a crypto exchange backend in half an hour.*

---

## Introduction

A crypto exchange is a web application where users can trade their crypto assets. The exchange operator is the one who owns private keys to all of the user's crypto assets - in this case, we are talking about a custodial exchange leveraging a custodial wallet.

https://www.youtube.com/watch?v=pVBNvh04ixo&t=1s&ab_channel=Tatum

<!-- theme: info -->
>A custodial wallet is a wallet whose private keys are held by a third party, not by the crypto assets owner. This third party - in this case, the exchange operator - has full control over crypto assets, while users only have permission to send or receive payments.

Every exchange must have wallets for every crypto asset it supports. Every exchange user must obtain an account for every asset they are trading. The exchange operator defines the trading pairs that can be traded by users and usually charges a fee for every trade performed.

‌Trades are not performed on the blockchain (on-chain), as this would be extremely slow and expensive. All trades are in fact virtual transactions between user accounts.

---

‌There are four logical groups of actions to create an exchange:
- **[Application setup](#application-setup)** -  this includes prerequisites like creating blockchain wallets or generating exchange service accounts for gathering fees
- **[New user registration](#new-user-registration)** - steps that need to be taken when a new user registers in the exchange
- **[User application journey](#user-application-journey)** - actions that users take while in the application
- **[Trading](#trading)** - enabling users to trade their assets

<!-- theme: info -->
>In the following sections, we will use the Bitcoin and Ethereum blockchains to demonstrate the functionalities.

---

### Application setup

This is a one-time step that must be taken before launching the exchange. It involves [creating wallets](../blockchain/b3A6MjgzNjM1MTc-generate-wallet-and-address) for the blockchains your application will support or creating service fee accounts for the wallet provider.

#### Creating blockchain wallets

The example below shows how to [generate a Bitcoin wallet](../blockchain/b3A6MjgzNjM1MTc-generate-wallet-and-address). The path contains a `chainId` parameter which is used to indicate which blockchain you want to create the wallet for. 

<!-- theme: info -->
> Do not fill in the `address` parameter if you don't want to generate an address at the same time.

**Request example**
```json
curl --request POST \
  --url https://api-eu1.tatum.io/v4/blockchain/BTC/account \
  --header 'Content-Type: application/json' \
  --header 'env: ' \
  --header 'x-api-key: ' \
  --data '{
  "Account": {
    "mnemonic": "urge pulp usage sister evidence arrest palm math please chief egg abuse"
  },
}
```
**Response example**
```json
{
  "Account": {
    "secret": "snSFTHdvSYQKKkYntvEt8cnmZuPJB"
    },
  }
```
<!-- theme: info -->
>To generate a wallet for any ERC-20 token like USDT, LINK or others, use the same call as for Ethereum. These tokens are transported on the Ethereum network and leverage the same Ethereum addresses as native Ether.

<!-- theme: warning -->
>In these examples, blockchain wallets are created using the API, which is not a secure way of generating wallets. Your private keys and mnemonics should never leave your security perimeter. To generate a wallet correctly and securely, you can use the [Tatum CLI](https://github.com/tatumio/tatum-cli) or our complex key management system, [Tatum KMS](https://github.com/tatumio/tatum-kms).

#### Generating service accounts

A service ledger account should be created for every supported blockchain wallet. These accounts will be used to gather fees for the users' trades. A fee will be charged for every performed trade, and it will be transferred to the corresponding ledger account.

‌In Tatum, every account belongs to a specific customer. A customer represents an entity containing information about a user of your application, such as the customer's country of residence, accounting currency, etc. The customer is only created in the process of generating a new account, and the only required field is the external ID. All service accounts can be connected to a single service customer.

<!-- theme: info -->
>Accounting currency is part of Tatum's built-in compliance engine. It's enabled by default.

Every account should have a properly set up accounting currency. The value should represent the FIAT currency of the country where the accounting is performed. For example, for an exchange in Germany, accounting should be kept in Euro, and so the accounting currency value should be EUR. 

---

## New user registration

After the configuration has been completed and the exchange is live, users can register in the ecosystem. 

### Creating user accounts

When a new user signs up for the application, [ledger accounts must be created](../virtualAccounts/b3A6MzEwNDI1MzQ-create-account) for them. Every user should have an account for every supported blockchain asset in the exchange. Every account should be created with the customer's external ID. This makes it possible to list all accounts for a specific customer. Accounting currency should also be set up correctly in an account.

<!-- theme: info -->
>The customer's external ID should be a unique identifier of the user in your application, e.g., your ID or the hash.

Every account in the private ledger must have a defined currency. The currency cannot be changed in the future. During account creation, the xpub from the blockchain wallet must be entered. This is the first connection between the blockchain and the ledger.

<!-- theme: info -->
>Xpubs from the wallets generated in the application setup process will be used.

**Request example**
```json
curl --request POST \
  --url https://api-eu1.tatum.io/v4/enterprise/account \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: ' \
  --data '{
  "currency": "BTC",
  "xpub": "xpub6EsCk1uU6cJzqvP9CdsTiJwT2rF748YkPnhv5Qo8q44DG7nn2vbyt48YRsNSUYS44jFCW9gwvD9kLQu9AuqXpTpM1c5hgg9PsuBLdeNncid",
  "customer": {
    "accountingCurrency": "USD",
    "customerCountry": "US",
    "externalId": "123654",
    "providerCountry": "US"
  },
  "compliant": false,
  "accountCode": "AC_1011_B",
  "accountingCurrency": "AED",
  "accountNumber": "123456"
}'
```
**Response example**
```json
{
  "id": "5e68c66581f2ee32bc354087",
  "balance": {
    "accountBalance": "1000000",
    "availableBalance": "1000000"
  },
  "currency": "BTC",
  "frozen": false,
  "active": true,
  "customerId": "5e68c66581f2ee32bc354087",
  "accountCode": "03_ACC_01",
  "xpub": "xpub6FB4LJzdKNkkpsjggFAGS2p34G48pqjtmSktmK2Ke3k1LKqm9ULsg8bGfDakYUrdhe2EHw5uGKX9DrMbrgYnVfDwrksT4ZVQ3vmgEruo3Ka"
}
```
### Generating a blockchain deposit address for the account
When an account is created, it is not yet synchronized with the blockchain. There is no blockchain address connected to it, only a blockchain wallet from which addresses will be chosen. To connect a specific address, you need to [generate an account address](../blockchain/b3A6MjgzNjM1MTc-generate-wallet-and-address) using the off-chain method. The path contains a `chainId` parameter which is used to indicate which blockchain you want to create the address for. Next, fill out the address parameters in the request body.

**Request example**
```json
curl --request POST \
  --url https://api-eu1.tatum.io/v4/blockchain/BTC/account \
  --header 'Content-Type: application/json' \
  --header 'env: ' \
  --header 'x-api-key: ' \
  --data '{
  "Address": {
    "xpub": "xpub6EsCk1uU6cJzqvP9CdsTiJwT2rF748YkPnhv5Qo8q44DG7nn2vbyt48YRsNSUYS44jFCW9gwvD9kLQu9AuqXpTpM1c5hgg9PsuBLdeNncid",
    "index": 0
  }
}'
```
**Response example**
```json
{
  "Address": {
    "address": "2MsM67NLa71fHvTUBqNENW15P68nHB2vVXb",
    "xpub": "xpub6EsCk1uU6cJzqvP9CdsTiJwT2rF748YkPnhv5Qo8q44DG7nn2vbyt48YRsNSUYS44jFCW9gwvD9kLQu9AuqXpTpM1c5hgg9PsuBLdeNncid",
    "index": 0
  }
}
```
The response contains a blockchain address that has been connected to the ledger account. Any incoming blockchain transactions to this address will be automatically synchronized to the private ledger.

### Enabling notifications for incoming blockchain transactions
It is possible to [enable webhook notifications](../virtualAccounts/b3A6MjgwMDUxNzI-create-new-subscription) for every incoming transaction to the account. This notification is fired as an HTTP POST request with a JSON body and contains fields like transaction amount, currency, and account of the incoming transaction. A user's wallet page should indicate that there are pending incoming transactions - their crypto deposits.

**Request example**
```json
curl --request POST \
  --url https://api-eu1.tatum.io/v4/tatum/subscription \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: ' \
  --data '{
  "type": "ACCOUNT_INCOMING_BLOCKCHAIN_TRANSACTION",
  "attr": {
    "id": "5e6be8e9e6aa436299950c41",
    "url": "https://webhook.tatum.io/account"
  }
}'
```
**Response example**
```json
{
  "id": "5e68c66581f2ee32bc354087"
}
```
The subscription `id` obtained from the response can be later used to delete the subscription.

---

## User application journey

The user has successfully signed up and accounts and deposit addresses for all supported blockchains have been created. Let's take a look at what the application might show.

### List of the user's accounts with balances

<!-- theme: info -->
>The account balance is available in the account list by default and does not have to be queried separately. There are two balance types:
>- **Account balance** is the total balance of the account without any pending deposits or other trade blockages. 
> - **Available balance** is the balance that can be used for trading or other transaction types.

**Request example**
```json
curl --request GET \
  --url https://api-eu1.tatum.io/v4/tatum/account/customer/id \
  --header 'Content-Type: application/json'

```

**Response example**
```json
[
  {
    "id": "5e68c66581f2ee32bc354087",
    "balance": {
      "accountBalance": "1000000",
      "availableBalance": "1000000"
    },
    "currency": "BTC",
    "frozen": false,
    "active": true,
    "customerId": "5e68c66581f2ee32bc354087",
    "accountCode": "03_ACC_01",
    "xpub": "xpub6FB4LJzdKNkkpsjggFAGS2p34G48pqjtmSktmK2Ke3k1LKqm9ULsg8bGfDakYUrdhe2EHw5uGKX9DrMbrgYnVfDwrksT4ZVQ3vmgEruo3Ka"
  }
]
```

### List of recent transactions in any account

Usually, the last transactions that happened in any of the accounts are presented as well.

**Request example**

```json
curl --request GET \
  --url https://api-eu1.tatum.io/v4/blockchain/chainId/transaction/id \
  --header 'Content-Type: application/json' \
  --header 'env: ' \
  --header 'x-api-key: '
```

**Response example**

```json
[
  {
    "block": {
      "hash": "000ca231a439a5c0a86a5a5dd6dc1918a8",
      "height": 500124
    },
    "hash": 1,
    "index": "d1c75a84e4bdf0dd9bf1bcd0ce4fb25f89e2ed3c5e9574dbca2760b52c428717",
    "fee": {
      "price": 50,
      "limit": 50000,
      "currency": "CELO"
    },
    "addressFrom": "2MsM67NLa71fHvTUBqNENW15P68nHB2vVXb",
    "addressTo": "2MsM67NLa71fHvTUBqNENW15P68nHB2vVXb",
    "token": {
      "contractAddress": "2MsM67NLa71fHvTUBqNENW15P68nHB2vVXb",
      "currencySymbol": "BTC",
      "amount": 0
    },
    "status": "true",
    "timestamp": "2021-21-10 07:36:05"
  }
]
```

### Withdrawing funds from the exchange to blockchain

For every blockchain, there is a specific [API call](../virtualAccounts/b3A6MjgwOTI1Njk-create-withdrawal) for performing withdrawals. We will cover Bitcoin in this section, but it works similarly in others.

**Request example**

```json
curl --request POST \
  --url https://api-eu1.tatum.io/v4/tatum/withdrawal \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: ' \
  --data '{
  "senderAccountId": "5e68c66581f2ee32bc354087",
  "address": "mpTwPdF8up9kidgcAStriUPwRdnE9MRAg7",
  "amount": "0.001",
  "attr": "12345",
  "compliant": false,
  "fee": "0.0005",
  "multipleAmounts": [
    "0.1"
  ],
  "paymentId": "12345",
  "senderNote": "Sender note"
}'
```

**Response example**

```json
{
  "reference": "5e6be8e9e6aa436299950c41",
  "data": [
    {
      "address": {
        "address": "7c21ed165e294db78b95f0f181086d6f",
        "currency": "BTC",
        "derivationKey": 2147483647,
        "xpub": "xpub6FB4LJzdKNkkpsjggFAGS2p34G48pqjtmSktmK2Ke3k1LKqm9ULsg8bGfDakYUrdhe2EHw5uGKX9DrMbrgYnVfDwrksT4ZVQ3vmgEruo3Ka",
        "destinationTag": 5,
        "memo": "5",
        "message": "5"
      },
      "amount": 0,
      "vIn": "string",
      "vInIndex": 0,
      "scriptPubKey": "string"
    }
  ],
  "id": "5e68c66581f2ee32bc354087"
}
```

You can see that the required parameters include the ledger account identifier, information about the blockchain wallet, the recipient blockchain address, the amount to be sent, and the blockchain fee to be paid.

---

## Trading

Last but not least is the ability to perform trades. Keep in mind that this section describes exchanges like Binance or Coinbase, where the exchange provider is responsible for every trading pair's liquidity that the exchange supports.

<!-- theme: info -->
>You can picture pair liquidity as the number of open buy/sell trades present on the Order book and and as the volume that is traded throughout the day. The higher the liquidity, the more precise the chart, and the higher the accuracy of the asset price.

### Opening a new trade

Every user can open an unlimited number of trades. Trades can be of type BUY or SELL and they are connected to a specific trading pair. Trading pairs are created automatically with the first opened trade.

<!-- theme: info -->
>A trading pair consists of two assets. Take a BTC/ETH pair, for example. The first asset is Bitcoin, the second is Ethereum. When you open a new BUY trade with the pair BTC/ETH, it means that you want to buy Bitcoin and pay with your Ethereum.

Every trade must have a price and an amount of the asset you want to trade. In Tatum, every trade is a LIMIT trade by default, and you have to wait until the price hits your target.

<!-- theme: info -->
>Say you want to buy 1 Bitcoin for 40 Ethereum. You must open a BUY BTC/ETH trade with the price set to 40 and the amount set to 1. Your trade remains open until an opposite trade is opened. This opposite trade should be SELL BTC/ETH with the price set to 40 or below and the amount set to 1 or more.

<!-- theme: info -->
>MARKET trades can be executed by setting the price above or below the highest BUY or lowest SELL.

When you open a trade, there are two accounts you must enter:
- the ledger account with the currency of the first asset in the trading pair
- the ledger account with the currency of the second asset in the trading pair
The traded amount will be blocked and debited from one of these accounts based on the trade type, and the other account will be credited with the traded asset when the trade is fulfilled.

<!-- theme: info -->
>Let's say you want to buy 1 BTC for 40 ETH in a BTC/ETH pair. 40 ETH will be blocked from your ETH account and then transferred to the ETH account of the opposite SELL trade. 1 BTC will be credited to your BTC account after being transferred from the BTC account of the opposite SELL trade.

Every trade must be fulfilled and closed at some point. It is, however, possible to execute only a part of a trade. There might be a trade open for selling 1 BTC, but an opposing trade is set to buy only 0.6 BTC. The selling trade will be executed partially and stays open until the remaining 0.4 BTC is sold and the trade is thus closed.

<!-- theme: info -->
>A ledger transaction is performed for every partial trade fill, and assets associated with the trade are transferred to the ledger accounts. Also, the blockage is decreased accordingly.

Enough theory - let's open a [BUY trade](../exchange/b3A6Mjc2NTQzODc-store-buy-sell-trade).

**Request example**
```json
curl --request POST \
  --url https://api-eu1.tatum.io/v4/exchange/trade \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: ' \
  --data '{
  "type": "BUY",
  "price": "4350.4",
  "amount": "40",
  "pair": "ETH/EUR",
  "currency1AccountId": "7c21ed165e294db78b95f0f1",
  "currency2AccountId": "7c21ed165e294db78b95f0f1",
  "feeAccountId": "7c21ed165e294db78b95f0f1",
  "fee": 1.5,
  "attr": {
    "sealDate": 1572031674384,
    "percentBlock": 1.5,
    "percentPenalty": 1.5
  }
}'
```

**Response example**

```json
{
  "id": "5e68c66581f2ee32bc354087"
}
```


The response contains the open trade ID. When you list blockages on the ETH account, you will see that there is a blockage of 40 ETH. Additional information is present in the blockage response, such as the trade ID or a description.

**Request example**

```json
curl --request GET \
  --url https://api-eu1.tatum.io/v4/tatum/account/block/5e68c66581f2ee32bc354087 \
  --header 'Content-Type: application/json'
```

**Response example**

```json
[
  {
    "id": "5e68c66581f2ee32bc354087",
    "accountId": "5e68c66581f2ee32bc354087",
    "amount": "40",
    "type": "DEBIT_CARD_OP",
    "description": "Card payment in the shop."
  }
]
```

No transaction has been performed yet, only the blockage.

### Listing open trades

When you have open trades that are not yet filled, you can list them using the List active trades endpoints. You can list open BUY or SELL trades using two different endpoints. Keep in mind that, when using these endpoints, you can list either all open trades across all pairs or only trades for a specific account whose ID you provide as a query parameter.

To see a list of closed trades, you can call the List all closed trades endpoint.

**Request example**

```json
curl --request GET \
  --url https://api-eu1.tatum.io/v4/exchange/trade/id \
  --header 'Content-Type: application/json' \
  --header 'x-api-key: '
```

**Response example**

```json
{
  "id": "7c21ed165e294db78b95f0f1",
  "type": "BUY",
  "price": "4350.4",
  "amount": "40",
  "pair": "ETH/EUR",
  "isMaker": true,
  "fill": "1500",
  "feeAccountId": "7c21ed165e294db78b95f0f1",
  "fee": 1.5,
  "currency1AccountId": "7c21ed165e294db78b95f0f1",
  "currency2AccountId": "7c21ed165e294db78b95f0f1",
  "created": 1585170363103,
  "attr": {
    "sealDate": 1572031674384,
    "percentBlock": 1.5,
    "percentPenalty": 1.5
  }
}
```
---

There are many more things to enhance and features to implement, but this should be a good start for you and your exchange.

If you'd like to learn how to implement fiat currencies and securely work with private keys using Tatum KMS in your exchange, please continue to our **Crypto exchange part 2** workshop below.

https://www.youtube.com/watch?v=CGgyyTTv0yw&t=209s&ab_channel=Tatum