## I.II SUBNETTING & SWITCHING

### SUBNETING

BRANCH
interface Serial0/0/0
ip address 2.2.2.97 255.255.255.252
interface Serial0/0/1
ip address 2.2.2.101 255.255.255.252

GATE1

interface Serial0/0/0 
ip address 2.2.2.98 255.255.255.252
interface GigabitEthernet0/0
ip address 2.2.2.1 255.255.255.192
interface GigabitEthernet0/1
ip address 2.2.2.65 255.255.255.224

GATE2

interface Serial0/0/0 
ip address 2.2.2.102 255.255.255.252
interface GigabitEthernet0/0
ip address 2.2.2.2 255.255.255.192
interface GigabitEthernet0/1
ip address 2.2.2.66 255.255.255.224

WEB SERVER
2.2.2.4
255.255.255.192

FTP SERVER
2.2.2.5
255.255.255.192

### SWITCHING
1. Ethernet channel
BRSW1
enable
conf t
int range f0/12
channelgroup 1 mode on
BRSW2
enable
conf t
int range f0/12
channelgroup 1 mode on

check
show etherchannel summary
show etherchannel portchannel

### TRUNKING
2. Trunking (MLS, ALS1, ALS2) : Mode ON, disable DTP

SW MLS
int range f0/23
switchport trunk encapsulation dot1q 
switchport mode trunk
switchport nonegotiate

SW ALS1
int range f0/12
switchport mode trunk
switchport nonegotiate

SW ALS2
int range f0/12
switchport mode trunk
switchport nonegotiate

Verify

show interface trunk

VTP

SW MLS
vtp domain VINSYS
vtp version 2
vtp password vinsys@123

SW ALS1
vtp password vinsys@123

SW ALS2
vtp password vinsys@123

show vtp status

### CREATEVLANASSIGNIPADDRESS

SW MLS
VLAN 192
  name SALE
VLAN 172
  name TECH
VLAN 99
  name MANAGEMENT

int vlan 99
no shutdown
ip address 10.0.99.254 255.255.255.0

SW ALS1

int fa0/3
switchport mode access
switchport access vlan 172

int fa0/4
switchport mode access
switchport access vlan 192

int vlan 99
no shutdown
ip address 10.0.99.1 255.255.255.0

SW ALS2

int fa0/3
switchport mode access
switchport access vlan 172

int fa0/4
switchport mode access
switchport access vlan 192

int vlan 99
no shutdown
ip address 10.0.99.2 255.255.255.0

### FINETUNNING

 MLS>
MLS>enable
MLS#conf t
MLS(config)#int range f0/23
MLS(configifrange)#switchport trunk native vlan 99
MLS(configifrange)#

show interfaces trunk 

 ALS 1,2
ALS1#conf t
ALS1(config)#int range f0/12
ALS1(configifrange)#switchport trunk native vlan 99
show interfaces trunk 


ALS2#conf t
ALS2(config)#int range f0/12
ALS2(configifrange)#switchport trunk native vlan 99
show interfaces trunk 


 MLS, ALS1, ALS2
conf t
vtp mode transparent

### STP

 MLS

conf t

spanningtree vlan 99,172,192 priority 0

show spanningtree

### Inter VLAN Routing

 MLS

conf t

ip routing#!IMPORTANT

int vlan 172
no shutdown
ip address 10.0.172.254 255.255.255.0

int vlan 192
no shutdown
ip address 10.0.192.254 255.255.255.0

int vlan 99
no shutdown
ip address 10.0.99.254 255.255.255.0

show ip interface brief
show ip route


## III. ROUTING

### eBGP
trong cau lenh cau hinh bgp: 

 neighbor: xac dinh cac hang xong cua router dang cau hinh.
 network: xac dinh cac network ma router dang cau hinh se quang ba cho cac router neighbor
 redistribute static: cho phep router dang cau hinh quang ba cac route static cho cac router neighbor (ben canh cac network da khai bao trong cau lenh network)

