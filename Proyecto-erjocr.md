# 📑 Documentación de Proyecto: Infraestructura y Seguridad de Red

Este documento detalla la base técnica sobre la que se sustenta el laboratorio de servicios. Aquí se establece la comunicación, la seguridad perimetral y la visibilidad de los nodos internos.

---

## 📖 1. Resumen del Escenario
Se ha desplegado un entorno virtualizado en **Proxmox VE** que emula una arquitectura de red corporativa segmentada. El diseño se basa en la separación de tráfico mediante un **Router Linux** que actúa como frontera, protegiendo un **Nodo Cliente** que reside en una red privada aislada de Internet directo.



---

## 🎯 2. Objetivos del Proyecto
* **Segmentación de Red:** Aislar los servicios críticos en una red local (**LAN**) inaccesible desde el exterior por defecto.
* **Seguridad Perimetral:** Implementar un cortafuegos basado en **IPTables** para controlar el flujo de datos.
* **Gestión de Tráfico:** Configurar **NAT** (Network Address Translation) para permitir la salida a Internet y **Port Forwarding** para la administración remota.
* **Preparación de Servicios:** Establecer la conectividad necesaria para el despliegue posterior de servicios de mensajería y servidores web.

---

## 🗄️ 3. Recursos Disponibles (Lo que hay)

La infraestructura se compone de los siguientes elementos técnicos:

| Elemento | Descripción | Especificaciones Técnicas |
| :--- | :--- | :--- |
| **Hipervisor** | Proxmox VE | Host físico que gestiona la virtualización. |
| **Bridges (Switches)** | `vmbr0` y `vmbr1` | Capas de red para separar tráfico **WAN** (Público) y **LAN** (Privado). |
| **VM Router** | Puerta de Enlace | Ubuntu Server, 2 NICs, IP Forwarding habilitado. |
| **VM Cliente** | Nodo de Servicios | Ubuntu Server, 1 NIC, IP estática `10.10.10.2`. |
| **Software Base** | Linux Networking | Netplan, IPTables, Sysctl. |

---

# 🛡️ Configuración del Nodo Router (Gateway)

El Router actúa como un **Firewall Perimetral** y **Gateway**, gestionando el tráfico entre la red externa (WAN) y la red privada de confianza (LAN).

---

## 1. Arquitectura de Red
La máquina dispone de dos interfaces de red configuradas mediante **Netplan**, permitiendo la segmentación de tráfico:

* **ens18 (WAN):** Conectada al puente `vmbr0` de Proxmox. Obtiene conectividad externa (Internet/Red Real) mediante DHCP o IP estática de la red física.
* **ens19 (LAN):** Conectada al puente `vmbr1` (Red Interna). Actúa como puerta de enlace (Gateway) para los clientes internos con la IP fija `10.10.10.1`.



---

## 2. Configuración del Sistema (IP Forwarding)
Para que el Router permita el paso de paquetes entre ambas interfaces (de la red externa a la interna y viceversa), se habilitó el reenvío de IP (**IP Forwarding**) en el núcleo de Linux. Sin esto, el Router actuaría como un muro y no como un puente.

