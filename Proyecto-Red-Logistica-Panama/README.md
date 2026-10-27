# Red Corporativa Multisitio – Operador Logístico (Panamá)

Proyecto de portafolio en **Cisco Packet Tracer** | Metodología **Cisco PPDIOO** | Preparación **CCNA 200-301**

---

## Resumen

Este proyecto simula la infraestructura de red de una empresa logística ficticia con presencia en tres puntos estratégicos de Panamá: una sede corporativa en **Ciudad de Panamá**, una bodega en la **Zona Libre de Colón** y una sucursal comercial en el **interior del país**.

El objetivo es aplicar de forma integrada los conceptos del temario CCNA en un escenario realista, en lugar de practicarlos como labs aislados: múltiples sitios, alta disponibilidad, segmentación por departamento y administración centralizada y segura.

> **Nota:** es una simulación en Packet Tracer. Algunos elementos del negocio (Wi-Fi industrial, lectores RFID, enlaces reales de ISP) se representan de forma aproximada con los dispositivos genéricos del simulador.

## Motivación

Panamá basa buena parte de su economía en su posición logística (Canal, puertos, Zona Libre de Colón), y el comercio y la logística se mantienen entre los sectores con mayor demanda de contratación, junto con finanzas y tecnologías de la información. El proyecto forma parte de mi preparación para la certificación Cisco CCNA y sirve como pieza de portafolio para posiciones de soporte de redes / ingeniería de redes junior en el mercado panameño.

## Resumen ejecutivo (fase Prepare)

| Pregunta | Respuesta |
|---|---|
| ¿Qué empresa es y dónde opera? | Operador logístico panameño con sedes en Ciudad de Panamá, Colón (Zona Libre) e Interior |
| ¿Por qué necesita una red? | Es una empresa nueva sin infraestructura; necesita operar WMS, ERP y comunicación entre sedes desde el primer día |
| ¿Qué se justifica invertir? | Una red pensada desde el inicio para las tres sedes, evitando sobrecostos futuros por "parchar" la infraestructura |
| ¿Cuál es la prioridad #1? | Continuidad operativa: alta disponibilidad, especialmente entre Panamá y Colón |

**El negocio en una línea:** la mercancía entra por Colón, donde el **WMS** da la trazabilidad (la operación es 24/7); la facturación y la coordinación del transporte dependen del **ERP** en Ciudad de Panamá; la sucursal del interior atiende comercios locales.

## Topología

![Topología general](03_Disenar/Imagenes/topologia_general.png)
<!-- COMPLETAR: agregar la imagen de la topología general -->

| Sede | Diagrama |
|---|---|
| Ciudad de Panamá | [Ver diagrama](03_Disenar/Imagenes/sede_panama.png) |
| Colón (Zona Libre) | [Ver diagrama](03_Disenar/Imagenes/sede_colon.png) |
| Interior del país | [Ver diagrama](03_Disenar/Imagenes/sede_interior.png) |

## Sedes

| Sede | Rol en el negocio | Usuarios aprox. | Criticidad | Diseño |
|---|---|---|---|---|
| **Ciudad de Panamá** | Centro corporativo: dirección, finanzas, RRHH, ventas, TI. Aloja el ERP | ~50 | Alta | Núcleo y distribución duplicados, doble firewall, doble ISP, controlador WLC |
| **Colón (Zona Libre)** | Operación logística 24/7: bodega y mercancía de alto valor. Aloja el WMS | ~30 | **Máxima** | Núcleo y distribución duplicados, doble firewall, doble ISP, APs |
| **Interior** | Sucursal comercial de menor volumen | ~10 | Media | Núcleo único, respaldo de WAN por 4G |

## Objetivos técnicos

- Diseñar una topología multisitio con conectividad WAN entre sedes
- Segmentar el tráfico por departamento mediante VLANs
- Implementar enrutamiento entre VLANs y entre sitios con OSPF
- Garantizar redundancia de gateway en el sitio principal (HSRP)
- Centralizar servicios de red: DHCP, NTP, Syslog
- Aplicar seguridad básica: ACLs, SSH, port security
- Documentar el diseño de forma clara y reproducible

Elementos adicionales presentes en la topología:

- Firewalls perimetrales con NAT en las sedes principales
- Doble proveedor de Internet en Panamá y Colón
- Red inalámbrica con controlador WLC y puntos de acceso
- Respaldo de conexión 4G en la sede del interior
- Redundancia en capa 2 (STP / EtherChannel) <!-- COMPLETAR: confirmar qué se implementa -->

## Interconexión entre sedes

<!-- COMPLETAR: indicar cómo se unen las sedes (túneles GRE/IPsec sobre los ISP, VPN site-to-site, etc.) y enlazar a 03_Disenar/Decisiones_Diseno.md -->

## Metodología PPDIOO

El proyecto sigue el ciclo de vida Cisco PPDIOO. Cada fase tiene su carpeta:

| Fase | Contenido | Carpeta | Estado |
|---|---|---|---|
| **Prepare** | Necesidad y caso de negocio, supuestos, tecnologías candidatas | [01_Preparar](01_Preparar/) | Completa |
| **Plan** | Requerimientos, direccionamiento, inventario de equipos | [02_Planificar](02_Planificar/) | <!-- COMPLETAR --> |
| **Design** | Diseño lógico, topología, decisiones de diseño | [03_Disenar](03_Disenar/) | <!-- COMPLETAR --> |
| **Implement** | Configuraciones por sede | [04_Implementar](04_Implementar/) | <!-- COMPLETAR --> |
| **Operate** | Pruebas de conectividad, redundancia, fallos y seguridad | [05_Operar](05_Operar/) | <!-- COMPLETAR --> |
| **Optimize** | Hallazgos, mejoras propuestas, lecciones aprendidas | [06_Optimizar](06_Optimizar/) | <!-- COMPLETAR --> |

PPDIOO es iterativo: los cambios de diseño surgidos de las pruebas se documentan en el registro de cambios de cada fase. Al tratarse de una simulación, las fases *Operate* y *Optimize* son parciales (verificación y monitoreo simulado, y mejoras propuestas a partir de las pruebas de fallo).

## Estructura del repositorio

```
Proyecto-Red-Logistica-Panama/
├── README.md
├── 01_Preparar/          Contexto de negocio, supuestos y riesgos
├── 02_Planificar/        Requerimientos, direccionamiento, inventario
├── 03_Disenar/           Diseño lógico, decisiones de diseño, imágenes
├── 04_Implementar/       Configuraciones por sede (PTY, COL, INT, Servicios)
├── 05_Operar/            Pruebas y evidencias
├── 06_Optimizar/         Mejoras y lecciones aprendidas
└── Archivos_PacketTracer/
```

## Cómo reproducir

1. Descarga el archivo `.pkt` desde [Archivos_PacketTracer](Archivos_PacketTracer/) y ábrelo con Cisco Packet Tracer <!-- COMPLETAR: versión usada -->.
2. Revisa la documentación en orden, de `01_Preparar` a `06_Optimizar`.
3. Compara las configuraciones de `04_Implementar` con los dispositivos del simulador.
4. Repite las pruebas de `05_Operar` para validar conectividad, redundancia y seguridad.

## Tecnologías y conceptos aplicados

VLANs · Trunking · STP · EtherChannel · HSRP · OSPF · NAT · ACLs · SSH · Port Security · DHCP · NTP · Syslog · WLAN/WLC · Firewalls · Respaldo WAN 4G

## Autor

Miguel Ortega, LinkedIn: https://www.linkedin.com/in/miguel-ortega-b36414287/ , correo: miguel.ortega13@outlook.com
