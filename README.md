## Hardware Requirements

Minimum:
* CPU with 4+ cores
* 16GB RAM
* 2TB free storage space to sync the Mainnet
* Internet connection speed of 25+ MB/s

## Update system packages

```shell
apt -y update && apt -y upgrade
apt dist-upgrade && sudo apt autoremove
```

## Create a non-root user

```shell
adduser ethereum
usermod -aG sudo ethereum
su - ethereum
```

## Verify system time

```shell
timedatectl
```

## Configure the firewall

```shell
sudo ufw allow 30303/tcp
sudo ufw allow 30303/udp
sudo ufw allow 12000/udp
sudo ufw allow 13000/tcp
sudo ufw allow 22/tcp
sudo ufw enable
```

### Confirm the status
```shell
sudo ufw status
```

## Generate authentication secret

### Creating the users
```shell
sudo adduser --home /home/geth --disabled-password --gecos 'Geth Client' geth
sudo adduser --home /home/prysm-beacon --disabled-password --gecos 'Prysm Beacon Client' prysm-beacon
sudo groupadd eth
sudo usermod -a -G eth geth
sudo usermod -a -G eth prysm-beacon
```

### Create a directory to store the JWT secret
```shell
sudo mkdir -p /var/lib/secrets
sudo chgrp -R eth /var/lib/ /var/lib/secrets
sudo chmod 750 /var/lib/ /var/lib/secrets
sudo openssl rand -hex 32 | tr -d '\n' | sudo tee /var/lib/secrets/jwt.hex > /dev/null
```

### Set permissions
```shell
sudo chown root:eth /var/lib/secrets/jwt.hex
sudo chmod 640 /var/lib/secrets/jwt.hex
```

## Create data directories

```shell
sudo -u geth mkdir /home/geth/geth
sudo -u prysm-beacon mkdir /home/prysm-beacon/beacon
```

## Install and configure execution client (Geth)

### Add the Ethereum PPA and Install Geth
```shell
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install ethereum
```

### Create a Systemd Service for Geth
```shell
sudo nano /etc/systemd/system/geth.service
```

### Add the following configuration to the file
```shell
[Unit]
Description=Geth Full Node
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
Restart=always
RestartSec=5s
User=geth
WorkingDirectory=/home/geth
ExecStart=/usr/bin/geth \
  --http \
  --http.port 8546 \
  --http.api eth,net,engine,admin \
  --ws \
  --ws.port 8547 \
  --ws.api eth,net,admin,debug \
  --mainnet \
  --datadir /home/geth/geth \
  --authrpc.jwtsecret /var/lib/secrets/jwt.hex \
  --port 30303

[Install]
WantedBy=multi-user.target
```

### Start and Enable the Geth Service
```shell
sudo systemctl daemon-reload
sudo systemctl start geth
sudo systemctl enable geth
```

### Check the Status of the Geth Service
```shell
sudo systemctl status geth
```

exit with `Ctrl + C`

### View the logs
```shell
sudo journalctl -fu geth
```

## Configure consensus client (Prysm)

### Download and Prepare the Prysm Script
```shell
sudo -u prysm-beacon mkdir /home/prysm-beacon/bin
sudo -u prysm-beacon curl https://raw.githubusercontent.com/prysmaticlabs/prysm/master/prysm.sh --output /home/prysm-beacon/bin/prysm.sh
sudo -u prysm-beacon chmod +x /home/prysm-beacon/bin/prysm.sh
```

### Create a Systemd Service for Prysm
```shell
sudo nano /etc/systemd/system/prysm-beacon.service
```

### Add the following configuration to the file
```shell
[Unit]
Description=Prysm Beacon Chain
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
Restart=always
RestartSec=5s
User=prysm-beacon
ExecStart=/home/prysm-beacon/bin/prysm.sh beacon-chain \
  --mainnet \
  --datadir /home/prysm-beacon/beacon \
  --execution-endpoint=http://127.0.0.1:8551 \
  --jwt-secret=/var/lib/secrets/jwt.hex \
  --suggested-fee-recipient=YourWalletAddress \
  --checkpoint-sync-url=https://beaconstate.info \
  --genesis-beacon-api-url=https://beaconstate.info \
  --accept-terms-of-use \
  --rpc-port=4002 \
  --grpc-gateway-port=4003 \
  --p2p-quic-port=13000 \
  --p2p-tcp-port=13000 \
  --p2p-udp-port=12000 \
  --monitoring-port=8080 \
  --pprofport=6060

[Install]
WantedBy=multi-user.target
```

### Start and Enable the Prysm Service
```shell
sudo systemctl daemon-reload
sudo systemctl start prysm-beacon
sudo systemctl enable prysm-beacon
```

### Check the Status of the Prysm Service
```shell
sudo systemctl status prysm-beacon
```

### View the log
```shell
sudo journalctl -fu prysm-beacon
```