### Habilitación temporal:
```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

## 3. Configuración de IPTables (Tablas NAT)

Para gestionar el flujo de datos entre la red pública y la privada, se implementaron reglas de **Traducción de Direcciones de Red (NAT)**. Estas reglas permiten tanto la salida controlada a internet como la exposición selectiva de servicios.

### Salida a Internet (Masquerade / SNAT)
Esta regla permite que los nodos de la red interna (como el servidor de **Eric, Jose y Chris**) tengan acceso a internet. El router "enmascara" la IP privada del cliente (`10.10.10.2`) y utiliza su propia IP pública para realizar las peticiones externas.

```bash
# Habilitar el enmascaramiento de salida por la interfaz WAN
sudo iptables -t nat -A POSTROUTING -o ens18 -j MASQUERADE
```

### 📝 Tabla de Redireccionamiento de Puertos (DNAT)

A continuación se detallan las reglas de **Port Forwarding** configuradas en el Router para dar acceso externo a los servicios del Cliente (`10.10.10.2`):

| Servicio | Puerto Externo (Router) | Puerto Interno (Cliente) | Comando de Configuración (Ejecutar en Router) |
| :--- | :---: | :---: | :--- |
| **HTTP (Nginx)** | `80` | `80` | `sudo iptables -t nat -A PREROUTING -i ens18 -p tcp --dport 80 -j DNAT --to-destination 10.10.10.2:80` |


### Persistencia de las Reglas
Por defecto, las reglas de **IPTables** son volátiles y se borran tras un reinicio del sistema. Para garantizar que la configuración de red se mantenga activa permanentemente, se utiliza el paquete `iptables-persistent`:

```bash
# Instalación del servicio y captura de reglas actuales
sudo apt update && sudo apt install iptables-persistent -y

# Guardar cambios manuales realizados posteriormente
sudo netfilter-persistent save
```

### Verificación de Reglas Activas

Es fundamental auditar que el tráfico está siendo redirigido correctamente. El siguiente comando permite observar los **contadores de paquetes** en tiempo real, lo que confirma que las reglas están detectando y procesando el tráfico de entrada y salida:

```bash
# Listar reglas de la tabla NAT detalladamente con números de línea
sudo iptables -t nat -L -n -v --line-numbers
```
## 4. Comandos de Verificación

Para auditar que la configuración de red es correcta, el tráfico fluye sin bloqueos y las reglas de **IPTables** están procesando paquetes activamente, se utilizan los siguientes comandos de diagnóstico:

### Auditoría de Tráfico y Reglas NAT
Este comando es vital para comprobar si las redirecciones de puertos (DNAT) y el enmascaramiento (MASQUERADE) están funcionando. Permite ver cuántos paquetes y bytes han pasado por cada regla.

```bash
# Listar todas las reglas NAT activas con contadores de paquetes y números de línea
sudo iptables -t nat -L -n -v --line-numbers
```

### Comprobación de Conectividad (Desde el Cliente)

Para asegurar que el Router está cumpliendo correctamente su función de **Gateway** y que el **IP Forwarding** es efectivo, se deben realizar pruebas de conectividad desde el nodo interno (Cliente) hacia el exterior. Esto confirma que el tráfico atraviesa el Router de forma fluida.

```bash
# Probar salida directa a internet (Ping a los DNS de Google)
ping 8.8.8.8

# Probar la resolución de nombres DNS para asegurar la navegación web
nslookup google.com
```

## 🏁 5. Conclusiones y Logros Obtenidos

Con la implementación de esta infraestructura, hemos pasado de tener máquinas virtuales aisladas a un **entorno de red profesional, seguro y segmentado**. Los hitos alcanzados son:

* **Seguridad por Aislamiento:** La red interna (`10.10.10.0/24`) es ahora invisible para cualquier usuario externo, reduciendo drásticamente la superficie de ataque del Cliente.
* **Control Total del Tráfico:** Mediante **IPTables**, el administrador tiene el poder de decidir exactamente qué paquetes entran (DNAT) y qué máquinas pueden salir a Internet (Masquerade).
* **Conectividad Transparente:** El nodo Cliente, a pesar de estar en una red privada, opera como si tuviera conexión directa gracias al trabajo de enrutamiento del Gateway.
* **Persistencia Garantizada:** Gracias a `iptables-persistent`, la arquitectura de red es resiliente y sobrevivirá a reinicios del servidor, eliminando la necesidad de reconfiguración manual.

En resumen, el escenario está **listo y validado** para actuar como cimiento de cualquier servicio corporativo que se decida desplegar, garantizando que la comunicación fluya solo por donde nosotros hemos definido.
