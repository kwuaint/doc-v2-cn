作为区块链不可或缺的组成部分，交易提供了修改链上存储的信息状态的机制。每个交易历史都会被记录在区块中。交易有以下三种类型：

- 签名交易 (Signed Transaction)
- 不具签名交易 (Unsigned Transaction)
- 原生交易 (Inherent Transaction)

# 签名交易

在 CESS 网络中，签名交易是最常见的交易类型。在发送签名交易请求时，必须附上请求账户的签名。通常，这是通过使用请求账户的私钥对交易内容进行签名来实现的。此外，请求账户还需要支付交易费用。交易费用和其他元素的处理方法取决于程序逻辑。

对于签名交易，只有发送方需要支付交易费用。例如，如果您需要将一定数量的代币从您的账户转账给其他人，您可以在[**Balances Pallet**](https://paritytech.github.io/substrate/master/pallet_balances)中调用 [`transfer`](https://paritytech.github.io/substrate/master/pallet_balances/pallet/struct.Pallet.html#method.transfer) 外部函数。由于您的账户发起了这笔交易，您应该使用您的账户私钥对交易进行签名。同时，在发送请求的过程中，您可以选择添加小费，以提高该交易在网络中的处理优先级。

# 不具签名交易

绝大多数交易都是签名交易。只有少数未签名交易不是由用户发起的，而是通过链下工作程序由网络请求的。

未签名交易不会收取任何费用，也不需要请求者的信息。这意味着未签名交易没有经济限制，因此可能会出现垃圾请求或重放攻击。

因此，在执行未签名交易之前，必须非常严格地检查。此外，由于未签名交易涉及用户定义的验证过程，它们通常比已签名交易消耗更多资源。

在 CESS 网络中，只有当前轮值的验证者可以发起未签名交易。请求内容需要附带特定账户的签名摘要，以通过 CESS 网络中的用户定义验证。

在 Substrate 框架中还有一个使用未签名交易的模块，即 [`pallet_im_online::pallet::Call::heartbeat`](https://paritytech.github.io/substrate/master/pallet_im_online/pallet/struct.Pallet.html#method.heartbeat) 的調用。此交易允许验证者节点向网络发送消息以确认节点在线状态。对于此交易，Substrate 有严格的验证过程，只允许注册为验证者节点的账户发送该心跳消息。这样的措施确保了网络中验证者节点的可靠性和安全性。

# 原生交易

原生交易是一种特殊类型的不具签名交易。通过这种类型的交易，出块节点可以直接将信息插入到区块中。通常情况下，此类交易不会传播到其他节点或添加到交易队列中。我们认为这类交易在没有特定验证的情况下是有效的。

在 CESS 网络中，不使用原生交易。但在 Substrate 框架中，[pallet_timestamp::pallet::Call::set](https://paritytech.github.io/substrate/master/pallet_timestamp/pallet/struct.Pallet.html#method.set) 交易使用了原生类型，将当前时间戳插入生成的区块。尽管其他节点无法确认区块信息中包含的时间戳是否正确，但它们可以确定时间戳是否在可接受范围内。如果不在范围内，它们将拒绝该区块。

# 交易费用和交易权重

在执行交易并上传数据到区块链时，会改变区块链的状态并消耗区块链资源。正如前面提到的，由于区块链中的资源是有限的，如何使用和管理资源变得重要。除了固定的存储资源外，可能还有恶意用户发送大量消息导致网络超载，从而阻止网络生成区块。

为了防止资源滥用，设计了 **交易权重** 和 **交易费用**。

交易权重和交易费用之间的关系如下：交易执行所需的时间被表示为权重。权重影响交易费用的数量。权重越大，交易费用越高。

交易费用为限制执行时间和计算执行操作所需的调用数量提供了经济激励。交易费用还用于使区块链经济上可持续（其中一部分将返回到资金库，并作为新一轮代币进行分发），因为它们通常适用于用户发起的交易，并在执行交易请求之前扣除。

{% hint style="success" %}
### 最终手续费公式

**录入费用** = 基本费用 + 字节长度费用 + (目标费用调整 * 权重)

**最终手续费** = 录入费用 + 小费
{% endhint %}

**基本费用**：这是用户为交易支付的最低金额。

**字节长度费用**：与交易编码长度成比例的费用，每个字节的费用乘交易编码的长度。

**目标费用调整**：可以根据网络拥堵情况进行调整的乘数。

**权重**：交易权重。

**小费**：发送者为了加快其交易处理速度而自愿支付的金额。