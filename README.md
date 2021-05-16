# Cardano Stake Pool Operation Cheat Sheet

Full explanation on how to install-setup-run-use-monitor a Cardano Stake Pool (Debian/Ubuntu)

## Memento

### 1 ADA = 1,000,000 Lovelaces

### Check Node Syncing : 
```
cardano-cli query tip --mainnet
cardano-cli query tip --testnet-magic 1097911063
````
### Check UtXO State (by key address) : 

```
cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 1097911063
cardano-cli query utxo --address $(cat payment.addr) --mainnet 1097911063
```

### Protocole File
```
cardano-cli query protocol-parameters --mainnet --out-file protocol.json
```

### Firewall Configuration


## Full Instalation + Run

### Update

```
sudo apt-get update -y
sudo apt-get install build-essential pkg-config libffi-dev libgmp-dev -y
sudo apt-get install libssl-dev libtinfo-dev libsystemd-dev zlib1g-dev -y
sudo apt-get install make g++ tmux git jq wget libncursesw5 libtool autoconf -y 
```

### Cabal

```
wget https://downloads.haskell.org/~cabal/cabal-install-3.2.0.0/cabal-install-3.2.0.0-x86_64-unknown-linux.tar.xz
tar -xf cabal-install-3.2.0.0-x86_64-unknown-linux.tar.xz
rm cabal-install-3.2.0.0-x86_64-unknown-linux.tar.xz cabal.sig
mkdir -p ~/.local/bin
mv cabal ~/.local/bin/
echo $PATH
cd
nano .bashrc
put at the end
export PATH="~/.local/bin:$PATH"
then
source .bashrc
cabal update
cabal --version
```

### GHC

```
wget https://downloads.haskell.org/~ghc/8.10.2/ghc-8.10.2-x86_64-deb9-linux.tar.xz
tar -xf ghc-8.10.2-x86_64-deb9-linux.tar.xz
rm ghc-8.10.2-x86_64-deb9-linux.tar.xz
cd ghc-8.10.2
./configure
sudo make install
cd ..
ghc --version

```

### Libsodium

```
export LD_LIBRARY_PATH="/usr/local/lib:$LD_LIBRARY_PATH"
export PKG_CONFIG_PATH="/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH"

git clone https://github.com/input-output-hk/libsodium
cd libsodium
git checkout 66f017f1
./autogen.sh
./configure
make
sudo make install

```

### Cardano-node

```
cd
git clone https://github.com/input-output-hk/cardano-node.git
ls cardano-node
cd cardano-node
git fetch --all --tags
git tag
git checkout tags/<LATEST_VERSION>
cabal clean
cabal update
cabal build all
cp -p dist-newstyle/build/x86_64-linux/ghc-8.10.2/cardano-node-<VERSION>/x/cardano-node/build/cardano-node/cardano-node ~/.local/bin/
cp -p dist-newstyle/build/x86_64-linux/ghc-8.10.2/cardano-cli-<VERSION>/x/cardano-cli/build/cardano-cli/cardano-cli ~/.local/bin/
cardano-cli --version

```

#### Backup (Optionnal)

```
cd ~/.local/bin
mv cardano-cli cardano-cli-backup
mv cardano-node cardano-node-backup
cp -p dist-newstyle/build/x86_64-linux/ghc-8.10.2/cardano-node-<NEW VERSION>/x/cardano-node/build/cardano-node/cardano-node ~/.local/bin/
cp -p dist-newstyle/build/x86_64-linux/ghc-8.10.2/cardano-cli-<NEW VERSION>/x/cardano-cli/build/cardano-cli/cardano-cli ~/.local/bin/

```

### Configuration File

#### Testnet

```
cd
mkdir relay
cd relay
wget https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/testnet-config.json
wget https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/testnet-shelley-genesis.json
wget https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/testnet-byron-genesis.json
wget https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/testnet-topology.json

```

#### Mainnet

```
cd
mkdir relay
cd relay
wget https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/mainnet-config.json
wget https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/mainnet-shelley-genesis.json
wget https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/mainnet-byron-genesis.json
wget https://hydra.iohk.io/job/Cardano/cardano-node/cardano-deployment/latest-finished/download/1/mainnnet-topology.json
```

### Run a node (Passive/Active)

x.x.x.x = Your IP public Address 
You should be in the same directory as config file



#### Testnet

```
cardano-node run --topology testnet topology.json --database-path db --socket-path db/node.socket --host-addr x.x.x.x --port 3001 --config testnet-config.json
```

#### Mainnet

```
cardano-node run --topology mainnet-topology.json --database-path db --socket-path db/node.socket --host-addr x.x.x.x --port 3001 --config mainnet-config.json
```

#### Active run

Possible after completing [Stake Pool Registration](#stake)

```
cardano-node run --topology mainnet-topology.json --database-path /db --socket-path /db/node.socket --host-addr <PUBLIC IP> 
--port <PORT> \
--config mainnet-config.json \
--shelley-kes-key kes.skey \
--shelley-vrf-key vrf.skey \
--shelley-operational-certificate node.cert
```

## Initialisation & Registering

### Generating Key

#### Payment Key

```
cardano-cli address key-gen --verification-key-file payment.vkey --signing-key-file payment.skey`
```

