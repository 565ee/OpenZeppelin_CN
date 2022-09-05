• [建立本地区块链](#index1)  
• [部署智能合约](#index2)  
• [从控制台交互](#index3)  
• [以编程方式交互](#index4)  
• [获取合约实例](#index5)  
• [调用合约](#index6)  
• [发送交易](#index7)  
• [OpenZeppelin Tutorials 教程](#index98)   
• [Contact 联系方式](#index99) 

# <span id='index1'>• 建立本地区块链</span>  
在开始之前，我们首先需要一个可以部署合约的环境。以太坊区块链（通常称为“主网”，表示“主网络”）需要花费真金白银才能使用它，以以太币（其本币）的形式。在尝试新想法或工具时，这使其成为一个糟糕的选择。

为了解决这个问题，存在许多“测试网络”（用于“测试网络”）：其中包括 Ropsten、Rinkeby、Kovan 和 Goerli 区块链。它们的工作方式与主网非常相似，但有一个区别：您可以免费获得这些网络的以太币，因此使用它们不会花费您一分钱。但是，您仍然需要处理私钥管理、5 到 20 秒范围内的阻塞时间，以及实际获得这个免费的 Ether。

在开发过程中，最好使用本地区块链。它在您的机器上运行，不需要互联网访问，为您提供所需的所有以太币，并立即挖掘区块。这些原因也使得本地区块链非常适合自动化测试。

Hardhat 带有一个内置的本地区块链，即Hardhat Network。

启动时，Hardhat Network 将创建一组未锁定的帐户并为它们提供以太币。
```
$ npx hardhat node
```
Hardhat Network 将打印出其地址，http://127.0.0.1:8545以及可用帐户及其私钥的列表。

请记住，每次运行 Hardhat Network 时，它都会创建一个全新的本地区块链 -不会保留之前运行的状态。这对于短期实验来说很好，但这意味着您需要在这些指南期间打开一个运行 Hardhat Network 的窗口。

# <span id='index2'>• 部署智能合约</span>  
在开发智能合约指南中，我们设置了我们的开发环境。

如果您还没有此设置，请创建并设置项目，然后创建并编译我们的 Box 智能合约。

随着我们的项目设置完成，我们现在可以部署合约了。我们将从Box开发智能合约指南中进行部署。确保您有Box in的副本contracts/Box.sol。

Hardhat 目前没有原生部署系统，而是使用脚本来部署合约。

我们将创建一个脚本来部署我们的 Box 合约。我们将此文件另存为scripts/deploy.js.

```
// scripts/deploy.js
async function main () {
  // We get the contract to deploy
  const Box = await ethers.getContractFactory('Box');
  console.log('Deploying Box...');
  const box = await Box.deploy();
  await box.deployed();
  console.log('Box deployed to:', box.address);
}

main()
  .then(() => process.exit(0))
  .catch(error => {
    console.error(error);
    process.exit(1);
  });
```

我们在脚本中使用了ethers，因此我们需要安装它和@nomiclabs/hardhat-ethers 插件。
```
$ npm install --save-dev @nomiclabs/hardhat-ethers ethers
```
我们需要添加我们正在使用插件的配置。@nomiclabs/hardhat-ethers
```
// hardhat.config.js
require('@nomiclabs/hardhat-ethers');

...
module.exports = {
...
};
```

全部完成！在真实的网络上，这个过程需要几秒钟，但在本地区块链上几乎是即时的。

# <span id='index3'>• 从控制台交互</span>  
部署Box合约后，我们可以立即开始使用它。

我们将使用Hardhat 控制台与我们Box在 localhost 网络上部署的合约进行交互。
```
npx hardhat console --network localhost
Welcome to Node.js v12.22.1.
Type ".help" for more information.
> const Box = await ethers.getContractFactory('Box');
undefined
> const box = await Box.attach('0x5FbDB2315678afecb367f032d93F642f64180aa3')
undefined
```

发送交易
Box的第一个函数store接收一个整数值并将其存储在合约存储中。因为这个函数修改了区块链状态，所以我们需要向合约发送一个交易来执行它。

我们将发送一个事务来调用store具有数值的函数：
```
> await box.store(42)
{
  hash: '0x3d86c5c2c8a9f31bedb5859efa22d2d39a5ea049255628727207bc2856cce0d3',
```

查询状态
Box的另一个函数被调用retrieve，它返回存储在合约中的整数值。这是区块链状态的查询，所以我们不需要发送交易：
```
> await box.retrieve()
BigNumber { _hex: '0x2a', _isBigNumber: true }
```

# <span id='index4'>• 以编程方式交互</span>  
控制台对于原型设计和运行一次性查询或事务很有用。但是，最终您将希望通过自己的代码与您的合约进行交互。

在本节中，我们将了解如何通过 JavaScript 与我们的合约进行交互，并使用Hardhat通过我们的 Hardhat 配置运行我们的脚本。

设置
让我们开始在一个新scripts/index.js文件中编码，我们将在其中编写 JavaScript 代码，从一些样板开始，包括编写异步代码。
```
// scripts/index.js
async function main () {
  // Our code will go here
}

main()
  .then(() => process.exit(0))
  .catch(error => {
    console.error(error);
    process.exit(1);
  });
```

我们可以通过询问本地节点来测试我们的设置，例如启用的帐户列表：
```
// Retrieve accounts from the local node
const accounts = await ethers.provider.listAccounts();
console.log(accounts);
```

使用 运行上面的代码hardhat run，并检查您是否获得了可用帐户列表作为响应。

```
$ npx hardhat run --network localhost ./scripts/index.js
[
  '0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266',
  '0x70997970C51812dc3A010C7d01b50e0d17dc79C8',
...
]
```

这些帐户应与您之前启动本地区块链时显示的帐户相匹配。现在我们有了第一个从区块链中获取数据的代码片段，让我们开始使用我们的合约。请记住，我们在上面定义的main函数中添加了代码。

# <span id='index5'>• 获取合约实例</span>  
为了与Box我们部署的合约进行交互，我们将使用一个以太合约实例。

ethers 合约实例是一个 JavaScript 对象，它代表我们在区块链上的合约，我们可以使用它与我们的合约进行交互。要将其附加到我们部署的合约中，我们需要提供合约地址。
```
// Set up an ethers contract, representing our deployed Box instance
const address = '0x5FbDB2315678afecb367f032d93F642f64180aa3';
const Box = await ethers.getContractFactory('Box');
const box = await Box.attach(address);
```

# <span id='index6'>• 调用合约</span>  
让我们从显示Box合约的当前价值开始。

我们需要调用合约的只读retrieve()公共方法，并等待响应：

```
// Call the retrieve() function of the deployed Box contract
const value = await box.retrieve();
console.log('Box value is', value.toString());
```

这个片段相当于我们之前从控制台运行的查询。现在，通过再次运行脚本并检查打印值来确保一切运行顺利：
```
npx hardhat run --network localhost ./scripts/index.js
Box value is 42
```

# <span id='index7'>• 发送交易</span>  
现在，我们将向storeBox 中的新值发送交易。

让我们在 中存储一个值，然后使用23我们Box之前编写的代码来显示更新后的值：
```
// Send a transaction to store() a new value in the Box
await box.store(23);

// Call the retrieve() function of the deployed Box contract
const value = await box.retrieve();
console.log('Box value is', value.toString());
```

我们现在可以运行代码片段，并检查框的值是否已更新！

```
$ npx hardhat run --network localhost ./scripts/index.js
Box value is 23
```

# <span id='index98'>• OpenZeppelin Tutorials 教程</span>  
CN 中文 Github  [OpenZeppelin 教程 : github.com/565ee/OpenZeppelin_CN](https://github.com/565ee/OpenZeppelin_CN)  
CN 中文 CSDN    [OpenZeppelin 教程 : blog.csdn.net/wx468116118](https://blog.csdn.net/wx468116118/category_11997496.html)  
EN 英文 Github  [OpenZeppelin Tutorials : github.com/565ee/OpenZeppelin_EN](https://github.com/565ee/OpenZeppelin_EN)  

# <span id='index99'>• Contact 联系方式</span>  
Homepage : [565.ee](https://565.ee)  
微信公众号 : wx468116118  
微信 QQ   : 468116118  
GitHub   : [github.com/565ee](https://github.com/565ee)   
CSDN     : [blog.csdn.net/wx468116118](https://blog.csdn.net/wx468116118)  
Email    : 468116118@qq.com
