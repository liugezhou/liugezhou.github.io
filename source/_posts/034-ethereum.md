---
layout: post
title: 以太坊基础笔记
date: 2023-02-17
toc: true
image: https://cdn.staticaly.com/gh/liugezhou/image@master/web3/000_ether.png
category:
    Web3
tags:
    ethereum
---

![ethereum](https://cdn.staticaly.com/gh/liugezhou/image@master/web3/000_ether.png)
<!--more-->

这篇文章用来记录在学习以太坊的过程中,一些相关的基本概念和 solidity 语言的相关基础语法等。

# 以太坊基础
## 涉及工具

![tools](https://cdn.staticaly.com/gh/liugezhou/image@master/web3/001_tools.png)

## 区块链发展简史

![history](https://cdn.staticaly.com/gh/liugezhou/image@master/web3/002_history.png)

[EOS是什么](https://cloud.tencent.com/developer/news/175867)

## 以太坊中的重要概念

![concept](https://cdn.staticaly.com/gh/liugezhou/image@master/web3/003_concept.png)

## 以太坊的货币

- 以太坊 `ethereum` ,货币为 以太 `ether`
- 2014年7/8月 众筹发行大约 7200万以太币(矿前)，后经过挖矿产生，每年被限制不超过 7200万的25%，未来产量变化不确定。
- [以太币供应量查询](https://etherscan.io/stat/supply)

## 以太币单位

![wei](https://cdn.staticaly.com/gh/liugezhou/image@master/web3/004_wei.png)

# Solidity
## 变量

### 基本类型
- `int` (整型)
- `uint` (无符号整型)
- `bool` (布尔类型)
- `address` (地址类型)
- `string` (字符串)
- `byte` (字节)

### 引用类型
- `bytes32` (字节数组)
- `mapping(type => type)` (一对一映射)
- `strut`  结构体
- `Type[8]`  定长数组
- `Type[]`  动态数组
- `strut`  结构体

## 一些结构
- `strut`  结构体
- `enum` (枚举类型)
- `contract`  合约
- `function`  函数
- `event`   事件
- `modifier`  修饰符

## 存储方式
- `storage`: 成员变量，永久保存在状态树中(付费)
- `memory`:局部变量，临时存储(值传递)
- `calldata`: 函数参数变量(临时存储的一个数据位置)

## 表达式
- 逻辑运算符： `and` 、`or`、 `not`
- 关系运算符:  `==`、 `=` 、`>`、 `<=`
- 位运算符: `&`、`|` 
- 条件运算符: `?:`
- 算数运算符: `+` 、`-`、 `*`、 `/`

## 控制流程
`if, else, while, do, for, switch, continue, break, return`


## 内置对象 block   

    Block 在调用某个方法的时候，solidity会提供一个block的变量，把当前块的信息返回。 
- `block.coinbase()`: 返回挖掘此块的节点地址
- `block.difficulty()`: 返回当前区块的难度
- `block.gaslimit()`: 返回当前块的最大燃气量
- `block.limit()`: 返回当前区块的gas消耗限制
- `block.number()`: 返回链上当前块高、编号
- `block.timestamp()`: 返回当前区块的时间戳

## 内置对象 msg
- `msg.sender()`: 返回当前调用合约的发送者的地址
- `msg.gas()`: 返回燃料的消耗量
- `msg.sig()`: 返回数据的前四个字节
- `msg.value`: 返回发送消息的数量

## 内置其他函数
- `account.balance()`: 返回地的址余额（以wei为单位）
- `address.transfer()`: 在两个账户之间转移ether
- `assert(bool condition,string memory reason)`:自信某一条件一定成立，用于安全设计，如果不成立，扣光所有gas
- `require(bool condition,string memory reason)`:温和认定某条件成立，如果不满足，退回剩余的gas
- `revert(string memory reason)`:终止合约执行，并还原状态变更
- `now()`: 返回当前时间的时间戳（秒）
- 随机数：  
```
random = uint(keccak256(abi.encodePacked(msg.sender,block.difficulty,now)))
```

