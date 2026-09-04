# Topología Física - Sede Interior del País

## Proyecto: Red Empresarial para Operador Logístico
## Fase: Design (PPDIOO) - Diseño de Red
## Alcance: Sede Interior del País

---

##  Descripción General

Este documento describe la **topología física** necesaria para la sede del **Interior del país**, según lo definido en la Fase Design del proyecto.

La sede del Interior tiene un diseño de **Core/Distribución colapsado**, lo que significa que un solo switch L3 cumple las funciones de núcleo y distribución. Su criticidad es menor que Panamá y Colón, por lo que no cuenta con redundancia de equipos (HSRP, EtherChannel), pero sí con un respaldo de conectividad móvil 4G/5G.

---

##  Dispositivos Necesarios

| Dispositivo | Cantidad | Descripción Técnica | Equivalente en Packet Tracer |
| :--- | :--- | :--- | :--- |
| **Switch L3 (Core/Distribución)** | 1 | Switch multicapa que actúa como gateway de la red local (enrutamiento inter-VLAN). | Cisco Catalyst 3560 o 3650 |
| **Switch L2 (Acceso)** | 1 | Switch de acceso para conectar equipos finales (PCs, impresoras, teléfonos IP). | Cisco Catalyst 2960 |
| **Router / Firewall de Sucursal** | 1 | Concentra el enlace WAN (Internet) y sirve como firewall perimetral. | Cisco 1941 / 2901 / 4331 |
| **Router 4G/5G (Respaldo)** | 1 | Router celular para respaldo por ruta estática flotante (R-06). | Cisco 1941 (simulado) |
| **Access Points (APs)** | 2-3 | Puntos de acceso inalámbrico para cobertura de oficina y área comercial (VLAN 10). | Access Point genérico PT |

---

##  Conexiones Físicas

### Diagrama de Topología
<img width="1107" height="507" alt="image" src="https://github.com/user-attachments/assets/e967f2dd-9a48-4e51-9dfd-6614fa4760d4" />


### Tabla de Conexiones

| Origen | Destino | Tipo de Cable | Notas |
| :--- | :--- | :--- | :--- |
| RTR-Interior | ISP (Internet) | Serial DCE o Copper Straight-Through | Enlace WAN principal |
| RTR-4G-Interior | RTR-Interior | Copper Straight-Through | Respaldo celular (conexión directa entre routers) |
| RTR-Interior | SW-Core-Interior | Copper Straight-Through | Enlace troncal / ruteo |
| SW-Core-Interior | SW-Acceso-Interior | Copper Straight-Through | Enlace troncal (Trunk) |
| SW-Acceso-Interior | PC0 | Copper Straight-Through | Acceso (VLAN 10 - Datos) |
| SW-Acceso-Interior | IP Phone0 | Copper Straight-Through | Acceso (VLAN 30 - Voz) |
| SW-Acceso-Interior | AP-1 | Copper Straight-Through | Acceso (PoE - Wi-Fi) |
| SW-Acceso-Interior | AP-2 | Copper Straight-Through | Acceso (PoE - Wi-Fi) |

---

##  Justificación Técnica

### 1. Topología Colapsada (Core/Distribución)
- El documento de Design especifica para el Interior: *"Core/Distribución colapsado (1 switch L3, o 2 en HA si presupuesto lo permite)"*.
- Se utiliza **1 switch L3** para reducir costos, ya que la sede tiene menor criticidad que Panamá o Colón.

### 2. Respaldo 4G/5G
- Según la sección 4.2 del documento, el respaldo móvil **no corre OSPF**.
- Se implementa una **ruta estática flotante** con distancia administrativa mayor que la ruta OSPF.
- El router 4G/5G se conecta **directamente al router principal** (RTR-Interior), no al switch L3, para que el router de borde decida el failover sin afectar al core.

### 3. Acceso Inalámbrico
- Se utilizan **2-3 APs** para cubrir oficina y área comercial.
- Los APs son gestionados remotamente por el WLC ubicado en Panamá (arquitectura FlexConnect/Local).

### 4. Seguridad
- El RTR-Interior actúa como firewall perimetral de la sucursal.
- El tráfico hacia Internet **no sale localmente**: todo el tráfico se encapsula en el túnel SD-WAN hacia Panamá.

---

##  Cumplimiento de Requisitos (Fase Design)

| Requisito | ¿Cumple? | Detalle |
| :--- | :--- | :--- |
| Topología colapsada (1 switch L3) | ✅ | SW-Core-Interior |
| Capa de acceso (1 switch L2) | ✅ | SW-Acceso-Interior |
| Router/Firewall de sucursal | ✅ | RTR-Interior |
| Respaldo 4G/5G | ✅ | RTR-4G-Interior conectado al router principal |
| APs inalámbricos | ✅ | AP-1 y AP-2 |
| Enlace WAN | ✅ | Conexión al ISP para túnel SD-WAN |

---

##  Archivos Relacionados

- `Fase_Design_Red_Logistica.pdf` - Documento fuente de la Fase Design
- `topologia_fisica_interior.pkt` - Archivo Packet Tracer (si aplica)

---

##  Próximos Pasos

- Configuración lógica de VLANs (10, 30, 99).
- Configuración de enrutamiento (OSPF área 30 como stub).
- Configuración de ruta estática flotante hacia el router 4G/5G.
- Configuración del túnel SD-WAN hacia Panamá.

---

*Documento generado como parte del proyecto de portafolio técnico - Fase Design (PPDIOO)*

