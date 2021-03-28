# easy-mina-block-producer-docker
A very summarized setup of a Mina protocol block producer node in Docker

Note: This is just to easily get your node up and running. For a setup with any serious amount of tokens, you should consider doing [something more secure](https://minaprotocol.com/docs/advanced/hot-cold-block-production).

### Requirements:
* Docker

### Steps (Linux or WSL)

1. Create a directory for your Mina related files:
```sh
mkdir -p ~/mina && cd ~/mina
```

2. *If you have no keys created already*, create a keypair in a new directory called `keys`:
```sh
 docker run  --interactive --tty --rm --volume $(pwd)/keys:/keys minaprotocol/generate-keypair:0.2.12-718eba4 -privkey-path keys/my-wallet
```

3. To add your newly created account to the public ledger, which is needed for it to be practically usable, you need to get a transaction of >= 1 Mina to your public address (`keys/my-wallet.pub`). 1 Mina of this first transaction will be burned as a fee for account synchronization.

4. Now you can fire up your block producer node. The probability of you getting a block will be proportional to the balance of your account. Substitute YOUR_PRIVATE_KEY_PASSWORD for your private key password you entered in step 2:
```sh
docker run --name mina_node -d \
-p 8302:8302 \
-p 8303:8303 \
--restart=always \
--mount "type=bind,source=`pwd`/keys,dst=/keys,readonly" \
--mount "type=bind,source=`pwd`/.mina-config,dst=/root/.mina-config" \
-e CODA_PRIVKEY_PASS="YOUR_PRIVATE_KEY_PASSWORD" \
minaprotocol/mina-daemon-baked:1.1.4-a8893ab \
daemon \
--block-producer-key /keys/my-wallet \
--insecure-rest-server \
--file-log-level Debug \
--log-level Info \
--peer-list-url https://storage.googleapis.com/mina-seed-lists/mainnet_seeds.txt
```

5. Optionally, if you want to run a SNARK worker (for which the current rates can be seen at [MinaExplorer](https://minaexplorer.com/snarketplace), you can append the following lines to the daemon's command:
```sh
--run-snark-worker <YOUR_PUBLIC_KEY> \
--snark-worker-fee <YOUR_FEE> \
```

6. View your node's progress (or failures) by running:
```sh
docker logs -f mina_node
```
