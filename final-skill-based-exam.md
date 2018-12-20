# Final Skill-Based Exam

## Subnetting

* http://www.vlsm-calc.net/

## Switching

1. EtherChannel (BR-SW1, BR-SW2)

* Port-channel 1 mode ON

```
int range f0/1-2
    channel-group 1 mode on
```

2. Trunking (MLS ALS1, ALS2)

* Mode ON, disable DTP

```
(MLS) int range f0/2-3
    switchport trunk encapsulation dot1q
    switchport mode trunk
    switchport nonegotiate

(ALS1, ALS2) int range f0/1-2
    switchport mode trunk
    switchport nonegotiate
# Verify
show interface trunk
```

3. VTP

* Domain name "VINSYS", version 2, password "vinsys@123"

```
vtp domain VINSYS
vtp version 2
vtp password vinsys@123
```

4. VLAN

* Create a VLAN.
* Assign an IP Address to VLAN 99 for managerment.

```

```

5. Fine tuning

* Native VLAN on all trunking port is VLAN 99
* VTP mode transparent

```
(MLS) int f0/2-3
    switchport trunk native vlan 99
(ALS1, ALS2) int f0/1-2
    switchport trunk native vlan 99
(MLS, ALS1, ALS2)
    vtp mode transparent
```

6. STP

* MLS is root of all of VLANs with priority 0

```
(MLS, ALS1, ALS2) spanning-tree vlan 99,192,172 priority 0
# Verify
show spanning-tree ?
```

7. Inter-VLAN Routing

* Use SVIs on MLS

```
(MLS) ip routing
    int vlan 192
        no shutdown
        ip address 10.0.192.254 255.255.255.0
    int vlan 172
        no shutdown
        ip address 10.0.172.254 255.255.255.0

# Verify
show ip int brief
show ip route
```

## Routing

1. eBGP (IPS1, IPS2, IPS3)

* Adverties the directly connected networks and the static routes that assigning to customers

```
(ISP1) ip route 4.4.4.0 255.255.255.248 1.1.1.2
    router bgp 65001
        neighbor 3.3.12.2 remote-as 65002
        neighbor 3.3.13.2 remote-as 65003
        redistribute static
        network 1.1.1.0 mask 255.255.255.252
(ISP2) ip route 2.2.2.0 255.255.255.128 1.1.1.6
    router bgp 65002
        neighbor 3.3.12.1 remote-as 65001
        neighbor 3.3.23.2 remote-as 65003
        redistribute static
        network 1.1.1.4 mask 255.255.255.252
(ISP3)
    router bgp 65003
        neighbor 3.3.13.1 remote-as 65001
        neighbor 3.3.23.1 remote-as 65002
        network 8.8.8.0 mask 255.255.255.0

# Verify
show ip bgp
show ip bgp summary
show ip bgp neighbors
show ip route bgp
```

2. EIGRP AS 1 (BRANCH, GATE1, GATE2)

```
(BRANCH) ip route 0.0.0.0 0.0.0.0 1.1.1.5
    router eigrp 1
        network 2.0.0.0
        redistribute static
        no auto-summary
(GATE1)
    router eigrp 1
        network 2.0.0.0
        no auto-summary
        passive-interface g0/0

(GATE2)
    router eigrp 1
        network 2.0.0.0
        no auto-summary
        passive-interface g0/0
```

3. OSPF single area 0 (HQ, MLS)

```
(HQ) ip route 0.0.0.0 0.0.0.0 1.1.1.1
    router ospf 1
        router-id 1.1.1.1
        network 10.0.1.0 0.0.0.255 area 0
        network 10.0.0.0 0.0.0.255 area 0
        default-information originate
        passive-interface g0/1
    int vlan 1
        ip ospf priority 255
(MLS)
    router ospf 1
        router-id 2.2.2.2
        passive-interface vlan172
        passive-interface vlan192
        network 10.0.1.0 0.0.0.255 area 0
        network 10.0.192.0 0.0.0.255 area 0
        network 10.0.172.0 0.0.0.255 area 0
# Verify
show ip route ospf
```

## HA

1. HSRP (GATE1, GATE2)

```
(GATE1)
    int g0/0
        standby 60 ip 2.2.2.3
        standby 60 priority 255
        standby 60 preempt
        standby 60 track s0/0/0
(GATE2)
    int g0/0
        standby 60 ip 2.2.2.3
        standby 60 priority 254
        standby 60 preempt
        standby 60 track s0/0/0
# Verify
show standby
show standby brief
```

## Service

1. DHCP

```
(MLS)
    int vlan 192
        ip helper-address 10.0.0.251
    int vlan 172
        ip helper-address 10.0.0.251
    int vlan 192
        ip add 10.0.192.254/24
    int vlan 172
        ip add 10.0.172.254/24
```

2. NAT/PAT(HQ)

```
(HQ)
    access-list 1 permit 10.0.192.0 0.0.0.255
    access-list 1 permit 10.0.172.0 0.0.0.255
    ip nat inside source static 10.0.0.253 4.4.4.1
    ip nat inside source static 10.0.0.252 4.4.4.2
    ip nat inside source list 1 interface g0/0 overload
    int g0/0
        ip nat outside
    int g0/1
        ip nat inside
    int vlan 1
        ip nat inside
```

3. EMAIL (mail.vinsys.vn)

4. NTP


```
(HQ)
    ntp server 8.8.8.253
    show clock
```

5. AAA-SSH

```
(MLS)
    aaa new-model
    aaa authentication login SSH group tacacs+
    tacacs-server host 10.0.0.250 key aaa@123
    line vty 0 15
        login authentication SSH
        transport input ssh
    ip domain-name vinsys.vn
    crypto key generate rsa
    ip ssh version 2
    ip ssh authentication-retries 3
    ip ssh time-out 90
    enable secret mls@123
```
