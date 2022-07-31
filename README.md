# DOCUMENTACION

## Punto de partida
El punto de partida es el Challenge 1:  
[https://github.com/near/stakewars-iii/tree/main/challenges](https://github.com/near/stakewars-iii/tree/main/challenges)

## Challenge 1
Crear una Wallet y desplegar NEAR-CLI

### 1.1 Crear la Wallet de guardia.shardnet.near
![](https://www.oceanblock.co/wp-content/uploads/2022/07/guardia-wallet.jpg)

### 1.2 Desplegar NEAR-CLI

	sudo apt update && sudo apt upgrade -y
	curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -  
	sudo apt install build-essential nodejs
	PATH="$PATH"

Comprobar versiones:

	node -v; npm -v;
>v18.6.0
>8.13.2

Desplegar NEAR-CLI desde NPM

	sudo npm install -g near-cli

### 1.3 Comprobar el estado de los validadores
	export NEAR_ENV=shardnet
	echo 'export NEAR_ENV=shardnet' >> ~/.bashrc
	near proposals
	near validators current
	near validators next
>Proposals for the epoch after next (new: 287, passing: 375, expected seat price = 30)
>| Status   | Validator  | Stake => New Stake | # Seats |
>| -------- | ---------- | ------------------ | ------- |
>| Rollover | boot1.near | 530,000            | 1       |

## Challenge 2
Compilar y ejecutar NEARD

### 2.1 Configurar servidor
Comprobar hardware
**4 CPU, 8 RAM, 500G SSD.**

	lscpu | grep -P '(?=.*avx )(?=.*sse4.2 )(?=.*cx16 )(?=.*popcnt )' > /dev/null && echo "Supported" || echo "Not supported"
>Supported

### 2.2 Instalar las herramientas de desarrollo
	sudo apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python docker.io protobuf-compiler libssl-dev pkg-config clang llvm cargo
	sudo apt install python3-pip
	USER_BASE_BIN=$(python3 -m site --user-base)/bin
	export PATH="$USER_BASE_BIN:$PATH"
	sudo apt install clang build-essential make
	curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
>stable-x86_64-unknown-linux-gnu installed - rustc 1.62.1 (e092d0b6b 2022-07-16)
> 
> Rust is installed now. Great!
> 
> To get started you may need to restart your current shell
> 
> This would reload your PATH environment variable to include
> 
> Cargo's bin directory ($HOME/.cargo/bin).

### 2.3 Compilar NEARCORE
	source $HOME/.cargo/env
	git clone https://github.com/near/nearcore
	cd nearcore
	git fetch
	git checkout 0f81dca95a55f975b6e54fe6f311a71792e21698
	cargo build -p neard --release --features shardnet
>info: syncing channel updates for '1.62.0-x86_64-unknown-linux-gnu'
> 
>info: latest update on 2022-06-30, rust version 1.62.0 (a8314ef7d 2022-06-27
> 
>Compiling nearcore v0.0.0 (/home/usuario/nearcore/nearcore)
> 
>Compiling state-viewer v0.0.0 (/home/usuario/nearcore/tools/state-viewer)
> 
>Finished release [optimized] target(s) in 8m 42s

### 2.4 Inicializar directorios de NEAR y descargar último config.json
	cd ~/nearcore
	./target/release/neard --home ~/.near init --chain-id shardnet --download-genesis
	rm ~/.near/config.json
	wget -O ~/.near/config.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json
>2022-07-22T12:23:26.523950Z  INFO neard: version="trunk" build="crates-0.14.0-234-g0f81dca95" latest_protocol=100
> 
>2022-07-22T12:23:26.524286Z  INFO near: Using key ed25519:3KAntfaM3gwgrQedWQgA3Xg4vzWScn8Zdcp6pN5mimqZ for node
> 
>2022-07-22T12:23:26.524338Z  INFO near: Downloading genesis file from: https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/genesis.json.xz ...
> 
>[00:00:01] [################################################################################################################################################] 3.29MB/3.29MB [2.17MB/s] (0s)
> 
>2022-07-22T12:23:28.783706Z  INFO near: Saved the genesis file to: /home/usuario/.near/genesis.json ...
> 
>2022-07-22T12:23:30.871685Z  INFO near: Generated for shardnet network node key and genesis file in /home/usuario/.near

### 2.5 Generar nuevas claves y activar validator_key.json
	near generate-key guardia.factory.shardnet.near
	cp ~/.near-credentials/shardnet/guardia.factory.shardnet.near.json ~/.near/validator_key.json
	nano ~/.near/validator_key.json
	cat ~/.near/validator_key.json
>{
>"account_id":"guardia.factory.shardnet.near",
>"public_key":"ed25519:ByngjDsAY6cpaMk3ce6DDCXmzRcpvZB4tHXd4MK7bH41",
>"secret_key":"ed25519:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
>}

### 2.6 Autorizar acceso a la cuenta de la Wallet
	near login

>Would you like to opt in (y/n)? y
>Please authorize NEAR CLI on at least one of your accounts.
>If your browser doesn't automatically open, please visit this URL
>https://wallet.shardnet.near.org/login/?referrer=NEAR+CLI&public_key=ed25519%3AHZDabNzEp2dt4ijrtnwHFLudd7tKeQmwxQzJt927oSr1&success_url=http%3A%2F%2F127.0.0.1%3A5000
>Please authorize at least one account at the URL above.
>Which account did you authorize for use with NEAR CLI?
>Enter it here (if not redirected automatically):
>guardia.shardnet.near
>Logged in as [ guardia.shardnet.near ] with public key [ ed25519:9TGuzH... ] successfully

### 2.7 Crear el servicio de Ubuntu
	sudo nano /etc/systemd/system/neard.service

Script

	[Unit]
	Description=NEARd Daemon Service
	
	[Service]
	Type=simple
	User=usuario
	#Group=usuario
	WorkingDirectory=/home/usuario/.near
	ExecStart=/home/usuario/nearcore/target/release/neard run
	Restart=on-failure
	RestartSec=30
	KillSignal=SIGINT
	TimeoutStopSec=45
	KillMode=mixed
	
	[Install]
	WantedBy=multi-user.target

### Arrancar el validador y esperar a que se sincronice
	 journalctl -n 100 -f -u neard | ccze -A
> 22 20:58:50 usuario neard[5184]: 2022-07-22T12:48:50.235239Z  INFO neard: version="trunk" build="crates-0.14.0-234-g0f81dca95" latest_protocol=100
> 
>jul 22 20:58:51 usuario neard[5184]: 2022-07-22T12:48:51.823118Z  INFO near: Creating new RocksDB database path=/home/usuario/.near/data
> 
>jul 22 20:58:52 usuario neard[5184]: 2022-07-22T12:48:52.232129Z  INFO db: Created a new RocksDB instance. num_instances=1
>
>jul 22 20:58:53 usuario neard[5184]: 2022-07-22T12:48:53.623573Z  INFO near_network::peer_manager::peer_manager_actor: Bandwidth stats total_bandwidth_used_by_all_peers=0 total_msg_received_count=0 max_max_record_num_messages_in_progress=0
>
>jul 22 20:58:53 usuario neard[5184]: 2022-07-22T12:48:53.232739Z  INFO stats: # 1050008 Waiting for peers 0 peers ⬇ 0 B/s ⬆ 0 B/s 0.00 bps 0 gas/s CPU: 0%, Mem: 372 MB

## Challenge 3

### 3.1 Desplegar el SmartContract del staking y depositar seat price
	near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "guardia", "owner_id": "guardia.shardnet.near", "stake_public_key": "ed25519:ByngjDsAY6cpaMk3ce6DDCXmzRcpvZB4tHXd4MK7bH41", "reward_fee_fraction": {"numerator": 5, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="guardia.shardnet.near" --amount=30 --gas=300000000000000
	- Stake NEAR for seat price
	near call guardia.factory.shardnet.near deposit_and_stake --amount 510 --accountId guardia.shardnet.near --gas=300000000000000

![](https://www.oceanblock.co/wp-content/uploads/2022/07/guardia-contract.jpg)

### 3.2 Hacer ping a la pool del staking smartcontract para entrar en validator proposal
	near call guardia.factory.shardnet.near ping '{}' --accountId guardia.shardnet.near --gas=300000000000000
>Scheduling a call: guardia.factory.shardnet.near.ping({})
>
>Doing account.functionCall()
>
>Transaction Id 8g8yvkb3UhFuzVD7uoTSWVB5mgcJTub1eqGGK8VRGPJr
>
>To see the transaction in the transaction explorer, please open this url in your browser
>
>https://explorer.shardnet.near.org/transactions/8g8yvkb3UhFuzVD7uoTSWVB5mgcJTub1eqGGK8VRGPJr

### 3.3 Ver balance de staked
	near view guardia.factory.shardnet.near get_account_staked_balance '{"account_id": "guardia.shardnet.near"}'
>View call: guardia.factory.shardnet.near.get_account_staked_balance({"account_id": "guardia.shardnet.near"})
> 
>'2728021113101289446419868131'

### 3.4 Comprobar estado para entrar en el set de activo.
	near proposals | grep guardia
>| Proposal(Accepted) | guardia.factory.shardnet.near              | 540                | 1       |

## Challenge 4

### 4.1 Log files
	journalctl -n 100 -f -u neard | ccze -A
>jul 23 01:24:18 shardnet01 neard[57318]: 2022-07-22T23:24:18.229058Z  INFO stats: # 1187840 8WBJg3JcZoUnyuBwuMoSDpCnUZSkymtQhcUNtyxRofvd Validator | 100 validators 30 peers ⬇ 455 kB/s ⬆ 710 kB/s 0.80 bps 1.53 Tgas/s CPU: 39%, Mem: 3.75 GB

### 4.2 RPC
	curl -s http://127.0.0.1:3030/status | jq .version
>{
>  "version": "trunk",
>  "build": "crates-0.14.0-234-g0f81dca95",
>  "rustc_version": "1.62.0"
>}

	curl -s -d '{"jsonrpc": "2.0", "method": "validators", "id": "dontcare", "params": [null]}' -H 'Content-Type: application/json' 127.0.0.1:3030 | jq -c '.result.prev_epoch_kickout[] | select(.account_id | contains ("guardia.factory.shardnet.near"))' | jq .reason

### 4.3 Otros metodos de interes
	near view guardia.factory.shardnet.near get_accounts '{"from_index": 0, "limit": 10}' --accountId guardia.shardnet.near
	View call: guardia.factory.shardnet.near.get_accounts({"from_index": 0, "limit": 10})
	[
	{
		account_id: 'guardia.shardnet.near',
		unstaked_balance: '10',
		staked_balance: '2728021113101289446419868131',
		can_withdraw: false
	},
	{
		account_id: '0000000000000000000000000000000000000000000000000000000000000000',
		unstaked_balance: '0',
		staked_balance: '9142972411794863663056',
		can_withdraw: true
	}
	]

![](https://www.oceanblock.co/wp-content/uploads/2022/07/guardia-explorer2.jpg)

## Challenge 5
Crear el validador en un DataCenter y documentar.

> Dar de alta en Hetzner un servidor dedicado en Hetznet.
> 
> Hacer este documento.
> 
![](https://raw.githubusercontent.com/ivanguardia/stakewarsiii/main/hetzner.jpg)
> 
> Rellenar y enviar el formulario del Challenge 5.


## Challenge 6

### 6.1 Crear el script de ping

#### ping.sh
	 #!/bin/sh
	 # Ping call to renew Proposal added to crontab
	 
	 export NEAR_ENV=shardnet
	 export LOGS=/home/usuario/logs
	 export POOLID=guardia
	 export ACCOUNTID=guardia
	 
	 echo "---" >> $LOGS/all.log
	 date >> $LOGS/all.log
	 near call $POOLID.factory.shardnet.near ping '{}' --accountId $ACCOUNTID.shardnet.near --gas=300000000000000 >> $LOGS/all.log
	 near proposals | grep $POOLID >> $LOGS/all.log
	 near validators current | grep $POOLID >> $LOGS/all.log
	 near validators next | grep $POOLID >> $LOGS/all.log

### 6.2 Create cron
Ejecutar cada 2 horas

	crontab -e

	# m h  dom mon dow   command
	0 */2 * * * sh /home/usuario/scripts/ping.sh


### 6.3 Check logs
	cat ~/logs/all.log
>sáb 23 jul 2022 01:10:01 CEST
> 
>Scheduling a call: guardia.factory.shardnet.near.ping({})
>
>Doing account.functionCall()
>
>Transaction Id 8g8yvkb3UhFuzVD7uoTSWVB5mgcJTub1eqGGK8VRGPJr>

>To see the transaction in the transaction explorer, please open this url in your browser>

>https://explorer.shardnet.near.org/transactions/8g8yvkb3UhFuzVD7uoTSWVB5mgcJTub1eqGGK8VRGPJr
> 
> | Proposal(Accepted) | guardia.factory.shardnet.near               | 2,758 => 2,758     | 1       |
> 
> | guardia.factory.shardnet.near               | 2,758   | 1       | 96.29%   |               4 |               4 |              22 |              23 |
> 
> | Rewarded   | guardia.factory.shardnet.near            | 2,758 -> 2,758     | 1       |

### 6.4 Registrarse cron.cat y dar de alta el ping
![](https://www.oceanblock.co/wp-content/uploads/2022/07/guardia-croncat.jpg)

## ACTUALIZACIONES 31/07/2022
  - Hasta le fecha se han hecho 2 hardfork.  
  - Se han solicitado actualizar el software varias veces. Ultimo commit c1b047b8187accbf6bd16539feb7bb60185bdc38.
  - Se ha actualizado la genesis y el config.
  - Se han cambiado la frecuencia del ping de 5 horas a 2 horas.
  - El RPC no acaba de funcionar y no se actualiza adecuadamete el node explorer.
 [Shardnet node explorer](https://explorer.shardnet.near.org/nodes/validators)
- Se ha restringido los token al crear una cuenta por abusos. Se tiene que solicitar en discord.
- He colaborado en el canal de Discord a aquellas personas que tenian problemas con al puesta en marcha.
- Se mantiene el servidor online para mantener la estabilidad de la red.
  
Script para actualizar genesis, config y build de NEARCORE
	# STOP SERVICE NEARD
	sudo systemctl stop neard

	# REMOVE OLD DB
	rm ~/.near/data/*

	# BUILD NEW COMMIT
	export NEAR_ENV=shardnet
	echo 'export NEAR_ENV=shardnet' >> ~/.bashrc
	cd ~/nearcore
	git fetch
	git checkout c1b047b8187accbf6bd16539feb7bb60185bdc38
	sudo systemctl stop neard
	cargo build -p neard --release --features shardnet
	sudo systemctl start neard
	sudo journalctl -n 1000 -f -u neard | ccze -A


	#  DOWNLOAD NEW GENESIS AND CONFIG
	cd ~/.near
	rm config.json
	rm genesis.json
	wget https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json
	wget https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/genesis.json


	#START SERVICE NEARD
	sudo systemctl start neard

	# CHECK STATUS, PEERS AND SYNCING
	sudo journalctl -n 1000 -f -u neard | ccze -A
