<div align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/automata-network/automata-brand-kit/main/PNG/ATA_White%20Text%20with%20Color%20Logo.png">
    <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/automata-network/automata-brand-kit/main/PNG/ATA_Black%20Text%20with%20Color%20Logo.png">
    <img src="https://raw.githubusercontent.com/automata-network/automata-brand-kit/main/PNG/ATA_White%20Text%20with%20Color%20Logo.png" width="50%">
  </picture>
</div>

# Automata DCAP Attestation
[![Automata DCAP Attestation](https://img.shields.io/badge/Power%20By-Automata-orange.svg)](https://github.com/automata-network)

## Summary

Automata DCAP Attestation consists of three parts:

- PCCS Router: A central contract to read collaterals from [`automata-on-chain-pccs`](https://github.com/automata-network/automata-on-chain-pccs)

- Automata DCAP Attestation: This is the entrypoint contract for users to submit a quote to be verified. This contract parses the Quote header to identify the version, which then forwards the quote to the respective QuoteVerifier contract.

- Quote Verifier(s): This contract provides the full implementation on verifying a given quote specific to its version. This contract is intended to be called only from the Automata DCAP Attestation contract.

## On-Chain vs SNARK Attestations

Automata DCAP Attestation contract implements two attestation methods available to users. Here is a quick comparison:

|  | On-Chain | Groth16 Proof Verification with RiscZero | Groth16 Proof Verification with SP1 V3 | Plonk Proof Verification with SP1 V3| 
| --- | --- | --- | --- | --- |
| Quote Verification Time | Instant | Proving takes 2 - 5 minutes, instant verification | Proving takes <2 minutes, instant verification  | Proving takes <2 minutes, instant verification |
| Gas Cost | ~4-5M gas (varies by collateral size) | 351k gas | 325k gas | 410k gas |
| Execution | Runs fully on-chain | Execution proven by remote prover Bonsai | Execution proven by the SP1 Network | Execution proven by the SP1 Network |

## Integration

To integrate your contract with Automata DCAP Attestation, you need to first install [Foundry](https://book.getfoundry.sh/getting-started/installation).

Add to your dependency, by running:

```bash
forge install automata-network/automata-dcap-attestation
```

Then, add the following to your `remappings.txt`

```
@automata-network/dcap-attestation/=lib/automata-dcap-attestation/contracts/
```

### Example

```solidity
import "@automata-network/dcap-attestation/AutomataDcapAttestationFee.sol";

contract ExampleDcapContract {

    AutomataDcapAttestationFee attest;

    constructor(address _attest) {
        attest = AutomataDcapAttestationFee(_attest);
    }

    // On-Chain Attestation example
    function attestOnChain(bytes calldata quote) public {
        (bool success, bytes memory output) = attest.verifyAndAttestOnChain(quote);

        if (success) {
            // ... implementation to handle successful attestations
        } else {
            string memory errorMessage = string(output);
            // ... implementation to handle failed attestations
        }
    }

    // SNARK Attestation example
    // ZkCoProcessorType can either be RiscZero or Succinct
    function attestWithSnark(
        bytes calldata output,
        ZkCoProcessorType zkvm,
        bytes calldata proofBytes
    ) public 
    {
        (bool success, bytes memory output) = attest.verifyAndAttestWithZKProof(
            output,
            zkvm,
            proofBytes
        );

        if (success) {
            // ... implementation to handle successful attestations
        } else {
            string memory errorMessage = string(output);
            // ... implementation to handle failed attestations
        }
    }

}
```

To execute the DCAP RiscZero Guest Program and fetch proofs from Bonsai, we recommend checking out the [DCAP Bonsai Demo CLI](https://github.com/automata-network/dcap-bonsai-cli).

---

# BUIDL 🛠️

## Getting Started

Clone this repo, by running the following command:

```bash
git clone git@github.com:automata-network/automata-dcap-attestation.git --recurse-submodules
```

Before you begin, make sure to create a copy of the `.env` file with the example provided. Then, please provide any remaining variables that are missing.

```bash
cp .env.example .env
```

---

## Building With Foundry

Compile the contracts:

```bash
forge build
```

Testing the contracts:

```bash
forge test
```

To view gas report, pass the `--gas-report` flag.

To provide additional test cases, please include those in the `/forge-test` directory.

To provide additional scripts, please include those in the `/forge-script` directory.

### Deployment Scripts

Deploy the PCCS Router:

```bash
forge script DeployRouter --rpc-url $RPC_URL --broadcast -vvvv
```

Deploy Automata DCAP Attestation Entrypoint:

```bash
forge script AttestationScript --rpc-url $RPC_URL --broadcast -vvvv --sig "deployEntrypoint()"
```

Deploy Quote Verifier(s):

```bash
forge script DeployV3 --rpc-url $RPC_URL --broadcast -vvvv
```

The naming format for the script is simply `DeployV{x}`, where `x` is the quote version supported by the verifier. Currently, we only support V3 and V4 quotes.

Whitelist QuoteVerifier(s) in the Entrypoint contract:

```bash
forge script AttestationScript --rpc-url $RPC_URL --broadcast -vvvv --sig "configVerifier(address)" <verifier-address>
```

#### Deployment Information

The [ImageID](https://dev.risczero.com/terminology#image-id) currently used for the DCAP RiscZero Guest Program is `83613a8beec226d1f29714530f1df791fa16c2c4dfcf22c50ab7edac59ca637f`.

The [VKEY](https://docs.succinct.xyz/verification/onchain/solidity-sdk.html?#finding-your-program-vkey) currently used for the DCAP SP1 Program is
`0043e4e0c286cf4a2c03472ca2384f35a008558bc5de4e0f39d1d1bc989badca`.

> ℹ️ **Note**: 
>
> The deployment addresses shown here are currently based on the latest [changes](https://github.com/automata-network/automata-dcap-attestation/pull/6) made.
>
> To view deployments on the previous version (will be deprecated soon), you may refer to this [branch](https://github.com/automata-network/automata-dcap-attestation/tree/v0).

##### Testnet

| Contract | Network | Address |
| --- | --- | --- |
| `PCCSRouter.sol` | Automata Testnet | [0x3095741175094128ae9F451fa3693B2d23719940](https://explorer-testnet.ata.network/address/0x3095741175094128ae9F451fa3693B2d23719940) |
| `AutomataDcapAttestationFee.sol` | Automata Testnet | [0x6D67Ae70d99A4CcE500De44628BCB4DaCfc1A145](https://explorer-testnet.ata.network/address/0x6D67Ae70d99A4CcE500De44628BCB4DaCfc1A145) |
| `V3QuoteVerifier.sol` | Automata Testnet | [0x6cc70fDaB6248b374A7fD4930460F7b017190872](https://explorer-testnet.ata.network/address/0x6cc70fDaB6248b374A7fD4930460F7b017190872) |
| `V4QuoteVerifier.sol` | Automata Testnet | [0x015E89a5fF935Fbc361DcB4Bac71e5cD8a5CeEe3](https://explorer-testnet.ata.network/address/0x015E89a5fF935Fbc361DcB4Bac71e5cD8a5CeEe3) |
