搭建PoA以太坊私有链
---
##### 安装Geth及Tools
[Geth各版本下载地址](https://geth.ethereum.org/downloads/)
````
wget -c https://gethstore.blob.core.windows.net/builds/geth-alltools-linux-amd64-1.10.22-unstable-c4cd632f.tar.gz
tar -zxvf geth-alltools-linux-amd64-1.10.22-unstable-c4cd632f.tar.gz
cd geth-alltools-linux-amd64-1.10.22-unstable-c4cd632f
sudo cp ./geth /usr/local/bin/geth
sudo cp ./puppeth /usr/local/bin/puppeth
sudo cp ./abigen /usr/local/bin/abigen
sudo cp ./bootnode /usr/local/bin/bootnode
sudo cp ./evm /usr/local/bin/evm
sudo cp ./rlpdump /usr/local/bin/rlpdump
````
##### 创建私有链工作文件夹
````
mkdir private-chain
cd private-chain
mkdir -p node1 node2 node3
创建各节点账户
sudo geth --datadir node1 account new
echo "111111" >> node1/password.txt //保存账号秘密到文件中
sudo geth --datadir node2 account new
echo "111111" >> node2/password.txt
sudo geth --datadir node3 account new
echo "111111" >> ./node3/password.txt

//保存各账户地址方便后续使用
node1: 0x0c83EA416af45dAD2d07eBA97b2ca858DCa8d422
node2: 0xAa9c13c86aCC58cf51D38c0f12c7484F30E2A81E
node3: 0x08514392aD3af21C170f5F5408c2d0114897aBf7
````

##### 使用puppeth工具生成创世块文件

````
ubuntu@ip-172-31-91-110:~$ puppeth
+-----------------------------------------------------------+
| Welcome to puppeth, your Ethereum private network manager |
|                                                           |
| This tool lets you create a new Ethereum network down to  |
| the genesis block, bootnodes, miners and ethstats servers |
| without the hassle that it would normally entail.         |
|                                                           |
| Puppeth uses SSH to dial in to remote servers, and builds |
| its network components out of Docker containers using the |
| docker-compose toolset.                                   |
+-----------------------------------------------------------+

Please specify a network name to administer (no spaces, hyphens or capital letters please)
> amc

Sweet, you can set this via --network=amc next time!

INFO [08-11|03:09:26.498] Administering Ethereum network           name=amc
WARN [08-11|03:09:26.498] No previous configurations found         path=/home/ubuntu/.puppeth/amc

What would you like to do? (default = stats)  //设定新的创世块，这里选2
1. Show network stats
2. Configure new genesis
3. Track new remote server
4. Deploy network components
> 2

What would you like to do? (default = create)
1. Create new genesis from scratch
2. Import already existing genesis
> 1

Which consensus engine to use? (default = clique) //选择共识机制，选2，Clique PoA
1. Ethash - proof-of-work
2. Clique - proof-of-authority
> 2

How many seconds should blocks take? (default = 15) //设置生产出一个Block的时间，在这里设10 秒，可自拟
> 8

Which accounts are allowed to seal? (mandatory at least one) // 授权节点。这里使用上面的node1的address
> 0x0c83EA416af45dAD2d07eBA97b2ca858DCa8d422
> 0x

Which accounts should be pre-funded? (advisable at least one) // 给账户设置一些ether。这里选node1的address，可自拟
> 0x0c83EA416af45dAD2d07eBA97b2ca858DCa8d422 //再输入一遍node1的账户地址
> 0x

Should the precompile-addresses (0x1 .. 0xff) be pre-funded with 1 wei? (advisable yes)
> yes

Specify your chain/network ID if you want an explicit one (default = random) // Network Id，在这里设8848，可自拟，也可直接用random；添加genesis的，留空
> 8848 //这里输入一个大于2018的整数
INFO [08-11|03:11:44.833] Configured new genesis block

What would you like to do? (default = stats)
1. Show network stats
2. Manage existing genesis
3. Track new remote server
4. Deploy network components
> 2
// 存档，选2
1. Modify existing configurations
2. Export genesis configurations
3. Remove genesis configuration
> 2

Which folder to save the genesis spec into? (default = current)
Will create amc.json
> //回车保存genesis
INFO [08-12|13:34:27.792] Saved native genesis chain spec          path=amc.json

What would you like to do? (default = stats)
1. Show network stats
2. Manage existing genesis
3. Track new remote server
4. Deploy network components
   // Ctrl+D to exit
> CRIT [08-11|03:12:55.379] Failed to read user input                err=EOF
````
以上全部完成后会在当前路径下出现 amc.json文件，amc.json 是我们的genesis block配置信息文件。
##### 初始化各节点
````
sudo geth --datadir node1 init amc.json
sudo geth --datadir node2 init amc.json
sudo geth --datadir node3 init amc.json
````
##### 启动各节点
node1
````
sudo geth --networkid 8848 --datadir node1 --identity "node1" --syncmode full --port 30303 \
--http --http.addr "0.0.0.0" --http.port 8545 --http.corsdomain "*" --http.api "admin,debug,web3,eth,txpool,personal,clique,miner,net" \
--authrpc.addr "0.0.0.0" --authrpc.port 8551 --authrpc.vhosts "*" \
--mine --unlock 0 --password ./node1/password.txt --allow-insecure-unlock \
--pprof --pprof.addr "0.0.0.0" --pprof.port 6060 \
--metrics --metrics.addr "0.0.0.0" --metrics.port 6061 \
--snapshot=false \
console
````
node2
````
sudo geth --networkid 8848 --datadir node2 --identity "node2" --syncmode full --port 31303 \
--http --http.addr "0.0.0.0" --http.port 8645 --http.corsdomain "*" --http.api "admin,debug,web3,eth,txpool,personal,clique,miner,net" \
--authrpc.addr "0.0.0.0" --authrpc.port 8651 --authrpc.vhosts "*" \
--mine --unlock 0 --password ./node2/password.txt --allow-insecure-unlock \
--pprof --pprof.addr "0.0.0.0" --pprof.port 6160 \
--metrics --metrics.addr "0.0.0.0" --metrics.port 6161 \
--snapshot=false \
console
````
node3
````
sudo geth --networkid 8848 --datadir node3 --identity "node3" --syncmode full --port 32303 \
--http --http.addr "0.0.0.0" --http.port 8745 --http.corsdomain "*" --http.api "admin,debug,web3,eth,txpool,personal,clique,miner,net" \
--authrpc.addr "0.0.0.0" --authrpc.port 8751 --authrpc.vhosts "*" \
--mine --unlock 0 --password ./node3/password.txt --allow-insecure-unlock \
--pprof --pprof.addr "0.0.0.0" --pprof.port 6260 \
--metrics --metrics.addr "0.0.0.0" --metrics.port 6261 \
--snapshot=false \
console
````
````
//保存各节点地址方便后续使用
enode://ffc263e19aef0a1505e2b697de0344d7ffe0f79cada22196f938de0be9d0f2696fca3cce78f1795fd492b3d3885034c6e8747d20b80b84ab1a72abe234566d4d@127.0.0.1:30303
enode://1c470e9c12d30d32cdf57483146dcea5c20f38f01f91233f13a5066d744071594f30ef6dcee1e26fb24fdd71b599311e0793acb384f527a94e1b21f13e6a44d9@127.0.0.1:31303
enode://878a86e4e6b73dff3a0c4c7c8378bacc4ac13a273fe95b5cb16bc2e3f209fa0dbbc2518a1bbe184961e7fbd386813f1a61e604fb4c03f2b235c91790dd86a7e0@127.0.0.1:32303

需要注意的地方：
a）因为3个节点都运行在同一台电脑上，所以每个节点的port都要配置为不相同的，否则会造成启动节点时报错。
b）networkid 不一致的三个节点无法建立连接

这里有必要把一些参数解释一下，省得以后自己也忘了。
–networkid：我们创建的私有链的网络id
–datadir：节点的数据文件夹
–http：表示允许远程调用。这个参数以前叫rpc，后来在新版的Geth中改成http了，当然用rpc也可以，只不过这个参数很快就被丢弃了，还是早点改的好，这个可以在Geth的帮助中看到。
–http.port：表示允许远程调用的端口。默认是8545。这个参数以前是rpcport。
–http.addr：把这个值写成“0.0.0.0”表示允许远程访问，否则只能本地访问。这个参数以前是rpcaddr。
–port：表示网络监听端口，默认值是30303。
–http.corsdomain：允许跨域请求的域列表，这里指定为“*”。这个参数以前是rpccorsdomain
–http.api：允许远程调用的API，用逗号间隔，凡是列出来的，在远程调用时均可以使用。
–unlock：表示被解锁账户的编号，0表示node数据文件中第一个被创建的账户，这个账户被解锁才能使用该账户进行交易。
–password：表示解锁账户时的账户密码，就是在创建账户时输入的密码。
–allow-insecure-unlock：允许使用不安全的账户解锁（？？？翻译的不好，大概是这个意思吧）。
console：表示打开Geth JavaScript console。
````
````
在node2, node3 的geth console 界面下分别运行如下指令：
admin.addPeer("enode://d82177203847b752899e014bf6714b8716eb9b5f5de614609ef51c9d45017b43dbe0c94447fd4c0c050555f46d346b72f1c20f7e9a4af22f393607f8bd4a2144@219.143.174.248:30303")
链接成功后，三个节点都可以通过admine.peers查看到对方的节点信息，节点二，三就会开始同步节点一的区块，同步完成后，任意一个节点开始挖矿，其他节点会自动同步区块，向任意一个节点发送交易，其他节点也会收到该笔交易。
然后在节点上测试，
net.peerCount // 会返回已连接的其他节点的个数
admin.peers // 返回其他节点的信息

进行交易
在node1的console下，使用指令eth.getBalance("<Address>")来查询账户余额。
eth.getBalance("0x0c83EA416af45dAD2d07eBA97b2ca858DCa8d422")
使用eth.sendTransaction指令来把一些ether从node1转到node2，node3
eth.sendTransaction({from: eth.coinbase, to: "0xAa9c13c86aCC58cf51D38c0f12c7484F30E2A81E", value:web3.toWei(100,"ether")})
eth.sendTransaction({from: eth.coinbase, to: "0x08514392aD3af21C170f5F5408c2d0114897aBf7", value:web3.toWei(100,"ether")})
查看node2和node3的余额
eth.getBalance("0xaa9c13c86acc58cf51d38c0f12c7484f30e2a81e")
eth.getBalance("0x08514392aD3af21C170f5F5408c2d0114897aBf7")

clique.propose('0xaa9c13c86acc58cf51d38c0f12c7484f30e2a81e',true)
clique.propose('0x08514392aD3af21C170f5F5408c2d0114897aBf7',true)
eth.sendTransaction({from: eth.coinbase, to: "0x8e28DaDF2b8E3D8AD524d62457a5e3edc76abFEa", value:web3.toWei(10000000,"ether")})
eth.getBalance("0x8e28DaDF2b8E3D8AD524d62457a5e3edc76abFEa")
````
##### 以后台的方式启动
node1
````
ps -ef | grep node1 | grep -v grep | awk '{print "kill -9 "$2}'
ps -ef | grep node1 | grep -v grep | awk '{print "sudo kill -9 "$2}' | sh
nohup sudo geth --networkid 8848 --datadir node1 --identity "node1" --syncmode full --port 30303 \
--http --http.addr "0.0.0.0" --http.port 8545 --http.corsdomain "*" --http.api "admin,debug,web3,eth,txpool,personal,clique,miner,net" \
--authrpc.addr "0.0.0.0" --authrpc.port 8551 --authrpc.vhosts "*" \
--mine --unlock 0 --password ./node1/password.txt --allow-insecure-unlock \
--pprof --pprof.addr "0.0.0.0" --pprof.port 6060 \
--metrics --metrics.addr "0.0.0.0" --metrics.port 6061 \
--snapshot=false \
> log/node1.log 2>&1 &
````
node2
````
ps -ef | grep node2 | grep -v grep | awk '{print "kill -9 "$2}'
ps -ef | grep node2 | grep -v grep | awk '{print "sudo kill -9 "$2}' | sh
nohup sudo geth --networkid 8848 --datadir node2 --identity "node2" --syncmode full --port 31303 \
--http --http.addr "0.0.0.0" --http.port 8645 --http.corsdomain "*" --http.api "admin,debug,web3,eth,txpool,personal,clique,miner,net" \
--authrpc.addr "0.0.0.0" --authrpc.port 8651 --authrpc.vhosts "*" \
--mine --unlock 0 --password ./node2/password.txt --allow-insecure-unlock \
--pprof --pprof.addr "0.0.0.0" --pprof.port 6160 \
--metrics --metrics.addr "0.0.0.0" --metrics.port 6161 \
--snapshot=false \
--bootnodes "" \
> log/node2.log 2>&1 &
````
node3
````
ps -ef | grep node3 | grep -v grep | awk '{print "kill -9 "$2}'
ps -ef | grep node3 | grep -v grep | awk '{print "sudo kill -9 "$2}' | sh
nohup sudo geth --networkid 8848 --datadir node3 --identity "node3" --syncmode full --port 32303 \
--http --http.addr "0.0.0.0" --http.port 8745 --http.corsdomain "*" --http.api "admin,debug,web3,eth,txpool,personal,clique,miner,net" \
--authrpc.addr "0.0.0.0" --authrpc.port 8751 --authrpc.vhosts "*" \
--mine --unlock 0 --password ./node3/password.txt --allow-insecure-unlock \
--pprof --pprof.addr "0.0.0.0" --pprof.port 6260 \
--metrics --metrics.addr "0.0.0.0" --metrics.port 6261 \
--snapshot=false \
--bootnodes "" \
> log/node3.log 2>&1 &
````
##### linux连接控制台
````
sudo geth attach ipc:node1/geth.ipc
sudo geth attach ipc:node2/geth.ipc
sudo geth attach ipc:node3/geth.ipc
````
````
配置默认账户
eth.defaultAccount = eth.coinbase

查询账户余额
eth.getBalance(eth.coinbase)

解锁账户 地址，密码，时间
personal.unlockAccount(eth.coinbase,"123456",150000)

转账
eth.sendTransaction({from: eth.coinbase, to: "0x8e28DaDF2b8E3D8AD524d62457a5e3edc76abFEa", value:web3.toWei(10000,"ether")})

查看自身节点的peer信息
admin.nodeInfo

添加节点
admin.addPeer("enode://ffc2...6d4d@127.0.0.1:30303")

查看当前节点和其它哪些节点联系上
admin.peers

会返回已连接的其他节点的个数
net.peerCount

增加投票者（51%的客户端执行才能成功）,必须在授权名单内的节点的console界面下，将新的节点加入。输入指令：
clique.propose('0x8e28DaDF2b8E3D8AD524d62457a5e3edc76abFEa',true)
````
##### 参考链接：
https://blog.csdn.net/ursamjnor/article/details/109709695  
https://blog.csdn.net/code_segment/article/details/78160660  
https://blog.csdn.net/inthat/article/details/109697760  
https://blog.csdn.net/zhj_fly/article/details/79933799  
https://blog.csdn.net/powervip/article/details/107225345  
http://t.zoukankan.com/ahuo-p-9364958.html  