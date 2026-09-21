# Configuración de VLANs — Distribución de bodega y acceso (Colón)

**Fase:** Implement — Etapa 4.2 (VLANs y asignación de puertos)
**Dispositivos:** `SW-DIST-BODEGA-COL`, `SW-DIST-BODEGA-COL-2`, `SW-ACC-COL`, `SW-ACC-BODEGA-COL`

---

## SW-DIST-BODEGA-COL / SW-DIST-BODEGA-COL-2

Mismo bloque en ambos, porque cumplen el mismo rol (distribución de bodega, en dual-homing entre sí hacia el acceso).

**VLANs: 15 (WMS-BODEGA), 20 (SERVERS), 99 (MGMT)**

```
enable
configure terminal

vlan 15
 name WMS-BODEGA
exit

vlan 20
 name SERVERS
exit

vlan 99
 name MGMT
exit

end
write memory
```

**Verificación:** `show vlan brief` en cada uno — deben aparecer esas 3 VLANs en ambos.

---

## SW-ACC-COL

**VLANs: 10 (ADMIN), 30 (VOICE), 99 (MGMT)**

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

---

## SW-ACC-BODEGA-COL

**VLANs: 15 (WMS-BODEGA), 99 (MGMT)**

```
enable
configure terminal

vlan 15
 name WMS-BODEGA
exit

vlan 99
 name MGMT
exit

end
write memory
```

---

## Estado

Con esto quedan los 7 switches de Colón con sus VLANs declaradas.

- [ ] `SW-DIST-BODEGA-COL` — VLANs aplicadas y verificadas
- [ ] `SW-DIST-BODEGA-COL-2` — VLANs aplicadas y verificadas
- [ ] `SW-ACC-COL` — VLANs aplicadas y verificadas
- [ ] `SW-ACC-BODEGA-COL` — VLANs aplicadas y verificadas

(Ver también `vlans-core-colon.md` y `vlans-dist-admin-colon.md` para el resto de los 7 switches.)

## Siguiente paso

Segunda mitad de la etapa 4.2 — ya no basta con que las VLANs existan:

1. **Asignación de puertos de acceso** — cada puerto donde hay un dispositivo final (ej. `PHONE-1`, los APs, `SERVER-WMS`) debe quedar en modo access, en la VLAN correspondiente.
2. **Enlaces troncales (802.1Q)** — entre acceso↔distribución y distribución↔core, para que las VLANs puedan cruzar entre switches.
