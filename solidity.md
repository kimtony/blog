## 智能合约开发




### 在以太坊中，以下语句被视为修改链上状态：

```
写入状态变量。
释放事件。
创建其他合约。
使用selfdestruct.
通过调用发送以太币。
调用任何未标记view或pure的函数。
使用低级调用（low-level calls）。
使用包含某些操作码的内联汇编。
```

### 有哪些常见的智能合约漏洞需要注意
* 安全问题和智能合约的代码、平台、编程语言、编程语言的版本都有关系。
```
整数溢出和下溢：一些数值在计算过程中可能会超出整数的最大值和最小值，发生意料之外的情况。我是通过 OpenZeppline 提供的 SafeMath 库来处理这种情况的。

可重入攻击：如果智能合约在发送 ETH 之前没有改变状态，那么攻击者可以递归调用智能合约，最后抽空合约里面的资产。为了避免这种情况，可以使用检查-生效-交互（checks-effects-interactions）的模式来避免。

短地址攻击：ERC20 正常的输入字节码是 136 位，如果低于 136 位，EVM 会在末位补 0。攻击者可以使用比正常短的地址，来诱导智能合约错误解析交易参数，这样可能导致资金丢失。我的解决方案就是验证字节码的长度。

selfdestruct 的使用：一定要慎用合约毁灭操作，因为一旦合约毁灭会导致资产丢失，无法找回。不过最新版本的 Solidity 删除了这个功能。

权限管理：正确配置权限和管理多签，避免权限被滥用。
```