#### Stake Key

```
cardano-cli stake-address key-gen --verification-key-file stake.vkey --signing-key-file stake.skey
cardano-cli stake-address build --stake-verification-key-file stake.vkey --out-file stake.addr --mainnet
cardano-cli address build --payment-verification-key-file payment.vkey --stake-verification-key-file stake.vkey --out-file paymentwithstake.addr --mainnet
```

### Registering

#### Stake Adress
```
cardano-cli stake-address registration-certificate --stake-verification-key-file stake.vkey --out-file stake.cert`
cardano-cli transaction build-raw --tx-in <b64ae44e1195b04663ab863b62337e626c65b0c9855a9fbb9ef4458f81a6f5ee>#<TxIx> --tx-out $(cat paymentwithstake.addr)+0 --ttl 0 --fee 0 --out-file tx.raw --certificate-file stake.cert
cardano-cli transaction calculate-min-fee --tx-body-file tx.draft --tx-in-count 1 --tx-out-count 1 --witness-count 2 --byron-witness-count 0 --mainnet --protocol-params-file protocol.json -> min-fee
expr UtxOBalance - min_Fee - KeyDeposit -> Result
cardano-cli transaction build-raw --tx-in <b64ae44e1195b04663ab863b62337e626c65b0c9855a9fbb9ef4458f81a6f5ee>#<TxIx> --tx-out $(cat paymentwithstake.addr)+<Result> --invalid-hereafter 987654 --fee <min_fee> --out-file tx.raw --certificate-file stake.cert
cardano-cli transaction sign --tx-body-file tx.raw --signing-key-file payment.skey --signing-key-file stake.skey --mainnet --out-file tx.signed 
cardano-cli transaction submit --tx-file tx.signed --mainnet
```

#### <a name="stake">Stake Pool</a>

```
mkdir pool-keys
cd pool-keys
cardano-cli node key-gen --cold-verification-key-file cold.vkey --cold-signing-key-file cold.skey --operational-certificate-issue-counter-file cold.counter
cardano-cli node key-gen-VRF --verification-key-file vrf.vkey --signing-key-file vrf.skey
cardano-cli node key-gen-KES --verification-key-file kes.vkey --signing-key-file kes.skey
cat mainnet-shelley-genesis.json | grep KESPeriod -> KES
Compute KESPeriod = slot / KES
cardano-cli node issue-op-cert --kes-verification-key-file kes.vkey --cold-signing-key-file cold.skey --operational-certificate-issue-counter cold.counter --kes-period <KESPeriod> --out-file node.cert
```

Move the cold key into a secure storage (usb etc)
```
scp -rv -P<SSH PORT> -i ~/.ssh/<SSH_PRIVATE_KEY> ~/pool-keys USER@<PUBLIC_IP>:~/
```

## Configuration

#### Protocol Parameter


## Operating

### Transaction

First you need know your 2 involving addresses, destination and sender : The transaction will be between payment.addr (sender) sending 100 ADA (100,000,000 lovelaces) to payment.addr2 (destination).

Be sure that you already have your protocol parameter file.

(use --mainnet or --testnet-magic 1097911063 according to your case)

#### Creating payment address

```
cardano-cli address key-gen --verification-key-file <addressName>.vkey --signing-key-file <addressName>.skey
cardano-cli address build --payment-verification-key-file <addressName>.vkey --out-file <addressName>.addr --testnet-magic 1097911063
```

#### Time-To-Live

```
cardano-cli query tip --testnet-magic 1097911063
```
Get the slot Number of the current block and add your time to live (1200 = 20 minutes recommendend)
keep your result close as we will need it to build our transaction (i.e slotNo = 1000 TTl = 2200)

TTL = ```expr <slotNumber> + <TimeAdded>```

#### Draft

```
cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 1097911063
```

Get the information that comes from the output of your utxo : TxHash/TxIx/Lovelace

```
cardano-cli transaction build-raw --tx-in <TxHash>#<TxIx> --tx-out $(cat payment2.addr)+100000000 --tx-out $(cat payment.addr)+0 --ttl 0 --fee 0 --out-file tx.raw
```

#### Minimum fee

```
cardano-cli transaction calculate-min-fee --tx-body-file tx.raw --tx-in-count 1 --tx-out-count 2 --witness-count 1 --byron-witness-count 0 --testnet-magic 1097911063 --protocol-params-file protocol.json
````

Output will be the minimum fee in lovelaces, we have all information to build the transaction now 

change = ```expr <Lovelace> - <AmountToSend> - <MinFee>```

#### Build

```
cardano-cli transaction build-raw --tx-in <TxHash>#<TxIx> --tx-out $(cat payment2.addr)+<AmountToSend> --tx-out $(cat payment.addr)+<change> --ttl <TTL> --fee <MinFee> --out-file tx.raw
cardano-cli transaction sign --tx-body-file tx.raw --signing-key-file payment.skey --testnet-magic 1097911063 --out-file tx.signed
export CARDANO_NODE_SOCKET_PATH=~/cardano-node/relay/db/node.socket
cardano-cli transaction submit --tx-file tx.signed --testnet-magic 1097911063
```

Check the balance of the 2 utxo.



### Withdrawal

### Querying

## Monitoring

## Delete

