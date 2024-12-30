# `docker-bitcoind` Project

Docker image that runs the `bitcoind` node in a container for easy deployment.

The binary is compiled from source by pulling it from the [Bitcoin Core website](https://bitcoincore.org/).
The file's checksum is verified prior building the image.

The provided bitcoin node has support for:

* Bitcoin Wallet
* Port mapping: UPnP and NAT-PMP
* ZMQ
* User-Space, Statically Defined Tracing (USDT)

Within the image the following binaries are provided, all under the `/usr/local/bin/` directory:

* `bitcoind`
* `bitcoin-cli`
* `bitcoin-tx`
* `bitcoin-util`
* `bitcoin-wallet`

## Getting Started

In order to setup a Bitcoin node with the default options we first need to create a docker volume for the node's data:

```bash
docker volume create --name=bitcoin-data
```

Using a volume will allow to not lose the data across container recreations which will happen in the case of host restart or updating the node's image.

To launch the node with the default options using the latest version. The following command can be used:

```bash
docker run -d \
   --name bitcoin \
   -v bitcoin-data:/home/bitcoin/.bitcoin \
   -p 8333:8333 \ # Bitcoin's P2P port
   -p 127.0.0.1:8332:8332 \ # Only accept RPC from localhost
   --restart unless-stopped \
   salessandri/bitcoind \
      /usr/local/bin/bitcoind \
      -disablewallet=1
```

## Custom Configuration

Two mechanisms can be used to change the default configuration options: command line arguments and a configuration file.
Either or both can used at the same time. If both are used, the command line options will override the configuration values found in the file.

### Command Line Arguments

The image has been built in such a way that any parameters added to the docker run command will be passed to the `bitcoind` binary. This makes it easy to change the configuration options in the docker command instantiating the container. For example:

```bash
docker run -d \
   --name bitcoin \
   -v bitcoin-data:/home/bitcoin/.bitcoin \
   -p 8333:8333 \
   -p 127.0.0.1:8332:8332 \
   --restart unless-stopped \
   salessandri/bitcoind \
      /usr/local/bin/bitcoind \
      -disablewallet=1 \
      -txindex=1 \
      -natpmp=1
```

## Configuration File

The other possibility to change the default configuration values is through the use of a configuration file. The default location this image will look the config file is `/home/bitcoin/.bitcoin/bitcoin.conf`, thus one can mount a file in that location:

```bash
docker run -d \
   --name bitcoin \
   -v bitcoin-data:/home/bitcoin/.bitcoin \
   -v /path/to/bitcoin.conf:/home/bitcoin/.bitcoin/bitcoin.conf \
   -p 8333:8333 \
   -p 127.0.0.1:8332:8332 \
   --restart unless-stopped \
   salessandri/bitcoind
```

It is also possible to change the location the bitcoin process will look for the file through the `-conf` command line argument:

```bash
docker run -d \
   --name bitcoin \
   -v bitcoin-data:/home/bitcoin/.bitcoin \
   -v /path/to/bitcoin.conf:/etc/bitcoin.conf \
   -p 8333:8333 \
   -p 127.0.0.1:8332:8332 \
   --restart unless-stopped \
   salessandri/bitcoind \
      /usr/local/bin/bitcoind \
      -printtoconsole=1 \
      -conf=/etc/bitcoin.conf
```

An example what this file can contain can be found in `/usr/local/share/bitcoin.conf` within the container.

## Build Image

If instead of using the pre-built images you prefer to build it from source that can be done by issuing the following command from the root of this repo:

```bash
docker buildx build --load -t bitcoin -f docker/Dockerfile .
```

## License

This project is licensed under the MIT License. See the LICENSE file for more details.
