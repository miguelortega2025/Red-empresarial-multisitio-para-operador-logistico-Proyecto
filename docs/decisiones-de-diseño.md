## Dimensionamiento de la Sede Central (Ciudad de Panamá)

### Contexto de negocio
Empresas de logística medianas en Panamá suelen tener entre 51-200 
empleados. Sin embargo, gran parte de esa fuerza operativa está en 
bodega/puerto (Colón), no en la sede administrativa. Por eso HQ se 
modeló como una operación más pequeña: ~40-50 personas en oficina.

### Estructura por piso
| Piso | Departamento | VLAN |
|------|-------------|------|
| 1 | Ventas + Invitados | 20, 99 |
| 2 | Administración/Gerencia | 10 |
| 3 | Operaciones | 30 |
| 4 | IT/Servidores | 40 |

### Por qué no modelar 200 empleados/dispositivos
El objetivo del proyecto es demostrar dominio de VLANs, trunking, 
inter-VLAN routing, STP y DHCP — no simular una empresa real dispositivo 
por dispositivo. Con 8-10 endpoints (PCs + IP Phones + servidor) ya se 
cubre la lógica de segmentación completa sin ruido visual innecesario.

### Departamentos y su justificación
- **Administración/Gerencia**: dirección, RRHH, finanzas
- **Ventas/Comercial**: atención a clientes, cotizaciones
- **Operaciones**: coordinación de flotas, tracking, enlace con 
  Colón/Interior — es el corazón operativo de una empresa logística
- **IT/Servidores**: soporte técnico, cuarto de servidores (DHCP/NTP/Syslog)
- **Invitados**: red aislada para visitas/auditores
