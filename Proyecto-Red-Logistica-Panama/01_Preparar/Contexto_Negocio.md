# Fase 1: PREPARE (Preparar)

**Metodología Cisco PPDIOO**

Diseño de red empresarial para un operador logístico con presencia en Ciudad de Panamá, Colón (Zona Libre) e Interior del país.

*Documento de preparación y caso de negocio — Proyecto de portafolio técnico*

[← Volver al README](../README.md) | [Siguiente: fase Plan →](../02_Planificar/)

---

## 1. ¿Qué es la fase Prepare?

Dentro de la metodología PPDIOO de Cisco (Prepare, Plan, Design, Implement, Operate, Optimize), la fase **Prepare** es el punto de partida de todo proyecto de red. Es la etapa en la que no se toca ni un cable, ni un router, ni una VLAN. Se trata exclusivamente de responder una pregunta fundamental:

> **¿Por qué esta empresa necesita una red, y qué justifica invertir tiempo y dinero en construirla?**

Es habitual que, al hablar de "diseñar una red", uno piense de inmediato en switches, direcciones IP o VLANs. Ese es el error más común de quienes empiezan en redes: saltar directamente al aspecto técnico sin haber entendido primero para qué existe la empresa, cómo opera y qué problema resuelve la red dentro de ese negocio. La fase Prepare existe precisamente para evitar ese error.

> **Analogía simple**
> Antes de construir una casa, un arquitecto no empieza a pedir ladrillos. Primero pregunta: ¿quién va a vivir aquí?, ¿cuántas habitaciones necesitan?, ¿tienen carro?, ¿es para toda la vida o para vender en unos años? La fase Prepare es exactamente eso, pero aplicado a una red: antes de elegir routers, primero se entiende el negocio.

### 1.1 ¿Por qué es la primera fase y no se puede saltar?

Si se empieza a diseñar la red sin haber hecho esta fase, se corre un riesgo muy concreto: terminar construyendo una red técnicamente correcta pero inútil para el negocio. Por ejemplo, se podría invertir en redundancia costosa en una sede que en realidad no es crítica, o dejar sin respaldo de conexión a la sede que sí lo necesita. La fase Prepare evita ese desperdicio de recursos porque obliga a entender el negocio primero.

---

## 2. Los tres componentes de la fase Prepare

La fase Prepare se construye respondiendo, en orden, tres preguntas. Cada una alimenta a la siguiente:

| Componente | Pregunta que responde | Resultado esperado |
|---|---|---|
| **1. Necesidad de negocio** | ¿Por qué se necesita la red? | Una historia clara y justificada del problema u oportunidad |
| **2. Caso de negocio** | ¿Se justifica la inversión? | Comparación costo / beneficio / riesgo de no hacerlo |
| **3. Tecnologías candidatas** | ¿Con qué tipo de soluciones se podría resolver? | Lista de opciones tecnológicas, sin comprometerse aún a ninguna |

### Cómo se relacionan entre sí

Estos tres componentes no son independientes: cada uno alimenta al siguiente, en un flujo secuencial. Primero se entiende el negocio, después se justifica la inversión con base en esa necesidad, y solo al final se listan posibles caminos tecnológicos.

```
1. Necesidad de negocio  →  2. Caso de negocio  →  3. Tecnologías candidatas
 (la historia y el          (costo / beneficio /     (familias de solución
  problema)                  riesgo)                  posibles)
```

### 2.1 Necesidad de negocio

Es el punto de partida narrativo: describir por qué existe el problema o la oportunidad que la red va a resolver. No se habla todavía de tecnología, sino de la realidad operativa de la empresa. Las preguntas típicas que se responden aquí son:

- ¿Qué hace la empresa día a día?
- ¿Dónde están ubicadas sus sedes y qué función cumple cada una?
- ¿Qué pasa hoy si no existe la red (o si la actual no alcanza)?
- ¿Qué se pierde si no se resuelve este problema (tiempo, dinero, clientes, seguridad)?

### 2.2 Caso de negocio (Business Case)

Una vez identificada la necesidad, se arma el argumento que justificaría, ante una gerencia o un inversionista, invertir en esta red. En un proyecto real esto se traduce en cifras concretas; en un proyecto de portafolio, se traduce en una justificación creíble y coherente, aunque los números sean estimados.

| Elemento | Pregunta clave |
|---|---|
| Costo del problema actual | ¿Qué se pierde hoy por no tener la red (errores, tiempo, riesgo)? |
| Costo de la solución | ¿Cuánto cuesta construir la red (equipos, cableado, personal)? |
| Beneficio esperado | ¿Qué mejora directamente (velocidad, errores, visibilidad)? |
| Riesgo de no hacerlo | ¿Qué pasa si la empresa sigue operando sin esta red? |

### 2.3 Tecnologías candidatas

