# Supuestos, Restricciones y Riesgos

**Fase 1: PREPARE | Metodología Cisco PPDIOO**

[← Volver al README](../README.md) | [Contexto de negocio](Contexto_Negocio.md) | [Siguiente: fase Plan →](../02_Planificar/)

---

## 1. Propósito de este documento

Complementa el [Contexto de Negocio](Contexto_Negocio.md) con tres registros que alimentan las fases siguientes:

1. **Supuestos:** lo que se da por cierto para poder diseñar, y que debe validarse en la fase Plan.
2. **Restricciones:** límites que condicionan las decisiones de diseño.
3. **Riesgos:** eventos que podrían afectar la continuidad operativa de la empresa o la calidad del proyecto, con su mitigación.

Como el proyecto es una simulación con una empresa ficticia, los valores son estimaciones razonables, no mediciones reales.

---

## 2. Supuestos

Cada supuesto indica en qué fase se valida o se convierte en un requisito concreto.

| ID | Supuesto | Valor estimado | Se valida en |
|---|---|---|---|
| S-01 | Usuarios en Ciudad de Panamá | ≈ 50 (administración, comercial, TI) | Plan |
| S-02 | Usuarios en Colón | ≈ 30 (bodega, coordinación logística) | Plan |
| S-03 | Usuarios en el Interior | ≈ 10 (sucursal comercial) | Plan |
| S-04 | Empresa nueva, sin red previa | No existe infraestructura que migrar ni inventario heredado | Plan |
| S-05 | Operación en Colón 24/7 | La bodega no se detiene; los mantenimientos requieren plan de respaldo | Design |
| S-06 | Picos estacionales | El volumen puede duplicarse en temporada navideña; el ancho de banda se dimensiona para el pico | Plan |
| S-07 | Plataformas WMS y ERP | Software estándar de mercado, on-premise o en la nube, compatible con conexión remota | Plan |
| S-08 | Presupuesto | Rango medio, priorizando disponibilidad en Panamá y Colón por encima del Interior | Plan |
| S-09 | Despliegue por etapas | Panamá primero, luego Colón e Interior | Plan |
| S-10 | Trazabilidad auditable | Por ser Zona Libre, Colón probablemente requiere registros auditables de la mercancía | Design |

---

## 3. Restricciones

| ID | Restricción | Efecto sobre el diseño |
|---|---|---|
| C-01 | Oferta limitada de proveedores de Internet en el Interior | Condiciona el tipo de respaldo disponible (se contempla 4G/5G) |
| C-02 | Presupuesto de rango medio | La redundancia se concentra en las sedes críticas (Panamá y Colón); el Interior queda con diseño más simple |
| C-03 | Plataformas WMS/ERP aún sin definir | El diseño no puede depender de un producto concreto; se parte de requisitos genéricos (acceso remoto, baja latencia) |
| C-04 | Proyecto realizado en Cisco Packet Tracer | Algunos elementos del negocio (Wi-Fi industrial, lectores RFID, enlaces reales de ISP) se representan de forma aproximada con dispositivos genéricos |
| C-05 | Alcance alineado al temario CCNA 200-301 | Las tecnologías del diseño se limitan a lo que el simulador y el temario permiten implementar y verificar |

---

## 4. Registro de riesgos

### 4.1 Escala de evaluación

| Nivel | Probabilidad | Impacto |
|---|---|---|
| **Baja** | Poco probable que ocurra | Afecta a un servicio secundario o a pocos usuarios |
| **Media** | Puede ocurrir ocasionalmente | Degrada un servicio importante o afecta a una sede |
| **Alta** | Probable que ocurra | Detiene la operación de una sede crítica |

El **nivel de riesgo** combina ambos: *Alta + Alta = Crítico*; si solo una es Alta, el riesgo es *Alto*; en los demás casos, *Medio* o *Bajo*.

### 4.2 Riesgos de negocio y operación

