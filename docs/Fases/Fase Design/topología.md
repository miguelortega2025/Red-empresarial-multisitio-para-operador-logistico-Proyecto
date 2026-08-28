# Avance del Proyecto de Red Corporativa Multi-Sitio (Cisco Packet Tracer)

## Estado Actual: Sede Principal (Panamá - PTY) — Topología Física Validada

Se ha completado la validación y estructuración de la **Topología Física de la Sede Principal (Panamá)** respetando la arquitectura jerárquica de 3 capas (Core, Distribución y Acceso), garantizando alta disponibilidad, redundancia perimetral y segmentación lógica.

---

## 1. Diagrama de Arquitectura de Red (Sede Panamá)

```text
                       [ ISP-A ]      [ ISP-B ]
                           \            /
                            \          /
                             [ FW-PTY ]
                             /        \
                            /          \
                   [ SW-CORE-PTY-1 ] <----> [ SW-CORE-PTY-2 ]
                      /        \              /        \
                     /          \            /          \
            [ SW-DIST-PTY-1 ] <-------------> [ SW-DIST-PTY-2 ]
               /   |   |   \                     /   |   |   \
             [WLC] |   |   +--------------------+    |   |    \
                   |   +--------------------------+   |     \
                   |                                  |      \
            [ SW-ACC-PTY-1 ]  [ SW-ACC-PTY-2 ]  [ SW-ACC-PTY-3 ]  [ SW-ACC-PTY-4 ]
