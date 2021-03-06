# 1. 万维链星系共识入门手册

# 2. 概述

参考入门手册可帮助您快速的接入星系共识，参与挖矿或代理。

参与者主要分为验证人和委托人两种角色。

验证人需要准备合适的节点服务器，在矿工模式下运行，执行星系共识协议，并参与挖矿。

委托人不需要运行矿工节点，可以选择投注到验证人，分享奖励。

Alpha版的explorer地址为：

(待发布)

后续功能会陆续更新。

# 3. 目录

<!-- TOC -->

- [1. 万维链星系共识入门手册](#1-万维链星系共识入门手册)
- [2. 概述](#2-概述)
- [3. 目录](#3-目录)
- [4. 通过Docker启动节点](#4-通过docker启动节点)
    - [4.1. 成为验证人](#41-成为验证人)
    - [4.2. 成为委托人](#42-成为委托人)
- [5. 安装和运行教程](#5-安装和运行教程)
    - [5.1. 使用代码编译运行](#51-使用代码编译运行)
    - [5.2. 运行教程](#52-运行教程)
        - [5.2.1. 同步节点](#521-同步节点)
        - [5.2.2. 验证节点（矿工）](#522-验证节点矿工)
- [6. 常用操作](#6-常用操作)
    - [6.1. 账号创建](#61-账号创建)
    - [6.2. 查询余额](#62-查询余额)
    - [6.3. 获取测试币](#63-获取测试币)
    - [6.4. Stake注册和代理](#64-stake注册和代理)
- [7. 测试数据](#7-测试数据)

<!-- /TOC -->

# 4. 通过Docker启动节点

## 4.1. 成为验证人

1） 安装 docker (Ubuntu):
```
$ sudo wget -qO- https://get.docker.com/ | sh

$ sudo usermod -aG docker YourUserName

$ exit
```

2）使用docker中的gwan创建keystore账号:
```
$ docker pull wanchain/wanpos

$ docker run -d -v /home/YourUserName/.wanchain:/root/.wanchain wanchain/wanpos /bin/gwan --pluto

YourContainerID

$ docker exec -it YourContainerID /bin/bash

root> gwan attach .wanchain/pluto/gwan.ipc

> personal.newAccount('YourPassword')

"YourAccountAddress"

> personal.showPublicKey("YourAccountAddress", 'YourPassword')

["YourPK1", "YourPK2"]

> exit

root> echo "YourPassword" > /root/.wanchain/pw.txt

root> exit

```

![img](./img_get_start/1.png)

3）为您的测试账号申请测试币 "YourAccountAddress", 例如申请： 100100个万币。

可参考 [6.3. Get test wan coins of PoS](#63-get-test-wan-coins-of-pos) 教程申请万币。

在收到万币后，再执行第4步。

![img](./img_get_start/4.png)

4） 创建一个矿工注册脚本文件: `/home/YourUserName/.wanchain/minerRegister.js`

```
//minerRegister.js

// If you want to register to be a miner you can modify and use this script to run.


//-------INPUT PARAMS YOU SHOULD MODIFY TO YOURS--------------------

// tranValue is the value you want to stake in minValue is 100000 
var tranValue = "100000"

// secpub is the miner node's secpub value
var secpub    = "YourPK1"

// g1pub is the miner node's g1pub value
var g1pub     = "YourPK2"

// feeRate is the delegate dividend ratio if set to 10000, means it's a single miner do not accept delegate in.
// range 0~10000 means 0%~100%
var feeRate   = 1000

// lockTime is the time for miner works which measures in epoch count. And must >= 7.
var lockTime  = 30

// baseAddr is the fund source account.
var baseAddr  = "YourAccountAddress"

// passwd is the fund source account password.
var passwd    = "YourPassword"

//-------INPUT PARAMS YOU SHOULD MODIFY TO YOURS--------------------


//------------------RUN CODE DO NOT MODIFY------------------
personal.unlockAccount(baseAddr, passwd)
var cscDefinition = [{"constant":false,"inputs":[{"name":"addr","type":"address"},{"name":"lockEpochs","type":"uint256"}],"name":"stakeUpdate","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"addr","type":"address"}],"name":"stakeAppend","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"constant":false,"inputs":[{"name":"secPk","type":"bytes"},{"name":"bn256Pk","type":"bytes"},{"name":"lockEpochs","type":"uint256"},{"name":"feeRate","type":"uint256"}],"name":"stakeIn","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"constant":false,"inputs":[{"name":"delegateAddress","type":"address"}],"name":"delegateIn","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"constant":false,"inputs":[{"name":"delegateAddress","type":"address"}],"name":"delegateOut","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"}];


var contractDef = eth.contract(cscDefinition);
var cscContractAddr = "0x00000000000000000000000000000000000000d8";
var coinContract = contractDef.at(cscContractAddr);

var payload = coinContract.stakeIn.getData(secpub, g1pub, lockTime, feeRate)
var tx = eth.sendTransaction({from:baseAddr, to:cscContractAddr, value:web3.toWin(tranValue), data:payload, gas: 200000, gasprice:'0x' + (200000000000).toString(16)});
console.log("tx=" + tx)
//------------------RUN CODE DO NOT MODIFY------------------

```
![img](./img_get_start/2.png)


![img](./img_get_start/3.png)

脚本中的FeeRate字段为验证人接受委托投注时的分红比例。如果设为100，则表示不接受委托。

设为10，则表示验证人独享总收益的10%后，再与委托人按金额比例分红。

5） 在gwan中执行脚本

如果第二步的docker没有关闭，可以直接按下述代码进入执行，如果已关闭，请再启动起来: 

```
$ docker exec -it YourContainerID /bin/gwan attach .wanchain/pluto/gwan.ipc

> loadScript("/root/.wanchain/minerRegister.js")

> exit

$ docker stop YourContainerID

$ docker run -d -p 17717:17717 -p 17717:17717/udp -v /home/YourUserName/.wanchain:/root/.wanchain wanchain/wanpos /bin/gwan --pluto --etherbase "YourAccountAddress" --unlock "YourAccountAddress" --password /root/.wanchain/pw.txt --mine --minerthreads=1 

```

执行完上述脚本，即可完成开启挖矿运行。

可通过:
```
docker logs -f `docker ps -q`
```
命令查看工作日志。

挖矿工作，将在所有块同步完成后正式开始。

![img](./img_get_start/5.png)


![img](./img_get_start/6.png)

## 4.2. 成为委托人

在Beta版本发布后，可通过轻钱包方便的完成委托投注。

在Beta版本之前，可按照如下命令执行投注。

1）安装 docker (Ubuntu):
```
$ sudo wget -qO- https://get.docker.com/ | sh

$ sudo usermod -aG docker YourUserName

$ exit
```

2）创建账号，查找验证人信息:

验证人信息可以通过命令行查找，也可以通过浏览器查找。请注意，在使用pos.getStakerInfo获取验证节点信息前，请确认当前已经同步到最新块。可通过eth.blockNumber来查看。

```
$ docker run -d -v /home/YourUserName/.wanchain:/root/.wanchain wanchain/wanpos /bin/gwan --pluto

YourContainerID

$ docker exec -it YourContainerID /bin/bash

root> gwan attach .wanchain/pluto/gwan.ipc

> personal.newAccount('YourPassword')

"YourAccountAddress"

> pos.getStakerInfo(eth.blockNumber)
[
	{...},
	{...},
	{	Address: "DelegateAddress",
    Amount: 2e+23,
    Clients: [],
    FeeRate: 10,
    From: "...",
    LockEpochs: 30,
    PubBn256: "...",
    PubSec256: "...",
    StakingEpoch: 117
	}
]
```

通过上述执行，得到本地账号 `YourAccountAddress` 和想要投注的具备理想`FeeRate`的验证人地址 `DelegateAddress`.

3）申请测试币（方法同上）


4）创建投注脚本 `/home/YourUserName/.wanchain/sendDelegate.js`

```
//sendDelegate.js

// If you want to send to a delegate you can modify and use this script to run.


//-------INPUT PARAMS YOU SHOULD MODIFY TO YOURS--------------------

// tranValue is the value you want to stake in minValue is 100
var tranValue = "100000"

// delegateAddr is the validator address.
var delegateAddr = "DelegateAddress"

// baseAddr is the fund source account.
var baseAddr  = "YourAccountAddress"

// passwd is the fund source account password.
var passwd    = "YourPassword"

//-------INPUT PARAMS YOU SHOULD MODIFY TO YOURS--------------------


//------------------RUN CODE DO NOT MODIFY------------------
personal.unlockAccount(baseAddr, passwd)
var cscDefinition = [{"constant":false,"inputs":[{"name":"addr","type":"address"},{"name":"lockEpochs","type":"uint256"}],"name":"stakeUpdate","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"addr","type":"address"}],"name":"stakeAppend","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"constant":false,"inputs":[{"name":"secPk","type":"bytes"},{"name":"bn256Pk","type":"bytes"},{"name":"lockEpochs","type":"uint256"},{"name":"feeRate","type":"uint256"}],"name":"stakeIn","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"constant":false,"inputs":[{"name":"delegateAddress","type":"address"}],"name":"delegateIn","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"constant":false,"inputs":[{"name":"delegateAddress","type":"address"}],"name":"delegateOut","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"}];


var contractDef = eth.contract(cscDefinition);
var cscContractAddr = "0x00000000000000000000000000000000000000d8";
var coinContract = contractDef.at(cscContractAddr);

var payloadDelegate = coinContract.delegateIn.getData(delegateAddr)
var tx2 = eth.sendTransaction({from:baseAddr, to:cscContractAddr, value:web3.toWin(tranValue), data:payloadDelegate, gas: 200000, gasprice:'0x' + (200000000000).toString(16)});
console.log("tx2=" + tx2)
//------------------RUN CODE DO NOT MODIFY------------------
```

5）在gwan中运行投注脚本

```
$ docker exec -it YourContainerID /bin/bash

root> gwan attach .wanchain/pluto/gwan.ipc

> loadScript("/root/.wanchain/sendDelegate.js")

```

委托人投注完成。


# 5. 安装和运行教程

## 5.1. 使用代码编译运行

需要提前安装和配置golang运行环境：https://golang.org/

配置环境变量 $GOPATH 和 $GOROOT:

从github获取最新代码：

```
$ mkdir -p $GOPATH/src/github.com/wanchain/

$ cd $GOPATH/src/github.com/wanchain/

$ git clone https://github.com/wanchain/go-wanchain.git

$ cd go-wanchain

$ git checkout posalpha

$ git pull

$ make
```

编译得到的gwan在此目录下： `build/bin/gwan`

## 5.2. 运行教程

可在如下不同角色下运行：

### 5.2.1. 同步节点

```
$ gwan --pluto --rpc --syncmode "full"
```

### 5.2.2. 验证节点（矿工）

在下面命令中请替换地址为您的个人地址 `0x8d8e7c0813a51d3bd1d08246af2a8a7a57d8922e` ，并替换 `/tmp/pw.txt` 为您地址的密码文本文件。

```
$ gwan --pluto --rpc --etherbase "0x8d8e7c0813a51d3bd1d08246af2a8a7a57d8922e" --unlock "0x8d8e7c0813a51d3bd1d08246af2a8a7a57d8922e" --password /tmp/pw.txt --rpc  --mine --minerthreads=1 --syncmode "full"
```

# 6. 常用操作

## 6.1. 账号创建


```
$ gwan --pluto account new
```


执行上述命令后，keystore文件会存储在默认目录 `~/.wanchain/pluto/keystore/` in Ubuntu 或者 `~/Library/Wanchain/pluto/keystore/` in Mac OS.

使用如下命令获取两个星系共识需要用到的公钥。

```
$ gwan --pluto account pubkeys 'Your Address' 'Your Password'
```

星系共识需要使用key1和key3，作为SecPk和G1PK。

## 6.2. 查询余额


```
// In ubuntu
$ gwan attach ~/.wanchain/pluto/gwan.ipc

// In MacOS
$ gwan attach ~/Library/Wanchain/pluto/gwan.ipc

```

在链同步完成后，可通过下面指令查询余额。

```
$ eth.getBalance("Your Address Fill Here")

// Such as address example shown above.
$ eth.getBalance("0x8c35B69AC00EC3dA29a84C40842dfdD594Bf5d27")
```

## 6.3. 获取测试币

测试申请当前为邮件申请，后续会推出faucet网页，自动发放测试币。

| Index            | Email         | 
| --------------  | :------------  | 
|1| techsupport@wanchain.org| 



## 6.4. Stake注册和代理

用户注册一个节点服务器为星系共识验证节点（矿工）的步骤如下图所示：

![img](./img_get_start/99.png)

You can register as a mining node through Stake register.

We have given a smart contract for register and unregister.

Its contract interface is shown as below.
```
var cscDefinition = [{"constant":false,"inputs":[{"name":"addr","type":"address"},{"name":"lockEpochs","type":"uint256"}],"name":"stakeUpdate","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"addr","type":"address"}],"name":"stakeAppend","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"constant":false,"inputs":[{"name":"secPk","type":"bytes"},{"name":"bn256Pk","type":"bytes"},{"name":"lockEpochs","type":"uint256"},{"name":"feeRate","type":"uint256"}],"name":"stakeIn","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"constant":false,"inputs":[{"name":"delegateAddress","type":"address"}],"name":"delegateIn","outputs":[],"payable":true,"stateMutability":"payable","type":"function"},{"constant":false,"inputs":[{"name":"delegateAddress","type":"address"}],"name":"delegateOut","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"}]
```

在智能合约的参数中, `feeRate` 是接受委托后的分红费率，如果设置为10000，则为独立节点，不接受委托。

如果您想成为可以接受委托的验证节点，则应设置友好的费率值 `feeRate` 来吸引用户投注。

`feeRate`取值范围[ 0, 10000 ]. 表示分红的百分比0.00% ~ 100.00%.

可以直接修改代码目录下的脚本文件来完成矿工注册 `loadScript/minerRegister.js`.

委托人投注脚本`loadScript/sendDelegate.js`

脚本可在IPC链接到节点后执行。

```
// This path is a relative path for your run.
$ loadScript('loadScript/register.js')
```

收益信息，收益率预测，可在浏览器中查询。

# 7. 测试数据

下面展示了使用不同的锁定金额，不同的锁定时间的收益实测数据。

锁定时间单位是epoch。

测试时，epoch时间为20分钟，每个epoch有120个slot。6个epoch为120分钟。

（注意！本测试不是在alpha测试网络开展的，是在少量节点的私链上进行，alpha网络的时间参数为每个epoch 2天，每个epoch有1440*12个slot）

总stake约为 6000000 ~ 8000000 万币。

| Address     | stake | locktime | ep 1| ep 2 | ep 3 | ep 4 | ep 5 | total incentive |
| ----------  | ---- | :---: | --- | --- | ----| ---- | ---- | ---- | 
|0xbec1f01f5cbe494279a3c1455644a16aebfd700d| 100000 | 6 |0 |0.32 |1.07 |1.02 |1.94 | 4.35|
|0xa38c0aafc0b4ee45e006814e5769f17fda60f994| 200000 | 6 |0.32 |1.39 |4.40 |3.06 |2.33 |11.5 |
|0x711a9967d0b61ab92a86e14102de1233d3de5ead| 500000 | 6 |2.49 |6.03 |9.62 |10.32 |5.14 |33.6 | 
|0x52eee1ccb29adc742449a3e87fe7acaad605bd4c| 200000 | 12 |1.93 |4.81 |1.08 |1.17 |0.32 |9.31 |


如果收益为 0, 则意味着概率较低，没有被选中。


![img](./img_get_start/7.png)
