# All starter code is available after this explanation.

This project utilizes the hardhat-circom library called Zardkat for writing Zero-knowledge circuits, generating circuits & also deploying verifiers.

Various templates used in circuit.circom file are

├── binaryCheck --> Validates if the given input is a binary.

├── And2 --> logical equivalent of AND gate.

├── Not --> logical equivalent of NOT gate

├── Or2 --> logical equivalent of OR gate

├── Mainer2 --> execution template or main component

### Execution Template: Mainer2()

in1 ~ A & in2 ~ B

input signals in1 and in2 are captured
And, Or, Not functions are loaded as components ander, orer, noter
And of in1 & in2 are calculated and output is stored in orer.in1
Not of in2 calculated and output is given to orer.in2
The function output is calculated finally
```circom
template Mainer2(){
   // Declaration of signals and components
   signal input in1;
   signal input in2;
   signal output out;
   component ander = And2();
   component orer = Or2();
   component noter = Not();
   //And and Not values
   ander.in1 <== in1;
   ander.in2 <== in2;
   noter.in1 <== in1;
   orer.in1 <== ander.out;
   orer.in2 <== noter.out;
   out <== orer.out;

}
```
### input.json file

in1 == 0 == A

in2 == 1 == B

```json
{
  "in1": "0",
  "in2": "1"
}
```

# zardkat 🐱

A [hardhat-circom](https://github.com/projectsophon/hardhat-circom) template to generate zero-knowledge circuits, proofs, and solidity verifiers

## Quick Start
Compile the Multiplier2() circuit and verify it against a smart contract verifier

### Install
`npm i`

### Compile
`npx hardhat compile` 
This will generate the **out** file with circuit intermediaries and geneate the **MultiplierVerifier.sol** contract

### Prove and Deploy
`npx hardhat run scripts/deploy.ts`
This script does 4 things  
1. Deploys the MultiplierVerifier.sol contract
2. Generates a proof from circuit intermediaries with `generateProof()`
3. Generates calldata with `generateCallData()`
4. Calls `verifyProof()` on the verifier contract with calldata

With two commands you can compile a ZKP, generate a proof, deploy a verifier, and verify the proof 🎉

## Configuration
### Directory Structure
**circuits**
```
├── multiplier
│   ├── circuit.circom
│   ├── input.json
│   └── out
│       ├── circuit.wasm
│       ├── multiplier.r1cs
│       ├── multiplier.vkey
│       └── multiplier.zkey
├── new-circuit
└── powersOfTau28_hez_final_12.ptau
```
Each new circuit lives in it's own directory. At the top level of each circuit directory lives the circom circuit and input to the circuit.
The **out** directory will be autogenerated and store the compiled outputs, keys, and proofs. The Powers of Tau file comes from the Polygon Hermez ceremony, which saves time by not needing a new ceremony. 


**contracts**
```
contracts
└── MultiplierVerifier.sol
```
Verifier contracts are autogenerated and prefixed by the circuit name, in this example **Multiplier**

## hardhat.config.ts
```
  circom: {
    // (optional) Base path for input files, defaults to `./circuits/`
    inputBasePath: "./circuits",
    // (required) The final ptau file, relative to inputBasePath, from a Phase 1 ceremony
    ptau: "powersOfTau28_hez_final_12.ptau",
    // (required) Each object in this array refers to a separate circuit
    circuits: JSON.parse(JSON.stringify(circuits))
  },
```
### circuits.config.json
circuits configuation is separated from hardhat.config.ts for **autogenerated** purposes (see next section)
```
[
  {
    "name": "multiplier",
    "protocol": "groth16",
    "circuit": "multiplier/circuit.circom",
    "input": "multiplier/input.json",
    "wasm": "multiplier/out/circuit.wasm",
    "zkey": "multiplier/out/multiplier.zkey",
    "vkey": "multiplier/out/multiplier.vkey",
    "r1cs": "multiplier/out/multiplier.r1cs",
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
