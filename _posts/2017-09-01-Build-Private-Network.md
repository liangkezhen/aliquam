---
layout: post
title:  搭建以太坊调试环境 
category: tech 
comments: true
description: 
tags:
    - blockchain 
---

最近一直在看以太坊的白皮书，然而36页的文档却看了后面忘了前面，花费了时间和精力，效果却并不好，这真是一件让人感到沮丧的事情。对于学习这件事，我们古人可是总结了不少有益的经验，比如古诗有云“纸上得来终觉浅，绝知此事要躬行”，想要充分理解以太坊的运行机制，单单看文档是不太现实的。于是今天决心放下白皮书，自己动手搭建一个测试环境，在战争中学习战争。

闲话少叙，直入主题。根据一个下午google， 成功搭建了一个只有一个节点的以太坊私网，并且挖出了个人定制的ETH，还是挺有成就感的。当年比特币刚刚出来的时候，中本聪自己给自己印钱玩的时候，大概也是这种感觉吧。

先看个结果吧，下面就是我的账户和账户余额
```
> Welcome to the Geth JavaScript console!
> 
> instance: Geth/v1.6.7-stable-ab5646c5/darwin-amd64/go1.7.1
> coinbase: 0x2324bc069a60f756c52084f98075018332ffbab0
> at block: 1144 (Sat, 02 Sep 2017 21:26:47 CST)
>  datadir: /Users/philiang/privatechain
>  modules: admin:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0 web3:1.0
>  
> 
>  eth.accounts
> 
> ["0x2324bc069a60f756c52084f98075018332ffbab0"]
>  
> eth.getBalance("0x2324bc069a60f756c52084f98075018332ffbab0")
>    
> 5.72e+21
```
总结一下，整个步骤包括以下几步：

1. 安装geth
2. 创建自己定制的创世文件 custom_genesis.json
3. 初始化自己的创世区块和private 网络
4. 创建账户
5. 修改custom_genesis.json
6. 开始挖矿
7. 检查成果


## 1 安装geth

Geth 是以太坊官方提供的客户端之一，用Go 语言写成。其他客户段当然也可以用于学习以太坊。我选择geth 的原因很简单，Go 语言是21 世纪的 C 语言。

在MAC 上执行如下语句即可开始安装，安装完成后，检查geth version.

```
brew tap ethereum/ethereum

brew install ethereum
``` 

````
host-mbp:~ host$ geth version
Geth
Version: 1.6.7-stable
Git Commit: ab5646c532292b51e319f290afccf6a44f874372
Architecture: amd64
Protocol Versions: [63 62]
Network Id: 1
Go Version: go1.7.1
Operating System: darwin
GOPATH=
GOROOT=/usr/local/Cellar/go/1.7.1/libexec
````
## 2 初始化 custom_genesis.json

genesis.json 文件定义了区块链一些初始数据。现在不必关心其中每一项的具体含义，只需要知道他定制了一个独有的区块链即可。刚开始，我在网络上参考了不少genesis.json 文件，包括以太坊官方的范例文件，但总是不能成功初始化，后来google 到一个类似的问题，在其中增加了configure 项，问题解决。这个文件是自己创建的，用vim 创建即可，路径没有特殊要求。注意，该文件中的alloc 项，在开始创建文件时是空的，其中的内容表示区块链建立时特定的 {账户：余额}

```
{
    "config": {
        "chainId": 15, 
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },  
    "difficulty": "0x4000",
    "gasLimit": "2100000",
    "alloc": {
        "0x2324bc069a60f756c52084f98075018332ffbab0" :{"balance":"300000"}
    }   
}
```

## 3 初始化自己的创始区块和private 网络


这里需要说一下，如果直接在自己的shell 上执行geth 命令，会开始连接以太坊公网，并且从公网上同步数据。因此，要区分公网和私网，首先数据要分开存放，其次私网要有自己独特的 ID。 geth 的--datadir 指定了私网数据的存放目录，--networkid 指定了私网的id。命令的具体含义可以通过 geth -help 查询。

