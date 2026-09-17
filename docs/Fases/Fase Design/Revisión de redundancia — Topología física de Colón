# Revisión de redundancia — Topología física de Colón

**Fase:** Implement — Validación de diseño físico
**Sede:** Colón (Zona Libre)
**Estado:** Aprobada, con una observación menor pendiente de decisión

---

## Resumen de verificación

| # | Punto verificado | Estado |
|---|---|---|
| 1 | `SW-ACC-BODEGA-COL` dual-homed a `SW-DIST-BODEGA-COL` y `SW-DIST-BODEGA-COL-2` | ✅ Correcto |
| 2 | `SW-DIST-BODEGA-COL-2` conectado a ambos cores (full mesh distribución-core) | ✅ Correcto |
| 3 | Modelo jerárquico de tres capas respetado, sin saltos acceso→core | ✅ Correcto |
| 4 | Core, firewalls, ISPs y sync entre firewalls | ✅ Correcto (sin cambios respecto a versión anterior) |

### Detalle

- **`SW-ACC-BODEGA-COL`** ahora tiene doble camino hacia distribución — sobrevive a la caída de cualquiera de los dos switches de distribución de bodega.
- **`SW-DIST-BODEGA-COL-2`** replica el mismo nivel de redundancia que ya tenía la distribución de admin, manteniendo el full mesh distribución-core.
- Ningún dispositivo de acceso salta directo a core — se mantiene la separación de capas definida en la fase Design (sección 2 del documento de diseño).

---

## Inconsistencia menor identificada

**`SERVER-WMS`** permanece conectado únicamente a `SW-DIST-BODEGA-COL` (single-homed), mientras que el resto de la infraestructura de bodega (switch de acceso) ya quedó con doble camino tras esta revisión.

### Por qué importa

El WMS es el sistema que corre la operación de bodega 24/7 — tiene la misma lógica de criticidad que justificó el dual-homing de los APs y del switch de acceso.

### Opciones

**A. Dejarlo single-homed.**
Válido si se asume que el servidor en sí no es el punto único de falla que preocupa (por ejemplo, si existe un segundo servidor WMS redundante a nivel de aplicación, fuera del alcance de este diagrama de red).

**B. Dual-homing del servidor.**
```
SERVER-WMS <-> SW-DIST-BODEGA-COL
SERVER-WMS <-> SW-DIST-BODEGA-COL-2
```
Requiere que el servidor tenga doble tarjeta de red (NIC teaming / bonding) para aprovechar ambos enlaces. Es coherente con el nivel de redundancia ya aplicado al resto del diseño de bodega.

### Decisión

*(Pendiente de registrar — completar cuando se defina)*

- [ ] Opción A — single-homed, justificación: ___________
- [ ] Opción B — dual-homed con NIC teaming

---

## Estado final de la topología de Colón

Fuera del punto anterior, la topología física de Colón queda **aprobada** para pasar a la fase de configuración lógica (VLANs, direccionamiento IP, HSRP, OSPF).
