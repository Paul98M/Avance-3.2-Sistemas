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



