SW-CORE-PTY-1 ( f0/2 )   ->  SW-CORE-PTY-2 ( f0/1 )
SW-CORE-PTY-1 ( g0/2 )   ->  SW-DIST-PTY-1 ( g0/1 ) 

# Diseño e implementación de red LAN jerárquica – Sede Panamá

Red de campus para la sede de Panamá basada en el modelo jerárquico de tres capas (Core, Distribución y Acceso), con segmentación por VLANs, enrutamiento entre VLANs en el Core y diseño preparado para alta disponibilidad con HSRP.

![Estado](https://img.shields.io/badge/estado-en%20desarrollo-yellow)
![Plataforma](https://img.shields.io/badge/Cisco-IOS-blue)

## Tabla de contenido

- [Objetivo](#objetivo)
- [Herramientas](#herramientas)
- [Topología](#topología)
- [Plan de VLANs y direccionamiento](#plan-de-vlans-y-direccionamiento)
- [Decisiones de diseño](#decisiones-de-diseño)
- [Implementación](#implementación)
- [Verificación](#verificación)
- [Problemas encontrados y aprendizajes](#problemas-encontrados-y-aprendizajes)
- [Próximos pasos](#próximos-pasos)

## Objetivo

Diseñar y configurar la infraestructura LAN de la sede de Panamá de forma escalable y segmentada, centralizando el enrutamiento entre VLANs en la capa Core y dejando la base lista para redundancia de gateway con HSRP.

## Herramientas

- Simulador: Cisco Packet Tracer / GNS3 *(indica el que usaste)*
- Sistema operativo: Cisco IOS
- Equipos: switches L3 (Core) y switches de Distribución/Acceso

## Topología

![Topología de la sede Panamá](docs/topologia.png)

*Sustituye esta imagen por la captura de tu topología en `docs/topologia.png`.*

| Capa | Dispositivos |
|------|--------------|
| Core | `SW-CORE-PTY-1`, `SW-CORE-PTY-2` |
| Distribución | `SW-DIST-PTY-1` |
| Acceso | *(pendiente)* |

## Plan de VLANs y direccionamiento

### VLANs

| VLAN | Nombre  | Subred          | Propósito                |
|------|---------|-----------------|--------------------------|
| 10   | ADMIN   | 10.10.10.0/24   | Usuarios administrativos |
| 20   | SERVERS | 10.10.20.0/24   | Servidores               |
| 30   | VOICE   | 10.10.30.0/24   | Telefonía IP             |
| 90   | GUEST   | 10.10.90.0/24   | Invitados                |
| 99   | MGMT    | 10.10.99.0/24   | Gestión de equipos       |
| 999  | NATIVE  | —               | VLAN nativa sin uso (troncales) |

### Gestión (VLAN 99)

| Dispositivo     | IP             |
|-----------------|----------------|
| `SW-CORE-PTY-1` | 10.10.99.1/24  |
| `SW-CORE-PTY-2` | 10.10.99.2/24  |
| `SW-DIST-PTY-1` | 10.10.99.11/24 |

### SVIs de usuario en el Core

Cada Core tiene una IP real por VLAN. La `.1` queda reservada como gateway virtual (HSRP).

| VLAN | Core 1        | Core 2        | Gateway virtual (HSRP) |
|------|---------------|---------------|------------------------|
| 10   | 10.10.10.2    | 10.10.10.3    | 10.10.10.1             |
| 20   | 10.10.20.2    | 10.10.20.3    | 10.10.20.1             |
| 30   | 10.10.30.2    | 10.10.30.3    | 10.10.30.1             |
| 90   | 10.10.90.2    | 10.10.90.3    | 10.10.90.1             |

### Enlaces

| Origen          | Puerto | Destino         | Puerto | Tipo  |
|-----------------|--------|-----------------|--------|-------|
| `SW-CORE-PTY-1` | f0/2   | `SW-CORE-PTY-2` | f0/1   | Trunk |
| `SW-CORE-PTY-1` | g0/2   | `SW-DIST-PTY-1` | g0/1   | Trunk |

## Decisiones de diseño

- **Enrutamiento entre VLANs en el Core.** Los switches Core son L3 con `ip routing` y una SVI por VLAN. Distribución solo transporta tráfico (troncales) y gestión, lo que simplifica el diseño y centraliza el enrutamiento.
- **Preparación para HSRP.** Cada Core usa su propia IP real (`.2` y `.3`) y se reserva la `.1` como gateway virtual, de modo que agregar HSRP no obliga a renumerar.
- **Troncales restringidos.** Solo se permiten las VLANs necesarias (10, 20, 30, 90, 99) con encapsulación 802.1Q, reduciendo el dominio de broadcast y la superficie de ataque.
- **VLAN nativa dedicada (999).** Se evita usar la VLAN 1 como nativa para mitigar ataques de VLAN hopping.
- **DTP deshabilitado.** `switchport nonegotiate` en los troncales evita negociaciones de trunk no deseadas.
- **VLAN de gestión dedicada.** Aísla el tráfico administrativo del tráfico de usuarios.
- **Distribución como L2 para gestión.** El switch de Distribución no enruta; usa `ip default-gateway` apuntando al Core, manteniendo una única capa de enrutamiento.

## Implementación

> Las contraseñas se representan con placeholders. **No publiques credenciales reales.**

<details>
<summary><b>SW-CORE-PTY-1</b></summary>

```cisco
enable
configure terminal
hostname SW-CORE-PTY-1
no ip domain-lookup
service password-encryption
enable secret <CONTRASEÑA>

! VLANs
vlan 10
 name ADMIN
vlan 20
 name SERVERS
vlan 30
 name VOICE
vlan 90
 name GUEST
vlan 99
 name MGMT
vlan 999
 name NATIVE
exit

! Enrutamiento L3
ip routing

! Troncal hacia SW-CORE-PTY-2
interface f0/2
 description Enlace a SW-CORE-PTY-2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport nonegotiate
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,30,90,99
 no shutdown
exit

! Troncal hacia SW-DIST-PTY-1
interface g0/2
 description Enlace a SW-DIST-PTY-1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport nonegotiate
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,30,90,99
 no shutdown
exit

! SVIs de usuario
interface vlan 10
 ip address 10.10.10.2 255.255.255.0
 no shutdown
interface vlan 20
 ip address 10.10.20.2 255.255.255.0
 no shutdown
interface vlan 30
 ip address 10.10.30.2 255.255.255.0
 no shutdown
interface vlan 90
 ip address 10.10.90.2 255.255.255.0
 no shutdown

! SVI de gestión
interface vlan 99
 ip address 10.10.99.1 255.255.255.0
 no shutdown
exit

end
write memory
```

</details>

<details>
<summary><b>SW-CORE-PTY-2</b></summary>

```cisco
enable
configure terminal
hostname SW-CORE-PTY-2
no ip domain-lookup
service password-encryption
enable secret <CONTRASEÑA>

! VLANs
vlan 10
 name ADMIN
vlan 20
 name SERVERS
vlan 30
 name VOICE
vlan 90
 name GUEST
vlan 99
 name MGMT
vlan 999
 name NATIVE
exit

! Enrutamiento L3
ip routing

! Troncal hacia SW-CORE-PTY-1
interface f0/1
 description Enlace a SW-CORE-PTY-1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport nonegotiate
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,30,90,99
 no shutdown
exit

! SVIs de usuario
interface vlan 10
 ip address 10.10.10.3 255.255.255.0
 no shutdown
interface vlan 20
 ip address 10.10.20.3 255.255.255.0
 no shutdown
interface vlan 30
 ip address 10.10.30.3 255.255.255.0
 no shutdown
interface vlan 90
 ip address 10.10.90.3 255.255.255.0
 no shutdown

! SVI de gestión
interface vlan 99
 ip address 10.10.99.2 255.255.255.0
 no shutdown
exit

end
write memory
```

</details>

<details>
<summary><b>SW-DIST-PTY-1</b></summary>

```cisco
enable
configure terminal
hostname SW-DIST-PTY-1
no ip domain-lookup
service password-encryption
enable secret <CONTRASEÑA>

! VLANs
vlan 10
 name ADMIN
vlan 20
 name SERVERS
vlan 30
 name VOICE
vlan 90
 name GUEST
vlan 99
 name MGMT
vlan 999
 name NATIVE
exit

! Troncal hacia SW-CORE-PTY-1
interface g0/1
 description Enlace a SW-CORE-PTY-1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport nonegotiate
 switchport trunk native vlan 999
 switchport trunk allowed vlan 10,20,30,90,99
 no shutdown
exit

! IP de gestión
interface vlan 99
 ip address 10.10.99.11 255.255.255.0
 no shutdown
exit

! Gateway de gestión (el enrutamiento se realiza en el Core)
ip default-gateway 10.10.99.1

end
write memory
```

</details>

## Verificación

Comandos utilizados en cada switch:

```cisco
show vlan brief
show interfaces trunk
show ip interface brief
```

Resultado esperado: VLANs creadas, puertos en modo troncal con las VLANs permitidas y SVIs en estado `up/up`.

| Comando | Captura |
|---------|---------|
| `show vlan brief` | ![vlan](docs/show-vlan-brief.png) |
| `show interfaces trunk` | ![trunk](docs/show-interfaces-trunk.png) |
| `show ip interface brief` | ![ip](docs/show-ip-interface-brief.png) |

*Reemplaza las imágenes con tus propias capturas.*

## Problemas encontrados y aprendizajes

*(Completa esta sección con tu experiencia real. Ejemplos de lo que suma valor:)*

- Incompatibilidad de encapsulación en troncales (`switchport trunk encapsulation dot1q` necesario en switches L3).
- SVIs en estado `down` por no existir la VLAN o no haber puertos activos en ella.
- Diferencias de VLAN nativa entre extremos del troncal y cómo se detectan.

## Próximos pasos

- [ ] HSRP entre `SW-CORE-PTY-1` y `SW-CORE-PTY-2`
- [ ] `SW-DIST-PTY-2` y switches de acceso
- [ ] Integración del WLC
- [ ] Conectividad con firewalls (`FW-PTY-1`, `FW-PTY-2`)
- [ ] Hardening: SSH, port-security, STP (root primario/secundario, BPDU Guard)

## Estructura del repositorio

```
.
├── README.md
├── configs/
│   ├── SW-CORE-PTY-1.txt
│   ├── SW-CORE-PTY-2.txt
│   └── SW-DIST-PTY-1.txt
└── docs/
    ├── topologia.png
    └── (capturas de verificación)
```
