# Babylon Validator Setup

## Table of Contents

1. [Prerequisites](#1prerequisites)
2. [System Requirements](#2system-requirements)
3. [Key Management](#3-key-management)
   3.1 [CometBFT Validator Keys](#31-keys-for-a-comebft-validator)
   3.2 [BLS Voting Keys](#32-keys-for-a-bls-voting)
     3.2.1 [What is BLS Voting](#321-what-is-bls-voting)
     3.2.2 [Create BLS Key](#322-create-bls-key)
  4. [Validator Configuration](#4-prepare-validator-configuration)
5. [Creating Validator](#5-creating-validator)
   5.1 [Verifying Validator Setup](#51-verifying-validator-setup)
   5.2 [Understanding Validator Status](#52-understanding-validator-status)
   5.3 [Managing Your Validator](#53-managing-your-validator)
6. [Advanced Security Architecture](#6-advanced-security-architecture)
   6.1 [Sentry Node Architecture](#61-sentry-node-architecture)
7. [Enhanced Monitoring](#7-enhanced-monitoring)
   7.1 [Prometheus Configuration](#71-prometheus-configuration)
   7.2 [Basic Health Checks](#72-basic-health-checks)

## 1. Prerequisites

Before setting up a validator, you'll need:
1. A fully synced Babylon node. For node setup instructions, see our 
[Node Setup Guide](../babylon-node/README.md)
2. Sufficient BBN tokens. For instructions on obtaining BBN tokens, see the 
[Get Funds section](../babylon-node/README.md#get-funds) in the Node Setup Guide.

## 2.System Requirements

Recommended specifications for running a Babylon validator node:
<!-- TODO: RPC Nodes more ram is required add when we have completed node load test  -->
- CPU: Quad Core AMD/Intel (amd64)
- RAM: 32GB
- Storage: 2TB NVMe
- Network: 100MBps bidirectional
- Encrypted storage for keys and sensitive data
- Regular system backups (hourly, daily, weekly)
- DDoS protection

These are reference specifications for a production validator. 
Requirements may vary based on network activity and your operational needs.

## 3. Key Management
### 3.1 Keys for a CometBFT validator

>Note: if you already have set up a key, you can skip this section.

The validator key is a fundamental component of your validator's identity 
within the Babylon network. This cryptographic key-pair serves multiple critical 
functions: it signs blocks during the consensus process, validates transactions, 
and manages your validator's operations on the network. Creating and securing this 
key is one of the most important steps in setting up your validator.

We will be using [Cosmos SDK]https://docs.cosmos.network/v0.52/user/run-node/keyring) 
backends for key storage, which offer support for the following 
keyrings:

- `test` is a password-less keyring and is unencrypted on disk.
- `os` uses the system's secure keyring and will prompt for a passphrase at startup.
- `file` encrypts the keyring with a passphrase and stores it on disk.


To generate your validator key, use the following command:

```shell
babylond keys add <name> --home <path> --keyring-backend <keyring-backend>
```

Parameters:
The `<name>` specifies a unique identifier for the key.
- `--home` specifies the directory where your node files will be stored 
  (e.g. `--home ./nodeDir`)
- `--keyring-backend` specifies the keyring backend to use, can be `test`, `file`, or `os`.

The execution result displays the address of the newly generated key and its 
public key. Following is a sample output for the command:

```shell
- address: bbn1kvajzzn6gtfn2x6ujfvd6q54etzxnqg7pkylk9
  name: <name>
  pubkey: '{"@type":"/cosmos.crypto.secp256k1.PubKey",
           key: "Ayau+8f945c1iQp9tfTVaCT5lzhD8n4MRkZNqpoL6Tpo"}'
  type: local
```

Make sure to securely store this information, particularly your address and 
private key details. Losing access to these credentials would mean losing 
control of your validator and any staked funds.

## Get Funds

To interact with the Babylon network, you'll need some BBN tokens to:
1. Pay for transaction fees (gas)
2. Send transactions
3. Participate in network activities

You can obtain testnet tokens through two methods:

1. Request funds from the Babylon Testnet Faucet 
[here](#tbd) 
<!-- add link to faucet -->
2. Join our Discord server and visit the `#faucet` channel: 
[Discord Server](https://discord.com/channels/1046686458070700112/1075371070493831259)

Note: These are testnet tokens with no real value, used only for testing 
and development purposes.

## Keys for a BLS Voting
### What is BLS Voting

Babylon validators are required to participate in
[BLS](https://en.wikipedia.org/wiki/BLS_digital_signature) voting
at the end of each epoch.
The Babylon blockchain defines epochs as a fixed period
within the blockchain, defined or set by a number of blocks,
during which the validator set remains consistent. 
At the end of the epoch,
the validator BLS signatures are aggregated to create a compact checkpoint
that is timestamped on the Bitcoin ledger.
The BLS voting mechanism achieves a significant reduction in the cost of
checkpoints, while the epoching mechanism specifies a defined frequency
for checkpointing in the Bitcoin blockchain.

### Create BLS Key

To generate your BLS key, you'll need to use your validator address from the 
previous step. 

Run this command:

```shell
babylond create-bls-key <address>
```

Replace `<address>` with your validator address from the earlier keyring 
generation (it should look similar to `bbn1kvajzzn6gtfn2x6ujfvd6q54etzxnqg7pkylk9`). 

The system will automatically:
1. Generate a new BLS key pair
2. Associate it with your validator address
3. Store it in your node's configuration file at 
`~/.<path>/config/priv_validator_key.json`

This key will be used automatically by your validator software when it needs 
to participate in epoch-end signature collection. The BLS signatures help 
create compact, efficient proofs of consensus that can be later timestamped to Bitcoin.

Important: The `priv_validator_key.json` file contains sensitive key material. 
Make sure to backup this file and store it securely, as it's essential for your 
validator's operation and cannot be recovered if lost.

## Prepare Validator Configuration

Create your validator configuration file:

```shell
cat > <path>/config/validator.json << EOF
{
  "pubkey": $(cat pubkey.json),
  "amount": "1000ubbn",
  "moniker": "my-validator",
  "commission-rate": "0.10",
  "commission-max-rate": "0.20",
  "commission-max-change-rate": "0.01",
  "min-self-delegation": "1"
  "website": "https://myweb.site",
  "security": "security-contact@gmail.com",
  "details": "description of your validator",
}
EOF
```

This command creates the configuration file with your validator settings:
- `pubkey`: Your validator's public key
- `amount`: Initial self-delegation amount
- `moniker`: Your validator's name/identifier
- `commission-rate`: Current commission rate 
- `commission-max-rate`: Maximum commission rate allowed
- `commission-max-change-rate`: Maximum daily commission change rate
- `min-self-delegation`: Minimum amount you must keep self-delegated
- `website`: (Optional) Your validator's website
- `security`: (Optional) Security contact email
- `details`: (Optional) Description of your validator

> Note: The command will create the file if it doesn't exist. No need for a separate `touch` command.

## Creating Validator

> ⚠️ **Important:** You will need a funded account for this step

Unlike traditional Cosmos SDK chains that use the `staking` module, 
Babylon uses the [`checkpointing`](https://docs.babylonlabs.io/docs/developer-guides/modules/checkpointing) module for validator creation and management.

The creation process requires your previously generated BLS key, 
which should be located at `<path>/config/priv_validator_key.json`, 
where `<path>` is the `--home` directory you specified when setting up your node.

To create your validator, run the following command:

```shell 
babylond tx checkpointing create-validator \
    ./<path>/config/validator.json \
    --chain-id bbn-test-5 \
    --gas "auto" \
    --gas-adjustment 1.5 \
    --gas-prices "0.005ubbn" \
    --from <your-key-name>
```

- `--chain-id`: The network identifier
- `--gas`: Set to "auto" to automatically calculate the gas needed
- `--gas-adjustment`: A multiplier for the estimated gas (1.5 adds 50% extra for safety)
- `--gas-prices`: Transaction fee in ubbn per unit of gas
- `--from`: Your key name or address that will sign and pay for this transaction

Upon successful creation, you'll be asked to approve the transaction. 
Once approved, you'll receive a transaction hash and your validator
operator address 
(e.g., `bbnvaloper1qh8444k43spt6m8ernm8phxr332k85teavxmuq`).

### Verifying Validator Setup

To verify your validator setup, you can use the following steps:

First, get your validator's operator address using your Babylon address: 

```shell
babylond keys show <your-key-name> --address --bech val --home <path> --keyring-backend test
```

For example, for the address we used above is `bbn1qh8444k43spt6m8ernm8phxr332k85teavxmuq`, 
the operator address is `bbnvaloper1qh8444k43spt6m8ernm8phxr332k85teavxmuq`. 

Next, inspect your validator's details: 

```shell 
babylond query staking validator <validator-operator-address> --home <path>
```

The output should return your validator's configuration.

```yaml 
validator:
  commission:
    rates:
      current: "100000000000000000" 
      max: "1000000000000000000" 
      max_change: "10000000000000000"
  description:
    moniker: "my-validator" 
    website: "https://myweb.site" 
    security_contact: "my-validator0@gmail.com"
  status: 1 tokens: "100"
```

### Understanding Validator Status

Your validator enters the active set based on two conditions: 
1. The completion of the current epoch (a network-wide time period for 
coordinating activities) 
2. Having sufficient stake to qualify for the active set.

When active, your status will show as `BOND_STATUS_BONDED`.

### Managing Your Validator

For delegation operations (`delegate`, `redelegate`, `unbond`, `cancel-unbond`),
you must use the wrapped messages in the `checkpointing` and `epoching` modules.
This is because standard staking module messages are disabled in Babylon.

For detailed information about these operations, visit our
[documentation](https://docs.babylonlabs.io/docs/developer-guides/modules/epoching#delaying-wrapped-messages-to-the-end-of-epochs).

## Advanced Security Architecture

Validators should be run with additional security measures:

**Sentry Node Architecture**  
For enhanced security, validators should implement a sentry node architecture. 
This involves deploying sentry nodes as a protective layer around your 
validator node, following the established [Cosmos Sentry Node Architecture](https://forum.cosmos.network/t/sentry-node-architecture-overview/454). 
The sentry nodes act as a buffer between your validator and the public network, 
with a private network maintained between your validator and its sentry nodes. 
This architecture helps protect your validator from DDoS attacks and other 
network-level threats by ensuring your validator only communicates with trusted 
sentry nodes rather than directly with the public network.

## Enhanced Monitoring

In addition to basic node monitoring, validators should:

1. Monitor validator performance metrics
   - Signing performance
   - Missed blocks
   - Voting power changes
   - BLS signing status

2. Set up alerts for:
   - Slashing conditions
   - Validator status changes
   - Network participation metrics
   - System resource thresholds

### Prometheus Configuration

This information can be found in the [Node Monitoring](../babylon-node/README.md#monitoring-your-node) section.

Example Prometheus scrape configuration:

```yaml
scrape_configs:
  - job_name: 'babylon_validator'
    static_configs:
      - targets: ['localhost:26660', 'localhost:1317']
    metrics_path: '/metrics'
    scrape_interval: 10s
```

We do this to scrape metrics to monitor your validator's health and performance.

### Basic Health Checks

Implement these essential health checks:
```bash
# Check node sync status
curl -s localhost:26657/status | jq '.result.sync_info.catching_up'

# Compare local vs network height
curl -s localhost:26657/status | jq '.result.sync_info.latest_block_height'

# Monitor validator signing status
curl -s localhost:26657/consensus_state | jq '.result.round_state.height_vote_set[0].prevotes_bit_array'
```

We recommend setting up alerts when:
- Node falls out of sync (catching_up = true)
- Block height falls behind network
- Missing validator signatures
- Prometheus metrics become unavailable

Congratulations! Your validator is now part of the Babylon network. Remember to
monitor your validator's performance and maintain good uptime to avoid
penalties.