# M5D1_Implementacion_Red – Informe Técnico

## 1. Objetivo del Proyecto

Implementar una red empresarial con segmentación mediante VLAN, enrutamiento inter-VLAN (Router-on-a-Stick), DHCP centralizado, enlaces troncales 802.1Q, seguridad en puertos y comunicación hacia un ISP externo.

---

## 2. Topología de Red

La red se compone de:

- **R1 y R2** → Routers de acceso que manejan las VLANs locales.
- **R3** → Router central y servidor DHCP.
- **ISP** → Router externo que provee salida hacia Internet.
- **SW-CENTRAL** → Switch de distribución.
- **SWA y SWB** → Switches de acceso para hosts finales.

![Topología](/docs/topologia_m5d1.png)

---

## 3. VLANs Configuradas

```bash
| VLAN | Nombre         | Red IPv4        | Gateway           | Asignada en |
|------|----------------|-----------------|-------------------|-------------|
| 10   | Administración | 192.168.10.0/24 | 192.168.10.1 (R1) | SWA         |
| 20   | Finanzas       | 192.168.20.0/24 | 192.168.20.1 (R1) | SWA         |
| 30   | Ventas         | 192.168.30.0/24 | 192.168.30.1 (R2) | SWB         |
| 40   | Soporte        | 192.168.40.0/24 | 192.168.40.1 (R2) | SWB         |
```

---

## 4. Direccionamiento IPv4 e IPv6

```bash
| Dispositivo   | Interfaz | IPv4            | Máscara | IPv6                      |
|-------------- |----------|-----------------|---------|---------------------------|
| R1            | Fa0/0.10 | 192.168.10.1    | /24     | 3001:ACAD:ACAD:100::1/112 |
| R1            | Fa0/0.20 | 192.168.20.1    | /24     | —                         |
| R1            | Fa0/1    | 192.168.64.1    | /27     | 3001:ACAD:ACAD:100::1/112 |
| R2            | Fa0/0.30 | 192.168.30.1    | /24     | 3001:ACAD:ACAD:100::2/112 |
| R2            | Fa0/0.40 | 192.168.40.1    | /24     | —                         |
| R2            | Fa0/1    | 192.168.64.2    | /27     | 3001:ACAD:ACAD:100::2/112 |
| R3            | Fa0/0.10 | 192.168.10.254  | /24     | 3001:ACAD:ACAD:100::3/112 |
| R3            | S0/0/0   | 200.0.0.1       | /30     | 3001:ACAD:ACAD:E::1/112   |
| R3            | S0/0/1   | 201.0.0.1       | /30     | 3001:ACAD:ACAD:D::1/112   |
| ISP           | S0/1/1   | 200.0.0.2       | /30     | 2018:ACAD:ACAD:E::2/112   |
| ISP           | S0/0/0   | 201.0.0.2       | /30     | 3001:ACAD:ACAD:E::2/112   |
```

---

## 5. DHCP Centralizado (R3)

```bash
ip dhcp excluded-address 192.168.10.1 192.168.10.5
ip dhcp excluded-address 192.168.20.1 192.168.20.5
ip dhcp excluded-address 192.168.30.1 192.168.30.5
ip dhcp excluded-address 192.168.40.1 192.168.40.5
```

CAda VLAN tiene su pool:

```bash
ip dhcp pool VLAN10
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 220.0.0.2
```

Routers R1 y R2 reenvían peticiones DHCP con:

```bash
ip helper-address 192.168.64.3
```

### 6. VLANs y Troncales

# SW-CENTRAL

- Puertos Fa0/1–Fa0/3 en modo trunk
- DTP deshabilitado con switchport nonegotiate
- VLANs 10,20,30,40 activas

**SWA**

```bash
| Puerto | VLAN            | Descripción         |
| ------ | --------------- | ------------------- |
| Fa0/1  | Trunk (1,10,20) | Enlace a SW-CENTRAL |
| Fa0/2  | 20              | Host Finanzas       |
| Fa0/3  | 10              | Host Administración |
```

**SWB**

```bash
| Puerto | VLAN            | Descripción         |
| ------ | --------------- | ------------------- |
| Fa0/1  | Trunk (1,30,40) | Enlace a SW-CENTRAL |
| Fa0/2  | 30              | Host Ventas         |
| Fa0/3  | 40              | Host Soporte        |

```

### 7. Seguridad de Puertos

Configuración en todos los puertos de acceso:

```bash
switchport port-security
switchport port-security maximum 2
switchport port-security violation restrict
```

### 8. Rutas Flotantes y OSPF (propuesta según requerimiento)

**R3 (Hacia ISP)**

```bash
ip route 0.0.0.0 0.0.0.0 200.0.0.2 1
ip route 0.0.0.0 0.0.0.0 201.0.0.2 10
```

**ISP (ruta sumarizada flotante)**

```bash
ip route 192.168.0.0 255.255.0.0 200.0.0.1 10
```

**OSPF (en R1, R2, R3)**

```bash
router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 192.168.30.0 0.0.0.255 area 0
 network 192.168.40.0 0.0.0.255 area 0
 network 192.168.64.0 0.0.0.31 area 0
 passive-interface FastEthernet0/0.10
 passive-interface FastEthernet0/0.20
 passive-interface FastEthernet0/0.30
 passive-interface FastEthernet0/0.40
```

### 9. ACLs Extendidas (según requerimientos)

```bash
ip access-list extended VLAN10_BLOCK_VLAN40
 deny ip 192.168.10.0 0.0.0.255 192.168.40.0 0.0.0.255
 permit ip any any

ip access-list extended VLAN30_BLOCK_TFTP
 deny udp 192.168.30.0 0.0.0.255 any eq 69
 permit icmp 192.168.30.0 0.0.0.255 any
 permit ip any any

ip access-list extended VLAN30_BLOCK_VLAN20
 deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
 permit ip any any

ip access-list extended VLAN10_NO_INTERNET
 deny ip 192.168.10.0 0.0.0.255 any
 permit ip any any

```

Aplicar según tráfico (in/out) en R3 o interfaces de VLAN correspondientes.

### 10. Pruebas

```bash
| Prueba                                 | Resultado esperado    |
| -------------------------------------- | --------------------  |
| Ping entre VLANs (10↔20, 30↔40)        | ✅ Permitido         |
| VLAN10 hacia VLAN40                    | ❌ Denegado          |
| VLAN30 hacia VLAN20                    | ❌ Denegado          |
| VLAN30 hacia TFTP                      | ❌ Denegado (UDP 69) |
| VLAN30 hacia Internet (ICMP permitido) | ✅                   |
| VLAN10 hacia Internet                  | ❌ Bloqueado         |

```

### 11. Conclusión

La topología cumple con la segmentación por VLANs, DHCP centralizado, troncales 802.1Q, seguridad en puertos y conectividad hacia un ISP externo.
Los requerimientos de ACLs y rutas flotantes fueron diseñados y validados teóricamente para complementar la configuración base.
