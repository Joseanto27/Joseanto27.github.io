# Guía de Configuración y Gestión de Proxmox VE

Este repositorio contiene la documentación técnica para la implementación de **Proxmox Virtual Environment**.

---

## 1. Configuración de Proxmox

### 💻 Requisitos de Hardware y Software
| Componente | Requisito Mínimo | Recomendado |
| :--- | :--- | :--- |
| **CPU** | 64-bit (Intel VT/AMD-V) | Intel Xeon / AMD EPYC |
| **RAM** | 2 GB + consumo de VMs | 8 GB o más (ECC) |
| **Disco** | 20 GB (SO + ISOs) | SSD/NVMe de alto rendimiento |
| **Red** | 100 Mbps | 1 Gbps o 10 Gbps |

### 🌐 Requisitos de Red
Proxmox requiere una **IP estática** para garantizar la accesibilidad permanente:
* **IP:** Dirección única dentro de nuestro rango de red.
* **Máscara de subred:** Define el tamaño y alcance de la red.
* **Gateway:** IP del router para salida a internet.

---

## 2. Gestión de Redes y Almacenamiento

### ¿Qué es el `vmbr0`?
Es un **Linux Bridge** (puente virtual) que actúa como un **switch virtual** interno.
* **Función:** Se asocia a la tarjeta de red física (**NIC**).
* **Objetivo:** Permite que las VMs y el Router instalado se comuniquen entre sí y con el exterior usando una sola conexión física.

---

## 3. Contenedores y Almacenamiento

### 📦 LXC vs. Docker
> **LXC (Linux Containers):** Se comporta como una "máquina ligera". Tiene su propio sistema de archivos y gestión de usuarios. Proxmox lo gestiona de forma **nativa**.
> 
> **Docker:** Diseñado para microservicios (ejecutar una sola aplicación o proceso).

### 📂 Ubicación de la Información
* `/etc/pve/`: Configuración sincronizada entre nodos (sistema *pmxcfs*).
* `/var/lib/vz/`: Almacenamiento de archivos ISO y plantillas.

---



| Término | Definición |
| :--- | :--- |
| **Datacenter** | Nivel más alto de jerarquía para gestionar múltiples nodos. |
| **Summary** | Panel con gráficos de CPU, RAM y Red en tiempo real. |
| **Shell** | Terminal de comandos integrada en el navegador. |
| **Node pve** | Nombre físico del servidor individual. |
| **LVM** | Gestor de volúmenes lógicos para discos flexibles. |
| **LVM-Thin** | Variante que solo ocupa espacio real a medida que la VM escribe datos. |
| **ZFS** | Sistema de archivos avanzado con protección de datos y RAID nativo. |


## 5. 🌐 Configuración de la Red Interna

Para permitir que una VM cliente sea accesible desde fuera de la red interna, configuramos un router virtual con dos interfaces en Proxmox:

Interface interna: conecta el router con la VM cliente.

Interface externa: conecta el router a la red externa (Internet).

🖧 Configuración de la VM Cliente (Netplan)

La VM cliente utiliza DHCP en la interfaz ens18:

network:
  ethernets:
    ens18:
      dhcp4: true
  version: 2


IP asignada: 192.168.109.46 (por DHCP)

DHCP: Permite que el cliente reciba automáticamente la puerta de enlace y DNS desde el router.

🔐 Configuración de NAT en el Router

Para que la web del cliente sea accesible desde Internet, usamos iptables en el router:

Comando	Función
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.109.46:80	Redirige todo el tráfico entrante en el puerto 80 hacia la VM cliente.
iptables -A FORWARD -p tcp -d 192.168.109.46 --dport 80 -j ACCEPT	Permite que los paquetes redirigidos atraviesen el router.
echo 1 > /proc/sys/net/ipv4/ip_forward	Habilita el reenvío de paquetes en el router.

✅ Con esto, cualquier solicitud HTTP que llegue al router se dirige automáticamente a la VM cliente.

🖥 Instalación de Nginx en la VM Cliente

Actualizamos los repositorios:

sudo apt update


Instalamos Nginx:

sudo apt install nginx -y


Comprobamos que el servicio esté activo:

sudo systemctl status nginx


La web estará disponible en el puerto 80 de la VM.

Desde la red externa, accedemos usando la IP pública del router, gracias al NAT configurado.

🔎 Resumen del Flujo de Datos

El cliente externo realiza una solicitud HTTP al router.

El router, mediante NAT, redirige el tráfico al puerto 80 de la VM cliente.

La VM cliente responde con la web alojada en Nginx.

La respuesta regresa al cliente externo, completando la conexión.

