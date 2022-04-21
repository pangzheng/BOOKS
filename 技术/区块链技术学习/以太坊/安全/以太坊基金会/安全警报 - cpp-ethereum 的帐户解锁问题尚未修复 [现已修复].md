# 安全警报 - cpp-ethereum 的帐户解锁问题尚未修复 [现已修复]
- 受影响的配置： cpp-ethereum (eth, AlethZero, ...) 版本 1.2.0 到 1.2。6 注意：“geth”、“Mist”和“Ethereum Wallet”（除非明确与 cpp-ethereum 一起使用）都不受此影响，它们会再次正确锁定帐户。

这只是一个简短的提示，cpp-ethereum 围绕帐户安全的安全问题尚未正确修复。作为 cpp-ethereum 版本 1.2.6 的一部分的修复并没有降低安全性，但它也没有完全修复错误。我们将努力发布另一个错误修复版本，并在修复得到正确验证后宣布。

现在有一个修复：版本 1.2.7 - https://github.com/ethereum/webthree-umbrella/releases/tag/v1.2.7