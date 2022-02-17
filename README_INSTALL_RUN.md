
# Note for Building and Running TuringBitChain on Ubuntu 2004 LTS


## How To Install Dependencies For Ubuntu 20.04 LTS:
```
sudo apt-get install build-essential libtool autotools-dev automake pkg-config libssl-dev libevent-dev bsdmainutils
sudo apt-get install libboost-system-dev libboost-filesystem-dev libboost-chrono-dev libboost-program-options-dev libboost-test-dev libboost-thread-dev
sudo apt-get install libdb-dev
sudo apt-get install libdb++-dev

```


## How To Build

C++ compilers are memory-hungry. It is recommended to have at least 1.5 GB of
memory available when compiling Bitcoin SV. 

For Memory > 1.5GB:
```bash
./autogen.sh

./configure --enable-cxx --disable-shared --with-pic --prefix=/home/$USER/tbc/v105rc 
or
./configure --enable-cxx --disable-shared --with-pic --prefix=/home/$USER/tbc/v105rc --enable-prod-build

make
make install  # optional
```

On systems with less, gcc can be
tuned to conserve memory with additional CXXFLAGS:
```
./configure CXXFLAGS="--param ggc-min-expand=1 --param ggc-min-heapsize=32768"
```


## Output Log of Building:
``` 
...
configure: creating ./config.status
config.status: creating Makefile
config.status: creating libsecp256k1.pc
config.status: creating src/libsecp256k1-config.h
config.status: src/libsecp256k1-config.h is unchanged
config.status: executing depfiles commands
config.status: executing libtool commands
Fixing libtool for -rpath problems.

Options used to compile and link:
  prod build    = no
  with wallet   = yes
  with zmq      = yes
  with test     = yes
  with bench    = yes
  with upnp     = yes
  use asm       = yes
  debug enabled = no
  werror        = no

  sanitizers    
          asan  = no
          tsan  = no
          ubsan = no

  memory allocators
       tcmalloc = no
       jemalloc = no

  target os     = linux
  build os      = 

  CC            = gcc
  CFLAGS        = -g -O2
  CPPFLAGS      =  -DHAVE_BUILD_INFO -D__STDC_FORMAT_MACROS
  CXX           = g++ -std=c++17
  CXXFLAGS      = -g -O2 -Wall -Wextra -Wformat -Wvla -Wformat-security -Wno-unused-parameter
  LDFLAGS       = 
```

## How to Run


### Configure

