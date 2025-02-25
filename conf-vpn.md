# Configuración de VPN IPsec y Firewall en Routers Cisco

## 1️⃣ Configurar Fase 1 de IPsec (IKE Policy)

Define los parámetros de autenticación y cifrado para la fase 1 de la VPN.

```shell
crypto isakmp policy 10
encryption aes 256
hash sha
authentication pre-share
group 2
lifetime 86400
exit
```

## 2️⃣ Configurar la Clave Precompartida (Pre-Shared Key)

Cada router debe conocer la clave secreta para autenticarse entre sí.

```shell
crypto isakmp key VPN-KEY address <IP-DEL-PEER>
```

- **R1:** `crypto isakmp key VPN-KEY address 10.0.0.2`
- **R2:** `crypto isakmp key VPN-KEY address 10.0.0.1`

## 3️⃣ Configurar Fase 2 de IPsec (Transform Set)

Define los algoritmos de cifrado y autenticación para la fase 2 de la VPN.

```shell
crypto ipsec transform-set VPN-SET esp-aes 256 esp-sha-hmac
```

## 4️⃣ Crear el Crypto Map

Especifica los pares, transform set y la lista de acceso que definirá el tráfico cifrado.

```shell
crypto map VPN-MAP 10 ipsec-isakmp
set peer <IP-DEL-PEER>
set transform-set VPN-SET
match address 101
exit
```

- **R1:** `set peer 10.0.0.2`
- **R2:** `set peer 10.0.0.1`

## 5️⃣ Definir Lista de Acceso (ACL) para VPN

Filtra el tráfico que debe ser cifrado por la VPN.

```shell
access-list 101 permit ip 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255
```

## 6️⃣ Aplicar el Crypto Map a la Interfaz WAN

Habilita la VPN en la interfaz de salida de cada router.

```shell
interface GigabitEthernet0/0/1
crypto map VPN-MAP
exit
```

## 🔥 Configurar Firewall Básico en el Router

**1. Permitir tráfico VPN:**

```shell
access-list 110 permit esp any any
access-list 110 permit udp any any eq isakmp
access-list 110 permit udp any any eq non500-isakmp
```

**2. Aplicar Firewall en la interfaz WAN:**

```shell
interface GigabitEthernet0/0/1
ip access-group 110 in
exit
```

## ✅ Verificación de la VPN

Para comprobar que la VPN está funcionando correctamente, usa estos comandos:

```shell
show crypto isakmp sa  # Verifica la fase 1
show crypto ipsec sa   # Verifica la fase 2
```

---

Con esta configuración, la **VPN IPsec** estará activa y protegida por un **Firewall** básico. 🚀
