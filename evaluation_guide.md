# Born2BeRoot - Guia de Evaluacion

## Comandos para la Evaluacion entre Pares

Esta guia contiene todos los comandos necesarios durante la evaluacion/defensa. Cada seccion corresponde a lo que el evaluador comprobara.

---

## Verificaciones Iniciales

### Verificar la firma del disco

```bash
shasum <nombre_vdi>                  # Linux/Mac
certUtil -hashfile <nombre_vdi> sha1 # Windows
```

### Verificar que no hay interfaz grafica

```bash
ls /usr/bin/*session                 # No deberia devolver nada relacionado con X/wayland
dpkg -l | grep -i xorg               # No deberia devolver nada
dpkg -l | grep -i wayland            # No deberia devolver nada
```

### Verificar que los servicios estan funcionando

```bash
sudo service ufw status              # UFW debe estar activo
sudo service ssh status              # SSH debe estar activo
systemctl status apparmor            # AppArmor debe estar activo
systemctl status cron                # Cron debe estar activo
```

### Verificar el sistema operativo

```bash
uname -v                             # Muestra la version de Debian
uname -a                             # Informacion completa del sistema
cat /etc/os-release                  # Detalles del SO
```

---

## Hostname

### Comprobar el hostname actual

```bash
hostname                             # Debe terminar en 42 (ej., wil42)
```

### Modificar el hostname (durante la evaluacion si se solicita)

```bash
sudo hostnamectl set-hostname <nuevo_nombre>42
sudo nano /etc/hosts                 # Actualizar referencias
sudo reboot
```

---

## Particionado y LVM

### Verificar el esquema de particiones

```bash
lsblk                                # Muestra particiones y puntos de montaje
lsblk -f                             # Muestra tipos de sistema de archivos y UUIDs
```

### Verificar que LVM esta activo

```bash
lvdisplay                            # Muestra volumenes logicos
vgdisplay                            # Muestra grupos de volumenes
pvdisplay                            # Muestra volumenes fisicos
```

### Verificar el cifrado

```bash
lsblk                                # Comprobar entradas crypt/lvm
dmsetup ls                           # Muestra entradas de device-mapper
```

---

## Gestion de Usuarios y Grupos

### Listar todos los usuarios

```bash
cat /etc/passwd | grep -v nologin    # Usuarios con shell de inicio de sesion
```

### Comprobar grupos

```bash
getent group user42                  # Muestra miembros del grupo user42
getent group sudo                    # Muestra miembros del grupo sudo
```

### Verificar que tu usuario pertenece a user42 y sudo

```bash
id <tu_login>                        # Debe mostrar los grupos user42 y sudo
```

### Crear un nuevo usuario (durante la evaluacion)

```bash
sudo adduser <nuevo_usuario>         # Crea un nuevo usuario
sudo usermod -aG <nuevo_grupo> <nuevo_usuario>  # Anade el usuario a un grupo
```

### Crear un nuevo grupo (durante la evaluacion)

```bash
sudo addgroup <nuevo_grupo>          # Crea un nuevo grupo
getent group <nuevo_grupo>           # Verifica los miembros del grupo
```

---

## Politica de Contrasenas

### Comprobar la politica de caducidad de contrasenas

```bash
cat /etc/login.defs | grep PASS_     # Muestra PASS_MAX_DAYS, PASS_MIN_DAYS, PASS_WARN_AGE
chage -l <tu_login>                  # Muestra la caducidad de la contrasena del usuario
```

### Valores esperados

| Parametro | Valor | Descripcion |
|-----------|-------|-------------|
| `PASS_MAX_DAYS` | 30 | La contrasena expira cada 30 dias |
| `PASS_MIN_DAYS` | 2 | Minimo 2 dias entre cambios |
| `PASS_WARN_AGE` | 7 | Aviso 7 dias antes de la expiracion |

### Comprobar la complejidad de la contrasena

```bash
cat /etc/pam.d/common-password       # Muestra la configuracion de pam_pwquality
```

