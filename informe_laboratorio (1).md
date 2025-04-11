# 🧪 Laboratorio Avanzado de SSH y Seguridad en Linux

**Nombres:** María Camila Hormaza Ruales / Alejandro Castrillón Buitrón  
**Fecha:** 11 Abril 2025  
**Máquinas utilizadas:**

- Ubuntu Server
- Ubuntu Cliente
- Windows 11 (con PuTTY)

---

## Desafío 1: Configuración Segura de SSH

### Instalación del servidor SSH

Se ejecutó:

```bash
sudo apt update
sudo apt install openssh-server
```

Resultado:

![Instalación SSH](capturas/ssh_instalado.png)

**Verificación del servicio SSH:**

```bash
sudo systemctl status ssh
```

Resultado:

![Estado del servicio SSH](capturas/estado_ssh.png)

Si está fallando:

```bash
journalctl -u sshd
```

Para validar

```bash
sudo systemctl status ssh
```

Resultado:

```bash
● ssh.service - OpenBSD Secure Shell server
   Loaded: loaded (/usr/lib/systemd/system/ssh.service; disabled; preset: enabled)
   Active: inactive (dead)
   TriggeredBy: ● ssh.socket
```

Habilitarlo para que inicie con el sistema:

```bash
sudo systemctl enable ssh
```

Iniciar el servicio ahora mismo:

```bash
sudo systemctl start ssh
```

Verificar de nuevo:

```bash
sudo systemctl status ssh
```

Resultado:

```bash
Active: active (running)
```

### Hardening SSH (`/etc/ssh/sshd_config`)

Parámetros configurados:

```bash
Port 2222
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
LoginGraceTime 30
MaxAuthTries 3
MaxSessions 2

```

**Reinicio del servicio:**

```bash
sudo systemctl restart ssh
```

Se verificó con el comando:

```bash
ss -tulpn | grep sshd
```

Resultado:

```bash
LISTEN 0 496 *:2222
```

---

## Desafío 2: Conexiones cruzadas

### Desde Windows (PuTTY)

Conexión usando:

- IP del servidor: `192.168.1.69`
- Puerto: `2222`

![Conexión con PuTTY](capturas/putty_conexion.png)

# Informe - Desafío 3: Permisos y Seguridad Avanzada

## Objetivo

Aplicar combinaciones de permisos con `chmod` para controlar el acceso a archivos y carpetas, además de analizar su efecto en distintos contextos de seguridad.

---

## Estructura de Trabajo

- **Ruta utilizada**: `~/practica3ssh/s_operativos/mchormaza`
- **Archivo principal**: `sshlinux.txt`
- **Contenido del archivo**:

````bash
chmod 750 carpeta1
chmod 644 archivo1
chmod 600 archivo2
chmod 777 carpeta2
chmod 700 archivo3
chmod 1777 carpeta3
chmod 2750 carpeta4

---

chmod ug+x sshlinux.txt
chmod uo+wx sshlinux.txt
Resultado: Se modificaron correctamente los permisos del archivo sshlinux.txt.
chmod go-rx sshwindows.txt
chmod u=rwx,g=rw,o= sshwindows.txt
Resultado: No se modificaron los permisos debido a que estos comandos se usan en windows y se est accediendo remotamente a ubuntu.

# ANALSIS

chmod ug+x sshlinux.txt: otorga permisos de ejecución al usuario y grupo sobre el archivo.

chmod uo+wx sshlinux.txt: permite escritura y ejecución al usuario y a otros (everyone else).

La verificación con ls -l mostró los cambios en los permisos correctamente.

## Desafío 4: Usuarios y Grupos

Comandos ejecutados:

## Objetivo

Gestionar la creación de grupos y usuarios, asignarlos a grupos específicos, modificar propiedades de cuentas y aplicar restricciones de acceso dentro de un sistema Linux.

---

## 1. Creación de Grupos

```bash
sudo groupadd gruposistemas
sudo groupadd gruppoperativos
Se crean los grupos gruposistemas y gruppoperativos.

Se crean los grupos gruposistemas y gruppoperativos.

```bash
getent group | grep grupo
Se verifica la existencia de los grupos.

Se crean los usuarios:
```bash
sudo useradd -m -G gruposistemas mchormaza       # Ya existía
sudo useradd -m -G gruppoperativos alecastrillon
sudo useradd -m -G gruposistemas,gruppoperativos micPosada
sudo useradd -m -G gruppoperativos nruales
Se crea correctamente a alecastrillon, micPosada y nruales.

El usuario mchormaza no se crea porque ya existe.

Se asignan contraseñas:
```bash
sudo passwd nruales
sudo passwd alecastrillon
sudo passwd micPosada
Se establecen contraseñas para los nuevos usuarios.

