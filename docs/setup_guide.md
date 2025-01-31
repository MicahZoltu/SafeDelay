# SafeDelay Setup Guide

This guide shows how to setup a delay-module with a Gnosis Safe on the Rinkeby testnetwork.

## Prerequisites

To start the process you need to create a Safe on the Rinkeby test network (e.g. via https://rinkeby.gnosis-safe.io). This Safe will represent the DAO and hold all the assets (e.g. tokens and collectibles). A Safe transaction is required to setup the delay-module.

For the hardhat tasks to work the environment needs to be properly configured. See the [sample env file](../.env.sample) for more information.

## Setting up the module

The first step is to deploy the module. Every Safe will have their own module. The module is linked to a Safe (called executor in the contract). This cannot be changed after deployment.

### Deploying the module

Hardhat tasks can be used to deploy a delay-module instance. There are two different tasks to deploy the module, the first one is through a normal deployment and passing arguments to the constructor (with the task `setup`), or, deploy the Module through a [Minimal Proxy Factory](https://eips.ethereum.org/EIPS/eip-1167) and save on gas costs (with the task `factorySetup`) - In rinkeby the address of the Proxy Factory is: `0x569F2e024D0aD6bBfBd8135097DFa7D0641Ae79b` and the Master Copy of the Delay Module: `0xAe78DF7a4184917dA49f7fCA6139f924A79D0488`.

These setup tasks requires the following parameters:

- `executor` - the address of the executor.
- `owner` - the address of the owner
- `cooldown` - optional, by default is set to 24 hours
- `expiration` - optional, by default is set to 7 days

For more information run `yarn hardhat setup --help` or `yarn hardhat factorySetup --help`.

An example for this on Rinkeby would be:
`yarn hardhat --network rinkeby setup --owner <owner_address> --executor <executor_address>`

or

`yarn hardhat --network rinkeby factorySetup --factory <factory_address> --mastercopy <mastercopy_address> --owner <owner_address> --executor <executor_address>`

This should return the address of the deployed delay-module. For this guide we assume this to be `0x4242424242424242424242424242424242424242`

Once the module is deployed you should verify the source code (Note: If you used the factory deployment the contract should be already verified). If you use a network that is Etherscan compatible and you configure the `ETHERSCAN_API_KEY` in your environment you can use the provided hardhat task to do this.

An example for this on Rinkeby would be:
`yarn hardhat --network rinkeby verifyEtherscan --module 0x4242424242424242424242424242424242424242 --owner <owner_address> --executor <executor_address>`

### Enabling the module

To allow the delay-module to actually execute transaction it is required to enable it on the Safe that it is connected to. For this it is possible to use the Transaction Builder on https://rinkeby.gnosis-safe.io. For this you can follow our tutorial on [adding a module](https://help.gnosis-safe.io/en/articles/4934427-add-a-module).

### Enabling modules on the delay-module

The delay-module implements the same interface as the Safe for enabling and disabling modules, along with enqueueing transactions.

Before an address can enqueue transactions, it will need to be added as a module to the delay-module. To enable an address as a module to the delay-module, follow the [adding a module](https://help.gnosis-safe.io/en/articles/4934427-add-a-module) guide, replacing the Safe address with the delay-module address.

## Monitoring your module

For the delay module to be effective, it is important to know which items are in queue. To make sure that all the involved stakeholders can react in a timely manner, the events emitted by the delay-module contract should be monitored. Each time a new transaction is added the contract will emit a `TransactionAdded` event with the following parameters:
```
event TransactionAdded(
  uint indexed queueNonce, // the transactions place in the queue
  bytes32 indexed txHash, // the hash of the transaction
  address to, // the to address of the transaction
  uint256 value, // the ETH value of the transaction, in wei
  bytes data, // the encoded transaction data
  Enum.Operation operation // (0) call or (1) delegate call
);
```

There are different services available for this such as the [OpenZepplin Defender Sentinel](https://docs.openzeppelin.com/defender/sentinel).