### Verificar los requisitos de contrasena

| Parametro | Requisito |
|-----------|-----------|
| `minlen=10` | Al menos 10 caracteres |
| `ucredit=-1` | Al menos 1 mayuscula |
| `lcredit=-1` | Al menos 1 minuscula |
| `dcredit=-1` | Al menos 1 numero |
| `maxrepeat=3` | No mas de 3 caracteres identicos consecutivos |
| `usercheck=1` | No debe contener el nombre de usuario |
| `diffok=7` | Al menos 7 caracteres nuevos (no aplica a root) |

### Cambiar una contrasena (durante la evaluacion)

```bash
sudo passwd <usuario>                # Cambiar contrasena del usuario
sudo passwd root                     # Cambiar contrasena de root
```

---

## Configuracion de Sudo

### Verificar que sudo esta instalado

```bash
which sudo                           # Ruta al binario de sudo
dpkg -s sudo                         # Detalles del paquete
```

### Comprobar la configuracion de sudo

```bash
sudo cat /etc/sudoers.d/sudo_config  # Muestra las reglas personalizadas de sudo
sudo visudo -c                       # Valida la sintaxis de sudoers
```

### Comprobar el directorio de logs de sudo

```bash
ls -la /var/log/sudo/                # Muestra los archivos de log de sudo
sudo cat /var/log/sudo/sudo.log      # Muestra los logs de comandos sudo
```

### Probar las restricciones de sudo

```bash
sudo -l                              # Lista los privilegios sudo del usuario
```

### Verificar las reglas de sudo

| Regla | Descripcion |
|-------|-------------|
| `passwd_tries=3` | Maximo 3 intentos |
| `badpass_message="..."` | Mensaje de error personalizado |
| `logfile="/var/log/sudo/sudo.log"` | Ubicacion del archivo de log |
| `log_input, log_output` | Archiva entradas y salidas |
| `requiretty` | Modo TTY activado |
| `secure_path="..."` | Rutas restringidas |

---

## Firewall (UFW)

### Comprobar el estado de UFW

```bash
sudo ufw status                      # Muestra las reglas activas
sudo ufw status verbose              # Estado detallado con politicas por defecto
sudo ufw status numbered             # Reglas con numeros
```

### Configuracion esperada

```
Status: active
4242/tcp    ALLOW    Anywhere        # Solo el puerto 4242 abierto
```

### Gestionar reglas (durante la evaluacion)

```bash
sudo ufw allow <numero_puerto>       # Anadir una regla
sudo ufw delete <numero_regla>       # Eliminar una regla (por numero)
sudo ufw reload                      # Recargar el firewall
```

---

## Servicio SSH

### Comprobar la configuracion de SSH

```bash
sudo cat /etc/ssh/sshd_config | grep -i port         # Muestra el puerto SSH (4242)
sudo cat /etc/ssh/sshd_config | grep -i permitroot   # Debe ser "no"
cat /etc/ssh/ssh_config | grep -i port               # Configuracion del cliente SSH
```

### Verificar que SSH esta instalado

```bash
which ssh                            # Ruta al cliente SSH
which sshd                           # Ruta al demonio SSH
dpkg -s openssh-server               # Detalles del paquete del servidor
```

### Probar la conexion SSH

```bash
ssh <tu_login>@localhost -p 4242     # Conectar como usuario regular (debe funcionar)
ssh root@localhost -p 4242           # Conectar como root (debe ser DENEGADO)
```

### Crear nuevo usuario y probar SSH (durante la evaluacion)

```bash
sudo adduser <nuevo_usuario>
ssh <nuevo_usuario>@localhost -p 4242  # Probar SSH con el nuevo usuario
```

---

## Cron y Script de Monitoreo

### Comprobar los cron jobs

```bash
sudo crontab -u root -l
# Debe mostrar: */10 * * * * /root/monitoring.sh
```

### Verificar que el script de monitoreo existe

```bash
ls -la /root/monitoring.sh           # El script debe existir y ser ejecutable
cat /root/monitoring.sh              # Ver el contenido del script
```

