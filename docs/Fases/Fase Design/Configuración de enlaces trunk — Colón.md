# Configuración de enlaces trunk — Colón

**Fase:** Implement — Etapa 4.3 (Enlaces troncales 802.1Q)
**Sede:** Colón (Zona Libre)
**Prerrequisito:** VLANs ya declaradas y puertos de acceso ya asignados (ver `vlans-core-colon.md`, `vlans-dist-admin-colon.md`, `vlans-dist-bodega-y-acceso-colon.md`)

---

## Por qué este paso es necesario

Con solo VLANs creadas y puertos de acceso asignados, el tráfico queda atrapado dentro de cada switch — por defecto, los puertos que conectan switch con switch solo dejan pasar una VLAN (la VLAN 1). Un trunk le dice a un puerto "deja pasar TODAS las VLANs que yo indique, etiquetadas", y se configura únicamente en los cables switch-a-switch, nunca en los puertos donde hay un dispositivo final (esos se quedan en modo access).

Los enlaces hacia los firewalls (`FW-COL`, `FW-COL-2`) no se incluyen aquí — esos se configuran como enlaces routeados simples, no como trunk multi-VLAN, y se documentan aparte.

## Regla aplicada para decidir qué VLANs permitir en cada trunk

Un trunk debe permitir la unión de todas las VLANs que puedan llegar por cualquiera de los enlaces hacia abajo del switch, no solo las que ese switch usa directamente en sus propios puertos de acceso. Por eso `SW-DIST-BODEGA-COL` (que tiene un enlace extra hacia `SW-ACC-COL`) permite más VLANs en sus uplinks a core que `SW-DIST-BODEGA-COL-2`.

---

## Paso 1 — SW-ACC-COL ↔ SW-DIST-ADMIN-COL

**Interfaces:** F0/1 (SW-ACC-COL) ↔ Fa0/1 (SW-DIST-ADMIN-COL)
**VLANs:** 10, 30, 99

```
enable
configure terminal

interface FastEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,30,99
exit

end
write memory
```

Mismo bloque en ambos extremos, cambiando el nombre de la interfaz según el equipo.

---

## Paso 2 — SW-ACC-COL ↔ SW-DIST-BODEGA-COL

**Interfaces:** G0/1 (SW-ACC-COL) ↔ G0/2 (SW-DIST-BODEGA-COL)
**VLANs:** 10, 30, 99

```
enable
configure terminal

interface GigabitEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,30,99
exit

end
write memory
```

**Nota:** aunque el puerto físico está en `SW-DIST-BODEGA-COL` (que normalmente solo maneja 15/20/99), aquí se permiten 10/30/99 porque este trunk específico es el segundo camino de `SW-ACC-COL` hacia el core — `SW-DIST-BODEGA-COL` actúa solo como tránsito para esas VLANs.

---

## Paso 3 — SW-ACC-BODEGA-COL ↔ SW-DIST-BODEGA-COL

**Interfaces:** Fa0/3 (SW-ACC-BODEGA-COL) ↔ F0/2 (SW-DIST-BODEGA-COL)
**VLANs:** 15, 99

```
enable
configure terminal

interface FastEthernet0/3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 15,99
exit

end
write memory
```

---

## Paso 4 — SW-ACC-BODEGA-COL ↔ SW-DIST-BODEGA-COL-2

**Interfaces:** Fa0/4 (SW-ACC-BODEGA-COL) ↔ F0/1 (SW-DIST-BODEGA-COL-2)
**VLANs:** 15, 99

```
enable
configure terminal

interface FastEthernet0/4
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 15,99
exit

end
write memory
```

Con los pasos 3 y 4, `SW-ACC-BODEGA-COL` queda con dos trunks activos hacia sus dos distribuciones — la base para que Rapid PVST+ elija un camino activo y deje el otro en bloqueo, listo para tomar el relevo si el primero falla.

---

## Paso 5 — SW-DIST-ADMIN-COL ↔ SW-CORE-COL / SW-CORE-COL-2

**VLANs:** 10, 30, 99

**Enlace 1:** G0/1 (SW-DIST-ADMIN-COL) ↔ F0/24 (SW-CORE-COL)
**Enlace 2:** G0/2 (SW-DIST-ADMIN-COL) ↔ G0/1 (SW-CORE-COL-2)

```
enable
configure terminal

interface GigabitEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,30,99
exit

end
write memory
```

(Mismo bloque en los 4 extremos, con la interfaz correspondiente a cada uno.)

---

## Paso 6 — SW-DIST-BODEGA-COL ↔ SW-CORE-COL / SW-CORE-COL-2

**VLANs:** 10, 15, 20, 30, 99 (incluye 10 y 30 porque este switch tiene el enlace extra hacia `SW-ACC-COL` del paso 2)

