# Tutorial Getting Started With Contracts

We rephrase an [article](#https://github.com/EOSIO/eos/wiki/Tutorial-Getting-Started-With-Contracts) from EOSIO wiki.

## Getting Started with Contracts

The purpose of this tutorial is to demonstrate primary experiments with smart contracts. This tutorial assumes that you have installed both [*EOSIO*](#https://github.com/EOSIO/eos) and [*Tokenika Logos*](#https://github.com/tokenika/logos). If so, you have installed *python3*, as well.

To begin with, open a bash terminal in the `teos_python` directory of the *Logos* repository. Start `python3`:
```
import teos
teos.set_verbose(True)
```

## Starting a Private Blockchain

Launch an object of `teos.DaemonClear()` that starts the local EOSIO node, beginning a new blockchain:
```
teos.DaemonClear()
#       nodeos exe file: /mnt/e/Workspaces/EOS/eos/build/programs/nodeos/nodeos
#    genesis state file: /mnt/e/Workspaces/EOS/eos/build/programs/daemon/data-dir/genesis.json
#        server address: 127.0.0.1:8888
#      config directory: /mnt/e/Workspaces/EOS/eos/build/programs/daemon/data-dir
#      wallet directory: /mnt/e/Workspaces/EOS/eos/build/programs/daemon/data-dir/wallet
#     head block number: 2
#       head block time: 2018-04-10T17:20:54
```
Now, a newly started terminal has the local node running.

## Creating a Wallet

A wallet is a repository of private keys necessary to authorize actions on the blockchain. These keys are stored on disk encrypted using a password generated by the EOSIO node. 

Launch an object `teos.Wallet` that implements a local wallet:
```
wallet = teos.Wallet()
#              password: PW5KLhcjMHeLxiyyrFU8EWncXUHHLNWgw7uxmvKMwMC19jMHR6zWk
#  You need to save this password to be able to lock/unlock the wallet!
```
The wallet name argument is not set here: it defaults to 'default'.

When doing with a real value, this password should be stored in a secure password manager, however, for the purpose of development environment, the password is kept in the wallet object. Therefore, the following instructions make sense:
```
wallet.lock()
wallet.list()
#                wallet: default
wallet.unlock()
>>> wallet.list()          # The starlet marks unlocked:
#                wallet: default *
```

## The Eosio Account and the Bios Contract

You have to owe an valid EOSIO account to be authorized to interact with EOSIO. For tests, you can use the ‘eosio’ account:

```
account_eosio = teos.EosioAccount()
account_eosio.key_private
'5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3'
```
You have to have the private key to your property in your wallet. Perhaps, it is there already:
```
wallet.keys()
#  {
#      "wallet keys": [
#          [
#              "EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV",
#              "5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3"
#          ]
#      ]
#  }
```
Indeed, your wallet contains the key to `account_eosio`. I it was not there, you could import it. Now, the following attempt is rejected:
```
wallet.import_key(account_eosio)
ERROR! status code is 500  eosd response is Content-Length: 234  Content-type:
application/json  Server: WebSocket++/0.7.0    {"code":500,"message":"Internal
Service Error","error":{"code":10,"name":"assert_exception","what":"Assert
Exception","details":[{"message":"!\"Key already in wallet\":
","file":"wallet.cpp","line_number":150,"method":"import_key"}]}}
```

Now that you have a wallet with the key for the eosio account loaded, you can set a *default system contract*. For the purposes of development, the default `eosio.bios` contract can be used. This contract enables you to have direct control over the resource allocation of other accounts and to access other privileged API calls. In a public blockchain, this contract will manage the staking and unstaking of tokens to reserve bandwidth for CPU and network activity, and memory for contracts.

The eosio.bios contract can be found in the contracts/eosio.bios folder of your EOSIO source code. The Tokenika teos python is configured to find the proper path.
```
contract_eosio_bios = teos.SetContract(
  account_eosio, "eosio.bios", permission=account_eosio)
#        transaction id: fd45de61bd9f05a570df44faf5fd4db182f6a4f4d4da3b0302735f23645f63d2
``` 
The transaction is authorized and signed by `account_eosio`. 

As the set contract command call has produced the transaction id, the default contract is operational. The result of this transaction setting is that it is generated a transaction with two actions, eosio::setcode and eosio::setabi.

The contract is set to the node in two parts: `code` and `abi` The code defines how the contract runs and the abi describes how to convert between binary and json representations of the arguments. While an abi is technically optional, all of the EOSIO tooling depends upon it for ease of use.

## Creating Accounts

Now that you have set the basic system contract, you can start to create our own accounts. You will create two accounts, user and tester, and you will need to associate a key with each account. In this example, the same key will be used for both accounts.

To do this you first generate a key for the accounts and import it to the wallet.
```
key_accounts = teos.CreateKey("key_accounts")
#              key name: key_accounts
#           private key: 5JG3gEsowhnQXEYD1z4Vmh2iJMnNrb8oSg9TBVbLFjV7dSuDTrW
#            public key: EOS4vA5JV7zKTJCa83LHrvzbHv4z6G5g6GLXMgSB3JZQu5rDpRueS

wallet.import_key(key_accounts)
```
### Create Two User Accounts

Next you will create two accounts, user and tester, using the key you created and imported above.
```
account_user = teos.Account(account_eosio, "user", key_accounts, key_accounts)
#        transaction id: f94c26662da514fac7027270531e023a6fc8cd4dcd739dc67e...

account_tester = teos.Account(
   account_eosio, "tester", key_accounts, key_accounts)
   #        transaction id: 6eb3e7a64ac2265de855f9c8b0c5677848e8451b4d6dda8...
```
NOTE: The create account subcommand requires two keys, one for the OwnerKey (which in a production environment should be kept highly secure) and one for the ActiveKey. In this tutorial example, the same key is used for both.

You can query all accounts that are controlled `key_accounts` key:
```
teos.GetAccounts(key_accounts)
#  {
#      "account_names": [
#          "tester",
#          "user"
#      ]
#  }
```

## Eosio.token, Exchange, and Eosio.msig Contracts

At this stage the blockchain doesn't do much, so let's deploy the eosio.token contract. This contract enables the creation of many different tokens all running on the same contract but potentially managed by different users.

Before we can deploy the token contract we must create an account to deploy it to.
```
account_eosio_token = teos.Account(
  account_eosio, "eosio.token", key_accounts, key_accounts)
#        transaction id: 64dcf4a8f522daea9699b48bcacee797273f4ca42ee6cb9de0...
```
Then we can deploy the contract which can be found in ${EOSIO_SOURCE}/build/contracts/eosio.token:
```
contract_eosio_token = teos.Contract(
  account_eosio_token, "eosio.token", permission=account_eosio_token)
#        transaction id: 286fecdb671fd8e0128acb4f28a1a10cc2a8e195047748ef54...
```

## Create the EOS Token

You can view the interface to eosio.token as defined by contracts/eosio.token/eosio.token.hpp:
```
   void create( account_name issuer,
                asset        maximum_supply,
                uint8_t      can_freeze,
                uint8_t      can_recall,
                uint8_t      can_whitelist );


   void issue( account_name to, asset quantity, string memo );

   void transfer( account_name from,
                  account_name to,
                  asset        quantity,
                  string       memo );
```
To create a new token we must call the create(...) action with the proper arguments. This command will use the symbol of the maximum supply to uniquely identify this token from other tokens. The issuer will be the one with authority to call issue and or perform other actions such as freezing, recalling, and whitelisting of owners.
```
contract_eosio_token.action(
  "create",
  '{"issuer":"eosio", "maximum_supply":"1000000000.0000 EOS", \
    "can_freeze":0, "can_recall":0, "can_whitelist":0}')
#        transaction id: 42df2cca11971b5af6478156d33e2cd8f5869de6a80090efc6...
```
You can read the above: 

Let contract `contract_eosio_token` use its `create` method with arguments as defined with the second argument. Let this action be explicitly (default setting) signed by its creator, that is `account_eosio_token`.

This command created a new token EOS with a pecision of 4 decimials and a maximum supply of 1000000000.0000 EOS.

### Issue Tokens to Account "User"

Now that we have created the token, the issuer can issue new tokens to the account user we created earlier.
```
contract_eosio_token.action(
  "issue",
  '{"to":"user","quantity":"100.0000 EOS","memo":"memo"}',
  permission=account_eosio)
#        transaction id: 015a870a42ffd3d9e86776cc36c06b28d9d4246ad585d8566f...
```
From the original [tutorial](#https://github.com/EOSIO/eos/wiki/Tutorial-eosio-token-Contract#issue-tokens-to-account-user),
we know that the above action takes, in fact, 3 transactions: one issue and three transfers.

They say: *Technically, the eosio.token contract could have skipped the inline transfer and opted to just modify the balances directly. However, in this case, the eosio.token contract is following our token convention that requires that all account balances be derivable by the sum of the transfer actions that reference them. It also requires that the sender and receiver of funds be notified so they can automate handling deposits and withdrawals.*

We do not guess that this knowledge could be helpful to you, who orderers this action. Anyhow, you can see the actual transaction that was broadcast:

```
contract_eosio_token.action(
  "issue",
  '{"to":"user","quantity":"100.0000 EOS","memo":"memo"}',
  permission=account_eosio,
  dont_broadcast=1)                     # set dont_broadcast=1

#        transaction id: 0195bef180de1bbc298e2bd1a086288cfbfbc3f8d5e6e3f0604e21a84687ac05

{'processed': {'_profiling_us': '4659',
               '_setup_profiling_us': '1633',
               'action_traces': [{'_profiling_us': '2510',
                                  'act': {'account': 'eosio.token',
                                          'authorization': [{'actor': 'eosio',
                                                             'permission': 'active'}],
                                          'data': {'memo': 'memo',
                                                   'quantity': '100.0000 EOS',
                                                   'to': 'user'},
                                          'hex_data': '00000000007015d640420f000000000004454f5300000000046d656d6f',
                                          'name': 'issue'},
                                  'console': 'issue',
                                  'context_free': 'false',
                                  'cpu_usage': '13568',
                                  'data_access': [{'code': 'eosio.token',
                                                   'scope': '........ehbo5',
                                                   'sequence': '2',
                                                   'type': 'write'},
                                                  {'code': 'eosio.token',
                                                   'scope': 'eosio',
                                                   'sequence': '2',
                                                   'type': 'write'}],
                                  'receiver': 'eosio.token'},
                                 {'_profiling_us': '1290',
                                  'act': {'account': 'eosio.token',
                                          'authorization': [{'actor': 'eosio',
                                                             'permission': 'active'}],
                                          'data': {'from': 'eosio',
                                                   'memo': 'memo',
                                                   'quantity': '100.0000 EOS',
                                                   'to': 'user'},
                                          'hex_data': '0000000000ea305500000000007015d640420f000000000004454f5300000000046d656d6f',
                                          'name': 'transfer'},
                                  'console': 'transfer',
                                  'context_free': 'false',
                                  'cpu_usage': '6865',
                                  'data_access': [{'code': 'eosio.token',
                                                   'scope': 'eosio',
                                                   'sequence': '3',
                                                   'type': 'write'},
                                                  {'code': 'eosio.token',
                                                   'scope': 'user',
                                                   'sequence': '1',
                                                   'type': 'write'},
                                                  {'code': 'eosio.token',
                                                   'scope': '........ehbo5',
                                                   'sequence': '3',
                                                   'type': 'read'}],
                                  'receiver': 'eosio.token'},
                                 {'_profiling_us': '37',
                                  'act': {'account': 'eosio.token',
                                          'authorization': [{'actor': 'eosio',
                                                             'permission': 'active'}],
                                          'data': {'from': 'eosio',
                                                   'memo': 'memo',
                                                   'quantity': '100.0000 EOS',
                                                   'to': 'user'},
                                          'hex_data': '0000000000ea305500000000007015d640420f000000000004454f5300000000046d656d6f',
                                          'name': 'transfer'},
                                  'console': '',
                                  'context_free': 'false',
                                  'cpu_usage': '21',
                                  'data_access': '',
                                  'receiver': 'eosio'},
                                 {'_profiling_us': '8',
                                  'act': {'account': 'eosio.token',
                                          'authorization': [{'actor': 'eosio',
                                                             'permission': 'active'}],
                                          'data': {'from': 'eosio',
                                                   'memo': 'memo',
                                                   'quantity': '100.0000 EOS',
                                                   'to': 'user'},
                                          'hex_data': '0000000000ea305500000000007015d640420f000000000004454f5300000000046d656d6f',
                                          'name': 'transfer'},
                                  'console': '',
                                  'context_free': 'false',
                                  'cpu_usage': '0',
                                  'data_access': '',
                                  'receiver': 'user'}],
               'cpu_usage': '125952',
               'cycle_index': '1',
               'deferred_transaction_requests': '',
               'id': '0195bef180de1bbc298e2bd1a086288cfbfbc3f8d5e6e3f0604e21a84687ac05',
               'kcpu_usage': '123',
               'net_usage': '256',
               'net_usage_words': '32',
               'packed_trx_digest': '218563a2f4a414c5b69b191e00b6e5b1cf5945f7849a20b519106b096b7af114',
               'read_locks': '',
               'region_id': '0',
               'shard_index': '0',
               'status': 'executed',
               'write_locks': [{'account': 'eosio.token',
                                'scope': '........ehbo5'},
                               {'account': 'eosio.token', 'scope': 'eosio'},
                               {'account': 'eosio.token', 'scope': 'user'}]},
 'transaction_id': '0195bef180de1bbc298e2bd1a086288cfbfbc3f8d5e6e3f0604e21a84687ac05'}
```

### Transfer Tokens to Account "Tester"

```
contract_eosio_token.action(
  "transfer", 
  '{"from":"user","to":"tester","quantity":"25.0000 EOS","memo":"m"}',
  permission="user")
#        transaction id: ba296986977ff784b4c0bc24a57df6e341828e578760a61c87...
```

## Deploy Exchange Contract

Similar to the examples shown above, we can deploy the exchange contract. The exchange contract provides capabilities to create and trade currency. It is assumed this is being run from the root of the EOSIO source.
```
account_exchange = teos.Account(
  account_eosio, "exchange", key_accounts, key_accounts)
#        transaction id: 2cef0a51d770218119a0288b9e7034bf77ac5f9c3600f203c9...

contract_exchange = teos.Contract(
  account_eosio_token, "exchange", permission=account_eosio_token)
#        transaction id: 76c280a7e286d1013f07069b74df5f1bcc904441c6e58c025a...
)
```
## Deploy Eosio.msig Contract

The eosio.msig contract allows multiple parties to sign a single transaction asynchronously. EOSIO has multi-signature (multisig) support at a base level, but it requires a synchronous side channel where data is ferried around and signed. Eosio.msig is a more user friendly way of asynchronously proposing, approving and eventually publishing a transaction with multiple parties' consent.

The following steps can be used to deploy the eosio.msig contract.
```
account_eosio_msig = teos.Account(
  account_eosio, "eosio.msig", key_accounts, key_accounts)
#        transaction id: 75ba5884512b078ccb06d10818b4a1ea1894dc466e0f5d3630...

contract_eosio_msig = teos.Contract(
  account_eosio_token, "eosio.msig", permission=account_eosio_token)
#        transaction id: f069fe366ff81478c030eaec7e5b8c9887093a6b22a2f634d9...
```

## Summary

```
daemon = teos.DaemonClear()
wallet = teos.Wallet()
wallet.lock()
wallet.list()
wallet.unlock()
wallet.list()
account_eosio = teos.EosioAccount()
account_eosio.key_private
wallet.keys()
wallet.import_key(account_eosio)
contract_eosio_bios = teos.SetContract(
  account_eosio, "eosio.bios", permission=account_eosio)
key_accounts = teos.CreateKey("key_accounts")
wallet.import_key(key_accounts)
account_user = teos.Account(
   account_eosio, "user", key_accounts, key_accounts)
account_tester = teos.Account(
   account_eosio, "tester", key_accounts, key_accounts) 
teos.GetInfo()
teos.GetAccounts(key_accounts)
account_eosio_token = teos.Account(
  account_eosio, "eosio.token", key_accounts, key_accounts)
contract_eosio_token = teos.Contract(
  account_eosio_token, "eosio.token", permission=account_eosio_token)
contract_eosio_token.action(
  "create",
  '{"issuer":"eosio", "maximum_supply":"1000000000.0000 EOS", \
    "can_freeze":0, "can_recall":0, "can_whitelist":0}') 
contract_eosio_token.action(
  "issue",
  '{"to":"user","quantity":"100.0000 EOS","memo":"memo"}',
  permission=account_eosio)
contract_eosio_token.action(
  "issue",
  '{"to":"user","quantity":"100.0000 EOS","memo":"memo"}',
  permission=account_eosio,
  dont_broadcast=1)
account_exchange = teos.Account(
  account_eosio, "exchange", key_accounts, key_accounts)
contract_exchange = teos.Contract(
  account_eosio_token, "exchange", permission=account_eosio_token)
account_eosio_msig = teos.Account(
  account_eosio, "eosio.msig", key_accounts, key_accounts)
contract_eosio_msig = teos.Contract(
  account_eosio_token, "eosio.msig", permission=account_eosio_token)

```