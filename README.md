# Red-empresarial-multisitio-para-operador-logstico-Proyecto-CCNA
## Descripción
Este proyecto simula, en Cisco Packet Tracer, la infraestructura de red de una empresa de logística con presencia en tres puntos estratégicos de Panamá: una sede administrativa en Ciudad de Panamá, una bodega/puerto en Colón (Zona Libre) y una sucursal comercial en el interior del país.

El objetivo es aplicar de forma integrada los conceptos del temario CCNA (200-301) en un escenario realista, en lugar de practicarlos como labs aislados. La topología busca reflejar cómo opera en la práctica una empresa panameña del sector logístico: múltiples sitios, alta disponibilidad, segmentación por departamento y necesidad de administración centralizada y segura.

## Motivación

Panamá basa buena parte de su economía en su posición logística (Canal, puertos, Zona Libre de Colón). El sector de comercio y logística se mantiene entre los de mayor demanda de contratación en el país, junto con finanzas y tecnología de la información. Este proyecto fue diseñado como parte de mi preparación para la certificación Cisco CCNA y como pieza de portafolio para aplicar a posiciones de soporte de redes / ingeniería de redes junior en el mercado panameño.

## Objetivos técnicos

- Diseñar una topología multisitio con enlaces WAN punto a punto
- Segmentar el tráfico por departamento mediante VLANs
- Implementar enrutamiento entre VLANs y entre sitios con OSPF
- Garantizar redundancia de gateway en el sitio principal (HSRP)
- Centralizar servicios de red: DHCP, NTP, Syslog
- Aplicar seguridad básica: ACLs, SSH, port security
- Documentar el diseño de forma clara y reproducible

## Topología

| Sitio | Rol | Dispositivos principales |
|---|---|---|
| Ciudad de Panamá (HQ) | Sede administrativa | Router core, switch L3, servidor DHCP/NTP/Syslog |
| Colón (Zona Libre) | Bodega y puerto | Router, switch, punto de acceso inalámbrico |
| Interior | Sucursal comercial | Router, switch |

Los tres sitios están conectados en estrella hacia la sede central mediante enlaces seriales WAN, con OSPF área 0 corriendo entre los tres routers.

*(Ver `/docs/topologia.png` para el diagrama completo.)*

## Direccionamiento IP

| Sitio | VLAN | Red | Uso |
|---|---|---|---|
| HQ Panamá | 10 | 192.168.10.0/24 | Administración |
| HQ Panamá | 20 | 192.168.20.0/24 | Ventas |
| HQ Panamá | 30 | 192.168.30.0/24 | Operaciones |
| HQ Panamá | 40 | 192.168.40.0/24 | IT / Servidores |
| HQ Panamá | 99 | 192.168.99.0/24 | Invitados |
| Colón | 30 | 192.168.130.0/24 | Operaciones / Aduanas |
| Colón | 50 | 192.168.150.0/24 | Wifi bodega |
| Interior | 10 | 192.168.210.0/24 | Administración |
| Interior | 20 | 192.168.220.0/24 | Ventas |
| WAN R1↔R2 | — | 10.0.0.0/30 | Enlace punto a punto |
| WAN R1↔R3 | — | 10.0.0.4/30 | Enlace punto a punto |

## Tecnologías y protocolos aplicados

VLANs, trunking (802.1Q), EtherChannel, Spanning Tree (PVST+), inter-VLAN routing, OSPF multiárea, HSRP, DHCP, NAT/PAT, ACLs estándar y extendidas, SSH, NTP, Syslog, port security, wireless (WLC/AP).

## Estructura del repositorio

```
├── README.md
├── proyecto.pkt              # Archivo final de Packet Tracer
├── docs/
│   ├── topologia.png          # Diagrama de la red
│   ├── tabla-direccionamiento.pdf
│   └── decisiones-de-diseño.md
└── config/
    ├── R1-config.txt
    ├── R2-config.txt
    ├── R3-config.txt
    └── SW-configs.txt
```

## Estado del proyecto

- [x] Diseño de topología y direccionamiento
- [ ] Cableado y configuración básica de IPs
- [ ] VLANs e inter-VLAN routing
- [ ] OSPF entre los tres sitios
- [ ] HSRP en HQ
- [ ] DHCP, NAT y servicios centralizados
- [ ] ACLs y seguridad
- [ ] Documentación final y video demostrativo

## Autor

