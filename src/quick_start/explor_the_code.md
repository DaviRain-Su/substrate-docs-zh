# 探索代码

在启动节点中，您在开发模式下编译并启动了本地 Substrate 节点。这个特定的节点—— `substrate-node-template` ——提供了一个简化的环境，只有几个通用模块可以帮助您入门。无需深入了解细节，您可以从探索节点模板代码的基本构建块中学到很多东西。

## 关于节点模板

节点模板包括一些默认的区块链要素，如点对点网络、简单的共识机制和交易处理。节点模板还包括一些用于处理账户、余额和交易费用以及执行管理操作的基本功能。这组核心功能是通过几个实现特定功能的预定义模块（称为托盘）提供的。

例如，节点模板中预定义了以下核心模块：
- `pallet_balances` 用于管理账户资产和账户间转账。
- `pallet_transaction_payment` 用于管理所执行交易的交易费用。
- `pallet_sudo` 用于执行需要管理权限的操作。

节点模板还提供了一个启动器 `pallet_template` ，它说明了如何在自定义托盘中实现功能。

现在您已经大致了解了节点模板中包含的功能，让我们仔细看看 `substrate-node-template` 目录及其子目录中的代码。


## Manifest files

因为 Substrate 是一个基于 Rust 的框架，所以每个包都有一个清单文件—— `Cargo.toml` 文件——其中包含编译包所需的信息。如果您打开位于 `substrate-node-template` 根目录中的 `Cargo.toml` 文件，您可以看到它描述了构成节点模板工作区的成员包。例如：

```toml
[workspace]
members = [
    "node",
    "pallets/template",
    "runtime",
]
[profile.release]
panic = "unwind"
```

从此清单中，您可以看到节点模板工作区包括三个包：
- `node` 包为许多核心区块链服务提供 Rust 模块，例如对等网络、区块创作、区块终结和交易池管理。
- `pallets` 子目录中的 `template` 包是入门模板，它说明了在构建您自己的自定义模块时如何实现功能。
- `runtime` 包提供了所有用于处理帐户、余额、交易费用和节点模板中包含的其他功能的应用程序逻辑。

每个成员包也有自己的清单——它自己的 `Cargo.toml` 文件——包含特定于包的信息，包括编译该成员包所需的依赖项和配置设置。例如，工作区 `node` 成员的 `Cargo.toml` 文件指定包的名称为 `node-template` ，并列出使节点模板能够提供基本区块链服务的核心库和原语。您将在 [Architecture 和 Rust 库](../learn/arhitecture_and_rust.md)中了解有关库和原语的更多信息。现在，了解清单在描述每个包的依赖关系和其他关键信息方面的重要性就足够了。

如果您打开 `runtime/Cargo.toml` 文件和 `pallets/template/Cargo.toml` ，您将看到不同的库和基元作为依赖项，但您将大致了解编译这些包需要什么。例如，运行时的清单列出了所有托盘——包括 `frame_system` 、 `frame_support` 和前面提到的 `pallet_balances` 、 `pallet_transaction_payment` 和 `pallet_sudo` 模块——它们构成了节点模板的默认运行时.

## 核心客户端源代码

Substrate 最重要的方面之一是节点由两个主要部分组成：核心客户端和运行时。节点模板还包含 `node/src` 目录中的核心客户端服务和 `runtime/src` 目录中的运行时的单独包。

默认情况下， node/src 目录包含以下 Rust 模块：
- `benchmarking.rs`
- `chain_spec.rs`
- `cli.rs`
- `command.rs`
- `lib.rs`
- `main.rs`
- `rpc.rs`
- `service.rs`

大多数核心客户端服务都封装在 `node/src/service.rs` Rust 模块中。您很少需要修改此文件或 `node/src` 目录中的其他 Rust 模块。您可能要修改的文件是 `chain_spec.rs` 文件。 `chain_spec.rs` 文件描述了默认Development链和Local Testnet链的配置，包括默认预注资开发账户信息和预配置出块权限的节点信息。如果创建自定义链，则使用此文件来标识节点连接到的网络以及本地节点与之通信的其他节点。

## 默认节点模板运行时


由于 Substrate 为构建区块链提供了模块化和灵活的框架，您可以对工作区中的任何包进行更改。然而，大多数应用程序开发工作是在运行时和用于构建运行时的模块（托盘）中完成的。在开始为您自己的项目自定义运行时之前，您应该花一点时间探索默认节点模板中的内容。

### Default manifest

您已经看到运行时的默认清单如何在类似于以下的行中列出运行时的默认依赖项和功能：

```toml
pallet-balances = { version = "4.0.0-dev", default-features = false, git = "https://github.com/paritytech/substrate.git", branch = "polkadot-vX.Y.Z" }

pallet-sudo = { version = "4.0.0-dev", default-features = false, git = "https://github.com/paritytech/substrate.git", branch = "polkadot-vX.Y.Z" }

pallet-transaction-payment = { version = "4.0.0-dev", default-features = false, git = "https://github.com/paritytech/substrate.git", branch = "polkadot-vX.Y.Z" }
```

还有对核心包的依赖关系——例如 `frame-system` 、 `frame-support` 和 `frame-executive` 。您将在核心框架服务中了解有关这些核心服务的更多信息。现在，只需注意这些和其他模块是编译节点模板的运行时所必需的。


 ### 默认源代码

 运行时的主要源代码位于 `runtime/src/lib.rs` 文件中。如果您在代码编辑器中打开此文件，起初它可能看起来很复杂。文档的其他部分涵盖了一些细微差别，但本质上，源代码执行以下操作：

 - 导入 `framesystem` 和 `framesupport` 核心服务。
 - 指定运行时的版本信息。
 - 声明要包含的托盘。
 - 声明包含的每个托盘的类型和参数。
 - 为包含的每个托盘设置常量和变量值。
 - 为包含的每个托盘实现 `Config` trait。
 - 从包含的托盘构建运行时。
 - 准备用于评估托盘性能的基准测试框架。
 - 实现使核心客户端能够调用运行时的接口。


 您将在[构建](../build/index.md)和[测试](../test/index.md)部分的主题中了解有关构建运行时、定义基准和使用运行时接口的更多信息。现在，您只需要大致了解运行时是如何组成的，以及默认托盘是如何使用 `Config` Trait实现的。
