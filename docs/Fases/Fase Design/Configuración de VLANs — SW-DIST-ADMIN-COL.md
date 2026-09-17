# Configuración de VLANs — SW-DIST-ADMIN-COL

**Fase:** Implement — Etapa 4.2 (VLANs y asignación de puertos)
**Dispositivo:** `SW-DIST-ADMIN-COL`

---

## VLANs que necesita este switch

Según la tabla de reparto, `SW-DIST-ADMIN-COL` solo necesita 3 VLANs: 10 (ADMIN), 30 (VOICE) y 99 (MGMT) — a diferencia del core, que tiene las 6 completas.

## Configuración aplicada

```
enable
configure terminal

vlan 10
 name ADMIN
exit

vlan 30
 name VOICE
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

Resultado esperado: solo esas 3 VLANs (más la VLAN 1 y las reservadas por defecto) — no las 6 que tiene el core.

---

## Estado

- [ ] `SW-DIST-ADMIN-COL` — VLANs aplicadas y verificadas

## Siguiente paso

Orden que sigue en la etapa 4.2:

1. `SW-DIST-BODEGA-COL` y `SW-DIST-BODEGA-COL-2` → VLANs 15 (WMS-BODEGA), 20 (SERVERS), 99 (MGMT)
2. `SW-ACC-COL` → VLANs 10 (ADMIN), 30 (VOICE), 99 (MGMT)
3. `SW-ACC-BODEGA-COL` → VLANs 15 (WMS-BODEGA), 99 (MGMT)
