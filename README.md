*This project has been created as part of the 42 curriculum by mario*

## Descripcion

Born2BeRoot es un proyecto de administracion de sistemas del plan de estudios de 42. El objetivo es configurar una maquina virtual (VM) con un servidor Linux minimo, siguiendo estrictas reglas de seguridad. El proyecto cubre habilidades esenciales de administracion de sistemas: instalacion del SO, particionado con LUKS/LVM, configuracion de servicios (SSH, UFW), gestion de usuarios y grupos, politicas de contrasenas, endurecimiento de sudo, y monitoreo automatizado mediante scripts bash y cron.

La VM se configura sin ninguna interfaz grafica, usando solo herramientas de linea de comandos. Todos los servicios estan endurecidos para cumplir con las mejores practicas de seguridad, y un script de monitoreo transmite informacion del sistema cada 10 minutos a todas las terminales conectadas.

## Instrucciones

### Requisitos previos

- VirtualBox instalado en tu maquina host
- Imagen ISO de Debian 12 (ultima version estable) descargada de https://www.debian.org/
- Git instalado en tu maquina host

### Instalacion

1. Clona este repositorio:
   ```
   git clone <url_del_repositorio>
   ```

2. Crea una nueva maquina virtual en VirtualBox:
   - Tipo: Linux
   - Version: Debian (64-bit)
   - RAM: 1024 MB (minimo recomendado)
   - Disco: Crear un disco duro virtual (VDI, asignado dinamicamente, ~8 GB recomendado)

3. Monta la ISO de Debian y arranca la VM.

4. Sigue la guia paso a paso en `project_guide.txt` para completar la instalacion y configuracion.

5. Una vez terminado, genera la firma del disco y guardala en `signature.txt`:
   - Windows: `certUtil -hashfile <nombre_vm>.vdi sha1`
   - Linux/Mac: `sha1sum <nombre_vm>.vdi`

### Ejecucion de la VM

- Arranca la VM desde VirtualBox.
- Conectate por SSH al servidor: `ssh <usuario>@localhost -p 4242`
- El script de monitoreo se ejecuta automaticamente cada 10 minutos mediante cron.

## Descripcion del Proyecto

### Eleccion del Sistema Operativo: Debian

Se eligio Debian sobre Rocky Linux para este proyecto. A continuacion se explica el razonamiento y la comparacion de tecnologias clave.

### Debian vs Rocky Linux

| Caracteristica       | Debian                              | Rocky Linux                          |
|----------------------|-------------------------------------|--------------------------------------|
| Base                 | Distribucion comunitaria independiente | Derivado de RHEL (empresarial)    |
| Gestor de paquetes   | apt / aptitude                      | dnf / yum                            |
| Modulo de seguridad  | AppArmor                            | SELinux                              |
| Firewall             | UFW (frontend de iptables/nftables) | firewalld                            |
| Dificultad           | Mas facil, mejor para principiantes | Mas complejo, orientado a empresas   |
| Documentacion        | Amplia documentacion comunitaria    | Documentacion empresarial            |
| Estabilidad          | Muy estable, lanzamientos probados  | Muy estable, compatible con RHEL     |

Debian es recomendado para principiantes debido a su configuracion mas sencilla, amplia documentacion y gestion de paquetes mas directa. Rocky Linux anade complejidad con politicas SELinux y configuracion de firewalld, que aunque potentes, son mas dificiles de aprender desde cero.

### AppArmor vs SELinux

| Caracteristica       | AppArmor                            | SELinux                              |
|----------------------|-------------------------------------|--------------------------------------|
| Enfoque              | Basado en rutas (perfiles por programa) | Basado en etiquetas (contextos de seguridad) |
| Complejidad          | Mas simple de configurar y entender | Mas complejo, curva de aprendizaje alta |
| Predeterminado en    | Debian, SUSE, Ubuntu                | RHEL, Rocky, Fedora, CentOS          |
| Granularidad         | Nivel de ruta de archivo            | Etiquetado fino de objetos           |
| Curva de aprendizaje | Baja                                | Alta                                 |

AppArmor restringe programas mediante rutas de archivo definidas en perfiles, haciendolo intuitivo. SELinux usa etiquetas de seguridad en todos los objetos y aplica politicas de control de acceso obligatorio, ofreciendo mayor granularidad pero requiriendo un entendimiento mas profundo.

