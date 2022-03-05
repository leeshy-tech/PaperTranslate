# Wireless SDN Mobile Ad Hoc Network:from Theory to Practice.

## 论文概况

[https://ieeexplore.ieee.org/abstract/document/7996340](https://ieeexplore.ieee.org/abstract/document/7996340)

[2017 IEEE International Conference on Communications (ICC)](https://ieeexplore.ieee.org/xpl/conhome/7985734/proceeding)

## 摘要

> D2D （device to device） MANET （移动自组织网络）	VANET（车辆自组织网络）

> The cellular base station can help coordinate all the devices in the ad hoc network by reusing the software tools developed for software-defined networks (SDNs), which divide the control and the data messages, transmitted in two separate interfaces. 

通过重用SDN软件工具（它把控制信息和数据信息分开，并在两个独立的接口中传输），蜂窝基站可以帮助协调ad hoc网络中的所有设备。

在本文中，我们提出了一个SDN MANET的实际实现，详细描述了我们采用的软件组件，并为我们开发的所有新组件提供了一个存储库。这项工作可以作为无线网络社区设计具有SDN功能的新试验台的起点，该试验台可以具有D2D数据传输的优势和集中式网络管理的灵活性。我们将其与分布式自组织网络相比，展示了其应用于真实设备的性能，证明了它的可行性。

## 引言

根据思科[1]最近的一份报告，通过无线网络传输的数据流量正在不断增加，不久将超过有线流量。在未来的5G场景中，无线流量的模式将对本地流量产生越来越大的需求。比如说设备产生了这样的流量：智慧家庭或智慧城市中的物联网传感器，朋友用智能手机在平台上分享了一个视频剪辑，或者自主驾驶车辆需要分享即时的交通信息。

> local traffic：data connections where the source and the destination of the traffic flow are in close proximity.
>
> 本地流量：指的是信息流的源和目的距离很近的数据连接。

使用当前的网络标准，即使是本地数据也要通过蜂窝网络传输到一个集中的实体，然后再重新分发。对于本地流量的交换来说，这种方法显然不是最优的。一种更有前途的方法依赖于设备到设备(D2D)技术[4]，其中设备可以在MANET或者VANET[6]中以分布式方式组织。在[7]中可以找到这样一个网络的实际实现，包括一些优化功能。

这种类型的网络的特点是没有一个中心实体。中心实体，在Wi-Fi网络中称为接入点(AP)，在蜂窝网络中称为基站(BS)，是开始拓扑的中心，它可以组织网络并将每个数据包路由到预定的目的地。在没有集中控制的情况下，每个节点必须独立行动，做出路由决策，并在移动网络中动态地适应快速变化的拓扑结构。在文献中，提出了几种解决方案，在适应动态网络的情况下，以分布式方式进行路由决策。特别时，当流量是偶发的，解决方案是一种仅在需要时才寻找路由路径的反应性协议，如按需距离矢量(AODV)协议[8]。在几个源-目的对之间的流量比较正常的情况下，预先定义网络中每个可能的节点对之间的路由的主动协议可能会工作得更好，例如，优化的链路状态路由（OLSR）协议[9]。

后一种方法的主要问题是，它需要节点之间频繁的控制消息交换，以维护每个节点上更新的网络拓扑。这些控制消息可能会导致高开销，更重要的是，在拓扑快速变化的情况下，它们可能不足以在每个节点上提供更新的拓扑信息。

在非拓扑路由中，为了减少控制消息的数量，提出了一种主动路由和反应路由的混合方法。该方法将网络划分为多个集群，如果源节点和目的节点在同一个集群中，则使用OLSR等主动路由策略;否则，采用反应性策略，如AODV。这种算法就是区域路由协议ZRP (zone routing protocol)[10]。这种混合方法的优点是可以显著减少控制消息的数量，因为更新的拓扑信息必须只在每个集群中的节点之间进行维护。主要的缺点是，当一个包被发送到一个属于不同集群的节点时，会引入显著的延迟，因为新的路径应该以响应式的方式搜索。

> Some assumptions about nodes’ mobility in the network are at the basis of all these approaches, and these assumptions allow us to optimize the tradeoff between reducing the overhead, in terms of control messages exchanged in the network, and having a perfect knowledge of the network topology, which allows the design of optimal traffic routes, but at the cost of more frequent control messages.

这些方法的基础是对于节点移动性的一些假设，它允许我们在减少开销（网络中交换的控制信息）和完全掌握网络拓扑之间做出权衡，如果要设计更好的传输路线，则要更频繁地交换控制信息。虽然根据网络移动性找到一个最优的权衡是有可能的，但是处理一个移动特征未知或者可以随时间变化的网络是十分困难的。

为了适应这种网络不断变化的移动特性，我们提出了物理划分数据平面（包括节点之间本地交换的所有数据流量）和控制平面（包括所有的控制报文和本地路由决策）。这种逻辑借用了SDN范例[11]，它在有线网络中被用于提供对包路由的集中控制，即使网络拓扑不包括接收和重定向所有流量的中央单元。

在本文中，我们研究了在使用真实设备的MANET中实现SDN架构所涉及的主要问题，并提出了一种使用运行Linux操作系统的设备的SDN模式的实际实现。在我们的设备中，我们有两个用于 DP （data plane）和 CP（control plane） 的无线接口。 对于 CP，我们构建了一个星型拓扑，其中每个节点都直接与集中式单元 (CU) 通信，该集中式单元 (CU) 以集中的方式做出路由决策，几乎实时地了解拓扑。

这个框架有两个主要优点。

- ad hoc网络中引入的开销是最小的,因为每个节点只需要知道它的邻居并且把该信息传给CU。
- 网络中每个节点的复杂性从根本上减少,因为路由算法运行在CU。

本工作可以作为实现一个无线SDN测试平台的起点，提供一个通用的SDN架构，以及如何在考虑到商业设备的限制和约束的情况下实现该架构的实际建议。本文的主要贡献如下:

- 在第二节中描述了相关工作之后，在第三节中，我们提供了我们的SDN MANET体系结构的每个组件的逻辑细节，根据设备中运行的Linux操作系统的限制，解释了我们所做的每个设计选择的原因。
- 我们的SDN架构的实现在第4节中描述，特别注意由MANET拓扑施加的设计约束;本文中引用我们的SDN MANET实现使用的代码，可以在无线网络社区[12]找到我们提供的源代码。
- 在第五节中，我们强调了我们的方法与具有OLSR的分布式最先进的方法相比的优势;该结果是在实际的物理设备上实现的。最后，第六节对本文进行了总结。

## 相关工作

第一个无线SDN测试平台是在[13]中提出的，使用Wi-Fi路由器，评估Wi-Fi和WiMAX之间切换的性能。

无线网状网络中SDN模式的另一个实际实现是使用NOX控制器[14]实现的。在本文中，CP和DP使用两个独立的服务集(ss)，具有相同的IEEE 802.11接口。OpenFlow控制消息使用一个SS标识符(SSID)交换，而数据包使用不同的SSID传输。

在OLSR-to-OpenFlow (O2O)架构[15]中，提出了一种混合方法。根据这种方法，选择网络中的一个节点作为控制器。OLSR用于初始阶段，所有拓扑和通道信息都被发送到控制器。然后利用OpenFlow在控制器上做出整个网络的路由决策，并将路由规则分发到网络中的所有节点。当拓扑发生变化时，再次使用OLSR将更新后的网络信息发送给控制器。

O2O方法的主要优点是它可以实现一个集中的路由算法，因为所有的网络信息都在控制节点上可用。主要的缺点是，当拓扑发生变化时，网络更新非常慢，因为它们需要从网络中收集大量信息。

在我们的方法中，通过利用两个无线接口，我们保持了O2O的优势，同时提供了额外延迟的解决方案，因为我们的CP是在星型(单跳)拓扑中组织的。

## SDN MANET的结构

在本节中，我们将介绍SDN MANET的设计架构。在DP中，我们只需要选择一个合适的MAC层，它允许我们建立一个ad hoc网络。对于CP，我们应该选择1)SDN协议，它允许交换所有需要的控制数据包;2)无线交换机，允许每个节点在多跳拓扑中选择下一跳;3)位于CU的控制器接收所有拓扑信息并运行路由算法。在下文中，我们将描述所有这些组件。

