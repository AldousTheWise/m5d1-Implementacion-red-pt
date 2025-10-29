## 🧩 **`docs/Comandos.md`**

# M5D1_Implementacion_Red – Comandos y Configuraciones

## R1

```bash
interface FastEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 ip helper-address 192.168.64.3
!
interface FastEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 ip helper-address 192.168.64.3
!
interface FastEthernet0/1
 ip address 192.168.64.1 255.255.255.224
 ipv6 address 3001:ACAD:ACAD:100::1/112
```

## R2

```bash
interface FastEthernet0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
 ip helper-address 192.168.64.3
!
interface FastEthernet0/0.40
 encapsulation dot1Q 40
 ip address 192.168.40.1 255.255.255.0
 ip helper-address 192.168.64.3
!
interface FastEthernet0/1
 ip address 192.168.64.2 255.255.255.224
 ipv6 address 3001:ACAD:ACAD:100::2/112

```

## R3

```bash
ip dhcp excluded-address 192.168.10.1 192.168.10.5
ip dhcp pool VLAN10
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 220.0.0.2
!
interface Serial0/0/0
 ip address 200.0.0.1 255.255.255.252
 ipv6 address 3001:ACAD:ACAD:E::1/112
!
interface Serial0/0/1
 ip address 201.0.0.1 255.255.255.252
 ipv6 address 3001:ACAD:ACAD:D::1/112
 clock rate 2000000
```

## ISP

```bash
interface Serial0/1/1
 ip address 200.0.0.2 255.255.255.252
 ipv6 address 2018:ACAD:ACAD:E::2/112
!
interface Serial0/0/0
 ip address 201.0.0.2 255.255.255.252
 ipv6 address 3001:ACAD:ACAD:E::2/112
!
ip route 192.168.0.0 255.255.0.0 200.0.0.1 10

```

## SW-CENTRAL

```bash
interface Fa0/1
 switchport mode trunk
 switchport nonegotiate
!
interface Fa0/2
 switchport mode trunk
 switchport nonegotiate
!
interface Fa0/3
 switchport mode trunk
 switchport nonegotiate
```

## SWA

```bash
interface Fa0/1
 switchport trunk allowed vlan 1,10,20
 switchport mode trunk
 switchport nonegotiate
!
interface Fa0/2
 switchport access vlan 20
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
!
interface Fa0/3
 switchport access vlan 10
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict

```

## SWB

```bash
interface Fa0/1
 switchport trunk allowed vlan 1,30,40
 switchport mode trunk
 switchport nonegotiate
!
interface Fa0/2
 switchport access vlan 30
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
!
interface Fa0/3
 switchport access vlan 40
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict

```
