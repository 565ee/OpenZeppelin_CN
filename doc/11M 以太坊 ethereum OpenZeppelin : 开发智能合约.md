• [设置项目](#index1)  
• [第一份合同](#index2)  
• [编译 Solidity](#index3)  
• [添加更多合约](#index4)  
• [使用 OpenZeppelin 合约](#index5)  
• [OpenZeppelin Tutorials 教程](#index98)   
• [Contact 联系方式](#index99) 

# <span id='index1'>• 设置项目</span>  
创建项目后的第一步是安装开发工具。

以太坊最流行的开发框架是Hardhat，我们用ethers.js介绍了它最常见的用途。下一个最受欢迎的是使用web3.js的Truffle。每个人都有自己的长处，舒适地使用它们是很有用的。

在这些指南中，我们将展示如何使用 Truffle 和 Hardhat 开发、测试和部署智能合约。

要开始使用 Hardhat，我们将把它安装在我们的项目目录中。
```
$ npm install --save-dev hardhat
```
安装好后，我们就可以运行了npx hardhat。hardhat.config.js这将在我们的项目目录中创建一个 Hardhat 配置文件 ( )。

# <span id='index2'>• 第一份合同</span>  
我们将 Solidity 源文件 ( .sol) 存储在一个contracts目录中。这相当于src您可能从其他语言中熟悉的目录。

我们现在可以编写我们的第一个简单的智能合约，称为Box：它将让人们存储一个可以稍后检索的值。

我们将此文件另存为contracts/Box.sol. 每个.sol文件都应该包含单个合约的代码，并以它命名。

```
// contracts/Box.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Box {
    uint256 private _value;

    // Emitted when the stored value changes
    event ValueChanged(uint256 value);

    // Stores a new value in the contract
    function store(uint256 value) public {
        _value = value;
        emit ValueChanged(value);
    }

    // Reads the last stored value
    function retrieve() public view returns (uint256) {
        return _value;
    }
}
```

# <span id='index3'>• 编译 Solidity</span>  
以太坊虚拟机（EVM）不能直接执行 Solidity 代码：我们首先需要将其编译成 EVM 字节码。

我们的Box.sol合约使用 Solidity 0.8，因此我们需要首先配置 Hardhat 以使用适当的 solc 版本。

我们在hardhat.config.js.

```
// hardhat.config.js

/**
 * @type import('hardhat/config').HardhatUserConfig
 */
 module.exports = {
  solidity: "0.8.4",
};
```

然后可以通过运行单个编译命令来实现编译：
```
npx hardhat compile
Solidity 0.8.4 is not fully supported yet. You can still use Hardhat, but some features, like stack traces, might not work correctly.

Learn more at https://hardhat.org/reference/solidity-support"

Compiling 1 file with 0.8.4
Compilation finished successfully
```

compile内置任务将自动查找目录中的所有合约，contracts并使用 Solidity 编译器使用hardhat.config.js.

您会注意到artifacts创建了一个目录：它包含已编译的工件（字节码和元数据），它们是 .json 文件。将此目录添加到您的.gitignore.

# <span id='index4'>• 添加更多合约</span>  
随着项目的发展，您将开始创建更多相互交互的合约：每个合约都应该存储在自己的.sol文件中。

为了看看这看起来如何，让我们在我们的合约中添加一个简单的访问控制系统Box：我们将在名为 的合约中存储一个管理员地址Auth，并且只让Box那些Auth允许的帐户使用。

因为编译器会拾取contracts目录和子目录中的所有文件，所以您可以自由地组织您认为合适的代码。在这里，我们将Auth合约存储在一个access-control子目录中：

```
// contracts/access-control/Auth.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Auth {
    address private _administrator;

    constructor(address deployer) {
        // Make the deployer of the contract the administrator
        _administrator = deployer;
    }

    function isAdministrator(address user) public view returns (bool) {
        return user == _administrator;
    }
}
```

跨多个合约分离关注点是保持每个合约简单的好方法，并且通常是一种很好的做法。

但是，这不是将代码拆分为模块的唯一方法。您还可以在 Solidity 中使用继承进行封装和代码重用，我们将在接下来看到。

# <span id='index5'>• 使用 OpenZeppelin 合约</span>  
可重用的模块和库是伟大软件的基石。OpenZeppelin Contracts包含许多有用的构建块，可用于构建智能合约。在构建它们时您可以高枕无忧：它们已经过多次审计，其安全性和正确性经过实战考验。

导入 OpenZeppelin 合约
可以通过运行以下命令下载最新发布的 OpenZeppelin Contracts 库：
```
$ npm install @openzeppelin/contracts
```

要使用 OpenZeppelin 合约之一，import它通过在其路径前加上@openzeppelin/contracts. 例如，为了替换我们自己的Auth合约，我们将导入@openzeppelin/contracts/access/Ownable.sol以添加访问控制Box：
```
// contracts/Box.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// Import Ownable from the OpenZeppelin Contracts library
import "@openzeppelin/contracts/access/Ownable.sol";

// Make Box inherit from the Ownable contract
contract Box is Ownable {
    uint256 private _value;

    event ValueChanged(uint256 value);

    // The onlyOwner modifier restricts who can call the store function
    function store(uint256 value) public onlyOwner {
        _value = value;
        emit ValueChanged(value);
    }

    function retrieve() public view returns (uint256) {
        return _value;
    }
}
```

OpenZeppelin合约文档是学习开发安全智能合约系统的好地方。它具有指南和详细的 API 参考：例如，请参阅访问控制指南以了解有关Ownable上述代码示例中使用的合约的更多信息。

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
