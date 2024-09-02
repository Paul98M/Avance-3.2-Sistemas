# ![image](https://github.com/user-attachments/assets/c8e698d0-d2e4-4c6c-a0ea-885d90addfb6)

## Procedimiento seguido para configurar una central telefónica en CentOS Stream 9
				
## ##
**Paso 1: Ingresar como root**

Para ingresar como superusuario en CentOS 9, se coloca el siguiente comando:

`su -`

## ##
**Paso 2: Actualización del sistema**

Se actualizó la información de los repositorios utilizando el siguiente comando:

`sudo dnf update -y`

![image](https://github.com/user-attachments/assets/609a62e9-e8fb-4c5a-94d9-005853b1e8b1)


## ##
**Paso 3: Deshabilitar SELinux**

Para garantizar que no haya conflictos durante la instalación de Asterisk, es recomendable deshabilitar SELinux. Por lo cual hay que realizar cambios en el archivo '/etc/selinux/config'.

`nano /etc/selinux/config`

Cambiar la política de acción de aplicada a deshabilitada

`SELINUX=disabled`

![image](https://github.com/user-attachments/assets/e5c934a6-ffb4-4b9d-b5e0-03a781479be8)

Despues es nesecario guardar el archivo y reinicie el sistema:

`sudo reboot`

## ##
**Paso 4: Instalar herramientas de desarrollo**

Asterisk requiere varias bibliotecas y herramientas de desarrollo. Se intalaron con los comandos:

```
sudo dnf groupinstall 'Development Tools' -y
sudo dnf install wget vim git tar -y
```

## ##
**Paso 5: Instalar las dependencias de Asterisk**

Se instalaron bibliotecas necesarias para que Asterisk funcione con el comando:

`sudo dnf install libxml2-devel sqlite-devel libuuid-devel -y`

## ##
**Paso 6: Descarga y extracción del paquete Asterisk**

Nos dirigimos al directorio /usr/src, que es una ubicación estándar en Linux donde se suelen almacenar los archivos fuente (source) de aplicaciones y software a compilar.

`cd /usr/src`

se utiliza wget para descargar el archivo comprimido más reciente de Asterisk versión 18 desde el sitio oficial.

`wget https://downloads.asterisk.org/pub/telephony/asterisk/asterisk-18-current.tar.gz`

Este comando utiliza tar para descomprimir el archivo descargado. 

`tar zxvf asterisk-18-current.tar.gz`

Después de la extracción, se elimina el archivo tarball comprimido para liberar espacio en el disco y mantener el entorno limpio. 

`rm -rf asterisk-18-current.tar.gz`

Finalmente cambiamos el directorio de trabajo al nuevo directorio de Asterisk que se creó durante la descompresión. El uso del comodín * permite entrar en el directorio sin necesidad de escribir el nombre completo.

`cd asterisk-18*/`

## ##
**Paso 7: Instalación de las dependencias necesarias**

En este paso, se ejecutará un script incluido en el paquete de Asterisk que se encarga de instalar todas las dependencias necesarias para que Asterisk funcione correctamente en el sistema.

`contrib/scripts/install_prereq install`

**Salida del comando:**
![image](https://github.com/user-attachments/assets/881ee1f2-04bc-464f-aa1d-d90d7c973ba4)

## ##
**Paso 8: Configuración del entorno de compilación**

En este paso, se configura el entorno para la compilación de Asterisk. Este comando prepara el sistema para compilar Asterisk con las opciones específicas seleccionadas. 

`./configure --with-jansson-bundled`

Luego de completar exitosamente la configuración del software veremos el logo del sistema en forma de asterisco.

![image](https://github.com/user-attachments/assets/a8fd0ec9-d70b-4421-a812-f03d4b57b080)

## ##
**Paso 9: Compilación y configuración de Asterisk**

Se abre  una interfaz de menú en la qus se puede activar o desactivar diferentes opciones, como codecs, módulos de red, aplicaciones, y otras características específicas de Asterisk con el comando:

`make menuselect`

![image](https://github.com/user-attachments/assets/cff8e8e2-7f28-42e7-b8b3-58947be7f2f7)

Se inicia el proceso de compilación de Asterisk utilizando los archivos de configuración generados anteriormente. Durante este proceso, todo el código fuente de Asterisk se convierte en ejecutables y bibliotecas que podrán ser instalados en el sistema con el comando:

`make`

**Salida del comando:**
![image](https://github.com/user-attachments/assets/4f7eaa17-bd3a-4424-9c43-427e308102ee)

Una vez completada la compilación, make install copia los archivos ejecutables, bibliotecas, y demás componentes necesarios al sistema, ubicándolos en los directorios adecuados.  En este paso se instala Asterisk en el servidor, haciéndolo disponible para su configuración y uso. 

`make install`

**Salida del comando:**
![image](https://github.com/user-attachments/assets/bc78a812-8c66-447d-97ee-08ac58c7551a)

Se instalo un conjunto de archivos de configuración de ejemplo que son útiles para comenzar a trabajar con Asterisk.

`make samples`

Finalmente, make config configura Asterisk para que se ejecute como un servicio en el sistema.

## ##
**Paso 10: Creación y configuración del servicio systemd para Asterisk**

Se genero un problema con el servicio del sistema por lo cual se creará un archivo de servicio de systemd para gestionar Asterisk.

`touch /usr/lib/systemd/system/asterisk.service`

Finalmente se escribe la configuración detallada en el archivo systemd

```
cat <<'EOF' >/usr/lib/systemd/system/asterisk.service
[Unit]
Description=Asterisk PBX and telephony daemon.
#After=network.target
#include these if asterisk need to bind to a specific IP (other than 0.0.0.0)
Wants=network-online.target
After=network-online.target network.target

[Service]
Type=simple
Environment=HOME=/var/lib/asterisk
WorkingDirectory=/var/lib/asterisk
ExecStart=/usr/sbin/asterisk -mqf -C /etc/asterisk/asterisk.conf
ExecReload=/usr/sbin/asterisk -rx 'core reload'
ExecStop=/usr/sbin/asterisk -rx 'core stop now'

LimitCORE=infinity
Restart=always
RestartSec=4

# Prevent duplication of logs with color codes to /var/log/messages
StandardOutput=null

PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF
```
 
## ##
**Paso 11: Habilitar e iniciar asterisk**
Se agregó el servicio Asterisk al inicio, se inició y se verificó su estado.

```
systemctl enable asterisk.service
systemctl start asterisk
systemctl status asterisk
```
Si se ve de la siguiente manera signifa qeu asterisk esta funcionado y lito para usarse.

![image](https://github.com/user-attachments/assets/b7dba681-9b51-42f7-a2ab-46e8844b392e)

## ##
**Paso 12: Configuración de pjsip.conf y extensions.conf**

Una vez que el servicio Asterisk se haya iniciado correctamente, nos dirigimos al directorio /etc/asterisk y ver todos los archivos de configuracion.
```
cd /etc/asterisk
ls
```

![image](https://github.com/user-attachments/assets/a5262de3-22df-431d-9bd1-7281ce5dc3cb)

Para configurar Asterisk como una central telefónica, se deben ajustar los archivos de configuración 'pjsip.conf' y 'extensions.conf'. Primero, se eliminan los ejemplos predeterminados para empezar desde cero.

```
> pjsip.conf
> extensions.conf
```

### 1. Configurar pjsip.conf:

En este archivo se definen los detalles de los dispositivos y la autenticación.

- [transport]: Define el tipo de transporte para las comunicaciones, en este caso, UDP en el puerto 5060.
  
- [100]: Configura el primer teléfono (o extensión) con el identificador 100. Especifica el contexto, los codecs permitidos, y cómo autenticarse.

- [100auth]: Configura la autenticación para el teléfono 100 con un nombre de usuario y contraseña.

- [101] y [101auth]: Configura un segundo teléfono (o extensión) con el identificador 101 de manera similar.

Se realizo esta configuracion para la conexion y autenticacion de dos teléfonos IP en la red y se coloco la siguiente configuracion:
```
[transport]
type=transport
protocol=udp
bind=192.168.1.9:5060

[100]
type=endpoint
context=default
disallow=all
allow=gsm
aors=100
auth=100auth
transport=transport

[100]
type=aor
max_contacts=1
contact=sip:100@192.168.1.9:5060

[100auth]
type=auth
auth_type=userpass
username=100
password=pass100

[101]
type=endpoint
context=default
disallow=all
allow=gsm
aors=101
auth=101auth
transport=transport

[101]
type=aor
max_contacts=1
contact=sip:101@192.168.1.9:5060

[101auth]
type=auth
auth_type=userpass
username=101
password=pass101
```

### 2. Configurar extensions.conf:

Aqui se definen cómo se manejan las llamadas entre extensiones y las acciones a tomar cuando se marca un número específico.

- [default]: Este es el contexto en el que se definen las reglas para las llamadas. Aquí se permite la comunicación entre las extensiones 100 y 101.
- exten => 100,1,Dial(PJSIP/100): Cuando se marca el número 100, Asterisk llama a la extensión 100.
- exten => 100,n,Hangup(): Después de que la llamada a la extensión 100 termina, se cuelga.
- Para la extencion 101 es lo mismo de la 100
- exten => 200,1,Answer(): Cuando se marca el número 200, Asterisk responde la llamada.
- exten => 200,n,Playback(tt-monkeysintro): Reproduce el sonido tt-monkeysintro al que se ha marcado el número 200.
- exten => 200,n,Playback(tt-monkeys): Luego, reproduce el sonido tt-monkeys.
- exten => 200,n,Hangup(): Finalmente, cuelga la llamada después de reproducir los sonidos.

```
[default]
; Configuración para permitir comunicación entre los usuarios 100 y 101
exten => 100,1,Dial(PJSIP/100)
exten => 100,n,Hangup()

exten => 101,1,Dial(PJSIP/101)
exten => 101,n,Hangup()

; Configuración de la extensión 200 con reproducción de sonidos
exten => 200,1,Answer()
exten => 200,n,Playback(tt-monkeysintro)
exten => 200,n,Playback(tt-monkeys)
exten => 200,n,Hangup()
```

## ##
**Paso 13: Configuración del Firewall**
Para permitir que Asterisk funcione correctamente y acepte llamadas, es necesario configurar el firewall para permitir el tráfico en los puertos que utiliza con los comandos:
```
sudo firewall-cmd --zone=public --add-port=5060/udp --permanent
sudo firewall-cmd --zone=public --add-port=10000-20000/udp --permanent
sudo firewall-cmd --reload
```
Despues se verifico que los puertos necesarios para Asterisk están correctamente configurados.

`sudo firewall-cmd --list-all`

![image](https://github.com/user-attachments/assets/5f0ac7ed-bc39-4aa3-a2cb-fabd826c15d7)

## ##
**Paso 13: Pruebas de funcionamiento**

Se verificó que el servicio de Asterisk estuviera funcionando correctamente mediante la realización de llamadas. Para ello, se utilizó la aplicación Zoiper la cual es una aplicación de cliente SIP que permite realizar y recibir llamadas a través de Asterisk.

![Captura de pantalla 2024-08-29 121800](https://github.com/user-attachments/assets/6cd07df7-9050-420f-9782-774fa7380a80)