### Probar el script de monitoreo manualmente

```bash
sudo /root/monitoring.sh             # Ejecutar el script y comprobar la salida
```

### Salida esperada

```
#Architecture: Linux <hostname> <kernel> ...
#CPU physical : <numero>
#vCPU : <numero>
#Memory Usage: <usado>/<total>MB (<porcentaje>%)
#Disk Usage: <usado>/<total>Gb (<porcentaje>%)
#CPU load: <porcentaje>%
#Last boot: <fecha> <hora>
#LVM use: yes
#Connections TCP : <numero> ESTABLISHED
#User log: <numero>
#Network: IP <ipv4> (<mac>)
#Sudo : <numero> cmd
```

### Parar/arrancar cron (durante la evaluacion)

```bash
sudo /etc/init.d/cron stop           # Para el servicio cron
sudo /etc/init.d/cron start          # Inicia el servicio cron
sudo /etc/init.d/cron restart        # Reinicia el servicio cron
```

### Interrumpir el script sin modificarlo

```bash
sudo crontab -u root -e
# Comentar la linea: # */10 * * * * ...
# O bien:
sudo /etc/init.d/cron stop           # Parar el servicio cron
```

---

## AppArmor

### Verificar que AppArmor esta funcionando

```bash
sudo apparmor_status                 # Muestra los perfiles cargados
systemctl status apparmor            # Estado del servicio
```

### Verificar que AppArmor esta habilitado en el arranque

```bash
systemctl is-enabled apparmor        # Debe decir "enabled"
```

---

## Comandos Adicionales de Verificacion

```bash
# Comprobar servicios en ejecucion
systemctl list-units --type=service --state=running

# Comprobar puertos en escucha
ss -tlnp

# Comprobar el uso de disco
df -h
free -m

# Comprobar el tiempo de actividad del sistema
uptime
who -b

# Comprobar la configuracion de red
ip addr show
hostname -I
ip link show

# Comprobar el numero de conexiones activas
ss -t state established

# Comprobar usuarios conectados
who
w
```

---

## Preguntas Comunes de Evaluacion a Preparar

**P: Cual es la diferencia entre aptitude y apt?**

R: aptitude es un gestor de paquetes de nivel superior con interfaz de texto y mejor resolucion de dependencias. apt es mas simple y mas comunmente usado. Ambos usan los mismos repositorios.

**P: Que es AppArmor?**

R: AppArmor es un modulo de seguridad de Linux que restringe las capacidades de los programas usando perfiles por programa basados en rutas de archivo. Proporciona control de acceso obligatorio.

**P: Que es SELinux?**

R: SELinux (Security-Enhanced Linux) es un modulo de seguridad que usa etiquetas y contextos de seguridad para aplicar politicas de control de acceso obligatorio. Se usa en RHEL/Rocky.

**P: Que es UFW?**

R: UFW (Uncomplicated Firewall) es un frontend de iptables/nftables que simplifica la configuracion del firewall en sistemas Debian/Ubuntu.

**P: Que es firewalld?**

R: firewalld es un gestor de firewall dinamico que usa zonas y servicios. Es el predeterminado en sistemas RHEL/Rocky/Fedora.

**P: Por que no podemos conectarnos por SSH como root?**

R: El acceso SSH de root esta deshabilitado (`PermitRootLogin no`) para prevenir ataques de fuerza bruta sobre la cuenta mas privilegiada. Los usuarios deben conectarse por SSH como usuarios regulares y usar sudo.

**P: Que es LVM?**

R: LVM (Logical Volume Manager) permite una gestion flexible del disco abstrayendo el almacenamiento fisico en volumenes logicos que pueden ser redimensionados, movidos o crear snapshots.

**P: Que hace el script monitoring.sh?**

R: Recopila informacion del sistema (arquitectura, CPU, memoria, disco, red, etc.) y la transmite a todas las terminales usando el comando `wall` cada 10 minutos mediante cron.

---

*Fin de la Guia de Evaluacion*
