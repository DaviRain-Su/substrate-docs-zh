# 启动一个节点


所有 Substrate 教程和操作指南都要求您在开发环境中构建和运行 Substrate 节点。为了帮助您快速设置工作环境，[Substrate Developer Hub](https://github.com/substrate-developer-hub/) 维护了模板供您使用。例如，[substrate-node-template](https://github.com/substrate-developer-hub/substrate-node-template/tags/) 是主要 Substrate node-template 二进制文件的快照，其中包含一组核心功能，可帮助您入门。


启动节点后，您可以使用 Web 浏览器和一个简单的应用程序连接到它，该应用程序允许您查找预定义帐户的余额。


## 在你开始之前

在开始之前，请验证以下内容：

- 您有互联网连接并可以访问本地计算机上的交互式 shell 终端。
- 您通常熟悉软件开发和使用命令行界面。
- 您已经安装了 Rust 编译器和工具链。

您可以通过运行 `rustup show` 命令检查是否安装了 Rust。如果安装了 Rust，此命令会显示工具链和编译器的版本信息。如果未安装 Rust，该命令不会返回任何输出。有关安装 Rust 的信息，请参阅[安装](../install/index.md)。

## 构建Subtrate node template

1. 通过运行以下命令克隆节点模板存储库：

```bash
git clone https://github.com/substrate-developer-hub/substrate-node-template
```

此命令克隆 main 分支。

或者，您可以使用 `--branch` 命令行选项和标签来指定您希望节点兼容的 Polkadot 版本。

2. 更改为克隆目录的根目录：

```bash
cd substrate-node-template
```


3. 通过运行类似于以下的命令创建一个新分支以保存您的工作：

```bash
git switch -c my-learning-branch-yyyy-mm-dd
```

您可以使用您选择的任何识别信息来命​​名分支机构。在大多数情况下，您应该在名称中包含克隆分支的年月日信息。例如：

```bash
git switch -c my-learning-branch-2023-03-31
```

4.  编译节点模板：

```bash
cargo build --package node-template --release
```

第一次编译节点时，可能需要一些时间才能完成。编译完成后，您应该看到如下一行：


```bash
Finished release [optimized] target(s) in 11m 23s
```


## 查看节点信息

1. 通过运行以下命令验证您的节点是否已准备好使用并查看有关可用命令行选项的信息：

```bash
./target/release/node-template --help
```

使用信息显示可用于以下操作的命令行选项：
- 启动节点
- 使用帐户和密钥
- 修改节点操作

2. 通过运行以下命令查看预定义的 `Alice` 开发帐户的帐户信息：

```bash
./target/release/node-template key inspect //Alice
```

该命令显示以下帐户和地址信息：

```bash
Secret Key URI `//Alice` is account:
Network ID:        substrate
Secret seed:       0xe5be9a5092b81bca64be81d212e7f2f9eba183bb7a90954f7b76361f6edb5c0a
Public key (hex):  0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d
Account ID:        0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d
Public key (SS58): 5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
SS58 Address:      5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
```

`Alice` 和 `Bob` 等预定义开发帐户在链规范文件中配置。您将在[探索代码](./explor_the_code.md)中了解有关节点模板文件的更多信息，并在[链规范](../build/chain_speficication.md)中更具体地了解链规范文件。现在，只要知道存在开发帐户就可以测试余额转账等简单交易就足够了。


## 启动区块链

1. 通过运行以下命令以开发模式启动节点：

```bash
./target/release/node-template --dev
```

在开发模式下，链不需要任何对等计算机来完成块。当节点启动时，终端会显示有关所执行操作的输出。如果您看到正在提议和最终确定块的消息，则您有一个正在运行的节点。

```bash
... Idle (0 peers), best: #3 (0xcc78…5cb1), finalized #1 ...
... Starting consensus session on top of parent ...
... Prepared block for proposing at 4 (0 ms) ...
```


## 连接到节点

现在您的节点正在运行，您可以连接到它以检查预定义的 `Alice` 帐户的余额。对于这个简单的应用程序，您可以创建一个 `index.html` HTML 文件，该文件使用 JavaScript 和 [Polkadot-JS API](https://polkadot.js.org/docs/api) 与区块链进行交互。

例如，此示例 [index.html](https://docs.substrate.io/assets/quickstart/) 演示了如何使用 JavaScript、Polkadot-JS API 和 HTML 来执行以下操作：

- 输入一个账户地址。
- 使用 `onClick` 事件查找帐户余额。
- 将帐户余额显示为输出。

要连接到节点并检查帐户余额：
1. 将快速入门的[示例代码](https://docs.substrate.io/assets/quickstart/)复制并粘贴：获取 Balance 应用程序到代码编辑器中的一个新文件中，并将该文件保存在本地计算机上。
2. 在 Web 浏览器中打开 `index.html` 文件。
3. 在输入字段中复制并粘贴 `Alice` 帐户的 SS58 地址，然后单击获取余额。

## 停止节点

1. 转到显示区块链操作的终端。
2. 按 control-c 组合键停止本地区块链并清除所有状态。
