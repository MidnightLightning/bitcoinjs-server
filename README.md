# BitcoinJS Server

[![Build Status](https://secure.travis-ci.org/bitcoinjs/bitcoinjs-server.png?branch=master)](http://travis-ci.org/bitcoinjs/bitcoinjs-server)

This is an implementation of a Bitcoin node in Node.js. It is intended
as a powerful alternative to the standard bitcoind that ships with the
original client.

[Website](http://bitcoinjs.org/) • [Mailing List](https://groups.google.com/group/bitcoinjs) • [Twitter](https://twitter.com/bitcoinjs)

# Differences to original client

The original client contains the node, wallet, GUI and miner. This
library contains a highly optimized version of the node, i.e. the P2P
part of Bitcoin. Its main intended use is as a server component to
give lighter clients access to the data in the block chain (in
real-time.)

But it can also be used for writing other software that
requires real-time data from the block chain or wants to run queries
against it.


# Documentation

Please visit the [BitcoinJS Server
Wiki](https://github.com/bitcoinjs/bitcoinjs-server/wiki) on Github
for documentation.

There is also a built-in documentation system which you can access by
running `bitcoinjs help`.


# Installation

## Prerequisites

* [OpenSSL](http://www.openssl.org/) (lib and headers)
* [pkg-config](http://www.freedesktop.org/wiki/Software/pkg-config)
* [Node.js](https://github.com/joyent/node) 0.10.5+

Node.js should be compiled manually based on the latest stable
release.

### LevelDB

LevelDB is the default database back end in BitcoinJS 0.2+. It is
bundled with the `node-leveldb` NPM module and is therefore installed
automatically. No manual steps should be necessary.

### MongoDB (deprecated)

If you wish to use a MongoDB database, you need to setup a MongoDB
server. This can be done using any method.

You also need to install the MongoDB Node.js module:

``` sh
cd /path/to/bitcoinjs-server
npm install mongodb
```

Note that we don't recommend using MongoDB. We see BitcoinJS as a
special purpose database server, so having a general purpose database
server behind introduces tremendous overhead. For more information on
how to query BitcoinJS in LevelDB mode, please refer to the wiki.


## Installation

Once you have all prerequisites, this one-liner will install the
latest version straight from NPM:

``` sh
# Install BitcoinJS Server globally
sudo npm install bitcoinjs -g
```

Please refer to the wiki for detailed [installation
instructions](https://github.com/bitcoinjs/bitcoinjs-server/wiki/Installation).

If you run into problems, please take a look at the "Troubleshooting"
section below or go to
[Issues](https://github.com/bitcoinjs/bitcoinjs-server/issues) to open
a new ticket.

# Usage

For your first experience with the BitcoinJS daemon, try running it
right from the terminal.

``` sh
bitcoinjs run --testnet --bchdbg
```

You can find out more about the various functions of the command line
utility via the help feature:

``` sh
bitcoinjs help
```


## Uninstall

``` sh
# Remove the database
bitcoinjs db-drop

# Uninstall the software
sudo npm uninstall bitcoinjs -g
```


## Logging

BitcoinJS logs using the winston library. Currently, it
defaults to logging anything on the `debug` log level and higher. Here
are the available log levels:

- `netdbg` - Networking events (sending/receiving messages)
- `bchdbg` - Block chain events (adding blocks)
- `rpcdbg` - JSON-RPC API events (requests/responses)
- `scrdbg` - Script interpreter events (custom scripts, errors)
- `debug` - Other verbose logging
- `info` - General information and status messages
- `warn` - Something rare happened (e.g. strange pubKeyScript)
- `error` - Something bad happened

The XXXdbg levels can be enabled individually by editing
lib/logger.js or via the command line, e.g. `bitcoinjs run --bchdbg`.


## Advanced usage

BitcoinJS is not only a daemon, but also a Node.js
module/library. In most cases it's best to use the daemon via RPC. But
sometimes you need the extra control that comes with directly linking
to the code.

For details on developing with BitcoinJS as a library, take a
look at the Developer Guide on the
[wiki](https://github.com/bitcoinjs/bitcoinjs-server/wiki).


# Upgrading

When upgrading BitcoinJS it is a good idea to reset its
database:

``` sh
bitcoinjs db-drop
```

This won't be necessary once BitcoinJS is more stable, but for
now new versions often break database compatibility, so it's easiest
to just reset it.


# Tests

To run the test suite, please install [Vows](http://vowsjs.org) and
run the following command:

``` sh
vows test/*.js --spec
```

# Status

The library is currently alpha quality. Here are some things it
currently lacks:

- Respond to getblocks requests
- Manage knowledge about other peers
- DoS protections from Bitcoin 0.4+

On top of that, it could use a lot more documentation, test
cases and general bug fixing across the board.

You can find more information on the Issues tab on Github.

# Troubleshooting

## Native module missing

If you see this error:

    Error: Cannot find module '../build-cc/default/native'

This happens when the native components of BitcoinJS are not compiled
yet.

Go to the `bitcoinjs` folder and run:

node.js > v0.6.0 
``` sh
node-gyp configure build
```

node.js < v0.6.0 
``` sh
node-waf configure build
```

# License

This product is free and open-source software released under the MIT
license.

# Credits

BitcoinJS Server - Node.js Bitcoin client<br>
Copyright (c) 2011-2012 Stefan Thomas <justmoon@members.fsf.org>.

Some native extensions are<br>
Copyright (c) 2011-2012 Andrew Schaaf <andrew@andrewschaaf.com>

Parts of this software are based on [BitcoinJ](http://code.google.com/p/bitcoinj/)<br>
Copyright (c) 2011-2012 Google Inc.
