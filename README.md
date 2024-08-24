
# MODULE 3 PROJECT

This program is a Module 3 Project program that demonstrates the basic syntax of the Circom programming language and Typescript. It is a hardhat-circom Project to generate zero-knowledge circuits, proofs, and solidity verifiers

## Description

This program is a simple code written in Circom, a programming language used for making Circuits .The purpose of this project is to :

1. Write a correct circuit.circom implementation
2. Compile the circuit to generate circuit intermediaries
3. Generate a proof using inputs A=0 B=1
4. Deploy a solidity verifier to Sepolia Testnet
5. Call the verifyProof() method on the verifier contract and assert output is true

This program serves as a simple and straightforward introduction to Circom programming, and can be used as a stepping stone for more complex projects in the future.

# Getting Started

## Quick Start

Compile the CustomCircuit() circuit and verify it against a smart contract verifier

```
pragma circom 2.0.0;

/*The goal of this Custom circuit template is to prove that we know the inputs A (0) & B (1) that yield a 0 output.*/ 

template AND() {
    signal input a;
    signal input b;
    signal output out;

    out <== a*b;
}

template NOT() {
    signal input in;
    signal output out;

    out <== 1 + in - 2*in;
}

template OR() {
    signal input a;
    signal input b;
    signal output out;

    out <== a + b - a*b;
}

template CustomCircuit () {  
    //signal inputs
    signal input a;
    signal input b;

    //signals from gates
    signal x;
    signal y;

    //final signal output
    signal output q;

    //component gates used to create custom circuit
    component andGate = AND();
    component notGate = NOT();
    component orGate = OR();

    //circuit logic
    //logic of AND gate
    andGate.a <== a;
    andGate.b <== b;
    x <== andGate.out;

    //logic of NOT gate
    notGate.in <== b;
    y <== notGate.out;

    //logic of OR gate
    orGate.a <== x;
    orGate.b <== y;
    q <== orGate.out;
}

component main = CustomCircuit();
```

## To run the project you can use this steps:

You will want to do the following steps to run the project :-

1. First of all , install the metamask wallet on Browser.
2. In Metamask , you have to Add sepolia test network . For adding this , go to chainlist website (https://chainlist.org/?testnets=true) then click on 'Use Metamask' button in the Pop-up and make sure the 'Include Testnets' checkbox is enabled . Then search for 11155111 chain ID in 'Search Networks' input field and then click on Add to Metamask button to add network to Metamask and then click on Approve button in the pop-up by Metacrafters. The Network is added to our metamask wallet .
3. Now, we need testnet tokens in the network to run the project . For Obtaining Sepolia testnet token go to https://cloud.google.com/application/web3/faucet/ethereum/sepolia and get the testnet tokens in the network.
4. Clone the Github repository(make sure each file here is available) in your Github.
5. Copy the URL link of your Github Repository.
6. Open Gitpod(https://gitpod.io/workspaces) , then click on New Workspace and paste link of repository then click on 'continue' button and run the workspace in VS-CODE desktop or browser.
7. Inside the project directory, in the terminal type: `npm i` and wait for it to complete. This is not necessary step if you are using gitpod because in that it automatically install all the dependencies while opening the gitpod.
8. Put your private key in .env file. You will get your private key from your metamask wallet .
9. Run `npx hardhat circom`. This will generate the **out** file with circuit intermediaries and generate the **MultiplierVerifier.sol** contract
10. Run `npx hardhat run scripts/deploy.ts`. This script does 4 things  
   - Deploys the `MultiplierVerifier.sol` contract
   - Generates a proof from circuit intermediaries with `generateProof()`
   - Generates calldata with `generateCallData()`
   - Calls `verifyProof()` on the verifier contract with calldata
11. Copy 'Verifier deployed to' address from the terminal and search it on the sepolia testnet explorer(https://sepolia.etherscan.io/) then see the Transaction of Contract Creation means our script is working correctly.
12. If we get Verifier result as true in terminal then Project is successfully completed. 

With two commands you can compile a ZKP, generate a proof, deploy a verifier, and verify the proof 🎉

## Configuration
### Directory Structure
**circuits**
```
├── CustomCircuit
│   ├── circuit.circom
│   ├── input.json
│   └── out
│       ├── circuit.wasm
│       ├── CustomCircuit.r1cs
│       ├── CustomCircuit.vkey
│       └── CustomCircuit.zkey
├── new-circuit
└── powersOfTau28_hez_final_12.ptau
```
Each new circuit lives in it's own directory. At the top level of each circuit directory lives the circom circuit and input to the circuit.
The **out** directory will be autogenerated and store the compiled outputs, keys, and proofs. The Powers of Tau file comes from the Polygon Hermez ceremony, which saves time by not needing a new ceremony. 


**contracts**
```
contracts
└── CustomCircuitVerifier.sol
```
Verifier contracts are autogenerated and prefixed by the circuit name, in this example **Multiplier**

## hardhat.config.ts
```
import { HardhatUserConfig } from "hardhat/config";
import "@nomicfoundation/hardhat-toolbox";
// https://github.com/projectsophon/hardhat-circom
import "hardhat-circom";
// circuits
import circuits = require('./circuits.config.json');
require('dotenv').config();

// set env var to the root of the project
process.env.BASE_PATH = __dirname;

// tasks
import "./tasks/newcircuit.ts"

const config: HardhatUserConfig = {
  solidity: {
    compilers: [
      {
        version: "0.8.17",
      },
      {
        version: "0.6.11",
      }
    ]
  },
  networks: {
    sepolia: {
      url: 'https://rpc.sepolia.org',
      accounts: [`0x${process.env.PRIVATE_KEY}`],
    },
  },
  circom: {
    // (optional) Base path for input files, defaults to `./circuits/`
    inputBasePath: "./circuits",
    // (required) The final ptau file, relative to inputBasePath, from a Phase 1 ceremony
    ptau: "powersOfTau28_hez_final_12.ptau",
    // (required) Each object in this array refers to a separate circuit
    circuits: JSON.parse(JSON.stringify(circuits))
  },
};

export default config;
```
### circuits.config.json
circuits configuation is separated from hardhat.config.ts for **autogenerated** purposes (see next section)
```
[
  {
    "name": "CustomCircuit",
    "protocol": "groth16",
    "circuit": "CustomCircuit/circuit.circom",
    "input": "CustomCircuit/input.json",
    "wasm": "CustomCircuit/out/circuit.wasm",
    "zkey": "CustomCircuit/out/CustomCircuit.zkey",
    "vkey": "CustomCircuit/out/CustomCircuit.vkey",
    "r1cs": "CustomCircuit/out/CustomCircuit.r1cs",
    "beacon": "0102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f"
  }
]
```

**adding circuits**   
To add a new circuit, you can run the `newcircuit` hardhat task to autogenerate configuration and directories i.e  
```
npx hardhat newcircuit --name newcircuit
```

**determinism**
> When you recompile the same circuit using the groth16 protocol, even with no changes, this plugin will apply a new final beacon, changing all the zkey output files. This also causes your Verifier contracts to be updated.
> For development builds of groth16 circuits, we provide the --deterministic flag in order to use a NON-RANDOM and UNSECURE hardcoded entropy (0x000000 by default) which will allow you to more easily inspect and catch changes in your circuits. You can adjust this default beacon by setting the beacon property on a circuit's config in your hardhat.config.js file.

## Author

GAURAV GARG
STUDENT
CHANDIGARH UNIVERSITY
