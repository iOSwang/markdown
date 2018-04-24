##第一章：以太坊上的代币
如果你对以太坊的世界有一些了解，你很可能听过人们聊到代币——尤其是 ERC20 代币.

一个 代币 在以太坊基本上就是一个遵循一些共同规则的智能合约——即它实现了所有其他代币合约共享的一组标准函数，例如 transfer(address _to, uint256 _value) 和 balanceOf(address _owner).

在智能合约内部，通常有一个映射， mapping(address => uint256) balances，用于追踪每个地址还有多少余额。

所以基本上一个代币只是一个追踪谁拥有多少该代币的合约，和一些可以让那些用户将他们的代币转移到其他地址的函数。

##第二章：ERC721 标准, 多重继承

```
contract ERC721 {
  event Transfer(address indexed _from, address indexed _to, uint256 _tokenId);
  event Approval(address indexed _owner, address indexed _approved, uint256 _tokenId);

  function balanceOf(address _owner) public view returns (uint256 _balance);
  function ownerOf(uint256 _tokenId) public view returns (address _owner);
  function transfer(address _to, uint256 _tokenId) public;
  function approve(address _to, uint256 _tokenId) public;
  function takeOwnership(uint256 _tokenId) public;
}
```
在实现一个代币合约的时候，我们首先要做的是将接口复制到它自己的 Solidity 文件并导入它，import ./erc721.sol。 接着，让我们的合约继承它，然后我们用一个函数定义来重写每个方法。

但等一下—— ZombieOwnership已经继承自 ZombieAttack了 —— 它如何能够也继承于 ERC721呢？

幸运的是在Solidity，你的合约可以继承自多个合约，参考如下：

contract SatoshiNakamoto is NickSzabo, HalFinney {
  // 啧啧啧，宇宙的奥秘泄露了
}
正如你所见，当使用多重继承的时候，你只需要用逗号 , 来隔开几个你想要继承的合约。在上面的例子中，我们的合约继承自 NickSzabo 和 HalFinney。

来试试吧。

##第三章:balanceOf 和 ownerOf

```
  function balanceOf(address _owner) public view returns (uint256 _balance);

  function ownerOf(uint256 _tokenId) public view returns (address _owner);

```
##第5章: ERC721: 转移标准
```
function transfer(address _to, uint256 _tokenId) public;

function approve(address _to, uint256 _tokenId) public;
function takeOwnership(uint256 _tokenId) public;
```
第一种方法是代币的拥有者调用transfer 方法，传入他想转移到的 address 和他想转移的代币的 _tokenId。
第二种方法是代币拥有者首先调用 approve，然后传入与以上相同的参数。接着，该合约会存储谁被允许提取代币，通常存储到一个 mapping (uint256 => address) 里。然后，当有人调用 takeOwnership 时，合约会检查 msg.sender 是否得到拥有者的批准来提取代币，如果是，则将代币转移给他。

##第7章: ERC721: 批准
现在，让我们来实现 approve。

记住，使用 approve 或者 takeOwnership 的时候，转移有2个步骤：

你，作为所有者，用新主人的 address 和你希望他获取的 _tokenId 来调用 approve

新主人用 _tokenId 来调用 takeOwnership，合约会检查确保他获得了批准，然后把代币转移给他。

因为这发生在2个函数的调用中，所以在函数调用之间，我们需要一个数据结构来存储什么人被批准获取什么。

```
pragma solidity ^0.4.19;

import "./zombieattack.sol";
import "./erc721.sol";

contract ZombieOwnership is ZombieAttack, ERC721 {

  // 1. 在这里定义映射
  mapping(uint => address) zombieApprovals;

  function balanceOf(address _owner) public view returns (uint256 _balance) {
    return ownerZombieCount[_owner];
  }

  function ownerOf(uint256 _tokenId) public view returns (address _owner) {
    return zombieToOwner[_tokenId];
  }

  function _transfer(address _from, address _to, uint256 _tokenId) private {
    ownerZombieCount[_to]++;
    ownerZombieCount[_from]--;
    zombieToOwner[_tokenId] = _to;
    Transfer(_from, _to, _tokenId);
  }

  function transfer(address _to, uint256 _tokenId) public onlyOwnerOf(_tokenId) {
    _transfer(msg.sender, _to, _tokenId);
  }

  // 2. 在这里添加方法修饰符
  function approve(address _to, uint256 _tokenId) public onlyOwnerOf(_tokenId) {
    // 3. 在这里定义方法
    zombieApprovals[_tokenId] = _to;
    Approval(msg.sender, _to, _tokenId);
  }

  function takeOwnership(uint256 _tokenId) public {

  }
}

```