### UFW vs firewalld

| Caracteristica       | UFW                                 | firewalld                            |
|----------------------|-------------------------------------|--------------------------------------|
| Base                 | Frontend de iptables/nftables       | Frontend de nftables (dinamico)      |
| Complejidad          | Muy simple, basado en comandos      | Moderada, basado en zonas            |
| Estilo de configuracion | Reglas estaticas                 | Reglas dinamicas con zonas           |
| Predeterminado en    | Debian, Ubuntu                      | RHEL, Rocky, Fedora                  |
| Facilidad de uso     | Mas facil para configuraciones simples | Mejor para topologias de red complejas |

UFW es una herramienta de gestion de firewall sencilla, ideal para configuraciones de servidor simples. firewalld usa zonas y servicios para una gestion de red mas dinamica, adecuada para entornos empresariales.

### VirtualBox vs UTM

| Caracteristica       | VirtualBox                          | UTM                                  |
|----------------------|-------------------------------------|--------------------------------------|
| Plataforma           | Windows, Linux, macOS (Intel)       | macOS (Apple Silicon e Intel)        |
| Virtualizacion       | Hipervisor tipo 2                   | Basado en QEMU (tipo 2)              |
| Formato de disco     | VDI / VMDK                          | qcow2                                |
| Snapshots            | Soportado (pero prohibido aqui)     | Soportado (pero prohibido aqui)      |
| Rendimiento          | Bueno para invitados x86/x64        | Bueno para invitados ARM en Apple Silicon |
| Coste                | Gratuito (codigo abierto)           | Gratuito (codigo abierto)            |

VirtualBox es la opcion mas comun para virtualizacion x86/x64 en multiples plataformas. UTM es la opcion preferida para Macs con Apple Silicon donde VirtualBox no esta disponible.

### Decisiones de Diseno

- **Particionado**: LVM con cifrado LUKS para al menos 2 particiones (raiz y swap/home), asegurando cifrado de disco y gestion flexible de volumenes.
- **Politicas de seguridad**: AppArmor activado en el arranque, firewall UFW con solo el puerto 4242 abierto, acceso SSH restringido a usuarios no root en el puerto 4242.
- **Politica de contrasenas**: Expiracion cada 30 dias, minimo 2 dias entre cambios, aviso 7 dias antes, 10+ caracteres con requisitos de complejidad, exclusion del nombre de usuario.
- **Gestion de usuarios**: Usuario regular en los grupos `user42` y `sudo`, acceso SSH de root deshabilitado.
- **Servicios**: Solo SSH (puerto 4242) y UFW instalados. Sin interfaz grafica (X.org/Wayland prohibidos).
- **Monitoreo**: Un script bash (`monitoring.sh`) se ejecuta cada 10 minutos mediante cron, transmitiendo estadisticas del sistema a todas las terminales usando `wall`.

## Recursos

- [Guia de Instalacion de Debian](https://www.debian.org/releases/stable/installmanual)
- [Wiki de Debian - Cifrado LUKS](https://wiki.debian.org/LUKS)
- [Wiki de Debian - LVM](https://wiki.debian.org/LVM)
- [Documentacion de AppArmor](https://gitlab.com/apparmor/apparmor/-/wikis/home)
- [UFW - Uncomplicated Firewall](https://help.ubuntu.com/community/UFW)
- [Guia de Endurecimiento SSH](https://www.ssh.com/academy/ssh/hardening)
- [Manual de Sudo](https://www.sudo.dev/man/)
- [Cron Howto](https://help.ubuntu.com/community/CronHowto)
- [Comando `wall` de Linux](https://man7.org/linux/man-pages/man1/wall.1.html)

### Uso de IA

Se utilizaron herramientas de IA como referencia y ayuda de aprendizaje durante este proyecto. Especificamente:
- Comprension de conceptos de administracion de sistemas Linux (particionado, LVM, LUKS, AppArmor).
- Depuracion de archivos de configuracion (sudoers, sshd_config, politicas de contrasenas).
- Escritura y prueba de la logica del script bash `monitoring.sh`.
- Explicacion de diferencias entre herramientas del sistema y modulos de seguridad.

Todas las configuraciones y scripts fueron aplicados y verificados manualmente en la maquina virtual. La IA se utilizo solo como guia; no se realizo ninguna configuracion automatizada.
