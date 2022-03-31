<h1> From Gravity Contract Compilation to Deployment </h1> 
<p> This document describes every step involved in compiling a contract and then deploying it.
This document is meant to be complete and does not skip any details for complete understanding. </p> 

<h2> Solidity Compilation: </h2> 


1) run npm command typechain </br>
This tells node package manager to look at config.json located at `Gravity-Bridge/solidity/package.json`
2) The typechain command is described in package.json </br> 
```
  "scripts": {
    "evm": "npx hardhat node",
    "test": "npx hardhat test --network localhost",
    "typechain": "npx hardhat typechain", //typechain command defined here
    "compile-deployer": ...
    "evm_fork": ...
  },
```
<p> 
npx: is the command for node package execute </br> 
hardhat: this is an ethereum development tool we will be using </br> 
typechain: name from hardhat.config.ts to be used, also name of a hardhat library </br> 
</p> 

3) We can see “typechain” command described in in hardhat.config.ts at `Gravity-Bridge/solidity/hardhat.config.ts` </br> 

```
  typechain: {
    outDir: "typechain",
    target: "ethers-v5",
    runOnCompile: true
  },
  ```
4) Also in this config we are able to specify the desired Solidity compiler: </br> 
 ```
module.exports = {
  // This is a sample solc configuration that specifies which version of solc to use
  solidity: {
    version: "0.8.10",
    settings: {
      optimizer: {
        enabled: true
      }
    }
enabled: true
```
  5) Hardhat TypeChain command is used to generate two types of files: </br> 
   <ul> 
     <li>  .d.ts files located at Gravity-Bridge/solidity/typechain </br>
       - this is the additional functionality that we get from hardhat-typechain </l1> 
  <li> Contract artifacts located at Gravity-Bridge/solidity/artifacts/contracts </br>
      - this is the original functionality when compiling solidity contracts </li> 
  </ul> 

  6) Understanding sources files for hardhat  </br> 
From hardhat docs (<a> https://hardhat.org/config/#path-configuration </a>): </br> 
```
root: The root of the Hardhat project. This path is resolved from hardhat.config.js's directory. Default value: the directory containing the config file.
sources: The directory where your contract are stored. This path is resolved from the project's root. Default value: './contracts'.
```
In our case the root directory is `Gravity-Bridge/solidity` therefore the sources directory is `/gravity/Gravity-Bridge/solidity/contracts/` 

  7) For those familiar with Solidity compilation using solc, note that hardhat-typechain overrides the compile command with TypeChain </br >
From hardhat typechain docs (<a> https://github.com/rhlsthrm/hardhat-typechain/blob/master/README.md#tasks </a>): <br> 

```
  This plugin overrides the compile task and automatically generates new Typechain artifacts on each compilation.
```

  8) Understanding how hardhat TypeChain overrides solidity compiler command: </br> 
If look at the source code at `Gravity-Bridge/solidity/node_modules/hardhat-typechain/dist/src/index.js` <br> 
(GitHub source: <a>https://github.com/rhlsthrm/hardhat-typechain/blob/master/src/index.ts#L21</a>) </br> 
We see this code:
```
config_1.task(task_names_1.TASK_COMPILE, "Compiles the entire project, building all artifacts")
    .addFlag("noTypechain", "Skip Typechain compilation")
    .setAction(async ({ global, noTypechain }, { config }, runSuper) => {
    if (global) {
        return;
    }
    await runSuper();
    ...
```

It is in the `await runSuper();` that the original solidity compile is invoked. </br>
runSuper() is a special command made for Hardhat, not a general javascript command. 

9) Understanding runSuper() in Hardhat </br> 
If we look at the documentation at <a> https://hardhat.org/guides/create-task.html#the-runsuper-function </a> we see:

```
runSuper is a function available to override a task's actions. It can be received as the third argument of the 
task or used directly from the global object.
```

runSuper() is available to run on any of the builtin commands for Hardhat at:
<a> https://github.com/nomiclabs/hardhat/tree/master/packages/hardhat-core/src/builtin-tasks </a> 

If we look at <a> https://github.com/nomiclabs/hardhat/blob/master/packages/hardhat-core/src/builtin-tasks/compile.ts#L1388 </a> 

We see the original Solidity compile function we had before adding TypeChain:

```
task(TASK_COMPILE, "Compiles the entire project, building all artifacts")
    ...
    const compilationTasks: string[] = await run(
      TASK_COMPILE_GET_COMPILATION_TASKS
    );

    for (const compilationTask of compilationTasks) {
      await run(compilationTask, compilationArgs);
    }
```

10) Compilation output </br> 
As mentioned before, once compilation is complete we will have two sets of files:
<ul> 
  <li> .d.ts files located at Gravity-Bridge/solidity/typechain  </li> 
  <li> contract artifacts located at Gravity-Bridge/solidity/artifacts/contracts </li> 
 </ul> 

<h2> Gravity.sol deployment: </h2> 

1) Running contract-deployer.sh </br> 
   contract-deployer.sh is located at `/Gravity-Bridge/solidity/scripts/contract-deployer.sh`

2) Specifying location of Gravity.json artifact </br> 
  In contract-deployer.sh we can see where Gravity.json artifact is referenced: </br> 
  ```
  npx ts-node \
contract-deployer.ts \
--cosmos-node=...
--eth-node=...
--eth-privkey=...
--contract=/gravity/solidity/artifacts/contracts/Gravity.sol/Gravity.json \. #CONTRACT REFERENCE HERE
--test-mode=true
 ```

3) Running contract-deployer.ts </br> 
  The aforementioned .sh command will run contract-deployer.ts located at `Gravity-Bridge/solidity/contract-deployer.ts`

4) Loading the gravity.json artifact </br> 

  If we look at line# 173 (<a>https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/solidity/contract-deployer.ts#L173</a>)

```
  console.log("Starting Gravity contract deploy");
  const { abi, bytecode } = getContractArtifacts(args["contract"]);
  const factory = new ethers.ContractFactory(abi, bytecode, wallet);
```

<p> The argument “contract” was provided as —contract in the contract-deployer.sh which is the location of our Gravity.json artifact from before.  </p> 

The ethers library is used to read the json contract and load into const named factory.

5) Get valset and check powers: </br> 
This line gets the latest valset:

```
const latestValset = await getLatestValset();
```

<p> For each member in the valset, the voting power is checked and if less than 66% of the current voting power has unset Ethereum Addresses then the deploy will fail </p> 

6) Use ethers library to deploy contract byte code to running chain: (<a> https://github.com/Gravity-Bridge/Gravity-Bridge/blob/main/solidity/contract-deployer.ts#L205 </a>) </br> 

```
  const gravity = (await factory.deploy(
    // todo generate this randomly at deployment time that way we can avoid
    // anything but intentional conflicts
    gravityId,
    eth_addresses,
    powers,
    overrides
  )) as Gravity;
```
 In the ethers documentation (<a> https://docs.ethers.io/v5/api/contract/contract-factory/#ContractFactory-deploy </a>) </br>

```
 Uses the signer to deploy the Contract with args passed into the constructor and returns a Contract which is attached to the address where this contract will be deployed once the transaction is mined.
```

7) Contract will now be deployed