```
cd /home/$USER/tbc/v105rc
cat > node.noprune.conf << EOF
##
## bitcoin.conf configuration file. Lines beginning with # are comments.
##

#start in background
daemon=1

#Required Consensus Rules for Genesis
excessiveblocksize=10000000000 #10GB
maxstackmemoryusageconsensus=100000000 #100MB

#Mining
#biggest block size you want to mine
blockmaxsize=4000000000 
blockassembler=journaling  #journaling is default as of 1.0.5

#preload mempool
preload=1

# Index all transactions, prune mode don&t support txindex
txindex=1
#reindex=1
#reindex-chainstate=1

#Other Sys, ws add
maxmempool=6000
dbcache=1000 

#Other Block, ws add
threadsperblock=6
#prune=196000

#Other Tx Conf:
maxscriptsizepolicy=0
blockmintxfee=0.00000250

 
# Network-related settings:

# Run on the test network instead of the real bitcoin network.
#testnet=0

# Run a regression test network
#regtest=0

# Connect via a SOCKS5 proxy
#proxy=127.0.0.1:9050

# Bind to given address and always listen on it. Use [host]:port notation for IPv6
#bind=<addr>

# Bind to given address and whitelist peers connecting to it. Use [host]:port notation for IPv6
#whitebind=<addr>

##############################################################
##            Quick Primer on addnode vs connect            ##
##  Let's say for instance you use addnode=4.2.2.4          ##
##  addnode will connect you to and tell you about the      ##
##    nodes connected to 4.2.2.4.  In addition it will tell ##
##    the other nodes connected to it that you exist so     ##
##    they can connect to you.                              ##
##  connect will not do the above when you 'connect' to it. ##
##    It will *only* connect you to 4.2.2.4 and no one else.##
##                                                          ##
##  So if you're behind a firewall, or have other problems  ##
##  finding nodes, add some using 'addnode'.                ##
##                                                          ##
##  If you want to stay private, use 'connect' to only      ##
##  connect to "trusted" nodes.                             ##
##                                                          ##
##  If you run multiple nodes on a LAN, there's no need for ##
##  all of them to open lots of connections.  Instead       ##
##  'connect' them all to one node that is port forwarded   ##
##  and has lots of connections.                            ##
##       Thanks goes to [Noodle] on Freenode.               ##
##############################################################

# Use as many addnode= settings as you like to connect to specific peers
#addnode=69.164.218.197
#addnode=10.0.0.2:8333
addnode=node.metasv.com:8333

# Alternatively use as many connect= settings as you like to connect ONLY to specific peers
#connect=69.164.218.197
#connect=10.0.0.1:8333
#connect=node.metasv.com:8333

# Listening mode, enabled by default except when 'connect' is being used
#listen=1

# Maximum number of inbound+outbound connections.
maxconnections=12

#
# JSON-RPC options (for controlling a running Bitcoin/bitcoind process)
#

# server=1 tells bitcoind to accept JSON-RPC commands
server=1

# Bind to given address to listen for JSON-RPC connections. Use [host]:port notation for IPv6.
# This option can be specified multiple times (default: bind to all interfaces)
rpcbind=127.0.0.1

# If no rpcpassword is set, rpc cookie auth is sought. The default `-rpccookiefile` name
# is .cookie and found in the `-datadir` being used for bitcoind. This option is typically used
# when the server and client are run as the same user.
#
# If not, you must set rpcuser and rpcpassword to secure the JSON-RPC api. The first
# method(DEPRECATED) is to set this pair for the server and client:
rpcuser=tbcuser
rpcpassword=randompasswd
#
# The second method `rpcauth` can be added to server startup argument. It is set at intialization time
# using the output from the script in share/rpcuser/rpcuser.py after providing a username:
#
# ./share/rpcuser/rpcuser.py alice
# String to be appended to bitcoin.conf:
# rpcauth=alice:f7efda5c189b999524f151318c0c86$d5b51b3beffbc02b724e5d095828e0bc8b2456e9ac8757ae3211a5d9b16a22ae
# Your password:
# DONT_USE_THIS_YOU_WILL_GET_ROBBED_8ak1gI25KFTvjovL3gAM967mies3E=
#
# On client-side, you add the normal user/password pair to send commands:
#rpcuser=alice
#rpcpassword=DONT_USE_THIS_YOU_WILL_GET_ROBBED_8ak1gI25KFTvjovL3gAM967mies3E=
#
# You can even add multiple entries of these to the server conf file, and client can use any of them:
# rpcauth=bob:b2dd077cb54591a2f3139e69a897ac$4e71f08d48b4347cf8eff3815c0e25ae2e9a4340474079f55705f40574f4ec99

# How many seconds bitcoin will wait for a complete RPC HTTP request.
# after the HTTP connection is established. 
#rpcclienttimeout=30

# By default, only RPC connections from localhost are allowed.
# Specify as many rpcallowip= settings as you like to allow connections from other hosts,
# either as a single IPv4/IPv6 or with a subnet specification.

# NOTE: opening up the RPC port to hosts outside your local trusted network is NOT RECOMMENDED,
# because the rpcpassword is transmitted over the network unencrypted.

# server=1 is read by bitcoind to determine if RPC should be enabled 
#rpcallowip=10.1.1.34/255.255.255.0
#rpcallowip=1.2.3.4/24
#rpcallowip=2001:db8:85a3:0:0:8a2e:370:7334/96
rpcallowip=211.86.151.13/255.255.255.0

# Listen for RPC connections on this TCP port:
rpcport=8332
EOF
```

### Exe Command

/home/$USER/tbc/v105rc/bin/bitcoind -conf=/home/$USER/tbc/v105rc/node.noprune.conf -datadir=/home/$USER/tbc/node_data_dir

alias tbc-cli="/home/$USER/tbc/v105rc/bin/bitcoind-cli -conf=/home/$USER/tbc/v105rc/node.noprune.conf" 

tbc-cli  getinfo
tbc-cli  getmemoryinfo
tbc-cli  checkjournal

tbc-cli  getblockchaininfo
tbc-cli  getblockbyheight 0
tbc-cli  getblockcount
tbc-cli  getmininginfo

tbc-cli  getpeerinfo
tbc-cli  getnetworkinfo
tbc-cli  getnettotals
tbc-cli  listbanned

tbc-cli  listwallets
tbc-cli  listaccounts
tbc-cli  getwalletinfo
tbc-cli  getaddressesbyaccount

tbc-cli  stop


### Use Process Manager: pm2

recommend to use pm2 for auto restart and monitoring ...

sudo npm install pm2 -g

sed -i 's/daemon=1/daemon=0/' node.zws.noprune.conf

pm2 --name tbcd --max-restarts 20 start "/home/$USER/tbc/v105rc/bin/bitcoind -conf=/home/$USER/tbc/v105rc/node.noprune.conf -datadir=/home/$USER/tbc/node_data_dir"

pm2 ps


