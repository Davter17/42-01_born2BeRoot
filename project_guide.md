# Born2BeRoot - Guia del Proyecto

## Configuracion Paso a Paso (Debian)

## Indice

1. [Creacion de la VM en VirtualBox](#paso-1-creacion-de-la-vm-en-virtualbox)
2. [Instalacion de Debian con Particiones LVM Cifradas](#paso-2-instalacion-de-debian-con-particiones-lvm-cifradas)
3. [Primer Arranque y Configuracion Inicial](#paso-3-primer-arranque-y-configuracion-inicial)
4. [Configuracion del Hostname](#paso-4-configuracion-del-hostname)
5. [Gestion de Usuarios y Grupos](#paso-5-gestion-de-usuarios-y-grupos)
6. [Configuracion de la Politica de Contrasenas](#paso-6-configuracion-de-la-politica-de-contrasenas)
7. [Configuracion de Sudo](#paso-7-configuracion-de-sudo)
8. [Configuracion de SSH](#paso-8-configuracion-de-ssh)
9. [Configuracion del Firewall UFW](#paso-9-configuracion-del-firewall-ufw)
10. [Verificacion de AppArmor](#paso-10-verificacion-de-apparmor)
11. [Script de Monitoreo (monitoring.sh)](#paso-11-script-de-monitoreo-monitoringsh)
12. [Configuracion de Cron](#paso-12-configuracion-de-cron)
13. [Verificaciones Finales y Firma del Disco](#paso-13-verificaciones-finales-y-firma-del-disco)

---

## Paso 1: Creacion de la VM en VirtualBox

1. Abre VirtualBox y haz clic en **"Nueva"**.
2. **Nombre:** `born2beroot` (o el nombre que prefieras)
3. **Tipo:** Linux
4. **Version:** Debian (64-bit)
5. **Memoria (RAM):** 1024 MB (minimo recomendado)
6. **Disco duro:** "Crear un disco duro virtual ahora"
7. **Tipo de disco:** VDI (VirtualBox Disk Image)
8. **Almacenamiento:** "Reservado dinamicamente"
9. **Tamano del disco:** 8.00 GB (suficiente para este proyecto)
10. Haz clic en **"Crear"**.

> **IMPORTANTE:** NO actives aceleracion 3D ni ninguna caracteristica grafica.
>
> **IMPORTANTE:** NO crees ningun snapshot (estan prohibidos).

---

## Paso 2: Instalacion de Debian con Particiones LVM Cifradas

1. Monta la ISO estable de Debian en la configuracion de la VM (**Almacenamiento > CD/DVD**).
2. Inicia la VM y arranca desde la ISO.
3. Selecciona **"Graphical Install"** (o **"Install"** para modo texto).
4. Elige idioma, pais y distribucion de teclado.
5. Configura la red: establece un hostname (ej., `wil42`) o configuralo despues.
   - **Hostname:** `login42` (ej., `wil42`)
   - **Nombre de dominio:** dejar vacio

### Particionado (PASO CRITICO)

6. Cuando pregunte sobre particionado, selecciona **"Manual"**.
7. Selecciona el disco virtual y crea una nueva tabla de particiones si se solicita.
8. Crea las siguientes particiones:

#### a) Particion de arranque (sin cifrar, ext4)

- **Tamano:** 500 MB
- **Tipo:** Primaria
- **Ubicacion:** Inicio del espacio
- **Usar como:** Sistema de archivos con registro ext4
- **Punto de montaje:** `/boot`

#### b) Configurar particion cifrada para LVM

- Selecciona **"Configurar volumenes cifrados"** > Si
- Selecciona **"Crear volumenes cifrados"**
- Selecciona el espacio libre del disco
- **Tamano:** usa TODO el espacio restante
- Elige una frase de contrasena fuerte para el cifrado (recuerdala!)
- Finaliza y escribe los cambios al disco

#### c) Configurar LVM sobre el volumen cifrado

- Selecciona **"Configurar el Gestor de Volumenes Logicos"** > Si
- Crear grupo de volumenes:
  - **Nombre:** `lvm_group` (o cualquier nombre que prefieras)
  - **Dispositivo:** selecciona la particion cifrada
- Crear volumenes logicos dentro del grupo de volumenes:

| LV | Nombre | Tamano |
|----|--------|--------|
| 1 - Raiz | `root` | ~5 GB (o apropiado para el tamano de tu disco) |
| 2 - Swap | `swap` | ~1 GB (o igual al tamano de tu RAM) |
| 3 - Home (opcional) | `home` | espacio restante |

- Finaliza la configuracion de LVM

#### d) Asignar puntos de montaje a los volumenes logicos

- Selecciona el LV `root` > **Usar como:** Ext4 > **Punto de montaje:** `/`
- Selecciona el LV `swap` > **Usar como:** area de intercambio
- Selecciona el LV `home` (si se creo) > **Usar como:** Ext4 > **Punto de montaje:** `/home`

9. Finaliza el particionado y escribe los cambios al disco.

### Completar la Instalacion

10. Establece la contrasena de root (debe cumplir la politica de contrasenas).
11. Crea tu cuenta de usuario:
    - **Nombre completo:** tu login
    - **Nombre de usuario:** tu login
    - **Contrasena:** debe cumplir la politica de contrasenas
12. Configura el gestor de paquetes (apt):
    - Elige un mirror (o usa el predeterminado)
13. Seleccion de software:
    - **SOLO** selecciona **"SSH server"** y **"standard system utilities"**
    - **NO** selecciones "Debian desktop environment" ni ninguna opcion grafica
14. Instala el cargador de arranque GRUB en el disco primario.
15. Reinicia y retira la ISO.

---

## Paso 3: Primer Arranque y Configuracion Inicial

1. Arranca la VM. Se te pedira la frase de contrasena de cifrado.
2. Inicia sesion como root primero para instalar paquetes adicionales:

```bash
su -
```

3. Instala los paquetes necesarios:

```bash
apt update && apt upgrade -y
apt install -y sudo vim lsblk
```

4. Verifica que no hay interfaz grafica instalada:

```bash
ls /usr/bin/*session
# No deberia devolver nada o ninguna sesion de X.org/wayland
```

5. Verifica que AppArmor esta funcionando:

```bash
apparmor_status
# Deberia mostrar perfiles cargados
```

Si no esta activo, habilitalo:

```bash
systemctl enable apparmor
systemctl start apparmor
```

---

## Paso 4: Configuracion del Hostname

1. Establece el hostname (tu login + 42):

```bash
hostnamectl set-hostname <tu_login>42
# Ejemplo: hostnamectl set-hostname wil42
```

2. Edita `/etc/hosts` para actualizar las referencias:

```bash
nano /etc/hosts
```

Cambia:

```
127.0.0.1   <tu_login>42 localhost
::1         <tu_login>42 localhost
```

3. Reinicia para aplicar los cambios:

```bash
reboot
```

4. Verifica:

```bash
hostname
# Deberia mostrar: <tu_login>42
```

---

## Paso 5: Gestion de Usuarios y Grupos

1. Crea el grupo `user42`:

```bash
addgroup user42
```

2. Crea tu usuario (si no se creo durante la instalacion) y anadelo a los grupos:

```bash
adduser <tu_login>
usermod -aG user42,sudo <tu_login>
```

3. Verifica la pertenencia a grupos:

```bash
getent group user42
getent group sudo
# Tu usuario deberia aparecer en ambos grupos
```

4. Verifica que el usuario existe:

```bash
id <tu_login>
```

---

## Paso 6: Configuracion de la Politica de Contrasenas

1. Edita `/etc/login.defs` para establecer la caducidad de contrasenas:

```bash
nano /etc/login.defs
```

Modifica las siguientes lineas:

```
PASS_MAX_DAYS   30
PASS_MIN_DAYS   2
PASS_WARN_AGE   7
```

2. Instala `libpam-pwquality` para la complejidad de contrasenas:

```bash
apt install -y libpam-pwquality
```

3. Edita la configuracion de PAM common-password:

```bash
nano /etc/pam.d/common-password
```

Busca la linea que empieza con `password requisite pam_pwquality.so` y modificala a:

```
password    requisite    pam_pwquality.so retry=3 minlen=10 ucredit=-1 lcredit=-1 dcredit=-1 maxrepeat=3 usercheck=1 diffok=7 enforce_for_root
```

**Explicacion de los parametros:**

| Parametro | Descripcion |
|-----------|-------------|
| `minlen=10` | Minimo 10 caracteres |
| `ucredit=-1` | Al menos 1 letra mayuscula |
| `lcredit=-1` | Al menos 1 letra minuscula |
| `dcredit=-1` | Al menos 1 numero |
| `maxrepeat=3` | No mas de 3 caracteres identicos consecutivos |
| `usercheck=1` | La contrasena no debe contener el nombre de usuario |
| `retry=3` | 3 intentos antes de fallar |
| `diffok=7` | Al menos 7 caracteres nuevos (no aplica a root) |
| `enforce_for_root` | Aplica la politica tambien a root |

4. Aplica los cambios a los usuarios existentes cambiando todas las contrasenas:

```bash
passwd root
passwd <tu_login>
# Asegurate de que todas las contrasenas cumplan la nueva politica.
```

5. Establece la caducidad de contrasenas para los usuarios existentes:

```bash
chage -M 30 -m 2 -W 7 root
chage -M 30 -m 2 -W 7 <tu_login>
```

---

## Paso 7: Configuracion de Sudo

1. Instala sudo (si no esta instalado):

```bash
apt install -y sudo
```

2. Crea el archivo de configuracion de sudo:

```bash
nano /etc/sudoers.d/sudo_config
```

3. Anade el siguiente contenido:

```
Defaults    env_reset
Defaults    secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
Defaults    mail_badpass
Defaults    passwd_tries=3
Defaults    badpass_message="Contrasena incorrecta. Acceso denegado. Este incidente sera reportado."
Defaults    logfile="/var/log/sudo/sudo.log"
Defaults    log_input, log_output
Defaults    requiretty
```

**Explicacion:**

| Regla | Descripcion |
|-------|-------------|
| `passwd_tries=3` | Limita a 3 intentos de autenticacion |
| `badpass_message` | Mensaje de error personalizado al introducir contrasena incorrecta |
| `logfile` | Registra la actividad de sudo en `/var/log/sudo/sudo.log` |
| `log_input, log_output` | Archiva todas las entradas y salidas |
| `requiretty` | Activa el modo TTY por seguridad |
| `secure_path` | Restringe las rutas utilizables por sudo |

4. Crea el directorio de logs de sudo:

```bash
mkdir -p /var/log/sudo
```

5. Establece los permisos adecuados:

```bash
chmod 750 /var/log/sudo
chown root:root /var/log/sudo
```

6. Verifica que el archivo sudoers es valido:

```bash
visudo -c
```

7. Prueba que sudo funciona para tu usuario:

```bash
su - <tu_login>
sudo whoami
# Deberia mostrar: root
```

---

## Paso 8: Configuracion de SSH

1. Instala el servidor SSH (si no se instalo durante la configuracion):

```bash
apt install -y openssh-server
```

2. Edita la configuracion del demonio SSH:

```bash
nano /etc/ssh/sshd_config
```

Modifica/anade las siguientes lineas:

```
Port 4242
PermitRootLogin no
PasswordAuthentication yes
```

> **IMPORTANTE:** Cambia `Port 22` a `Port 4242`
>
> **IMPORTANTE:** Establece `PermitRootLogin no` para bloquear el acceso SSH de root

3. Edita tambien la configuracion del cliente SSH (para referencia):

```bash
nano /etc/ssh/ssh_config
```

Anade o verifica:

```
Port 4242
```

4. Reinicia el servicio SSH:

```bash
systemctl restart ssh
```

5. Habilita SSH para que arranque al inicio:

```bash
systemctl enable ssh
```

6. Verifica que SSH esta funcionando en el puerto 4242:

```bash
ss -tlnp | grep 4242
```

7. Prueba la conexion SSH (desde la propia VM):

```bash
ssh <tu_login>@localhost -p 4242
```

8. Verifica que root no puede conectarse por SSH:

```bash
ssh root@localhost -p 4242
# Deberia ser denegado
```

---

## Paso 9: Configuracion del Firewall UFW

1. Instala UFW:

```bash
apt install -y ufw
```

2. Habilita UFW:

```bash
ufw enable
```

3. Permite SSH en el puerto 4242:

```bash
ufw allow 4242/tcp
```

4. Deniega todo el trafico entrante (politica por defecto):

```bash
ufw default deny incoming
```

5. Permite el trafico saliente (politica por defecto):

```bash
ufw default allow outgoing
```

6. Verifica el estado del firewall:

```bash
ufw status verbose
```

La salida esperada deberia mostrar:

```
Status: active
4242/tcp    ALLOW    Anywhere
```

7. Verifica que UFW arranca al inicio:

```bash
systemctl enable ufw
```

8. Lista las reglas numeradas (para gestion):

```bash
ufw status numbered
```

---

## Paso 10: Verificacion de AppArmor

1. Verifica que AppArmor esta instalado y activo:

```bash
apparmor_status
```

2. Verifica que AppArmor esta habilitado en el arranque:

```bash
systemctl status apparmor
```

3. Si no esta activo, habilitalo:

```bash
systemctl enable apparmor
systemctl start apparmor
```

4. Verifica que muestra perfiles cargados.

---

## Paso 11: Script de Monitoreo (monitoring.sh)

1. Crea el script:

```bash
nano /root/monitoring.sh
```

2. Pega el siguiente contenido:

```bash
#!/bin/bash

ARCH=$(uname -a)
PHYSICAL_CPU=$(grep "physical id" /proc/cpuinfo | sort -u | wc -l)
VCPU=$(nproc)
MEM_TOTAL=$(free -m | awk '/^Mem:/{print $2}')
MEM_USED=$(free -m | awk '/^Mem:/{print $3}')
MEM_PERCENT=$(awk "BEGIN {printf \"%.2f\", ($MEM_USED/$MEM_TOTAL)*100}")
DISK_TOTAL=$(df -m --total | grep '^total' | awk '{print $2}')
DISK_USED=$(df -m --total | grep '^total' | awk '{print $3}')
DISK_TOTAL_GB=$(awk "BEGIN {printf \"%.1f\", $DISK_TOTAL/1024}")
DISK_USED_GB=$(awk "BEGIN {printf \"%.0f\", $DISK_USED/1024}")
DISK_PERCENT=$(awk "BEGIN {printf \"%.0f\", ($DISK_USED/$DISK_TOTAL)*100}")
CPU_LOAD=$(top -bn1 | grep '^%Cpu' | awk '{printf "%.1f", 100 - $8}')
LAST_BOOT=$(who -b | awk '{print $3, $4}')

if lsblk | grep -q "lvm"; then
    LVM_STATUS="yes"
else
    LVM_STATUS="no"
fi

TCP_CONN=$(ss -t state established | tail -n +2 | wc -l)
NUM_USERS=$(who | wc -l)
IP_ADDR=$(hostname -I | awk '{print $1}')
MAC_ADDR=$(ip link show | awk '/link/ether/{print $2; exit}')
SUDO_COUNT=$(cat /var/log/sudo/sudo.log 2>/dev/null | grep "COMMAND" | wc -l)

wall "
#Architecture: $ARCH
#CPU physical : $PHYSICAL_CPU
#vCPU : $VCPU
#Memory Usage: ${MEM_USED}/${MEM_TOTAL}MB (${MEM_PERCENT}%)
#Disk Usage: ${DISK_USED_GB}/${DISK_TOTAL_GB}Gb (${DISK_PERCENT}%)
#CPU load: ${CPU_LOAD}%
#Last boot: $LAST_BOOT
#LVM use: $LVM_STATUS
#Connections TCP : $TCP_CONN ESTABLISHED
#User log: $NUM_USERS
#Network: IP $IP_ADDR ($MAC_ADDR)
#Sudo : $SUDO_COUNT cmd
"
```

3. Haz el script ejecutable:

```bash
chmod +x /root/monitoring.sh
```

4. Prueba el script manualmente:

```bash
/root/monitoring.sh
# Deberias ver el mensaje de broadcast en tu terminal
```

---

## Paso 12: Configuracion de Cron

1. Edita el crontab de root:

```bash
crontab -u root -e
```

2. Anade la siguiente linea para ejecutar cada 10 minutos:

```
*/10 * * * * /root/monitoring.sh
```

3. Guarda y sal.

4. Verifica que el cron job esta registrado:

```bash
crontab -u root -l
```

5. Asegurate de que el servicio cron esta funcionando:

```bash
systemctl enable cron
systemctl start cron
systemctl status cron
```

---

## Paso 13: Verificaciones Finales y Firma del Disco

1. Verifica que todos los servicios estan funcionando:

```bash
systemctl status ssh
systemctl status ufw
systemctl status apparmor
systemctl status cron
```

2. Verifica el hostname:

```bash
hostname
```

3. Verifica los grupos del usuario:

```bash
id <tu_login>
```

4. Verifica que no hay interfaz grafica:

```bash
ls /usr/bin/*session
dpkg -l | grep -i xorg
dpkg -l | grep -i wayland
```

5. Verifica el esquema de particiones:

```bash
lsblk
```

6. Verifica SSH en el puerto 4242:

```bash
ss -tlnp | grep 4242
```

7. Verifica UFW:

```bash
ufw status
```

8. Genera la firma del disco:

**En Windows:**

```
certUtil -hashfile "C:\Users\<usuario>\VirtualBox VMs\<nombre_vm>\<nombre_vm>.vdi" sha1
```

**En Linux:**

```bash
sha1sum ~/VirtualBox\ VMs/<nombre_vm>/<nombre_vm>.vdi
```

9. Guarda la firma en `signature.txt` en la raiz de tu repositorio:

```bash
nano signature.txt
# Pega el hash SHA1
```

10. Haz commit de todos los archivos a tu repositorio Git:

```bash
git add README.md project_guide.md evaluation_guide.md signature.txt
git commit -m "Documentacion completa de Born2BeRoot"
```

---

*Fin de la Guia del Proyecto*
