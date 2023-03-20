# NS-3 程序注释

## 1.first.cc

```c++
#include "ns3/core-module.h"
#include "ns3/network-module.h"
#include "ns3/internet-module.h"
#include "ns3/point-to-point-module.h"
#include "ns3/applications-module.h"
// 导入宏文件

using namespace ns3;
// 定义命名空间

NS_LOG_COMPONENT_DEFINE ("FirstScriptExample");
// 文档定义

int main (int argc, char *argv[])
{
  CommandLine cmd (__FILE__);
  cmd.Parse (argc, argv); // 命令行指令解析
  
  Time::SetResolution (Time::NS);
  // 定义时间刻度 默认1ns
  LogComponentEnable ("UdpEchoClientApplication", LOG_LEVEL_INFO);
  LogComponentEnable ("UdpEchoServerApplication", LOG_LEVEL_INFO);
  // 启用内置于Echo客户端和Echo服务器应用程序中的两个日志记录组件
  
  /**
  	在日志记录组件中，每个组件上面启用多个级别的日志记录，以使日志记录尽可能详细
  */

  /**
  	接下来进入核心代码，构建拓扑网络
  */
  NodeContainer nodes;
  nodes.Create (2);
  // 创建ns3 Node对象，每一个Node对象将代表模拟中的计算机/终端  
  
  // 接下来创建点对点链路
  PointToPointHelper pointToPoint;
  // 实例化堆栈中的 PointToPointHelper 对象
  pointToPoint.SetDeviceAttribute ("DataRate", StringValue ("5Mbps"));
  // 使用5Mbps作为DataRate
  pointToPoint.SetChannelAttribute ("Delay", StringValue ("2ms"));
  // 延时2ns
  
    /* 得到了一个 PointToPointHelper 对象后，它准备好制作PointToPointNetDevices并在它们之间链接PointToPointChannel对象 */
    
  NetDeviceContainer devices;
  devices = pointToPoint.Install (nodes);
  // 第一行声明设备容器，第二行执行PointToPointHelper的Install方法将NodeContainer作为参数。在内部创建一个NetDeviceContainer
  
  /* 现在我们有了两个节点，每个节点都有一个已安装的点位点网络设备和他们之间的单个点对点通道，两个网络设备的配置为：1、具有2ns传输延迟；2、通道以每秒5兆比特的速度传输数据 */

  // 接下来在节点上安装协议栈
  InternetStackHelper stack;
  stack.Install (nodes);
  // InternetStackHelper是一个拓扑助手，该助手的安装方法将 NodeContainer 作为参数，当它被执行时，在每个节点上安装一个互联网堆栈（TCP/UDP/IP）
  
  // 将节点设备与IP地址相关联
  Ipv4AddressHelper address;
  address.SetBase ("10.1.1.0", "255.255.255.0");
  // 从网络10.1.1.0 开始分配IP地址，单调增
  // 如果同一地址被分配两次将产生很难调试的错误
  
  Ipv4InterfaceContainer interfaces = address.Assign (devices);
  // 该对象用于在IP地址和设备之间建立联系
  
  /* 应用程序是ns-3系统的另一个核心应用
  	 1、UdpEchoServerApplication
     2、UdpEchoClientApplication
  */
  
    
  // 在创建的两个节点之一上面创建回显服务器应用程序
  UdpEchoServerHelper echoServer (9);
  // 声明 UdpEchoServerHelper 帮助我们创建实际应用程序的对象
  // 将所需的属性放在 helper 程序的构造函数中
  ApplicationContainer serverApps = echoServer.Install (nodes.Get (1));
  // nodes.Get(1) 返回指向节点对象的智能指针ptr，并在未命名的NodeContainer的构造函数中使用该指针
  serverApps.Start (Seconds (1.0));
  serverApps.Stop (Seconds (10.0));
  // 将让应用程序在模拟开始一秒时启动，并在模拟开始10秒时停止

  // echo 客户端应用程序的使用方法和服务器的方法基本类似
  UdpEchoClientHelper echoClient (interfaces.GetAddress (1), 9);
  // 将客户端的远程ip地址设置为分配给服务器所在节点的ip地址，安排将数据包发送给端口9
  echoClient.SetAttribute ("MaxPackets", UintegerValue (1));
  echoClient.SetAttribute ("Interval", TimeValue (Seconds (1.0)));
  echoClient.SetAttribute ("PacketSize", UintegerValue (1024));
  
  ApplicationContainer clientApps = echoClient.Install (nodes.Get (0));
  clientApps.Start (Seconds (2.0));
  clientApps.Stop (Seconds (10.0));

  /* 运行全局函数完成模拟 */
  Simulator::Run ();
  Simulator::Destroy ();
  return 0;
}

```

