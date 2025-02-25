# ✅ Configurar una VPN IPSec Site-to-Site entre dos routers en Cisco Packet Tracer

## Paso 1: Seleccionar los Dispositivos
En este paso, definimos la infraestructura de la red:

1. Routers (R1 y R2)

- Cada router tiene dos interfaces:
- Una LAN (red interna, conectada al switch).
- Una WAN (conexión entre los routers).
2. Switches (S1 y S2)

- Conectan los routers a las PCs dentro de cada red local.
3. PCs (PC1 y PC2)

- PC1 pertenece a la red 192.168.1.0/24.
- PC2 pertenece a la red 192.168.2.0/24.

## Paso 2: Configurar Direccionamiento IP en los Routers
Cada interfaz de los routers debe tener una IP para comunicarse correctamente.

🔹 En Router 1 (R1)
```sh
enable
configure terminal
hostname R1
```

- Establece un nombre para identificar el router.
```sh
interface GigabitEthernet0/0/0
ip address 192.168.2.1 255.255.255.0
no shutdown
exit
```
- Configura la interfaz LAN con la dirección IP 192.168.2.1 (puerta de enlace para PC2).
```sh
interface GigabitEthernet0/0/1
ip address 10.0.0.2 255.255.255.252
no shutdown
exit
```
- Configura la interfaz WAN con 10.0.0.2 para conectar con R1.
```sh
ip route 192.168.1.0 255.255.255.0 10.0.0.1
exit
```
- Agrega una ruta estática para llegar a la red 192.168.1.0/24 a través de 10.0.0.1.
## ✅ Verificación:
Usa el comando show ip interface brief en cada router para confirmar que las interfaces están activas y tienen la IP correcta.

## Paso 3: Configurar VPN IPsec (Fase 1 - IKE Policy)
En este paso, configuramos la autenticación inicial entre los routers.

```sh
crypto isakmp policy 10
encryption aes 256
hash sha
authentication pre-share
group 2
lifetime 86400
exit
```
- encryption aes 256 → Usa AES con clave de 256 bits.
- hash sha → Usa SHA para la integridad de los datos.
- authentication pre-share → Usa una clave compartida (pre-shared key).
- group 2 → Define el grupo Diffie-Hellman para el intercambio de claves.
- lifetime 86400 → Tiempo de vida del túnel (24 horas).
- ✅ Este paso debe hacerse en ambos routers.

## Paso 4: Configurar la Clave Precompartida
Cada router debe conocer la clave secreta para autenticarse.

🔹 En Router 1 (R1):
```sh
crypto isakmp key VPN-KEY address 10.0.0.2
```
🔹 En Router 2 (R2):
```sh
crypto isakmp key VPN-KEY address 10.0.0.1
```
- ✅ Ambos routers deben tener la misma clave (VPN-KEY).

## Paso 5: Configurar VPN IPsec (Fase 2 - Transform Set)
Define los métodos de encriptación.
```sh
crypto ipsec transform-set VPN-SET esp-aes 256 esp-sha-hmac
```
- ✅ Se debe configurar en ambos routers.

## Paso 6: Crear el Crypto Map
El crypto map define qué tráfico será protegido por la VPN.

🔹 En Router 1 (R1):
```sh
crypto map VPN-MAP 10 ipsec-isakmp
set peer 10.0.0.2
set transform-set VPN-SET
match address 101
exit
```
🔹 En Router 2 (R2):
```sh
crypto map VPN-MAP 10 ipsec-isakmp
set peer 10.0.0.1
set transform-set VPN-SET
match address 101
exit
```
⚠️ Posible Error:
Si ves este mensaje:
```sh
% NOTE: This new crypto map will remain disabled until a peer and a valid access list have been configured.
```
Debes asegurarte de configurar una lista de acceso.

## Paso 7: Crear la Lista de Acceso
Permite el tráfico entre ambas redes a través de la VPN.
```sh
access-list 101 permit ip 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255
```
- ✅ Configurar en ambos routers.

## Paso 8: Aplicar el Crypto Map a la Interfaz WAN
🔹 En Router 1 (R1):
```sh
interface GigabitEthernet0/0/1
crypto map VPN-MAP
exit
```
🔹 En Router 2 (R2):
```sh
interface GigabitEthernet0/0/1
crypto map VPN-MAP
exit
```
✅ Esto activa la VPN en cada router.

## Paso 9: Configurar las PCs
🔹 PC1
```sh
IP: 192.168.1.10
Máscara: 255.255.255.0
Puerta de enlace: 192.168.1.1 (IP del Router R1)
```
🔹 PC2
```sh
IP: 192.168.2.10
Máscara: 255.255.255.0
Puerta de enlace: 192.168.2.1 (IP del Router R2)
```
✅ Ambas PCs deben poder hacer ping a su puerta de enlace antes de probar la VPN.

## Paso 10: Verificación
En los routers:
```sh
show crypto isakmp sa
```
- Debe mostrar `QM_IDLE`, lo que indica que la fase 1 está activa.
```sh
show crypto ipsec sa
```
- Debe mostrar que los paquetes están siendo cifrados y descifrados.

## Pruebas
🔹 Desde PC1, prueba conectividad a PC2:
```sh
ping 192.168.2.10
```
🔹 Desde PC2, prueba conectividad a PC1:
```sh
ping 192.168.1.10
```


