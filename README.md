# hermez_sdk

## Description

This is a flutter Plugin for Hermez Mobile SDK (https://hermez.io). This plugin provides a cross-platform tool (iOS, Android) to communicate with the Hermez API and network.

## Installation

To use this plugin, add `hermez_sdk` as a [dependency](https://flutter.io/using-packages/) in your `pubspec.yaml` file like this

```yaml
dependencies:
  hermez_sdk: ^x.y.z
```
This will get you the latest version.

If you want to test a specific branch of the repository, pull `hermez_sdk` like this

```yaml
dependencies:
  hermez_sdk:
      git:
        url: ssh://git@github.com/hermeznetwork/hermez-mobile-library.git
        ref: branchPathName
```

## Setup

NOTE: In order to interact with Hermez, you will need to supply your own Ethereum node. You can check these links to help you set up a node (https://blog.infura.io/getting-started-with-infura-28e41844cc89, https://blog.infura.io/getting-started-with-infuras-ethereum-api).

## Usage

To start using this package first import it in your Dart file.

```dart
import 'package:hermez_sdk/hermez_sdk.dart';
```

### Initialization

To initialize the Hermez SDK you can call the init method with one of the supported environments as a parameter, or setup all the different parameters passing the environment 'custom'.

```dart
HermezSDK.init(
  'rinkeby',
   web3ApiKey: EXAMPLES_WEB3_API_KEY
);
```

or 

```dart
HermezSDK.init(
  'custom',
  envParams: EnvParams(
      EXAMPLES_WEB3_CHAIN_ID,
      {
        ContractName.hermez: EXAMPLES_HERMEZ_ROLLUP_ADDRESS, // Hermez
        ContractName.withdrawalDelayer:
            EXAMPLES_HERMEZ_WDELAYER_ADDRESS, // WithdrawalDelayer
      },
      EXAMPLES_HERMEZ_API_URL,
      EXAMPLES_HERMEZ_EXPLORER_URL,
      EXAMPLES_WEB3_URL + EXAMPLES_WEB3_API_KEY,
      EXAMPLES_WEB3_RDP_URL + EXAMPLES_WEB3_API_KEY),
);
```

### Supported Tokens

Before being able to operate on the Hermez Network, we must ensure that the token we want to operate with is listed. For that we make a call to the Hermez Coordinator API that will list all available tokens. All tokens in Hermez Network must be ERC20.

We can see there are 2 tokens registered. ETH will always be configured at index 0. The second token is HEZ. For the rest of the examples we will work with ETH. In the future, more tokens will be included in Hermez.

```dart
import 'package:hermez_sdk/api.dart' as coordinatorApi;
import 'package:hermez_sdk/model/tokens_response.dart';

...

Future<TokensResponse> getHermezSupportedTokens() async {
  TokensResponse tokensResponse = await coordinatorApi.getTokens();
  return tokensResponse;
}
```

```json
{
  "tokens": [
    {
      "itemId": 1,
      "id": 0,
      "ethereumBlockNum": 0,
      "ethereumAddress": "0x0000000000000000000000000000000000000000",
      "name": "Ether",
      "symbol": "ETH",
      "decimals": 18,
      "USD": 1787,
      "fiatUpdate": "2021-02-28T18:55:17.372008Z"
    },
    {
      "itemId": 2,
      "id": 1,
      "ethereumBlockNum": 8153596,
      "ethereumAddress": "0x2521bc90b4f5fb9a8d61278197e5ff5cdbc4fbf2",
      "name": "Hermez Network Token",
      "symbol": "HEZ",
      "decimals": 18,
      "USD": 5.365,
      "fiatUpdate": "2021-02-28T18:55:17.386805Z"
    }
  ],
  "pendingItems": 0
}
```

### Create Wallet

We can create a new Hermez wallet by providing the Ethereum private key of an Ethereum account. This wallet will store the Ethereum and Baby JubJub keys for the Hermez account. The Ethereum address is used to authorize L1 transactions, and the Baby JubJub key is used to authorize L2 transactions. We will create two wallets.

> [!NOTE]
> You will need to supply two private keys to test and initialize both accounts. The keys provided here are invalid and are shown as an example.

```dart
import 'package:hermez_sdk/hermez_wallet.dart';

...

void createHermezWallets() async {
    // load first account
    final wallet =
        await HermezWallet.createWalletFromPrivateKey(EXAMPLES_PRIVATE_KEY1);
    final HermezWallet hermezWallet = wallet[0];
    final String hermezEthereumAddress = wallet[1];

    // load second account
    final wallet2 =
        await HermezWallet.createWalletFromPrivateKey(EXAMPLES_PRIVATE_KEY2);
    final HermezWallet hermezWallet2 = wallet2[0];
    final String hermezEthereumAddress2 = wallet2[1];
}
```

### Move tokens from Ethereum to Hermez Network

Creating a Hermez account and depositing tokens is done simultaneously as an L1 transaction. In this example we are going to deposit 1 ETH tokens into the newly created Hermez accounts.

```dart
import 'package:hermez_sdk/tx.dart' as tx;
import 'package:hermez_sdk/utils.dart';
import 'package:hermez_sdk/hermez_compressed_amount.dart';

...

void moveTokensFromEthereumToHermez() async {
 
    // load  account and ethereum token

    ...

    // set amount to transfer
    final amount = 1.0;
    final amountDeposit = getTokenAmountBigInt(amount, tokenERC20.decimals);
    final compressedDepositAmount =
        HermezCompressedAmount.compressAmount(amountDeposit.toDouble());

    // perform deposit account 1
    String txHash = await tx.deposit(compressedDepositAmount, hermezEthereumAddress, tokenERC20,
        hermezWallet.publicKeyCompressedHex, EXAMPLES_PRIVATE_KEY1);
}
```

### Token Balance

A token balance can be obtained by querying the API and passing the hermezEthereumAddress of the Hermez account.

```dart
void getTokenBalance() async {

    // load  accounts and ethereum token
    
    ...

    // get sender account information
    final infoAccountSender = (await coordinatorApi
            .getAccounts(hermezEthereumAddress, [tokenERC20.id]))
        .accounts[0];

    // get receiver account information
    final infoAccountReceiver = (await coordinatorApi
            .getAccounts(hermezEthereumAddress2, [tokenERC20.id]))
        .accounts[0];
}
```

```json
[
  {
    "accountIndex": "hez:ETH:4253",
    "balance": "1099600000000000000",
    "bjj": "hez:dMfPJlK_UtFqVByhP3FpvykOg5kAU3jMLD7OTx_4gwzO",
    "hezEthereumAddress": "hez:0x74d5531A3400f9b9d63729bA9C0E5172Ab0FD0f6",
    "itemId": 4342,
    "nonce": 1,
    "token": {
      "USD": 1789,
      "decimals": 18,
      "ethereumAddress": "0x0000000000000000000000000000000000000000",
      "ethereumBlockNum": 0,
      "fiatUpdate": "2021-02-28T18:55:17.372008Z",
      "id": 0,
      "itemId": 1,
      "name": "Ether",
      "symbol": "ETH"
    }
  },
  {
    "accountIndex": "hez:ETH:4254",
    "balance": "1097100000000000000",
    "bjj": "hez:HESLP_6Kp_nn5ANmSGiOnhhYvF3wF5Davf7xGi6lwh3U",
    "hezEthereumAddress": "hez:0x12FfCe7D5d6d09564768d0FFC0774218458162d4",
    "itemId": 4343,
    "nonce": 6,
    "token": {
      "USD": 1789,
      "decimals": 18,
      "ethereumAddress": "0x0000000000000000000000000000000000000000",
      "ethereumBlockNum": 0,
      "fiatUpdate": "2021-02-28T18:55:17.372008Z",
      "id": 0,
      "itemId": 1,
      "name": "Ether",
      "symbol": "ETH"
    }
  }
]
```

We can see that the field accountIndex is formed by the token symbol it holds and an index. A Hermez account can only hold one type of token. Account indexes start at 256. Indexes 0-255 are reserved for internal use. Note that the balances do not match with the ammount deposited of 1 ETH because accounts already existed in Hermez Network before the deposit, so we performed a deposit on top instead.

Alternatively, an account query can be filtered using the assigned accountIndex

```dart
 
    ...

    final account1ByIdx = coordinatorApi.getAccount(infoAccountSender.accountIndex);

    final account2ByIdx = coordinatorApi.getAccount(infoAccountReceiver.accountIndex);

```

```json
[
  {
    "accountIndex": "hez:ETH:4253",
    "balance": "1099600000000000000",
    "bjj": "hez:dMfPJlK_UtFqVByhP3FpvykOg5kAU3jMLD7OTx_4gwzO",
    "hezEthereumAddress": "hez:0x74d5531A3400f9b9d63729bA9C0E5172Ab0FD0f6",
    "itemId": 4342,
    "nonce": 1,
    "token": {
      "USD": 1789,
      "decimals": 18,
      "ethereumAddress": "0x0000000000000000000000000000000000000000",
      "ethereumBlockNum": 0,
      "fiatUpdate": "2021-02-28T18:55:17.372008Z",
      "id": 0,
      "itemId": 1,
      "name": "Ether",
      "symbol": "ETH"
    }
  },
  {
    "accountIndex": "hez:ETH:4254",
    "balance": "1097100000000000000",
    "bjj": "hez:HESLP_6Kp_nn5ANmSGiOnhhYvF3wF5Davf7xGi6lwh3U",
    "hezEthereumAddress": "hez:0x12FfCe7D5d6d09564768d0FFC0774218458162d4",
    "itemId": 4343,
    "nonce": 6,
    "token": {
      "USD": 1789,
      "decimals": 18,
      "ethereumAddress": "0x0000000000000000000000000000000000000000",
      "ethereumBlockNum": 0,
      "fiatUpdate": "2021-02-28T18:55:17.372008Z",
      "id": 0,
      "itemId": 1,
      "name": "Ether",
      "symbol": "ETH"
    }
  }
]
```

### Move tokens from Hermez to Ethereum Network

#### Exit

#### Withdraw

#### Force Exit

### Transfers

### Transaction Status

### Create Account Authorization