En el caso de micPosada, hubo un error de coincidencia que fue corregido.

Verificación de Usuarios y Grupos
```bash
cat /etc/passwd | grep alecastrillon
groups alecastrillon
grep gruposistemas /etc/group
grep gruppoperativos /etc/group
Se verifica que los usuarios y grupos fueron correctamente configurados

Eliminación de un Usuario de un Grupo
```bash
sudo gpasswd -d nruales gruppoperativos
Se elimina correctamente al usuario nruales del grupo gruppoperativos.

Cambio de Shell y Bloqueo de Usuario

sudo usermod -s /usr/sbin/nologin nruales
sudo usermod -L nruales
Se evita que nruales pueda iniciar sesión (nologin).

Se bloquea la cuenta del usuario.


Este desafío permitió practicar operaciones fundamentales de gestión de usuarios y grupos en sistemas Linux. Se configuraron correctamente grupos, usuarios, pertenencias y restricciones de acceso, asegurando una administración básica pero sólida del sistema.

Verificación:

```bash
cat /etc/group | grep grupo
```

![Grupos creados](capturas/grupos_creados.png)

---

## Desafío 5: Comparativa SSH vs FTP

### Captura de tráfico en Wireshark

- FTP muestra datos en texto plano.
- SSH encriptado (no se ve contenido).

![Wireshark FTP](capturas/wireshark_ftp.png)
![Wireshark SSH](capturas/wireshark_ssh.png)

Se realizó la instalación de Wireshark, y se abrió la aplicación de Putty para conectar con el servidor.
Se accede a Wireshark, se selecciona ethernet y se ejecutó en filtros  tcp.port == 2222, en la que se logró ver el protocolo ssh con codigos cifrados.

## Requisitos previos

Antes de iniciar la prueba de FTP con Wireshark, se aseguró cumplir con lo siguiente:

- Tu **Ubuntu Server** debe tener instalado y activo el servicio **FTP**:

