---
title: Hyperledger Fabric and Composer on Docker Swarm
date: 2018-08-27
author: Alfonso Harita
author_id: aharita
tags:
    - hyperledger-fabric
    - hyperledger-composer
    - blockchain
    - docker
    - docker-swarm
    - distributed-network
    - node
---

We will learn how to deploy Hyperledger Fabric and Hyperledger Composer on a distributed network using physically separated nodes connected via Docker Swarm. This guide is not production ready, as it will need case by case specific configuration.

<!-- more -->

I highly recommend to follow the tutorials on each technology first to get familiar with the commands and concepts to understand what is happening.

* [Hyperledger Fabric Tutorial](https://hyperledger-fabric.readthedocs.io/en/release-1.1/tutorials.html)
* [Hyperledger Composer Tutorial](https://hyperledger.github.io/composer/latest/tutorials/tutorials)
* [Docker Swarm Tutoorial](https://docs.docker.com/engine/swarm/swarm-tutorial/)

## Network Distribution
The network consists on two nodes, which name and purpose are described below.

### hyperledger0
* Certificate Authority
  * Registration of identities, certificates management
* Orderer
  * Orders and package transactions into blocks, commits to ledgers
* Peer0
  * Has the ledger and smart contracts, execute chaincode
* CouchDB
  * You will have a CouchDB server paired for every Peer Node
  * Database server where the ledger is stored

### hyperledger1
* Peer1
* CouchDB

## Basic Setup and Dependencies
Unless otherwise specified, run the commands on all the nodes. Remember to follow the instructions when installing packages, as you might need to log out and log in for some commands to work.

### Firewall
Make sure that the appropriate ports are open for UDP and TCP. You need to open the ports for Hyperledger Fabric and Docker Swarm. The ports are particular to your own installation and configuration.

### Utils

```bash
sudo apt-get update
sudo apt-get install build-essential
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
sudo apt-get install python
```

### Docker

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install docker-ce
```

If you would like to manage Docker as a non-root user, follow [these steps](https://docs.docker.com/install/linux/linux-postinstall/)

### Node

```bash
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
nvm install --lts
```

### Hyperledger Fabric
Make sure the version you are installing is compatible with Hyperledger Composer

```bash
curl -sSL http://bit.ly/2ysbOFE | bash -s 1.1.0
export PATH=~/fabric-samples/bin:$PATH
```

### Hyperledger Composer
Make sure the version you are installing is compatible with Hyperledger Fabric

```bash
npm install -g composer-cli@0.20
npm install -g composer-rest-server@0.20
npm install -g generator-hyperledger-composer@0.20
npm install -g yo
npm install -g composer-playground@0.20
```

## Configuration on the Nodes
Clone the following repository, most of the commands are to be run on the root folder of that repository.

```bash
git clone git@github.com:aharita/Build-Multi-Host-Network-Hyperledger.git
cd Build-Multi-Host-Network-Hyperledger
```

### Artifacts
Let's generate the artifacts/crypto-config that all the nodes are going to share. You can run this only on hyperledger0

```bash
./bmhn.sh
```

We need to copy the same crypto-config to hyperledger1, you can use these commands as an example,

```bash
zip -r crypto-config.zip cripto-config
scp crypto-config.zip user@hyperledger1:/home/user/Build-Multi-Host-Network-Hyperledger/crypto-config.zip
```

Log in to hyperledger1. Make sure you also cloned the repository here

```bash
cd Build-Multi-Host-Network-Hyperledger
unzip crypto-config.zip
```

### Docker Swarm Connection
Once both hyperledger0 and hyperleger1 have the same crypto-config folder, we're ready to connect them using docker swarm. Pay attention to the instructions after each command.

```bash
# On hyperledger0
docker swarm init
docker swarm join-token manager
docker network create --attachable --driver overlay sample-network

# On hyperledger1, copy the output from the previous command, something like this:
docker swarm join --token {YOUR-UNIQUE-TOKEN} hyperledger0:2377
```

## hyperledger0 components
Don't forget to run the commands inside the `Build-Multi-Host-Network-Hyperledger` folder. You are going to run this on a new tab for the same hyperledger0 server.

### CA - Certificate Authority
There is one file key name that you need to copy and replace on the command, you can check that by running `ls -la ~/Build-Multi-Host-Network-Hyperledger/crypto-config/peerOrganizations/org1.example.com/ca`

```bash
docker run --rm -it --network="sample-network" --name ca.example.com -p 7054:7054 \
-e FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server -e FABRIC_CA_SERVER_CA_NAME=ca.example.com \
-e FABRIC_CA_SERVER_CA_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.org1.example.com-cert.pem \
-e FABRIC_CA_SERVER_CA_KEYFILE=/etc/hyperledger/fabric-ca-server-config/{YOUR-FILE-KEY-NAME_sk} \
-v $(pwd)/crypto-config/peerOrganizations/org1.example.com/ca/:/etc/hyperledger/fabric-ca-server-config \
-e CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=sample-network hyperledger/fabric-ca:1.1.0 \
sh -c 'fabric-ca-server start -b admin:adminpw -d'
```

### Orderer

```bash
docker run --rm -it --network="sample-network" --name orderer.example.com -p 7050:7050 \
-e ORDERER_GENERAL_LOGLEVEL=debug -e ORDERER_GENERAL_LISTENADDRESS=0.0.0.0 \
-e ORDERER_GENERAL_LISTENPORT=7050 -e ORDERER_GENERAL_GENESISMETHOD=file \
-e ORDERER_GENERAL_GENESISFILE=/var/hyperledger/orderer/orderer.genesis.block \
-e ORDERER_GENERAL_LOCALMSPID=OrdererMSP -e ORDERER_GENERAL_LOCALMSPDIR=/var/hyperledger/orderer/msp \
-e ORDERER_GENERAL_TLS_ENABLED=false -e CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=sample-network \
-v $(pwd)/channel-artifacts/genesis.block:/var/hyperledger/orderer/orderer.genesis.block \
-v $(pwd)/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp:/var/hyperledger/orderer/msp \
-w /opt/gopath/src/github.com/hyperledger/fabric hyperledger/fabric-orderer:1.1.0 orderer
```

### CouchDB

```bash
docker run --rm -it --network="sample-network" --name couchdb0 -p 5984:5984 -e COUCHDB_USER= \
-e COUCHDB_PASSWORD= -e CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=sample-network hyperledger/fabric-couchdb:0.4.10
```

### Peer0

```bash
docker run --rm -it --link orderer.example.com:orderer.example.com --network="sample-network" \
--name peer0.org1.example.com -p 8051:7051 -p 8053:7053 -e CORE_LEDGER_STATE_STATEDATABASE=CouchDB \
-e CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb0:5984 \
-e CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME= -e CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD= \
-e CORE_PEER_ADDRESSAUTODETECT=true -e CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock \
-e CORE_LOGGING_LEVEL=DEBUG -e CORE_PEER_NETWORKID=peer0.org1.example.com -e CORE_NEXT=true \
-e CORE_PEER_ENDORSER_ENABLED=true -e CORE_PEER_ID=peer0.org1.example.com \
-e CORE_PEER_PROFILE_ENABLED=true -e CORE_PEER_COMMITTER_LEDGER_ORDERER=orderer.example.com:7050 \
-e CORE_PEER_GOSSIP_IGNORESECURITY=true -e CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=sample-network \
-e CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org1.example.com:7051 -e CORE_PEER_TLS_ENABLED=false \
-e CORE_PEER_GOSSIP_USELEADERELECTION=false -e CORE_PEER_GOSSIP_ORGLEADER=true \
-e CORE_PEER_LOCALMSPID=Org1MSP -v /var/run/:/host/var/run/ \
-v $(pwd)/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp:/etc/hyperledger/fabric/msp \
-w /opt/gopath/src/github.com/hyperledger/fabric/peer hyperledger/fabric-peer:1.1.0 peer node start
```

## hyperledger1 components
Don't forget to run the commands inside the `Build-Multi-Host-Network-Hyperledger` folder. You are going to run this on a new tab for the same hyperledger1 server.

### CouchDB

```bash
docker run --rm -it --network="sample-network" --name couchdb1 -p 6984:5984 -e COUCHDB_USER= \
-e COUCHDB_PASSWORD= -e CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=sample-network hyperledger/fabric-couchdb:0.4.10
```

### Peer1

```bash
docker run --rm -it --network="sample-network" --link orderer.example.com:orderer.example.com \
--link peer0.org1.example.com:peer0.org1.example.com --name peer1.org1.example.com -p 9051:7051 \
-p 9053:7053 -e CORE_LEDGER_STATE_STATEDATABASE=CouchDB \
-e CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb1:5984 \
-e CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME= -e CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD= \
-e CORE_PEER_ADDRESSAUTODETECT=true -e CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock \
-e CORE_LOGGING_LEVEL=DEBUG -e CORE_PEER_NETWORKID=peer1.org1.example.com -e CORE_NEXT=true \
-e CORE_PEER_ENDORSER_ENABLED=true -e CORE_PEER_ID=peer1.org1.example.com \
-e CORE_PEER_PROFILE_ENABLED=true -e CORE_PEER_COMMITTER_LEDGER_ORDERER=orderer.example.com:7050 \
-e CORE_PEER_GOSSIP_ORGLEADER=true -e CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer1.org1.example.com:7051 \
-e CORE_PEER_GOSSIP_IGNORESECURITY=true -e CORE_PEER_LOCALMSPID=Org1MSP \
-e CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=sample-network \
-e CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org1.example.com:7051 \
-e CORE_PEER_GOSSIP_USELEADERELECTION=false -e CORE_PEER_TLS_ENABLED=false \
-v /var/run/:/host/var/run/ \
-v $(pwd)/crypto-config/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/msp:/etc/hyperledger/fabric/msp \
-w /opt/gopath/src/github.com/hyperledger/fabric/peer hyperledger/fabric-peer:1.1.0 peer node start
```

## Testing Network with CLI
Execute the script and verify that the installation is good. It will create a channel `mychannel` that we will use for our Hyperledger Composer installation. Peer0 and Peer1 will join the created channel.

```bash
docker run --rm -it --network="sample-network" --name cli \
--link orderer.example.com:orderer.example.com \
--link peer0.org1.example.com:peer0.org1.example.com \
--link peer1.org1.example.com:peer1.org1.example.com -p 12051:7051 -p 12053:7053 \
-e GOPATH=/opt/gopath -e CORE_PEER_LOCALMSPID=Org1MSP -e CORE_PEER_TLS_ENABLED=false \
-e CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock -e CORE_LOGGING_LEVEL=DEBUG \
-e CORE_PEER_ID=cli -e CORE_PEER_ADDRESS=peer0.org1.example.com:7051 -e CORE_PEER_NETWORKID=cli \
-e CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp \
-e CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=sample-network -v /var/run/:/host/var/run/ \
-v $(pwd)/chaincode/:/opt/gopath/src/github.com/hyperledger/fabric/examples/chaincode/go \
-v $(pwd)/crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ \
-v $(pwd)/scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/ \
-v $(pwd)/channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts \
-w /opt/gopath/src/github.com/hyperledger/fabric/peer hyperledger/fabric-tools:1.1.0 /bin/bash -c './scripts/script.sh'
```

## Hyperledger Composer
Finally, we're ready for installing a Hyperledger Composer application on our running Hyperledger Fabric network

Run on hyperledger0

```bash
# Follow instructions, you can name it tutorial-network
# Make sure to select a populated sample network, for testing
yo hyperledger-composer:businessnetwork
cd tutorial-network

# This command will package our blockchain sample application
composer archive create -t dir -n .
```

### Connection
We need to create a `connection.json` file that contains the configuration for all the servers (CA, Orderer, Peers) to be reachable by our Hyperledger Composer identity card and application.

Please update the values as necessary.

```json
{
  "name": "hlfv1",
  "x-type": "hlfv1",
  "x-commitTimeout": 300,
  "version": "1.0.0",
  "client": {
    "organization": "Org1",
    "connection": {
      "timeout": {
        "peer": {
          "endorser": "300",
          "eventHub": "300",
          "eventReg": "300"
        },
        "orderer": "300"
      }
    }
  },
  "channels": {
    "mychannel": {
      "orderers": ["orderer.example.com"],
      "peers": {
        "peer0.org1.example.com": {},
        "peer1.org1.example.com": {}
      }
    }
  },
  "organizations": {
    "Org1": {
      "mspid": "Org1MSP",
      "peers": ["peer0.org1.example.com", "peer1.org1.example.com"],
      "certificateAuthorities": ["ca.example.com"]
    }
  },
  "orderers": {
    "orderer.example.com": {
      "url": "grpc://hyperledger0:7050"
    }
  },
  "peers": {
    "peer0.org1.example.com": {
      "url": "grpc://hyperledger0:8051",
      "eventUrl": "grpc://hyperledger0:8053"
    },
    "peer1.org1.example.com": {
      "url": "grpc://hyperledger1:9051",
      "eventUrl": "grpc://hyperledger1:9053"
    }
  },
  "certificateAuthorities": {
    "ca.example.com": {
      "url": "http://hyperledger0:7054",
      "caName": "ca.example.com"
    }
  }
}
```

### Installing Business Network
Replace the file key name, similar to what we did for the Certificate Authority server

```bash
composer card create -p connection.json -u PeerAdmin \
-c ~/Build-Multi-Host-Network-Hyperledger/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts/Admin@org1.example.com-cert.pem \
-k ~/Build-Multi-Host-Network-Hyperledger/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore/{YOUR-FILE-KEY-NAME} \
-r PeerAdmin -r ChannelAdmin

composer card import -f PeerAdmin@hlfv1.card
composer network install -c PeerAdmin@hlfv1 -a tutorial-network@0.0.1.bna
composer network start --networkName tutorial-network --networkVersion 0.0.1 -A admin -S adminpw -c PeerAdmin@hlfv1
```

## Notes
There's still a lot of improvements to do, such as:

* Use `docker-composer` instead of running the `docker run` command
* Handle the case when Peer3 wants to join the Hyperledger Fabric Network
* Use single nodes for CA, Orderer and Peer1, but you get the idea

Based on the article https://medium.com/@wahabjawed/hyperledger-fabric-on-multiple-hosts-a33b08ef24f
