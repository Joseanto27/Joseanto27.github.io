# Configuración de Servidor XMPP con Ejabberd

Este repositorio documenta el proceso detallado de instalación, configuración y resolución de incidencias para poner en marcha un servidor de mensajería instantánea **ejabberd** en Ubuntu, permitiendo la comunicación con clientes multiplataforma (Windows y Linux).

## 📋 Requisitos del Sistema

* **Servidor:** Ubuntu Desktop / Server.
* **Cliente 1:** Usuario `admin` configurado en la máquina local.
* **Cliente 2:** Windows 11 (Usuario `punky`).
* **Red:** Red local con configuración de IP estática.

---

## 🚀 1. Instalación y Configuración Inicial

### Instalación del Servidor
```bash
sudo apt update
sudo apt install ejabberd
```
### Configuración del Dominio Virtual
Se editó el archivo principal de configuración `/etc/ejabberd/ejabberd.yml` para establecer la identidad del servidor:

* **Dominio:** `erjoay.local`
* **Administrador:** `admin@erjoay.local`

### Registro de Usuarios
Se utilizaron los siguientes comandos para dar de alta a los usuarios en la base de datos del servidor:

```Bash
sudo ejabberdctl register admin erjoay.local jose1234
sudo ejabberdctl register punky erjoay.local jose1234
```

---

## 🌐 2. Configuración de Red (IP Estática)
Para garantizar la estabilidad del servicio y evitar que los clientes pierdan la conexión al reiniciar, se fijó la IP del servidor mediante **Netplan**.

* **Archivo de configuración:** `/etc/netplan/01-network-manager-all.yaml`

* **IP fija asignada:** `192.168.109.48`

* **Puerta de enlace:** `192.168.109.1`

### Comando para aplicar la red:

```Bash
sudo netplan apply
```

---

## 💻 3. Configuración del Cliente en Windows 11
Para conectar desde un sistema externo, se realizaron los siguientes pasos en el equipo cliente:

### Modificación del archivo Hosts
Se añadió la resolución de nombres local en `C:\Windows\System32\drivers\etc\hosts`:

```Bash
192.168.109.48   erjoay.local
```

### Instalación de Pidgin vía Terminal
```Bash
winget install Pidgin.Pidgin
```

### Ajustes de Conexión en Pidgin
En la pestaña **Avanzadas**, se configuraron los parámetros necesarios para localizar el servidor:

* **Seguridad de la conexión:** `Usar cifrado si está disponible.`

* **Puerto:** `5222`

* **Conectar con el servidor:** `192.168.109.48`

---

### ⚠️ 4. Resolución de Incidencias (Log de Errores)
A lo largo de la actividad se identificaron y solventaron los siguientes problemas:
| Incidencia | Descripción | Solución Aplicada |
| :--- | :--- | :--- |
| **Error** `ERR_EMPTY_RESPONSE` | El navegador no cargaba el panel de administración tras cambiar la IP. | Se forzó el cierre de procesos "zombie" de Erlang (`pkill -9 beam`) y se reinició el servicio. |
| `No such file or directory` | Error al intentar listar el archivo de hosts como si fuera una carpeta. | Se corrigió el uso del comando `ls` por `nano` o `cat` para editar el archivo directamente. |
| **Bloqueo de** `systemctl` | El servicio no terminaba de arrancar tras cambiar el dominio. | Se realizó un arranque en modo "live" (`ejabberdctl live`) para depurar errores de sintaxis en el archivo `.yml`. |
| **Fallo de resolución DNS** | El cliente Windows no encontraba `erjoay.local`. | Se editaron los permisos de administrador en Windows para añadir la IP correcta al archivo `hosts`. |