```bash
sudo apt update
sudo apt install vsftpd
sudo systemctl start vsftpd

Se accede nuevamente a tcp.port == 21, en la cual ya permitía ver los usuarios y contraseñs expuestos.

---
# Comparativo: SSH vs FTP

## Objetivo

Analizar y comparar el nivel de cifrado y seguridad de los protocolos **SSH** y **FTP**, evidenciando sus diferencias mediante la captura de tráfico en Wireshark.

---

## ¿Qué es el cifrado?

El cifrado es el proceso de convertir datos legibles en un formato ilegible para protegerlos durante su transmisión. Es fundamental para evitar que terceros accedan a información sensible como contraseñas o archivos durante una conexión remota.

---

## FTP (File Transfer Protocol)

### Descripción
Protocolo clásico para transferencia de archivos. Diseñado antes de que la seguridad fuera una preocupación central.

### ¿Cifra los datos?
**No.** FTP transmite toda la información (incluyendo usuario y contraseña) en **texto plano**.

### Ejemplo en Wireshark
Se puede observar: USER mchormaza PASS miclave123

### Riesgos
- Exposición de contraseñas y archivos a través de sniffing.
- Vulnerable a ataques de suplantación y manipulación de datos.
- No ofrece validación de integridad ni autenticación fuerte.

### Ventajas
- Fácil de implementar.
- Muy extendido y compatible.

### Desventajas
- Inseguro en redes públicas o compartidas.
- Obsoleto para estándares modernos de ciberseguridad.

---

## SSH (Secure Shell)

### Descripción
Protocolo seguro para administración remota y transferencia de archivos (mediante SFTP o SCP).

### ¿Cifra los datos?
**Sí.** SSH cifra toda la comunicación usando criptografía simétrica, asimétrica e intercambio seguro de claves.

### Ejemplo en Wireshark
El contenido de los paquetes es ilegible. Solo se ve que el protocolo es `SSH` pero **no se muestra ningún dato sensible**.

### Seguridad
- Cifrado fuerte (AES, RSA, etc.).
- Soporte para autenticación por clave pública.
- Protección contra ataques MITM y sniffing.

### Ventajas
- Altamente seguro y confiable.
- Versátil: permite túneles, redirección de puertos, automatización.
- Usado ampliamente en servidores Linux.

### Desventajas
- Configuración inicial más técnica.
- Requiere más recursos que FTP.

---

## Comparación técnica: FTP vs SSH

| Característica               | FTP                         | SSH                          |
|-----------------------------|----------------------------- |------------------------------|
| ¿Cifra la sesión?           | ❌ No                        | ✅ Sí                         |
| ¿Cifra credenciales?        | ❌ No                        | ✅ Sí                         |
| ¿Cifra transferencia?       | ❌ No                        | ✅ Sí                         |
| ¿Protocolos relacionados?   | FTP                          | SSH, SFTP, SCP               |
| Seguridad                   | Baja (inseguro)              | Alta (cifrado extremo a extremo) |
| Capturable con Wireshark    | ✅ Usuario y clave visibles  | ❌ Solo datos cifrados       |
| Usabilidad                  | Fácil, pero desactualizado    | Seguro, moderno              |
| Recomendado para producción | ❌ No                         | ✅ Sí                         |

---

## Conclusión

- **FTP** es un protocolo inseguro para cualquier entorno que no sea completamente privado y controlado. Su uso está desaconsejado en redes modernas sin una capa adicional de seguridad (como FTPS).
- **SSH** es el estándar actual para conexiones seguras gracias a su cifrado, integridad y autenticación robusta.
- En ambientes de producción o redes expuestas, **SSH es obligatorio** para garantizar la seguridad de usuarios, datos y sistemas.

---

# Informe: Ventajas, Funcionalidad y Relevancia del Protocolo SSH

## Introducción

El protocolo **SSH (Secure Shell)** es una herramienta esencial en el mundo de los sistemas operativos y la administración remota de servidores. Desde su aparición en los años 90, ha reemplazado a protocolos inseguros como **TELNET** y **FTP**, gracias a su capacidad de proporcionar **comunicación cifrada**, **autenticación segura** y **túneles protegidos** entre sistemas remotos.

---

## ¿Cuál es la funcionalidad real de SSH?

SSH permite:
- Conectarse de forma **remota y segura** a un servidor.
- Ejecutar comandos a través de un canal cifrado.
- Transferir archivos con **SFTP** o **SCP**.
- Redirigir puertos o montar túneles para conexiones protegidas.
- Automatizar tareas y mantener scripts seguros sin exponer credenciales.

> SSH es una **puerta segura** hacia otro sistema, sin importar si está en la misma red o en la nube.

---

## Beneficios clave del uso de SSH

### 1. **Cifrado de extremo a extremo**
Todos los datos transmitidos por SSH están cifrados. Esto significa que incluso si un atacante intercepta el tráfico, no podrá leer su contenido.

### 2. **Autenticación segura**
SSH permite autenticación por contraseña o por **claves públicas/privadas**, que son mucho más seguras que las contraseñas tradicionales.

### 3. **Integridad y protección**
SSH no solo cifra los datos, también garantiza que **no han sido alterados** durante el tránsito (gracias a verificaciones de integridad).

### 4. **Versatilidad**
SSH es más que una conexión remota:
- Puedes montar túneles (tunneling).
- Redirigir puertos (port forwarding).
- Montar conexiones seguras para otros servicios (como MySQL, HTTP, etc.).

### 5. **Automatización segura**
Con SSH puedes automatizar backups, despliegues y tareas administrativas usando scripts o herramientas como **Ansible**.

---

## ¿Por qué SSH es importante?

En un mundo donde la **ciberseguridad** es una prioridad, SSH representa una capa básica y necesaria. Usarlo correctamente evita:

- El robo de credenciales.
- El acceso no autorizado a sistemas críticos.
- La exposición de archivos confidenciales en redes públicas o inseguras.

En entornos empresariales, es la **herramienta estándar para la gestión de servidores Linux y la nube**.

---

## Comparación rápida con otros protocolos

| Protocolo | ¿Cifra datos? | ¿Autenticación segura? | ¿Uso moderno? |
|-----------|----------------|-------------------------|----------------|
| TELNET    | ❌ No           | ❌ No                    | ❌ Obsoleto     |
| FTP       | ❌ No           | ❌ No                    | ❌ Inseguro     |
| SSH       | ✅ Sí           | ✅ Sí                    | ✅ Estándar     |

---

## Para concluir:

SSH no es solo una herramienta de conexión remota, es una **pieza fundamental de la infraestructura moderna**. Su capacidad para ofrecer **seguridad, autenticación, versatilidad y eficiencia** lo convierten en una herramienta imprescindible para administradores de sistemas, desarrolladores y operadores de nube.

> Usar SSH no es una opción: es una buena práctica de seguridad obligatoria en cualquier entorno profesional o educativo donde se gestionen sistemas remotos.

---

````
