# HSRP — Cores de Colón

**Fase:** Implement — Etapa 4.5 (Alta disponibilidad de gateway)
**Dispositivos:** `SW-CORE-COL` (primario), `SW-CORE-COL-2` (respaldo)
**Prerrequisito:** SVIs ya configuradas en ambos cores (ver `svis-cores-colon.md`)
**Requisito de diseño:** R-02 (alta disponibilidad de red en Colón)

---

## Concepto

HSRP agrega una tercera IP a cada VLAN — la IP virtual (`.1`) — que no vive fija en ningún core, sino que "flota" entre los dos. Esa `.1` es el gateway real que usan los dispositivos finales. HSRP se encarga de que esa IP siempre esté respondida por el core que esté activo en ese momento; si el primario cae, el respaldo toma el control automáticamente sin que los dispositivos finales lo perciban.

## Decisión de diseño

`SW-CORE-COL` se definió como primario en las 6 VLANs (prioridad 110), `SW-CORE-COL-2` como respaldo (prioridad por defecto, 100). Se eligió consistencia en un solo primario para simplificar el diseño y el troubleshooting, en vez de repartir el rol primario entre VLANs distintas.

## Prioridad vs. Preempt

- **`priority`** — responde "¿quién debería ser el activo cuando ambos estén disponibles?". Es la preferencia declarada.
- **`preempt`** — responde "si no soy el activo ahora pero mi prioridad es más alta que la del actual, ¿reclamo el puesto automáticamente?". Es la acción de recuperar el control.

**Por qué preempt es necesario en el primario:** sin él, el router activo se decide solo por quién ganó la elección inicial al arrancar, no por prioridad. Si `SW-CORE-COL` cae y `SW-CORE-COL-2` toma el control, al volver `SW-CORE-COL` no reclama el puesto automáticamente a menos que tenga `preempt` — se queda como standby pasivo indefinidamente pese a tener mayor prioridad.

**Por qué preempt también se deja en el respaldo:** en este escenario, `preempt` en `SW-CORE-COL-2` no actúa (su prioridad nunca supera a la del primario bajo condiciones normales), pero se deja por consistencia y por si en el futuro se agrega `standby track` sobre una interfaz (ej. el enlace WAN), lo que podría bajar dinámicamente la prioridad del primario y hacer que el respaldo sí necesite reclamar el puesto.

## Por qué se repite el bloque completo en cada VLAN

HSRP no es una elección "por switch" — es una elección por SVI (por VLAN), independiente una de otra. Cada `interface VlanXX` corre su propia instancia de HSRP, con su propio grupo, su propia elección y su propio estado.

---

## Configuración aplicada por VLAN

### VLAN 10 (ADMIN) — virtual 10.20.10.1

**`SW-CORE-COL` (primario):**
```
enable
configure terminal

interface Vlan10
 standby 10 ip 10.20.10.1
 standby 10 priority 110
 standby 10 preempt
exit

end
write memory
```

**`SW-CORE-COL-2` (respaldo):**
```
enable
configure terminal

interface Vlan10
 standby 10 ip 10.20.10.1
 standby 10 preempt
exit

end
write memory
```

### VLAN 15 (WMS-BODEGA) — virtual 10.20.15.1

**`SW-CORE-COL` (primario):**
```
enable
configure terminal

interface Vlan15
 standby 15 ip 10.20.15.1
 standby 15 priority 110
 standby 15 preempt
exit

end
write memory
```

**`SW-CORE-COL-2` (respaldo):**
```
enable
configure terminal

interface Vlan15
 standby 15 ip 10.20.15.1
 standby 15 preempt
exit

end
write memory
```

Nota: es la SVI más sensible en cuanto a prueba de failover, porque da servicio a los APs y lectores RFID de la operación crítica 24/7 de bodega.

### VLAN 20 (SERVERS) — virtual 10.20.20.1

**`SW-CORE-COL` (primario):**
```
enable
configure terminal

interface Vlan20
 standby 20 ip 10.20.20.1
 standby 20 priority 110
 standby 20 preempt
exit

end
write memory
```

**`SW-CORE-COL-2` (respaldo):**
```
enable
configure terminal

interface Vlan20
 standby 20 ip 10.20.20.1
 standby 20 preempt
exit

end
write memory
```