##第8章: ERC721: takeOwnership
最后一个函数 takeOwnership， 应该只是简单地检查以确保 msg.sender 已经被批准来提取这个代币或者僵尸。若确认，就调用 _transfer；

##第9章: 预防溢出
什么是 溢出 (overflow)?

假设我们有一个 uint8, 只能存储8 bit数据。这意味着我们能存储的最大数字就是二进制 11111111 (或者说十进制的 2^8 - 1 = 255).

来看看下面的代码。最后 number 将会是什么值？

uint8 number = 255;
number++;
在这个例子中，我们导致了溢出 — 虽然我们加了1， 但是 number 出乎意料地等于 0了。 (如果你给二进制 11111111 加1, 它将被重置为 00000000，就像钟表从 23:59 走向 00:00)。

下溢(underflow)也类似，如果你从一个等于 0 的 uint8 减去 1, 它将变成 255 (因为 uint 是无符号的，其不能等于负数)。

虽然我们在这里不使用 uint8，而且每次给一个 uint256 加 1 也不太可能溢出 (2^256 真的是一个很大的数了)，在我们的合约中添加一些保护机制依然是非常有必要的，以防我们的 DApp 以后出现什么异常情况。


使用 SafeMath
为了防止这些情况，OpenZeppelin 建立了一个叫做 SafeMath 的 库(library)，默认情况下可以防止这些问题。

不过在我们使用之前…… 什么叫做库?

一个库 是 Solidity 中一种特殊的合约。其中一个有用的功能是给原始数据类型增加一些方法。

比如，使用 SafeMath 库的时候，我们将使用 using SafeMath for uint256 这样的语法。 SafeMath 库有四个方法 — add， sub， mul， 以及 div。现在我们可以这样来让 uint256 调用这些方法

```
using SafeMath for uint256;

uint256 a = 5;
uint256 b = a.add(3); // 5 + 3 = 8
uint256 c = a.mul(2); // 5 * 2 = 10

library SafeMath {

  function mul(uint256 a, uint256 b) internal pure returns (uint256) {
    if (a == 0) {
      return 0;
    }
    uint256 c = a * b;
    assert(c / a == b);
    return c;
  }

  function div(uint256 a, uint256 b) internal pure returns (uint256) {
    // assert(b > 0); // Solidity automatically throws when dividing by 0
    uint256 c = a / b;
    // assert(a == b * c + a % b); // There is no case in which this doesn't hold
    return c;
  }

  function sub(uint256 a, uint256 b) internal pure returns (uint256) {
    assert(b <= a);
    return a - b;
  }

  function add(uint256 a, uint256 b) internal pure returns (uint256) {
    uint256 c = a + b;
    assert(c >= a);
    return c;
  }
}


```
**myUint = myUint.add(1);**

```
 using SafeMath for uint256;
  // 1. 为 uint32 声明 使用 SafeMath32
  using SafeMath32 for uint32;

  // 2. 为 uint16 声明 使用 SafeMath16
    using SafeMath16 for uint16;
```

**为了防止溢出和下溢，我们可以在我们的代码里找 +， -， *， 或 /，然后替换为 add, sub, mul, div**

// 这是一个单行注释，可以理解为给自己或者别人看的笔记

/// TODO: 把这里变成 natspec 标准的注释把
//@title hahha
//@author wang
//@dev GG

##第4章: 调用和合约函数
Web3.js 有两个方法来调用我们合约的函数: call and send.

call 用来调用 view 和 pure 函数。它只运行在本地节点，不会在区块链上创建事务。

复习: view 和 pure 函数是只读的并不会改变区块链的状态。它们也不会消耗任何gas。用户也不会被要求用MetaMask对事务签名。

使用 Web3.js，你可以如下 call 一个名为myMethod的方法并传入一个 123 作为参数

send 将创建一个事务并改变区块链上的数据。你需要用 send 来调用任何非 view 或者 pure 的函数。

注意: send 一个事务将要求用户支付gas，并会要求弹出对话框请求用户使用 Metamask 对事务签名。在我们使用 Metamask 作为我们的 web3 提供者的时候，所有这一切都会在我们调用 send() 的时候自动发生。而我们自己无需在代码中操心这一切，挺爽的吧。

使用 Web3.js, 你可以像这样 send 一个事务调用myMethod 并传入 123 作为参数：

myContract.methods.myMethod(123).send()
语法几乎 call()一模一样。














