<img src="https://raw.githubusercontent.com/Joseanto27/Joseanto27.github.io/main/IMG-20231116-WA0007%20(1).jpg" width="200">

# **José Antonio Pineda Fernández**
## Objetivo Profesional
Tengo gran afición al montaje y desmontaje de mis propios dispositivos electrónicos cosa que me ha ayudado ha tener experiencia en este ámbito. 
Así como también en sistemas operativos Windows y Linux. 
También tengo experiencia  en herramientas ofimáticas como Excel, Word, PowerPoint y Access.
Mi objetivo se centra en trabajar en el área de mantenimiento de equipos.

## Formación
### 2018 - 2022 ESO
Centre Educatiu Escola Balaguer,L'Hospitalet del Llobregat.
• Con un promedio de un 6 aprobé la ESO.
### 2022-2024 - Actualidad,Ciclo medio Sistemas Microinformáticos y Redes
 IFP planeta Formacion Profesional, L'Hospitalet del Llobregat.

## Experiencia
Si bien no dispongo de recorrido o no tengo directamente experiencia
estoy en proceso de mejora y aprender.

## Competencias personales
Soy una persona trabajadora la cual le gusta trabajar en equipo, tambien soy organizado y
trabajador.

## Otros datos de interés
Dispongo de tiempo para trabajar por las tardes por lo cual puedo
adaptarme con facilidad a las exigencias que en su momento se den.
Si bien el tiempo es una de las cosas que menos me preocupan, he de
decir que los martes y jueves hago 1h y 45min de futbol. Pero estas se
pueden adaptar perfectamente a mis prácticas, a fin de aprovechar
bien el tiempo.

### Idiomas
| Idioma | Nivel |
|:---:|   :---:|
|Castellano | Nivel Alto|
|Catalán| Nivel medio|
|Inglés| Nivel bajo|





coleee


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
