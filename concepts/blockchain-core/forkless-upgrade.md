# 升级 Runtime

无分叉升级是用于区块链开发的 Substrate 框架的一个定义性特征。 在不分叉代码库的情况下更新 runtime 逻辑的能力使您的区块链能够随着时间的推移而发展和改进。 通过将 runtime 执行环境 (runtime WebAssembly blob) 的定义作为区块链 runtime 状态中的一个元素，可以实现此功能。

因为 runtime 是区块链状态的一部分，网络维护者可以利用区块链的功能来实现无信任、去中心化的共识，以安全地增强 runtime。

在用 runtime 开发的框架 [**FRAME**](https://docs.substrate.io/learn/runtime-development/#frame)时，系统库定义了用于更新 runtime 定义的 `set_code()` 调用。 因此可在不关闭节点或中断区块链操作的情况下升级 runtime。 如果 runtime 升级需要更改现有状态，则很可能需要[存储迁移](data-migration.md)。

# Runtime 版本

编译 runtime 时，节点将生成 **原生二进制文件** 和 **WebAssembly 二进制文件**。您可以使用不同的命令行执行方法来控制在区块生成过程中使用哪一个二进制文件。选择 runtime 执行环境的组件称为执行器。 尽管您可以自定义不同的执行策略来覆盖默认选项，但在大多数情况下，执行程序通过评估本机和 WebAssembly runtime二进制文件的以下信息来选择最恰当的二进制文件：

- [`spec_name`](https://paritytech.github.io/substrate/master/sc_cli/struct.RuntimeVersion.html#structfield.spec_name)
- [`spec_version`](https://paritytech.github.io/substrate/master/sc_cli/struct.RuntimeVersion.html#structfield.spec_version)
- [`authoring_version`](https://paritytech.github.io/substrate/master/sc_cli/struct.RuntimeVersion.html#structfield.authoring_version)

为了向执行器提供这些信息，runtime 中包含以下 [`RuntimeVersion`](https://paritytech.github.io/substrate/master/sp_version/struct.RuntimeVersion.html) 结构体：

```rust
pub const VERSION: RuntimeVersion = RuntimeVersion {
  spec_name: create_runtime_str!("cess-node"),
  impl_name: create_runtime_str!("cess-node"),
  authoring_version: 1,
  spec_version: 100,
  impl_version: 1,
  apis: RUNTIME_API_VERSIONS,
  transaction_version: 1,
  state_version: 1,
};
```

# 无分柝升级 runtime

传统区块链在升级其链的状态转换功能时需要硬分叉。 硬分叉要求所有节点操作员停止其节点并手动升级到最新的可执行版本。 对于分布式生产网络，协调硬分叉升级可能是一个复杂的过程。

`RuntimeVersion` 属性使基于 Substrate 的区块链在不引发网络分叉的情况下，能够实时升级 runtime 逻辑。

为了执行无分叉 runtime 升级，Substrate 使用现有的 runtime 逻辑将存储在区块链上的 wasm runtime 更新为具有新逻辑并影响共识机制的新版本。 作为共识过程的一部分，此升级被推送到网络上的所有完整节点。
