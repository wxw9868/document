##### 编译erigon和rpcdaemon
````
git clone --recurse-submodules -j8 https://github.com/ledgerwatch/erigon.git
cd erigon
make erigon
make rpcdaemon
````
##### 搭建私有链
````
cd erigon/build/bin
````
###### Node1
````
./erigon --identity "node1" --datadir data/starlink --chain amc --private.api.addr 0.0.0.0:8090 \
         --http.addr=0.0.0.0 --http.port=8545 --http.corsdomain="*" \
         --http.api="eth,erigon,ots,web3,net,debug,trace,txpool,parity,addr" \
         --metrics --metrics.expensive --metrics.addr=0.0.0.0 --metrics.port=6060 \
         --pprof --pprof.addr=0.0.0.0 --pprof.port=6061 \
         --engine.addr=0.0.0.0 --engine.port=8551 \
         --ws --ws.compression \
         --healthcheck \
         --port=30303 --torrent.port=42069 \
         --mine --miner.etherbase 8e28DaDF2b8E3D8AD524d62457a5e3edc76abFEa --miner.sigfile key
````
###### Node2
````
./erigon --identity "node2" --datadir=data/starlink2 --chain starlink --private.api.addr 0.0.0.0:7090 \
         --http.addr=0.0.0.0 --http.port=7545 \
         --metrics --metrics.expensive --metrics.addr=0.0.0.0 --metrics.port=5060 \
         --pprof --pprof.addr=0.0.0.0 --pprof.port=5061 \
         --engine.addr=0.0.0.0 --engine.port=7551 \
         --ws --ws.compression \
         --healthcheck \
         --bootnodes "enode://2bdb95192a6d9501fef76a38bc3493cb9c3165dc5d926eaea3f4ded449f3bceeacb5e16b6f4c1dfdd1973a09f8850984e0202e4ce418c102e8240a61f8f63725@127.0.0.1:30303" \
         --port=20303 --torrent.port=32069
````
###### Node3
````
./erigon --identity "node3" --datadir=data/starlink3 --chain starlink --private.api.addr 0.0.0.0:6090 \
         --http.addr=0.0.0.0 --http.port=6545 \
         --metrics --metrics.expensive --metrics.addr=0.0.0.0 --metrics.port=4060 \
         --pprof --pprof.addr=0.0.0.0 --pprof.port=4061 \
         --engine.addr=0.0.0.0 --engine.port=6551  \
         --ws --ws.compression \
         --healthcheck \
         --bootnodes "enode://ad2f3118e55031615ea57e1353b72b287e210f19fce70e0c053dfbe1a2c38f5433257d03e1b9518f9c85907911af70bfe1183455211cbd0d21989033e148a771@127.0.0.1:30303" \
         --port=10303 --torrent.port=22069                                                                                                                                                             
````