# UpRoll Optimism Package
This is a fork of the optimism package built to allow more chain configurations. Its intended purpose is to give [UpRoll CLI](https://github.com/zapper-95/UpRoll-cli.git) greater control over deployed rollups.

## New Configurations
### Signer Information
In the upstream package, private keys are determined by the mnemonic `test test test test test test test test test test test junk` and cannot be customised.

We introduce customisation for private keys and signer information (address and endpoint) for each privileged role.


| Role       | Field       | YAML Path                                    |
|------------|---------------|-----------------------------------------|
| **Batcher**    | private_key    | `chain.batcher_params.private_key`    |
|            | signer_endpoint | `chain.batcher_params.signer_endpoint` |
|            | signer_address  | `chain.batcher_params.signer_address`  |
| **Sequencer**  | private_key    | `chain.sequencer_params.private_key`  |
|            | signer_endpoint | `chain.sequencer_params.signer_endpoint` |
|            | signer_address  | `chain.sequencer_params.signer_address`  |
| **Proposer**   | private_key    | `chain.proposer_params.private_key`   |
|            | signer_endpoint | `chain.proposer_params.signer_endpoint` |
|            | signer_address  | `chain.proposer_params.signer_address`  |
| **Challenger** | private_key    | `chain.challenger_params.private_key` |
|            | signer_endpoint | `chain.challenger_params.signer_endpoint` |
|            | signer_address  | `chain.challenger_params.signer_address` |

### Network Parameters
| Field                         | YAML Path                                              |
|---------------------------------|---------------------------------------------------|
| Withdrawal delay              | `chain.network_params.withdrawal_delay`          |
| Fee withdrawal network          | `chain.network_params.fee_withdrawal_network`    |
| Dispute game finality delay     | `chain.network_params.dispute_game_finality_delay` |

### Gas Parameters
| Field                   | YAML Path                                       |
|-----------------------------|--------------------------------------------|
| Block gas limit             | `chain.gas_params.gas_limit`               |
| EIP 1559 Elasticity         | `chain.gas_params.eip_1559_elasticity`       |
| EIP 1559 Denominator        | `chain.gas_params.eip_1559_denominator`   |
| Base Fee Scalar            | `chain.gas_params.base_fee_scalar`         |
| Blob Base Fee Scalar       | `chain.gas_params.blob_base_fee_scalar`    |


### Data Availability
| Field                         | YAML Path                                                           |
|-----------------------------------|----------------------------------------------------------------|
| Data availability type           | `optimism_package.altda_deploy_config.da_type`                 |
| Batch submissions frequency      | `optimism_package.altda_deploy_config.da_batch_submission_frequency` |
| DA server endpoint               | `chain.da_server_params.server_endpoint`                      |
| DA Challenge Contract Address    | `optimism_package.altda_deploy_config.da_challenge_contract_address` |



## Quickstart

### Run with your own configuration

Kurtosis packages are parameterizable, meaning you can customize your network and its behavior to suit your needs by storing parameters in a file that you can pass in at runtime like so:

```bash
kurtosis run github.com/ethpandaops/optimism-package --args-file https://raw.githubusercontent.com/ethpandaops/optimism-package/main/network_params.yaml
```

For `--args-file` parameters file, you can pass a local file path or a URL to a file.

To clean up running enclaves and data, you can run:

```bash
kurtosis clean -a
```

This will stop and remove all running enclaves and **delete all data**.

### Run with changes to the optimism package

If you are attempting to test any changes to the package code, you can point to the directory as the `run` argument

```bash
cd ~/go/src/github.com/ethpandaops/optimism-package
kurtosis run . --args-file ./network_params.yaml
```

## L2 Contract deployer

The enclave will automatically deploy an optimism L2 contract on the L1 network. The contract address will be printed in the logs. You can use this contract address to interact with the L2 network.

Please refer to this Dockerfile if you want to see how the contract deployer image is built: [Dockerfile](https://github.com/ethereum-optimism/optimism/blob/develop/op-deployer/Dockerfile.default)

## Configuration

To configure the package behaviour, you can modify your `network_params.yaml` file and use that as the input to `--args-file`.
The full YAML schema that can be passed in is as follows with the defaults provided:

```yaml
optimism_package:
  # Observability configuration
  observability:
    # Whether or not to configure observability (e.g. prometheus)
    enabled: true
    # Default prometheus configuration
    prometheus_params:
      storage_tsdb_retention_time: "1d"
      storage_tsdb_retention_size: "512MB"
      # Resource management for prometheus container
      # CPU is milicores
      # RAM is in MB
      min_cpu: 10
      max_cpu: 1000
      min_mem: 128
      max_mem: 2048
      # Prometheus docker image to use
      # Defaults to the latest image
      image: "prom/prometheus:v3.2.1"
    # Default grafana configuration
    grafana_params:
      # A list of locators for grafana dashboards to be loaded by the grafana service.
      # Each locator should be a URL to a directory containing a /folders and a /dashboards directory.
      # Those will be uploaded to the grafana service by using grizzly.
      # See https://github.com/ethereum-optimism/grafana-dashboards-public for more info.
      dashboard_sources:
        - github.com/ethereum-optimism/grafana-dashboards-public/resources
      # Resource management for grafana container
      # CPU is milicores
      # RAM is in MB
      min_cpu: 10
      max_cpu: 1000
      min_mem: 128
      max_mem: 2048
      # Grafana docker image to use
      # Defaults to the latest image
      image: "grafana/grafana:11.5.2"
  # Interop configuration
  interop:
    # Whether or not to enable interop mode
    enabled: false
    # Default supervisor configuration
    supervisor_params:
      # The Docker image that should be used for the supervisor; leave blank to use the default op-supervisor image
      image: ""

      # A JSON string containing chain dependencies (generated by default).
      dependency_set: ""

      # A list of optional extra params that will be passed to the supervisor container for modifying its behaviour
      extra_params: []

  # AltDA Deploy Configuration, which is passed to op-deployer.
  #
  # For simplicity we currently enforce chains to all be altda or all rollups.
  # Adding a single altda chain to a cluster essentially makes all chains have altda levels of security.
  #
  # To setup an altda cluster, make sure to
  # 1. Set altda_deploy_config.use_altda to true (and da_commitment_type to KeccakCommitment, see TODO below)
  # 2. For each chain,
  #    - Add "da_server" to the additional_services list if it should use alt-da
  #    - For altda chains, set da_server_params to use an image and cmd of your choice (one could use da-server, another eigenda-proxy, another celestia proxy, etc). If unset, op's default da-server image will be used.
  altda_deploy_config:
    use_altda: false
  
    # Specifies how transactions are posted. Allows auto, blobs calldata or custom. 
    da_type: "calldata" 

    # Determines how frequently (in minutes) the batcher submits aggregated transaction data to L1 (via the batcher transaction).
    da_batch_submission_frequency: 1

    # Represents the L1 address of the DataAvailabilityChallenge contract.
    da_challenge_contract_address: ""

    # Specifies the allowed commitment. Either KeccakCommitment or GenericCommitment. However, KeccakCommitment will be deprecated soon.
    da_commitment_type: KeccakCommitment

    # Represents the block interval during which the availability of a data commitment can be challenged.
    da_challenge_window: 100

    # Represents the block interval during which a data availability challenge can be resolved.
    da_resolve_window: 100

    # Represents the required bond size to initiate a data availability challenge.
    da_bond_size: 1000

    # Represents the percentage of the resolving cost to be refunded to the resolver such as 100 means 100% refund.
    da_resolver_refund_percentage: 0

  # An array of L2 networks to run
  chains:
    # Specification of the optimism-participants in the network
    - participants:
      # EL(Execution Layer) Specific flags
        # The type of EL client that should be started
        # Valid values are:
        # op-geth
        # op-reth
        # op-erigon
        # op-nethermind
        # op-besu
      - el_type: op-geth

        # The Docker image that should be used for the EL client; leave blank to use the default for the client type
        # Defaults by client:
        # - op-geth: us-docker.pkg.dev/oplabs-tools-artifacts/images/op-geth:v1.101503.1
        # - op-reth: parithoshj/op-reth:v1.3.4
        # - op-erigon: testinprod/op-erigon:v2.61.3-0.8.4
        # - op-nethermind: nethermindeth/nethermind:1.31.6
        # - op-besu: ghcr.io/optimism-java/op-besu:v0.2.2
        el_image: ""

        # The log level string that this participant's EL client should log at
        # If this is emptystring then the global `logLevel` parameter's value will be translated into a string appropriate for the client (e.g. if
        # global `logLevel` = `info` then Geth would receive `3`, Besu would receive `INFO`, etc.)
        # If this is not emptystring, then this value will override the global `logLevel` setting to allow for fine-grained control
        # over a specific participant's logging
        el_log_level: ""

        # A list of optional extra env_vars the el container should spin up with
        el_extra_env_vars: {}

        # A list of optional extra labels the el container should spin up with
        # Example; el_extra_labels: {"ethereum-package.partition": "1"}
        el_extra_labels: {}

        # A list of optional extra params that will be passed to the EL client container for modifying its behaviour
        el_extra_params: []

        # A list of tolerations that will be passed to the EL client container
        # Only works with Kubernetes
        # Example: el_tolerations:
        # - key: "key"
        #   operator: "Equal"
        #   value: "value"
        #   effect: "NoSchedule"
        #   toleration_seconds: 3600
        # Defaults to empty
        el_tolerations: []

        # Persistent storage size for the EL client container (in MB)
        # Defaults to 0, which means that the default size for the client will be used
        # Default values can be found in /src/package_io/constants.star VOLUME_SIZE
        el_volume_size: 0

        # Resource management for el containers
        # CPU is milicores
        # RAM is in MB
        # Defaults to 0, which results in no resource limits
        el_min_cpu: 0
        el_max_cpu: 0
        el_min_mem: 0
        el_max_mem: 0

      # CL(Consensus Layer) Specific flags
        # The type of CL client that should be started
        # Valid values are:
        # op-node
        # hildr
        cl_type: op-node

        # The Docker image that should be used for the CL client; leave blank to use the default for the client type
        # Defaults by client:
        # - op-node: us-docker.pkg.dev/oplabs-tools-artifacts/images/op-node:df1d18acb5f151b5d32b5e78df9c1331ed770505
        # - hildr: ghcr.io/optimism-java/hildr:v0.4.5
        cl_image: ""

        # The log level string that this participant's CL client should log at
        # If this is emptystring then the global `logLevel` parameter's value will be translated into a string appropriate for the client (e.g. if
        # If this is not emptystring, then this value will override the global `logLevel` setting to allow for fine-grained control
        # over a specific participant's logging
        cl_log_level: ""

        # A list of optional extra env_vars the cl container should spin up with
        cl_extra_env_vars: {}

        # A list of optional extra labels that will be passed to the CL client Beacon container.
        # Example; cl_extra_labels: {"ethereum-package.partition": "1"}
        cl_extra_labels: {}

        # A list of optional extra params that will be passed to the CL client Beacon container for modifying its behaviour
        # If the client combines the Beacon & validator nodes (e.g. Teku, Nimbus), then this list will be passed to the combined Beacon-validator node
        cl_extra_params: []

        # A list of tolerations that will be passed to the CL client container
        # Only works with Kubernetes
        # Example: el_tolerations:
        # - key: "key"
        #   operator: "Equal"
        #   value: "value"
        #   effect: "NoSchedule"
        #   toleration_seconds: 3600
        # Defaults to empty
        cl_tolerations: []

        # Persistent storage size for the CL client container (in MB)
        # Defaults to 0, which means that the default size for the client will be used
        # Default values can be found in /src/package_io/constants.star VOLUME_SIZE
        cl_volume_size: 0

        # Resource management for cl containers
        # CPU is milicores
        # RAM is in MB
        # Defaults to 0, which results in no resource limits
        cl_min_cpu: 0
        cl_max_cpu: 0
        cl_min_mem: 0
        cl_max_mem: 0

      # Builder client specific flags
        # The type of builder EL client that should be started
        # Valid values are:
        # op-geth
        # op-reth
        el_builder_type: ""

        # The Docker image that should be used for the builder EL client; leave blank to use the default for the client type
        # Defaults by client:
        # - op-geth: us-docker.pkg.dev/oplabs-tools-artifacts/images/op-geth:v1.101503.1
        # - op-reth: parithoshj/op-reth:v1.3.4
        el_builder_image: ""

        # The type of builder CL client that should be started
        # Valid values are:
        # op-node
        # hildr
        cl_builder_type: ""

        # The Docker image that should be used for the builder CL client; leave blank to use the default for the client type
        # Defaults by client:
        # - op-node: us-docker.pkg.dev/oplabs-tools-artifacts/images/op-node:df1d18acb5f151b5d32b5e78df9c1331ed770505
        # - hildr: ghcr.io/optimism-java/hildr:v0.4.5
        cl_builder_image: ""

        # Participant specific flags
        # Node selector
        # Only works with Kubernetes
        # Example: node_selectors: { "disktype": "ssd" }
        # Defaults to empty
        node_selectors: {}

        # A list of tolerations that will be passed to the EL/CL/validator containers
        # This is to be used when you don't want to specify the tolerations for each container separately
        # Only works with Kubernetes
        # Example: tolerations:
        # - key: "key"
        #   operator: "Equal"
        #   value: "value"
        #   effect: "NoSchedule"
        #   toleration_seconds: 3600
        # Defaults to empty
        tolerations: []

        # Count of nodes to spin up for this participant
        # Default to 1
        count: 1

      # Default configuration parameters for the network
      network_params:
        # Network name, used to enable syncing of alternative networks
        # Defaults to "kurtosis"
        network: "kurtosis"

        # The network ID of the network.
        # Must be unique for each network (if you run multiple networks)
        # Defaults to "2151908"
        network_id: "2151908"

        # Seconds per slots
        seconds_per_slot: 2

        # Name of your rollup.
        # Must be unique for each rollup (if you run multiple rollups)
        # Defaults to "op-kurtosis"
        name: "op-kurtosis"

        # Triggering future forks in the network
        # Fjord fork
        # Defaults to 0 (genesis activation) - decimal value
        # Offset is in seconds
        fjord_time_offset: 0

        # Granite fork
        # Defaults to 0 (genesis activation) - decimal value
        # Offset is in seconds
        granite_time_offset: 0

        # Holocene fork
        # Defaults to None - not activated - decimal value
        # Offset is in seconds
        holocene_time_offset: ""

        # Isthmus fork
        # Defaults to None - not activated - decimal value
        # Offset is in seconds
        isthmus_time_offset: ""

        # Interop fork
        # Defaults to None - not activated - decimal value
        # Offset is in seconds
        interop_time_offset: ""

        # Whether to fund dev accounts on L2
        # Defaults to True
        fund_dev_accounts: true

        # The number of seconds that a proof must be mature before it can be used to finalize a withdrawal.
        withdrawal_delay: 604800

        # Sets the fee withdrawal network (0 for L1, 1 for L2) for baseFee, l1FeeVault, sequencerFee, 
        fee_withdrawal_network: 0 

        # An additional number of seconds a dispute game must wait before it can be used to finalize a withdrawal.
        dispute_game_finality_delay: 302400

      # Default batcher configuration
      batcher_params:
        # The Docker image that should be used for the batcher; leave blank to use the default op-batcher image
        image: ""


        # If using a testnet, use either a private key or signer information (signer_endpoint and signer_address), but not both
        private_key: ""  
        signer_endpoint: "" # endpoint of the signer
        signer_address: "" # wallet address of the signer


        # A list of optional extra params that will be passed to the batcher container for modifying its behaviour
        extra_params: []

      # Default challenger configuration
      challenger_params:
        # Whether or not to enable the challenger
        enabled: true

        # The Docker image that should be used for the challenger; leave blank to use the default op-challenger image
        image: ""


        # If using a testnet, use either a private key or signer information (signer_endpoint and signer_address), but not both
        private_key: ""  
        signer_endpoint: "" # endpoint of the signer
        signer_address: "" # wallet address of the signer

        # A list of optional extra params that will be passed to the challenger container for modifying its behaviour
        extra_params: []

        # Path to folder containing cannon prestate-proof.json file
        cannon_prestates_path: "static_files/prestates"

        # Base URL to absolute prestates to use when generating trace data.
        cannon_prestates_url: ""

      # Default proposer configuration
      proposer_params:
        # The Docker image that should be used for the proposer; leave blank to use the default op-proposer image
        image: ""

        # If using a testnet, use either a private key or signer information (signer_endpoint and signer_address), but not both
        private_key: ""  
        signer_endpoint: "" # endpoint of the signer
        signer_address: "" # wallet address of the signer

        # A list of optional extra params that will be passed to the proposer container for modifying its behaviour
        extra_params: []

        # Dispute game type to create via the configured DisputeGameFactory
        game_type: 1

        # Interval between submitting L2 output proposals
        proposal_internal: 10m

      sequencer_params:
        # If using a testnet, use either a private key or signer information (signer_endpoint and signer_address), but not both
        private_key: ""  
        signer_endpoint: "" # endpoint of the signer
        signer_address: "" # wallet address of the signer
      
      gas_params:

        # Represents the chain's block gas limit.
        gas_limit: "0x17D7840"

        # The elasticity of the EIP1559 fee market.
        eip_1559_elasticity: 6

        # The denominator of EIP1559 base fee market.
        eip_1559_denominator: 50

        # Represents the value of the base fee scalar used for fee calculations.
        base_fee_scalar: 2

        # Represents the value of the blob base fee scalar used for fee calculations.
        blob_base_fee_scalar: 1
  
      # Default MEV configuration
      mev_params:
        # The Docker image that should be used for rollup boost; leave blank to use the default rollup-boost image
        # Defaults to "flashbots/rollup-boost:sha-628bb2d"
        rollup_boost_image: ""

        # The host of an external builder
        builder_host: ""

        # The port of an external builder
        builder_port: ""

      # Additional services to run alongside the network
      # Defaults to []
      # Available services:
      # - blockscout
      # - rollup-boost
      # - da_server
      additional_services: []

      # Configuration for da-server - https://specs.optimism.io/experimental/alt-da.html#da-server
      # TODO: each op-node and op-batcher should potentially have their own da-server, instead of sharing one like we currently do. For eg batcher needs to write via its da-server, whereas op-nodes don't.
      da_server_params:
        # include your custom da server endpoint
        server_endpoint: ""

  # L2 contract deployer configuration - used for all L2 networks
  # The docker image that should be used for the L2 contract deployer
  op_contract_deployer_params:
    image: us-docker.pkg.dev/oplabs-tools-artifacts/images/op-deployer:v0.0.11
    l1_artifacts_locator: https://storage.googleapis.com/oplabs-contract-artifacts/artifacts-v1-c193a1863182092bc6cb723e523e8313a0f4b6e9c9636513927f1db74c047c15.tar.gz
    l2_artifacts_locator: https://storage.googleapis.com/oplabs-contract-artifacts/artifacts-v1-c193a1863182092bc6cb723e523e8313a0f4b6e9c9636513927f1db74c047c15.tar.gz

  # The global log level that all clients should log at
  # Valid values are "error", "warn", "info", "debug", and "trace"
  # This value will be overridden by participant-specific values
  global_log_level: "info"

  # Global node selector that will be passed to all containers (unless overridden by a more specific node selector)
  # Only works with Kubernetes
  # Example: global_node_selectors: { "disktype": "ssd" }
  # Defaults to empty
  global_node_selectors: {}

  # Global tolerations that will be passed to all containers (unless overridden by a more specific toleration)
  # Only works with Kubernetes
  # Example: tolerations:
  # - key: "key"
  #   operator: "Equal"
  #   value: "value"
  #   effect: "NoSchedule"
  #   toleration_seconds: 3600
  # Defaults to empty
  global_tolerations: []

  # Whether the environment should be persistent; this is WIP and is slowly being rolled out across services
  # Defaults to false
  persistent: false

# Ethereum package configuration
ethereum_package:
  network_params:
    # The Ethereum network preset to use
    preset: minimal
    # The delay in seconds before the genesis block is mined
    genesis_delay: 5
    # Preloaded contracts for the Ethereum network
    additional_preloaded_contracts: '
      {
        "0x4e59b44847b379578588920cA78FbF26c0B4956C": {
          "balance": "0ETH",
          "code": "0x7fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffe03601600081602082378035828234f58015156039578182fd5b8082525050506014600cf3",
          "storage": {},
          "nonce": "1"
        }
      }
    '

```

### Additional configuration recommendations

#### L1 customization

It is required for you to launch an L1 Ethereum node to interact with the L2 network. You can use the `ethereum_package` to launch an Ethereum node. The `ethereum_package` configuration is as follows:

```yaml
optimism_package:
  chains:
    - participants:
        - el_type: op-geth
          cl_type: op-node
      additional_services:
        - blockscout
ethereum_package:
  participants:
    - el_type: geth
    - el_type: reth
  network_params:
    preset: minimal
    genesis_delay: 5
    additional_preloaded_contracts: '
      {
        "0x4e59b44847b379578588920cA78FbF26c0B4956C": {
          "balance": "0ETH",
          "code": "0x7fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffe03601600081602082378035828234f58015156039578182fd5b8082525050506014600cf3",
          "storage": {},
          "nonce": "1"
        }
      }
    '
  additional_services:
    - dora
    - blockscout
```

#### L2 customization with Hard Fork transitions

To spin up an L2 chain with specific hard fork transition blocks and any local docker image to run the EL/CL components,
use the `network_params` section of your arguments file to specify the hard fork transitions and custom images.

```yaml
optimism_package:
  chains:
    - participants:
      - el_type: op-geth
        el_image: "us-docker.pkg.dev/oplabs-tools-artifacts/images/op-geth:<tag>"
        cl_type: op-node
        cl_image: "us-docker.pkg.dev/oplabs-tools-artifacts/images/op-node:<tag>"
      - el_type: op-geth
        el_image: "us-docker.pkg.dev/oplabs-tools-artifacts/images/op-geth:<tag>"
        cl_type: op-node
        cl_image: "us-docker.pkg.dev/oplabs-tools-artifacts/images/op-node:<tag>"
      network_params:
        fjord_time_offset: 0
        granite_time_offset: 0
        holocene_time_offset: 4
        isthmus_time_offset: 8
```

#### Multiple L2 chains

Additionally, you can spin up multiple L2 networks by providing a list of L2 configuration parameters like so:

```yaml
optimism_package:
  chains:
    - participants:
        - el_type: op-geth
      network_params:
        name: op-rollup-one
        network_id: "3151909"
      additional_services:
        - blockscout
    - participants:
        - el_type: op-geth
      network_params:
        name: op-rollup-two
        network_id: "3151910"
      additional_services:
        - blockscout
ethereum_package:
  participants:
    - el_type: geth
    - el_type: reth
  network_params:
    preset: minimal
    genesis_delay: 5
    additional_preloaded_contracts: '
      {
        "0x4e59b44847b379578588920cA78FbF26c0B4956C": {
          "balance": "0ETH",
          "code": "0x7fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffe03601600081602082378035828234f58015156039578182fd5b8082525050506014600cf3",
          "storage": {},
          "nonce": "1"
        }
      }
    '
  additional_services:
    - dora
    - blockscout
```

Note: if configuring multiple L2s, make sure that the `network_id` and `name` are set to differentiate networks.

#### Rollup Boost for External Block Building

Rollup Boost is a sidecar to the sequencer op-node that allows blocks to be built by an external builder on the L2 network.

To use rollup boost, you can add `rollup-boost` as an additional service and configure the `mev_params` section of your arguments file to specify the rollup boost image. Optionally, you can specify the host and port of an external builder outside of the Kurtosis enclave.

```yaml
optimism_package:
  chains:
    - participants:
        - el_builder_type: op-geth
          cl_builder_type: op-node
      mev_params:
        rollup_boost_image: "flashbots/rollup-boost:sha-628bb2d"
        builder_host: "localhost"
        builder_port: "8545"
      additional_services:
        - rollup-boost
```

#### Run tx-fuzz to send l2 transactions

Compile [tx-fuzz](https://github.com/MariusVanDerWijden/tx-fuzz) locally per instructions in the repo. Run tx-fuzz against the l2 EL client's RPC URL and using the pre-funded wallet:

```bash
./livefuzzer spam  --sk "0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80" --rpc http://127.0.0.1:<port> --slot-time 2
```

### Additional configurations

Please find examples of additional configurations in the [test folder](.github/tests/).

### Useful Kurtosis commands

#### Inspect enclave -- Container/Port information

- List information about running containers and open ports

```bash
kurtosis enclave ls
kurtosis enclave inspect <enclave-name>
```

- Inspect chain state.

```bash
kurtosis files inspect <enclave-name> op-deployer-configs
```

- Dump all files generated by kurtosis to disk (for inspecting chain state/deploy configs/contract addresses etc.). A file that contains an exhaustive
  set of information about the current deployment is `files/op-deployer-configs/state.json`. Deployed contract address, roles etc can all be found here.

```bash
# dumps all files to a enclave-name prefixed directory under the current directory
kurtosis enclave dump <enclave-name>
kurtosis files download <enclave-name> op-deployer-configs <where-to-download>
```

- Get logs for running services

```bash
kurtosis service logs <enclave-name> <service-name> -f . # -f tails the log
```

- Stop/Start running service (restart sequencer/batcher/op-geth etc.)

```bash
kurtosis service stop <enclave-name> <service-name>
kurtosis service start <enclave-name> <service-name>
```

## Development

### Development environment

We use [`mise`](https://mise.jdx.dev/) as a dependency manager for these tools.
Once properly installed, `mise` will provide the correct versions for each tool. `mise` does not
replace any other installations of these binaries and will only serve these binaries when you are
working inside of the `optimism-package` directory.

#### Install `mise`

Install `mise` by following the instructions provided on the
[Getting Started page](https://mise.jdx.dev/getting-started.html#_1-install-mise-cli).

#### Install dependencies

```sh
mise install
```

## Contributing

If you have made changes and would like to submit a PR, test locally and make sure to run `lint` on your changes

```bash
kurtosis lint --format .
```

### Testing

#### Unit tests

We are using [`kurtosis-test`](https://github.com/ethereum-optimism/kurtosis-test) to run a set of unit tests against the starlark code:

```bash
# To run all unit tests
kurtosis-test .
```

The tests can be found in `*_test.star` scripts located in the `test` directory.
