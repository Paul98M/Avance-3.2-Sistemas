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

## ##
**Paso 3: Deshabilitar SELinux**

Para garantizar que no haya conflictos durante la instalación de Asterisk, es recomendable deshabilitar SELinux. Por lo cual hay que realizar cambios en el archivo '/etc/selinux/config'.

`nano /etc/selinux/config`

Cambiar la política de acción de aplicada a deshabilitada

`SELINUX=disabled`

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














