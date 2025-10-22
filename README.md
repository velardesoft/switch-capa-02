# CONFIGURACIÓN

<img width="1435" height="672" alt="Image" src="https://github.com/user-attachments/assets/4cb960b7-c508-457c-bb45-2a3afed31c04" />

## Paso a paso: Configuración para tu topología
Este guía usa el router-on-a-stick: el router realiza el enrutamiento entre VLANs. Todos los switches, incluidos los multilayer, operan como switches capa 2 (sin ip routing y sin SVIs para datos). La VLAN 99 es nativa y administrativa.

# 1. Switches de Acceso (2960)
Haz este proceso para cada uno, cambiando el número de VLAN y la IP de la SVI de administración (VLAN 99):

### Ejemplo para Switch "Ventas" (VLAN 10)

```bash
conf t
vlan 10
 name VENTAS
vlan 99
 name NATIVE

interface range fa0/1-2
 switchport mode access
 switchport access vlan 10
exit

interface gig0/1
 switchport mode trunk
 switchport trunk native vlan 99
exit

interface vlan 99
 ip address 192.168.99.2 255.255.255.0
 no shutdown
exit
ip default-gateway 192.168.99.1
```

- Switch "Admin" (VLAN 20): Cambia los números a vlan 20, acceso en 20, y SVI a 192.168.99.3.

- Switch "Logistica" (VLAN 30): vlan 30, acceso en 30, SVI 192.168.99.4.

- Switch "Marketing" (VLAN 40): vlan 40, acceso en 40, SVI 192.168.99.5.

# 2. Switches Multilayer (3560, capa 2)
No uses ip routing ni SVIs para datos. Sólo configura VLAN y troncal:

```bash
conf t
vlan 10
vlan 20
vlan 99
interface range gig1/0/1-2
 switchport mode trunk
 switchport trunk native vlan 99
exit
interface gig1/0/3
 switchport mode trunk
 switchport trunk native vlan 99
exit
```

- Para el otro multilayer, repite pero usando vlan 30, 40 y 99 según los switches conectados.

No configures ip routing ni SVIs para VLANs de usuario.


# 3. Multilayer Central (3560, capa 2)

Este une los anteriores y conecta al router. Configura todas las VLANs y troncales:

```bash
conf t
vlan 10
vlan 20
vlan 30
vlan 40
vlan 99

interface range gig1/0/1-2
 switchport mode trunk
 switchport trunk native vlan 99
exit
interface gig1/0/3
 switchport mode trunk
 switchport trunk native vlan 99
exit
```

- No configures SVI (excepto administración si lo requieres). No uses ip routing.

# 4. Router (2901)

Aquí configuras las subinterfaces dot1q para cada VLAN. Este router será el gateway de cada VLAN.

```bash
interface g0/1
no ip address
no shutdown
exit
```

configuración


```bash 
conf t
interface g0/1.10
 encapsulation dot1q 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
interface g0/1.20
 encapsulation dot1q 20
 ip address 192.168.20.1 255.255.255.0
 no shutdown
interface g0/1.30
 encapsulation dot1q 30
 ip address 192.168.30.1 255.255.255.0
 no shutdown
interface g0/1.40
 encapsulation dot1q 40
 ip address 192.168.40.1 255.255.255.0
 no shutdown
interface g0/1.99
 encapsulation dot1q 99 native
 ip address 192.168.99.1 255.255.255.0
 no shutdown
exit
```

# 5. Configuración de los hosts (PCs)

Cada PC debe tener como gateway la IP correspondiente de su VLAN en el router.

- VLAN 10 (Ventas): Gateway 192.168.10.1

- VLAN 20 (Admin): Gateway 192.168.20.1

- VLAN 30 (Logistica): Gateway 192.168.30.1

- VLAN 40 (Marketing): Gateway 192.168.40.1

La dirección IP de cada PC en su rango, máscara /24.

# 6. Resumen de pruebas y recomendaciones

- Revisa que TODOS los enlaces entre switches y router estén en modo trunk correctamente.

- NO uses ip routing ni SVIs (excepto administración) en multilayer ni switches de acceso.

- Verifica que los PCs tengan el gateway del router.

- Realiza pruebas ping entre PCs de distintas VLANs.

- Si no hay conectividad, revisa:

- - Estado de interfaces (show interfaces trunk)

- - Tablas de VLAN (show vlan brief)

- - Tablas de rutas en el router (show ip route)



--- 

# POSIBLES ERRORES SEA EN ROUTER O SWITCH

```bash
conf t
interface gig1/0/3
no shutdown
exit
```

# Enlace troncal mal configurado

El puerto que conecta el switch con el router debe estar en modo trunk y permitir todas las VLANs necesarias.

```bash
interface gig1/0/3
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan all
 no shutdown
```

# FIN



