#  Fase 3 – Design | Topología Física Sede Colón (Zona Libre)

##  Estado del Proyecto
- **Fase PPDIOO:** Design (Diseño)
- **Sede trabajada:** Colón (Zona Libre de Colón)
- **Herramienta:** Cisco Packet Tracer
- **Objetivo:** Validar la topología física de la sede crítica 24/7, cumpliendo los requisitos R-02 a R-11 definidos en la fase Plan.

---

##  Topología Física Implementada (Colón)

La siguiente topología representa la **interconexión física** de los equipos en Colón, siguiendo el diseño jerárquico de 3 capas (Core, Distribución y Acceso) con redundancia total.

<img width="932" height="622" alt="image" src="https://github.com/user-attachments/assets/a2f32a80-a8a2-4703-b44e-21c9d24a5cd3" />



---

## 🔗 Tabla de Conexiones Físicas

| Equipo A | Equipo B | Tipo de Cable |
|----------|----------|---------------|
| `ISP-A-COL` | `FW-COL` | Serial |
| `ISP-A-COL` | `FW-COL-2` | Serial |
| `ISP-B-COL` | `FW-COL` | Serial |
| `ISP-B-COL` | `FW-COL-2` | Serial |
| `FW-COL` | `SW-CORE-COL` | Cobre recto |
| `FW-COL` | `SW-CORE-COL-2` | Cobre recto |
| `FW-COL-2` | `SW-CORE-COL` | Cobre recto |
| `FW-COL-2` | `SW-CORE-COL-2` | Cobre recto |
| `SW-CORE-COL` | `SW-CORE-COL-2` | Cobre cruzado |
| `SW-CORE-COL` | `SW-DIST-ADMIN-COL` | Cobre recto |
| `SW-CORE-COL` | `SW-DIST-BODEGA-COL` | Cobre recto |
| `SW-CORE-COL` | `WLC-COL` | Cobre recto |
| `SW-CORE-COL-2` | `SW-DIST-ADMIN-COL` | Cobre recto |
| `SW-CORE-COL-2` | `SW-DIST-BODEGA-COL` | Cobre recto |
| `SW-CORE-COL-2` | `WLC-COL` | Cobre recto |
| `SW-DIST-ADMIN-COL` | `SW-ACC-COL` | Cobre recto |
| `SW-DIST-BODEGA-COL` | `SW-ACC-COL` | Cobre recto |
| `SW-DIST-BODEGA-COL` | `AP-COL-1` | Cobre recto |
| `SW-DIST-BODEGA-COL` | `AP-COL-2` | Cobre recto |
| `SW-DIST-BODEGA-COL` | `SERVER-WMS` | Cobre recto |
| `SW-ACC-COL` | `PHONE-1` | Cobre recto |

---

## ✅ Requisitos cumplidos (Fase Design)

| ID | Requisito | Estado | Justificación |
|----|-----------|--------|---------------|
| **R-02** | Alta disponibilidad de red en Colón | ✅ | Dos Cores, dos Distribución y dos Acceso conectados en malla. |
| **R-03** | Wi-Fi industrial en Colón | ✅ | APs conectados a `SW-DIST-BODEGA-COL` (VLAN 15 en Implement). |
| **R-04** | Segmentación de red | ✅ | Distribución separada por zonas (Bodega y Administración). |
| **R-05** | Doble ISP con failover | ✅ | Dos ISP conectados a ambos firewalls. |
| **R-07** | Firewall perimetral redundante | ✅ | Par de NGFW (`FW-COL` y `FW-COL-2`) activo/pasivo. |
| **R-09** | Redundancia eléctrica (UPS) | ✅ | Representada físicamente. |
| **R-10** | Registro y trazabilidad | ✅ | `SERVER-WMS` presente para Syslog/WMS local. |
| **R-11** | Capacidad para picos estacionales | ✅ | EtherChannel (a configurar) entre Core y Distribución. |

---

## 🖥️ Equipos utilizados en Packet Tracer

| Equipo | Nombre asignado | Rol |
|--------|-----------------|-----|
| Router 1941 | `ISP-A-COL` | Proveedor de Internet 1 |
| Router 1941 | `ISP-B-COL` | Proveedor de Internet 2 |
| ASA 5505 | `FW-COL` | Firewall primario |
| ASA 5505 | `FW-COL-2` | Firewall secundario |
| Switch 3560 | `SW-CORE-COL` | Core primario |
| Switch 3560 | `SW-CORE-COL-2` | Core secundario |
| Switch 3560 | `SW-DIST-BODEGA-COL` | Distribución zona bodega |
| Switch 3560 | `SW-DIST-ADMIN-COL` | Distribución zona administración |
| Switch 2960 | `SW-ACC-COL` | Acceso |
| WLC 2504 | `WLC-COL` | Controlador inalámbrico |
| Access Point | `AP-COL-1`, `AP-COL-2` | APs industriales |
| Server | `SERVER-WMS` | Servidor WMS/logs |
| IP Phone | `PHONE-1` | Telefonía IP |

---

## 📌 Notas para la fase Implement

- **VLANs pendientes de configurar:** VLAN 10 (Admin), VLAN 15 (WMS-Bodega), VLAN 30 (Voz), VLAN 90 (Guest), VLAN 99 (Gestión).
- **Protocolos pendientes:** OSPF (área 0), HSRP en Cores, EtherChannel entre Core y Distribución.
- **Firewall:** Configurar reglas de filtrado y NAT.
- **WLC:** Configurar SSID industrial con 802.11r (roaming rápido).
- **Syslog:** Configurar envío de logs desde equipos al `SERVER-WMS`.

---

## 🔜 Próximo paso

Construcción de la **topología física de la sede Interior del país** (sucursal con enlace único + respaldo 4G/5G) y posterior interconexión WAN entre las tres sedes vía SD-WAN.

---

## 📂 Archivos relacionados
- `Topologia_Colon.pkt` — Archivo de Packet Tracer con la topología física.
- `Fase_Design_Red_Logistica.pdf` — Documento de diseño (fase Design).