### DP MAC协议

DP的MAC协议应该在没有授权的频段内运行，没有一个集中的协调。一个自然的选择是带有避免冲突(CSMA/CA)协议的载波侦听多址访问，即标准的IEEE 802.11网络。问题是标准的IEEE 802.11是在基础架构模式下，因此需要存在一个AP来接收和重新转发数据包，而它不是为对等网络设计的，比如我们的场景中的网络。IEEE 802.11s (Open Mesh Wi-Fi)也不能使用，因为它的MAC层有一个内置的交换机制，这将不允许SDN控制器对路由有完全的控制。

取而代之，我们采用了ieee802.11的P2P模式独立基本服务集(IBSS)[16]，这是ieee802.11的特设模式标准，根据我们的方法的需要，将交换能力完全控制给上层。

### SDN协议

SDN协议应该定义一组api，允许在充当控制器的CU和网络节点之间交换控制包。在现有的几种SDN协议[17]-[23]中，Open Flow协议[24]非常流行，尤其在数据中心中得到了广泛的应用。与其他竞争协议相比，OpenFlow有几个优点。

首先，它提供了对MAC层、网络层和传输层参数的访问，以及修改所有这些参数的内置方法。如果使用跨层控制器，这种灵活性对于优化网络特别有用。

OpenFlow的另一个优点是有几个支持它的开源软件交换包，例如[25]、[26]。

