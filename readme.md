## Parcours français

Ce dépôt est accompagné d’un parcours documentaire en français consacré à Beefy Vaults. Voir [docs/fr/](docs/fr/).

# Beefy Contracts
Official repo for strategies and vaults from Beefy. Community strategists can contribute here to grow the ecosystem.


## Vault Deployment Process
### 1. Select a farm
The first step to have a vault deployed on Beefy is to select a farm to deploy a vault around. At the moment the rewards for a strategist are:
 - 0.5% of all rewards earned by a vault they deployed.


This means that you want to select a farm with:
1. High APR
2. High expected TVL
3. Long farm life


First time strategists must deploy contracts for farms on existing platforms on Beefy first. New platforms must undergo an audit by Beefy dev team before development can begin.


### 2. Prepare the smart contracts
If you decided to do a simple LP vault, or a single asset vault, the most likely thing is that there is a working template that you can use. Most farms work under a version of the [Masterchef](https://bscscan.com/address/0xe70E9185F5ea7Ba3C5d63705784D8563017f2E57#code) contract (like Goose Finance), or [Reward Pool](https://arbiscan.io/address/0x48f4634c8383af01bf71aefbc125eb582eb3c74d#code) contract (like Beefy Reward Pool).


### 3. Test the contracts
If you're doing something completely custom you should add automated tests to facilitate review and diminish risks. If it's a copy/paste from another strategy you can get by with manual testing for now as everything has been battle tested tested quite a bit.


For extra help in debugging a deployed vault during development, you can use the [ProdVaultTest.t.sol](./forge/test/ProdVaultTest.t.sol), which is written using the `forge` framework. Run `yarn installForge` to install if you don't have `forge` installed.


To prep to run the test suite, input the correct vault address, vaultOwner and stratOwner for the chain your testing in `ProdVaultTest.t.sol`, and modify the `yarn forgeTest:vault` script in package.json to pass in the correct RPC url of the chain your vault is on. Then run `yarn forgeTest:vault` to execute the test run. You can use `console.log` within the tests in `ProdVaultTest.t.sol` to output to the console.


### 4. Deploy the smart contracts
Once you are confident that everything works as expected you can do the official deploy of the vault + strategy contracts. There are [some scripts](https://github.com/beefyfinance/beefy-contracts/blob/master/scripts/) to help make deploying easier. 


Make sure the strategy is verified in the scanner. A fool-proof way to verify is to flatten the strategy file using the `yarn flat-hardhat` command and removing the excess licenses from the flattened file. Verify the strategy contract using the flattened file as the source code, solidity version is typically 0.6.12 and is optimized to 200 runs. Constructor arguments can be found from the end of the input data in the contract creation transaction; they are padded out with a large number of 0s (include the 0s).


### 5.  Update the app
The only file you really need to touch on the app is the respective json file located in the [vault](https://github.com/beefyfinance/beefy-v2/tree/main/src/config/vault) folder. This is the config file with all the live pools.  Just copy one of the other pools as template, paste it at the top (below the BIFI Maxi and boosted vaults) and fill it out with your data. `earnedTokenAddress`and `earnedContractAddress` should both be the address of the vault contract. These addresses must be checksummed. Use the `getPoolCreationTimestamp.js` script to get creation dates. You will also need to update the addressBook to the current version in package.json in order for Zap to work if the tokens are new to the address book. 


### 6. Test the vault


Run `yarn start` on the local app terminal and test the vault as if you were a user on the `localhost` page.


**Manual Testing Is Required for All Live Vaults**


0. Give vault approval to spend your want tokens. 
1. Deposit a small amount to test deposit functionality.
2. Withdraw, to test withdraw functionality.
3. Deposit a larger amount wait 30 seconds to a minute and harvest. Check harvest transaction to make sure things are going to the right places.
4. Panic the vault. Funds should be in the strategy.
5. Withdraw 50%.
6. Try to deposit, once you recieve the error message pop up in metamask you can stop. No need to send the transaction through.
7. Unpause.
8. Deposit the withdrawn amount.
9. Harvest again.
10. Switch harvest-on-deposit to `true` for low-cost chains (Polygon, Fantom, Harmony, Celo, Cronos, Moonriver, Moonbeam, Fuse, Syscoin, Emerald).
11. Check that `callReward` is not 0, if needed set `pendingRewardsFunctionName` to the relevant function name from the masterchef.
12. Transfer ownership of the vault and strategy contracts to the owner addresses for the respective chains found in the [address book](https://github.com/beefyfinance/beefy-api/tree/master/packages/address-book).
13. Leave some funds in the vault until users have deposited after going live, empty vaults will fail validation checks.
14. Run `yarn validate` to ensure that the validation checks will succeed when opening a pull request.


This is required so that maintainers can review everything before the vault is actually live on the app and manage it after its live.


### 7. Update the API
#### Existing platform
If you're deploying a vault for a platform where we already have live vaults, you will probably only need to add some data to the respective config file in the [data](https://github.com/beefyfinance/beefy-api/tree/master/src/data) folder. For example if you're doing a new Pancakeswap LP vault, you only need to add the relevant data at [cakeLpPools.json](https://github.com/beefyfinance/beefy-api/blob/master/src/data/cakeLpPools.json)
