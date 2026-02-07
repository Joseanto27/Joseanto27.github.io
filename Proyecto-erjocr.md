# Guía de Configuración y Gestión de Proxmox VE

Este repositorio contiene la documentación técnica detallada para la implementación y administración de **Proxmox Virtual Environment**.

---

## 1. Configuración de Proxmox

### 💻 Requisitos de Hardware y Software
| Componente | Requisito Mínimo | Recomendado |
| :--- | :--- | :--- |
| **CPU** | 64-bit (Intel VT/AMD-V) | Intel Xeon / AMD EPYC |
| **RAM** | 2 GB + consumo de VMs | 8 GB o más (ECC) |
| **Disco** | 20 GB (SO + ISOs) | SSD/NVMe de alto rendimiento |
| **Red** | 100 Mbps | 1 Gbps o 10 Gbps |

---

## 2. Gestión de Redes y Almacenamiento

### ¿Qué es el `vmbr0`?
Es un **Linux Bridge** (puente virtual) que actúa como un **switch virtual** interno conectado a la tarjeta de red física (**NIC**).

---

## 3. Contenedores y Almacenamiento

### 📦 LXC vs. Docker
* **LXC (Linux Containers):** Se comporta como una "máquina ligera". Proxmox lo gestiona de forma nativa.
* **Docker:** Diseñado para microservicios y aplicaciones aisladas.

---

## 4. 🌐 Configuración de la Red Interna

### 🖧 Configuración de la VM Cliente (Netplan)
* **IP asignada:** `192.168.109.46` (vía DHCP).
* **DHCP:** Permite que el cliente reciba automáticamente la puerta de enlace y DNS.

### 🔐 Configuración de NAT en el Router
Para que el servicio web del cliente sea accesible desde Internet, usamos `iptables` en el router:

| Comando | Función |
| :--- | :--- |
| `iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.109.46:80` | Redirige tráfico del puerto 80 a la VM cliente. |
| `iptables -A FORWARD -p tcp -d 192.168.109.46 --dport 80 -j ACCEPT` | Permite el paso de paquetes a través del router. |
| `echo 1 > /proc/sys/net/ipv4/ip_forward` | Habilita el reenvío de paquetes en el sistema. |

---

## 🖥 Instalación de Nginx en la VM Cliente

1. **Actualizar repositorios:**
   
   ```bash
   sudo apt update

3. **Instalar el servidor web:**
   ```bash
   sudo apt install nginx -y

4. **Instalar el servidor web:**
   ```bash
   sudo systemctl status nginx


## ⚠️ 6. Incidencias Comunes

1. **Olvidar activar el IP Forwarding:**
   ```bash
   echo 1 > /proc/sys/net/ipv4/ip_forward

2. **Conflictos con el Firewall de Proxmox:**

  A veces las reglas de iptables son correctas, pero el firewall integrado de Proxmox (a nivel de Datacenter o Nodo) está bloqueando el tráfico en el puerto 80.

3. **Persistencia de reglas de red:**

  Los cambios en el archivo `/etc/network/interfaces` requieren un reinicio del servicio de red o del nodo para aplicarse correctamente.

4. **Error de Gateway en la VM Cliente:**

  Si la VM cliente no tiene configurada la IP interna del Router como su puerta de enlace (Gateway), podrá recibir paquetes pero no podrá responder hacia Internet.