Por último, se hace un primer listado de posibles categorías de tecnología que podrían usarse para resolver la necesidad. Aquí es muy importante entender un límite: **todavía no se elige nada en firme**. No se decide "usaremos OSPF" ni "usaremos tal marca de switch". Eso ocurre después, en la fase Design. Aquí solo se listan las familias de solución posibles, para tenerlas en mente.

> Es como cuando alguien va a comprar carro y dice "necesito algo espacioso porque tengo familia grande, y que gaste poca gasolina". Todavía no eligió el modelo exacto, pero ya definió el tipo de carro que busca. Eso es lo que se hace aquí con la tecnología.

---

## 3. Aplicación al proyecto: empresa logística en Panamá

A continuación se desarrolla la fase Prepare completa, aplicada al proyecto de portafolio: una empresa logística ficticia, pero diseñada con una lógica de negocio realista, con presencia en tres puntos estratégicos de Panamá.

### 3.1 Perfil de la empresa

| Sede | Rol dentro del negocio | Por qué importa para la red |
|---|---|---|
| **Ciudad de Panamá** | Centro corporativo y tecnológico: dirección, finanzas, RRHH, ventas, TI | Aquí se aloja el ERP y, probablemente, el centro de servidores; necesita buen ancho de banda y seguridad administrativa |
| **Colón (Zona Libre)** | Operación logística: bodega y movimiento de mercancía de alto valor | Es el corazón operativo; necesita el WMS funcionando de forma confiable y seguridad reforzada |
| **Interior del país** | Sucursal comercial de menor volumen | Conectividad más limitada (zona rural); puede requerir respaldo por 4G/5G |

#### El negocio en la práctica: flujo físico de la mercancía

Para entender por qué cada sede necesita lo que necesita, ayuda imaginar el recorrido real de un pedido: la empresa recibe contenedores de mercancía en la Zona Libre de Colón, donde se desconsolidan, se etiquetan y se almacenan en bodega. Desde allí, vehículos transportan los pedidos hacia clientes en Ciudad de Panamá y hacia la sucursal del Interior, que a su vez atiende a comercios locales de esa región. La trazabilidad de cada caja —desde que ingresa al puerto hasta que se entrega al cliente final— depende del **WMS** en Colón, mientras que la facturación y la coordinación general del transporte dependen del **ERP** en la sede corporativa de Panamá.

#### Criticidad temporal de la operación

La operación en Colón es continua (**24/7**), ya que el movimiento de contenedores en zona portuaria no se detiene. Además, como es típico en logística, existen picos estacionales —por ejemplo, en temporada navideña— donde el volumen de mercancía puede duplicarse. Este dato es relevante para la red: una bodega que opera 24/7 no puede depender de un solo enlace de Internet ni permitirse mantenimientos prolongados sin plan de respaldo, y el dimensionamiento de ancho de banda debe pensarse para los picos, no solo para el promedio.

#### Modelo de ingresos (por qué paga el cliente)

La empresa factura por un servicio integral de logística: **almacenaje** (tiempo que la mercancía permanece en bodega) y **flete** (transporte hacia el destino final). Esto hace que el WMS no sea solo una herramienta operativa, sino una pieza directamente ligada a la facturación: sin datos confiables y en tiempo real sobre cuánto tiempo estuvo cada carga en bodega, la empresa no puede cobrar con precisión por el servicio prestado.

#### Supuestos y restricciones del proyecto

Como se trata de una empresa ficticia con fines de portafolio, se fijan aquí supuestos razonables. Estos datos son los que la fase Plan retomará para convertirlos en requisitos técnicos exactos.

| Supuesto | Valor estimado |
|---|---|
| Usuarios aproximados — Ciudad de Panamá | ≈ 50 personas (administración, comercial, TI) |
| Usuarios aproximados — Colón | ≈ 30 personas (bodega, coordinación logística) |
| Usuarios aproximados — Interior | ≈ 10 personas (sucursal comercial) |
| Presupuesto estimado | Rango medio — priorizando disponibilidad en Panamá y Colón por encima del interior |
| Plazo esperado de implementación | Por definir en fase Plan; se asume un despliegue por etapas (Panamá primero, luego Colón e Interior) |
| Restricción geográfica | La sede del Interior puede tener oferta limitada de proveedores de Internet, lo que condiciona el tipo de respaldo disponible |
| Plataforma de WMS / ERP | Por definir en fase Plan; se asume software estándar de mercado (on-premise o en la nube) compatible con conexión remota |

### 3.2 Necesidad de negocio (aplicada)

Es una empresa nueva, sin red previa. Esto significa que el caso de negocio no se justifica como "vamos a reemplazar algo que falla", sino como "vamos a habilitar la operación desde el primer día, de forma que pueda crecer sin tener que rehacerse". Los argumentos concretos son:

