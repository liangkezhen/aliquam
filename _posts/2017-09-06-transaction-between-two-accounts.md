---
layout: post
title:  智能合约简单交易 
category: tech 
comments: true
description: 
tags:
    - blockchain 
---


在我的私有链上，有两个账户，其中第一个账户是default 账户（accout[0]），挖矿所得全部进入这个账户。第二个账户(accout[1])没有第一个账户的特权，我需要通过交易的方式从第一个账户转一笔钱到第二个账户，那么这个交易如何进行呢？

首先，检查两个账户的余额，不幸的账户[1] 余额为0

```
> eth.getBalance(eth.accounts[1])
0
> eth.getBalance(eth.accounts[0])
1.87e+21
```

在以太坊中，转账是要收取费用的，这个费用用于激励挖矿的矿工，称之为GAS。 那么这笔转账需要支付多少gas 呢 ？

```
> eth.estimateGas({from:eth.accounts[0], to: eth.accounts[2], value:50000000000000})
53001
> eth.gasPrice
18000000000
```

转账，与现实生活类似，转出者需要输入密码，否则会告知账户被锁定，无法转账。

```
> eth.sendTransaction({from:eth.accounts[0], to: eth.accounts[1], value:50000000000000})
Error: authentication needed: password or unlock
    at web3.js:3104:20
    at web3.js:6191:15
    at web3.js:5004:36
    at <anonymous>:1:1

> personal.unlockAccount(eth.accounts[0],"philiang")
true
```

输入密码，解锁账户后，再次开始转账, 此时立即查询accout[1]的账户，会发现并没有到账，原因是需要挖矿将该交易确认下来，形成不可更改的记账。

```
> eth.sendTransaction({from:eth.accounts[0], to: eth.accounts[1], value:50000000000000})
INFO [09-05|19:11:00] Submitted transaction                    fullhash=0x024cb8e81d92d72128303a9396d96965ea9f848ae85e60c8bfe9980672692134 recipient=0x813f09b78e60a69e6373d9a3410b8d3274d7c0f3
"0x024cb8e81d92d72128303a9396d96965ea9f848ae85e60c8bfe9980672692134"
> 
> 
> 
> eth.getBalance(eth.accounts[1])
0
> eth.getBalance(eth.accounts[0])
1.87e+21
```

那么通过挖矿记录这笔交易吧

```
> miner.start()
INFO [09-05|19:11:36] Updated mining threads                   threads=0
INFO [09-05|19:11:36] Transaction pool price threshold updated price=18000000000
INFO [09-05|19:11:36] Starting mining operation 
null
> INFO [09-05|19:11:36] Commit new mining work                   number=375 txs=1 uncles=0 elapsed=397.596µs
INFO [09-05|19:11:36] Successfully sealed new block            number=375 hash=6a1fb4…8db268
INFO [09-05|19:11:36] 🔗 block reached canonical chain          number=370 hash=3f3ad2…1a0729
       
```

再次查询两个账户的余额,account[1]中现在有了一笔余额。

```


> eth.getBalance(eth.accounts[0])
1.89999995e+21
> eth.getBalance(eth.accounts[1])
50000000000000
> 
> 
```
