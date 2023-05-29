# 修改运行时


在探索代码中，您了解了构成默认节点模板的清单文件和 Rust 模块。现在您对运行时源代码的外观有了大致的了解，让我们看看通过一些简单的更改来自定义运行时是多么容易。

对于这个简单的演示，您将执行以下操作：
- 添加一个具有您要使用的某些功能的托盘。
- 更改一些常量值。
- 更新运行时版本。
- 重新编译运行时以包含您的更改。
- 提交交易以更新存储在链上的运行时。

您还将看到另一个使用 Polkadot-JS API 的应用程序，以及如何使用该应用程序的托管版本来查看链状态和提交交易。


## 在你开始之前

当您使用 --dev 命令行选项在开发模式下运行节点时，它会以干净的状态启动第一个块。为了最好地说明如何修改和更新运行时，您应该使用默认运行时重新启动默认节点模板，以便它开始生成块。


要使用默认运行时重新启动节点：

1. 在您的计算机上打开终端外壳。
2. 切换到编译 Substrate 节点模板的根目录。
3. 通过运行以下命令以开发模式启动本地节点：
    ```bash
    cargo run --release -- --dev
    ```

启动节点后，您可以使用使用 Polkadot-JS API 构建的基于浏览器的应用程序连接到它。


要连接到正在运行的节点：