- Sin red no hay manera de operar el WMS en Colón, por lo que no hay trazabilidad de mercancía.
- Sin red no hay ERP centralizado, por lo que la facturación y administración no pueden operar de forma confiable.
- Al ser zona franca, Colón probablemente requiere registros y trazabilidad auditable de la mercancía que se mueve.
- Diseñar pensando en las tres sedes desde el inicio evita tener que "parchar" la red más adelante, lo cual sale más caro.
- Si Colón —el punto operativo más crítico— pierde conexión con Panamá, se detiene la visibilidad de inventario y despacho.

### 3.3 Caso de negocio (aplicado)

| Elemento | Aplicación al proyecto |
|---|---|
| Costo del problema actual | No poder operar: sin red no hay forma de administrar inventario, facturar ni coordinar despachos entre sedes |
| Costo de la solución | Inversión en equipos de red, cableado, Wi-Fi industrial en Colón, enlaces WAN entre sedes y seguridad perimetral |
| Beneficio esperado | Trazabilidad de mercancía en tiempo real, facturación centralizada, comunicación confiable entre las tres sedes |
| Riesgo de no hacerlo | Aislamiento de Colón (el punto más crítico), pérdida de visibilidad operativa, imposibilidad de escalar a nuevas sedes |
| Prioridad de negocio | Continuidad operativa: que la red esté disponible de forma confiable, con redundancia en los puntos críticos (Panamá y Colón) |

### 3.4 Tecnologías candidatas (aplicadas, sin comprometer diseño)

Estas son las familias de tecnología identificadas como opciones razonables para este negocio. La decisión final de cuál usar y cómo configurarla se toma en la fase Design, no aquí.

- **Conectividad entre sedes:** VPN site-to-site sobre Internet, o alternativa más avanzada con opciones tipo SD-WAN
- **Conectividad a Internet por sede:** doble proveedor en Panamá y Colón por criticidad; posible respaldo móvil (4G/5G) en el interior
- **Red interna en Colón:** Wi-Fi industrial y cableado estructurado para terminales de bodega y lectores de código de barras/RFID
- **Seguridad:** firewall perimetral, segmentación de tráfico, control de acceso reforzado en Colón
- **Sistemas a soportar:** WMS (bodega), ERP (administración), telefonía IP, servicios de infraestructura (DNS, DHCP, NTP, Syslog)

---

## 4. Resumen ejecutivo de la fase Prepare

Este resumen es, en esencia, lo que se presentaría a alguien que evalúe el proyecto (un reclutador, un profesor, un cliente) para explicar por qué existe esta red antes de mostrar ni un diagrama técnico.

| Pregunta | Respuesta del proyecto |
|---|---|
| ¿Qué empresa es y dónde opera? | Operador logístico panameño con sedes en Ciudad de Panamá, Colón (Zona Libre) e Interior del país |
| ¿Por qué necesita una red? | Es una empresa nueva sin infraestructura; necesita operar WMS, ERP y comunicación entre sedes desde el día uno |
| ¿Qué se justifica invertir? | Una red diseñada desde el inicio pensando en las tres sedes, evitando sobrecostos futuros por parchar la infraestructura |
| ¿Cuál es la prioridad #1? | Continuidad operativa: alta disponibilidad, especialmente entre Panamá y Colón |
| ¿Qué tecnologías se consideran (sin definir aún)? | VPN/SD-WAN entre sedes, redundancia de Internet, Wi-Fi industrial en Colón, seguridad perimetral por capas |

### Checklist de cierre de la fase Prepare

Antes de avanzar a la siguiente fase, esta lista permite verificar que no se omitió nada esencial:

- [x] ¿Se describió la necesidad de negocio en términos de operación diaria, no solo en términos técnicos?
- [x] ¿Se justificó la inversión con costo del problema, costo de la solución, beneficio esperado y riesgo de no hacerlo?
- [x] ¿Se listaron tecnologías candidatas como familias de solución, sin comprometerse aún a ninguna en particular?
- [x] ¿Se identificó la sede crítica del negocio, para poder priorizar la continuidad operativa más adelante?
- [x] ¿Se documentaron los supuestos de tamaño, presupuesto y restricciones geográficas del proyecto?
- [x] ¿Se redactó un resumen ejecutivo que pueda presentarse a alguien no técnico?

---

## Siguiente paso: fase Plan

Con la fase Prepare cerrada, el proyecto ya tiene un "por qué" sólido. La fase Plan tomará este punto de partida y lo convertirá en una lista concreta de requisitos técnicos. Será necesario levantar, entre otros:

- El número exacto de usuarios y equipos por sede (afinando los supuestos de la sección 3.1).
- Los proveedores de Internet disponibles en cada ubicación, especialmente en el Interior.
- Las restricciones de espacio físico y cableado en cada sede (oficina vs. bodega).
- Un presupuesto detallado, desglosado por sede y por tipo de inversión.
- El inventario de lo que ya existe hoy, si es que algo existe.

Todo esto con el objetivo de que la fase Design —donde se definen las VLANs, la topología y los protocolos concretos— no parta de cero, sino de datos ya medidos y validados.
