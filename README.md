# VPN Site-to-Site IPSec IKEv1 — Route-Based
**Autor:** Yeury Lopez | **Matrícula:** 20250780 | **Repositorio:** YeuryLopez_20250780_IPSecIKEv1-Route-Based

---

## 1. Objetivo
Configurar una VPN Site-to-Site basada en enrutamiento utilizando IPSec IKEv1. Se crea una interfaz virtual Tunnel0 y se enruta el tráfico hacia la LAN remota a través de esa interfaz, sin necesidad de ACLs.

---

## 2. Topología

```
[Linux1] --- [SW1] --- [R1] --- [ISP] --- [R2] --- [SW2] --- [Linux2]
```

> 📸 **SCREENSHOT:** Insertar captura de la topología completa en EVE-NG

---

## 3. Direccionamiento IP

| Dispositivo | Interfaz | Dirección IP | Rol |
|---|---|---|---|
| R1 | Fa0/0 | 10.7.80.1/30 | WAN → ISP |
| R1 | Fa1/0 | 192.168.7.1/24 | LAN |
| R1 | Tunnel0 | 172.16.78.1/30 | Tunnel VPN |
| ISP | Fa0/1 | 10.7.80.2/30 | WAN → R1 |
| ISP | Fa0/0 | 10.7.81.1/30 | WAN → R2 |
| R2 | Fa0/0 | 10.7.81.2/30 | WAN → ISP |
| R2 | Fa1/0 | 192.168.80.1/24 | LAN |
| R2 | Tunnel0 | 172.16.78.2/30 | Tunnel VPN |
| Linux1 | ens3 | 192.168.7.2/24 | Host LAN R1 |
| Linux2 | ens3 | 192.168.80.2/24 | Host LAN R2 |

---

## 4. Parámetros VPN

| Parámetro | Valor |
|---|---|
| Tipo de VPN | IPSec IKEv1 Route-Based |
| Fase 1 — Cifrado | AES-128 |
| Fase 1 — Hash | SHA-1 |
| Fase 1 — Autenticación | Pre-Shared Key (PSK) |
| Fase 1 — Grupo DH | Group 2 (1024-bit) |
| Fase 2 — Modo | Tunnel |
| Pre-Shared Key | Yeury0780 |
| Interfaz Tunnel | Tunnel0 |
| Red Tunnel | 172.16.78.0/30 |

---

## 5. Configuración

### 5.1 Configuración R1

> 📸 **SCREENSHOT:** Insertar captura del `show running-config` de R1 mostrando crypto ipsec profile, interface Tunnel0 y ip route

```
crypto ipsec transform-set TS esp-aes esp-sha-hmac
 mode tunnel
crypto ipsec profile VPN-PROFILE
 set transform-set TS
interface Tunnel0
 ip address 172.16.78.1 255.255.255.252
 tunnel source FastEthernet0/0
 tunnel destination 10.7.81.2
 tunnel protection ipsec profile VPN-PROFILE
ip route 192.168.80.0 255.255.255.0 172.16.78.2
```

### 5.2 Configuración R2

> 📸 **SCREENSHOT:** Insertar captura del `show running-config` de R2

```
crypto ipsec transform-set TS esp-aes esp-sha-hmac
 mode tunnel
crypto ipsec profile VPN-PROFILE
 set transform-set TS
interface Tunnel0
 ip address 172.16.78.2 255.255.255.252
 tunnel source FastEthernet0/0
 tunnel destination 10.7.80.1
 tunnel protection ipsec profile VPN-PROFILE
ip route 192.168.7.0 255.255.255.0 172.16.78.1
```

---

## 6. Verificación y Funcionamiento

### 6.1 Estado ISAKMP SA

Ejecutar en R1:
```
show crypto isakmp sa
```
> 📸 **SCREENSHOT:** Insertar captura mostrando estado **QM_IDLE ACTIVE**

### 6.2 Estado Tunnel0

Ejecutar en R1:
```
show interface Tunnel0
```
> 📸 **SCREENSHOT:** Insertar captura mostrando `Tunnel0 is up, line protocol is up` y `Tunnel protection via IPSec (profile "VPN-PROFILE")`

### 6.3 Estado IPSec SA

Ejecutar en R1:
```
show crypto ipsec sa
```
> 📸 **SCREENSHOT:** Insertar captura mostrando `#pkts encaps` y `#pkts decaps` con valores mayores a 0

### 6.4 Demostración de conectividad

Ejecutar en Linux1:
```
ping -c 4 192.168.80.2
```
> 📸 **SCREENSHOT:** Insertar captura del ping exitoso desde Linux1 hacia Linux2

---

## 7. Archivos del repositorio

| Archivo | Descripción |
|---|---|
| `YeuryLopez_20250780_Script_P2.txt` | Script de configuración |
| `YeuryLopez_20250780_Informe_P2.pdf` | Documentación técnica en PDF |
| `YeuryLopez_20250780_Links_P2.txt` | Enlace al video |
| `README.md` | Este archivo |
