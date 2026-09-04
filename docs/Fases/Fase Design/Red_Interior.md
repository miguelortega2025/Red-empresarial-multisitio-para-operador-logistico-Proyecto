# Topología Física - Sede Interior del País

## Proyecto: Red Empresarial para Operador Logístico
## Fase: Design (PPDIOO) - Diseño de Red
## Alcance: Sede Interior del País

---

## 📌 Descripción General

Este documento describe la **topología física** necesaria para la sede del **Interior del país**, según lo definido en la Fase Design del proyecto.

La sede del Interior tiene un diseño de **Core/Distribución colapsado**, lo que significa que un solo switch L3 cumple las funciones de núcleo y distribución. Su criticidad es menor que Panamá y Colón, por lo que no cuenta con redundancia de equipos (HSRP, EtherChannel), pero sí con un respaldo de conectividad móvil 4G/5G.

---

## 🖥️ Dispositivos Necesarios

| Dispositivo | Cantidad | Descripción Técnica | Equivalente en Packet Tracer |
| :--- | :--- | :--- | :--- |
| **Switch L3 (Core/Distribución)** | 1 | Switch multicapa que actúa como gateway de la red local (enrutamiento inter-VLAN). | Cisco Catalyst 3560 o 3650 |
| **Switch L2 (Acceso)** | 1 | Switch de acceso para conectar equipos finales (PCs, impresoras, teléfonos IP). | Cisco Catalyst 2960 |
| **Router / Firewall de Sucursal** | 1 | Concentra el enlace WAN (Internet) y sirve como firewall perimetral. | Cisco 1941 / 2901 / 4331 |
| **Router 4G/5G (Respaldo)** | 1 | Router celular para respaldo por ruta estática flotante (R-06). | Cisco 1941 (simulado) |
| **Access Points (APs)** | 2-3 | Puntos de acceso inalámbrico para cobertura de oficina y área comercial (VLAN 10). | Access Point genérico PT |

---

## 🔗 Conexiones Físicas

### Diagrama de Topología
