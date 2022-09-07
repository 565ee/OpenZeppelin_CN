• [介绍](#index1)  
• [访问测试网节点](#index2)  
• [创建新帐户](#index3)  
• [配置网络](#index4)  
• [为测试网账户注资](#index5)  
• [在测试网上工作](#index6)   
• [OpenZeppelin Tutorials 教程](#index98)   
• [Contact 联系方式](#index99) 

# <span id='index1'>• 介绍</span>  
在您编写好合约并在本地试用并彻底测试之后，是时候转移到持久的公共测试环境了，您和您的 beta 用户可以开始与您的应用程序交互。

为此，我们将使用公共测试网络（又名测试网），这些网络的运行方式与以太坊主网络类似，但以太坊没有价值并且可以免费获得——这使得它们成为免费测试您的合约的理想选择。

请记住，部署到公共测试网络是开发以太坊项目时的必要步骤。它们提供了一个安全的测试环境，与主网络非常相似——您不希望在网络中拿出您的项目进行试驾，因为错误会让您和您的用户付出金钱！

# <span id='index2'>• 访问测试网节点</span>  
虽然您可以启动连接到测试网的自己的Geth或OpenEthereum节点，但访问测试网的最简单方法是通过Alchemy或Infura等公共节点服务。Alchemy 和 Infura 通过免费和付费计划为所有测试网和主网络提供对公共节点的访问。

当一个节点可以被公众访问时，我们说它是公共的，并且不管理任何帐户。这意味着它可以回复查询并中继已签名的交易，但不能自行签署交易。

在本指南中，我们将使用 Alchemy，但您可以使用Infura或您选择的其他公共节点提供商。

前往Alchemy（包括推荐代码），注册并记下您分配的 API 密钥 - 我们稍后将使用它连接到网络。

# <span id='index3'>• 创建新帐户</span>  
要在测试网中发送交易，您需要一个新的以太坊账户。有很多方法可以做到这一点：这里我们将使用mnemonics包，它将输出一个新的助记符（一组 12 个单词），我们将使用它来导出我们的帐户：
```
npx mnemonics
drama film snack motion ...
```
确保确保您的助记符安全。不要将秘密提交给版本控制。即使只是为了测试目的，仍然有恶意用户会为了好玩而对您的测试网部署造成严重破坏！

# <span id='index4'>• 配置网络</span>  
由于我们使用公共节点，我们需要在本地签署所有交易。我们将使用助记符和 Alchemy 端点配置网络。

我们需要使用与测试网的新网络连接来更新我们的配置文件。在这里，我们将使用 Rinkeby，但您可以使用任何您想要的：
```
// hardhat.config.js
+ const { alchemyApiKey, mnemonic } = require('./secrets.json');
...
  module.exports = {
+    networks: {
+     rinkeby: {
+       url: `https://eth-rinkeby.alchemyapi.io/v2/${alchemyApiKey}`,
+       accounts: { mnemonic: mnemonic },
+     },
+   },
...
};
```

请注意，在第一行中，我们正在从secrets.json文件中加载项目 id 和助记符，该文件应如下所示，但使用您自己的值。确保.gitignore它确保您不会将秘密提交给版本控制！
```
{
  "mnemonic": "drama film snack motion ...",
  "alchemyApiKey": "JPV2..."
}
```

我们现在可以通过列出可用于 Rinkeby 网络的帐户来测试此配置是否有效。请记住，您的会有所不同，因为它们取决于您使用的助记符。
```
npx hardhat console --network rinkeby
Welcome to Node.js v12.22.1.
Type ".help" for more information.
> accounts = await ethers.provider.listAccounts()
[
  '0xEce6999C6c5BDA71d673090144b6d3bCD21d13d4',
  '0xC1310ade58A75E6d4fCb8238f9559188Ea3808f9',
...
]
```

# <span id='index5'>• 为测试网账户注资</span>  
大多数公共测试网都有一个水龙头：一个免费为您提供少量测试以太币的站点。如果您在 Rinkeby，请前往Rinkeby Authenticated Faucet，通过使用您的 Twitter 或 Facebook 帐户进行身份验证来获取资金。或者，您也可以使用MetaMask 的水龙头直接向您的 MetaMask 账户索取资金。

有了一个资金账户，让我们将我们的合约部署到测试网！

# <span id='index6'>• 在测试网上工作</span>  
通过配置为在公共测试网上工作的项目，我们现在终于可以部署我们的Box合约了。此处的命令与您在本地开发网络上的命令完全相同，尽管在挖掘新块时需要几秒钟才能运行。

```
npx hardhat run --network rinkeby scripts/deploy.js
Deploying Box...
Box deployed to: 0xD7fBC6865542846e5d7236821B5e045288259cf0
```

而已！你的Box合约实例将永远存储在测试网中，任何人都可以公开访问。

您可以在Etherscan等区块浏览器上查看您的合约。请记住访问您部署合约的测试网上的资源管理器，例如 Rinkeby 的rinkeby.etherscan.io。

您还可以像往常一样与您的实例进行交互，使用控制台或以编程方式。
```
npx hardhat console --network rinkeby
Welcome to Node.js v12.22.1.
Type ".help" for more information.
> const Box = await ethers.getContractFactory('Box');
undefined
> const box = await Box.attach('0xD7fBC6865542846e5d7236821B5e045288259cf0');
undefined
> await box.store(42);
{
  hash: '0x330e331d30ee83f96552d82b7fdfa6156f9f97d549a612eeef7283d18b31d107',
...
> (await box.retrieve()).toString()
'42'
```

请记住，每笔交易都会花费一些汽油，因此您最终需要为您的账户充值更多资金。

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
