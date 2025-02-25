# 🔹 Objetivo del Ejemplo

## ✅ Configurar una VPN IPSec Site-to-Site entre dos routers en Cisco Packet Tracer

## ✅ Aplicar reglas de firewall con listas de control de acceso (ACLs)

### 🖥️ Topología de Red

### 📌 Dispositivos

- 2 Routers (R1 y R2)

- 2 Switches

- 2 PCs (una en cada red local)

### 📌 Direcciones IP

Dispositivo  | Interfaz     | IP

------------ | ----------- | -----------------

R1          | G0/0        | 192.168.1.1/24

R1          | S0/0/0      | 10.0.0.1/30

R2          | S0/0/0      | 10.0.0.2/30

R2          | G0/0        | 192.168.2.1/24

PC1         | NIC         | 192.168.1.10/24

PC2         | NIC         | 192.168.2.10/24

# 🔹 Paso 1: Configurar las Interfaces en los Routers

## Configuración en R1

```sh
enable
configure terminal

interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
exit

interface Serial0/0/0
 ip address 10.0.0.1 255.255.255.252
 clock rate 64000
 no shutdown
exit
```

## Configuración en R2

```sh
enable
configure terminal

interface GigabitEthernet0/0
 ip address 192.168.2.1 255.255.255.0
 no shutdown
exit

interface Serial0/0/0
 ip address 10.0.0.2 255.255.255.252
 no shutdown
exit
```

## 📌 Verifica conectividad con ping

```sh
ping 10.0.0.2   # Desde R1 a R2
ping 10.0.0.1   # Desde R2 a R1
```

# 🔹 Paso 2: Configurar VPN IPSec Site-to-Site

## 📌 En R1

```sh
crypto isakmp policy 10
 encr aes
 hash sha256
 authentication pre-share
 group 2
 lifetime 86400
exit

crypto isakmp key MI_CLAVE_VPN address 10.0.0.2

crypto ipsec transform-set VPN-SET esp-aes esp-sha-hmac

crypto map VPN-MAP 10 ipsec-isakmp
 set peer 10.0.0.2
 set transform-set VPN-SET
 match address ACL-VPN
exit

interface Serial0/0/0
 crypto map VPN-MAP
exit

access-list 100 permit ip 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255
```

## 📌 En R2

```sh
crypto isakmp policy 10
 encr aes
 hash sha256
 authentication pre-share
 group 2
 lifetime 86400
exit

crypto isakmp key MI_CLAVE_VPN address 10.0.0.1

crypto ipsec transform-set VPN-SET esp-aes esp-sha-hmac

crypto map VPN-MAP 10 ipsec-isakmp
 set peer 10.0.0.1
 set transform-set VPN-SET
 match address ACL-VPN
exit

interface Serial0/0/0
 crypto map VPN-MAP
exit

access-list 100 permit ip 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255
```

## 📌 Verifica la VPN

```sh
show crypto isakmp sa
show crypto ipsec sa
```

# 🔹 Paso 3: Configurar el Firewall con ACL

## 📌 En R1: Bloquear Ping y Permitir solo HTTP

```sh
access-list 110 deny icmp any any
access-list 110 permit tcp any any eq 80
access-list 110 deny ip any any

interface GigabitEthernet0/0
 ip access-group 110 in
exit
```

## 📌 En R2: Bloquear Todo Menos HTTP y HTTPS

```sh
access-list 120 deny ip any any
access-list 120 permit tcp any any eq 80
access-list 120 permit tcp any any eq 443

interface GigabitEthernet0/0
 ip access-group 120 in
exit
```

## 📌 Verifica las ACL activas

```sh
show access-lists
```

# 🔹 Paso 4: Pruebas Finales

### ✅ Desde PC1 (192.168.1.10) haz ping a PC2 (192.168.2.10) → ❌ Debe fallar porque bloqueamos ICMP

### ✅ Desde PC1 abre un navegador e ingresa <http://192.168.2.10> → ✅ Debe funcionar porque permitimos HTTP

### ✅ Desde PC1 intenta acceder por SSH a 192.168.2.10 → ❌ Debe fallar porque solo HTTP/HTTPS está permitido

# 📌 Conclusión

🔹 VPN IPSec cifra la comunicación entre R1 y R2

🔹 Firewall con ACLs restringe el tráfico no autorizado

🔹 Se aplicaron reglas de seguridad para evitar accesos no deseados
