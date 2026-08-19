# Decisiones de diseño

Este documento registra el razonamiento detrás de las decisiones de diseño del proyecto, más allá de lo que se muestra en el README. La idea es que cualquier persona que revise el repositorio entienda no solo *qué* se construyó, sino *por qué* se construyó así.

---

## 1. Dimensionamiento de la Sede Central (Ciudad de Panamá)

### Contexto de negocio

Las empresas de logística medianas en Panamá suelen tener entre 51 y 200 empleados. Sin embargo, en una empresa de este sector, gran parte de la fuerza operativa está en la bodega/puerto (Colón, Zona Libre), no en la sede administrativa. Por esa razón, la Sede Central se modeló como una operación de oficina más pequeña: aproximadamente 40-50 personas, distribuidas en un edificio de 4 pisos.

### Estructura por piso

| Piso | Departamento | VLAN | Justificación |
|------|-------------|------|----------------|
| 1 | Ventas/Comercial + Invitados | 20, 99 | Áreas con más contacto con el público; recepción y sala de espera comparten el piso de entrada |
| 2 | Administración/Gerencia | 10 | Dirección general, RRHH, finanzas — separado de las áreas operativas |
| 3 | Operaciones | 30 | Coordinación de flotas, tracking de carga, enlace directo con Colón e Interior — es el corazón operativo de la empresa |
| 4 | IT/Servidores | 40 | Cuarto de servidores con acceso restringido; ubicado en el piso más alto por seguridad física |

### Por qué no modelar 200 dispositivos

El objetivo del proyecto es demostrar dominio de VLANs, trunking (802.1Q), inter-VLAN routing, Spanning Tree, DHCP y OSPF — no simular una empresa real dispositivo por dispositivo. Modelar 150-200 endpoints en Packet Tracer no aportaría más valor técnico, solo ruido visual y un archivo más pesado de mantener. Con 8-10 dispositivos finales (PCs, IP Phones, servidor) por sitio ya se cubre la lógica de segmentación completa de forma clara y revisable.

### Departamentos definidos y su rol

- **Administración/Gerencia** (VLAN 10): dirección, RRHH, contabilidad, finanzas.
- **Ventas/Comercial** (VLAN 20): atención a clientes, cotizaciones, cuentas corporativas.
- **Operaciones** (VLAN 30): coordinación de flotas, planificación de rutas, comunicación con Colón e Interior.
- **IT/Servidores** (VLAN 40): soporte técnico y alojamiento de los servicios centralizados (DHCP, NTP, Syslog).
- **Invitados** (VLAN 99): red aislada para visitas, auditores o clientes que llegan a oficina, sin acceso a la red interna.

---

## 2. Jerarquía física: distribución y acceso

La Sede Central sigue un modelo de dos capas dentro del propio sitio:

- **SW1 y SW2** operan como capa de distribución, con redundancia hacia R1 y HSRP configurado entre ambos.
- **Switches de acceso por piso** (uno por cada uno de los 4 pisos) cuelgan de la distribución y llevan un enlace trunk con las VLANs correspondientes a los departamentos de ese piso.

Este diseño refleja cómo se cablea normalmente un edificio de oficinas: cada piso tiene su propio switch de acceso, y las políticas de VLAN/ACL se aplican por función (departamento), no por ubicación física. Esto evita tener que rediseñar la red si un departamento cambia de piso.

---

## 3. Loops físicos intencionales (S3-S4-S5 / S6-S7-S8)

Los enlaces triangulares entre los switches de acceso de un mismo lado (por ejemplo S3-S4-S5) se diseñaron a propósito como loops físicos redundantes. El objetivo es demostrar la convergencia de **PVST+ (Spanning Tree)**: Packet Tracer bloquea automáticamente uno de los tres enlaces para evitar un loop de capa 2, lo cual sirve como evidencia funcional de que STP está operando correctamente en la topología.

---

## 4. Estado: pendiente de definir

- Dimensionamiento equivalente para Colón (Zona Libre) e Interior.
- Cantidad final de VLANs de voz (IP Phones) por sitio.
- Política de ACL entre VLAN de Invitados y el resto de la red.
