# Video NFT Devnet

## Quick start
checkout the repo, get the submodules and run the docker-compose.
```
git submodule update --init --recursive
```
**You may see a message: fatal: No url found for submodule path 'nft-app/src/Opensea-js' in .gitmodules during the above command. You may ignore it for now and proceed with further steps**

Update the docker-compose to lattest vetsion(v1.29.1)  
https://docs.docker.com/compose/install/


Get your nft.storage or textilehub credentials and update marketplace_env.list  

https://nft.storage/
```
NFTSTORAGE_API_KEY=
STORAGE_BACKEND=nftstorage
```
Alternately, if you are using textilehub follow the link and obtain the credentials.

https://docs.textile.io/hub/apis/


```
MARKETPLACE_TEXTILE_AUTH_KEY=""
MARKETPLACE_TEXTILE_AUTH_SECRET=""
MARKETPLACE_TEXTILE_THREAD_ID=""
MARKETPLACE_TEXTILE_BUCKET_ROOT_KEY=""
TORAGE_BACKEND=textile
```

## Run the Video NFT Devnet

### Start ganache testnet
```
docker-compose up -d ganache
```
**Note the ganache testnet satrted as above runs in background.**

### Deploy Video NFT Token Contract
```
docker-compose up nft-contracts-deploy
```

### Deploy Wyvern exchange
```
docker-compose up wyvern-contracts-deploy
```
### Update environment and build arguments
Note down the Video NFT and Wyvern Exchange contract addresses from the previous steps and update the following files.

```
.env
nft-contracts_env.list
wyvern-contracts_env.list
marketplace_env.list
```

See in the relevant sections of the modules in this document for details of environment varaibles in the above files

**Note: If you use the test wallet mnemonic/private key specified in this doc, the generated contract addresses are deterministic, if the contracts are deployed in the order specified in the document due to deterministic nonce update. This will useful for quick testing**
### Build nft-app
```
docker-compose build nft-app
```
### Run the nft-app and marketplace services
```
docker-compose up postgres marketplace_init marketplace nft-app
```
## Open the browser and launch the aft-app.
```
http://localhost:8080/
```

Use the follwing test private-key in Metamask with "Localhost 8545" Netowork
```
Private Key: 0x9312d1929dedd6005d9bfd4b6d51446fa0732958cef08c7592493cd855a2566f
Account: 0x3393faCcA448B53B509306c53D2Ee1980725A0A0
```

Open Lightweight Block Explorer
```
http://localhost:8090/
```

Bringdown the Video NFT Devnet
```
docker-compose down -v
```
**Note: If you are going to run the video-nft-devnetwork again after shutting down, flush your browser cache to remove the stale authentication tokens of of nft-app from the previous session.**

### Check integrity and DRM

* Retrieve the drm data from the token URI
* Use the marketplace docker container to decrypt the drm data. You need to supply the private key corresponding to the public key used for creating DRM data.
* This step outputs the content encryption key.
* Use the content encryption key to decode the encrypted video asset using ffmpeg.

Example: Decrypt DRM data and retrieve content encryption key.
```
docker run --rm -it -e "PRIV_KEY=<Your_Priv_Key>" -e "DRM_KEY=<DRM_DATA_FROM_TOKEN_URI>" registry.videocoin.net/cloud/marketplace:8c7af9248ac36e1e698e143d47782ddf2e4b2d7d /mpek
```
Outputs content deryption key on succesful decryption of DRM.

Decrypt the content using the encryption key obtained in the previous step.
```
ffmpeg  -decryption_key <decryption key> -i encrypted_firstfilm.mp4 -c:v copy -c:a copy test_dec.mp4
```

## VideoNFT Marketplace Components
Docker-compose based devnet that includes the following services:
* NFT-APP(Video NFT Frontend)
* Marketplace(API Backend)
* Postgres(Database for userinfo)
* Ganache(Blockchain Devnet)
* Explorer(Lightweight Block Explorer)
* Token Contracts(Contract deployer)

![Video NFT Devenet](./docs/devnet.drawio.svg)



The above components are configurable. The Video NFT GUI installer obtains the configuration parameters and configures the components. The configuration options of each component are listed below for reference.

## nft-app
VideoCoin NFT Frontend