```
geth --datadir "./privatechain" --networkid 54321 init custom_genesis.json
```

如果私网初始化成功，进一步通过console 进入client 调试界面

```
geth --datadir "./privatechain" --networkid 54321 console
```

## 4 创建账户

```
> personal.newAccount('ethfans')

INFO [09-02|22:50:19] New wallet appeared                      
url=keystore:///Users/philiang/priv… status=Locked
"0xeea998eb31f32f12f6d9a7253f9d28aa0b7fe0cd"
```
检查当前账户，发现刚刚生成的账户已经存在了

```
> eth.accounts
["0x2324bc069a60f756c52084f98075018332ffbab0", 
"0xeea998eb31f32f12f6d9a7253f9d28aa0b7fe0cd"]
> 
```
## 5 修改custom_genesis.json 中的alloc项
## 6 开始挖矿

命令miner.start() 开始挖矿进程, 命令miner.stop()关闭挖矿进程

```
> miner.start()
INFO [09-02|22:55:31] Updated mining threads                   threads=0
INFO [09-02|22:55:31] Transaction pool price threshold updated price=18000000000
INFO [09-02|22:55:31] Starting mining operation 
null
> INFO [09-02|22:55:31] Commit new mining work                   number=1145 txs=0 uncles=0 elapsed=218.132µs
INFO [09-02|22:55:31] Successfully sealed new block            number=1145 hash=051d08…7b73f7
INFO [09-02|22:55:31] 🔨 mined potential block                  number=1145 hash=051d08…7b73f7
INFO [09-02|22:55:31] Commit new mining work                   number=1146 txs=0 uncles=0 elapsed=130.34µs
INFO [09-02|22:55:32] Successfully sealed new block            number=1146 hash=39dc54…b9e13f
INFO [09-02|22:55:32] 🔨 mined potential block                  number=1146 hash=39dc54…b9e13f
INFO [09-02|22:55:32] Commit new mining work                   number=1147 txs=0 uncles=0 elapsed=105.84µs
INFO [09-02|22:55:36] Successfully sealed new block            number=1147 hash=2a7561…ac2f29
INFO [09-02|22:55:36] 🔨 mined potential block                  number=1147 hash=2a7561…ac2f29
INFO [09-02|22:55:36] Commit new mining work                   number=1148 txs=0 uncles=0 elapsed=135.99µs
INFO [09-02|22:55:36] Successfully sealed new block            number=1148 hash=5928c1…7dd885
INFO [09-02|22:55:36] 🔨 mined potential block                  number=1148 hash=5928c1…7dd885
INFO [09-02|22:55:36] Commit new mining work                   number=1149 txs=0 uncles=0 elapsed=99.261µs
INFO [09-02|22:55:37] Successfully sealed new block            number=1149 hash=30fad8…75eae3
INFO [09-02|22:55:37] 🔨 mined potential block                  number=1149 hash=30fad8…75eae3
INFO [09-02|22:55:37] Commit new mining work                   number=1150 txs=0 uncles=0 elapsed=119.516µs
```

## 7 检查成果

当前的私网中有两个账户，挖出来的币默认都是放到第一个账户中去了，第二个账户balance 为0.

```
> eth.getBalance("0x2324bc069a60f756c52084f98075018332ffbab0")
5.8e+21
> eth.getBalance("0xeea998eb31f32f12f6d9a7253f9d28aa0b7fe0cd")
0
```

## 参考文档

http://ethfans.org/topics/85

http://www.debugrun.com/a/81AOucw.html

https://github.com/ethereum/go-ethereum/wiki/Connecting-to-the-network#custom-networks

https://medium.com/taipei-ethereum-meetup/以太坊私網建立-一-43f8677fc9f8


https://lightrains.com/blogs/setup-local-ethereum-blockchain-private-testnet

https://github.com/Musicoin/go-musicoin/issues/18

https://ethereum.stackexchange.com/questions/23594/fatal-error-while-initializing-node-from-custom-genesis-file-cannot-unmarshal

