Questions: HW4
-------------------------------------------
TASK - 1
-------------------------------------------
1. What is the output of “nodes” and “net”
ANS.
mininet> nodes
available nodes are: 
c0 h1 h2 h3 h4 h5 h6 h7 h8 s1 s2 s3 s4 s5 s6 s7

mininet> net
h1 h1-eth0:s3-eth1
h2 h2-eth0:s3-eth2
h3 h3-eth0:s4-eth1
h4 h4-eth0:s4-eth2
h5 h5-eth0:s6-eth1
h6 h6-eth0:s6-eth2
h7 h7-eth0:s7-eth1
h8 h8-eth0:s7-eth2
s1 lo:  s1-eth1:s2-eth3 s1-eth2:s5-eth3
s2 lo:  s2-eth1:s3-eth3 s2-eth2:s4-eth3 s2-eth3:s1-eth1
s3 lo:  s3-eth1:h1-eth0 s3-eth2:h2-eth0 s3-eth3:s2-eth1
s4 lo:  s4-eth1:h3-eth0 s4-eth2:h4-eth0 s4-eth3:s2-eth2
s5 lo:  s5-eth1:s6-eth3 s5-eth2:s7-eth3 s5-eth3:s1-eth2
s6 lo:  s6-eth1:h5-eth0 s6-eth2:h6-eth0 s6-eth3:s5-eth1
s7 lo:  s7-eth1:h7-eth0 s7-eth2:h8-eth0 s7-eth3:s5-eth2
c0


