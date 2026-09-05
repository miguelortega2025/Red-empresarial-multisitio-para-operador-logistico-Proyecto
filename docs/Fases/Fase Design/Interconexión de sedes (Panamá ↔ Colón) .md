# Changelog de diseño — Red Logística

## [v1.1] - Interconexión de sedes (Panamá ↔ Colón) + firewall redundante en Panamá

### Objetivo principal de este cambio

Hasta la versión anterior, las sedes de Ciudad de Panamá y Colón estaban diseñadas
como topologías independientes: cada una terminaba en sus propios routers de ISP sin
ningún enlace que las uniera. Este cambio cierra esa brecha e implementa la
conectividad WAN entre ambas sedes, tal como exige la sección 6 del documento de
Design (doble transporte, failover automático) y el requisito **R-01** de la matriz
de trazabilidad. Como consecuencia directa de dividir la salida a Internet en dos
transportes independientes, también se corrigió la redundancia de firewall en
Panamá (ver segunda sección de este changelog).

### 1. Enlace físico entre sedes (doble transporte WAN)

**Agregado**
- Enlace serial `ISP-A` (Panamá) ↔ `ISP-A-COL` (Colón) — Transporte 1.
- Enlace serial `ISP-B` (Panamá) ↔ `ISP-B-COL` (Colón) — Transporte 2.

**Direccionamiento**

| Enlace | Subred |
|---|---|
| `ISP-A` ↔ `ISP-A-COL` | `172.16.0.0/30` |
| `ISP-B` ↔ `ISP-B-COL` | `172.16.0.4/30` |

**Topología resultante (doble transporte)**

<img width="1803" height="586" alt="image" src="https://github.com/user-attachments/assets/30b2df13-0dd7-45be-94e6-7ec34b5e2a74" />


Con esto, si cae el Transporte 1, el tráfico entre sedes sigue cursando por el
Transporte 2, sin dejar ninguna sede aislada — cumpliendo el mecanismo de "failover
automático entre transportes" descrito en la sección 6 del documento.

**Pendiente para que la unión sea funcional (no solo física)**
- [ ] Levantar túneles GRE/IPsec sobre ambos transportes (entre `FW-PTY`/`FW-PTY-2`
      y `FW-COL`/`FW-COL-2`), ya que el tráfico interno no debe cursar directo sobre
      los enlaces públicos de ISP.
- [ ] Configurar OSPF área 0 únicamente sobre las interfaces tunnel.
- [ ] Configurar resumen de ruta por sede (`10.10.0.0/16` desde Panamá,
      `10.20.0.0/16` desde Colón) para que cada lado reciba una sola ruta agregada.

### 2. Firewall perimetral redundante en Ciudad de Panamá

#### Contexto

Al dividir la salida a Internet de Panamá en dos transportes independientes (punto 1),
quedó expuesta una asimetría: la fase Design (sección 5, tabla de Alta disponibilidad)
especifica **par de firewall activo/pasivo con sincronización de estado** para Panamá
y Colón, en cumplimiento del requisito **R-07**. La topología inicial de Panamá tenía
un único firewall (`FW-PTY`) recibiendo ambos ISP y ambos switches core, lo cual no
cumplía ese requisito. Colón sí lo cumplía desde el diseño original (`FW-COL` /
`FW-COL-2`).

Este cambio corrige la asimetría, replicando en Panamá el mismo patrón que ya existía
en Colón.

#### Cambios realizados

**Agregado**
- Nuevo firewall `FW-PTY-2` en la sede Ciudad de Panamá.

**Recableado**
| Enlace anterior | Enlace nuevo |
|---|---|
| `ISP-A` — `FW-PTY` | `ISP-A` — `FW-PTY-2` |
| `FW-PTY` — `SW-CORE-PTY-2` | `FW-PTY-2` — `SW-CORE-PTY-2` |
| `FW-PTY` — `SW-CORE-PTY-1` | *(sin cambio)* |
| `ISP-B` — `FW-PTY` | *(sin cambio)* |

**Nuevo enlace de sincronización (failover)**
- `FW-PTY` ↔ `FW-PTY-2`, subred dedicada `172.16.1.0/30`.

#### Topología resultante — Panamá (firewall)

```
ISP-A ── FW-PTY-2 ── SW-CORE-PTY-2
              |
      (enlace failover, 172.16.1.0/30)
              |
ISP-B ── FW-PTY ──── SW-CORE-PTY-1
```

Cada firewall queda con exactamente tres conexiones: una hacia su ISP (outside),
una hacia su core (inside), y una hacia el firewall par (failover). Ningún firewall
queda conectado a los dos cores simultáneamente.

#### Direccionamiento agregado (failover firewall)

| Enlace | Subred | Interfaz |
|---|---|---|
| Failover `FW-PTY` ↔ `FW-PTY-2` | `172.16.1.0/30` | `FW-PTY`: `172.16.1.1/30` · `FW-PTY-2`: `172.16.1.2/30` |

> Nota: esta subred es distinta de `172.16.0.0/24`, reservada para los enlaces WAN
> punto a punto entre sedes (sección 3.1 del documento de Design).

### Impacto en la matriz de trazabilidad (sección 9)

| ID | Requisito | Estado antes | Estado después |
|---|---|---|---|
| R-01 | Conectividad WAN entre las tres sedes | Parcial — Panamá y Colón sin enlace físico | Cumplido — doble transporte Panamá ↔ Colón |
| R-07 | Firewall perimetral redundante (Panamá y Colón) | Parcial — solo Colón cumplía | Cumplido en ambas sedes |

### Limitación conocida (entorno de simulación)

Packet Tracer no simula failover activo/pasivo real (no replica estado de conexiones
ni hace preemption automático). La topología física redundante queda correctamente
representada, pero:
- Las reglas de NAT/ACL deben configurarse manualmente e idénticas en ambos firewalls.
- La demostración de "failover" se hace apagando un firewall y verificando que el
  otro mantiene la conectividad, no como una conmutación transparente de sesiones.

### Pendiente / próximos pasos

- [ ] Configurar NAT/ACL idénticas en `FW-PTY` y `FW-PTY-2`.
- [ ] Levantar túneles GRE/IPsec sobre los dos transportes WAN (Panamá ↔ Colón).
- [ ] Configurar OSPF área 0 sobre las interfaces tunnel, con resumen de ruta por sede.
- [ ] Verificar en Colón si el enlace de sincronización `FW-COL` ↔ `FW-COL-2` ya existe;
      si no, agregarlo siguiendo este mismo patrón.