### 多跳无线交换机

我们的目标是实现一个多跳网络，在这个网络中，我们可以通过 SDN 架构使用集中控制器控制路由。 在 Linux 系统中，有两种主要方法可以修改网络中的路由路径。 第一种方法（A1）基于直接修改每个节点的路由表。 这需要一个协议，该协议根据从节点中运行的 SDN 应用程序接收到的信息修改 Linux 内核中的 NET 层。 第二种方法（A2）基于使用 SDN 模块（无线交换机），它是 Linux 内核顶部的软件组件，可以将 SDN 控制器做出的路由决策付诸实施，即实际上将新的目标地址附加到每个数据包。

我们选择 (A2) 是因为它与之前的 SDN 框架兼容，以便为网络社区提供更通用的测试平台。 在支持的无线交换机中，我们选择 Open vSwitch (OVS) [25] 和 Centro de Pesquisa e Desenvolvimento em Telecomunicacoes (CPqD) 交换机 [26]。 两者都提供了我们场景中所需的功能，但我们实现了 OVS，因为与 CPqD 相比，它具有更简单的架构，并且它支持更多版本的 Linux 内核，从而在选择节点设备时具有更大的灵活性。

### 控制器

SDN控制器是安装在CU中的一个软件组件，负责MANET的管理，特别是路由决策。SDN控制器的工作原理如下:首先，它请求每个OpenFlow交换机发送链路层发现报文(link layer discovery packet, lldp)来发现邻居，并将收到的所有lldp转发给控制器。通过这种方式，SDN控制器对网络拓扑有一个完整的(并且更新的)了解。然后，控制器根据拓扑结构，采用集中式的基于拓扑的路由算法决定每对节点之间的所有路由。

对于兼容OpenFlow[14]、[27]-[30]的SDN控制器，有几个选项可用，它们根据实现的路由协议而有所不同。开发者社区支持的控制器有:ON开发的Open Network Operating System (ONOS)[31]。选择Lab是因为它可以为这个项目提供大量的资源。

在图 1 中，我们展示了 ONOS 的架构。 网络信息，以链路信息的形式，由 OpenFlow 收集并通过南向连接传递给 ONOS 的分布式核心。 分布式核心负责将这些信息转换为路由算法可以处理的格式。 然后，此新信息将向上（通过北向连接）发送到正在运行路由算法的应用程序。 有几个可用的应用示例，包括边界网关协议 (BGP) [32] 和开放最短路径优先 (OSPF) [33]。 通常，任何类型的集中式路由算法都可以在这里实现。

## 多跳SDN MANET的实施