### OSPF

 Neu port noi voi router la port layer 3: Su dung passive interface g0/1
 Neu port noi voi router la port layer 2 (trong truong hop router cau hinh la multi layer switch): Su dung passive interface Vlan 172, 192

https://lpmazariegos.com/2016/01/21/ospfpassiveinterface/

 HQ

passive interface g0/1

 MLS

ip routing: bat chuc nang routing tren MLS switch
 passive cac interface VLAN: passive vao cac cong VLAN de ngan ban tin ospf gui qua cac vlan 172 va 192

passive interface Vlan 172
passive interface Vlan 192

### EIGRP
 no autosummary: Disable auto summary
 cau lenh: network 2.0.0.0 co 2 chuc nang:
   Xac dinh cac interface nao duoc active de trao doi goi tin (cac interface co IP address thuoc mang duoc khai bao tren network thi duoc phep truyen/nhan ban tin eigrp. Vi du router BRANCH co 2 interface thuoc mang 2.2.2.0/25 thi 2 interface nay se truyen nhan goi tin eigrp, con interface con lai noi voi router ISP2 se khong tham gia truyen nhan goi tin cua giao thuc eigrp.
   Xac dinh cac mang dau noi truc tiep duoc dong goi de truyen di.
  
 GATE2
enable
conf t
router eigrp 1
  network 2.0.0.0
  no autosummary
  passiveinterface g0/0

 GATE2
router eigrp 1
  network 2.0.0.0
  no autosummary
  passiveinterface g0/0

 BRANCH

ip route 0.0.0.0 0.0.0.0 1.1.1.5
enable
conf t 
router eigrp 1
  network 2.0.0.0
  redistributed static
  no autosummary
  
## IV. HA
  
   GATE1
  int g0/0
    standby 60 ip 2.2.2.3
    standby 60 priority 255
    standby 60 preempt
    standby 60 track s0/0/0
  
   GATE2
  int g0/0
    standby 60 ip 2.2.2.3
    standby 60 priority 254
    standby 60 preempt
    standby 60 track s0/0/0
    
 show stanby
 show standby brief
 
## V. SERVICE
 
### 1. DHCP
 
  DHCP la giao thuc application hoat dong o layer 7
 
#### 1.1 DHCP  Qua trinh cap phat IP
 
  PC gui goi tin DHCP discovery service voi IP dich: 255.255.255.255
   DHCP gui ban tin OFFER cho PC voi 1 IP
   PC gui ban tin chua IP ma DHCP gui cho toi cac may khac trong mang 
  thong qua ban tin co dst IP 255.255.255.255 de dam bao khong co may nao trong mang
  co IP nay.
   Neu khong co may nao trong mang so huu IP nay, PC su dung IP nay de lam IP cho chinh no
 
#### 1.2. DHCP: Qua trinh cap phat ip neu DHCP server nam khac mang voi PC can cap phat:
 
 ![Image of DHCP](https://lh6.googleusercontent.com/TdFv6hDlNzsjl92LALZlxmkZ3FVT1qURviyMt-M-o_ZlplO9AcvUE5oOaDNmdI74ArOLgleQzdZsOA=w710-h740-rw)

#### 1.3 Cau hinh DHCP

- Cau hinh o DHCP Server: [DHCP-SERVER](https://lh4.googleusercontent.com/yUjK3RIUI8PY_CBkd530csD6h6sfji5JxMmElTuNlxRrVI5CMg2J3TTRIn9CRLCJ95zMoGYvlIy9BA=w710-h740-rw)
  - Luu y: Bat active = on 
- Cau hinh o MLS:

enable 
conf t
int vlan 192
  ip helper-address 10.0.0.251
int vlan 172
  ip helper-address 10.0.0.251

Luu y: Cau hinh ip cho int vlan 192 va vlan 172: 10.0.192.254/24 va 10.0.172.254/24

### 2. Cau hinh NAT/PAT






