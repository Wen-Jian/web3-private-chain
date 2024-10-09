# Private chain with Proof Of Authority consensus
[TOC]
## Initialize

- miner node

    - use the following command to init a new account for sealing block

            geth account new --datadir ./poa_private_net/miner_node

    - replace password.txt content with the password you entered

    - replace the extract data with newly generated account
        ex:
        let's say we have newly generated account 0x6bc5e3c1f6b57530d7118193518d171b5996772e. change the extract data for:
        > 0x00000000000000000000000000000000000000000000000000000000000000006BC5e3c1F6B57530D7118193518d171b5996772E0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000

    - and then initialize the net via the command below
        ```shell=
        geth --datadir ./poa_private_net/miner_node init ./poa_private_net/genesis.json
        ```
- http rpc node
    - initialize http rpc node
        ```shell
        geth init --datadir ./poa_private_net/http_rpc_node ./poa_private_net/genesis.json
        ```


## Launch the private chain
- miner node 
    * launch private chain miner node and start mining
        ```shell=
        geth --unlock '685D0608bf1f8746FeAb9823BdEf9Eb870CbbE0E' --password ./poa_private_net/miner_node/password.txt --mine --miner.etherbase '685D0608bf1f8746FeAb9823BdEf9Eb870CbbE0E' --datadir ./poa_private_net/miner_node --networkid 313 --authrpc.port 8552 --bootnodes enode://e843afd3e6f67753291f1e842d0623ef7e3584ef7ec143b77d616deaf1ed62d6fd571b3dbd64b3cbe5a87c1d8edd456e11d6c3cbcd095140156f1bb948827262@127.0.0.1:30304 --allow-insecure-unlock console
        ```

    - add peer node to miner node(launch target node first)
        - get enode id
            ```shell
            cd ./poa_private_net/http_rpc_node/geth
            ```
            ```shell
            enodeid=`bootnode -nodekey nodekey -writeaddress`
            ```
            ```shell
            echo "enode://$enodeid@127.0.0.1:30304"
            ```
        - access miner node via console, and then add the rpc node as peer node
            ```shell
            admin.addPeer("enodeid") # the generated enode id 
            ```



- http rpc node
    - launch http rpc node
        ```shell
        geth --datadir ./poa_private_net/http_rpc_node --networkid 313 --port 30304 --http --http.addr "localhost" --http.port 10003 --http.corsdomain "*"
        ```

# Private chain with Proof Of Stake consensus
## Prepare Cli tools
### clone prysm project
```shell
git clone https://github.com/prysmaticlabs/prysm 
```

### build tools
- beacon-chain
    ```shell
    go build -o=../beacon-chain ./cmd/beacon-chain
    ```
- validator
    ```shell
    go build -o=../validator ./cmd/validator
    ```
- prysmctl
    ```shell 
    go build -o=../prysmctl ./cmd/prysmctl
    ```
### generate JWT secret key
```shell
beacon-chain generate-auth-secret
```

## Create execution node & Launch the execution node
same as miner node of POA chain

- Initialize execution node

```shell
geth account new --datadir ./pos_private_net/execution_node
```

```shell
geth --datadir ./pos_private_net/execution_node init ./pos_private_net/genesis.json
```

- Launch execution node
```shell
geth --unlock '0x54d95b6fB8f0F643C24860DE07D07443aedD544d' --password ./pos_private_net/execution_node/password.txt --mine --miner.etherbase '0x54d95b6fB8f0F643C24860DE07D07443aedD544d' --datadir ./pos_private_net/execution_node --networkid 313 --authrpc.port 8551 --allow-insecure-unlock --authrpc.jwtsecret=./pos_private_net/jwt.hex --http --http.addr "localhost" --http.port 10003 --http.corsdomain "*"  console
```

## Generate genesis state file 
```shell
prysmctl testnet generate-genesis --num-validators=5 --output-json=./pos_private_net/genesis_state.json --execution-endpoint=http://localhost:8551 --chain-config-file=./pos_private_net/pos_config.yaml --geth-genesis-json-in=./pos_private_net/genesis.json --geth-genesis-json-out=./pos_private_net/genesit_out.json --output-ssz=./pos_private_net/genesis_state.ssz
```

## Launch validator
```shell
validator --datadir=./pos_private_net/validator_node --accept-terms-of-use --wallet-dir=./pos_private_net/validator_node --interop-num-validators=64 --chain-config-file=./pos_private_net/pos_config.yaml --config-file=./pos_private_net/pos_config.yaml
```

## Launch Beacon Chain
```shell
beacon-chain --execution-endpoint=http://localhost:8551 --jwt-secret=./pos_private_net/jwt.hex --chain-id=313 --deposit-contract=0x4242424242424242424242424242424242424242 --datadir=./pos_private_net/beacon_node --accept-terms-of-use --genesis-state ./pos_private_net/genesis_state.ssz --chain-config-file=./pos_private_net/pos_config.yaml
```

path=/Users/user/Library/Ethereum/geth


beacon-chain --execution-endpoint=http://localhost:8551 --mainnet --jwt-secret=./pos_private_net/jwt.hex  --checkpoint-sync-url=https://beaconstate.info --genesis-beacon-api-url=https://beaconstate.info