在本节中，我们提供实现SDN MANET的实现细节，其体系结构详细介绍在第三节。特别地，我们将概述为实现每个SDN节点所选择的平台，并提供每个节点的Linux内核中所需修改的详细信息。然后，我们讨论了SDN框架对多跳SDN网络中数据包的影响，逐步显示了数据包头部的所有修改。

### A 平台

为了与 OVS 兼容，运行所有 SDN 代码的平台选择仅限于运行 Linux 操作系统的设备。 选择 Linux 操作系统的另一个原因是，在 Linux 设备中实现的任何解决方案也可以轻松移植到其他基于 Linux 的系统，例如 Android 智能手机或平板电脑。

在可用的设备中，我们选择了 Rasp berry Pi (RPi) Model B+。 我们选择背后的原因解释如下。 1) 为了给网络社区提供一个有用的工具，我们选择了 RPi，因为它拥有一个庞大而活跃的开发者社区，他们可以快速回答任何软件问题。 2) 感谢活跃的开发者社区，最新的 Linux 内核版本可用于 RPi。 3）这些设备的成本降低对于计划构建大型SDN测试平台的开发人员来说是一个很大的优势。

在图 2 中，我们展示了 SDN 节点的实现方案。 在图中，我们观察到一个 SDN 节点，它由两台设备组成，一台主机和一台交换机，通过以太网电缆连接2。 特别地，交换机负责管理两个无线接口，这些接口指向控制器（作为 CP 的一部分）和网络中其他 SDN 节点（作为 DP 的一部分，即 MANET）。

### B Linux内核

为了实现SDN架构，我们需要修改交换机设备的Linux内核，允许CU直接控制节点的路由。 主要修改如图 3 所示。

在图 3-(a) 中，我们展示了标准 Linux 内核协议栈中从物理层 (PHY) 到应用层 (APP) 的逻辑数据流，无需任何修改。 首先，我们将注意力集中在图中最左侧的以太网帧的数据流上。 以太网帧通过铜缆（在 PHY 层）到达设备，并被发送到以太网驱动程序（在 MAC 层），在删除 PHY 标头并将它们发送到以太网堆栈之前检查它们的完整性。 在以太网堆栈中，在将数据包发送到路由所在的 NET 层之前，MAC 标头也被删除。

如果从无线接口接收到 Wi-Fi 数据包，则数据包首先被发送到 Wi-Fi 驱动程序，在那里去除 PHY 标头，然后由特定模块处理，将 Wi-Fi 数据包转换为 以太网数据包，随后被发送到以太网堆栈。

在图 3-(b) 中，我们展示了逻辑数据流，包括插入 Linux 内核以允许 SDN 控制器修改每个节点的路由行为的新模块。 特别是，SDN 桥接器是以太网/Wi-Fi 驱动程序和以太网堆栈之间的一个新模块，它可以将传入的数据报发送到 SDN 块进行进一步处理。

在 SDN 块中，数据报可以根据从 SDN 控制器接收到的规则进行修改。 特别是，为了控制路由路径，SDN控制器强加的规则涉及修改相应数据报的MAC头。 在这些修改之后，数据报将被发送回 SDN 网桥，并转发到以太网堆栈进行与图 3-(a) 的情况相同的处理。 本工作中提供的所有新 SDN 模块都可以在 [12] 中下载。

我们在此强调 SDN 规则通过 SDN 应用程序（在 APP 层中运行）与 SDN 块通信的事实。 此应用程序通过 CP 使用相同的协议栈直接与 CU 中的控制器通信，但对于 DP 数据包使用不同的无线接口。

### C 实践中的SDN多跳网络

为了解释SDN框架如何解释和修改多跳网络中的每个数据包，我们提供了一个清晰的示例，如图4所示，涉及从源节点（S）到目的地的两跳传输 节点 (D)。 数据包由 S 生成。它首先传输到辅助节点 (H)，然后由 H 中继到 D。在此示例中，S 和 D 之间没有直接连接。此外，地址解析协议 ( ARP) 信息由运行在 CU 的 SDN 控制器上的 ProxyARP 应用程序直接提供，该应用程序通过 CP 与网络中的所有节点直接通信。

