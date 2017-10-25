# Cita Bootstrap

通过本文所述方法和项目中的脚本，我们可以快速的搭建好自己的Cita私链进行开发测试。

## 一、部署Cita

为了简化 `Cita` 的多机部署，帮助用户快速搭建 `Cita` 运行环境，我们推荐使用 `docker` 部署 `Cita`。

### 1. 安转docker

安装 `docker` 及 `docker-compose`

```
sudo apt-get install docker docker-compose  docker.io
```

国内访问 `hub.docker.com` 可采用镜像加速：

```
sudo cat > /etc/docker/daemon.json  << EOF
{
  "registry-mirror": [ "https://registry.docker-cn.com" ]
}
EOF

```

### 2. 获取Cita镜像

直接从 `hub.docker.com` 拉取

```
docker pull cryptape/play
```

有两种方式可以完成发布件构建，分别是二进制发布件构建和从源码编译生成发布件。

**(1) 从二进制发布件构建**

```
wget https://github.com/cryptape/cita/releases/cita-v0.1.0.tar.bz2
tar xf cita-v0.1.0.tar.bz2
cd cita
scripts/build_image_from_binary.sh 
```
完成后可以通过 `docker images` 找到 `cryptape/play` 的 `docker` 镜像。

**(2) 从源码编译生成发布件**

```
git clone http://github.com/cryptape/cita
cd cita
scripts/build_image_from_source.sh 
```
### 3. 使用 `docker-compose` 部署 `Cita`

##### 准备

```
mkdir cita
cd cita
wget https://raw.githubusercontent.com/cryptape/cita/develop/scripts/docker-compose.yaml

```

##### 生成配置数据

```
docker-compose run admin
```

##### 启动4个节点

```'
docker-compose up node0 node1 node2 node3
```

##### 关闭节点

```

# stop single node
docker-compose stop node0
# start single node
docker-compotse start node0
# stop all nodes
docker-compose down 

```

## 二、部署智能合约

Cita完全兼容以太坊的智能合约，`solidity` 是智能合约最为推荐的语言，因此我们也采用 `solidity` 语言来编写和测试智能合约。

### 1. 编译 `solidity` 文件，返回文件 `bin` 值

要想编译 `solidity`文件，你需要先安装编译器，`solidity` 智能合约可以通过多种方式进行编译

1. 通过在线 `solidity` 实时编译器来编译。具体地址为https://remix.ethereum.org/
2. 安装 `solc` 编译器编译
3. 在客户端 `javascript console` 中，通过 `web3.eth.compile.solidity` 编译。

本文采用第二种方式，`solc` 编译器是一个来自 `C++` 客户端实现的组件，安装方法请参考http://www.ethdocs.org/en/latest/ethereum-clients/cpp-ethereum/index.html。

安装完成后，在Terminal中执行`solc --version`，如果返回值为：

```
solc, the solidity compiler commandline interface
Version: 0.4.18+commit.9cf6e910.Linux.g++
```
表示安装成功，接下来就可以使用 `solc` 命令编译 `solidity` 文件了。

在`Terminal`执行： 

```
cd $(DIR)/cita/cita/scripts/contracts/tests    // 其中`$(DIR)`代表你的工程目录
solc test_example.sol --bin
```
 
如果文件没有错误，返回结果中将会包括 `test_example.sol` 的二进制代码，这个数值就是 `Cita` 链上该智能合约的唯一标示值。

### 2. 部署合约，发送者需要构建合约权限

得到 `solidity` 文件二进制代码后，就可以将其部署到 `Cita` 链上了，部署的方法已经用 `python` 脚本封装，只需要传入私钥和二进制代码即可。在 `Terminal` 执行以下命令：

```
python make_tx.py --privkey "352416e1c910e413768c51390dfd791b414212b7b4fe6b1a18f58007fa894214" --code "606060405234156100105760006000fd5b610015565b60e0806100236000396000f30060606040526000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806360fe47b114604b5780636d4ce63c14606c576045565b60006000fd5b341560565760006000fd5b606a60048080359060200190919050506093565b005b341560775760006000fd5b607d60a3565b6040518082815260200191505060405180910390f35b8060006000508190909055505b50565b6000600060005054905060b1565b905600a165627a7a72305820942223976c6dd48a3aa1d4749f45ad270915cfacd9c0bf3583c018d4c86f9da20029"
```

参数解释： `code` 为第一步中获得的 `bin` 值, `privkey` 可随意获取

