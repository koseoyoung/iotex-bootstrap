# IoTeX Delegate Manual

## News

- We have upgraded testnet to `v0.8.0`, which contains the upgrade of evm, so that developers are able to deploy
the contract written in Solidity 0.5.0+. It's a hard fork that would happen at block height 699841. If people want
to run the node for testnet, please pull the latest testnet genesis again, and restart the node with `v0.8.0` image.
- `v0.7.3` is available. it's a compatible upgrade from `0.7.2`.
- Please upgrade MainNet to v0.7.2. Delegates only need to restart the node with the docker image tagged `v0.7.2`.

## Index

- [Release Status](#status)
- [Join MainNet Alpha](#mainnet)
- [Join TestNet](#testnet)
- [Interact with Blockchain](#ioctl)
- [Operate Your Node](#ops)


## <a name="status"/>Release Status

Here are the software versions we use:

- MainNet: [v0.7.3](https://github.com/iotexproject/iotex-core/tree/b3421b05b12da3b04e4e36d4d8c064b3eea25872)
- TestNet: v0.8.0

## <a name="mainnet"/>Join MainNet Alpha

1. Pull the docker image:

```
docker pull iotex/iotex-core:v0.7.3
```

Please check if the docker image digest is `5600873ecb9fa06567a13fab515148775ec8c2ca5d3d5aeb89724569b262014d`.

2. Set the environment with the following commands:

```
mkdir -p ~/iotex-var
cd ~/iotex-var

export IOTEX_HOME=$PWD

mkdir -p $IOTEX_HOME/data
mkdir -p $IOTEX_HOME/log
mkdir -p $IOTEX_HOME/etc

curl https://raw.githubusercontent.com/iotexproject/iotex-bootstrap/master/config_mainnet.yaml > $IOTEX_HOME/etc/config.yaml
curl https://raw.githubusercontent.com/iotexproject/iotex-bootstrap/master/genesis_mainnet.yaml > $IOTEX_HOME/etc/genesis.yaml
```

3. Edit `$IOTEX_HOME/etc/config.yaml`, look for `externalHost` and `producerPrivKey`, uncomment the lines and fill in your external IP and private key.

4. (Optional) If you prefer to start from a snapshot, run the following commands:

```
curl -L https://t.iotex.me/mainnet-data-latest > $IOTEX_HOME/data.tar.gz
tar -xzf data.tar.gz
```
We will update the snapshot once a day. If you plan to run your node as a gateway, please use the snapshot with index data:
https://t.iotex.me/mainnet-data-with-idx-latest.

5. Run the following command to start a node:

```
docker run -d --restart on-failure --name iotex \
        -p 4689:4689 \
        -p 8080:8080 \
        -v=$IOTEX_HOME/data:/var/data:rw \
        -v=$IOTEX_HOME/log:/var/log:rw \
        -v=$IOTEX_HOME/etc/config.yaml:/etc/iotex/config_override.yaml:ro \
        -v=$IOTEX_HOME/etc/genesis.yaml:/etc/iotex/genesis.yaml:ro \
        iotex/iotex-core:v0.7.3 \
        iotex-server \
        -config-path=/etc/iotex/config_override.yaml \
        -genesis-path=/etc/iotex/genesis.yaml
```

Now your node should be started successfully.

If you want to also make your node be a gateway, which could process API requests from users, use the following command instead:

```
docker run -d --restart on-failure --name iotex \
        -p 4689:4689 \
        -p 14014:14014 \
        -p 8080:8080 \
        -v=$IOTEX_HOME/data:/var/data:rw \
        -v=$IOTEX_HOME/log:/var/log:rw \
        -v=$IOTEX_HOME/etc/config.yaml:/etc/iotex/config_override.yaml:ro \
        -v=$IOTEX_HOME/etc/genesis.yaml:/etc/iotex/genesis.yaml:ro \
        iotex/iotex-core:v0.7.3 \
        iotex-server \
        -config-path=/etc/iotex/config_override.yaml \
        -genesis-path=/etc/iotex/genesis.yaml \
        -plugin=gateway
```

6. Make sure TCP ports 4689, 8080 (also 14014 if used) are open on your firewall and load balancer (if any).

## <a name="testnet"/>Join TestNet

There's almost no difference to join TestNet, but in step 2, you need to use the config and genesis files for TestNet:

```
curl https://raw.githubusercontent.com/iotexproject/iotex-bootstrap/master/config_testnet.yaml > $IOTEX_HOME/etc/config.yaml
curl https://raw.githubusercontent.com/iotexproject/iotex-bootstrap/master/genesis_testnet.yaml > $IOTEX_HOME/etc/genesis.yaml
```

In step 4, you need to use the snapshot for TestNet: https://t.iotex.me/testnet-data-latest and https://t.iotex.me/testnet-data-with-idx-latest. 

In step 5, you need to replace the docker image tag in the command with `v0.8.0`.

## <a name="ioctl"/>Interact with Blockchain


You can install `ioctl` (a command-line interface for interacting with IoTeX blockchain)

```
curl https://raw.githubusercontent.com/iotexproject/iotex-core/master/install-cli.sh | sh
```

You can point `ioctl` to your node (if you enable the gateway plugin):

```
ioctl config set endpoint localhost:14014 --insecure
```

Or you can point it to our nodes:

- MainNet secure: `api.iotex.one:443`
- MainNet insecure: `api.iotex.one:80`
- TestNet secure: `api.testnet.iotex.one:443`
- TestNet insecure: `api.testnet.iotex.one:80`

If you want to set an insecure endpoint, you need to add `--insecure` option.

Generate key:
```
ioctl account create
```

Get consensus delegates of current epoch:
```
ioctl node delegate
```

Refer to [CLI document](https://github.com/iotexproject/iotex-core/blob/master/ioctl/README.md) for more details.

### Other Commonly Used Commands

Claim reward:
```
ioctl action claim ${amountInIOTX} -l 10000 -p 1 -s ${ioAddress|alias}
```

Exchange IoTeX native token to ERC20 token on Ethereum via Tube service:
```
ioctl action invoke io1p99pprm79rftj4r6kenfjcp8jkp6zc6mytuah5 ${amountInIOTX} -s ${ioAddress|alias} -l 400000 -p 1 -b d0e30db0
```
Click [IoTeX Tube docs](https://github.com/iotexproject/iotex-bootstrap/blob/master/tube/tube.md) for detailed documentation of the tube service.

## <a name="ops"/>Operate Your Node

### Checking Node log

Container logs can be accessed with the following command. 

```
docker logs iotex
```

Content can be filtered with:

```
docker logs -f --tail 100 iotex |grep --color -E "epoch|height|error|rolldposctx"
```

### Stop and remove container

When starting the container with ```--name=iotex```, you must remove the old container before a new build.

```
docker stop iotex
docker rm iotex
```

### Pause and Restarting container

Container can be "stopped" and "restarted" with:

```
docker stop iotex
docker start iotex
```