数据包由主机 S (hS) 生成，它在 IP 报头和 MAC 报头中指定目标主机 D (hD) 的地址。 hS 不知道到达 hD 所需的任何路由信息。 它只是通过有线信道将数据包传输到节点 S (sS) 的交换机，该交换机负责无线传输它。 sS 正在运行 SDN 模块，如图 3 所示，该模块根据从 SDN 控制器接收到的指令修改 MAC 层的源地址和目标地址。 MAC头中的源和目的地分别变成sS和sH（节点H的交换机）。 然后，数据包可以被 sS 发送，也可以被 sH 接收。 由于数据包中的目的MAC地址为sH，因此数据包由sH进一步处理，并向sS发回确认（ACK）。

我们强调，对于大多数 Wi-Fi 适配器，如果 固件的MAC 和目的地不匹配，则固件中会丢弃数据包，因此修改 MAC 地址是多跳传送数据包的必要步骤 网络。

根据从 SDN 控制器接收到的 SDN 规则，由 sH 执行类似的过程。 同样在这种情况下，目的MAC地址被修改，数据包可以被转发到目的节点（sD）的交换机。

最后，sD 根据来自 SDN 控制器的另一条规则，将源 MAC 地址和目标 MAC 地址分别修改为 hS 和 hD。 这个过程，称为 MAC 恢复，是必要的，以便 hD 识别数据包是由 hS 发送的，而不需要来自 SDN 框架的任何进一步信息。

## SDN测试平台：性能比较

在本节中，我们展示了我们的 SDN 框架在商业设备上的实际实现，我们建立了网络并发送了一些具有多跳拓扑的数据流量。 为了展示我们框架的潜在优势，我们使用分布式路由协议运行相同的网络，并展示了 SDN 框架在拓扑突然变化的情况下的好处。

### A 网络设置

实验场景由具有三个 SDN 节点的 SDN MANET 组成，标记为 S、H 和 D，如上一节所示，部署如图 5 所示。每个节点由一个 RPi Model B+ 和一个 Wi -Fi 适配器（Ralink RT5370 USB），DP 中的传输使用 IEEE 802.11g ad-hoc 模式（在通道 6 上）。 三个 SDN 节点配备了 OVS-2.4.0，并连接到运行 ONOS（选择的 OpenFlow 控制器）的 CU 和我们的 MANET 应用程序 [12]。

第二个网络名为 OLSR MANET，它使用相同位置和相同拓扑的相同三个节点（S、H 和 D）进行比较。 实际上，在第二个网络中，节点没有配备我们的 SDN 框架，但它们正在运行分布式路由策略 OLSR。 特别是，由 HSMM-Pi 项目 [34] 提供的 olsrd-0.8.8 安装在三个节点中。 OLSR 的参数设置如下。 hello 消息间隔、hello 有效时间和拓扑控制消息间隔分别设置为 10、1 和 0.5 秒。

在这两个网络中，数据流量都是在节点 S 使用流量生成器 iPerf3 [35] 生成的，该流量生成器会创建一个随机 TCP 流，流向目的地节点 D。实验时间为 N 秒，时间间隔为 1秒。 对于每个间隔 τn，n = 1,...,N，端到端吞吐量以每秒比特 (bps) 为单位测量为：
$$
T\left(\tau_{n}\right)=\frac{\text { TCP } \text { RWND } \times 8}{\text { RTT }}
$$
其中 TCP RWND 是间隔 τi 期间 TCP 会话的平均接收窗口大小，RTT 是平均往返时间，即从发送的 TCP 段的第一个比特传输到接收到该 TCP 段的 TCP ACK的最后一个比特所经过的时间。 

为了比较 SDN MANET 和 OLSR MANET 在拓扑突然变化的情况下的行为，我们改变了图5-a的全连接拓扑为图5-b的多跳拓扑（S和D之间没有直接连接）。

由于通过改变节点的位置来完美控制链路故障并不容易，我们通过在节点的 MAC 层设计一个模块来模拟 S 和 D 之间的链路故障，该模块可以拒绝来自 S 的所有数据包（对于 节点 D），或来自 D（对于节点 S）。 这样，我们可以在我们的实验中完美地控制 S 和 D 之间的链路何时发生故障，或者何时重新启动。

