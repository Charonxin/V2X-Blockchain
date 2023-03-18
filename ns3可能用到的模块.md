## ns3搭建车联网环境可能用到的模块

1. Mobility模块：NS-3的Mobility模块提供了各种移动模型，可以用于模拟车辆和节点的移动。可以选择不同的移动模型来模拟车辆之间的运动和交互，例如常用的RandomWalk2d、RandomWaypoint、Gauss-Markov和ConstantPosition等。

   ```C++
   // 创建节点
   NodeContainer nodes;
   nodes.Create(10);
   
   // 设置移动模型
   MobilityHelper mobility;
   mobility.SetMobilityModel("ns3::RandomWalk2dMobilityModel",
                             "Bounds", RectangleValue(Rectangle(-100, 100, -100, 100)));
   
   // 给节点添加移动模型
   mobility.Install(nodes);
   
   // 启动仿真
   Simulator::Run();
   Simulator::Destroy();
   
   // 在这个示例中，我们创建了10个节点，并将它们放在一个边长为200的正方形区域内。然后，我们使用RandomWalk2d移动模型来模拟节点的移动。
   ```

   

2. Channel模块：NS-3的Channel模块用于模拟无线信道。Channel模块提供了多种传输介质和信道模型，例如OmniAntenna、LogDistancePropagationLoss和TwoRayGroundPropagationLoss等。

   ```C++
   // 创建节点
   NodeContainer nodes;
   nodes.Create(2);
   
   // 创建网络设备
   WifiHelper wifi;
   wifi.SetStandard(WIFI_PHY_STANDARD_80211b);
   wifi.SetRemoteStationManager("ns3::ConstantRateWifiManager",
                                "DataMode", StringValue("DsssRate1Mbps"),
                                "ControlMode", StringValue("DsssRate1Mbps"));
                                
   // 创建无线信道
   YansWifiChannelHelper channel = YansWifiChannelHelper::Default();
   channel.SetPropagationDelay("ns3::ConstantSpeedPropagationDelayModel");
   channel.AddPropagationLoss("ns3::TwoRayGroundPropagationLossModel");
   
   // 安装无线设备和信道
   NetDeviceContainer devices = wifi.Install(nodes);
   YansWifiPhyHelper wifiPhy = YansWifiPhyHelper::Default();
   wifiPhy.SetChannel(channel.Create());
   wifiPhy.Set("TxPowerStart", DoubleValue(14.0));
   wifiPhy.Set("TxPowerEnd", DoubleValue(14.0));
   wifiPhy.Set("TxGain", DoubleValue(0));
   wifiPhy.Set("RxGain", DoubleValue(0));
   wifiPhy.Set("EnergyDetectionThreshold", DoubleValue(-98));
   wifiPhy.SetErrorRateModel("ns3::YansErrorRateModel");
   
   // 启动仿真
   Simulator::Run();
   Simulator::Destroy();
   
   // 在这个示例中，我们创建了两个节点，并为它们安装了802.11b标准的WiFi设备。然后，我们创建了一个TwoRayGroundPropagationLoss信道模型，并将其设置为节点之间的通信信道。最后，我们启动了仿真。
   ```

   

3. Internet模块：NS-3的Internet模块用于模拟IP网络和TCP / UDP传输。可以使用Internet模块来模拟车辆之间的通信和数据传输。

   ```C++
   // 创建节点
   NodeContainer nodes;
   nodes.Create(2);
   
   // 安装Internet协议栈
   InternetStackHelper stack;
   stack.Install(nodes);
   
   // 创建设备
   PointToPointHelper p2p;
   p2p.SetDevice
   ```

   

4. LTE模块：NS-3的LTE模块用于模拟LTE和LTE-Advanced无线网络。

5. Application模块：NS-3的Application模块用于模拟应用程序和协议。例如基于HTTP和FTP的数据传输、车辆定位和路由协议等。

## 数据导出可能用到的模块

1. `AsciiTraceHelper`: 该API可以帮助生成ASCII格式的跟踪文件，其中包含有关网络中各种事件的详细信息。这些事件可以包括数据包的发送和接收，节点之间的通信等。可以使用`AsciiTraceHelper`在仿真期间将跟踪信息输出到文件中。
2. `PcapHelper`: 该API可以帮助生成PCAP格式的跟踪文件，其中包含有关网络中数据包的详细信息，例如包的大小，时间戳和源和目标地址等。与ASCII跟踪文件类似，可以使用`PcapHelper`在仿真期间将跟踪信息输出到文件中。
3. `FlowMonitor`: 该API可以帮助监视仿真期间的数据流，并收集关于流量的统计信息，例如流的大小，持续时间和吞吐量等。可以使用`FlowMonitor`在仿真结束时输出收集到的统计信息。

## 数据直接写入数据库可能用到的模块

1. `ns3::SqliteOutput`: 这是一个NS-3模块，它允许将数据写入SQLite数据库。可以使用`ns3::SqliteOutput`在仿真期间收集数据，并在仿真结束时将其写入数据库。该模块提供了一个简单的接口，可以使用它来指定要保存的数据字段以及要保存数据的表格。
2. `ns3::PostgresqlOutput`: 这是另一个NS-3模块，它允许将数据写入PostgreSQL数据库。它与`ns3::SqliteOutput`类似，但是使用不同的数据库引擎。

```C++
#include "ns3/core-module.h"
#include "ns3/network-module.h"
#include "ns3/internet-module.h"
#include "ns3/applications-module.h"
#include "ns3/mobility-module.h"
#include "ns3/config-store.h"
#include "ns3/sqlite-output.h"

using namespace ns3;

NS_LOG_COMPONENT_DEFINE ("VehicleExample");

int main (int argc, char *argv[])
{
  // 设置仿真时长
  Time simTime = Seconds (10.0);

  // 创建一个仿真环境
  NodeContainer nodes;
  nodes.Create (2);

  // 安装移动模型
  MobilityHelper mobility;
  mobility.SetMobilityModel ("ns3::ConstantPositionMobilityModel");
  mobility.Install (nodes);

  // 设置仿真结束时间并创建SQLite输出对象
  Simulator::Stop (simTime);
  SqliteOutput sqliteOutput ("vehicle-trace.sqlite", Seconds (0.5));

  // 添加要收集的数据
  sqliteOutput.AddField ("Time", "DOUBLE");
  sqliteOutput.AddField ("VehicleId", "INTEGER");
  sqliteOutput.AddField ("X", "DOUBLE");
  sqliteOutput.AddField ("Y", "DOUBLE");

  // 每隔0.5秒将车辆位置信息写入SQLite数据库
  sqliteOutput.ConnectWithoutContext ("pre-update", MakeCallback (&sqliteOutput.PreUpdate));
  sqliteOutput.ConnectWithoutContext ("post-update", MakeCallback (&sqliteOutput.PostUpdate));
  Config::Connect ("/NodeList/*/Mobility/Position", MakeCallback (&SqliteOutput::UpdatePosition, &sqliteOutput));

  // 运行仿真
  Simulator::Run ();

  // 写入剩余的数据并关闭数据库连接
  sqliteOutput.Write ();
  sqliteOutput.Close ();

  return 0;
}
```