**Enlace 1:** G0/1 (SW-DIST-BODEGA-COL) ↔ G0/2 (SW-CORE-COL)
**Enlace 2:** F0/1 (SW-DIST-BODEGA-COL) ↔ G0/2 (SW-CORE-COL-2)

```
enable
configure terminal

interface GigabitEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,15,20,30,99
exit

end
write memory
```

(Mismo bloque en los 4 extremos, con la interfaz correspondiente a cada uno.)

---

## Paso 7 — SW-DIST-BODEGA-COL-2 ↔ SW-CORE-COL / SW-CORE-COL-2

**VLANs:** 15, 20, 99 (sin 10/30 — este switch no tiene el enlace extra hacia `SW-ACC-COL`)

**Enlace 1:** F0/3 (SW-DIST-BODEGA-COL-2) ↔ F0/1 (SW-CORE-COL)
**Enlace 2:** F0/2 (SW-DIST-BODEGA-COL-2) ↔ F0/3 (SW-CORE-COL-2)

```
enable
configure terminal

interface FastEthernet0/3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 15,20,99
exit

end
write memory
```

(Mismo bloque en los 4 extremos, con la interfaz correspondiente a cada uno.)

---

## Paso 8 — SW-CORE-COL ↔ SW-CORE-COL-2

**Interfaces:** F0/2 (SW-CORE-COL) ↔ F0/1 (SW-CORE-COL-2)
**VLANs:** 10, 15, 20, 30, 90, 99 — todas. Es el enlace más importante del bloque: conecta los dos cores entre sí y es el que permitirá que HSRP se sincronice entre ambos en la siguiente etapa.

```
enable
configure terminal

interface FastEthernet0/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,15,20,30,90,99
exit

end
write memory
```

---

## Verificación general

En cada uno de los 7 switches:

```
show interfaces trunk
show interfaces status
```

Qué confirmar:

- Cada trunk aparece como `trunking` (no `not-trunking` ni `down`)
- Las VLANs permitidas coinciden con lo configurado en cada enlace
- Ningún puerto que debería ser trunk aparece como `access` por error

**Problema típico:** si el `show interfaces trunk` no muestra un puerto que debería, casi siempre es porque falta el comando en uno de los dos extremos del cable — el trunk se negocia entre ambos lados.

---

## Resumen de enlaces trunk

| # | Enlace | VLANs permitidas |
|---|---|---|
| 1 | SW-ACC-COL ↔ SW-DIST-ADMIN-COL | 10, 30, 99 |
| 2 | SW-ACC-COL ↔ SW-DIST-BODEGA-COL | 10, 30, 99 |
| 3 | SW-ACC-BODEGA-COL ↔ SW-DIST-BODEGA-COL | 15, 99 |
| 4 | SW-ACC-BODEGA-COL ↔ SW-DIST-BODEGA-COL-2 | 15, 99 |
| 5 | SW-DIST-ADMIN-COL ↔ SW-CORE-COL | 10, 30, 99 |
| 5 | SW-DIST-ADMIN-COL ↔ SW-CORE-COL-2 | 10, 30, 99 |
| 6 | SW-DIST-BODEGA-COL ↔ SW-CORE-COL | 10, 15, 20, 30, 99 |
| 6 | SW-DIST-BODEGA-COL ↔ SW-CORE-COL-2 | 10, 15, 20, 30, 99 |
| 7 | SW-DIST-BODEGA-COL-2 ↔ SW-CORE-COL | 15, 20, 99 |
| 7 | SW-DIST-BODEGA-COL-2 ↔ SW-CORE-COL-2 | 15, 20, 99 |
| 8 | SW-CORE-COL ↔ SW-CORE-COL-2 | 10, 15, 20, 30, 90, 99 |

---

## Estado

- [ ] Paso 1 — SW-ACC-COL ↔ SW-DIST-ADMIN-COL
- [ ] Paso 2 — SW-ACC-COL ↔ SW-DIST-BODEGA-COL
- [ ] Paso 3 — SW-ACC-BODEGA-COL ↔ SW-DIST-BODEGA-COL
- [ ] Paso 4 — SW-ACC-BODEGA-COL ↔ SW-DIST-BODEGA-COL-2
- [ ] Paso 5 — SW-DIST-ADMIN-COL ↔ SW-CORE-COL / SW-CORE-COL-2
- [ ] Paso 6 — SW-DIST-BODEGA-COL ↔ SW-CORE-COL / SW-CORE-COL-2
- [ ] Paso 7 — SW-DIST-BODEGA-COL-2 ↔ SW-CORE-COL / SW-CORE-COL-2
- [ ] Paso 8 — SW-CORE-COL ↔ SW-CORE-COL-2
- [ ] Verificación general (`show interfaces trunk` en los 7 switches)

## Siguiente paso

Etapa 4.4 — SVIs y direccionamiento IP en los switches core (aquí empieza el direccionamiento IP de la red).