在下文中，我们描述了在拓扑发生变化的情况下比较 SDN MANET 和 OLSR MANET 的三个实验（由于现有链路的故障，或以前不存在的链路的恢复）。 对于 SDN MANET 和 OLSR MANET，每个实验重复 M = 20 次。 结果中显示的平均吞吐量为：
$$
\bar{T}\left(\tau_{n}\right)=\frac{\sum_{m=1}^{M} T_{m}\left(\tau_{n}\right)}{M}
$$
其中 Tm(τn) 是第 m 次实验在时间间隔 τn 期间获得的吞吐量。

### B 链路断开实验

在图 6 的第一个实验中，我们观察了链路故障事件的影响，它改变了我们网络的拓扑结构。 实验从 t = 0 开始，拓扑如图 5-(a) 所示，其中三个节点（S、H 和 D）完全连接。 S 开始向 D 发送 TCP 流量，SDN MANET 和 OLSR MANET 都选择 S 和 D 之间的直接链路。

然后，在时间 t = 10，S 和 D 之间的直接链路发生故障，因此拓扑变为图 5-(b) 中的拓扑。 SDN控制器立即收到此事件的通知，并迅速做出反应，将新的SDN规则强加给节点S和H。这样，节点S将所有发往D的数据包发送给H，H将这些数据包转发给D。 SDN MANET 的吞吐量立即恢复到初始吞吐量的一半，因为从 S 到 D 的新路径现在有两跳。

OLSR MANET 能够识别链路故障并通过仅在 t = 25 时将路径更改为 D 来对其做出反应，延迟约 15 秒，从而导致严重的吞吐量中断。 这个结果是意料之中的，因为 OLSR 有一个完全分布式的路由算法，它需要大量时间来更新。 另一方面，SDN MANET 可以利用 CP，它允许路由算法以集中方式在 CU 上运行，所有关于链路状况的信息都被及时收集。

### C 连接实验

在第二个实验中，在图 7 中，我们观察了当初始拓扑是图 5-(b) 中的拓扑时，SDN MANET 和 OLSR MANET 经历的平均吞吐量，即 S 和 D 之间的两跳路径。 在 t = 10 时，S 和 D 之间的直接链接也被激活，如图 5-(a) 所示。 正如预期的那样，我们观察到，在 SDN MANET 的情况下，网络能够迅速对拓扑的变化做出反应，并且吞吐量在 t > 10 时几乎翻了一番。另一方面，OLSR MANET 的延迟约为 20 秒 在它能够充分利用直接链路，达到最大吞吐量之前。

### D 快速变化的拓扑实验

在第三个实验中，在图 8 中，我们有一系列连续的拓扑变化。 在 t = 0 时，拓扑是图 5-(a) 中的拓扑（S 和 D 之间的直接链路），然后在 t = 30 时，拓扑变为图 5-(b) 中的拓扑（两跳） ，然后在 t = 60 时再次切换到图 5-(a)，最后在 t = 90 时切换到图 5-(b)。

同样在这种情况下，实验重复 20 次，结果取所有试验的平均值。 对于每个拓扑变化，我们观察到 SDN MANET 如何能够几乎立即对拓扑变化做出反应，而 OLSR MANET 对变化的反应有一定的延迟，正如预期的那样，会导致显着的吞吐量损失。

## 结论

在这项工作中，我们提出了一个 SDN MANET 的实际实现，它提供了 D2D 数据传输的所有优点，同时具有集中网络管理的灵活性。 我们描述了 SDN 架构的细节，并概述并引用了我们采用的所有软件组件。 我们通过提供新组件为这项工作做出了贡献，这些组件可在 [12] 中的无线网络社区下载。

为了展示 SDN MANET 的优势以及所提供的所有软件的有效性，我们将我们的 SDN MANET 与以分布式方式管理的 ad hoc 网络进行了比较。 我们用几个简单的例子强调了我们方法的显着优势，特别是对于快速变化的网络拓扑。

在未来的工作中，我们计划处理大规模的 SDN MANET，解决可能出现的可扩展性问题。 特别是，我们计划与其他感兴趣的国家和国际研究机构合作，建立一个大型的分布式测试平台，我们可以利用它来解决几个研究问题，例如如何促进 D2D 通信在许可频段的共存 [36]。