# Configuración base — Switches (Colón)

**Fase:** Implement — Etapa 4.1 (Configuración base de dispositivos)
**Dispositivos:** 7 switches de la sede Colón

---

## Plantilla base

Usé la misma plantilla para los 7 switches de Colón, cambiando únicamente el hostname en cada uno. La aplico en modo de configuración global (`enable` → `configure terminal`):

```
hostname SW-CORE-COL

enable secret Colon2026!

no ip domain-lookup

banner motd #
ACCESO RESTRINGIDO - Red operador logistico
Solo personal autorizado. Colon - Sede critica 24/7
#

line console 0
 password consola2026
 login
 logging synchronous
 exec-timeout 5 0
exit

line vty 0 15
 password vty2026
 login
 exec-timeout 5 0
exit

service password-encryption

end
write memory
```

## Hostnames aplicados

| Equipo | Hostname usado |
|---|---|
| Core 1 | `SW-CORE-COL` |
| Core 2 | `SW-CORE-COL-2` |
| Distribución admin | `SW-DIST-ADMIN-COL` |
| Distribución bodega 1 | `SW-DIST-BODEGA-COL` |
| Distribución bodega 2 | `SW-DIST-BODEGA-COL-2` |
| Acceso admin/voz | `SW-ACC-COL` |
| Acceso bodega | `SW-ACC-BODEGA-COL` |

---

## Por qué elegí cada línea de la plantilla

- **`enable secret`** en vez de `enable password` — se guarda con hash MD5, no en texto plano.
- **`no ip domain-lookup`** — evita que el switch se cuelgue varios segundos intentando resolver DNS cada vez que escribo mal un comando (típico dolor de cabeza en Packet Tracer).
- **`exec-timeout 5 0`** — cierra sesiones de consola/vty inactivas tras 5 minutos, buena práctica de seguridad básica.
- **`service password-encryption`** — cifra (débilmente, pero cifra) las contraseñas de línea que quedarían en texto plano en el `show running-config`.

## Lo que dejé pendiente a propósito

Todavía no configuré `line vty` con `transport input ssh` ni gestión remota real, porque eso depende de que primero exista una IP de management (VLAN 99), que corresponde al paso 2. Por ahora las vty quedan con acceso básico por Telnet dentro del simulador.

---

## Pendiente antes de continuar con los firewalls

Al revisar mi catálogo disponible en Packet Tracer no encontré ningún modelo de firewall (ASA, PT-Firewall, etc.), solo switches. Necesito definir qué dispositivo voy a usar para `FW-COL` y `FW-COL-2`, porque la sintaxis de configuración base cambia bastante según sea:

- Un Cisco ASA (usa `nameif`, `security-level`, etc.)
- Un router genérico haciendo de firewall (IOS normal con ACLs)
- El dispositivo genérico "Firewall" que trae Packet Tracer por defecto

*(Resuelto: se definió ASA 5506-X — ver documento de configuración base de firewalls)*

---

## Estado

- [ ] `SW-CORE-COL`
- [ ] `SW-CORE-COL-2`
- [ ] `SW-DIST-ADMIN-COL`
- [ ] `SW-DIST-BODEGA-COL`
- [ ] `SW-DIST-BODEGA-COL-2`
- [ ] `SW-ACC-COL`
- [ ] `SW-ACC-BODEGA-COL`

## Siguiente paso

Etapa 4.2 — VLANs y asignación de puertos.