1.在 Chrome 或基于 Chromium 的浏览器中打开 [Polkadot/Substrate 门户](https://polkadot.js.org/apps/#/explorer)。 如果您使用限制性更强的浏览器——例如 Firefox——您可能会发现 Polkadot/Substrate 门户和节点之间的连接被阻止。


2. 如有必要，连接到开发网络和默认本地节点端点 127.0.0.1:9944 。
在大多数情况下，[Polkadot/Substrate Portal](https://polkadot.js.org/apps/#/explorer) 会自动初始化与正在运行的本地节点的连接。如果需要，单击 Unknown 以显示网络选择菜单，然后选择 Development 和 Local Node，然后单击 Switch。

3. 请注意，在开发下，节点模板版本是默认版本 100。

    ![](https://docs.substrate.io/static/c2fb81c66cb08d25c1a3d253bc4e339a/01a2d/quickstart-100.avif)


## 添加托盘

开始使用 Substrate 和 FRAME 进行构建的最常见方法涉及添加 pallet，方法是从现有库中导入 pallet 或创建自己的 pallet。从头开始创建您自己的 pallet 并不困难，但它需要更多的工作来设计应用程序逻辑、存储要求、错误处理等。为了简单起见，让我们通过从现有库中导入一个托盘来添加一个托盘。


默认情况下，节点模板不包含 [Utility pallet]()。如果此 pallet 包含您要使用的功能，您可以将其添加到默认运行时。

要添加实用程序托盘：

1.在您的计算机上打开第二个终端 shell 并切换到节点模板根目录。
2.在代码编辑器中打开运行时清单 — runtime/Cargo.toml 。
3. 找到 [dependencies] 部分并将 Utility pallet 添加为依赖项。
例如，您应该添加类似于以下内容的一行。
```toml
pallet-utility = { version = "4.0.0-dev", default-features = false, git = "https://github.com/paritytech/substrate.git", branch = "polkadot-vX.Y.Z" }
```

请务必将 branch = "polkadot-vX.Y.Z" 替换为用于其他 pallet 的 Polkadot 分支。

您可以将任何现有托盘依赖项复制为模型，以确保 pallet-utility 依赖项的分支设置与所有其他托盘的分支设置相同。

4. 找到 [features] 部分并将 Utility pallet 添加到标准二进制文件的默认功能列表中。

例子：

```toml
[features]
default = ["std"]
std = [
  ...
  "pallet-utility/std",
  ...
]
```

您将了解更多有关在 [Rust 和 WebAssembly]() 中为标准和 WebAssembly 二进制文件构建功能的信息。

5. 保存更改并关闭 Cargo.toml 文件。
6. 在代码编辑器中打开 runtime/src/lib.rs 文件。
7. 为 Utility pallet 添加 Config trait 的实现。

例子：

```rust
impl pallet_utility::Config for Runtime {
  type RuntimeEvent = RuntimeEvent;
  type RuntimeCall = RuntimeCall;
  type PalletsOrigin = OriginCaller;
  type WeightInfo = pallet_utility::weights::SubstrateWeight<Runtime>;
}
```

每个 pallet 都有一个 Config 特性来表示它需要的特定参数和类型。您可以随时查看 pallet 的 Rust 文档，以了解有关其配置要求的更多信息。例如，您可以查看 [pallet-utility]() 的 Rust 文档。

8.  在 construct_runtime! 宏中添加 Utility pallet。


例子：

```rust
construct_runtime!(
 pub struct Runtime
 where
    Block = Block,
    NodeBlock = opaque::Block,
    UncheckedExtrinsic = UncheckedExtrinsic
 {
        System: frame_system,
        RandomnessCollectiveFlip: pallet_randomness_collective_flip,
        Timestamp: pallet_timestamp,
        Aura: pallet_aura,
        ...
        Utility: pallet_utility, // Add this line
        ...
 }

```

您可以了解有关 `construct_runtime` 宏如何在 [FRAME 宏]()和[运行时]()构造宏中工作的更多信息。

## 更改常量值

默认情况下，节点模板中的 Balances pallet 定义了一个 EXISTENTIAL_DEPOSIT 常量。 EXISTENTIAL_DEPOSIT 代表一个账户必须被视为有效活跃账户的最低余额。默认情况下，常量被定义为值为 500 的 128 位无符号整数类型。为简单起见，您要将此常量的值从 500 更改为 1000。

1. 在代码编辑器中打开 runtime/src/lib.rs 文件。

2. 找到 Balances pallet 的 EXISTENTIAL_DEPOSIT 。

```rust
/// Existential deposit.
pub const EXISTENTIAL_DEPOSIT: u128 = 500;
```

3. 更新 EXISTENTIAL_DEPOSIT 的值。

```rust
pub const EXISTENTIAL_DEPOSIT: u128 = 1000 // Update this value.
```

## 更新运行时版本

默认情况下，节点模板使用 spec_version 和值 100 标识 VERSION 常量中的默认运行时版本。要指示您已对默认运行时进行更改，您将更改 @2 # 从 100 到 101。



请注意，对于您在快速入门中对默认运行时所做的更改，并非严格要求更新 spec_version 。但是，通过更新版本，您可以看到执行无叉升级所涉及的基本步骤。


要更新运行时版本：

1. 在代码编辑器中打开 runtime/src/lib.rs 文件。

2. 找到 runtime_version 宏。

```rust
#[sp_version::runtime_version]
pub const VERSION: RuntimeVersion = RuntimeVersion {
    spec_name: create_runtime_str!("node-template"),
    impl_name: create_runtime_str!("node-template"),
    authoring_version: 1,
    spec_version: 100,
    impl_version: 1,
    apis: RUNTIME_API_VERSIONS,
    transaction_version: 1,
    state_version: 1,
};
```

3. 更新 spec_version 以指定新的运行时版本。

```rust
spec_version: 101,  // Change the spec_version from 100 to 101
```

4. 保存更改并关闭 runtime/src/lib.rs 文件。

此时，您已经修改了运行时代码并更改了版本信息。但是，正在运行的节点仍在使用先前编译的运行时版本。如果您仍然使用 [Polkadot/Substrate 门]()户连接到正在运行的节点，您可以看到节点模板版本仍然是默认版本 100，余额常量 existentialDeposit 的[链状态]()仍然是 500。


### 重新编译运行时

在更新节点模板以使用修改后的运行时之前，您必须重新编译运行时


要重新编译运行时包：

1. 打开第二个终端 shell 并切换到编译节点模板的根目录。
2.通过运行以下命令重新编译运行时：
```bash
cargo build --release --package node-template-runtime
```

--release 命令行选项需要更长的编译时间。但是，它会生成一个更小的构建工件，更适合提交到区块链网络。存储优化对任何区块链都至关重要。使用此命令，构建工件将输出到 target/release 目录。 WebAssembly 构建工件位于 target/release/wbuild/node-template-runtime 目录中。例如，如果您列出 target/release/wbuild/node-template-runtime 目录的内容，您应该会看到以下 WebAssembly 工件：

```bash
node_template_runtime.compact.compressed.wasm
node_template_runtime.compact.wasm
node_template_runtime.wasm
```

## 提交交易

您现在有一个更新的 WebAssembly 对象，它描述了修改后的运行时。但是，正在运行的节点尚未使用升级后的运行时。要更新存储在链上的运行时，您必须提交一个更改要使用的 WebAssembly 对象的交易。


要更新运行时：

1. 在 Polkadot/Substrate 门户中，单击 Developer 并选择 Extrinsics。
2. 选择管理 Alice 帐户。
3. 选择 sudo pallet 和 sudoUncheckedWeight(call, weight) 函数。
4.选择 system 和 setCode(code) 作为使用 Alice 帐户进行的调用。
5. 单击文件上传，然后选择或拖放您为更新的运行时生成的紧凑且压缩的 WebAssembly 文件 — node_template_runtime.compact.compressed.wasm 。


例如，导航到 target/release/wbuild/node-template-runtime 目录并选择 node_template_runtime.compact.compressed.wasm 作为要上传的文件。

6.将两个权重参数设置为默认值 0 。

![](https://docs.substrate.io/static/1abb15872875a741ba7957a77463480a/0f4c1/set-code-transaction.avif)

7. 单击提交交易。
8. 查看授权，然后单击“签名并提交”。

## 验证修改后的运行时

事务包含在块中后，您可以验证您使用的是修改后的运行时。

要验证您的更改：

1. 在 [Polkadot/Substrate Portal]() 中，点击 Network 并选择 Explorer 可以看到已经有一个成功的 sudo.Sudid 事件。

![](https://docs.substrate.io/static/acc82575c3cb918243bcb63731c7e2fd/ae6f6/set-code-sudo-event.avif)


2. 检查节点模板版本现在是 101 。

例如：

![](https://docs.substrate.io/static/d2b284ae84ff285bd030d91aa16f35b0/01a2d/quickstart-101.avif)


3. 单击 Developer 并选择 Extrinsics。

4. 单击 submit the following extrinsic 并滚动到列表底部以验证实用工具托盘是否作为一个选项可用。

![](https://docs.substrate.io/static/b6d4b1158817952d65b3969e402f57a3/58d4a/quickstart-utility-pallet.avif)

5. 点击 Developer ，选择 Chain state，然后点击 [Constants](
https://polkadot.js.org/apps/#/chainstate/constants?rpc=ws://127.0.0.1:9944)。

6. 选择balances pallet，选择existentialDeposit，然后点击+查询常量值。

![](https://docs.substrate.io/static/b0ccb722efcfbc95f68eefd19af972f0/0cdde/quickstart-chain-state.avif)




## 下一步去哪里

验证更改后，您知道您正在运行自定义版本的节点模板，并且已成功升级本地节点以使用修改后的运行时。


这是一项了不起的成就，但您还可以做更多的事情。要深入了解概念和核心组件，请查看“学习”部分中的主题，或者通过探索“构建”部分中的主题，开始构建您目前所学的内容。