| ID | Riesgo | Prob. | Impacto | Nivel | Mitigación prevista | Se aborda en |
|---|---|---|---|---|---|---|
| R-01 | **Aislamiento de Colón:** pérdida de conexión con Panamá, con pérdida de visibilidad de inventario y despacho | Media | Alta | **Alto** | Doble ISP, doble firewall y núcleo duplicado en Colón; pruebas de fallo | Design / Operate |
| R-02 | **Caída de un proveedor de Internet** en Panamá o Colón | Media | Alta | **Alto** | Segundo proveedor con conmutación automática; verificación en pruebas de fallo de ISP | Design / Operate |
| R-03 | **Fallo del gateway** en Panamá, que deja a los usuarios sin salida a otras VLANs o sedes | Media | Alta | **Alto** | Redundancia de gateway con HSRP en núcleo duplicado | Design / Implement |
| R-04 | **Saturación de ancho de banda en picos** (temporada navideña) | Media | Media | **Medio** | Dimensionar los enlaces para el pico, no el promedio; priorizar tráfico del WMS | Plan / Design |
| R-05 | **Sede del Interior sin respaldo suficiente** por oferta limitada de ISP | Alta | Baja | **Medio** | Enlace 4G como respaldo de WAN; aceptar que no tiene redundancia de núcleo | Design |
| R-06 | **Acceso no autorizado** a la red de bodega, donde circula mercancía de alto valor | Media | Alta | **Alto** | Segmentación por VLAN, ACLs, firewall perimetral, port security y SSH | Design / Implement |
| R-07 | **Falta de trazabilidad auditable** en Zona Libre | Baja | Alta | **Medio** | Syslog centralizado y sincronización NTP en todos los equipos | Design / Implement |
| R-08 | **Servicios de infraestructura caídos** (DHCP, NTP, Syslog), que dejan equipos sin direccionamiento o sin registros | Media | Media | **Medio** | Ubicar los servicios de forma centralizada y documentar su dependencia; evaluar respaldo | Design |
| R-09 | **Presupuesto insuficiente** para cubrir la redundancia deseada | Media | Media | **Medio** | Priorizar redundancia en Panamá y Colón; despliegue por etapas | Plan |
| R-10 | **Plataformas WMS/ERP con requisitos no previstos** (latencia, puertos, acceso remoto) | Media | Media | **Medio** | Diseñar con margen y documentar los requisitos al definir la plataforma | Plan |

### 4.3 Riesgos técnicos de diseño y del proyecto

| ID | Riesgo | Prob. | Impacto | Nivel | Mitigación prevista | Se aborda en |
|---|---|---|---|---|---|---|
| R-11 | **Bucles en capa 2** por la redundancia de enlaces entre núcleo, distribución y acceso | Media | Alta | **Alto** | STP bien configurado (raíz definida) y EtherChannel donde aplique; verificación en pruebas | Design / Implement |
| R-12 | **Diseño más complejo que su documentación:** elementos dibujados pero no configurados ni probados | Media | Media | **Medio** | Trazabilidad requerimiento → configuración → prueba; actualizar los objetivos del README | Plan / Operate |
| R-13 | **Interconexión entre sedes sin definir** (túneles, VPN u otra opción) | Media | Alta | **Alto** | Decidir y justificar en el documento de decisiones de diseño | Design |
| R-14 | **Limitaciones del simulador** que impidan reproducir una función real | Media | Baja | **Bajo** | Documentar cada limitación y su aproximación; no afirmar que se probó lo que el simulador no permite | Design / Operate |
| R-15 | **Errores de configuración** que rompan la conectividad entre sedes | Alta | Media | **Alto** | Configuraciones guardadas como `.txt` por equipo, versionadas en Git; pruebas tras cada cambio | Implement / Operate |
| R-16 | **Pérdida o corrupción del archivo `.pkt`** (binario, sin control de versiones útil) | Baja | Media | **Bajo** | Guardar versiones finales como releases de GitHub y mantener las configuraciones en texto | Implement |

---

## 5. Mapa de riesgos

Distribución de los 16 riesgos según su nivel:

| Nivel | Riesgos |
|---|---|
| **Alto** | R-01, R-02, R-03, R-06, R-11, R-13, R-15 |
| **Medio** | R-04, R-05, R-07, R-08, R-09, R-10, R-12 |
| **Bajo** | R-14, R-16 |

Los riesgos altos coinciden con la prioridad #1 del negocio: **continuidad operativa entre Panamá y Colón**. Por eso la redundancia (doble ISP, doble firewall, núcleo duplicado, HSRP) se concentra en esas dos sedes.

---

## 6. Trazabilidad hacia las siguientes fases

| Fase | Qué toma de este documento |
|---|---|
| **Plan** | Valida los supuestos S-01 a S-10 y los convierte en requisitos medidos; afina el presupuesto (R-09) y los requisitos de WMS/ERP (R-10) |
| **Design** | Responde a los riesgos R-01, R-02, R-03, R-05, R-06, R-11 y R-13 con decisiones concretas, y las registra en `Decisiones_Diseno.md` |
| **Implement** | Aplica las mitigaciones de configuración (HSRP, STP, ACLs, port security, Syslog/NTP) y controla R-15 con las configuraciones versionadas |
| **Operate** | Verifica con pruebas de fallo, redundancia y seguridad que las mitigaciones funcionan (R-01, R-02, R-03, R-06, R-11) |
| **Optimize** | Revisa qué riesgos se materializaron en las pruebas y qué mejoras se proponen |

---

## 7. Control de cambios

Este registro se revisa al cerrar cada fase. Los riesgos que se materialicen, se mitiguen o cambien de nivel se actualizan aquí, con su fecha.

| Fecha | Cambio | Fase |
|---|---|---|
| <!-- COMPLETAR: fecha --> | Versión inicial del registro | Prepare |
