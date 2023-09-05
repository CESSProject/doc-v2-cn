当链的运行逻辑被修改后，如果需要升级链，可能会涉及数据存储结构的变更。在这种情况下，用户需要定义数据迁移逻辑，以将旧数据存储到新结构中。

# 存储迁移

存储迁移是用户定义的一次性功能，允许开发人员重新设计现有存储，以便将其转换为符合更新预期的格式。例如，想象一次 runtime 升级，将用于表示用户余额的数据类型从无符号整数更改为有符号整数 - 在这种情况下，存储迁移会将现有值读取为无符号整数，并写回已转换为有符号整数的更新值。如果在需要时未执行存储迁移，将导致运行时执行引擎错误解释表示运行时状态的存储值，从而导致未定义的行为。

# 框架实现

FRAME 存储迁移是通过 [`OnRuntimeUpgrade` trait](https://paritytech.github.io/substrate/master/frame_support/traits/trait.OnRuntimeUpgrade.html) 来实现的，内有一个函数 `on_runtime_upgrade()`。该函数内您可以自定义迁移逻辑，在 runtime 升级后，以及 `on_initialize` 函数执行之前，该逻辑会被执行。

# 迁移测试

存储迁移过程中，测试非常重要。在 `OnRuntimeUpgrade` trait 中，定义了两个函数 `pre_upgrade()`和 `post_upgrade()`，以供测试使用。

如需更具体的方法，请参考官方的 [Substrate 迁移示例](https://github.com/paritytech/substrate/pulls?q=is%3Apr+is%3Aclosed+label%3AE0-runtime_migration)。
