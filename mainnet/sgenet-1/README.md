# sgenet-1

> This chain is a public-mainnet.

1st MainNet for the SGE Network application.

## Hardware Requirements

- **Minimal**
  - 1 GB RAM
  - 25 GB HDD
  - 1.4 GHz CPU
- **Recommended**
  - 2 GB RAM
  - 100 GB HDD
  - 2.0 GHz x2 CPU

---

## Operating System

- Linux/Windows/MacOS(x86)
- **Recommended**
  - Linux(x86_64)
  - Ubuntu 20.04 LTS

---

## Network

- Sentry node architecture and secure firewall configurations are highly recommended, [ref](https://forum.cosmos.network/t/sentry-node-architecture-overview/454)
- Port & Firewall settings (Ports Allowed)
  - RPC port: 26657
  - P2P port: 26656
  - GRPC port: 9090

---

## Installation Steps

> Prerequisite: go1.19 required [ref](https://golang.org/doc/install) (build using higher versions cause consensus error)

```shell
sudo snap install go --channel=1.19/stable --classic
```

> Prerequisite: git [ref](https://github.com/git/git)

```shell
sudo apt install -y git gcc make
```

> Prerequisite: Set environment variables

```shell
sudo nano $HOME/.profile
# Add the following two lines at the end of the file
GOPATH=$HOME/go
PATH=$GOPATH/bin:$PATH
# Save the file and exit the editor
source $HOME/.profile
# Now you should be able to see your variables like this:
echo $GOPATH
/home/[your_username]/go
echo $PATH
/home/[your_username]/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```

> Recommended requirement: Increase 'number of open files' limit

```shell
sudo nano /etc/security/limits.conf
# Before the end of the file, add:
[your_username] soft nofile 4096
# Then reboot the instance for it to take effect and check with:
ulimit -Sn
```

> Optional requirement: GNU make [ref](https://www.gnu.org/software/make/manual/html_node/index.html)

- Clone the git repository and Network

```shell
git clone https://github.com/sge-network/sge
git clone https://github.com/sge-network/networks
```

- Checkout release tag

```shell
cd sge
git fetch --tags
git checkout v1.1.0
```

- Install

```shell
cd sge
go mod tidy
make install
```

### Generate keys

`sged keys add [key_name]`

or

`sged keys add [key_name] --recover` to regenerate keys with your [BIP39](https://github.com/bitcoin/bips/tree/master/bip-0039) mnemonic

---

## Node start and validator setup

### Prior to genesis creation and network launch

- [Install](#installation-steps) the SGE Network application
- Initialize node

```shell
sged init {{NODE_NAME}} --chain-id sgenet-1
```

- Create a new key

```shell
sged keys add <keyName>
```

- Add a genesis account with `10000000usge tokens`

```shell
sged add-genesis-account {{KEY_NAME}} 10000000usge
```

- Make a genesis transaction to become a validator

```shell
sged gentx \
  [account_key_name] \
  8000000usge \
  --commission-max-change-rate 0.01 \
  --commission-max-rate 0.2 \
  --commission-rate 0.05 \
  --min-self-delegation 1 \
  --details [optional-details] \
  --identity [optional-identity] \
  --security-contact "[optional-security@example.com]" \
  --website [optional.web.page.com] \
  --moniker [node_moniker] \
  --chain-id sgenet-1
```

- Copy the contents of `${HOME}/.sge/config/gentx/gentx-XXXXXXXX.json`
- Fork the [network repository](https://github.com/sge-network/networks)
- Create a file `gentx-{{VALIDATOR_NAME}}.json` under the `mainnet/sgenet-1/gentxs` folder in the newly created branch, Paste the copied text into the file (note: find reference file `gentx-examplexxxxxxxx.json` in the same folder)
- Run `sged tendermint show-node-id` and copy your nodeID
- Run `ifconfig` or `curl ipinfo.io/ip` and copy your publicly reachable IP address
- Create a file `peers-{{VALIDATOR_NAME}}.json` under the `mainnet/sgenet-1/peers` folder in the new branch, Paste the copied text from the last two steps into the file (note: find reference file `peers-examplexxxxxxxx.json` in the same folder)
- Create a Pull Request to the `master` branch of the [network repository](https://github.com/sge-network/networks)
  > **NOTE:** the Pull Request will be merged by the maintainers to confirm the inclusion of the validator at the genesis. The final genesis file will be published under the file `mainnet/sgenet-1/genesis.json`.
- Once the submission process has closed and the genesis file has been created, replace the contents of your `${HOME}/.sge/config/genesis.json` with that of `mainnet/sgenet-1/genesis.json`
- Add the required `persistent_peers` or `seeds` in `${HOME}/.sge/config/config.toml` from `mainnet/sgenet-1/peers-nodes.txt`
- Start node

```shell
sged start --minimum-gas-prices [desired-gas-price(ex. 0.001usge)]
```

## Genesis Time

The genesis transactions should be sent before 0400HRS UTC on 25th September 2023 and the same will be used to publish the `genesis.json` at 1200HRS UTC on 6th September 2023

<!-- > Submitting Gentx is now closed. Genesis has been published and block generation has started -->

---

## After genesis creation and network launch

### Step 1: Start a full node

- [Install](#installation-steps) the SGE Network application
- Initialize node

```shell
sged init {{NODE_NAME}}
```

- Replace the contents of your `${HOME}/.sge/config/genesis.json` with that of `mainnet/sgenet-1/genesis.json` from the `master` branch of [network repository](https://github.com/sge-network/networks)

```shell
curl https://github.com/sge-network/blob/master/networks/mainnet/sgenet-1/genesis.json > $HOME/.sge/config/genesis.json
```

- Add `persistent_peers` or `seeds` in `${HOME}/.sge/config/config.toml` from `mainnet/sgenet-1/peers.txt` from the `master` branch of [network repository](https://github.com/sge-network/networks/blob/master/mainnet/sgenet-1/peers.txt)
- Start node

```shell
sged start --minimum-gas-prices [desired-gas-price(ex. 0.001usge)]
```

> Note: if you are only planning to run a full node, you can stop here

### Step 2: Create a validator

- Acquire SGE tokens
- Wait for your full node to catch up to the latest block (compare to the [explorer]())
- Run `sged tendermint show-validator` and copy your consensus public key
- Send a create-validator transaction

```shell
sged tx staking create-validator \
  --amount 500000000usge \
  --commission-max-change-rate 0.01 \
  --commission-max-rate 0.2 \
  --commission-rate 0.1 \
  --from [account_key_name] \
  --fees 400000usge \
  --min-self-delegation 1 \
  --moniker [validator_moniker] \
  --pubkey $(sged tendermint show-validator) \
  --chain-id sgenet-1 \
  -y
```

---

## Persistent Peers

The `persistent_peers` needs a comma-separated list of trusted peers on the network, you can acquire it from the [peers-nodes.txt](https://github.com/sge-network/networks/blob/master/mainnet/sgenet-1/peer-nodes.txt) for example:

```text
4980b478f91de9be0564a547779e5c6cb07eb995@3.239.15.80:26656,0e7042be1b77707aaf0597bb804da90d3a606c08@3.88.40.53:26656
```
## Seed Nodes

``` "seeds": [
      {
        "id": "babc3f3f7804933265ec9c40ad94f4da8e9e0017",
        "address": "seed.rhinostake.com:17756",
        "provider": "rhinostake.com"
      },
      {
        "id": "a973f744ec9b00cd387f62fc8d69ae1d753c060e",
        "address": "seed.sge.cros-nest.com:26656",
        "provider": "cros-nest.com"
      },
      {
        "id": "20e1000e88125698264454a884812746c2eb4807",
        "address": "seeds.lavenderfive.com:17756",
        "provider": "lavenderfive.com"
      },
      {
        "id": "df949a46ae6529ae1e09b034b49716468d5cc7e9",
        "address": "seeds.stakerhouse.com:11156",
        "provider": "stakerhouse.com"
      },
      {
        "id": "af9d9bd15ca597eb77dab73c56b0ae51bafcbb28",
        "address": "sge.ramuchi.tech:16656",
        "provider": "ramuchi.tech"
      },
      {
        "id": "6a727128f427d166d90a1185c7965b178235aaee",
        "address": "rpc.sge.nodestake.top:666",
        "provider": "nodestake.top"
      }
    ]
```
## Blockexplorers
```
 [
    {
      "kind": "nodeist.net",
      "url": "https://exp.nodeist.net/Sge",
      "tx_page": "https://exp.nodeist.net/Sge/tx/${txHash}"
    },
    {
      "kind": "nodestake.top",
      "url": "https://explorer.nodestake.top/sge",
      "tx_page": "https://explorer.nodestake.top/sge/tx/${txHash}"
    },
    {
      "kind": "stakerhouse",
      "url": "https://cosmotracker.com/sge",
      "tx_page": "https://cosmotracker.com/sge/tx/${txHash}"
    }
  ]
```
## RPC EndPoints
```
 [
      {
        "address": "https://sge-api.lavenderfive.com/",
        "provider": "Lavenderfive"
      },
      {
        "address": "https://sge-api.polkachu.com/",
        "provider": "Polkachu"
      },
      {
        "address": "https://api.sge.nodestake.top/",
        "provider": "Nostake"
      },
      {
        "address": "https://api-sge.nodeist.net/",
        "provider": "Nodeist"
      }
    ]
```
## Rest EndPoints
```
 [
      {
        "address": "https://sge-api.lavenderfive.com/",
        "provider": "Lavenderfive"
      },
      {
        "address": "https://sge-api.polkachu.com/",
        "provider": "Polkachu"
      },
      {
        "address": "https://api.sge.nodestake.top/",
        "provider": "Nostake"
      },
      {
        "address": "https://api-sge.nodeist.net/",
        "provider": "Nodeist"
      }
    ]
```

## grpc EndPoints
```
 [
      {
        "address": "https://sge-api.lavenderfive.com/",
        "provider": "Lavenderfive"
      },
      {
        "address": "https://api.sge.nodestake.top/",
        "provider": "Nostake"
      },
      {
        "address": "https://api-sge.nodeist.net/",
        "provider": "Nodeist"
      }
    ]
```

## Version

This chain is currently running on SGE [v1.1.0](https://github.com/sge-network/sge/releases/tag/v1.1.0)
Commit Hash: [031b1b16abcc8025b96d3df260f31819b19c68ed](https://github.com/sge-network/sge/commit/031b1b16abcc8025b96d3df260f31819b19c68ed)

## Binary

The binary can be downloaded from [here](https://github.com/sge-network/sge/releases/tag/v1.1.0)
