# Configuración de VLANs — Switches Core (Colón)

**Fase:** Implement — Etapa 4.2 (VLANs y asignación de puertos)
**Dispositivos:** `SW-CORE-COL`, `SW-CORE-COL-2`

---

## Criterio de reparto de VLANs por switch

No declaro todas las VLANs en todos los switches por defecto — solo donde el switch tiene un puerto de acceso en esa VLAN, o donde un trunk necesita transportarla hacia otro switch que sí la usa.

Los dos cores son la excepción: ahí van a vivir las SVIs (gateways) de cada VLAN, así que necesitan conocer las **6 VLANs completas** para poder enrutar entre ellas y para que HSRP funcione entre ambos más adelante.

## Configuración aplicada

Mismo bloque en ambos cores (`SW-CORE-COL` y `SW-CORE-COL-2`):

```
enable
configure terminal

vlan 10
 name ADMIN
exit

vlan 15
 name WMS-BODEGA
exit

vlan 20
 name SERVERS
exit

vlan 30
 name VOICE
exit

vlan 90
 name GUEST
exit

vlan 99
 name MGMT
exit

end
write memory
```

## Verificación

```
show vlan brief
```

Resultado esperado: tabla con las 6 VLANs (10, 15, 20, 30, 90, 99), cada una con su nombre correcto, todas en estado `active`.

**Notas sobre el resultado esperado:**

- Todavía no debe aparecer ningún puerto asignado a ninguna VLAN — normal y correcto, porque el core no tiene dispositivos finales conectados directamente, solo trunks hacia distribución (pendiente) y las SVIs (etapa 4.4, más adelante).
- El switch ya trae por defecto VLAN 1 (default) y las VLANs de rango 1002–1005 reservadas para protocolos legacy (Token Ring/FDDI). No las toco — es normal que aparezcan junto a las mías en el `show vlan brief`.

---

## Reparto de VLANs por switch (referencia para las siguientes etapas)

| Switch | VLANs que necesita | Por qué |
|---|---|---|
| `SW-CORE-COL` / `SW-CORE-COL-2` | 10, 15, 20, 30, 90, 99 — todas | Aquí viven las SVIs (gateways) de cada VLAN |
| `SW-DIST-ADMIN-COL` | 10, 30, 99 | Transporta tráfico admin y voz hacia `SW-ACC-COL` |
| `SW-DIST-BODEGA-COL` / `-COL-2` | 15, 20, 99 | Transporta bodega (APs) y aloja `SERVER-WMS` |
| `SW-ACC-COL` | 10, 30, 99 | Tiene conectado `PHONE-1` y potencialmente PCs admin |
| `SW-ACC-BODEGA-COL` | 15, 99 | Tiene conectados los APs de bodega |

**Pendiente de decidir:** VLAN 90 (GUEST) está declarada en el core por si se necesita después, pero todavía no tiene ningún puerto de acceso asignado en ningún switch de la topología actual. Falta decidir si se le asigna un puerto físico de invitados/contratistas o se deja solo declarada.

---

## Estado

- [ ] `SW-CORE-COL` — VLANs aplicadas y verificadas
- [ ] `SW-CORE-COL-2` — VLANs aplicadas y verificadas

## Siguiente paso

`SW-DIST-ADMIN-COL` — VLANs 10 (ADMIN), 30 (VOICE), 99 (MGMT).