## 2.second.cc

```c++

#include "ns3/core-module.h"
#include "ns3/network-module.h"
#include "ns3/csma-module.h"
#include "ns3/internet-module.h"
#include "ns3/point-to-point-module.h"
#include "ns3/applications-module.h"
#include "ns3/ipv4-global-routing-helper.h"

// Default Network Topology
//
//       10.1.1.0
// n0 -------------- n1   n2   n3   n4
//    point-to-point  |    |    |    |
//                    ================
//                      LAN 10.1.2.0


using namespace ns3;

NS_LOG_COMPONENT_DEFINE ("SecondScriptExample");

int 
main (int argc, char *argv[])
{
  bool verbose = true;
  // verbose 是否启用日志记录组件
  uint32_t nCsma = 3;

  CommandLine cmd (__FILE__);
  cmd.AddValue ("nCsma", "Number of \"extra\" CSMA nodes/devices", nCsma);
  cmd.AddValue ("verbose", "Tell echo applications to log if true", verbose);

  cmd.Parse (argc,argv);

  if (verbose)
    {
      LogComponentEnable ("UdpEchoClientApplication", LOG_LEVEL_INFO);
      LogComponentEnable ("UdpEchoServerApplication", LOG_LEVEL_INFO);
    }

  nCsma = nCsma == 0 ? 1 : nCsma;
  // 确保有一个额外的节点

  NodeContainer p2pNodes;
  p2pNodes.Create (2);
  // 创建两个点对点节点容器

  NodeContainer csmaNodes;
  csmaNodes.Add (p2pNodes.Get (1));// *注意，这边从csmaNodes容器中取出了一个
  csmaNodes.Create (nCsma);
  // 创建nCsma个额外节点加入这个Csma网络中

  PointToPointHelper pointToPoint;
  pointToPoint.SetDeviceAttribute ("DataRate", StringValue ("5Mbps"));
  pointToPoint.SetChannelAttribute ("Delay", StringValue ("2ms"));
  // 实例化一个PointToPointHelper并设置关联的默认属性，创建每秒5Mbps的发射器和两秒的传输延迟

  NetDeviceContainer p2pDevices;
  p2pDevices = pointToPoint.Install (p2pNodes);

  CsmaHelper csma;
  csma.SetChannelAttribute ("DataRate", StringValue ("100Mbps"));
  csma.SetChannelAttribute ("Delay", TimeValue (NanoSeconds (6560)));
  // 数据传输速率设置为每秒100Mbps，然后将通道的光速延迟为6560纳秒

  NetDeviceContainer csmaDevices;
  csmaDevices = csma.Install (csmaNodes);
  // 创建一个NetDeviceContainer保存由CsmaHelper创建的设备，并调用CsmaHelper的Install方法将设备安装到csmaNodes NodeContainer

  InternetStackHelper stack;
  stack.Install (p2pNodes.Get (0));
  stack.Install (csmaNodes);
  // 这里只需要在剩余的p2pNodes节点上安装堆栈，并在csmaNodes容器中安装所有节点，以覆盖模拟中的节点

  Ipv4AddressHelper address;
  address.SetBase ("10.1.1.0", "255.255.255.0");
  // 使用Ipv4AddressHelper为设备接口分配IP地址 首先用网络10.1.1.0创建两个点对点设备所需的两个地址
  Ipv4InterfaceContainer p2pInterfaces;
  p2pInterfaces = address.Assign (p2pDevices);
  // 将创建的接口保存在容器中用于设置应用程序

  address.SetBase ("10.1.2.0", "255.255.255.0");
  Ipv4InterfaceContainer csmaInterfaces;
  csmaInterfaces = address.Assign (csmaDevices);

  UdpEchoServerHelper echoServer (9);

  ApplicationContainer serverApps = echoServer.Install (csmaNodes.Get (nCsma));
  serverApps.Start (Seconds (1.0));
  serverApps.Stop (Seconds (10.0));

  UdpEchoClientHelper echoClient (csmaInterfaces.GetAddress (nCsma), 9);
  echoClient.SetAttribute ("MaxPackets", UintegerValue (1));
  echoClient.SetAttribute ("Interval", TimeValue (Seconds (1.0)));
  echoClient.SetAttribute ("PacketSize", UintegerValue (1024));

  ApplicationContainer clientApps = echoClient.Install (p2pNodes.Get (0));
  clientApps.Start (Seconds (2.0));
  clientApps.Stop (Seconds (10.0));

  Ipv4GlobalRoutingHelper::PopulateRoutingTables ();

  pointToPoint.EnablePcapAll ("second");
  csma.EnablePcap ("second", csmaDevices.Get (1), true);

  Simulator::Run ();
  Simulator::Destroy ();
  return 0;
}
```