> 目前支持的 `python` 版本是2.7， 如果执行过程中报 `module` 找不到，可直接安装该 `module` 即可，具体命令为 `sudo python2.7 -m pip install pathlib`

### 3. 通过 `python` 脚本发送交易命令

在 `Terminal` 中执行：  `python send_tx.py`

结果如下: 

```
{
	"jsonrpc":"2.0",
    "id":1,
    "result":
    {
    	"hash":"0xa02fa9a94de11d288449ccbe8c5de5916116433b167eaec37455e402e1ab53d3",
    	"status":"Ok"
    }
}

```
`status`为`OK`，表示合约已经发送到 `Cita` 链上。

### 4. 获得来自 `Cita` 区块链网络的回执

在 `Terminal` 中执行： `python get_receipt.py`

结果如下:

```
{
  "contractAddress": "0x73552bc4e960a1d53013b40074569ea05b950b4d",          // 合约地址
  "cumulativeGasUsed": "0xafc8",
  "logs": [],
  "blockHash": "0x14a311d9f026ab592e6d156f2ac6244b153816eeec18717802ee9e675f0bfbbd",
  "transactionHash": "0x61854d356645ab5aacd24616e59d76ac639c5a5c2ec79292f8e8fb409b42177b",
  "root": null,
  "errorMessage": null,
  "blockNumber": "0x6",                                                     // 区块高度
  "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  "transactionIndex": "0x0",
  "gasUsed": "0xafc8"
}
```
这里需要重点关注 `contractAddress` 和 `blockNumber`，下文调用合约方法会用到。

### 5. 获得合约文件方法的 `hash` 值

合约的调用是通过发送交易命令完成，调用具体的方法则是通过方法hash值完成

在 `Terminal` 中执行： `solc test_example.sol --hash`

结果如下:

```
======= example.sol:SimpleStorage =======
Function signatures: 
6d4ce63c: get()                         // get方法hash值
60fe47b1: set(uint256)                  // set方法hash值 
```
这里的get和set方法hash值是Cita链上的唯一标示值，下文调用合约方法会用到。

### 6. 调用合约文件中的set方法

假定我们调用set方法，参数为1，也就是说将数值1存储到区块链内存中

在 `Terminal` 中执行：

```
python make_tx.py --privkey "352416e1c910e413768c51390dfd791b414212b7b4fe6b1a18f58007fa894214" --to "73552bc4e960a1d53013b40074569ea05b950b4d" --code "60fe47b10000000000000000000000000000000000000000000000000000000000000001"
```

privkey是你的私钥，to参数是合约的目标地址，code参数是set方法和参数hash值的拼接，set方法的hash值为60fe47b1，将参数1转换为uint256，转换成16进制就是64位。

### 7. 通过 `python` 脚本发送交易命令

在 `Terminal` 中执行： `python sen_tx.py`

```
{ 
	"jsonrpc":"2.0",
    "id":1,
    "result":
    {
    	"hash":"0xf29935d0221cd8ef2cb6a265e0a963ca172aca4f6e43728d2ccae3127631d590",
    	"status":"Ok"
    }
}

```

### 8. 获得来自 `Cita` 区块链网络的回执

在 `Terminal` 中执行： `python get_receipt.py`

结果如下：

```
{
  "contractAddress": null,
  "cumulativeGasUsed": "0x4f2d",
  "logs": [],
  "blockHash": "0x2a10ae38be9e1816487dbfb34bce7f440d60035e8978146caef5d14608bb222c",
  "transactionHash": "0xf29935d0221cd8ef2cb6a265e0a963ca172aca4f6e43728d2ccae3127631d590",
  "root": null,
  "errorMessage": null,
  "blockNumber": "0x15",                    // 区块的高度
  "logsBloom": "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  "transactionIndex": "0x0",
  "gasUsed": "0x4f2d"
}
```

### 9.发送 `eth_call Post` 请求，验证合约执行效果

调用合约中的 `get` 方法，验证之前 `set` 方法的执行效果

```
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_call", "params":[{"to":"0x73552bc4e960a1d53013b40074569ea05b950b4d", "data":"0x6d4ce63c"}, "0x15"],"id":2}' 127.0.0.1:1337
```

其中`0x15`为以上命令获取的`blockNumber`， `to` 参数为合约目标地址， `data` 为 `get` 方法的 `hash` 值

结果如下：

```
{
	"jsonrpc":"2.0",
    "id":2,
    "result":"0x0000000000000000000000000000000000000000000000000000000000000001"
}

```

如果返回值中 `result` 值为1，表明合约调用生效

