2.What is the output of “h7 ifconfig”
ANS.
mininet> h7 ifconfig
h7-eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.0.7  netmask 255.0.0.0  broadcast 10.255.255.255
        inet6 fe80::880b:eeff:fe43:c491  prefixlen 64  scopeid 0x20<link>
        ether 8a:0b:ee:43:c4:91  txqueuelen 1000  (Ethernet)
        RX packets 67  bytes 5158 (5.1 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 10  bytes 796 (796.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

-------------------------------------------
TASK - 2
-------------------------------------------

1. Draw the function call graph of this controller. For example, once a packet comes to the
controller, which function is the first to be called, which one is the second, and so forth?
ANS.
We turn the POX on, which turns on the start switch. When a packet reaches the switch the function _handle_PacketIn() is called by the start switch to get the incoming packet.
next function calls are shown in the following function call grpah:
start_switch() : _handle_PacketIn() -> act_like_hub() -> resend_packet() -> send(msg)

2. Have h1 ping h2, and h1 ping h8 for 100 times (e.g., h1 ping -c100 p2).
a. How long does it take (on average) to ping for each case?
b. What is the minimum and maximum ping you have observed?
c. What is the difference, and why?

ANS.
h1 ping -c100 h2
--- 10.0.0.2 ping statistics ---
100 packets transmitted, 100 received, 0% packet loss, time 99148ms
rtt min/avg/max/mdev = 1.825/4.387/6.881/0.644 ms
-------------------
h1 ping -c100 h8
--- 10.0.0.8 ping statistics ---
100 packets transmitted, 100 received, 0% packet loss, time 99155ms
rtt min/avg/max/mdev = 5.600/15.391/22.127/3.494 ms

a.
avg - h1 - h2 : 4.387
avg - h1 - h8 : 15.391
b.
h1-h2: min : 1.825, max : 6.881
h1-h8: min : 5.600, max : 22.127
c. 
The ping times are longer in the second case i.e., from h1 to h8 because of the number of jumps are more in the second case than in the first case.
from h1-h2 : only s3
from h1-h8 : s3,s2,s1,s5,s7

3. Run “iperf h1 h2” and “iperf h1 h8”
a. What is “iperf” used for?
b. What is the throughput for each case?
c. What is the difference, and explain the reasons for the difference.

ANS.
a.
iperf is a tool used for measuring network bandwidth and throughput, to determine the network performance and line quality. Two hosts running iperf are limiting the network link.It is used to determine the amount of data transferred between any two nodes on a network line. 
b. second values are throughputs:
mininet> iperf h1 h2
*** Iperf: testing TCP bandwidth between h1 and h2 
*** Results: ['20.1 Mbits/sec', '23.1 Mbits/sec']
mininet> iperf h1 h8
*** Iperf: testing TCP bandwidth between h1 and h8 
*** Results: ['4.62 Mbits/sec', '5.37 Mbits/sec']
c. Difference is around 17 Mbits/sec, throughput is higher between h1 and h2 than between h1 and h8 (same as ping time being slower) because the number of jumps between h1 and h2 is lower, more data can be sent in less time.Because the number of jumps between h1 and h8 is higher, less data may be transferred in a given amount of time. 

4. Which of the switches observe traffic? Please describe your way for observing such traffic on switches (e.g., adding some functions in the “of_tutorial” controller).

ANS.
We can observe the traffic info by adding log.info("Switch observing traffic: "% s % (self.connection)" to the line number 107 "of tutorial"  log.debug("Installing flow..."), # Maybe the log statement should have source/destination/port?) controller. We know that all switches monitor traffic, particularly when they are overloaded with packets. The event listener function _handle PacketIn is invoked whenever a packet is received. 

---------------------------------------------
TASK - 3
---------------------------------------------

1. Describe how the above code works, such as how the "MAC to Port" map is established.
You could use a ‘ping’ example to describe the establishment process (e.g., h1 ping h2).
ANS.
The act_like_switch function in our code shows where MAC addresses are located.
The controller maps a MAC address to which a senders wants to send a message to. This also shows the controller's speed when delivering packets to already known addresses, as the packet is simply directed to that known port. The function just floods the packet to all destinations if the destination is unknown because flooding occurs less frequently.
Establishment process could be well understood by seeing the output for a ping of 1 packet from h1 to h2

root@62dc6144cc66:~/pox# ./pox.py log.level --DEBUG misc.of_tutorial
POX 0.7.0 (gar) / Copyright 2011-2020 James McCauley, et al.
DEBUG:core:POX 0.7.0 (gar) going up...
DEBUG:core:Running on CPython (3.6.9/Feb 28 2023 09:55:20)
DEBUG:core:Platform is Linux-5.19.0-32-generic-x86_64-with-Ubuntu-18.04-bionic
WARNING:version:Support for Python 3 is experimental.
INFO:core:POX 0.7.0 (gar) is up.
DEBUG:openflow.of_01:Listening on 0.0.0.0:6633
INFO:openflow.of_01:[00-00-00-00-00-07 2] connected
DEBUG:misc.of_tutorial:Controlling [00-00-00-00-00-07 2]
INFO:openflow.of_01:[00-00-00-00-00-06 3] connected
DEBUG:misc.of_tutorial:Controlling [00-00-00-00-00-06 3]
INFO:openflow.of_01:[00-00-00-00-00-01 4] connected
DEBUG:misc.of_tutorial:Controlling [00-00-00-00-00-01 4]
INFO:openflow.of_01:[00-00-00-00-00-04 5] connected
DEBUG:misc.of_tutorial:Controlling [00-00-00-00-00-04 5]
INFO:openflow.of_01:[00-00-00-00-00-03 6] connected
DEBUG:misc.of_tutorial:Controlling [00-00-00-00-00-03 6]
INFO:openflow.of_01:[00-00-00-00-00-02 7] connected
DEBUG:misc.of_tutorial:Controlling [00-00-00-00-00-02 7]
INFO:openflow.of_01:[00-00-00-00-00-05 8] connected
DEBUG:misc.of_tutorial:Controlling [00-00-00-00-00-05 8]
Learning that a6:f4:d4:19:f2:d0 is attached at port 2
33:33:ff:19:f2:d0 not known, resend to everybody
Learning that a6:f4:d4:19:f2:d0 is attached at port 2
33:33:ff:19:f2:d0 not known, resend to everybody
Learning that a6:f4:d4:19:f2:d0 is attached at port 3
33:33:ff:19:f2:d0 not known, resend to everybody
Learning that a6:f4:d4:19:f2:d0 is attached at port 2
33:33:ff:19:f2:d0 not known, resend to everybody
Learning that a6:f4:d4:19:f2:d0 is attached at port 3
33:33:ff:19:f2:d0 not known, resend to everybody
Learning that a6:f4:d4:19:f2:d0 is attached at port 3
33:33:ff:19:f2:d0 not known, resend to everybody
Learning that a6:f4:d4:19:f2:d0 is attached at port 3
33:33:ff:19:f2:d0 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
Learning that 4a:b4:29:7e:6d:c9 is attached at port 1
33:33:ff:7e:6d:c9 not known, resend to everybody
Learning that 4a:b4:29:7e:6d:c9 is attached at port 2
33:33:ff:7e:6d:c9 not known, resend to everybody
Learning that 4a:b4:29:7e:6d:c9 is attached at port 3
33:33:ff:7e:6d:c9 not known, resend to everybody
Learning that 4a:b4:29:7e:6d:c9 is attached at port 1
33:33:ff:7e:6d:c9 not known, resend to everybody
Learning that 4a:b4:29:7e:6d:c9 is attached at port 3
33:33:ff:7e:6d:c9 not known, resend to everybody
Learning that 4a:b4:29:7e:6d:c9 is attached at port 3
33:33:ff:7e:6d:c9 not known, resend to everybody
Learning that 4a:b4:29:7e:6d:c9 is attached at port 3
33:33:ff:7e:6d:c9 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
Learning that 8e:d5:aa:c7:6b:96 is attached at port 1
33:33:ff:c7:6b:96 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
Learning that 8e:d5:aa:c7:6b:96 is attached at port 1
33:33:ff:c7:6b:96 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
Learning that 8e:d5:aa:c7:6b:96 is attached at port 1
33:33:ff:c7:6b:96 not known, resend to everybody
Learning that 8e:d5:aa:c7:6b:96 is attached at port 3
33:33:ff:c7:6b:96 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
Learning that 8e:d5:aa:c7:6b:96 is attached at port 3
33:33:ff:c7:6b:96 not known, resend to everybody
Learning that 8e:d5:aa:c7:6b:96 is attached at port 3
33:33:ff:c7:6b:96 not known, resend to everybody
Learning that 8e:d5:aa:c7:6b:96 is attached at port 3
33:33:ff:c7:6b:96 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
Learning that 7a:44:e8:4c:c7:3a is attached at port 2
33:33:ff:4c:c7:3a not known, resend to everybody
Learning that 7a:44:e8:4c:c7:3a is attached at port 1
33:33:ff:4c:c7:3a not known, resend to everybody
Learning that 7a:44:e8:4c:c7:3a is attached at port 3
33:33:ff:4c:c7:3a not known, resend to everybody
Learning that 7a:44:e8:4c:c7:3a is attached at port 2
33:33:ff:4c:c7:3a not known, resend to everybody
Learning that 7a:44:e8:4c:c7:3a is attached at port 3
33:33:ff:4c:c7:3a not known, resend to everybody
Learning that 7a:44:e8:4c:c7:3a is attached at port 3
33:33:ff:4c:c7:3a not known, resend to everybody
Learning that 7a:44:e8:4c:c7:3a is attached at port 3
33:33:ff:4c:c7:3a not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
Learning that 12:1a:85:84:99:4f is attached at port 1
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
Learning that 12:1a:85:84:99:4f is attached at port 2
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
Learning that 12:1a:85:84:99:4f is attached at port 2
33:33:00:00:00:16 not known, resend to everybody
Learning that 12:1a:85:84:99:4f is attached at port 3
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
Learning that 12:1a:85:84:99:4f is attached at port 3
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
Learning that 12:1a:85:84:99:4f is attached at port 3
33:33:00:00:00:16 not known, resend to everybody
Learning that 12:1a:85:84:99:4f is attached at port 3
33:33:00:00:00:16 not known, resend to everybody
Learning that d6:20:75:ed:72:ef is attached at port 1
33:33:00:00:00:16 not known, resend to everybody
Learning that d6:20:75:ed:72:ef is attached at port 1
33:33:00:00:00:16 not known, resend to everybody
Learning that d6:20:75:ed:72:ef is attached at port 3
33:33:00:00:00:16 not known, resend to everybody
Learning that d6:20:75:ed:72:ef is attached at port 2
33:33:00:00:00:16 not known, resend to everybody
Learning that d6:20:75:ed:72:ef is attached at port 3
33:33:00:00:00:16 not known, resend to everybody
Learning that d6:20:75:ed:72:ef is attached at port 3
33:33:00:00:00:16 not known, resend to everybody
Learning that d6:20:75:ed:72:ef is attached at port 3
33:33:00:00:00:16 not known, resend to everybody
Learning that 36:7d:b6:df:b4:de is attached at port 2
33:33:ff:df:b4:de not known, resend to everybody
Learning that 36:7d:b6:df:b4:de is attached at port 2
33:33:ff:df:b4:de not known, resend to everybody
Learning that 36:7d:b6:df:b4:de is attached at port 1
33:33:ff:df:b4:de not known, resend to everybody
Learning that 36:7d:b6:df:b4:de is attached at port 3
33:33:ff:df:b4:de not known, resend to everybody
Learning that 36:7d:b6:df:b4:de is attached at port 3
33:33:ff:df:b4:de not known, resend to everybody
Learning that 36:7d:b6:df:b4:de is attached at port 3
33:33:ff:df:b4:de not known, resend to everybody
Learning that 36:7d:b6:df:b4:de is attached at port 3
33:33:ff:df:b4:de not known, resend to everybody
Learning that 56:84:e9:7b:ee:07 is attached at port 2
33:33:00:00:00:16 not known, resend to everybody
Learning that 56:84:e9:7b:ee:07 is attached at port 1
33:33:00:00:00:16 not known, resend to everybody
Learning that 56:84:e9:7b:ee:07 is attached at port 1
33:33:00:00:00:16 not known, resend to everybody
Learning that 56:84:e9:7b:ee:07 is attached at port 3
33:33:00:00:00:16 not known, resend to everybody
Learning that 56:84:e9:7b:ee:07 is attached at port 3
33:33:00:00:00:16 not known, resend to everybody
Learning that 56:84:e9:7b:ee:07 is attached at port 3
33:33:00:00:00:16 not known, resend to everybody
Learning that 56:84:e9:7b:ee:07 is attached at port 3
33:33:00:00:00:16 not known, resend to everybody
33:33:ff:84:99:4f not known, resend to everybody
33:33:ff:84:99:4f not known, resend to everybody
33:33:ff:84:99:4f not known, resend to everybody
33:33:ff:84:99:4f not known, resend to everybody
33:33:ff:84:99:4f not known, resend to everybody
33:33:ff:84:99:4f not known, resend to everybody
33:33:ff:84:99:4f not known, resend to everybody
33:33:ff:7b:ee:07 not known, resend to everybody
33:33:ff:7b:ee:07 not known, resend to everybody
33:33:ff:7b:ee:07 not known, resend to everybody
33:33:ff:7b:ee:07 not known, resend to everybody
33:33:ff:7b:ee:07 not known, resend to everybody
33:33:ff:7b:ee:07 not known, resend to everybody
33:33:ff:7b:ee:07 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:16 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
ff:ff:ff:ff:ff:ff not known, resend to everybody
ff:ff:ff:ff:ff:ff not known, resend to everybody
ff:ff:ff:ff:ff:ff not known, resend to everybody
ff:ff:ff:ff:ff:ff not known, resend to everybody
ff:ff:ff:ff:ff:ff not known, resend to everybody
ff:ff:ff:ff:ff:ff not known, resend to everybody
ff:ff:ff:ff:ff:ff not known, resend to everybody
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
DEBUG:openflow.of_01:1 connection aborted
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
a6:f4:d4:19:f2:d0 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
8e:d5:aa:c7:6b:96 destination known. only send message to it
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody
33:33:00:00:00:02 not known, resend to everybody

2. (Comment out all prints before doing this experiment) Have h1 ping h2, and h1 ping
h8 for 100 times (e.g., h1 ping -c100 p2).
a. How long did it take (on average) to ping for each case?
b. What is the minimum and maximum ping you have observed?
c. Any difference from Task 2 and why do you think there is a change if there is?
ANS.
mininet> h1 ping -c100 h2
--- 10.0.0.2 ping statistics ---
100 packets transmitted, 100 received, 0% packet loss, time 99137ms
rtt min/avg/max/mdev = 1.119/4.153/4.920/0.346 ms

mininet> h1 ping -c100 h8
--- 10.0.0.8 ping statistics ---
100 packets transmitted, 100 received, 0% packet loss, time 99159ms
rtt min/avg/max/mdev = 2.678/13.376/21.529/2.611 ms

a.
avg - h1-h2 - 4.153
avg - h1-h8 - 13.376
b.
h1-h2: min : 1.119, max : 4.920
h1-h8: min : 2.678, max : 21.529
c.
Task 3(in both cases) has a slight edge over the ping statistics than the task 2, it might be due to the lack of network congestion, which was found in task 2. As mapping is done, the switches can easily send packets to the corresponding ports.

3. Run “iperf h1 h2” and “iperf h1 h8”.
a. What is the throughput for each case?
b. What is the difference from Task 2 and why do you think there is a change if there is?
ANS.
a.
mininet> iperf h1 h2
*** Iperf: testing TCP bandwidth between h1 and h2 
*** Results: ['65.2 Mbits/sec', '65.8 Mbits/sec']
mininet> iperf h1 h8
*** Iperf: testing TCP bandwidth between h1 and h8 
*** Results: ['4.10 Mbits/sec', '4.66 Mbits/sec']
b.
In the first case, the throughput is almost thrice because routes are more pre-computed and learned with changes in controller. Also there'll be less network congestion as mac_to_port map has learned all the ports.
