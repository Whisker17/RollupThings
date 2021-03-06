# Fucking Layer2

**一层解决不了的事情，那就再加一层来解决**

注：取这个标题并不是表达对 layer2 的不满，纯粹是为了致敬一下自己很喜欢的一个 github 的项目  [fucking-algorithm](https://github.com/labuladong/fucking-algorithm)

Bitcoin 和 Ethereum 作为最为人耳熟能详的两个区块链项目，为人诟病最多的便是其效率问题，同为金融产品，VISA 可以做到 1700 的 tps ，甚至理论上可以达到 24000 tps ，而 Bitcoin 的 tps 为 7 ，Ethereum 的 tps 则为 7-15 。Hello？请问你这些东西做出来的意义是什么呢？当然有！并且是对当前社会的组成形式的一种诉求与反抗，当然，这些东西并不会在这篇文章中体现出来。

我们可以设想一下，假设原本的一笔普通交易我们把它转化为一笔包含多笔普通交易的特殊交易，那么在一个区块里面所能打包的交易数量恒定的情况下，这样的措施是否就把原本的 tps 成倍数的增加了，这就呼应了文章开头的那句 `一层解决不了的事情，那就再加一层来解决` 。于是乎，聪明的 developers 纷纷想出了各种各样的 Layer2 方案，接下来我们就逐个进行介绍。



## 闪电网络（Lightning Network）

让我们把时间调回到 2017 年，由于 Bitcoin 的出块时间平均需要 10 分钟，同时每个区块的大小为 1

 M。那么每 10 分钟能打包的交易数量则为有限的，那么大家为了让自己的交易被打包确认的几率变大，只能不断地提高自己的交易费，于是就引发了一场对 Bitcoin 的扩容运动。主要由两方组成，一方希望做到的是链上扩容，比如加快出快的速度，或者扩大区块的大小，使其能够打包更多的交易，但是这两种方案都有自己无法避免的问题，这里先不展开去说。与此同时，有另一方想的是我们可以采取链下扩容的方案，于是**闪电网络**就应运而生了。

闪电网络是基于微支付通道衍生而来的，以 Bitcoin 作为结算基础，在链下实现了真正的点对点微支付交易。主要提出了**序列到期可撤销合约 RSMC**（Recoverable Sequence Maturity Contract）和**哈希时间锁定合约 HTLC**（Hashed Timelock Contract）。**前者解决了链下交易的确认问题，后者解决了支付通道的问题**。

### RSMC

闪电网络的基础是交易双方之间的双向微支付通道，RSMC 定义了该双向微支付通道的最基本工作方式。

举一个例子， Alice 和 Bob 如果需要借助闪电网络去进行转账，他们则需要分别往一个 2/2 签名的多重签名地址打入自己的币，比如各自打了 0.5 个 BTC。此时在 Bitcoin 的链上会产生一条交易记录通道打开时的状态，即 {Alice: 0.5 BTC , Bob: 0.5 BTC} 。之后，Alice 和 Bob 就可以进行一系列的转账，比如 Alice 给 Bob 打了 0.1 个 BTC，双方则需要对链下最新的分配结果 {Alice: 0.4 BTC , Bob: 0.6 BTC} 进行签名，同时签名作废之前的结果。如果 Alice 想要退出该通道。她可以像区块链 Commit 带有双方签名的余额结果，如果在一定的挑战期内 Bob 没有提出异议，那么区块链就会根据该结果将币转回到各自设置的提现地址。**如果 Bob 能在这段时间内提交证据证明 Alice 企图使用的是一个双方已同意作废的分配方案，则 Alice 的资金将被罚没并给到Bob 。** 为了鼓励双方尽可能久地利用通道进行交易，RSMC 对主动终止通道方给予了一定的惩罚：即**主动提出方其资金到账将比对方晚**。

### HTLC

如果 Alice 想和 Cathy 进行交易，但是她们俩之间并没有通道，与此同时 Alice 和 Bob 之间存在通道，Bob 和 Cathy 之间也存在通道，那我们是否可以通过这两个通道完成 Alice 和 Cathy 之间的链下交易呢？

这就是 HTLC 被提出的原因，我们可以通过智能合约，双方约定转账方先 Lock 一笔钱，并提供一个哈希值，如果在一定时间内有人能提出一个字符串，使得它哈希后的值跟已知值匹配（实际上意味着转账方授权了接收方来提现），则这笔钱转给接收方。

过程如下：

1. Alice 签名了一个转账密文 H 给 Cathy，由于 Alice 与 Cathy 之间没有直接的支付通道，Cathy 收不到这笔转账。现在，Alice 和 Bob 商定一个 HTLC 合约：只要 Bob 能在 2 天内向 Alice 出示正确的密文，Alice 则支付 Bob 0.1 BTC，否则钱会自动退还给 Alice 。

2. 与此同时 Bob 和 Cathy 商定一个 HTLC 合约：只要 Cathy 能在 1 天内向 Bob 出示正确的密文，Bob 则支付 Cathy 0.09 BTC，否则钱会自动退还给 Bob。

3. 现在，由于 Alice 已经把密文告诉了 Cathy，Cathy 只要向 Bob 出示正确的密文，就能收到 0.09 BTC。而多的 0.01 BTC 成了 Bob 的佣金，也可以理解为矿工费。

### Pros And Cons

首先我们要肯定闪电网络对于 Layer2 这个领域的先导作用，同时它也很好的完成了对于链下交易的基本功能的实现：

**优点**：

1. **非常低的 GAS 费**
2. **短确定时间**完成一笔 Bitcoin 交易。

**缺点**：

1. **网络的路由问题**
2. **中心化**的问题
3. **隐私的缺陷**
4. 由于锁定带来的**资产缩水**的风险

## 状态通道（State Channel）

状态通道技术，受启发于比特币的闪电网络。**状态通道是固定一组参与者（通常是两名参与者）之间的协议，用以实现安全的链下交易，其中支付通道专门用来支付**。状态通道是支付通道更为普遍的形式——它们不仅可以处理支付，也可以处理区块链的“状态更新”——就像智能合约的更改。

### Pros And Cons

**优点**：

1. **交互延迟在毫秒级别**
2. **交易手续费极低**
3. **水平扩展性强**；加节点就能增加总系统容量，TPS 无上限，且互相之间不隔离，不需要有跨分片或者跨链之类的复杂操作。

**缺点**：

1. 由于存在挑战期，退出无法做到即时性。
2. **资金利用率低**；**状态通道是要这个双方都把这个钱存到链上的通道，之后再互相进行发送支付**。**如果一个用户给另一个用户发送一大笔钱，那中间每一个转发的节点都要有这么大的容量，在现实生活中是不太可能的**。
3. **状态机只适用于这个固定的人数**。所以把一般的 dApp 迁移到状态通道中是相当困难的。即使你把棋类游戏或是稍大的 PC 游戏搬到状态通道上，这些游戏也必须写成状态机的形式。**他们每一个状态的转移，要非常清楚地写出来**。

## 侧链（Side Chain）

**侧链的本质就是在基础层上再搭一个链，然后用完全另外一套验证人**。它的整个安全性是分开的：**主链有主链的安全性，侧链有侧链的安全性**，是一种跨区块链的解决方案。

通过这种解决方案，可以实现数字资产从第一个区块链到第二个区块链的转移，又可以在稍后的时间点从第二个区块链安全返回到第一个区块链。侧链通过**双向挂钩**连接到一个基础层协议。由于没有第一层设计的负担，**侧链可以支持超出其基础层能力的某些特性，包括但不限于可扩展性和互操作性，同时不依赖于第一层的存储**。

侧链会通过**选择性地将区块头的 snapshot 发送回主链**从而避免分叉导致的作恶行为。侧链上的分叉选择遵循一个原则：**合法的链必须构建在最近一个进行过 snapshot 的区块之上** 。对于状态通道来说，其安全性就是双方互相签过名，就具备主链的安全性。只要一方做恶，另外一方都可以提交到主链，把它这个争议解决掉。而侧链的话就是你要信任多数的验证人是好人，所以它的安全性要比主链低很多。

### 无效状态转移

由于侧链并不会将所有在侧链生成的区块返回到主链去进行验证（否则就没有做侧链的意义了），那么如果有超过 50% 或 66%（取决于侧链的架构）的验证者串谋的话，他们可以创建一个完全无效的区块，窃取其他参与者的资产，并将这个区块的 snapshot 发送至主链，发起并完成一个退出交易，就可以成功偷走这些资产。

### Pros And Cons

**优点**：

1. 侧链的延迟是相对低的，比状态通道的毫秒级高一些，比主链的十几秒几十秒延迟低很多。
2. 代码和数据独立，不增加主链的负担。

**缺点**：

1. **安全性稍弱**

## Plasma

Plasma 结构是通过使用**智能合约**和 **Merkle 树**来构建的，从而可以创建无限数量的子链——其本质上是父以太坊区块链的**迷你副本**。其主要思想就是建立一个**侧链框架**，它将尽可能少地与主链进行通信和交互。每条链都以各自的方式工作，通过共存和独立运行来满足不同的需求。 在每个子链的顶部，可以创建更多链，这就是构建树状结构的原因。

具有状态转换的 Plasma 资产的存款和取款可通过**欺诈证明**来实现。 这确保了**可执行的状态**和**可交换性**。 它还允许**在基础层上以较少的数据负载处理大量交易**。 任何用户都可以向其他人发送资产，包括来自不同参与者的资金。 可以使用原生平台代币支付和提取这些资金转账。

### 欺诈证明

子链和根链之间的通信通过**欺诈证明**来保证安全性。每条子链都有自己特定的验证块的机制和防欺诈实现，同时也可以**基于不同的共识算法构建**。最常见的是 PoW 工作量证明，PoS 权益证明和 PoA 等等。

欺诈证明可确保**一旦有恶意活动，用户能够报告不诚实的节点，保护其资金并退出交易**（这涉及与主链的交互）。换句话说，欺诈证明被用作 Plasma 子链向其父链或根链提出申诉的一种机制。

这些证明使用**交互式提款协议**。为了提取一定数量的资金，需要**退出时间**。退出方确认输出以请求退出。然后，网络参与者可以提交一份证明，即如果已花费任何资金，则必须进行确认和测试。如果有所出入，则将其视为欺诈，并取消确认。随着时间的流逝，另一个绑定回合允许撤回发生，并在提交的时间戳之前回复到原状态。参与者可能会迅速退出错误的 Plasma 链。发生攻击时，参与者可以快速退出并节省成本，从而确保系统内的安全性。

### Plasma MVP 以及 Plasma Cash

Plasma 作为一个扩容框架，众人对此进行了衍生，于是产生了多种 Plasma ，这里我们介绍两个比较常见的：

#### Plasma MVP

**Plasma MVP(Minimal Viable Plasma)，最小可行性的Plasma**

在每个 Plasma chain 中会有一个 operator，operator 负责**产生区块**。如果我们需要准入 Plasma chain 的话就需要进行一个 deposit 的过程，**此后 Plasma chain 每产生一个区块就必须和主链回馈**，即 Merkle root ，这样 Plasma chain 上产生的块才算是被 confirm。

由于每个 Plasma chain 之间是独立的，所以**不能直接进行跨链交易，必须借助主链才能进行**。

![img](https://github.com/Whisker17/Layer2Things/raw/main/plasma/src/98886683-7b46cd80-24cf-11eb-8196-f3078fa060b2.png)

可以将每条 Plasma chain 想象成一件赌场，我们在进入赌场前需要把自己的钱兑换成赌场自己的筹码，这就是一个 deposit 的过程，在这间赌场里你的筹码是有潜在效益的，但是对于隔壁的赌场而言（另一条 Plasma chain）是完全没有意义的，当我们需要在另一间赌场里进行活动的时候，我们就需要把这些筹码兑换成现实中的货币，然后再去那间赌场将现实货币兑换成相应的筹码，这就相当于一个跨链的交易过程了。而赌场里的交易需要记录，然后和政府报备之后这些交易才具有效益。

当然，我们为了保证安全性，在兑换筹码的时候需要有一定时间的缓冲期，这段时间就被称为**挑战期**，如果在这段时间里有人提出你在赌场造假的证据，那你就无法兑换成现实货币了。

#### Plasma Cash

相较于 Plasma MVP ，Plasma Cash 对此做了两处修改：

1. 每一笔存到 Plasma contract 的钱，都会赋予一个 unique token ID
2. Merkle tree 的 index 存的是 token ID，内容存的是这个 token ID 的交易记录

这样的话，**你每次存进去的钱都是独立的**，当有人想要去偷钱的时候，他需要一次次去分散操作，**但是这样也让这样两笔钱你无法合并使用**。而第二点，相当于 token ID 作为索引，**加快查询的速度**，但是与之而来的是**对于存储的要求的提升**。

### Plasma 的问题

Plasma 通过将高频的交易迁移到以太坊网络之外的 **侧链** 之上，**定期将批量交易的哈希值发布到以太坊主网**，然后设置一些 **防恶意攻击机制** ，确保资金安全性。最终的目的是为以太坊扩容，提升交易吞吐量，减少交易成本。

但是最终暴露出两个问题：

1. **数据可用性** ：因为仅将批量交易的整体哈希值发布到 Layer 1 上，而不是每一笔交易均发布到底层公链，所以具体的交易数据不存在 Layer 1 上，**用户需要自己存储具体的交易数据**。
2. **用户体验差** ：为了避免恶意攻击，Plasma 在设计挑战期的机制的时候，用户需要**定期上线网络**，否则可能错过而遭受不必要的损失。它**只能够把支付做好**，**对于稍微复杂的智能合约却无能为力**。**作恶的一方是不会把数据给你提交上链，以至于这种争议至少要等两个星期才能解决**。这个过程是对于用户来说难以接受的。

在最坏的情况下，如果所有用户都需要退出一个 Plasma 链，那么**该链的整个有效状态必须在一个挑战期内发布到以太网主网上**。考虑到 Plasma 链可以任意增长，而以太坊区块已经接近其容量，几乎不可能将整个 Plasma 链倾倒到以太坊的主网上。因此，几乎可以肯定的是，大量退出会把以太坊挤爆。这就是所谓**批量退出问题**。

与状态通道相比较而言，当同一通道内的所有用户都同意退出时，状态通道可以即时退出，但是在 Plasma 中，用户必须等待一段时间，才能退出。与此同时状态通道也比 Plasma 更便宜，更便捷。

## Rollup

Rollup 方案可以被认为一种**压缩技术**，多笔交易可以压缩在一起（**几千笔交易可以被打包到一个 Rollup 区块中**），既能**减少交易数据规模**，又能**降低交易验证负担**，因此使得以太坊区块链能处理更多交易，tps 可达到 3000 左右。它是将所有 layer2 上的交易数据，也就是 Rollup 区块的快照发送到主链上某个智能合约内，用主链上的单个合约来保管所有的资金，而 Rollup 则通过在主链上为每一笔交易公开一些数据，让任何人都能**通过观察区块链上的 calldata （交易输入数据）来获得 layer2 的所有数据**。**Rollup 区块的状态是由用户以及链下运营者来维护的，因此不会占用主链的存储空间**。所有交易的收据都存储在以太坊区块链上，这就提升了 Layer2 交易的安全性。同时这也**解决了 Plasma 的数据可用性问题**，且用户**验证的结果具有唯一性**，验证人链下把这个智能合约跑一遍，就会发现验证链下的计算是否按照链上的智能合约。从经济学角度来说，一般的验证人不太会去做恶，因为他的质押额太大了。

### Optimistic Rollup

由于 Plasma 的缺陷比较明显，同时也比较难以从根本进行解决，于是 Plasma 团队最终选择重组成为 Optimistic Rollup 团队。Optimistic Rollup 保留了 calldata，可以**主链获得所有 layer2 的数据**，但那些**刷新 Layer-2 状态的交易不会在链上被验证**，只让主链存储一系列的历史状态根，添加了一个新的状态的一段时间（例如 1 周）后才将新状态最终敲定，也就是**数据可用性放在链下**。采用**欺诈证明**（跟 Plasma 方案一样），对提交无效状态的执行者进行惩罚。其**链下 OVM 虚拟机可以支持任意智能合约逻辑的实现，与以太坊 EVM 虚拟机搭配使用**，开发者就可以用 Solidity 来写码，实现 dapp 和智能合约之间的无缝互操作性。

下图是 EVM 和 OVM 的一个对比：

![EVM 和 OVM 的对比](https://github.com/Whisker17/Layer2Things/raw/main/rollup/oprollup/src/98760055-686bc480-240d-11eb-8f4b-9cc4cde919ec.png)

#### Pros And Cons

**优点**：

1. 通用计算的灵活性（图灵完备 / 兼容 EVM）
2. 可扩展性的增加（每秒 200 到 2000 个交易（tps），以太坊第 1 层当前为 10 tps）
3. 所有数据都可以在链上使用（无需信任链下数据提供者）
4. 更好的用户体验

**缺点**：

1. 与某些其他第 2 层解决方案（ Plasma，ZK-Rollup 等）相比，吞吐量受到限制
2. 存在一些其他安全问题
3. 具有一种**反网络效应**，即**单个 Optimistic Rollup 的交易量越大，全节点验证者会越少**，即，依靠 **1 of N** 的诚实假设是更不安全的。**1 of N** 诚实假设是指，参与者虽然有 N 个，只要其中**至少有一个按期望运作，系统就会正常工作**。任何基于欺诈证明的系统都属于这一类，可信设置也是如此，尽管在这种情况下，N 通常更小。但你希望 N 尽可能地大。

### ZK Rollup

ZK Rollup 是靠着在**主链完成零知识证明**，**链上无需包含签名数据**，因为零知识证明就足以证明交易的有效与否，交易有效性就立刻确认，**保证无效的状态绝不会发生**，也即**数据可用性放在链上**，所以 ZK Rollups **对数据存储方面也带来了一定程度上的扩展性提升**。但由于零知识证明生成的复杂性，目前适合简单的转账。

ZK Rollup 是一个类似于 Plasma 的架构，并可**打包**交易，它无需依赖于对运营商的信任，打包的正确性可以通过使用一个 **SNARK 的链上证据来证明**。将交易数据作为函数参数发布意味着**可以在发布时对其进行验证**，随后将其丢弃，这样就不会占据以太坊的存储空间。ZK Rollup 可以在不牺牲可负担性或安全性的情况下，避免 Plasma 的退出游戏和挑战期。

ZK-Rollup 方案包括两种类型的用户：**交易者**和**中继者**。交易者创建交易，并广播到网络。交易数据中包含 “from” 和 “to” 地址的索引，以及交易的量，网络费用和随机数组成。地址的缩短的 3 字节索引版本减少了处理资源需求。交易的值大于或小于零将分别创建存款或提款。智能合约将数据记录在两个 Merkle 树中；一棵 Merkle 树是**地址**，另一棵是**交易额**。

中继器收集大量交易以创建 Rollup 。**生成 SNARK 证明是中继器的工作**。 SNARK 证明是代表区块链状态增量的哈希值。状态是指“存在状态”。 SNARK 证明将交易之前的区块链快照与交易之后的区块链快照（即钱包值）进行比较，并仅将可验证哈希中的更改报告给主网。

值得注意的是，只要将智能合约中要求的保证金押入赌注中，任何人都可以成为中继器。这使中继器不能篡改或阻止 Rollup 。

#### Pros And Cons

**优点**：

1. 降低每次用户交易的费用
2. 比 Plasma 和 Optimistic Rollup 更快
3. 区块将在鼓励去中心化的并行计算模型中进行计算
4. 每个交易中包含的数据更少，从而提高了 layer2 的吞吐量和可伸缩性
5. 不需要像 Optimistic Rollup 那样的欺诈证明，Optimistic Rollup 提款最多可能会延迟两周

**缺点**：

1. 计算零知识证明的难度将要求数据优化以获取最大吞吐量
2. ZK-Rollups 的**初始设置更倾向于中心化方案**
3. 该安全方案假定不可验证的信任级别
4. 它对于这个节点要求是非常非常高的。**ZK Rollup 就是耗 CPU 和内存，并需要很强的服务器去做这个打包**。

## Fraud Proof vs Validity Proof

通过我们上述对于 layer2 的方案的介绍，我们发现 Layer2 扩容的思路简而言之是**将许多笔交易用一个证明所取代以节省 Layer1 的资源**，相应的，这个应该可以证明对应的多笔交易的有效性。自然地，证明的方式可以分为两种：

- **欺诈证明**，提出证据以**证明链上的状态转换并不正确**；
- **有效性证明**，提出证据以**证明链上的状态转换正确**。

### 欺诈证明

欺诈证明对应的 Layer2 扩容方案**乐观地假设链下交易均真实有效，直到某方提出证据证明其存在错误**。多数我们熟悉的扩容方案都属于欺诈证明类方案，其中包括**状态通道**（State Channel）、**Plasma**、**Optimistic Rollup** 等，这些方案**在作恶者为少数的系统中非常高效**，因为多数交易无需证明即被乐观地认为是真实有效的，证明这些交易真实有效所需要的开销被节省了。

但为了避免恶意交易成为漏网之鱼，采用欺诈证明的系统必须留出足够的**容错时间**或者说质疑时间来接收欺诈证明**以证明某些交易并不真正有效而将之取消**。**在质疑时间之内，Layer2 上的交易仍然有被修改的可能，关联账户的余额在 Layer1 上仍将被锁定，无法退出**。同时，这种机制的有效建立在**被害者有能力在质疑时间内认识到系统的错误并提交欺诈证明以纠正的在线假设（Liveness Assumption）基础之上**。在这一假设中，对错误的沉默等同于对错误的默许。攻击者常常通过各种手段**阻止利益相关方在质疑时间之内提交欺诈证明而完成攻击**。欺诈证明类方案必须在质疑时间的长短上做出取舍。

### 有效性证明

有效性证明类方案，如 **ZK Rollup** 与 **Validium** 等，相比欺诈证明稳重许多，**系统每次在 Layer1 上进行状态转换时都提交足以证明其转换真实有效的证明供 Layer1 节点验证**。有效性证明因此**无需设置质疑时间**，**从 Layer2 退出无额外延迟**。**如何节约生产证明的开销是有效性证明类扩容方案的艺术**。

**零知识证明**是一种精巧有效性证明方式，它使得 Layer1 的验证节点**无需获取链下交易的全部信息并进行重复计算即可证明其有效性**。零知识证明算法的研究一直处在密码学领域的前沿，由于涉及复杂的数学和安全性证明，零知识证明的研究进展相较应用技术缓慢许多。目前主流的零知识证明算法包括 zk-SNARK、zk-STARK、PLONK 和 SONIC 等。好的零知识证明算法应该是验证开销低、生产快的。值得注意的是，目前实用的零知识证明算法生成证明的时间大约在 1~10 分钟，**这是额外的无法消除的交易确认延迟**。另外，由于零知识证明的生产需要大量计算，开发者通常通过开发电路（由低级语言编写的特定逻辑运行程序）减少计算时间，这使得零知识证明更倾向于**运算格式一致**的交易，也因此更适用于**去中心交易所等需要处理的交易类型固定、合约简单的应用**。