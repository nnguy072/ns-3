/* -*- Mode:C++; c-file-style:"gnu"; indent-tabs-mode:nil; -*- */
/*
 * This program is free software; you can redistribute it and/or modify
 * it under the terms of the GNU General Public License version 2 as
 * published by the Free Software Foundation;
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU General Public License for more details.
 *
 * You should have received a copy of the GNU General Public License
 * along with this program; if not, write to the Free Software
 * Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307  USA
 */

#include "ns3/core-module.h"
#include "ns3/network-module.h"
#include "ns3/internet-module.h"
#include "ns3/point-to-point-module.h"
#include "ns3/applications-module.h"

using namespace ns3;

NS_LOG_COMPONENT_DEFINE ("FirstScriptExample");

int
main (int argc, char *argv[])
{
  Time::SetResolution (Time::NS);
  LogComponentEnable ("UdpEchoClientApplication", LOG_LEVEL_INFO);
  LogComponentEnable ("UdpEchoServerApplication", LOG_LEVEL_INFO);

  NS_LOG_INFO("Creating Topology");
  // create nodes
  NodeContainer nodes;
  nodes.Create (6);
  
  CommandLine cmd;
  // using your own variables (also changed echoClient.Set Attribute from (1) to (nPackets))
  uint32_t nPackets = 1;
  cmd.AddValue("nPackets", "Number of packets to echo", nPackets);
  cmd.Parse(argc, argv);  
  
  NS_LOG_INFO(nPackets); 
 
  NS_LOG_INFO("Creating Channel");
  // create channel helper to create the channel
  PointToPointHelper pointToPoint;
  //pointToPoint.SetDeviceAttribute ("DataRate", StringValue (myDataRate));
  //pointToPoint.SetChannelAttribute ("Delay", StringValue (myDelay));

  // create internetStack
  InternetStackHelper stack;
  stack.Install (nodes);
 
  NS_LOG_INFO("Creating echoServer");
  // Build UDP Echo Server
  UdpEchoServerHelper echoServer (9);

  ApplicationContainer serverApps = echoServer.Install (nodes.Get (5));
  serverApps.Start (Seconds (1.0));
  serverApps.Stop (Seconds (10.0));

  // set IP network address
  // we want each client to belong to a different subnet
  Ipv4AddressHelper address;
  address.SetBase ("10.1.1.0", "255.255.255.0");
  
  // install echo client  
  ApplicationContainer clientApps;
  
  // create NetDevice and bind them to Channel
  // .Install adds two devices
  for(int i = 0; i < 5; i++){
    NetDeviceContainer devices;
    devices.Add(pointToPoint.Install (nodes.Get(i),nodes.Get(5)));
  
    // assign IP to NetDevice
    NS_LOG_INFO("Assigning IP Addresses");
    Ipv4InterfaceContainer interfaces = address.Assign (devices);

    // Build UDP Echo Client attributes
    // each client needs to connect to a different server interface
    UdpEchoClientHelper echoClient (interfaces.GetAddress (1), 9);
    echoClient.SetAttribute ("MaxPackets", UintegerValue(nPackets));
    echoClient.SetAttribute ("Interval", TimeValue (Seconds (1.0)));
    echoClient.SetAttribute ("PacketSize", UintegerValue (1024));
    // UDP max packet size is 65,507
  
    // install echo client  
    //ApplicationContainer clientApps;
    clientApps.Add(echoClient.Install(nodes.Get (i)));
    address.NewNetwork();
  }

  clientApps.Start (Seconds (2.0));
  clientApps.Stop (Seconds (10.0));
  
  AsciiTraceHelper ascii;
  pointToPoint.EnableAsciiAll (ascii.CreateFileStream ("myfirst.tr"));
  pointToPoint.EnablePcapAll ("myfirst");
  Simulator::Run ();
  Simulator::Destroy ();
  return 0;
}