## 3.third.cc

```C++
#include "ns3/core-module.h"
#include "ns3/point-to-point-module.h"
#include "ns3/network-module.h"
#include "ns3/applications-module.h"
#include "ns3/mobility-module.h"
#include "ns3/csma-module.h"
#include "ns3/internet-module.h"
#include "ns3/yans-wifi-helper.h"
#include "ns3/ssid.h"

// Default Network Topology
//
//   Wifi 10.1.3.0
//                 AP
//  *    *    *    *
//  |    |    |    |    10.1.1.0
// n5   n6   n7   n0 -------------- n1   n2   n3   n4
//                   point-to-point  |    |    |    |
//                                   ================
//                                     LAN 10.1.2.0

using namespace ns3;

NS_LOG_COMPONENT_DEFINE ("ThirdScriptExample");

int 
main (int argc, char *argv[])
{
  bool verbose = true;
  uint32_t nCsma = 3;
  uint32_t nWifi = 3;
  bool tracing = false;

  CommandLine cmd (__FILE__);
  cmd.AddValue ("nCsma", "Number of \"extra\" CSMA nodes/devices", nCsma);
  cmd.AddValue ("nWifi", "Number of wifi STA devices", nWifi);
  cmd.AddValue ("verbose", "Tell echo applications to log if true", verbose);
  cmd.AddValue ("tracing", "Enable pcap tracing", tracing);

  cmd.Parse (argc,argv);

  // The underlying restriction of 18 is due to the grid position
  // allocator's configuration; the grid layout will exceed the
  // bounding box if more than 18 nodes are provided.
  if (nWifi > 18)
    {
      std::cout << "nWifi should be 18 or less; otherwise grid layout exceeds the bounding box" << std::endl;
      return 1;
    }

  if (verbose)
    {
      LogComponentEnable ("UdpEchoClientApplication", LOG_LEVEL_INFO);
      LogComponentEnable ("UdpEchoServerApplication", LOG_LEVEL_INFO);
    }

  NodeContainer p2pNodes;
  p2pNodes.Create (2);
// 创建两个点对点链路并链接他们
    
  PointToPointHelper pointToPoint;
  pointToPoint.SetDeviceAttribute ("DataRate", StringValue ("5Mbps"));
  pointToPoint.SetChannelAttribute ("Delay", StringValue ("2ms"));
// 实例化一个 PointToPointHelper 并设置关联的默认属性，以便我们在使用助手创建的设备上创建每秒 5 兆位的发射器，并在助手创建的通道上创建两毫秒的延迟。然后我们在节点上安装设备以及它们之间的通道。
    
// 接下来，我们声明另一个 NodeContainer 来保存将成为总线 (CSMA) 网络一部分的节点。
  NetDeviceContainer p2pDevices;
  p2pDevices = pointToPoint.Install (p2pNodes);
    
// 从点对点节点容器中获取第一个节点（索引为 1），并将其添加到将获取 CSMA 设备的节点容器中。所讨论的节点将以点对点设备和 CSMA 设备结束。然后我们创建一些“额外”节点，它们构成了 CSMA 网络的其余部分
  NodeContainer csmaNodes;
  csmaNodes.Add (p2pNodes.Get (1));
  csmaNodes.Create (nCsma);

// 然后我们实例化一个 CsmaHelper 并设置它的属性，就像我们在前面的例子中所做的那样。我们创建一个 NetDeviceContainer 来跟踪创建的 CSMA 网络设备，然后我们在选定的节点上安装 CSMA 设备。
  CsmaHelper csma;
  csma.SetChannelAttribute ("DataRate", StringValue ("100Mbps"));
  csma.SetChannelAttribute ("Delay", TimeValue (NanoSeconds (6560)));

  NetDeviceContainer csmaDevices;
  csmaDevices = csma.Install (csmaNodes);
    
// 创建将成为 Wi-Fi 网络一部分的节点。我们将创建一些由命令行参数指定的“站”节点，我们将使用点对点链路的“最左边”节点作为接入点的节点
  NodeContainer wifiStaNodes;
  wifiStaNodes.Create (nWifi);
  NodeContainer wifiApNode = p2pNodes.Get (0);

// 下一段代码构建了 wifi 设备和这些 wifi 节点之间的互连通道。首先，我们配置 PHY 和通道助手：
  YansWifiChannelHelper channel = YansWifiChannelHelper::Default ();
  YansWifiPhyHelper phy;
    // 使用默认的 PHY 层配置和通道模型
  phy.SetChannel (channel.Create ());
    
// 一旦配置了 PHY 助手，我们就可以专注于 MAC 层。 WifiMacHelper 对象用于设置 MAC 参数。下面的第二条语句创建一个 802.11 服务集标识符 (SSID) 对象，该对象将用于设置 MAC 层实现的“Ssid”属性的值
  WifiMacHelper mac;
  Ssid ssid = Ssid ("ns-3-ssid");
    
// 默认情况下，WifiHelper 将使用的标准配置为 802.11ax
  WifiHelper wifi;

    /**
   我们现在准备在节点上安装 Wi-Fi 模型，使用这四个帮助对象（YansWifiChannelHelper、YansWifiPhyHelper、WifiMacHelper、WifiHelper）和上面创建的 Ssid 对象。这些助手封装了很多默认配置，如果需要，可以使用额外的属性配置进一步定制。我们还将创建 NetDevice 容器来存储指向助手创建的 WifiNetDevice 对象的指针。
    */
    
    // 设置各种mac属性
  NetDeviceContainer staDevices;
  mac.SetType ("ns3::StaWifiMac",
               "Ssid", SsidValue (ssid),
               "ActiveProbing", BooleanValue (false));
  staDevices = wifi.Install (phy, mac, wifiStaNodes);

// 创建单个 AP，它与站点共享同一组 PHY 级属性（和信道）
  NetDeviceContainer apDevices;
  mac.SetType ("ns3::ApWifiMac",
               "Ssid", SsidValue (ssid));
  apDevices = wifi.Install (phy, mac, wifiApNode);

    /**我们要添加移动模型。我们希望 STA 节点是可移动的，在边界框内四处游荡，我们希望让 AP 节点静止不动。我们使用 MobilityHelper 来简化这一过程。首先，我们实例化一个 MobilityHelper 对象并设置一些控制“位置分配器”功能的属性
    */
  MobilityHelper mobility;

  mobility.SetPositionAllocator ("ns3::GridPositionAllocator",
                                 "MinX", DoubleValue (0.0),
                                 "MinY", DoubleValue (0.0),
                                 "DeltaX", DoubleValue (5.0),
                                 "DeltaY", DoubleValue (10.0),
                                 "GridWidth", UintegerValue (3),
                                 "LayoutType", StringValue ("RowFirst"));
    
// 沿随机方向移动
  mobility.SetMobilityModel ("ns3::RandomWalk2dMobilityModel",
                             "Bounds", RectangleValue (Rectangle (-50, 50, -50, 50)));
  mobility.Install (wifiStaNodes);

  mobility.SetMobilityModel ("ns3::ConstantPositionMobilityModel");
  mobility.Install (wifiApNode);

    // 安装协议栈
  InternetStackHelper stack;
  stack.Install (csmaNodes);
  stack.Install (wifiApNode);
  stack.Install (wifiStaNodes);

  Ipv4AddressHelper address;

    // 使用网络 10.1.1.0 创建两个点对点设备所需的两个地址
  address.SetBase ("10.1.1.0", "255.255.255.0");
  Ipv4InterfaceContainer p2pInterfaces;
  p2pInterfaces = address.Assign (p2pDevices);

    // 使用网络 10.1.2.0 为 CSMA 网络分配地址
  address.SetBase ("10.1.2.0", "255.255.255.0");
  Ipv4InterfaceContainer csmaInterfaces;
  csmaInterfaces = address.Assign (csmaDevices);

    // 从网络 10.1.3.0 为无线网络上的 STA 设备和 AP 分配地址
  address.SetBase ("10.1.3.0", "255.255.255.0");
  address.Assign (staDevices);
  address.Assign (apDevices);

  UdpEchoServerHelper echoServer (9);

  ApplicationContainer serverApps = echoServer.Install (csmaNodes.Get (nCsma));
  serverApps.Start (Seconds (1.0));
  serverApps.Stop (Seconds (10.0));

  UdpEchoClientHelper echoClient (csmaInterfaces.GetAddress (nCsma), 9);
  echoClient.SetAttribute ("MaxPackets", UintegerValue (1));
  echoClient.SetAttribute ("Interval", TimeValue (Seconds (1.0)));
  echoClient.SetAttribute ("PacketSize", UintegerValue (1024));

  ApplicationContainer clientApps = 
    echoClient.Install (wifiStaNodes.Get (nWifi - 1));
  clientApps.Start (Seconds (2.0));
  clientApps.Stop (Seconds (10.0));

  Ipv4GlobalRoutingHelper::PopulateRoutingTables ();

  Simulator::Stop (Seconds (10.0));

  if (tracing)
    {
      phy.SetPcapDataLinkType (WifiPhyHelper::DLT_IEEE802_11_RADIO);
      pointToPoint.EnablePcapAll ("third");
      phy.EnablePcap ("third", apDevices.Get (0));
      csma.EnablePcap ("third", csmaDevices.Get (0), true);
    }

  Simulator::Run ();
  Simulator::Destroy ();
  return 0;
}
```



XAUTHORITY=/root/.Xauthority sudo。。。，从root启动firefox

不支持root运行firefox参考：https://askubuntu.com/questions/1037052/running-firefox-as-root-in-a-regular-users-session-is-not-supported-xauthori

由于linux中这个osm webwizard打不开，我们考虑在windows上生成sumocfg文件然后导入linux中