Nota: `SERVER-WMS` vive en esta VLAN y está dual-homed. Al probar failover, verificar que el servidor sigue alcanzable desde otras VLANs con el core activo caído.

### VLAN 30 (VOICE) — virtual 10.20.30.1

**`SW-CORE-COL` (primario):**
```
enable
configure terminal

interface Vlan30
 standby 30 ip 10.20.30.1
 standby 30 priority 110
 standby 30 preempt
exit

end
write memory
```

**`SW-CORE-COL-2` (respaldo):**
```
enable
configure terminal

interface Vlan30
 standby 30 ip 10.20.30.1
 standby 30 preempt
exit

end
write memory
```

### VLAN 90 (GUEST) — virtual 10.20.90.1

**`SW-CORE-COL` (primario):**
```
enable
configure terminal

interface Vlan90
 standby 90 ip 10.20.90.1
 standby 90 priority 110
 standby 90 preempt
exit

end
write memory
```

**`SW-CORE-COL-2` (respaldo):**
```
enable
configure terminal

interface Vlan90
 standby 90 ip 10.20.90.1
 standby 90 preempt
exit

end
write memory
```

Nota: VLAN aún sin puerto de acceso asignado (ver decisión pendiente en `revision-topologia-colon.md`). Ya tiene gateway funcional; queda lista para operar en cuanto se le asigne un puerto físico.

### VLAN 99 (MGMT) — virtual 10.20.99.1

**`SW-CORE-COL` (primario):**
```
enable
configure terminal

interface Vlan99
 standby 99 ip 10.20.99.1
 standby 99 priority 110
 standby 99 preempt
exit

end
write memory
```

**`SW-CORE-COL-2` (respaldo):**
```
enable
configure terminal

interface Vlan99
 standby 99 ip 10.20.99.1
 standby 99 preempt
exit

end
write memory
```

Nota: VLAN usada para la gestión remota de switches, WLC y firewall. La resiliencia de esta VLAN es lo que permite seguir gestionando la red a través de `10.20.99.1` aunque el core primario caiga.

---

## Verificación

```
show standby brief
```
Confirma estado (`Active` / `Standby`) e IP virtual de cada grupo.

```
show standby
```
Vista detallada — confirma que `preempt` está habilitado en ambos lados de cada VLAN.

**Resultado esperado:** las 6 VLANs con `SW-CORE-COL` en `Active` y `SW-CORE-COL-2` en `Standby`, cada una apuntando a su IP virtual `.1`.

## Prueba de failover realizada

Se apagó `SW-CORE-COL` y se verificó que `SW-CORE-COL-2` pasó a `Active` en las 6 VLANs. Al reencender `SW-CORE-COL`, retomó el rol de `Active` automáticamente gracias a `preempt`, sin intervención manual.

**Resultado:** ✅ Exitoso.

---

## Resumen de IPs virtuales (gateways)

| VLAN | Nombre | IP virtual (gateway) | Primario | Respaldo |
|---|---|---|---|---|
| 10 | ADMIN | 10.20.10.1 | SW-CORE-COL | SW-CORE-COL-2 |
| 15 | WMS-BODEGA | 10.20.15.1 | SW-CORE-COL | SW-CORE-COL-2 |
| 20 | SERVERS | 10.20.20.1 | SW-CORE-COL | SW-CORE-COL-2 |
| 30 | VOICE | 10.20.30.1 | SW-CORE-COL | SW-CORE-COL-2 |
| 90 | GUEST | 10.20.90.1 | SW-CORE-COL | SW-CORE-COL-2 |
| 99 | MGMT | 10.20.99.1 | SW-CORE-COL | SW-CORE-COL-2 |

---

## Estado

- [x] VLAN 10 — HSRP configurado y verificado
- [x] VLAN 15 — HSRP configurado y verificado
- [x] VLAN 20 — HSRP configurado y verificado
- [x] VLAN 30 — HSRP configurado y verificado
- [x] VLAN 90 — HSRP configurado y verificado
- [x] VLAN 99 — HSRP configurado y verificado
- [x] Prueba de failover (apagar/encender core primario) — exitosa

## Siguiente paso

Etapa 4.6 en adelante — Spanning Tree (Rapid PVST+), DHCP, y enrutamiento OSPF hacia Panamá.