Build-time arguments. Specify these values in .env file of docker-compose. These arguments are passed as build-time arguments to the nft-app.
Wyvern contract addresses can be obtained after deploying the wyvern contracts using wyvern-contract-deploye as shown in the previous step.
Look at the log using docker logs 
```
docker logs wyvern-contracts-deploy
```
Fill the wyvern exchange contract addresses in .env file with the relevant contract addresses displayed against "custom" blockchain in the log.

```
# marketplace endpint
REACT_APP_BASE_URL=
# hosting url of the nft-app
REACT_APP_NETWORKS=
# Video NFT Contract address
REACT_APP_TOKEN_ADDRESS=

# Wyvern contract addresses
REACT_APP_WYVERN_EXCHANGE=
REACT_APP_WYVERN_PROXY_REGISTRY=
REACT_APP_WYVERN_ATOMICIZER=
REACT_APP_WYVERN_TOKEN_TRANSFER_PROXY=

REACT_APP_WYVERN_DAO=
REACT_APP_WYVERN_TOKEN=
```
## Marketplace
This service provides the backend API for the VideoCoin NFT.

Testing the service docker image:

### Configure Marketplace
Example environment variables file (env.list)
```
AUTH_SECRET=
#NFTSTORAGE_API_KEY=
TEXTILE_AUTH_KEY=
TEXTILE_AUTH_SECRET=
TEXTILE_THREAD_ID=
TEXTILE_BUCKET_ROOT_KEY=

ERC721_CONTRACT_ADDRESS=0xA7b3d9092b87Fd73F98d0B9EA1bE9332deAFAda8
# change the contract address and exchange address to lowercase
ERC721_AUCTION_CONTRACT_ADDRESS=0xda563d7c33d08ec19b094fb253c4cc31cc8bc0e5
# Test key used fro deploying Video NFT contract
ERC721_CONTRACT_KEY={"address":"3393facca448b53b509306c53d2ee1980725a0a0","crypto":{"cipher":"aes-128-ctr","ciphertext":"9e1059dae28e760dbdeb11c1ae80d3a08d4a37615661728faab7f9ec161b898d","cipherparams":{"iv":"2d03e1f35c1119373bf752ed0bba7101"},"kdf":"scrypt","kdfparams":{"dklen":32,"n":262144,"p":1,"r":8,"salt":"d40a13f97124764eafdd460b09bd4d9e66de3abf59c0f41144943ac194446025"},"mac":"d9bcdd1596506ce2e6b4f7d0cec2f8d5c18d027d7850b445682c08631cd60980"},"id":"e829aa11-5ddd-4772-9c45-bb4bbf86aa04","version":3}
ERC721_CONTRACT_KEY_PASS=testkey
BLOCKCHAIN_URL=
BLOCKCHAIN_SCAN_FROM=0

```
**change the exchange address to lowercase**

## Ganache
Test network.
It can removed by providing Web3 provider for Mainnet or any testnet to the marketplace.

Note: We launch ganache with deterministic address option where the same test account addresses are created for every session. The value for the environment variable MARKETPLACE_ERC1155_CONTRACT_ADDRESS supplied for marketplace is also dertermenistic.

Specify the wallter of the contract deployer to generate prefunded keys in the .env
```
# env for ganache 
# test wallet
MNEMONIC="ship arena salad typical truly found start bind insane six wheel vendor"
```


## Token Contracts
Environment variables:
```
# ganache testnet rpc
CUSTOM_RPC="http://173.26.0.100:8545"
# test wallet
MNENONIC="ship arena salad typical truly found start bind insane six wheel vendor"
NFT721_NAME=MyTestNFT
NFT721_SYMBOL=MTNFT
INFURA_KEY=
ERC1155_TOKEN_URI_TEMPLATE="http://localhost:3000/tokens/nft1155/{id}.json"
```

## Wyvern Exchange Contracts
Environment variables:
```
# ganache testnet rpc
CUSTOM_RPC="http://173.26.0.100:8545"
# test wallet\
CUSTPOM_PRIM_KEY="ship arena salad typical truly found start bind insane six wheel vendor"
```
## Explorer
An open source Lightweight Block Explorer
It can show the status of blockchain and can be replaced with any Ethereum blockchain. It is useful if a local test chain such as Ganache is used.

https://github.com/Alethio/ethereum-lite-explorer

## Open Issues:
+ Spurious folder nft-app/src/Opensea-js needs to be removed after initial update of submodules
+ In nft-app, after minting the token, "View On IPFS" button is not showing the correct token uri. 