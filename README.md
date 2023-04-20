# forge-deploy

A cli and associated contracts to keep track of deployments by name and reuse them in solidity.

It tries to keep compatibility with [hardhat-deploy](https://github.com/wighawag/hardhat-deploy) as far as possible (work in progress).

## Features
- generate type-safe deployment function for forge contracts. no need to pass in string of text and hope the abi encoded args are in the correct order.
- save deployments in json file (based on hardhat-deploy schema)

## How to use

0. have a forge project and cd into it

```
cd my-project
forge init # if needed
```

1. add the forge package

```
forge install wighawag/forge-deploy
```


2. install the cli tool locally as the tool is likely to evolve rapidly
```
cargo install --locked 0.0.2 --root . forge-deplpoy
```

This will install version 0.0.2 in the bin folder,

You can then execute it via 

```
./bin/forge-deploy <command> ...
```

you can also compile it directly from the `lib/forge-deploy/` folder.


3.
add a deploy script, see below example

Note that to provide type-safety `forge-deploy` provide a `gen-deployer` command to generate type-safe deploy function for all contracts

```
./bin/forge-deploy gen-deployer
```

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

import "forge-deploy/DeployScript.sol";
import "generated/deployer/DeployerFunctions.g.sol";

contract DeployGreetingsRegistry is DeployScript {
    using DeployerFunctions for Deployer;
    
    // you can also use the run function and this way pass params to your script
    // if so you need to ensure to return with the new deployments via:
    // `return _deployer.newDeployments();`
    // example:
    // function run() override public returns (DeployerDeployment[] memory newDeployments) {
    //  // .... _deployer.deploy...
    //  return _deployer.newDeployments();
    // }
    // this is how forge-deploy keep track of deployment names 
    // and how the forge-deploy sync command can generate the deployments files
    //
        
    function deploy() override internal {
    
        // you can check for existence
        if (_deployer.has("MyRegistry")){
            console.log("MyRegistry already deployed");
            console.log(_deployer.getAddress("MyRegistry"));
        } else {
            console.log("No MyRegistry deployed yet");
        }

        // we can deploy a new contract and name it
        _deployer.deploy_GreetingsRegistry(
            "MyRegistry",
            vm.toString(address(existing)),
            DeployOptions({deterministic: 0, proxyOnTag: "", proxyOwner: address(0)})
        );

        // if you need to override an existing contract
        // you first ignore it
        _deployer.ignoreDeployment("MyRegistry");
        // and deploy a new one
        // the deployment file will only be overriden after broadcast
        _deployer.deploy_GreetingsRegistry(
            "MyRegistry",
            vm.toString(address(existing)),
            DeployOptions({deterministic: 0, proxyOnTag: "", proxyOwner: address(0)})
        );

        // also support a basic EIP173 Proxy similar to hardhat-deploy (not fully featured)
        _deployer.deploy_GreetingsRegistry(
            "ProxiedRegistry",
            vm.toString(address(existing)),
            DeployOptions({deterministic: 0, proxyOnTag: "local", proxyOwner: vm.envAddress("DEPLOYER")})
        );

        _deployer.deploy_GreetingsRegistry(
            "ProxiedRegistry",
            vm.toString(address(existing)),
            DeployOptions({deterministic: 0, proxyOnTag: "local", proxyOwner: vm.envAddress("DEPLOYER")})
        );

        _deployer.deploy_GreetingsRegistry2(
            "ProxiedRegistry",
            vm.toString(address(existing)),
            DeployOptions({deterministic: 0, proxyOnTag: "local", proxyOwner: vm.envAddress("DEPLOYER")})
        );
    }
}
```

5. You can now execute the script via forge script

Note that you need to execute `./bin/forge-deploy sync` directly afterward

For example:

```
forge script script/DeployGreetingsRegistry.s.sol --rpc-url $RPC_URL --broadcast --private-key $DEPLOYER_PRIVATE_KEY -v && forge-deploy sync
```

6. If you use [just](https://just.systems/), see example in [examples/basic](examples/basic) with its own [justfile](examples/basic/justfile)