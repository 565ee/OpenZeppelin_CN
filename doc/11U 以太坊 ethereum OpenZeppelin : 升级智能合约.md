• [升级中有什么](#index1)  
• [使用升级插件升级](#index2)  
• [升级如何运作](#index3)  
• [初始化](#index4)  
• [升级](#index5)  
• [测试](#index6)   
• [OpenZeppelin Tutorials 教程](#index98)   
• [Contact 联系方式](#index99) 

# <span id='index1'>• 升级中有什么</span>  
使用OpenZeppelin 升级插件部署的智能合约可以升级以修改其代码，同时保留其地址、状态和余额。这使您可以迭代地向项目添加新功能，或修复您在生产中可能发现的任何错误。

默认情况下，以太坊中的智能合约是不可变的。一旦你创建了它们，就无法改变它们，有效地充当参与者之间牢不可破的契约。

但是，对于某些情况，希望能够修改它们。想想双方之间的传统合同：如果他们都同意改变它，他们就可以这样做。在以太坊上，他们可能希望更改智能合约以修复他们发现的错误（这甚至可能导致黑客窃取他们的资金！），添加额外的功能，或者只是改变它强制执行的规则。

# <span id='index2'>• 使用升级插件升级</span>  
每当您使用OpenZeppelin Upgrades Plugins部署新合约deployProxy时，该合约实例可以稍后升级。默认情况下，只有最初部署合约的地址才有权对其进行升级。

deployProxy将创建以下交易：
部署实现合约（我们的Box合约）
部署ProxyAdmin合约（我们代理的管理员）。
部署代理合约并运行任何初始化函数。

让我们看看它是如何工作的，通过部署我们合约的可升级版本，使用与我们之前部署Box时相同的设置：
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

安装安全帽升级插件。
```
npm install --save-dev @openzeppelin/hardhat-upgrades
```

然后我们需要配置 Hardhat 以使用我们的@openzeppelin/hardhat-upgrades插件。为此，请在您的hardhat.config.js文件中添加插件，如下所示。
```
// hardhat.config.js
...
require('@nomiclabs/hardhat-ethers');
require('@openzeppelin/hardhat-upgrades');
...
module.exports = {
...
};
```

为了像Box我们一样升级合约，我们需要首先将其部署为可升级合约，这是与我们目前看到的不同的部署过程。我们将通过调用store值 42 来初始化我们的 Box 合约。

Hardhat 目前没有原生部署系统，而是使用脚本来部署合约。

我们将创建一个脚本来部署我们的可升级 Box 合约，使用deployProxy. 我们将此文件另存为scripts/deploy_upgradeable_box.js.
```
// scripts/deploy_upgradeable_box.js
const { ethers, upgrades } = require('hardhat');

async function main () {
  const Box = await ethers.getContractFactory('Box');
  console.log('Deploying Box...');
  const box = await upgrades.deployProxy(Box, [42], { initializer: 'store' });
  await box.deployed();
  console.log('Box deployed to:', box.address);
}

main();
```

然后我们可以部署我们的可升级合约。

使用该run命令，我们可以将Box合约部署到development网络。
```
npx hardhat run --network localhost scripts/deploy_upgradeable_box.js
Deploying Box...
Box deployed to: 0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0
```

Box然后，我们可以与我们的合约交互到retrieve我们在初始化期间存储的值。

我们将使用Hardhat 控制台与我们升级的Box合约进行交互。

我们需要在部署Box合约时指定代理合约的地址。
```
npx hardhat console --network localhost
Welcome to Node.js v12.22.1.
Type ".help" for more information.
> const Box = await ethers.getContractFactory('Box');
undefined
> const box = await Box.attach('0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0');
undefined
> (await box.retrieve()).toString();
'42'
```

upgradeProxy创建 Solidity 文件后，我们现在可以使用该功能升级我们之前部署的实例。

upgradeProxy将创建以下交易：
部署实现合约（我们的BoxV2合约）
调用ProxyAdmin更新代理合约以使用新的实现。

我们将创建一个脚本来升级我们的Box合约以BoxV2使用upgradeProxy. 我们将此文件另存为scripts/upgrade_box.js. 我们需要在部署Box合约时指定代理合约的地址。
```
// scripts/upgrade_box.js
const { ethers, upgrades } = require('hardhat');

async function main () {
  const BoxV2 = await ethers.getContractFactory('BoxV2');
  console.log('Upgrading Box...');
  await upgrades.upgradeProxy('0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0', BoxV2);
  console.log('Box upgraded');
}

main();
```

然后我们可以部署我们的可升级合约。

使用该run命令，我们可以在网络上升级Box合约。development

```
$ npx hardhat run --network localhost scripts/upgrade_box.js
Compiling 1 file with 0.8.4
Compilation finished successfully
Upgrading Box...
Box upgraded
```

完毕！我们的Box实例已升级到最新版本的代码，同时保持其状态和与以前相同的地址。我们不需要在新地址部署新地址，也不需要手动将value旧地址复制到新地址Box。

让我们通过调用新函数来尝试一下increment，然后检查一下value：

我们需要在部署Box合约时指定代理合约的地址。
```
npx hardhat console --network localhost
Welcome to Node.js v12.22.1.
Type ".help" for more information.
> const BoxV2 = await ethers.getContractFactory('BoxV2');
undefined
> const box = await BoxV2.attach('0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0');
undefined
> await box.increment();
...
> (await box.retrieve()).toString();
'43'
```

而已！请注意在整个升级过程中是如何value保留Box的，以及它的地址。无论你是在本地区块链、测试网还是主网络上工作，这个过程都是一样的。

让我们看看OpenZeppelin 升级插件是如何做到这一点的。

# <span id='index3'>• 升级如何运作</span>  
本部分将比其他部分更注重理论：如果您好奇，请随意跳过它，稍后再返回。

当您创建新的可升级合约实例时，OpenZeppelin 升级插件实际上会部署三个合约：
您编写的合约，称为包含逻辑的实现合约。
一个ProxyAdmin作为代理的管理员。
实现合约的代理，这是您实际与之交互的合约。

在这里，代理是一个简单的合约，它只是将所有调用委托给一个实现合约。委托调用类似于常规调用，不同之处在于所有代码都在调用者的上下文中执行，而不是在被调用者的上下文中。正因为如此，transfer执行合约的代码中的a实际上会转移代理的余额，任何对合约存储的读写操作都会从代理自己的存储中读取或写入。

这允许我们将合约的状态和代码解耦：代理持有状态，而实现合约提供代码。它还允许我们通过将代理委托给不同的实现合约来更改代码。

你可以有多个代理使用同一个实现合约，所以如果你计划部署同一个合约的多个副本，你可以使用这种模式节省燃料。
智能合约的任何用户始终与代理进行交互，代理永远不会更改其地址。这允许您推出升级或修复错误，而无需要求您的用户最终更改任何内容 - 他们只是一如既往地与相同的地址进行交互。

如果您想了解有关 OpenZeppelin 代理如何工作的更多信息，请查看Proxies。

# <span id='index4'>• 初始化</span>  
可升级合约不能有constructor. 为了帮助您运行初始化代码，OpenZeppelin Contracts提供了Initializable基础合约，允许您将方法标记为initializer，确保它只能运行一次。

举个例子，让我们Box用一个初始化器编写一个新版本的合约，存储一个admin谁将是唯一允许更改其内容的人的地址。
```
// contracts/AdminBox.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

contract AdminBox is Initializable {
    uint256 private _value;
    address private _admin;

    // Emitted when the stored value changes
    event ValueChanged(uint256 value);

    function initialize(address admin) public initializer {
        _admin = admin;
    }

    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() initializer {}

    // Stores a new value in the contract
    function store(uint256 value) public {
        require(msg.sender == _admin, "AdminBox: not admin");
        _value = value;
        emit ValueChanged(value);
    }

    // Reads the last stored value
    function retrieve() public view returns (uint256) {
        return _value;
    }
}
```

部署此合约时，我们需要指定initializer函数名称（仅当名称不是默认名称时initialize）并提供我们要使用的管理员地址。
```
// scripts/deploy_upgradeable_adminbox.js
const { ethers, upgrades } = require('hardhat');

async function main () {
  const AdminBox = await ethers.getContractFactory('AdminBox');
  console.log('Deploying AdminBox...');
  const adminBox = await upgrades.deployProxy(AdminBox, ['0xACa94ef8bD5ffEE41947b4585a84BdA5a3d3DA6E'], { initializer: 'initialize' });
  await adminBox.deployed();
  console.log('AdminBox deployed to:', adminBox.address);
}

main();
```

出于所有实际目的，初始化程序充当构造函数。但是，请记住，由于它是一个常规函数，因此您需要手动调用所有基本合约（如果有）的初始化程序。

您可能已经注意到我们包含了一个构造函数和一个初始化程序。此构造函数的目的是使实现合同处于初始化状态，这是对某些潜在攻击的缓解。

要在编写可升级合同时了解有关此警告和其他注意事项的更多信息，请查看我们的编写可升级合同指南。

# <span id='index5'>• 升级</span>  
由于技术限制，当您将合约升级到新版本时，您无法更改该合约的存储布局。

这意味着，如果您已经在合约中声明了一个状态变量，您就不能删除它、更改它的类型或在它之前声明另一个变量。在我们的Box示例中，这意味着我们只能在 之后 value添加新的状态变量。
```
// contracts/Box.sol
contract Box {
    uint256 private _value;

    // We can safely add a new variable after the ones we had declared
    address private _owner;

    // ...
}
```

幸运的是，这个限制只影响状态变量。您可以根据需要更改合约的功能和事件。

如果您不小心弄乱了合约的存储布局，升级插件会在您尝试升级时警告您。

# <span id='index6'>• 测试</span>  
为了测试可升级的合约，我们应该为实现合约创建单元测试，同时创建更高级别的测试来测试通过代理的交互。我们可以deployProxy像部署时一样在测试中使用。

当我们想要升级时，我们应该为新的实现合约创建单元测试，并在升级后创建更高级别的测试以测试通过代理的交互upgradeProxy，检查升级后的状态是否保持不变。

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
