# Practica 02: Interconexion de Dos Sedes mediante Enlace Serial y Enrutamiento Estatico

## Caso de Estudio
Una organizacion cuenta con dos sedes, **POSTA-1** y **POSTA-2**, cada una con su propia red local. Se requiere interconectarlas mediante un enlace WAN serial punto a punto para que los equipos de ambas sedes puedan comunicarse entre si.

Para cumplir con este requerimiento, se implemento una topologia con un router Cisco y un switch Cisco 2960 por sede. Los routers se enlazan por una interfaz serial (DCE/DTE) y utilizan rutas estaticas para alcanzar la red remota.

---

## Diagrama de la Topologia

![Topologia de Red](./img/topologia.png)

---

## Tabla de Direccionamiento

| Dispositivo | Interfaz | Direccion IP | Mascara | Gateway |
| :--- | :--- | :--- | :--- | :--- |
| POSTA-1 | Gig0/0 | 192.168.10.1 | 255.255.255.0 | N/A |
| POSTA-1 | Se0/3/0 | 192.168.100.1 | 255.255.255.252 | N/A |
| POSTA-2 | Gig0/0 | 192.168.20.1 | 255.255.255.0 | N/A |
| POSTA-2 | Se0/3/0 | 192.168.100.2 | 255.255.255.252 | N/A |
| Laptop1 | Fa0 | 192.168.10.2 | 255.255.255.0 | 192.168.10.1 |
| PC1 | Fa0 | 192.168.10.3 | 255.255.255.0 | 192.168.10.1 |
| Laptop2 | Fa0 | 192.168.20.2 | 255.255.255.0 | 192.168.20.1 |
| PC2 | Fa0 | 192.168.20.3 | 255.255.255.0 | 192.168.20.1 |

---

## Configuracion de Equipos

### 1. Router POSTA-1 (DCE)

```text
Router> enable
Router# configure terminal
Router(config)# hostname POSTA-1

POSTA-1(config)# interface GigabitEthernet0/0
POSTA-1(config-if)# ip address 192.168.10.1 255.255.255.0
POSTA-1(config-if)# no shutdown

POSTA-1(config)# interface Serial0/3/0
POSTA-1(config-if)# ip address 192.168.100.1 255.255.255.252
POSTA-1(config-if)# clock rate 64000
POSTA-1(config-if)# no shutdown
POSTA-1(config-if)# exit

! Ruta estatica hacia la LAN de POSTA-2
POSTA-1(config)# ip route 192.168.20.0 255.255.255.0 192.168.100.2
POSTA-1(config)# end
POSTA-1# copy running-config startup-config
```

### 2. Router POSTA-2 (DTE)

```text
Router> enable
Router# configure terminal
Router(config)# hostname POSTA-2

POSTA-2(config)# interface GigabitEthernet0/0
POSTA-2(config-if)# ip address 192.168.20.1 255.255.255.0
POSTA-2(config-if)# no shutdown

POSTA-2(config)# interface Serial0/3/0
POSTA-2(config-if)# ip address 192.168.100.2 255.255.255.252
POSTA-2(config-if)# no shutdown
POSTA-2(config-if)# exit

! Ruta estatica hacia la LAN de POSTA-1
POSTA-2(config)# ip route 192.168.10.0 255.255.255.0 192.168.100.1
POSTA-2(config)# end
POSTA-2# copy running-config startup-config
```

### 3. Switches
Los switches de ambas sedes operan con su configuracion por defecto (VLAN 1), sin segmentacion adicional. Los equipos finales se conectan a Fa0/1 y Fa0/2, y el enlace hacia el router sale por Gig0/1.

### 4. Equipos finales
Direccionamiento IP estatico segun la tabla anterior.

---

## Verificacion y Pruebas

```text
POSTA-1# show ip interface brief
POSTA-1# show ip route
POSTA-2# show ip route
```

| Origen | Destino | Resultado esperado |
| :--- | :--- | :--- |
| Laptop1 | PC1 | Exitoso (misma LAN) |
| Laptop1 | 192.168.10.1 (gateway) | Exitoso |
| Laptop1 | 192.168.100.2 (serial remoto) | Exitoso |
| Laptop1 | Laptop2 / PC2 | Exitoso (a traves del enlace WAN) |
| PC2 | Laptop1 / PC1 | Exitoso |

> Nota: el primer ping suele fallar mientras se resuelve ARP; los siguientes responden con normalidad.

![Pruebas de conectividad](./img/pruebas.png)

---

## Conclusiones
(Pendiente de redactar al terminar las pruebas.)
