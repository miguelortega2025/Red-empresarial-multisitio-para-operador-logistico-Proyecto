# Spanning Tree (Rapid PVST+) — Colón

**Fase:** Implement — Etapa 4.6 (Optimización de Spanning Tree)
**Dispositivos:** Los 7 switches de Colón
**Prerrequisito:** Trunks y HSRP ya configurados (ver `trunks-colon.md`, `hsrp-cores-colon.md`)
**Requisito de diseño:** R-02 (alta disponibilidad de red en Colón, Rapid PVST+ especificado en sección 5 del documento de Design)

---

## Por qué es necesario

La topología de Colón tiene redundancia física a propósito (dual-homing en acceso y distribución, full mesh entre cores y distribución), lo que crea loops físicos. En Capa 2, un loop sin control provoca broadcast storms porque Ethernet no tiene un mecanismo tipo TTL que mate el tráfico tras cierto número de saltos.

Spanning Tree bloquea lógicamente los enlaces redundantes (sin desconectarlos físicamente), dejando un único camino activo a la vez y los demás en espera, listos para activarse si el camino principal falla.

## Por qué Rapid PVST+ y no STP clásico

- **STP clásico (802.1D):** funciona, pero converge en 30-50 segundos tras un cambio — inaceptable para una sede con operación 24/7.
- **Rapid STP (802.1w):** converge en segundos.
- **PVST+:** corre una instancia de Spanning Tree independiente por VLAN, en vez de un árbol único compartido — permite que distintas VLANs usen distintos caminos como principal.
- **Rapid PVST+:** combina ambas — es el estándar de facto en redes Cisco modernas.

## Relación con HSRP

HSRP decide quién es el gateway lógico (a qué IP responden los dispositivos). Spanning Tree decide qué camino físico toma el tráfico para llegar a ese switch. Si no se alinean, el tráfico puede tomar una ruta física distinta a la que HSRP considera "activa", generando ineficiencia y dificultando el troubleshooting.

Por eso `SW-CORE-COL` (Active de HSRP en las 6 VLANs) se configuró también como Root Bridge primario en las 6 VLANs — mismo switch como "centro" tanto en Capa 3 (gateway) como en Capa 2 (árbol físico).

---

## Parte 1 — Modo Rapid PVST+ global

Aplicado igual en los 7 switches:

```
enable
configure terminal

spanning-tree mode rapid-pvst

end
write memory
```

**Verificación:**
```
show spanning-tree summary
```
Confirma `Switch is in rapid-pvst mode`.

---

## Parte 2 — Root Bridge primario y secundario

**En `SW-CORE-COL` (root primario — mismo switch que Active en HSRP):**
```
enable
configure terminal

spanning-tree vlan 10,15,20,30,90,99 root primary

end
write memory
```

**En `SW-CORE-COL-2` (root secundario):**
```
enable
configure terminal

spanning-tree vlan 10,15,20,30,90,99 root secondary

end
write memory
```

**Qué hace por debajo:** `root primary` asigna automáticamente una prioridad baja (24576 + ID de VLAN) a `SW-CORE-COL`, garantizando que gane la elección de root sin calcular el valor manualmente. `root secondary` asigna una prioridad ligeramente más alta (28672 + ID de VLAN), suficiente para ser el segundo en la fila sin ganar mientras el primario esté disponible.

**Verificación:**
```
show spanning-tree summary
show spanning-tree vlan 10
```
Confirmar la línea `This bridge is the root` en `SW-CORE-COL`.

**Resultado verificado (ejemplo con `show spanning-tree interface F0/3 detail` en `SW-CORE-COL-2`):**
- VLAN 15 — root con prioridad 24591 (24576 + 15) ✅
- VLAN 20 — root con prioridad 24596 (24576 + 20), puerto en `alternate blocking` (confirma que STP detectó el camino redundante y lo bloqueó correctamente) ✅
- VLAN 99 — root con prioridad 24675 (24576 + 99) ✅
- Designated bridge en los tres casos con prioridad base 28672 → confirma a `SW-CORE-COL-2` como secundario correctamente alineado ✅

---

## Parte 3 — BPDU Guard en puertos de acceso

**Qué es:** protección para que un puerto de acceso (donde solo debería haber un dispositivo final) se apague automáticamente (`err-disabled`) si recibe un BPDU — señal de que alguien conectó un switch no autorizado a ese puerto. Se aplica junto con `portfast`, nunca en puertos trunk entre switches (ahí los BPDUs son tráfico normal y esperado).

**Puertos donde se aplicó:**

```
enable
configure terminal

interface FastEthernet0/X
 spanning-tree bpduguard enable
exit

end
write memory
```

| Switch | Puerto | Dispositivo |
|---|---|---|
| SW-ACC-COL | Fa0/3 | PHONE-1 |
| SW-ACC-BODEGA-COL | Fa0/1 | AP-COL-2 |
| SW-ACC-BODEGA-COL | Fa0/2 | AP-COL-1 |
| SW-DIST-BODEGA-COL | F0/3 | SERVER-WMS |
| SW-DIST-BODEGA-COL-2 | F0/4 | SERVER-WMS |

**Alternativa más rápida para el futuro** (no usada aquí, pero documentada como referencia): un solo comando global aplica BPDU Guard a todo puerto con `portfast`:
```
spanning-tree portfast bpduguard default
```

**Verificación:**
```
show spanning-tree interface FastEthernet0/X detail
```
Buscar `Bpdu guard is enabled`.

---

## Incidencia encontrada durante la verificación **[abierta]**

Al verificar BPDU Guard en `SW-DIST-BODEGA-COL` F0/3, el `show spanning-tree interface FastEthernet0/3 detail` mostraba la interfaz participando solo en `VLAN0001` en vez de `VLAN0020`.

**Causa raíz:** el puerto nunca tuvo aplicado `switchport mode access` ni `switchport access vlan 20` — solo tenía el comando de BPDU Guard. Quedó en la VLAN 1 por defecto desde la etapa de asignación de puertos de acceso (varias etapas atrás), y el error no se detectó hasta esta verificación de Spanning Tree.

**Por qué no se notó antes:** BPDU Guard se aplicó correctamente sobre el puerto, así que el comando en sí no falló — pero como el puerto seguía en VLAN 1, buscar su estado en la instancia de VLAN 20 de Spanning Tree no mostraba nada, lo que generó la confusión inicial de "no aparece BPDU Guard" cuando en realidad el problema era la VLAN, no el BPDU Guard.

**Solución aplicada (a este puerto):**
```
enable
configure terminal

interface FastEthernet0/3
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
exit

end
write memory
```

**Pendiente:** verificar si el mismo problema (falta de `switchport mode access` / `switchport access vlan`) se repite en los otros 4 puertos de la tabla de BPDU Guard (`PHONE-1`, los 2 APs, y el segundo homing de `SERVER-WMS` en `SW-DIST-BODEGA-COL-2`). No confirmado todavía — queda como tarea para la siguiente sesión.

---

## Estado

- [x] Parte 1 — Rapid PVST+ global en los 7 switches
- [x] Parte 2 — Root primario/secundario en ambos cores, verificado
- [x] Parte 3 — BPDU Guard aplicado en los 5 puertos de acceso
- [ ] **Pendiente:** verificar `switchport access vlan` en los otros 4 puertos (solo confirmado y corregido en `SW-DIST-BODEGA-COL` F0/3)

## Siguiente paso

Etapa 4.8 en adelante — DHCP local por VLAN, y luego OSPF hacia Panamá.
