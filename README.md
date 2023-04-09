## Aave Credit Delegation

This project is intended to be used as a tool/reference for Aave v2 credit delegation, until [app.aave.com](https://app.aave.com) supports this feature in its UI.

It allows you to validate that a given collateral/loan pair can be used for credit delegation, provides instructions for performing the delegation via direct contract interaction, and provides some tools for simulating the delegation via forking.

### System overview

![credit-delegation](./imgs/credit-delegation.png)

1. Owner deposits collateral in Aave via its LendingPool
2. Owner uses the data provider contract to retrieve the associated DebtToken for the borrower's desired asset to borrow
3. Owner interacts with the DebtToken contract to approve the borrower to take out a given amount of credit on the asset
4. Borrower borrows in Aave via its LendingPool increasing the Owner's `totalDebtETH`
5. Eventually, Borrower repays loan, decreasing the Owner's `totalDebtETH`
6. Eventually, Owner withdraws collateral from Aave

### Resources

##### Mainnet addresses:
* LendingPool: `0x7d2768dE32b0b80b7a3454c06BdAc94A69DDc7A9`
* DataProvider: `0x057835Ad21a177dbdd3090bB1CAE03EaCF78Fc6d`
* dsDAI: `0x778A13D3eeb110A4f7bb6529F99c000119a08E92`
* DAI: `0x6B175474E89094C44Da98b954EedeAC495271d0F`

##### Etherscan links
* LendingPool: https://etherscan.io/address/0x7d2768dE32b0b80b7a3454c06BdAc94A69DDc7A9#readProxyContract
* DataProvider: https://etherscan.io/address/0x057835Ad21a177dbdd3090bB1CAE03EaCF78Fc6d#readContract
* dsDAI: https://etherscan.io/address/0x778A13D3eeb110A4f7bb6529F99c000119a08E92#readProxyContract
* DAI: https://etherscan.io/address/0x6B175474E89094C44Da98b954EedeAC495271d0F

##### Backup oneclickdapp interfaces:
* LendingPool: https://oneclickdapp.com/monica-axiom/
* DataProvider: https://oneclickdapp.com/juice-empty/
* DAI: https://oneclickdapp.com/samba-mars/
* dsDAI: https://oneclickdapp.com/helena-austin/

_NOTE_: Additional interfaces can be created at will by specifying addresses and ABIs at `artifacts/contracts/interfaces/`

##### Aave documentation:

https://docs.aave.com/developers/v/2.0/the-core-protocol/lendingpool

### STEP BY STEP INSTRUCTIONS: Credit delegation on Mainnet via Etherscan

1. Verify the validity of the credit delegation parameters
	* Automated: as described in "Validating a credit delegation pair" using unit tests
	* Manually: In [app.aave.com](https://app.aave.com), make sure that the deposit asset is available for use as collateral, and that the borrow asset is available for stable/variable borrow, and has sufficient liquidity
2. Use [app.aave.com](https://app.aave.com) to deposit the desired collateral as `lender`
3. Use Etherescan for `DataProvider.getReserveTokensAddresses(asset: <address of the token to delegate>)` to identify the associated debt token
4. From the previous point, use `stableDebtTokenAddress` or `variableDebtTokenAddress` depending on your desired interest rate model, then click on the address to be able to interact with the contract via Etherscan
7. Use `DebtToken.approveDelegation(delegatee: <credit delegation beneficiary>, amount: <amount of credit to approve>)`. If the associated DebtToken doesn't have verified sources, or doesn't have its proxy properly set up in Etherscan, build a UI with oneclickdapp using interfaces from `artifacts/contracts/interfaces`
8. Use `DebtToken.borrowAllowance(fromUser: <lender>, toUser: <borrower>)` to verify that `borrow` has been approved for delegated credit
9. Use [app.aave.com](https://app.aave.com) to borrow as `borrower`

### Validating a credit delegation pair using unit tests on mainnet

![unit-tests](./imgs/unit-tests.png)

Clone and install the repo

```
$ git clone clone git@github.com:ajsantander/aave-credit-delegation.git
$ cd aave-credit-delegation
$ npm install
```

Copy `.env.sample` to `.env` and specify your Infura private key, or Ethereum mainnet provider url. This will be used for forking mainnet and running simulations/checks against the fork.

To validate a pair, edit `test/CreditDelegation.test.js` to enter the desired collateral/loan pair, amounts, and interest model type, and then run the tests.

This will start a local fork of mainnet and simulate the credit delegation process with the specified parameters.

Edit `testPairs` in `test/CreditDelegation.test.js` with your desired parameters

Run `npm test`

If any test is skipped or fails, credit delegation may not be available for your desired parameters.

![unit-bad](./imgs/unit-bad.png)

### Simulating credit delegation with a fork using oneclickdapp

When using a fork of mainnet, Etherscan can be used to write to the fork, but not read, since it will always connect to mainnet while reading. Thus, the method described below uses [oneclickdapp.com](https://oneclickdapp.com) to provide a user interface for interacting with the contracts via a fork.

![oneclick](./imgs/oneclick.png)

1. Start a fork of mainnet with `npm run start-fork`
2. Add a "Mainnet (fork)" network to Metamask, with url `http://0.0.0.0:8545` and network id `1`
3. Select and/or set up two test addresses in Metamask, `lender` and `borrower`
4. Use the `LendingPool` interface to call `getUserAccountData` for `lender`, verifying that its `totalCollateralETH` is zero
5. Use the `DAI` interface to approve Aave's `LendingPool` address, on behalf of `lender`
6. Use the `LendingPool` interface to call `deposit` with `lender`
7. Use `LendingPool.getUserAccountData` again to verifiy that `lender` deposited
8. Use `DataProvider.getReserveTokensAddresses` to get the address of the associated debt token, in this case `dsDAI`
9. Use `dsDAI.approveDelegation`
10. Finally, use `LendingPool.borrow` with `borrower`
