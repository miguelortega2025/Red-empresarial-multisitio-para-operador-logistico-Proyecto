# Configuración base — Firewalls ASA 5506-X (Colón)

**Fase:** Implement — Etapa 4.1 (Configuración base de dispositivos)
**Dispositivos:** `FW-COL`, `FW-COL-2`
**Modelo:** Cisco ASA 5506-X

---

## Nota importante

El ASA 5506-X usa una sintaxis distinta a IOS. No se deben mezclar comandos de switch/router con comandos de ASA.

---

## Configuración base aplicada

```
enable
configure terminal

hostname FW-COL

enable password ColonFW2026! encrypted
domain-name logistica-col.local

banner motd #
ACCESO RESTRINGIDO - Firewall perimetral Colon
Solo personal de TI autorizado
#

clock timezone COL -5

passwd Colon2026!
! Nota: passwd es la contraseña del modo "login" inicial, distinta del enable

no ip domain-lookup

end
write memory
```

**Para `FW-COL-2`:** aplicar el mismo bloque, cambiando únicamente `hostname FW-COL` por `hostname FW-COL-2`.

---

## Diferencias clave frente a los switches IOS

| Concepto | En un switch/router IOS | En el ASA |
|---|---|---|
| Contraseña de privilegio | `enable secret` | `enable password` |
| Contraseña de acceso inicial | `line console 0` / `password` | `passwd` (comando global, no dentro de `line`) |
| Interfaces | `Fa0/1`, `Gi0/1` | Nombre lógico obligatorio: `nameif outside`, `nameif inside`, más `security-level` (0–100) |
| VTY para gestión remota | `line vty 0 15` | Se configura con `ssh` o `telnet` + `management-access` apuntando a una interfaz específica |

---

## Pendiente para la etapa de interfaces (no configurado en este paso)

Cada interfaz física del ASA necesita tres elementos obligatorios antes de pasar tráfico:

```
interface GigabitEthernet1/1
 nameif inside
 security-level 100
 ip address 10.20.99.1 255.255.255.0
 no shutdown

interface GigabitEthernet1/2
 nameif outside-ispa
 security-level 0
 ip address dhcp
 no shutdown
```

### Concepto de `security-level`

No existe en switches normales. Escala de 0 a 100:

- **100** = red más confiable (interna)
- **0** = red menos confiable (Internet / ISP)

Por defecto, el ASA **bloquea todo tráfico de una interfaz de menor security-level hacia una de mayor**, a menos que exista una regla o traducción NAT explícita. Esto se configura en la etapa de seguridad (ACLs y NAT), no en la configuración base.

---

## Estado

- [x] `FW-COL` — configuración base aplicada
- [ ] `FW-COL-2` — pendiente (aplicar mismo bloque con hostname distinto)

## Siguiente paso

Etapa 4.2 — VLANs y asignación de puertos.
