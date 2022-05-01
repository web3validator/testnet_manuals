<p align="center">
  <img height="100" height="auto" src="https://user-images.githubusercontent.com/50621007/166148846-93575afe-e3ce-4ca5-a3f7-a21e8a8609cb.png">
</p>

# Manual node setup
If you want to setup fullnode manually follow the steps below

## Setting up vars
Here you have to put name of your moniker (validator) that will be visible in explorer
```
NODENAME=<MY_MONIKER_NAME_GOES_HERE>
```

Save and import variables into system
```
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
echo "export WALLET=wallet" >> $HOME/.bash_profile
echo "export CHAIN_ID=quicktest-1" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Update packages
```
sudo apt update && sudo apt upgrade -y
```

## Install dependencies
```
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

## Install go
```
ver="1.17.2"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
go version
```

## Download and build binaries
```
cd $HOME
git clone https://github.com/ingenuity-build/quicksilver.git --branch v0.1.1
cd quicksilver
make build
chmod +x ./build/quicksilverd && mv ./build/quicksilverd /usr/local/bin/quicksilverd
```

## Config app
```
quicksilverd config chain-id $CHAIN_ID
quicksilverd config keyring-backend file
```

## Init app
```
quicksilverd init $NODENAME --chain-id $CHAIN_ID
```

## Download genesis
```
wget -qO $HOME/.quicksilverd/config/genesis.json "https://raw.githubusercontent.com/ingenuity-build/testnets/main/rhapsody/genesis.json"
```

## Set minimum gas price
```
sed -i.bak -e "s/^minimum-gas-prices = \"\"/minimum-gas-prices = \"0uqck\"/" $HOME/.quicksilverd/config/app.toml
```

## Set seeds and peers
```
SEEDS="dd3460ec11f78b4a7c4336f22a356fe00805ab64@seed.quicktest-1.quicksilver.zone:26656"
PEERS="node02.quicktest-1.quicksilver.zone:26657,node03.quicktest-1.quicksilver.zone:26657,node04.quicktest-1.quicksilver.zone:26657"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.quicksilverd/config/config.toml
```

## Enable prometheus
```
sed -i.bak -e "s/prometheus = false/prometheus = true/" $HOME/.quicksilverd/config/config.toml
```

## Reset chain data
```
quicksilverd unsafe-reset-all
```

## Create service
```
tee /etc/systemd/system/quicksilverd.service > /dev/null <<EOF
[Unit]
Description=quicksilverd
After=network.target
[Service]
Type=simple
User=$USER
ExecStart=$(which quicksilverd) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

## Register and start service
```
sudo systemctl daemon-reload
sudo systemctl enable quicksilverd
sudo systemctl restart quicksilverd
```