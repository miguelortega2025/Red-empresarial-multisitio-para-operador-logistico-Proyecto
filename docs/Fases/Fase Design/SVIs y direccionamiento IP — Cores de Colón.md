# SVIs y direccionamiento IP — Cores de Colón

**Fase:** Implement — Etapa 4.4 (Enrutamiento inter-VLAN, SVIs en el core)
**Dispositivos:** `SW-CORE-COL`, `SW-CORE-COL-2`
**Prerrequisito:** Trunks ya configurados (ver `trunks-colon.md`)

---

## Qué es una SVI y por qué es necesaria

Una VLAN por sí sola es solo un dominio de broadcast aislado — los dispositivos dentro de ella pueden hablar entre sí sin necesidad de IP en el switch. Pero en el momento en que un dispositivo de una VLAN necesita hablar con algo de otra VLAN, necesita un gateway: una puerta de salida hacia el resto de la red.

La SVI (`interface VlanXX` con una IP) es la interfaz virtual que convierte al switch core en el router de esa VLAN. Sin ella, un dispositivo puede operar dentro de su propia VLAN pero nunca comunicarse fuera de ella.

Cada VLAN cuyos dispositivos necesiten comunicarse con algo fuera de sí misma necesita su propia SVI — incluida GUEST, aunque hoy no tenga ningún puerto de acceso asignado, porque eventualmente necesitará salir a Internet.

## Esquema de direccionamiento usado

| IP | Uso |
|---|---|
| `.1` | IP virtual de HSRP (el gateway real que usarán los dispositivos) — se configura en la siguiente etapa, no en esta |
| `.2` | IP física de la SVI en `SW-CORE-COL` |
| `.3` | IP física de la SVI en `SW-CORE-COL-2` |

Se configura en dos pasos separados a propósito: primero cada core tiene su propia IP física (este documento), lo que permite probar el enrutamiento básico entre VLANs sin la complejidad de HSRP. Después se agrega HSRP encima, que añade la IP virtual `.1` compartida y la lógica de conmutación entre los dos cores.

## Prerrequisito: habilitar enrutamiento

Los switches L3 (3560/3650) no enrutan por defecto hasta que se activa. Se aplica una sola vez en cada core, antes o junto con la primera SVI:

```
enable
configure terminal
ip routing
end
write memory
```

---

## SVIs configuradas

### VLAN 10 (ADMIN) — 10.20.10.0/24

**En `SW-CORE-COL`:**
```
enable
configure terminal

interface Vlan10
 ip address 10.20.10.2 255.255.255.0
 no shutdown
exit

end
write memory
```

**En `SW-CORE-COL-2`:**
```
enable
configure terminal

interface Vlan10
 ip address 10.20.10.3 255.255.255.0
 no shutdown
exit

end
write memory
```

### VLAN 15 (WMS-BODEGA) — 10.20.15.0/24

VLAN dedicada a las terminales y lectores RFID/código de barras de bodega, conectados vía Wi-Fi industrial a través de `AP-COL-1` y `AP-COL-2`.

**En `SW-CORE-COL`:**
```
enable
configure terminal

interface Vlan15
 ip address 10.20.15.2 255.255.255.0
 no shutdown
exit

end
write memory
```

**En `SW-CORE-COL-2`:**
```
enable
configure terminal

interface Vlan15
 ip address 10.20.15.3 255.255.255.0
 no shutdown
exit

end
write memory
```

### VLAN 20 (SERVERS) — 10.20.20.0/24

**En `SW-CORE-COL`:**
```
enable
configure terminal

interface Vlan20
 ip address 10.20.20.2 255.255.255.0
 no shutdown
exit

end
write memory
```

**En `SW-CORE-COL-2`:**
```
enable
configure terminal

interface Vlan20
 ip address 10.20.20.3 255.255.255.0
 no shutdown
exit

end
write memory
```

### VLAN 30 (VOICE) — 10.20.30.0/24

**En `SW-CORE-COL`:**
```
enable
configure terminal

interface Vlan30
 ip address 10.20.30.2 255.255.255.0
 no shutdown
exit

end
write memory
```

**En `SW-CORE-COL-2`:**
```
enable
configure terminal

interface Vlan30
 ip address 10.20.30.3 255.255.255.0
 no shutdown
exit

end
write memory
```

### VLAN 90 (GUEST) — 10.20.90.0/24

Nota: ya se enruta aunque todavía no tiene ningún puerto de acceso asignado a ningún dispositivo real (decisión pendiente, ver `revision-topologia-colon.md`).

**En `SW-CORE-COL`:**
```
enable
configure terminal

interface Vlan90
 ip address 10.20.90.2 255.255.255.0
 no shutdown
exit

end
write memory
```

**En `SW-CORE-COL-2`:**
```
enable
configure terminal

interface Vlan90
 ip address 10.20.90.3 255.255.255.0
 no shutdown
exit

end
write memory
```

### VLAN 99 (MGMT) — 10.20.99.0/24

**En `SW-CORE-COL`:**
```
enable
configure terminal

interface Vlan99
 ip address 10.20.99.2 255.255.255.0
 no shutdown
exit

end
write memory
```

**En `SW-CORE-COL-2`:**
```
enable
configure terminal

interface Vlan99
 ip address 10.20.99.3 255.255.255.0
 no shutdown
exit

end
write memory
```

---

## Verificación

```
show ip interface brief
```

Resultado esperado: las 6 SVIs (`Vlan10`, `Vlan15`, `Vlan20`, `Vlan30`, `Vlan90`, `Vlan99`) en `up/up` en ambos cores, cada una con su IP `.2` o `.3` correspondiente.

**Prueba de conectividad realizada:** desde `SW-CORE-COL`, ping a la IP de `SW-CORE-COL-2` — confirmado exitoso. Esto valida que el trunk core↔core (paso 8 de `trunks-colon.md`) y el enrutamiento inter-VLAN básico funcionan correctamente.

---

## Resumen de direccionamiento

| VLAN | Nombre | Subred | IP Core-1 | IP Core-2 |
|---|---|---|---|---|
| 10 | ADMIN | 10.20.10.0/24 | 10.20.10.2 | 10.20.10.3 |
| 15 | WMS-BODEGA | 10.20.15.0/24 | 10.20.15.2 | 10.20.15.3 |
| 20 | SERVERS | 10.20.20.0/24 | 10.20.20.2 | 10.20.20.3 |
| 30 | VOICE | 10.20.30.0/24 | 10.20.30.2 | 10.20.30.3 |
| 90 | GUEST | 10.20.90.0/24 | 10.20.90.2 | 10.20.90.3 |
| 99 | MGMT | 10.20.99.0/24 | 10.20.99.2 | 10.20.99.3 |

---

## Estado

- [x] `ip routing` habilitado en ambos cores
- [x] VLAN 10 — SVIs configuradas en ambos cores
- [x] VLAN 15 — SVIs configuradas en ambos cores
- [x] VLAN 20 — SVIs configuradas en ambos cores
- [x] VLAN 30 — SVIs configuradas en ambos cores
- [x] VLAN 90 — SVIs configuradas en ambos cores
- [x] VLAN 99 — SVIs configuradas en ambos cores
- [x] Verificación de conectividad (ping core-a-core) — exitosa

## Siguiente paso

Etapa 4.5 — HSRP: agregar la IP virtual `.1` compartida en cada VLAN y definir cuál core es primario y cuál respaldo.
